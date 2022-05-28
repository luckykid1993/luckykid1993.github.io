## Should we share our demographic data with Starbucks?



#  Project Definition
## Project Overview

This data set contains simulated data that mimics customer behavior on the Starbucks rewards mobile app. 
Once every few days, Starbucks sends out an offer to users of the mobile app. 
An offer can be merely an advertisement for a drink or an actual offer such as a discount or BOGO (buy one get one free). 
Some users might not receive any offer during certain weeks. 

Not all users receive the same offer, and that is the challenge to solve with this data set.

Our task is to combine transaction, demographic and offer data to determine which demographic groups respond best to which offer type. 

## Problem Statement
Sometimes we don't want to share personal information with others. 
But with starbucks, I think it helps if we share personal information with them. 
With this information, they can be used to more accurately send us an offer, which benefits both them and us.

In this project, i try to answer 2 questions below:
1. What factors have a major impact on the use of a offer?
2. Is it possible to create a model that predicts whether or not someone will accept an offer based on demographic data?

## Metrics
I decided to focus to the F1 score as a measure to evaluate the model. 
The reason is that I want to avoid predicting false negatives, such as: It is good if we send someone an offer, but we predict false.

#  Analysis
## Data Exploration

### Data Dictionary
The data is contained in three files:

* portfolio.json - containing offer ids and meta data about each offer
* profile.json - demographic data for each customer
* transcript.json - records for transactions, offers received, offers viewed, and offers completed

**profile.json**
Rewards program users (17000 users x 5 fields)

* gender: (categorical) M, F, O, or null
* age: (numeric) missing value encoded as 118
* id: (string/hash)
* * became_member_on: (date) format YYYYMMDD
* income: (numeric)

**portfolio.json**
Offers sent during 30-day test period (10 offers x 6 fields)

* reward: (numeric) money awarded for the amount spent
* channels: (list) web, email, mobile, social
* difficulty: (numeric) money required to be spent to receive reward
* duration: (numeric) time for offer to be open, in days
* offer_type: (string) bogo, discount, informational
* id: (string/hash)

**transcript.json**
Event log (306648 events x 4 fields)

* person: (string/hash)
* event: (string) offer received, offer viewed, transaction, offer completed
* value: (dictionary) different values depending on event type
* offer id: (string/hash) not associated with any "transaction"
* amount: (numeric) money spent in "transaction"
* reward: (numeric) money gained from "offer completed"
* time: (numeric) hours after start of test

There are three types of offers that can be sent: buy-one-get-one (BOGO), discount, and informational. 

* In a BOGO offer, a user needs to spend a certain amount to get a reward equal to that threshold amount. 
* In a discount, a user gains a reward equal to a fraction of the amount spent. 
* In an informational offer, there is no reward, but neither is there a requisite amount that the user is expected to spend. Offers can be delivered via multiple channels.


We must examine the datasets in order to comprehend the data set, which involves checking for missing values, displaying the data distribution, and so on.

### Portfolio

We have 3 types of offer: bogo, discount and informational.
Most of offer open about 6-7 days.
User need to spend 7-8 (assume it is dollar(USD)) to recevie an offer.
Most of reward is about 4 (assume it is dollar(USD)).

### Profile
There are 17000 users, but there are 2175 records that missing value(age, gender, income).

Member has increased over the years.

Most of uses are Male an Female.

The distribution of age is nearly normal distribution for all gender (Male, Female and Other).

The income seems to be similar for all genders.


### transcript
The data were collected for about 30 days.

The number of unique users is same as the profile (17000).

Each user received about 4-5 offers, maximum is 6.

We have 4 types of event: offer received, offer viewed, offer completed, transaction



# Methodology

## Data Preprocessing
### Portfolio
Data in channels column is inside [] and separated by comma.
Split the channels column into 4 columns: web, email, mobile, social. 
Then drop the channels column and change id column's name to 'offer_id'.

### Profile
There are 2175 records that missing value (age, gender, income). 
I want to use these columns later, it's about only 12% of 17000 records, so I think it's ok to drop these records.
I changed the id column's name to 'person' to make it easier to remember.


### Transcript
We have to extract offer id, amout, reward from value column.
After extract data, i droped unuse column.
Because the event 'transaction' does not have a corresponding offer id, I used the offer id from earlier rows to fill it in.

We need to know whether or not the offer is effective.
So, i created a column indicates a effective offer, then combine with portfolio, profile data.
Effective offer: the customer makes the transaction after viewing offer. 
In other words, the offer need to meet 3 requirements below:

1. User viewed offer.
2. The user performed a transaction after reviewing the offer.
3. The offer happened within the duration.

Each person and offer id pair can have a many rows of data, such as: offer received, offer_viewed, transaction, offer_completed.
I only keep one row for each person and offer id pair. When effective = 1, the customer completed the offer after viewing the offer.
With effective = 0, the customer received an offer, but there was no Effective offer.

### Feature engineering
After cleaned data, i created 2 features:

1. A columns that indicate how many days that a user is a member (member_days)
2. A columns that tell us how many times a user received an offer (offer_rev_cnt)

I built 2 dummy columns (gender_M, gender_F) from the gender column because the user's gender is text data.

## Implementation
Because we have 3 types of offer, I'm going to create 3 different models (1 model for each type of offer) to predict that user make a purchase or not. 
It is in fact a supervised learning model of binary classification.

I will try to use 3 algorithms to find which is best ft for this case:

1. DecisionTreeClassifier
2. RandomForestClassifier
3. AdaBoostClassifier

After trying all 3 algorithms, AdaBoostClassifier algorithm gives the best results.:

1. Bogo model has a 70% accuracy rate and a 69% F1 score.
2. Discount model has a 73% accuracy rate and a 72% F1 score.
3. Infomational has a 66% accuracy rate and a 65% F1 score.

I will use these models to find the factors have major impact on the use of a offer.

## Refinement
I attempted to tune the AdaBoostClassifier model using GridSearchCV.

But the accuracy rate didn't change:

1. Bogo model has a 70% accuracy rate and a 69% F1 score.
2. Discount model has a 73% accuracy rate and a 72% F1 score.
3. Infomational has a 66% accuracy rate and a 65% F1 score.
# Results
## Model Evaluation and Validation
Based on the project's findings, I believe we can apply a machine learning model to predict whether or not a customer will accept the offer. 
The best model also tells us the most important factors that influence the likelihood of customers responding to the offer, such as membership term, age, and income.
The length of membership is the most crucial aspect in determining whether or not the offer will be accepted. 
That is, the longer a client has been a Starbucks member, the more likely he or she is to respond to an offer. 
The second and third most important criteria that influence a customer's likelihood of responding are age and income.

The best model is AdaBoostClassifier:

1. Bogo model has a 70% accuracy rate and a 69% F1 score.
2. Discount model has a 73% accuracy rate and a 72% F1 score.
3. Infomational has a 66% accuracy rate and a 65% F1 score.

The accuracy is greater than 60% for all three models, in terms of business, I think that it is a good accuracy rate, and it is acceptable in this project.


# Conclusion
## Reflection
In this project, I discarded any records that lacked demographic data. We lost data from 12% of our users. 

The accuracy is not great.

I didn't look at some demographic groups to see if they would buy even if they didn't get an offer. From a commercial standpoint, we wouldn't send a buy 10 get 2 off offer to a consumer who is already planning to make a $10 purchase without an offer.
## Improvement
I believe that if i can find a way to keep such data from being lost, the accuracy will improve. For example, probably, we can fill null demographic data with median.

I could add more features, or I could test removing some features to see how they effect the model's performance.

In addition, I believe we can test alternative algothrim to see which will fit our data the best.

If I have enough time, I'll create a website that generates a prediction result after entering the customer's information and the offer's details.
### Github:
[Starbucks Capstone]()

### Reference
The image just after the title is published here: 
[pixabay.com](https://pixabay.com/photos/starbucks-christmas-lawn-coffee-2226717/)