# Risk_Detection_with_Log-in_Behaviors
## Introduction

#### Motivation and objective
Jing Dong(JD), one of the largest B2C online retailer in China held challenges for all data scientists
around the world, which is inspiring because one of the challenge, risk detection
with users' log-in behaviors is about solving problems in a real-world business scenario with real data.

Big data can predict transaction risk using all types of user behaviors and improve safety measures
for online shopping. To this end user log-in behavior is an important source of information, such as
via PC or mobile end; via account name, phone number or mailbox; via password or scanning QR
code. The system can evaluate transaction risk based on log-in activities, automatically interfering
with fraudulent transactions.

The application scenario of this task is to predict risk level for transactions through recent log-in
activities, interfering with fraudulent transactions. We are expected to build classication algorithms
and construct commercially feasible models capable of pattern detection.

#### Data
Training data set(around 580000 records) includes user log-in information and the
transaction risk log from 2015-01-01 to 2015-06-30. The log-in information includes time spent for
log-in, device, log-in ip, ip city, log-in result(success or failure), log-in time, log-in type, log-in id,
whether it is scanned log-in and if security control is used. And the risk log includes transaction time,
log-in id and risk flag(target variable, 0 or 1)

Since the user does not necessarily make a transaction for every log-in, the number of trade should
be less than the number of log-ins. We need to decide how to associate log-in behavior with transaction
occurrence.

The challenge requires us to predict the risk for each transaction based on the log-in
information from 2015-07-01 to 2015-07-30. The log-in information for the testing set(around 130000
records) remains the same for the testing set(except that the time period is different). We need to
predict the risk of all transactions in the testing data set using our model and upload the result to JD
website. The score on test set will be returned to us after the submission

## Methodology
#### Feature engineering
We spend most of our time engineering features, which is the most important and difficult part of
the challenge. The first thought that comes to us is to match the transaction data to the most recent
log-in data based on user's id. Then we can transform the log-in information into features for the
trade data. However we cannot find the matches for some of the trade data(around 8% of all records).
Besides, this method does not make use of all information in the log-in data set.
Therefore, we approach the issue from a different angle. Looking at the trade data in training data
set marked with risk, we find some possible features, which can be categorizes into two groups: those
can be extracted only using trade data and, those need to be extracted by combining both data sets.
For example, in terms of trade data, it is observed that most risky trades happens several times in
a very short time. So we add features like the number of trade times of the day/in an hour/in ten
minutes of each trade. For combined data, for instance, we find that, if an IP address has been used
by different users in an hour, the probability of this IP being risky becomes larger. Extracting features
from combined data can be tricky. For the IP example we just referred, first, the number of users
who have traded using the same IP in an hour should be counted, by using the log-in data. Then the
problem occurs: how to connect this to our trade data? In this case, since we have matched the trade
data to the most recent log-in information, we merge the IP statistics to the trade data, joined by
log-in time and IP to make sure the uniqueness.
Here are some features that we use:
1. total trade times in ten minutes/an hour before each trade
2. total log-in times in ten minutes/an hour before each trade
3. time difference between each trade and its most recent log-in record
4. the total number of devices an user has ever used
5. the percentage of the most recent log-in city among all cities an user has ever logged in
6. the total number of users of the device used in the most recent log-in record

#### Modeling
The modelling process is primarily based on Random Forest(RF).

#### Evaluation metric
We are required to predict the risk of all transactions in the testing data set, 0 for no risk, 1 for
risk, to calculate 'precision' and 'recall' based on risk = 1, and then get the total f-beta score (where,
beta = 0.1):
![equation](https://latex.codecogs.com/gif.latex?F_%7B%5Cbeta%7D%20%3D%20%281&plus;%5Cbeta%5E2%29%5Cfrac%7Bprecision*recall%7D%7B%5Cbeta%5E2*precision%20&plus;recall%7D)

As is known, precision denotes the percentage of true positives among all the predicted positives.
The recall denotes the percentage of true positives among all positives. F-beta is defined based on
both metric in which beta = 0.1 means that precision is far more important than recall rate. In other
words, the key point to improve the score in detecting risk is to avoid predicting negatives as positives.

##Results
Without down-sampling, we tune the parameters to find the best RF model. When there are 90
trees and the maximum features is set to 2, the model performs the best with a f-beta score of 0.86981.
After down-sampling our negative samples, the RF model gives us a better result with a f-beta score
of 0.91094.

Here is the confusion matrix for both precision and recall for the second model. Obviously the
precision rate for risk of class 1 we get is extremely high while the recall rate is small(0.1). (Figure:
1) This result is exactly what we pursued in the first place. As a matter of fact, in a business scenario
like this, precision is far more important than recall. Classifying a customer's transaction as risk by
mistake will undermine user experience significantly.

![confusion matrix](https://github.com/sunan0519/Risk_Detection_with_Log-in_Behaviors/edit/master/confusion_matrix.jpg)


We also draw a bar plot of feature importance. From the figure we can observe that the
feature - device used in the most recent log-in behaviour among all devices a user has ever used - is
the most important one. The features - whether the most recent log-in is scanned log-in and, whether
it is successful - impact little on our model.

![feature importance](https://github.com/sunan0519/Risk_Detection_with_Log-in_Behaviors/edit/master/feature_importance.jpg)
