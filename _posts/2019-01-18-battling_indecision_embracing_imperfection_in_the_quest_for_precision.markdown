---
layout: post
title:      "Battling Indecision: Embracing Imperfection in the Quest for Precision"
date:       2019-01-18 22:57:37 +0000
permalink:  battling_indecision_embracing_imperfection_in_the_quest_for_precision
---



* *How much multicollinearity is too much?*

*  *Can I drop THIS much of the dataset?*

*  *What if I treat (insert feature here) as a categorical variable?  How much relevance do I lose?*

*  *Should I scale my features?  Should I log-normalize them?  Always?  What happens if I don’t?!*

If you guessed that this looks like the desperate inner monologue of a sleep-deprived data science student, you wouldn’t be wrong. These are just a few of the questions that were running through my mind for the last several weeks.  Questions I would constantly ask myself, my instructor, my wife, Google, my dog...really anyone or anything unlucky enough to be near me and my laptop as I tackled my first data science project, building a multivariable linear regression model using a popular housing dataset.  

The answer to each of these questions is…*drumroll please*…**it depends**. It took me a long time to accept that there can be a high degree of subjectivity in executing a task like this. In some ways I was drawn to data science specifically for the innate logic, structure, and precision of it all – the ability to reproduce a line of code and get the expected result, to parse through a large amount of data and say *this* is the answer, the underlying truth.  

What I’ve discovered is that it’s not so simple. One of my biggest takeaways from this exercise is that there is no objective, singular roadmap for how to navigate your way through a free-form project.  At times, I felt like screaming, “*Are there no rules here?!*”

Now, don't get me wrong; of course, there are methods to the madness and general guidelines to follow. However, at the end of the day, if you’re seeking the “perfect” approach on the quest for the one and only solution, you’re likely to be disappointed. YOU have to navigate your way through critical decisions based on your limited experience and knowledge of data science and your dataset, the context of the data, your interpretation of the data, and your ultimate objective.  

A couple of examples stand out in particular:

**Feature selection vis-a-vis multicollinearity: How do I deal with all these variables that run together?** 

After cleaning the dataset and doing exploratory data analysis on each of the variables, one of the first things I did was to create a heatmap to visualize the correlation matrix. There were a lot of variables that had fairly high correlations with our target variable, so that was exciting...right? My approach was to move forward featuring the *sqftliving* variable in my initial model since that demonstrated the highest correlation with our target variable. There was only one problem – a metric ton of other variables were highly correlated with *sqftliving*!

I really wanted to include some of those in my model, but I was staring at correlation values of 0.6 and higher (with *sqftliving*). * Could I use variables with that much collinearity? What’s the standard for a cutoff line? Surely, there’s a logical and universally accepted threshold, right?*

For a while, I grappled with the fact that there was not a definitive answer that would assure me that I was proceeding down the right path moving forward. Some guidance suggests not to use two variables that have a correlation higher than 0.5, while others suggest it could be as high as 0.7.  The threshold, it seems, was basically up to me. I had to decide what threshold I was comfortable with and what level of “noise” I would allow into my regression model for the sake of keeping some or all of those features in it. 

As it turns out, I had to mute my logically-driven brain and listen to my intuition to guide me through some of these critical decision points. It wasn’t easy. For instance, I was having trouble making the decision to just *disregard* bedrooms and bathrooms as potential predictors of housing price. Just think about real estate listings – the first specifications listed are always the number of bedrooms and bathrooms. It’s probably the first thing friends and family ask about a prospective new home if you’re in the market. Wouldn’t it make sense that those are major factors?  

As it turns out, they are – just maybe not as important as the feature I already decided I would like to utilize. Now you can understand why this is an anxiety-inducing process!  Eliminating something from my analysis that I know to be important is perhaps a not-so-distant cousin to FOMO.

**Making the decision to prune the dataset (or drop certain data points): **

There were certain decision points where I certainly could have chosen to drop an entire column because it could’ve been interpreted as a “rare event” or could have dropped null values because they represented such a small proportion of the overall data set. Yet, I decided against that in almost every instance that I could, perhaps to my detriment. I easily could have simplified the task in front of me had I decided to drop data whenever I could rationalize it. And in retrospect...I probably should have.  But in the moment, I felt as if I would somehow betray the integrity of the project if I didn’t fully explore each feature and whether I might be able to glean some significance or predictive value from each, however unlikely.  Perhaps it was an odd and misplaced sense of loyalty to the provided dataset...for whatever reason, I was determined to keep as much data intact as I could. 

There were, however, some instances in which I really couldn’t avoid dropping data. For example, I needed to reduce the skew of the pricing values, and there were just too many outliers on the high end of that feature to leave as is. But how much could I drop?  Again, there isn’t a blueprint for this, and it ultimately came down to my comfort level.  I ended up deciding on the top 0.5% of values but easily could’ve landed on 1%, 2%, or not dropping anything at all!  One could make a sensible argument for any of these decisions.  

This general mentality led me down the narrow and winding path of trying to throw anything and everything against the wall...and hoping something would stick .  If I decided to drop a variable early on in the process, I'd copy the dataset and do the opposite in a parallel draft.  One such approach treated everything that could conceivably be considered categorical as categorical, while another treated most of those same variables as continuous.  I created a LOT more work for myself – the phrase “paralysis by analysis” has a whole new meaning for me now.  

Going back to my indecision with feature selection, I turned to feature engineering to resolve my inner conflict and restore my peace of mind. Feature engineering allowed me to include important variables like *bedrooms, bathrooms, grade*, even seemingly irrelevant variables like *yr_built* and *yr_renovated*, in some form or fashion.  While I didn’t end up actually using any of my newly transformed variables in my final model, I felt satisfied that I had done my due diligence to fully explore the possibilities.  That sense of “completionism” ended up suiting my comfort level and my sensibilities, at least this time around. That said, I fully understand that others’ mileage may vary.

Ultimately, this project felt like the familiar “choose-your-own-adventure” books from childhood: there isn’t one correct way to maneuver through it. Obviously, there are certain fundamental skills of which one needs to exhibit mastery in order to yield a sound model. Fascination lies in both the adventure and the outcomes - giving 20+ students the same dataset and same basic objective probably results in 20+ unique approaches, none of them *wrong*!  Each one of us has to use our discretion and critically think through decisions, but as long as we can comfortably rationalize why we chose a particular path, then we can rest easy. As a wise person told me in the midst of my struggles completing this project: “Don’t let the perfect be the enemy of the good.”  Sage advice indeed.
