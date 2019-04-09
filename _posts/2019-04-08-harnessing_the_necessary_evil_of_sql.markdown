---
layout: post
title:      "Harnessing the “Necessary Evil” of SQL:"
date:       2019-04-09 01:31:28 +0000
permalink:  harnessing_the_necessary_evil_of_sql
---

### Hypothesis Testing with the Northwind Traders database


I heard someone recently refer to SQL as a “necessary evil”, a sentiment that seems to be shared by a lot of the data science community.  No one really likes it, the syntax is clunky, databases are “boring”, etc.  Maybe it’s just me, and maybe it’s because I’ve been subconsciously exposed to a lot of SQL in my job for the last few years (of course, up until a few months ago, giant blocks of SQL code looked like hierogylphics to my untrained eye)…but one thing I’ve become oddly comfortable with is writing SQL statements.  It has probably been the one thing that I’ve encountered so far on my data science journey that has almost immediately “clicked”.  I’ll let everyone else gripe about how awful using SQL can be.  Regardless of how you feel about the language, no one can argue the utility of SQL in accessing and analyzing a large relational database…which is precisely what I was tasked to do recently.

My focus was on exploration of the Northwind Traders database (for the uninitiated, this is a Microsoft-created fictitious database of a specialty foods retailer).  The objective was to utilize some version of SQL to explore the data, compose some interesting questions, and then choose an appropriate method of hypothesis testing (based on experimental design).  In this hypothetical scenario, I'm the newly hired data scientist who is supposed to explore the database and present findings and/or relevant business insights to the stakeholders.  My initial thought process is outlined below, built around the 4 key components of the business:


| Products | Customers| Employees | Suppliers
| -----------| ------------ |------------- |-------------||
| What do we sell?       | Who are they?| How many?|Who do we rely on most?       |
| How much do we sell?|Where are they?|Where are they?|Where are they? |
|How to optimize sales?|What do they order most?|Top performers?|Profit Margins?|


## EDA with SQL

I chose to use SQLAlchemy to create a connection to the database and spent the better part of a day writing SQL statements to explore, which led me down various paths.  My initial query on the “Product” table (and every other table) was as basic as you can get with a SQL statement:

```
pd.read_sql_query("SELECT * FROM Product", engine)
```

The first thing that caught my eye was that *Chef Anton's Gumbo Mix* was "Discontinued". This led me to ask:
* How many of our products are discontinued?
* What categories are they in?
* What commonalities do they share?
* What are their respective price points?
* Why don't these customers respect Creole food?!  Gumbo is delicious!

I refined my SQL statement to dig a little deeper:

```
discontinued = '''SELECT ProductName, SupplierId, CategoryId, UnitPrice FROM Product
                    WHERE Discontinued = 1
                    ORDER BY UnitPrice DESC'''
pd.read_sql_query(discontinued, engine)
```

And away we go from there.  It’s very easy to disappear down a rabbit hole throughout the Exploratory Data Analysis (EDA) phase.  Although I did discover some noteworthy things just from the above query (eg., 4 of the 8 discontinued products are in the “Meat/Poultry” category),  I ultimately bypassed the discontinued product inquisition in favor of some more impactful topics of exploration.



## Hypothesis Testing:

I chose to use parametric tests since the sample sizes were large enough to suspend the normality assumption.  Hence, I chose tests depending on the number of samples I was comparing:

* 1 sample vs. population mean: **1-sample t-test**
* 2 samples: **2-sample t-test**
* More than 2 samples: **ANOVA** testing with post-hoc **Tukey HSD** testing if necessary


### Q1: Do discounts have a statistically significant effect on the number of products customers order? If so, at what level(s) of discount?

* Business value:  Determining whether discounts are a significant driver of sales will help the business decide whether they should continue offering them.

* Visualizing the distribution:

![](https://i.imgur.com/Gt2dQZu.png)

* Methodology: Two-sample, one-tailed t-test <br>
     **H<sub>0</sub>**: There is no difference in the quantity of products that customers order when given a discount <br>
     **H<sub>A</sub>**: The quantity of products that customers order INCREASES when given a discount <br>
     **𝛼: 0.05**
		 
* Findings:  One tailed p-value: 5.720462261607983e-11 <br>
     Our p-value is microscopic and far below the alpha level of 0.05, so we *reject the null hypothesis and conclude that there is a statistically significant increase in quantity of products ordered when customers are given a discount.*


So, we move on to the second part of the question: **at what level(s) of discount?**

* Visualizing the distribution:

![](https://i.imgur.com/e8G7Eql.png)

* Methodology: ANOVA testing (since we have > 2 samples to compare) <br>
 **H<sub>0</sub>**: The average quantity ordered is the same across all levels of discount <br>
**H<sub>A</sub>**: The average quantity ordered differs depending on discount level <br>
**𝛼 : 0.05**

* Findings:  Our p-value is quite high here compared to our alpha level, 0.61 > 0.05. Based on the ANOVA results, we *fail to reject the null hypothesis and conclude that there is not a statistically significant difference in the quantity ordered based on the discount level offered.*

* **Business insight: We should be offering discounts, but there's no need to offer HIGHER discounts; we'll see comparable results at the 5% level vs. the 25% level, so we may as well retain as much profit as possible and discount at the lowest viable level, 5%.**


### Q2: Do British Isles "sales reps" generate more average revenue per order than their counterparts in North America?

* Business value: Determine whether our sales reps are in the right locations; potentially drive more revenue by relocating employees closer to where the majority of our customer base resides (Western Europe).

* Visualizing the distribution:

![](https://i.imgur.com/2dpFGQC.png)

* Methodology: Two-sample, one-tailed t-test: <br>
 **H<sub>0</sub>**: The average revenue generated is the same regardless of sales representative office region <br>
**H<sub>A</sub>**: The average revenue generated is higher for orders placed in the British Isles sales region than the North American sales region.  <br>
**𝛼 : 0.05**

* Findings:  Our p-value at 0.45 is well above 0.05, so we *fail to reject the null hypothesis and conclude that there is no significant difference in revenue generated based on a sales representative's office location.*


* **Business insight: Based on the findings, there is no justification to consider relocation of employees at this time.**  

* Future work: Conduct further testing to approach the broader idea (moving the base of operations to Europe) from other angles.

### Q3:  Does our average profit margin per order placed vary significantly depending on which region supplies the products?

* Business value: Potentially higher profits!

* Visualizing the distribution:

![](https://i.imgur.com/yd6cyGc.png)

* Methodology:  Initially ANOVA & Tukey HSD, but decided to switch to a series of one-sample t-tests, comparing each of 9 regions to the population mean  (iteratively using a for loop). <br>
**H<sub>0</sub>**: There is no difference in the average profit margin per order placed, regardless of the region from which the product was supplied. <br>
**H<sub>A</sub>**: One or more supply regions generate an average profit margin per order placed below the mean average profit margin per order. <br>
**𝛼: 0.05**

* Findings: 
The p-values were lower than 0.05 for the following supplier regions: British Isles, Eastern Asia, Northern Europe, & Scandinavia

* **Business insight: This feels like a breakthrough discovery.  A major shake-up could be in store for our supply chain. We should determine whether we can replace those same products using suppliers from our more profitable regions. If not, perhaps we should discontinue some of them and focus on selling higher quantities of the products that come with higher profit margins.**

* Future work: Further testing could also incorporate the freight costs, as those may further weigh down profitability (depending on who the responsible party is for the freight - customer or Northwind, or some combination of both depending on Service Level Agreements). 

### Q4: When a customer places an order, are the higher-priced products ordered in higher quantities than the lower-priced products?

* Business Value: Tighten our current product focus

* Visualizing the distribution:

![](https://i.imgur.com/5Q1K5qv.png)

* Methodology: Two-sample, one-tailed t-test <br>
**H<sub>0</sub>**: Unit price does not have an effect on quantity ordered (on a per order basis) <br>
**H<sub>A</sub>**: Higher unit prices result in higher quantities ordered (on a per order basis) <br>
**𝛼 : 0.05** 

* Findings: One tailed p-value: 0.027064361171080898 <br>
With this p-value, we can *reject the null hypothesis and conclude that there IS a statistically significant difference between "expensive" vs. "cheap" products in terms of quantity ordered (on a per order basis).*

* **Business insight:  Another very intriguing result.  My advice to Northwind is to focus exclusively on the higher-priced products. Combined with the earlier advice to move away from some of our less profitable suppliers, we may be able to pare down the product line significantly with this approach.**

* Future work: We should also conduct some further testing to determine if the median price point (used to subset our data here) is actually the ideal threshold.

## Final thoughts:

As it stands, Northwind Trader's value proposition is offering products in the marketplace that aren't easily obtained otherwise. Consumer demand is not driven by price for these items; they'll pay whatever it costs. As Northwind considers expanding into new product offerings, this insight should be a key consideration.  Based on my findings, I'd also strongly advocate for a potential rebrand. This is outside of my purview as a data scientist, but I think changing the name to something that will attract a "big-spender" demographic that is more likely to seek out expensive and obscure food items might be a worthwhile discussion. Do we want to be a company that sells quirky regional food products? Or one that offers "luxurious" products fit for royalty but accessible to anyone willing to pay for high quality goods? A few suggestions:
* Northwind Fine Foods Inc.
* Northwind Gourmet Retailers
* Northwind Epicurean Traders
* Northwind Delicacies & Co. <br>

In my humble opinion, any of these would be significant improvement on the vanilla "Northwind Traders." It would also give our company a more distinct identity and direction, which would undoubtedly help our sales and marketing teams pitch the company.

*So there you have it.  Give me an afternoon to write SQL and dig through your company's database, and I'll come out on the other side telling you to change the company name, drop half the supply chain, discontinue a bunch of your products, and think about relocating your base of operations halfway around the world!*





