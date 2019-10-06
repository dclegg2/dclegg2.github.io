---
layout: post
title:      "'Translating' a Visual Language via CNNs"
date:       2019-10-06 16:15:26 +0000
permalink:  translating_a_visual_language_via_cnns
---


* *Cześć, mam na imię*

* *こんにちは、私の名前は *

* *Tere minu nimi on* 



I'm assuming most of you don't speak Polish, Japanese, or Estonian.  I don't either, but it took me just a few seconds to find the above translation of "*Hello, my name is *" using widely available machine translation tools such as Google Translate.  This has been one of the great innovations of the 21st century using deep learning.  The benefits are myriad, facilitating communication worldwide amongst speakers who otherwise would have had an insurmountable language barrier.  However, this functionality is predicated on written input that can be fed into a neural network and broken down into component pieces (sentences > words > syllables > letters).  

What happens when you encounter a language that is *never* expressed in written form?  It becomes signficantly more challenging to "write" a sentence that conveys the same meaning:


![](https://www.signlanguageforum.com/include/fsscripts/php.php?t=LS0taGhoLS0tMS0tLWxsbC0tLWFzbC0tLWZmZi0tLUhFTExPIE1ZIE5BTUUgSVM=)

The below link describes the process (in only 11 steps!)  of introducing oneself in American Sign Language:

[How to Say Your Name in Sign Language](https://www.wikihow.com/Say-Your-Name-in-Sign-Language)



Sign Languages are spoken by well over 10 million people worldwide (estimates vary, with some sources indicating as many as 30 million).  This may very well be the most underrepresented segment of the global population when it comes to machine translation tools.  While this is a complex problem, I've attempted to start down the path of addressing it by building a very rudimentary "translation" engine - using images of single letters as our base unit.  Since this was an image classification task, using a convolutional neural network (CNN) is the logical solution.

One of the things I’ve struggled with throughout my data science journey is making decisions when the “path” seems subjective.  For example, In my first blogpost about a linear regression model, I had trouble deciding what the threshold was to determine when there was too much multicollinearity and certain features would have to be discarded.  This project was no different, but my primary challenge this time was deciding upon the model architecture of the CNN.  There isn’t really a one-size-fits-all way to go about it, and the computational cost involved means that you can’t just throw everything against the wall to see what sticks.  Below I’ll walk through some of the obstacles I faced and the solutions found.

## Inspecting the Data

First, let’s take a quick glance at the data using the following function:

```
def plot_one_sample_of_each(base_path):
    cols = 5
    rows = int(np.ceil(len(classes)/cols))
    fig = plt.figure(figsize=(16,20))
    
    imgs_for_plot = []
    labels_for_plot = []
    for i in range(len(classes)):
        cls = classes[i]
        labels_for_plot.append(cls)
        img_path = base_path + '/' + cls + '/**'
        path_contents = glob(img_path)
        
        img = random.sample(path_contents, 1)
        imgs_for_plot.append(img)
        
        sp = plt.subplot(rows, cols, i+1)
        plt.imshow(cv2.imread(img[0]))
        plt.title(cls)
        sp.axis('off')
        
    plt.show()
    return
```

I used a training dataset consisting of 87,000 images (10% held out for validation): 

![](https://i.imgur.com/arLx9EHl.png)


...and then a testing set of 870 images representing "real-world" data:

![](https://i.imgur.com/gDOz1zCl.png)


The testing set has considerably varied backgrounds, hues, and lighting compared to the relatively static environment in which the training images were produced.

## Building the network architecture

I found myself seeking something like a “gridsearch” method that would help guide me through building an appropriate CNN architecture.  The tidy solution offered by  [scikit-learn](https://scikit-learn.org/0.16/modules/generated/sklearn.grid_search.GridSearchCV.html) doesn't have a Keras equivalent, but I found a suggested method to use nested *for loops* to iterate through a number of convolutional layers, dense layers, and layer sizes. 

```
dense_layers = [0, 1, 2]
layer_sizes = [32, 64, 128]
conv_layers = [1, 2, 3]

for dense_layer in dense_layers:
    for layer_size in layer_sizes:
        for conv_layer in conv_layers:
            NAME = "{}-conv-{}-nodes-{}-dense-{}".format(conv_layer, layer_size, 
						dense_layer, int(time.time()))
            print(NAME)

            model = Sequential()

            model.add(Conv2D(layer_size, (3, 3), input_shape=target_dims))
            model.add(Activation('relu'))
            model.add(MaxPooling2D(pool_size=(2, 2)))

            for l in range(conv_layer-1):
                model.add(Conv2D(layer_size, (3, 3)))
                model.add(Activation('relu'))
                model.add(MaxPooling2D(pool_size=(2, 2)))

            model.add(Flatten())
            for _ in range(dense_layer):
                model.add(Dense(layer_size))
                model.add(Activation('relu'))

            model.add(Dense(29))
            model.add(Activation('softmax'))
```

A brief interlude here to point out that this can be *extremely* computationally expensive if using a CPU kernel.  I didn’t have ready access to a GPU kernel, but one valuable thing I learned on this project is that Kaggle offers free GPU kernel use for up to 30 hours a week.  Using the GPU kernel is literally a game-changer when working with CNNs - the processing speed is 10x-15x faster.  Without that option, I wouldn’t dream of iterating through this many combinations of models; there simply isn’t enough time or computing power to do it on my machine locally.  

The resulting insight from the nested for loops allows us to make some basic assumptions: 
* Generally, the more convolutional layers, the better the performance 
* Using at least 1 dense layer is necessary (there was only a marginal difference between 1 and 2 dense layers, but a drastic difference when choosing to eliminate dense layers entirely) 
* The larger the layer sizes, the better the performance (generally speaking)

Those conclusions may seem like common sense on a certain level, but they can’t be extrapolated to every dataset or deep learning task.  Without going through that exercise, it would be guesswork to say that we're proceeding down the right path just by assuming that "bigger is better".

From there, the model architecture was built out with some additional choices after some experimentation (using strides and max pooling to downsample, dropout layers between each convolutional layer to prevent overfitting, etc.).

## Finding the right optimization algorithm

One thing I still didn’t quite understand was how to choose the “optimizer”, which is an optimization algorithm that minimizes the loss function of a model.  While gradient descent is the traditional convergence algorithm, there are many others that have been developed to mitigate the oscillations of gradient descent and speed up performance.  ’Adam’ is most commonly used for deep neural network models, but in researching I found that there are several other adaptive learning-rate methods (computing adaptive learning rates for each parameter - i.e., the weights and bias in a CNN) that may be better suited for certain image classification tasks.  

I chose to use another *for loop* to iterate through various optimizers and compare performance, using a custom function to fit each to the CNN framework I had already created.  

```
optimizers = ['adam', 'Adagrad', 'Adadelta', 'Adamax']
epochs = [5,10]
for o in optimizers:
    for e in epochs:
        build_fit_test_cnn(o,e)
       
```

The results were interesting - there was a clear “best” performer in the Adadelta optimizer that consistently produced the highest validation accuracy balanced with a low validation loss:

![](https://i.imgur.com/5iSaums.png)

Testing on the "real-world" data yielded roughly 40% accuracy, which was pretty impressive given that the model had 29 classes to consider and the testing images had a lot of “noise” when compared to the training images.  At the end of the day, one might be able to achieve similar results just plugging in a “default” model architecture and the 'Adam' optimizer; but for me, the iterative process of comparing and contrasting model performance using different parameters and layers ultimately provided the piece of mind I needed to comfortably stand by the decisions I’ve made.

With more time and a more robust and diverse dataset, I think there is still a lot of room to further improve and build a neural network that can classify ASL signs with a very high degree of accuracy.

Until then...


![](https://www.signlanguageforum.com/include/fsscripts/php.php?t=LS0taGhoLS0tMS0tLWxsbC0tLWFzbC0tLWZmZi0tLUdPT0RCWUU=)

"Goodbye."







