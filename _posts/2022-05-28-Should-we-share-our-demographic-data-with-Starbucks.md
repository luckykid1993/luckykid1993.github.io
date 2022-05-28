## Should we share our demographic data with Starbucks?
![starbucks-2226717_640](https://user-images.githubusercontent.com/104986203/170838838-95bdf8ff-6a33-4bdd-ba2d-6d700c608245.jpg)


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
* became_member_on: (date) format YYYYMMDD
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
![1](https://user-images.githubusercontent.com/104986203/170838877-26bc4cfd-164e-45ce-9f13-42c8cdcff45b.PNG)
![3](https://user-images.githubusercontent.com/104986203/170838896-dc77baca-35a7-41f8-a511-87d81ae03eae.PNG)
There are no missing values in the data.

![7](https://user-images.githubusercontent.com/104986203/170838941-79ff01f5-1cfc-48f8-9d80-5e9054358d5c.PNG)

We have 3 types of offer: bogo, discount and informational.

![6](https://user-images.githubusercontent.com/104986203/170838952-aef244af-d2ed-485f-8dec-a66e564af599.PNG)

Most of offer open about 6-7 days.

### Profile
![8](https://user-images.githubusercontent.com/104986203/170839030-7a2218c9-85e2-4638-bef0-a4e39224b23b.PNG)
![9](https://user-images.githubusercontent.com/104986203/170839037-d2a7ec79-d3e0-406d-9cf0-b7f746b4759c.PNG)
![10](https://user-images.githubusercontent.com/104986203/170839040-8fa811c1-9799-4174-b7cb-60f0bec0be01.PNG)

There are 17000 users, but there are 2175 records that missing value(age, gender, income).

![13](https://user-images.githubusercontent.com/104986203/170839049-bf344a0e-46cf-4af0-bcd8-761ed92ecc74.PNG)

Member has increased over the years.

![14](https://user-images.githubusercontent.com/104986203/170839054-ba96119a-b1b7-4128-8fdc-9b3aca3914c8.PNG)

Most of uses are Male an Female.

![16](https://user-images.githubusercontent.com/104986203/170839064-e701a998-b18a-4cb3-8748-59d387855e4a.PNG)

The distribution of age is nearly normal distribution for all gender (Male, Female and Other).

![15](https://user-images.githubusercontent.com/104986203/170839069-25784e4e-c375-4a5d-80e2-205d22547a4b.PNG)

The income seems to be similar for all genders.


### transcript

![17](https://user-images.githubusercontent.com/104986203/170839095-1d7e222a-fe80-46e7-ad82-d8438fb7d998.PNG)
![19](https://user-images.githubusercontent.com/104986203/170839110-e6a09570-d4e9-4b37-9a76-110af8287df2.PNG)

The data were collected for about 30 days (700 hours).

![21](https://user-images.githubusercontent.com/104986203/170839127-47fda1b4-ef48-4b96-a887-b72cf420c77c.PNG)

The number of unique users is same as the profile (17000).

![23](https://user-images.githubusercontent.com/104986203/170839132-04d050de-3b8e-4ae6-9898-ec61d906e9a0.PNG)

Each user received about 4-5 offers, maximum is 6.

![25](https://user-images.githubusercontent.com/104986203/170839148-2976ee50-9bff-4bca-8087-045616d2b8d4.PNG)

We have 4 types of event: offer received, offer viewed, offer completed, transaction


# Methodology

## Data Preprocessing
### Portfolio
![1](https://user-images.githubusercontent.com/104986203/170839194-a7208004-14fb-4591-8301-4916d2001d81.PNG)

Data in channels column is inside [] and separated by comma.
![30](https://user-images.githubusercontent.com/104986203/170839252-efcb6278-3484-441e-b464-ae95a1c31ee4.PNG)

Split the channels column into 4 columns: web, email, mobile, social. 
Then drop the channels column and change id column's name to 'offer_id'.

![31](https://user-images.githubusercontent.com/104986203/170839210-30a71f1e-c05e-43b9-847c-28cabf8d7262.PNG)


### Profile
There are 2175 records that missing value (age, gender, income). 
I want to use these columns later, it's about only 12% of 17000 records, so I think it's ok to drop these records.
I changed the id column's name to 'person' to make it easier to remember.

![32](https://user-images.githubusercontent.com/104986203/170839236-d36f3962-77aa-4436-b187-eec8f6961cec.PNG)


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

![34](https://user-images.githubusercontent.com/104986203/170839305-245a725f-7bb3-4e37-958b-15f74821501e.PNG)
![35](https://user-images.githubusercontent.com/104986203/170839312-a729777e-06ac-4316-aae3-dcf4c6d6bca0.PNG)


### Feature engineering
After cleaned data, i created 2 features:

1. A columns that indicate how many days that a user is a member (member_days)
2. A columns that tell us how many times a user received an offer (offer_rev_cnt)

I built 2 dummy columns (gender_M, gender_F) from the gender column because the user's gender is text data.

![36](https://user-images.githubusercontent.com/104986203/170839344-8d8cacc3-0df2-4bb0-be32-2438046b3ebc.PNG)
![37](https://user-images.githubusercontent.com/104986203/170839357-231af99c-74f6-4986-8238-6c46d7bd230e.PNG)


## Implementation
Because we have 3 types of offer, I'm going to create 3 different models (1 model for each type of offer) to predict that user make a purchase or not. 
It is in fact a supervised learning model of binary classification.

I will try to use 3 algorithms to find which is best ft for this case:

1. DecisionTreeClassifier
2. RandomForestClassifier
3. AdaBoostClassifier

![42](https://user-images.githubusercontent.com/104986203/170839594-23b2633f-e1e5-458c-8709-1242efe90986.PNG)

![41](https://user-images.githubusercontent.com/104986203/170839596-e394b17f-6e8b-43d9-9b51-32d66926c431.PNG)

After trying all 3 algorithms, AdaBoostClassifier algorithm gives the best results.:

1. Bogo model has a 70% accuracy rate and a 69% F1 score.
2. Discount model has a 73% accuracy rate and a 72% F1 score.
3. Infomational has a 66% accuracy rate and a 65% F1 score.
![49](https://user-images.githubusercontent.com/104986203/170839389-07619529-a18c-42ff-b883-2c2e83c9f204.PNG)

![50](https://user-images.githubusercontent.com/104986203/170839391-fa24a22b-b164-4209-b246-b4f69d316345.PNG)

![51](https://user-images.githubusercontent.com/104986203/170839397-53c07ff4-b08e-4c20-aedc-3ecb375db287.PNG)

I will use these models to find the factors have major impact on the use of a offer.

![52](https://user-images.githubusercontent.com/104986203/170839402-71b0380c-f821-4b98-ad8f-4a1c7468215b.PNG)

![53](https://user-images.githubusercontent.com/104986203/170839405-ead14def-8ab7-4267-8817-100e1f1d9d99.PNG)
![54](https://user-images.githubusercontent.com/104986203/170839409-3e996bbb-8ec1-43d2-a779-c820a9d75d4b.PNG)
![55](https://user-images.githubusercontent.com/104986203/170839412-62d5e463-b180-4704-aec0-7ee342772af7.PNG)


## Refinement
I attempted to tune the AdaBoostClassifier model using GridSearchCV.

![56](https://user-images.githubusercontent.com/104986203/170839421-903be8a6-4aca-4954-9e9c-feb22397f4fc.PNG)

But the accuracy rate didn't change:

1. Bogo model has a 70% accuracy rate and a 69% F1 score.
2. Discount model has a 73% accuracy rate and a 72% F1 score.
3. Infomational has a 66% accuracy rate and a 65% F1 score.

![57](https://user-images.githubusercontent.com/104986203/170839423-1b19acb6-8524-421f-a674-fc6d92f46e5c.PNG)

![58](https://user-images.githubusercontent.com/104986203/170839427-a5475458-9669-4f34-b450-72c0be297481.PNG)

![59](https://user-images.githubusercontent.com/104986203/170839428-11fd85b6-7270-472a-af41-d7ba07a708f7.PNG)

# Results
## Model Evaluation and Validation
Based on the project's findings, I believe we can apply a machine learning model to predict whether or not a customer will accept the offer. 
The best model also tells us the most important factors that influence the likelihood of customers responding to the offer, such as membership term, age, and income.
The length of membership is the most crucial aspect in determining whether or not the offer will be accepted. 
That is, the longer a client has been a Starbucks member, the more likely he or she is to respond to an offer. 
The second and third most important criteria that influence a customer's likelihood of responding are age and income.

The best model is AdaBoostClassifier:

1. Bogo model has a 70% accuracy rate and a 69% F1 score.
![57](https://user-images.githubusercontent.com/104986203/170839423-1b19acb6-8524-421f-a674-fc6d92f46e5c.PNG)

2. Discount model has a 73% accuracy rate and a 72% F1 score.

![58](https://user-images.githubusercontent.com/104986203/170839427-a5475458-9669-4f34-b450-72c0be297481.PNG)

3. Infomational has a 66% accuracy rate and a 65% F1 score.
![59](https://user-images.githubusercontent.com/104986203/170839428-11fd85b6-7270-472a-af41-d7ba07a708f7.PNG)

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
[Starbucks Capstone](https://github.com/luckykid1993/Udacity/blob/main/StarbucksCapstone/Starbucks_Capstone_notebook.ipynb)

### Reference
The image just after the title is published here: 
[pixabay.com](https://pixabay.com/photos/starbucks-christmas-lawn-coffee-2226717/)
