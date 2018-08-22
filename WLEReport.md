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

The goal of this project was to accurately predict lack of form in a number of ways while preforming an exercise. 

The best preformin model was a random forest which preformed with an accuracy of 99%.

10-fold cross validation was used to create the model. 

Correct predictions in final testing data was 20 out of 20, 100%.

Original a cobination of models was planned to be used but given the effectiveness of the single model this was not required. 

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

I believe this assignment is seriously flawed in that rather than look at 
placing a set of data for an exercise into a class we are looking at placing 
single points in time. 

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
## [1] "2018-08-22 10:48:34 BST"
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

Look at which fields of the testing data are all NA. If this is the case we 
can't use these fields and they will be removed.


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
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
set.seed(2051)
inTrain <- createDataPartition(y=training$classe, p=0.7, list = FALSE)
training.nv <- training[inTrain,]
validation <- training[-inTrain,]
training <- training.nv
rm(training.nv)
```

Reviewing a variety of models using 10 fold cross validation. Only models with an accuracy above 70% will be used in the combined model. 


```
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1258
##      2        1.5235             nan     0.1000    0.0884
##      3        1.4655             nan     0.1000    0.0680
##      4        1.4204             nan     0.1000    0.0555
##      5        1.3843             nan     0.1000    0.0520
##      6        1.3514             nan     0.1000    0.0380
##      7        1.3268             nan     0.1000    0.0396
##      8        1.3014             nan     0.1000    0.0376
##      9        1.2783             nan     0.1000    0.0306
##     10        1.2584             nan     0.1000    0.0288
##     20        1.1042             nan     0.1000    0.0172
##     40        0.9331             nan     0.1000    0.0078
##     60        0.8248             nan     0.1000    0.0077
##     80        0.7452             nan     0.1000    0.0061
##    100        0.6825             nan     0.1000    0.0031
##    120        0.6324             nan     0.1000    0.0031
##    140        0.5880             nan     0.1000    0.0028
##    150        0.5660             nan     0.1000    0.0019
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1828
##      2        1.4893             nan     0.1000    0.1295
##      3        1.4068             nan     0.1000    0.1034
##      4        1.3407             nan     0.1000    0.0810
##      5        1.2883             nan     0.1000    0.0666
##      6        1.2445             nan     0.1000    0.0712
##      7        1.2002             nan     0.1000    0.0554
##      8        1.1647             nan     0.1000    0.0603
##      9        1.1270             nan     0.1000    0.0399
##     10        1.1009             nan     0.1000    0.0428
##     20        0.8982             nan     0.1000    0.0215
##     40        0.6859             nan     0.1000    0.0107
##     60        0.5575             nan     0.1000    0.0069
##     80        0.4712             nan     0.1000    0.0043
##    100        0.4073             nan     0.1000    0.0056
##    120        0.3543             nan     0.1000    0.0038
##    140        0.3122             nan     0.1000    0.0016
##    150        0.2955             nan     0.1000    0.0018
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2321
##      2        1.4606             nan     0.1000    0.1616
##      3        1.3587             nan     0.1000    0.1276
##      4        1.2796             nan     0.1000    0.1119
##      5        1.2094             nan     0.1000    0.0870
##      6        1.1542             nan     0.1000    0.0664
##      7        1.1113             nan     0.1000    0.0663
##      8        1.0694             nan     0.1000    0.0694
##      9        1.0267             nan     0.1000    0.0544
##     10        0.9922             nan     0.1000    0.0515
##     20        0.7612             nan     0.1000    0.0253
##     40        0.5360             nan     0.1000    0.0098
##     60        0.4084             nan     0.1000    0.0083
##     80        0.3281             nan     0.1000    0.0040
##    100        0.2687             nan     0.1000    0.0028
##    120        0.2249             nan     0.1000    0.0027
##    140        0.1916             nan     0.1000    0.0018
##    150        0.1772             nan     0.1000    0.0008
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1273
##      2        1.5229             nan     0.1000    0.0860
##      3        1.4645             nan     0.1000    0.0694
##      4        1.4199             nan     0.1000    0.0527
##      5        1.3837             nan     0.1000    0.0474
##      6        1.3518             nan     0.1000    0.0417
##      7        1.3254             nan     0.1000    0.0406
##      8        1.2985             nan     0.1000    0.0341
##      9        1.2759             nan     0.1000    0.0327
##     10        1.2552             nan     0.1000    0.0305
##     20        1.1022             nan     0.1000    0.0182
##     40        0.9311             nan     0.1000    0.0096
##     60        0.8234             nan     0.1000    0.0066
##     80        0.7449             nan     0.1000    0.0037
##    100        0.6828             nan     0.1000    0.0031
##    120        0.6336             nan     0.1000    0.0042
##    140        0.5896             nan     0.1000    0.0027
##    150        0.5695             nan     0.1000    0.0027
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1877
##      2        1.4879             nan     0.1000    0.1283
##      3        1.4054             nan     0.1000    0.1044
##      4        1.3383             nan     0.1000    0.0843
##      5        1.2839             nan     0.1000    0.0749
##      6        1.2362             nan     0.1000    0.0674
##      7        1.1938             nan     0.1000    0.0495
##      8        1.1623             nan     0.1000    0.0571
##      9        1.1256             nan     0.1000    0.0489
##     10        1.0954             nan     0.1000    0.0407
##     20        0.8891             nan     0.1000    0.0186
##     40        0.6822             nan     0.1000    0.0112
##     60        0.5552             nan     0.1000    0.0071
##     80        0.4705             nan     0.1000    0.0056
##    100        0.4038             nan     0.1000    0.0023
##    120        0.3524             nan     0.1000    0.0032
##    140        0.3099             nan     0.1000    0.0016
##    150        0.2913             nan     0.1000    0.0030
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2380
##      2        1.4600             nan     0.1000    0.1648
##      3        1.3553             nan     0.1000    0.1256
##      4        1.2756             nan     0.1000    0.1044
##      5        1.2103             nan     0.1000    0.0903
##      6        1.1540             nan     0.1000    0.0783
##      7        1.1044             nan     0.1000    0.0661
##      8        1.0618             nan     0.1000    0.0599
##      9        1.0228             nan     0.1000    0.0574
##     10        0.9870             nan     0.1000    0.0451
##     20        0.7604             nan     0.1000    0.0199
##     40        0.5375             nan     0.1000    0.0122
##     60        0.4121             nan     0.1000    0.0102
##     80        0.3288             nan     0.1000    0.0049
##    100        0.2701             nan     0.1000    0.0038
##    120        0.2279             nan     0.1000    0.0025
##    140        0.1939             nan     0.1000    0.0018
##    150        0.1793             nan     0.1000    0.0023
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1262
##      2        1.5240             nan     0.1000    0.0883
##      3        1.4653             nan     0.1000    0.0659
##      4        1.4214             nan     0.1000    0.0556
##      5        1.3849             nan     0.1000    0.0496
##      6        1.3522             nan     0.1000    0.0385
##      7        1.3261             nan     0.1000    0.0408
##      8        1.2999             nan     0.1000    0.0305
##      9        1.2794             nan     0.1000    0.0337
##     10        1.2584             nan     0.1000    0.0368
##     20        1.1044             nan     0.1000    0.0173
##     40        0.9327             nan     0.1000    0.0077
##     60        0.8283             nan     0.1000    0.0076
##     80        0.7520             nan     0.1000    0.0043
##    100        0.6868             nan     0.1000    0.0044
##    120        0.6348             nan     0.1000    0.0025
##    140        0.5913             nan     0.1000    0.0018
##    150        0.5725             nan     0.1000    0.0022
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1864
##      2        1.4887             nan     0.1000    0.1270
##      3        1.4048             nan     0.1000    0.1056
##      4        1.3382             nan     0.1000    0.0818
##      5        1.2857             nan     0.1000    0.0674
##      6        1.2429             nan     0.1000    0.0641
##      7        1.2018             nan     0.1000    0.0622
##      8        1.1634             nan     0.1000    0.0498
##      9        1.1317             nan     0.1000    0.0512
##     10        1.1003             nan     0.1000    0.0384
##     20        0.8977             nan     0.1000    0.0243
##     40        0.6826             nan     0.1000    0.0138
##     60        0.5577             nan     0.1000    0.0049
##     80        0.4716             nan     0.1000    0.0045
##    100        0.4036             nan     0.1000    0.0051
##    120        0.3532             nan     0.1000    0.0042
##    140        0.3113             nan     0.1000    0.0022
##    150        0.2937             nan     0.1000    0.0024
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2326
##      2        1.4631             nan     0.1000    0.1614
##      3        1.3605             nan     0.1000    0.1238
##      4        1.2831             nan     0.1000    0.1034
##      5        1.2179             nan     0.1000    0.0926
##      6        1.1610             nan     0.1000    0.0753
##      7        1.1120             nan     0.1000    0.0764
##      8        1.0640             nan     0.1000    0.0662
##      9        1.0219             nan     0.1000    0.0604
##     10        0.9845             nan     0.1000    0.0448
##     20        0.7593             nan     0.1000    0.0248
##     40        0.5377             nan     0.1000    0.0176
##     60        0.4099             nan     0.1000    0.0071
##     80        0.3272             nan     0.1000    0.0052
##    100        0.2687             nan     0.1000    0.0038
##    120        0.2254             nan     0.1000    0.0035
##    140        0.1912             nan     0.1000    0.0016
##    150        0.1785             nan     0.1000    0.0011
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1287
##      2        1.5236             nan     0.1000    0.0877
##      3        1.4659             nan     0.1000    0.0662
##      4        1.4218             nan     0.1000    0.0564
##      5        1.3859             nan     0.1000    0.0430
##      6        1.3565             nan     0.1000    0.0453
##      7        1.3274             nan     0.1000    0.0427
##      8        1.3019             nan     0.1000    0.0369
##      9        1.2794             nan     0.1000    0.0318
##     10        1.2598             nan     0.1000    0.0303
##     20        1.1058             nan     0.1000    0.0169
##     40        0.9345             nan     0.1000    0.0098
##     60        0.8308             nan     0.1000    0.0080
##     80        0.7479             nan     0.1000    0.0038
##    100        0.6870             nan     0.1000    0.0039
##    120        0.6356             nan     0.1000    0.0026
##    140        0.5914             nan     0.1000    0.0024
##    150        0.5724             nan     0.1000    0.0031
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1836
##      2        1.4898             nan     0.1000    0.1258
##      3        1.4084             nan     0.1000    0.1091
##      4        1.3385             nan     0.1000    0.0798
##      5        1.2862             nan     0.1000    0.0712
##      6        1.2410             nan     0.1000    0.0707
##      7        1.1967             nan     0.1000    0.0628
##      8        1.1576             nan     0.1000    0.0487
##      9        1.1254             nan     0.1000    0.0520
##     10        1.0925             nan     0.1000    0.0399
##     20        0.8887             nan     0.1000    0.0211
##     40        0.6853             nan     0.1000    0.0127
##     60        0.5566             nan     0.1000    0.0065
##     80        0.4709             nan     0.1000    0.0057
##    100        0.4059             nan     0.1000    0.0033
##    120        0.3538             nan     0.1000    0.0025
##    140        0.3112             nan     0.1000    0.0029
##    150        0.2938             nan     0.1000    0.0019
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2306
##      2        1.4612             nan     0.1000    0.1657
##      3        1.3588             nan     0.1000    0.1262
##      4        1.2800             nan     0.1000    0.0984
##      5        1.2177             nan     0.1000    0.0847
##      6        1.1641             nan     0.1000    0.0833
##      7        1.1103             nan     0.1000    0.0651
##      8        1.0687             nan     0.1000    0.0663
##      9        1.0274             nan     0.1000    0.0577
##     10        0.9907             nan     0.1000    0.0540
##     20        0.7588             nan     0.1000    0.0243
##     40        0.5352             nan     0.1000    0.0113
##     60        0.4086             nan     0.1000    0.0123
##     80        0.3221             nan     0.1000    0.0043
##    100        0.2657             nan     0.1000    0.0026
##    120        0.2217             nan     0.1000    0.0022
##    140        0.1893             nan     0.1000    0.0015
##    150        0.1752             nan     0.1000    0.0016
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1289
##      2        1.5240             nan     0.1000    0.0848
##      3        1.4661             nan     0.1000    0.0699
##      4        1.4196             nan     0.1000    0.0545
##      5        1.3838             nan     0.1000    0.0439
##      6        1.3546             nan     0.1000    0.0451
##      7        1.3255             nan     0.1000    0.0420
##      8        1.2991             nan     0.1000    0.0325
##      9        1.2777             nan     0.1000    0.0285
##     10        1.2584             nan     0.1000    0.0314
##     20        1.1042             nan     0.1000    0.0153
##     40        0.9340             nan     0.1000    0.0095
##     60        0.8260             nan     0.1000    0.0054
##     80        0.7471             nan     0.1000    0.0046
##    100        0.6825             nan     0.1000    0.0041
##    120        0.6315             nan     0.1000    0.0027
##    140        0.5896             nan     0.1000    0.0030
##    150        0.5710             nan     0.1000    0.0028
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1905
##      2        1.4860             nan     0.1000    0.1270
##      3        1.4038             nan     0.1000    0.1018
##      4        1.3377             nan     0.1000    0.0833
##      5        1.2847             nan     0.1000    0.0701
##      6        1.2398             nan     0.1000    0.0675
##      7        1.1969             nan     0.1000    0.0551
##      8        1.1615             nan     0.1000    0.0625
##      9        1.1231             nan     0.1000    0.0401
##     10        1.0971             nan     0.1000    0.0455
##     20        0.8915             nan     0.1000    0.0240
##     40        0.6785             nan     0.1000    0.0076
##     60        0.5566             nan     0.1000    0.0054
##     80        0.4675             nan     0.1000    0.0051
##    100        0.4026             nan     0.1000    0.0027
##    120        0.3514             nan     0.1000    0.0027
##    140        0.3099             nan     0.1000    0.0027
##    150        0.2922             nan     0.1000    0.0019
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2306
##      2        1.4619             nan     0.1000    0.1659
##      3        1.3572             nan     0.1000    0.1216
##      4        1.2791             nan     0.1000    0.1004
##      5        1.2149             nan     0.1000    0.0863
##      6        1.1605             nan     0.1000    0.0683
##      7        1.1165             nan     0.1000    0.0763
##      8        1.0695             nan     0.1000    0.0709
##      9        1.0255             nan     0.1000    0.0574
##     10        0.9894             nan     0.1000    0.0502
##     20        0.7568             nan     0.1000    0.0273
##     40        0.5369             nan     0.1000    0.0146
##     60        0.4068             nan     0.1000    0.0079
##     80        0.3239             nan     0.1000    0.0044
##    100        0.2650             nan     0.1000    0.0023
##    120        0.2230             nan     0.1000    0.0031
##    140        0.1894             nan     0.1000    0.0015
##    150        0.1764             nan     0.1000    0.0013
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1276
##      2        1.5226             nan     0.1000    0.0891
##      3        1.4627             nan     0.1000    0.0680
##      4        1.4179             nan     0.1000    0.0541
##      5        1.3818             nan     0.1000    0.0472
##      6        1.3504             nan     0.1000    0.0429
##      7        1.3232             nan     0.1000    0.0416
##      8        1.2968             nan     0.1000    0.0341
##      9        1.2746             nan     0.1000    0.0329
##     10        1.2539             nan     0.1000    0.0296
##     20        1.1002             nan     0.1000    0.0195
##     40        0.9307             nan     0.1000    0.0109
##     60        0.8236             nan     0.1000    0.0057
##     80        0.7476             nan     0.1000    0.0040
##    100        0.6860             nan     0.1000    0.0046
##    120        0.6323             nan     0.1000    0.0028
##    140        0.5894             nan     0.1000    0.0024
##    150        0.5698             nan     0.1000    0.0021
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1896
##      2        1.4877             nan     0.1000    0.1268
##      3        1.4056             nan     0.1000    0.1058
##      4        1.3378             nan     0.1000    0.0841
##      5        1.2848             nan     0.1000    0.0742
##      6        1.2378             nan     0.1000    0.0729
##      7        1.1926             nan     0.1000    0.0574
##      8        1.1558             nan     0.1000    0.0470
##      9        1.1257             nan     0.1000    0.0464
##     10        1.0961             nan     0.1000    0.0368
##     20        0.8949             nan     0.1000    0.0250
##     40        0.6863             nan     0.1000    0.0108
##     60        0.5596             nan     0.1000    0.0072
##     80        0.4714             nan     0.1000    0.0047
##    100        0.4027             nan     0.1000    0.0031
##    120        0.3510             nan     0.1000    0.0025
##    140        0.3076             nan     0.1000    0.0024
##    150        0.2909             nan     0.1000    0.0025
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2342
##      2        1.4599             nan     0.1000    0.1636
##      3        1.3562             nan     0.1000    0.1230
##      4        1.2781             nan     0.1000    0.1046
##      5        1.2121             nan     0.1000    0.0878
##      6        1.1569             nan     0.1000    0.0686
##      7        1.1128             nan     0.1000    0.0776
##      8        1.0642             nan     0.1000    0.0667
##      9        1.0232             nan     0.1000    0.0556
##     10        0.9882             nan     0.1000    0.0486
##     20        0.7586             nan     0.1000    0.0272
##     40        0.5327             nan     0.1000    0.0142
##     60        0.4052             nan     0.1000    0.0064
##     80        0.3243             nan     0.1000    0.0051
##    100        0.2699             nan     0.1000    0.0037
##    120        0.2258             nan     0.1000    0.0026
##    140        0.1928             nan     0.1000    0.0025
##    150        0.1787             nan     0.1000    0.0030
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1260
##      2        1.5239             nan     0.1000    0.0896
##      3        1.4652             nan     0.1000    0.0653
##      4        1.4216             nan     0.1000    0.0549
##      5        1.3858             nan     0.1000    0.0525
##      6        1.3528             nan     0.1000    0.0397
##      7        1.3271             nan     0.1000    0.0431
##      8        1.3004             nan     0.1000    0.0345
##      9        1.2782             nan     0.1000    0.0276
##     10        1.2599             nan     0.1000    0.0317
##     20        1.1039             nan     0.1000    0.0174
##     40        0.9310             nan     0.1000    0.0081
##     60        0.8248             nan     0.1000    0.0060
##     80        0.7458             nan     0.1000    0.0046
##    100        0.6815             nan     0.1000    0.0035
##    120        0.6298             nan     0.1000    0.0025
##    140        0.5859             nan     0.1000    0.0022
##    150        0.5672             nan     0.1000    0.0026
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1840
##      2        1.4870             nan     0.1000    0.1297
##      3        1.4034             nan     0.1000    0.1026
##      4        1.3370             nan     0.1000    0.0852
##      5        1.2833             nan     0.1000    0.0689
##      6        1.2379             nan     0.1000    0.0643
##      7        1.1966             nan     0.1000    0.0650
##      8        1.1569             nan     0.1000    0.0554
##      9        1.1221             nan     0.1000    0.0413
##     10        1.0952             nan     0.1000    0.0474
##     20        0.8876             nan     0.1000    0.0219
##     40        0.6780             nan     0.1000    0.0078
##     60        0.5550             nan     0.1000    0.0067
##     80        0.4660             nan     0.1000    0.0062
##    100        0.3954             nan     0.1000    0.0043
##    120        0.3451             nan     0.1000    0.0025
##    140        0.3062             nan     0.1000    0.0025
##    150        0.2870             nan     0.1000    0.0012
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2371
##      2        1.4602             nan     0.1000    0.1661
##      3        1.3540             nan     0.1000    0.1320
##      4        1.2719             nan     0.1000    0.1086
##      5        1.2040             nan     0.1000    0.0833
##      6        1.1505             nan     0.1000    0.0777
##      7        1.1007             nan     0.1000    0.0659
##      8        1.0592             nan     0.1000    0.0626
##      9        1.0193             nan     0.1000    0.0607
##     10        0.9819             nan     0.1000    0.0459
##     20        0.7546             nan     0.1000    0.0195
##     40        0.5309             nan     0.1000    0.0121
##     60        0.4047             nan     0.1000    0.0070
##     80        0.3241             nan     0.1000    0.0034
##    100        0.2666             nan     0.1000    0.0028
##    120        0.2227             nan     0.1000    0.0025
##    140        0.1911             nan     0.1000    0.0012
##    150        0.1778             nan     0.1000    0.0014
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1223
##      2        1.5240             nan     0.1000    0.0879
##      3        1.4654             nan     0.1000    0.0691
##      4        1.4201             nan     0.1000    0.0558
##      5        1.3829             nan     0.1000    0.0483
##      6        1.3519             nan     0.1000    0.0440
##      7        1.3241             nan     0.1000    0.0418
##      8        1.2978             nan     0.1000    0.0344
##      9        1.2756             nan     0.1000    0.0355
##     10        1.2538             nan     0.1000    0.0278
##     20        1.1018             nan     0.1000    0.0195
##     40        0.9308             nan     0.1000    0.0096
##     60        0.8244             nan     0.1000    0.0065
##     80        0.7450             nan     0.1000    0.0051
##    100        0.6830             nan     0.1000    0.0029
##    120        0.6330             nan     0.1000    0.0030
##    140        0.5881             nan     0.1000    0.0031
##    150        0.5695             nan     0.1000    0.0024
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1892
##      2        1.4879             nan     0.1000    0.1261
##      3        1.4061             nan     0.1000    0.1089
##      4        1.3381             nan     0.1000    0.0846
##      5        1.2842             nan     0.1000    0.0751
##      6        1.2358             nan     0.1000    0.0630
##      7        1.1958             nan     0.1000    0.0667
##      8        1.1548             nan     0.1000    0.0471
##      9        1.1243             nan     0.1000    0.0505
##     10        1.0921             nan     0.1000    0.0421
##     20        0.8895             nan     0.1000    0.0202
##     40        0.6800             nan     0.1000    0.0115
##     60        0.5539             nan     0.1000    0.0053
##     80        0.4691             nan     0.1000    0.0056
##    100        0.4027             nan     0.1000    0.0048
##    120        0.3520             nan     0.1000    0.0041
##    140        0.3101             nan     0.1000    0.0022
##    150        0.2942             nan     0.1000    0.0023
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2396
##      2        1.4582             nan     0.1000    0.1628
##      3        1.3544             nan     0.1000    0.1224
##      4        1.2749             nan     0.1000    0.1016
##      5        1.2111             nan     0.1000    0.0950
##      6        1.1515             nan     0.1000    0.0778
##      7        1.1018             nan     0.1000    0.0634
##      8        1.0618             nan     0.1000    0.0597
##      9        1.0241             nan     0.1000    0.0497
##     10        0.9921             nan     0.1000    0.0530
##     20        0.7583             nan     0.1000    0.0210
##     40        0.5346             nan     0.1000    0.0089
##     60        0.4089             nan     0.1000    0.0075
##     80        0.3273             nan     0.1000    0.0050
##    100        0.2703             nan     0.1000    0.0024
##    120        0.2245             nan     0.1000    0.0021
##    140        0.1917             nan     0.1000    0.0014
##    150        0.1789             nan     0.1000    0.0009
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1285
##      2        1.5230             nan     0.1000    0.0856
##      3        1.4649             nan     0.1000    0.0677
##      4        1.4202             nan     0.1000    0.0554
##      5        1.3840             nan     0.1000    0.0421
##      6        1.3550             nan     0.1000    0.0476
##      7        1.3251             nan     0.1000    0.0401
##      8        1.2990             nan     0.1000    0.0362
##      9        1.2763             nan     0.1000    0.0304
##     10        1.2565             nan     0.1000    0.0297
##     20        1.1003             nan     0.1000    0.0186
##     40        0.9314             nan     0.1000    0.0087
##     60        0.8238             nan     0.1000    0.0046
##     80        0.7442             nan     0.1000    0.0050
##    100        0.6808             nan     0.1000    0.0037
##    120        0.6289             nan     0.1000    0.0026
##    140        0.5858             nan     0.1000    0.0034
##    150        0.5657             nan     0.1000    0.0029
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1856
##      2        1.4886             nan     0.1000    0.1291
##      3        1.4044             nan     0.1000    0.1047
##      4        1.3365             nan     0.1000    0.0795
##      5        1.2843             nan     0.1000    0.0738
##      6        1.2379             nan     0.1000    0.0672
##      7        1.1951             nan     0.1000    0.0623
##      8        1.1555             nan     0.1000    0.0484
##      9        1.1239             nan     0.1000    0.0506
##     10        1.0919             nan     0.1000    0.0393
##     20        0.8936             nan     0.1000    0.0226
##     40        0.6769             nan     0.1000    0.0101
##     60        0.5536             nan     0.1000    0.0062
##     80        0.4661             nan     0.1000    0.0056
##    100        0.4002             nan     0.1000    0.0035
##    120        0.3495             nan     0.1000    0.0016
##    140        0.3092             nan     0.1000    0.0019
##    150        0.2911             nan     0.1000    0.0018
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2350
##      2        1.4607             nan     0.1000    0.1654
##      3        1.3572             nan     0.1000    0.1260
##      4        1.2771             nan     0.1000    0.1020
##      5        1.2126             nan     0.1000    0.0974
##      6        1.1524             nan     0.1000    0.0834
##      7        1.0996             nan     0.1000    0.0601
##      8        1.0606             nan     0.1000    0.0569
##      9        1.0233             nan     0.1000    0.0624
##     10        0.9851             nan     0.1000    0.0436
##     20        0.7575             nan     0.1000    0.0247
##     40        0.5347             nan     0.1000    0.0132
##     60        0.4078             nan     0.1000    0.0071
##     80        0.3284             nan     0.1000    0.0042
##    100        0.2704             nan     0.1000    0.0055
##    120        0.2263             nan     0.1000    0.0025
##    140        0.1925             nan     0.1000    0.0014
##    150        0.1780             nan     0.1000    0.0015
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1251
##      2        1.5231             nan     0.1000    0.0897
##      3        1.4634             nan     0.1000    0.0692
##      4        1.4181             nan     0.1000    0.0538
##      5        1.3823             nan     0.1000    0.0481
##      6        1.3502             nan     0.1000    0.0443
##      7        1.3216             nan     0.1000    0.0373
##      8        1.2975             nan     0.1000    0.0332
##      9        1.2763             nan     0.1000    0.0306
##     10        1.2566             nan     0.1000    0.0322
##     20        1.0999             nan     0.1000    0.0166
##     40        0.9295             nan     0.1000    0.0092
##     60        0.8233             nan     0.1000    0.0059
##     80        0.7446             nan     0.1000    0.0055
##    100        0.6851             nan     0.1000    0.0035
##    120        0.6324             nan     0.1000    0.0034
##    140        0.5903             nan     0.1000    0.0034
##    150        0.5691             nan     0.1000    0.0024
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.1869
##      2        1.4862             nan     0.1000    0.1286
##      3        1.4024             nan     0.1000    0.1034
##      4        1.3362             nan     0.1000    0.0873
##      5        1.2802             nan     0.1000    0.0706
##      6        1.2355             nan     0.1000    0.0728
##      7        1.1908             nan     0.1000    0.0562
##      8        1.1554             nan     0.1000    0.0548
##      9        1.1213             nan     0.1000    0.0420
##     10        1.0937             nan     0.1000    0.0461
##     20        0.8980             nan     0.1000    0.0234
##     40        0.6859             nan     0.1000    0.0111
##     60        0.5613             nan     0.1000    0.0083
##     80        0.4706             nan     0.1000    0.0054
##    100        0.4063             nan     0.1000    0.0071
##    120        0.3550             nan     0.1000    0.0035
##    140        0.3146             nan     0.1000    0.0027
##    150        0.2955             nan     0.1000    0.0023
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2336
##      2        1.4601             nan     0.1000    0.1591
##      3        1.3574             nan     0.1000    0.1238
##      4        1.2792             nan     0.1000    0.1144
##      5        1.2084             nan     0.1000    0.0898
##      6        1.1510             nan     0.1000    0.0725
##      7        1.1044             nan     0.1000    0.0680
##      8        1.0605             nan     0.1000    0.0543
##      9        1.0249             nan     0.1000    0.0653
##     10        0.9839             nan     0.1000    0.0519
##     20        0.7601             nan     0.1000    0.0225
##     40        0.5392             nan     0.1000    0.0159
##     60        0.4086             nan     0.1000    0.0107
##     80        0.3250             nan     0.1000    0.0045
##    100        0.2696             nan     0.1000    0.0022
##    120        0.2250             nan     0.1000    0.0020
##    140        0.1929             nan     0.1000    0.0018
##    150        0.1781             nan     0.1000    0.0012
## 
## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
##      1        1.6094             nan     0.1000    0.2361
##      2        1.4592             nan     0.1000    0.1563
##      3        1.3590             nan     0.1000    0.1249
##      4        1.2795             nan     0.1000    0.0985
##      5        1.2166             nan     0.1000    0.1043
##      6        1.1525             nan     0.1000    0.0771
##      7        1.1034             nan     0.1000    0.0668
##      8        1.0621             nan     0.1000    0.0541
##      9        1.0275             nan     0.1000    0.0589
##     10        0.9913             nan     0.1000    0.0457
##     20        0.7570             nan     0.1000    0.0218
##     40        0.5298             nan     0.1000    0.0088
##     60        0.4104             nan     0.1000    0.0073
##     80        0.3292             nan     0.1000    0.0035
##    100        0.2696             nan     0.1000    0.0049
##    120        0.2275             nan     0.1000    0.0032
##    140        0.1946             nan     0.1000    0.0021
##    150        0.1799             nan     0.1000    0.0013
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
