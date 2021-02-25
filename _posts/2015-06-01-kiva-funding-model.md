---
toc: true
layout: post
description: Random Forest Model to Predict Kiva Loan Funding.
categories: [markdown]
title: How to get loans funded on kiva
---

This post was originally written in 2015 and was reformatted for my fastpages blog in 2021. 

![Title Slide](https://github.com/mattlichti/kiva-fundraising-success/blob/master/img/title.jpg?raw=true) 

## Motivation: 

 This project is an analysis of what features make microfinance loans on [kiva.org](kiva.org) more likely to get funded. There are over 2 million small time lenders on kiva who have funded over 880,000 microfinance loans totaling $712 million since 2005. The loans are originally made by 296 different microfinance organizations in 85 countries who use kiva to raise money to backfill some of their loans. Lenders can view the loans on the website and lend as little as $25 to any of the borrowers they chose.

Since January 2012, kiva has usually had a 30 day expiration policy. If a loan is not fully funded in that time, the lenders are refunded and the microfinance organization does not receive any money. This is similar to the funding model used on kickstarter where projects do not receive any money unless they are fully funded. Because of the risk of loan expiration, it is important for the microfinance organizations to understand the characteristics that make loans more likely to get funded. The analysis could also help kiva.org increase the total number of loans getting funded, which would help more struggling entrepreneurs around the globe get the capital they need to build their businesses

## Data:

Kiva makes their loan data available through their api. They also periodically make downloadable snapshots of the data. I most recently used the May 18 2015 json snapshot for this analysis. The loan data is in a 1 GB zip file that is 5 GB when unzipped. 

### Pipeline:

[data_pipeline.py](https://github.com/mattlichti/kiva-fundraising-success/blob/master/data_pipeline.py) is used for processing the raw kiva loan data. It extracts the relevant data, performs the feature engineering, and then stores it all in a postgres SQL database. To run the pipeline, unzip the loans folder containing around 1800 json files that each have the data from 500 loans. Then setup a postgres database and run pipeline.py in the terminal the loans folder location and sql information as command line arguments. The most important part of the process is feature engineering. The features include anything that a potential borrower sees when viewing a loan on kiva org that they can use to decide whether or not to loan to a particular individual. 

## Feature Engineering:

The most important part of the process is feature engineering. The features include anything that a potential borrower sees when viewing a loan on kiva org that they can use to decide whether or not to loan to a particular individual. A typical view of a loan from the kiva website is pictured below with some of the important features highlighted.

![Feature Engineering](https://github.com/mattlichti/kiva-fundraising-success/blob/master/img/feature_engineering.jpg?raw=true)

### Continuous features
* loan amount
* repayment term (anywhere from 4 months to several years)
* group size (loans can be for 1 person or a group of people)

### Categorical features
* gender
* country - currently 84 countries
* sector - 15 categories like transportation or agriculture
* activity - 150 categories like "rickshaw" or "cattle"
* repayment interval - loans payments can be monthly, lump sum at the end, or irregular
* themes - searchable attributes like "green", "fair trade", "conflict zones", etc.
* currency loss - whether the lender is liable for losses due to currency fluctuations
* anonymous - whether the borrower has their name and photo on the website or chooses to remain anonymous.

### Transforming the text into features
I used the one sentence description of how the loan will be used to engineer features out of the most commonly used terms. The length of this text as well as the length of the larger description text were also useful features.

### Competing loans
In addition to the features that impact demand for particular loans, the supply of loans on the site can effect the chances of each loan getting funded. I used SQL to calculate the number of other loans on kiva at the time each loan was posted by comparing the timestamp of when each loan was posted to the timestamps for when other loans were posted and funded or expired.

![Competing Loans](https://github.com/mattlichti/kiva-fundraising-success/blob/master/plots/competing_loans.png?raw=true)

When there are few loans on the site, almost all of the loans get funded. When there is more competition, lenders have more options each loan has a higher chance of expiring. In the future, it might be useful to look at the total value of the loans currently fundraising and how far along they are in their fundraising, not just the number of loans. It might also be useful to look at attributes of those loans. For example, if there are a lot of loans currently fundraising from the same country or economic sector as the loan being posted, it may lower the chance of that loan getting funded.

## Modeling:
[build_model.py](https://github.com/mattlichti/kiva-fundraising-success/blob/master/build_model.py) is used to train the model, predict which loans have a higher risk of expiring, and determine which features are most important in predicting loan success. The model converts the categorical features into boolean dummy variables, and tokenizes, lemmatizes, and performs TF-IDF on the text. I used a random forest model. The classes were unbalanced with a much higher number of funded loans than expired loans, so I heavily weighted the expired loans in order to increase recall of expired loans at the expense of precision. The model can output a confusion matrix and a list of feature importance which could be used as recommendations on how to improve their odds of getting their loans funded.  

### Running the model
[run_model.py](https://github.com/mattlichti/kiva-fundraising-success/blob/master/run_model.py) is used to load the relevant data from the postgres sql database and run the model on that data. It can be run from the command line like data_pipeline.py.

## Plotting
[plots.py](https://github.com/mattlichti/kiva-fundraising-success/blob/master/plots.py) is used to make plots of the feature importance of the most important features in the random forest model, as well as plot the expiration rate based on a variety of features and the expiration rate over time.

