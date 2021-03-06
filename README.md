---
title: "Practical Machine Learning Project"
author: "Kelvin"
date: "20 April 2015"
output: html_document
---
## 1. Introduction  
Human Activity Recognition (HAR) devices emergence as a wear technology has resulted in development of applications focusing on the result of executing an activity. The data collected by the app can then be used for a variety of purpose e.g. correct posture to prevent injuries, optimize traininga and etc. 
  
This project focus on using data collected from HAR devices to predict if an activity (e.g. dumbbell lifting) is performed in an expected manner.     
    
For a more comprehensive report with figures in HTML format, please refer to this [link]()  
  
## 2. Test Setup   
Environment:
The four HAR devices were attached to the test subject to detect motions in the following place:  
- A gyro sensor on the Dumbbell  
- A glove sensor worn around the Wrist  
- An arm-band sensor worn around the Arm  
- A belt sensor worn around Waist  

In this test, six human test subjects are requested to do dumbbell lifting in five different ways as follow: 
A. Proper and correct execution of dumbbell lifting  
B. Throwing the elbows to the front  
C. Lifting the dumbbell only halfway  
D. Lowering the dumbbell only halfway  
E. Throwing the hips to the front  
  
  
To take the measurement, one set of 10 repetitions for each of the five different ways above were executed by the 6 test subjects. The data is then saved in a CSV file.  
  
## 3. Exploratory Data Analysis
The dataset has **19622** observations and **160** variables. 
  
```
pmltrain <- read.csv("pml-training.csv", header=T)
```
  
### 3.1 Cleaning and Filtering Data
In the CSV, there are some variables which have no value or NAs on most of the observations because these variables contain the max, var, stddev, min, etc. for a set of repetitions, let's filter off these variables. In addition, the first seven columns (id, names, timestamps, repetition window indicators) does not seems relevant so it will also be filtered.
  
```
x <- as.numeric()  
for(i in 1:ncol(pmltrain)) { if(pmltrain[1,i]=="" | is.na(pmltrain[1,i])) x <- c(x, i) }  
pmltrain <- pmltrain[-x]; pmltrain <- pmltrain[-c(1:7)]  
```
  
## 4. Data Slicing  
The dataset is split into 60% (training) 40% (testing) partitions. A seed is then set so that the same results can be reproduced.   
The prediction model is trained using the training partition to achieve a satisfactory result before applying to the testing partition. 
  
```
set.seed(5)  
inTrain <- createDataPartition(y=pmltrain$classe, p=0.6, list=F)  
training <- pmltrain[inTrain,]  
testing <- pmltrain[-inTrain,]  
```
  
## 5. Model Fitting  
The model picked for this test is Random Forest algorithm as this is the most accurate model among some of other model. To improve accuracy and peformance of the selected model, the data is first preprocessed within caret 'train' function using Principal Components Analysis (PCA). 
**10 fold cross validation** is performed within caret 'train' function to improve the accuracy.  
  
```
library(caret) 
set.seed(5)  
modelFit <- train(classe ~ ., data=training, preProcess=c("pca"), trControl=trainControl(method="cv"), method="rf")  
```
  
### 5.1 Out of Sample Error
Based on the results, Accurary was the criteria for selecting the optimal model. This model uses mtry of 2 and has an accuracy of **96.77%** sampling on the training data.  
  
The model is then applied to the testing data to predict the 'classe' results. The accurary for **Out of Sample Error** is **97.46%**. This is better than our training data and proves that there is no overfitting.  
  
```
## Using the model to predict on testing data (excluding the 'classe' column)  
ptest <- predict(modelFit, testing[,-53])  
confusionMatrix(ptest, testing$classe)  
```  
  
## 6. Final Prediction - Validatiing the dataset 
The prediction is now applied on the validation dataset which contains 20 test cases 
It is important to tidy and filtered the validation data set before applying to test cases for predicting the results.  
  
```
## Reading the  validation data set from file
validation <- read.csv("pml-testing.csv", header=T)  
  
## Perform data filtering and tidying as per training/testing data set
x <- as.numeric()  
for(i in 1:ncol(validation)) { if(validation[1,i]=="" | is.na(validation[1,i])) x <- c(x, i) }  
validation <- validation[-x]; validation <- validation[-c(1:7)]  
  
## Predict on validation data set excluding the last column which is problem_id
pred <- predict(modelFit, validation[,-53])  
```
