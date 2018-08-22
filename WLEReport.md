---
title: "Practical Machine Learning Assignment"
author: "Benjamin Estrade"
date: "21 August 2018"
output: 
  html_document: 
    keep_md: yes
    theme: spacelab
    toc: yes
---



#Executive Summary

The goal of this project was to accurately predict a lack of form in a number of ways while preforming an exercise. 

The best preforming model was a random forest which preformed with an accuracy of 99%.

10-fold cross validation was used to create the model. 

Correct predictions in final testing data was 20 out of 20, 100%.

Originally, a cobination of models was planned to be used but given the effectiveness of the single model this was not required. 

#Introduction

This project will be analyzing data from accelerometers on the waist, bicept, 
wrist and dumbell. 

This data was collected during a set of 10 Unilateral Dumbbell Biceps Curl by 6 
different participants.

The participants were asked to make different types of errors while completing 
the set which are reflected by the different classes. Each class is as follows:

* Class A - exactly according to the specification 

* Class B - throwing the elbows to the front 

* Class C - lifting the dumbbell only halfway 

* Class D - lowering the dumbbell only halfway 

* Class E - throwing the hips to the front 

All measurements y is up and down, x is right and left and z is forward and 
backward

#Aim of report

To try and build a predictive model based on this data that will accuractely 
predict the way in which the exercise is conducted. 


#Assumptions and Concerns

Assumptions:

* The date time data shouldn't be used in the prediction model as well as the 
variable x and num window

* The user name can be used as part of the prediction model

#Analysis

##Retriving and importing the data


```r
#recording time the data was downloaded for reproducablity
Sys.time()
```

```
## [1] "2018-08-22 11:52:19 BST"
```

```r
training.url <- 
        "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

testing.url <- 
        "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

if(!dir.exists("datafiles")){
        dir.create("datafiles")
}

if(!dir.exists("datafiles/training.csv")){
        download.file(training.url, "datafiles/training.csv")
}

if(!dir.exists("datafiles/testing.csv")){
        download.file(testing.url, "datafiles/testing.csv")
}

training <- read.csv("datafiles/training.csv")
testing <- read.csv("datafiles/testing.csv")
```

##Exploritory Analysis

Looking at the data.

More information about the dataset can be found here:

http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har


```r
dim(training)
```

```
## [1] 19622   160
```

```r
str(training)
```

```
## 'data.frame':	19622 obs. of  160 variables:
##  $ X                       : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ user_name               : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ raw_timestamp_part_1    : int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
##  $ raw_timestamp_part_2    : int  788290 808298 820366 120339 196328 304277 368296 440390 484323 484434 ...
##  $ cvtd_timestamp          : Factor w/ 20 levels "02/12/2011 13:32",..: 9 9 9 9 9 9 9 9 9 9 ...
##  $ new_window              : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
##  $ num_window              : int  11 11 11 12 12 12 12 12 12 12 ...
##  $ roll_belt               : num  1.41 1.41 1.42 1.48 1.48 1.45 1.42 1.42 1.43 1.45 ...
##  $ pitch_belt              : num  8.07 8.07 8.07 8.05 8.07 8.06 8.09 8.13 8.16 8.17 ...
##  $ yaw_belt                : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...
##  $ total_accel_belt        : int  3 3 3 3 3 3 3 3 3 3 ...
##  $ kurtosis_roll_belt      : Factor w/ 397 levels "","-0.016850",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_picth_belt     : Factor w/ 317 levels "","-0.021887",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_yaw_belt       : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_roll_belt      : Factor w/ 395 levels "","-0.003095",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_roll_belt.1    : Factor w/ 338 levels "","-0.005928",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_yaw_belt       : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
##  $ max_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_picth_belt          : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_yaw_belt            : Factor w/ 68 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ min_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_pitch_belt          : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_yaw_belt            : Factor w/ 68 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ amplitude_roll_belt     : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ amplitude_pitch_belt    : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ amplitude_yaw_belt      : Factor w/ 4 levels "","#DIV/0!","0.00",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ var_total_accel_belt    : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ avg_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stddev_roll_belt        : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ var_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ avg_pitch_belt          : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stddev_pitch_belt       : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ var_pitch_belt          : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ avg_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stddev_yaw_belt         : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ var_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ gyros_belt_x            : num  0 0.02 0 0.02 0.02 0.02 0.02 0.02 0.02 0.03 ...
##  $ gyros_belt_y            : num  0 0 0 0 0.02 0 0 0 0 0 ...
##  $ gyros_belt_z            : num  -0.02 -0.02 -0.02 -0.03 -0.02 -0.02 -0.02 -0.02 -0.02 0 ...
##  $ accel_belt_x            : int  -21 -22 -20 -22 -21 -21 -22 -22 -20 -21 ...
##  $ accel_belt_y            : int  4 4 5 3 2 4 3 4 2 4 ...
##  $ accel_belt_z            : int  22 22 23 21 24 21 21 21 24 22 ...
##  $ magnet_belt_x           : int  -3 -7 -2 -6 -6 0 -4 -2 1 -3 ...
##  $ magnet_belt_y           : int  599 608 600 604 600 603 599 603 602 609 ...
##  $ magnet_belt_z           : int  -313 -311 -305 -310 -302 -312 -311 -313 -312 -308 ...
##  $ roll_arm                : num  -128 -128 -128 -128 -128 -128 -128 -128 -128 -128 ...
##  $ pitch_arm               : num  22.5 22.5 22.5 22.1 22.1 22 21.9 21.8 21.7 21.6 ...
##  $ yaw_arm                 : num  -161 -161 -161 -161 -161 -161 -161 -161 -161 -161 ...
##  $ total_accel_arm         : int  34 34 34 34 34 34 34 34 34 34 ...
##  $ var_accel_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ avg_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stddev_roll_arm         : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ var_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ avg_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stddev_pitch_arm        : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ var_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ avg_yaw_arm             : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ stddev_yaw_arm          : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ var_yaw_arm             : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ gyros_arm_x             : num  0 0.02 0.02 0.02 0 0.02 0 0.02 0.02 0.02 ...
##  $ gyros_arm_y             : num  0 -0.02 -0.02 -0.03 -0.03 -0.03 -0.03 -0.02 -0.03 -0.03 ...
##  $ gyros_arm_z             : num  -0.02 -0.02 -0.02 0.02 0 0 0 0 -0.02 -0.02 ...
##  $ accel_arm_x             : int  -288 -290 -289 -289 -289 -289 -289 -289 -288 -288 ...
##  $ accel_arm_y             : int  109 110 110 111 111 111 111 111 109 110 ...
##  $ accel_arm_z             : int  -123 -125 -126 -123 -123 -122 -125 -124 -122 -124 ...
##  $ magnet_arm_x            : int  -368 -369 -368 -372 -374 -369 -373 -372 -369 -376 ...
##  $ magnet_arm_y            : int  337 337 344 344 337 342 336 338 341 334 ...
##  $ magnet_arm_z            : int  516 513 513 512 506 513 509 510 518 516 ...
##  $ kurtosis_roll_arm       : Factor w/ 330 levels "","-0.02438",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_picth_arm      : Factor w/ 328 levels "","-0.00484",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_yaw_arm        : Factor w/ 395 levels "","-0.01548",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_roll_arm       : Factor w/ 331 levels "","-0.00051",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_pitch_arm      : Factor w/ 328 levels "","-0.00184",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_yaw_arm        : Factor w/ 395 levels "","-0.00311",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ max_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_picth_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_yaw_arm             : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_yaw_arm             : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ amplitude_roll_arm      : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ amplitude_pitch_arm     : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ amplitude_yaw_arm       : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ roll_dumbbell           : num  13.1 13.1 12.9 13.4 13.4 ...
##  $ pitch_dumbbell          : num  -70.5 -70.6 -70.3 -70.4 -70.4 ...
##  $ yaw_dumbbell            : num  -84.9 -84.7 -85.1 -84.9 -84.9 ...
##  $ kurtosis_roll_dumbbell  : Factor w/ 398 levels "","-0.0035","-0.0073",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_picth_dumbbell : Factor w/ 401 levels "","-0.0163","-0.0233",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_yaw_dumbbell   : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_roll_dumbbell  : Factor w/ 401 levels "","-0.0082","-0.0096",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_pitch_dumbbell : Factor w/ 402 levels "","-0.0053","-0.0084",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_yaw_dumbbell   : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
##  $ max_roll_dumbbell       : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_picth_dumbbell      : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_yaw_dumbbell        : Factor w/ 73 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ min_roll_dumbbell       : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_pitch_dumbbell      : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ min_yaw_dumbbell        : Factor w/ 73 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ amplitude_roll_dumbbell : num  NA NA NA NA NA NA NA NA NA NA ...
##   [list output truncated]
```

The summary data seems to be available when new window = yes. Otherwise it is 
empty.

Looking at which fields of the testing data are all NA. If this is the case we 
can't use these fields for prediction and they will be removed.


```r
testing.not.empty <- apply(testing, 2, function(x) length(x) != sum(is.na(x)))
testing.not.empty[1:20]
```

```
##                    X            user_name raw_timestamp_part_1 
##                 TRUE                 TRUE                 TRUE 
## raw_timestamp_part_2       cvtd_timestamp           new_window 
##                 TRUE                 TRUE                 TRUE 
##           num_window            roll_belt           pitch_belt 
##                 TRUE                 TRUE                 TRUE 
##             yaw_belt     total_accel_belt   kurtosis_roll_belt 
##                 TRUE                 TRUE                FALSE 
##  kurtosis_picth_belt    kurtosis_yaw_belt   skewness_roll_belt 
##                FALSE                FALSE                FALSE 
## skewness_roll_belt.1    skewness_yaw_belt        max_roll_belt 
##                FALSE                FALSE                FALSE 
##       max_picth_belt         max_yaw_belt 
##                FALSE                FALSE
```

Removing the empty fields and data which can't be used for predicitions from the 
data sets


```r
testing <- testing[,testing.not.empty]
training <- training[,testing.not.empty]
vars.not.for.prediction = c(1,3,4,5,6,7)
testing <- testing[,-vars.not.for.prediction]
training <- training[,-vars.not.for.prediction]
```

Splitting testing set into a testing and validation set


```r
library(caret)

set.seed(2051)
inTrain <- createDataPartition(y=training$classe, p=0.7, list = FALSE)
training.nv <- training[inTrain,]
validation <- training[-inTrain,]
training <- training.nv
rm(training.nv)
```

Reviewing a variety of models using 10 fold cross validation. Only models with an accuracy above 70% will be used in the combined model. 


```r
#setting the validation method I will be using during these trails
train.control <- trainControl(method = "cv", number = 10)
#setting the seed
set.seed(1564)
#lda model
lda.model <- train(classe ~ ., data = training, method = "lda", trControl = train.control)
#lda with pca preprocessing
pca.model <- train(classe ~ ., data = training, method = "lda", preProcess = "pca", trControl = train.control)
#qda model
qda.model <- train(classe ~ ., data = training, method = "qda", trControl = train.control)
#boosting using trees
gbm.model <- train(classe ~ ., data = training, method = "gbm", trControl = train.control)
#random forest model
rf.model <- train(classe ~ ., data = training, method = "rf", trControl = train.control)
```

Looking at the effectiveness of the models on the validation data set


```r
lda.pred <- predict(lda.model, validation)
pca.pred <- predict(pca.model, validation)
qda.pred <- predict(qda.model, validation)
gbm.pred <- predict(gbm.model, validation)
rf.pred <- predict(rf.model, validation)
lda.ac <- confusionMatrix(validation$classe, lda.pred)$overall[1]
pca.ac <- confusionMatrix(validation$classe, pca.pred)$overall[1]
qda.ac <- confusionMatrix(validation$classe, qda.pred)$overall[1]
gbm.ac <- confusionMatrix(validation$classe, gbm.pred)$overall[1]
rf.ac <- confusionMatrix(validation$classe, rf.pred)$overall[1]
data.frame(lda = lda.ac, pca = pca.ac, qda = qda.ac, gbm = gbm.ac, rf = rf.ac)
```

```
##                lda       pca       qda       gbm       rf
## Accuracy 0.7308411 0.5223449 0.9150382 0.9656754 0.995582
```

#Conclusion

The random forest model is the best preforming on real data and at over 99% no combinition is required. Accurcacy of model on the validation set is as follows.


```r
confusionMatrix(validation$classe, rf.pred)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1673    1    0    0    0
##          B    8 1130    1    0    0
##          C    0    2 1020    4    0
##          D    0    0    7  956    1
##          E    0    0    1    1 1080
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9956          
##                  95% CI : (0.9935, 0.9971)
##     No Information Rate : 0.2856          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9944          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9952   0.9974   0.9913   0.9948   0.9991
## Specificity            0.9998   0.9981   0.9988   0.9984   0.9996
## Pos Pred Value         0.9994   0.9921   0.9942   0.9917   0.9982
## Neg Pred Value         0.9981   0.9994   0.9981   0.9990   0.9998
## Prevalence             0.2856   0.1925   0.1749   0.1633   0.1837
## Detection Rate         0.2843   0.1920   0.1733   0.1624   0.1835
## Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
## Balanced Accuracy      0.9975   0.9977   0.9950   0.9966   0.9993
```

The predictions for the 20 testing points are as follows:


```r
predict(rf.model, testing)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
