PracMachineLearning_WeightLiftingExerciseAnalysis
------------------------------------------------------------
PracMachineLearning - WeightLiftingExerciseAnalysis


Description
--------------------
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

Goal
----------------
Goal of the project is to fit a model with given training data set and predict `classe` variable of the testing data set.


Algorithms Testd For better accuracy
---------------------------------------
I have trained models using below algorithms and found `RandomForest` works best with applying tuning controls.
* logistic regression model - GBM
* lda - linear discriminant analysis 
* Naive Bayes
* Random Forest.
* Parallel Random Forest.
* Random Forest with tuning control.

Parallel processing to improve speed.
----------------------------
I have used doParallel library and made 4 clusters to do parallel processing. Later I registered these clusters to use them for fitting algorithms.

```
library(doParallel);
rCluster <- makePSOCKcluster(4);
registerDoParallel(rCluster);
```

Source, Cleaning and Processing Data
--------------------------
Imported pml-training.csv into workspace and assigned those values to trainRawData variable. In further steps, I cleaned data by removing columns with NA's from dataset. This is done to reduce the number of insignificant columns thereby improving the processing time and accuracy.

```
trainRawData <- read.csv("pml-training.csv",na.strings=c("NA",""));

NAs <- apply(trainRawData,2,function(x) {sum(is.na(x))});
validData <- trainRawData[,which(NAs == 0)];
```
After executing above code, we have received 60 variables are predictors.


Data Partitioning for Trainind and Testing
------------------------------
To make efficient model, we need to train our model with 70% of given test data. Once model is prepared, it is always good to test it with rest 30% data to cross validate the predicted values against already existing values.

```
library(caret);
trainIndex <- createDataPartition(y = validData$classe, p=0.7,list=FALSE);
trainData <- validData[trainIndex,];
```
I have used caret createDataPartition method in caret package to split data into two sections.
* Training data - 70% of pml-training.csv
* Testing data - 30% of pml-training.csv

CreatedDataPartition functions gives indices of split data. 

Further Cleaning
------------------------

According to the details mentioned in http://groupware.les.inf.puc-rio.br/har it is understood that timestamps, username and other details are not highly significant in predicting the Classe variable of the dataset. So, accordingly, I have removed the irrelevant variables using Grep function.

```
removeIndex <- grep("timestamp|X|user_name|new_window",names(trainData));
trainData <- trainData[,-removeIndex];
```

So, trainData would be our final dataset that we need to use for fitting a model which is accurate.
After complete cleansing, we got trainData which has below details.
*    13737 samples
*    53 predictors
*    5 classes: 'A', 'B', 'C', 'D', 'E'


Model Fitting
---------------------
I have used Random Forest as algorithm to get highest accuracy with available 53 predictor variables. To control the training process, trControl function is used with Cross Validation as method and parallel processing also set to true.


```
trControl = trainControl(method = "cv", number = 4, allowParallel =TRUE);
modFitRF <- train(trainData$classe ~.,data = trainData,method="rf",trControl=trControl);
```
I have passed all variables as predictor variable except classe to train function. Method is set to RandomForest and trainControl which we prepared in earlier step.

This model fitting took approximately 45 minutes on a Windows Machine with 64 Bit Chipset, i5, 4 Gigabytes of Ram.

Model details
-----------------
To look more into model, i excuted below command and found that this model is 99.6% accurate at mtry 27. Thus this would be most efficient model in predicitng our test values.

```
modFitRF;
```

```
Random Forest 

13737 samples
   53 predictors
    5 classes: 'A', 'B', 'C', 'D', 'E' 

No pre-processing
Resampling: Cross-Validated (4 fold) 

Summary of sample sizes: 10302, 10304, 10302, 10303 

Resampling results across tuning parameters:

  mtry  Accuracy  Kappa  Accuracy SD  Kappa SD
  2     0.994     0.992  0.00104      0.00131 
  27    0.996     0.995  0.0016       0.00203 
  53    0.995     0.994  0.00236      0.00299 

Accuracy was used to select the optimal model using  the
 largest value.
The final value used for the model was mtry = 27. 
```



Cross Validation
------------------------
As we have splot the data into two sets, we have trained our model with training data. We have out model ready and we have to used rest 30% of data to test the model.

Cleaning:
```
testData <- validData[-trainIndex,];
removeIndex <- grep("timestamp|X|user_name|new_window",names(testData));
testData <- testData[,-removeIndex];
```

Predicted Values:
```
predictedValues<-predict(modFitRF,testData);
```

Viewing Predicted Values:

```
View(predictedValues);
```


Comparing Predicted Values against Actual Classe values of testing data set:

```
testData$predictedValues<-predictedValues;
testData$Comparision<-testData$predictedValues==testData$classe;

length(testData$Comparision[testData$Comparision==FALSE]);
length(testData$Comparision[testData$Comparision==TRUE]);
```



Plotting the values:
```
qplot(testData$classe,testData$predictedValues, color=testData$Comparision);
```

```
qplot(length(testData$Classe), color=testData$Comparision);
```
