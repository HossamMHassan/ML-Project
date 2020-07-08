# ML-Project
---
title: "Results and analysis"
author: "Hossam Hassan"
date: "7/7/2020"
output: html_document
---
## Overview

It is now possible to collect a large amount of personal activity data relatively cheaply using devices such as Jawbone Up, Nike FuelBand and Fitbit. These kinds of devices are part of the quantified self-movement – a group of enthusiasts who regularly take measurements about themselves to improve their health, find patterns in their behaviour, or are tech geeks. One thing people do regularly is measure how much of an operation they do but never measure how well they do it. The goal in this project will be to use data from accelerometers on 6 participants' collar, forearm, head, and dumbell. They were asked to do barbell lifts in 5 different ways, correctly and incorrectly.

The purpose of this project is to predict how they conducted the exercise. This is the `classe` variable in the training set.

## Data description

The outcome variable is `classe`, a factor variable with 5 levels. For this data set, participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in 5 different fashions:

- exactly according to the specification (Class A)
- throwing the elbows to the front (Class B)
- lifting the dumbbell only halfway (Class C)
- lowering the dumbbell only halfway (Class D)
- throwing the hips to the front (Class E)

## Initial configuration

The initial configuration consists of loading some required packages and initializing some variables.

```{r configuration, echo=TRUE, results='hide'}
#Data variables
training.file   <- './data/pml-training.csv'
test.cases.file <- './data/pml-testing.csv'
training.url    <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

test.cases.url  <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
#Directories
if (!file.exists("data")){
  dir.create("data")
}
if (!file.exists("data/submission")){
  dir.create("data/submission")
}
#R-Packages
IscaretInstalled <- require("caret")
if(!IscaretInstalled){
    install.packages("caret")
    library("caret")
    }
IsrandomForestInstalled <- require("randomForest")
if(!IsrandomForestInstalled){
    install.packages("randomForest")
    library("randomForest")
    }
IsRpartInstalled <- require("rpart")
if(!IsRpartInstalled){
    install.packages("rpart")
    library("rpart")
    }
IsRpartPlotInstalled <- require("rpart.plot")
if(!IsRpartPlotInstalled){
    install.packages("rpart.plot")
    library("rpart.plot")
    }
# Set seed for reproducability
set.seed(9999)
```

## Data processing
In this section the data is downloaded and processed. Some basic transformations and cleanup will be performed, so that `NA` values are omitted. Irrelevant columns such as `user_name`, `raw_timestamp_part_1`, `raw_timestamp_part_2`, `cvtd_timestamp`, `new_window`, and  `num_window` (columns 1 to 7) will be removed in the subset.

The `pml-training.csv` data is used to devise training and testing sets.
The `pml-test.csv` data is used to predict and answer the 20 questions based on the trained model.

```{r dataprocessing, echo=TRUE, results='hide'}
# Download data if not exist

if(!file.exists(training.file)){
  download.file(training.url, training.file)
}

if(!file.exists(test.cases.file)){
download.file(test.cases.url,test.cases.file )
}  
  
# Clean data
training   <-read.csv(training.file, na.strings=c("NA","#DIV/0!", ""))
testing <-read.csv(test.cases.file , na.strings=c("NA", "#DIV/0!", ""))
training<-training[,colSums(is.na(training)) == 0]
testing <-testing[,colSums(is.na(testing)) == 0]
# Subset data
training   <-training[,-c(1:7)]
testing <-testing[,-c(1:7)]
```

## Cross-validation
In this section cross-validation will be performed by splitting the training data in training (75%) and testing (25%) data.

```{r datasplitting, echo=TRUE, results='hide'}
subSamples <- createDataPartition(y=training$classe, p=0.75, list=FALSE)
subTraining <- training[subSamples, ] 
subTesting <- training[-subSamples, ]
```

## Expected out-of-sample error
The estimated out-of-sample error in the cross-validation data would correspond to the quantity: 1-accuracy. Accuracy is the proportion of correct, classified observation in the subTesting data set over the total sample. The estimated precision in the out-of-sample data set ( i.e. initial test data set) is assumed to be the exactness. The expected value of the out-of-sample error would therefore correspond to the estimated number of misclassified observations / total observations in the Test data set, which is the amount: 1-accuracy found from the cross-validation data set.

## Exploratory analysis
The variable `classe` contains 5 levels. The plot of the outcome variable shows the frequency of each levels in the subTraining data.

```{r exploranalysis, echo=TRUE}
barplot(table(subTraining$classe), col="skyblue", main="Levels of the variable classe", xlab="classe levels", ylab="Frequency")
```
