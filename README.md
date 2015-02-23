---
title: "README"
author: "Michael Mappes"
date: "Sunday, February 22, 2015"
output: html_document
---

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

######          THIS FILE CONSISTS OF THE STEPS NECESSARY      ######
######          TO PRODUCE A TIMY DATA SET CONSISTING OF       ######
######          AVERAGES FOR THE MEAN AND STANDARD DEVIATION   ######
######          BY ACTIVITY AND SUBJECT FOR A NUMBER OF        ######
######          VARIABLES LISTED IN THE PUBLIC DOMAIN DATA     ######
######          SET DOR HUMAN ACTIVITY RECOGNITION USING       ######
######          SMARTPHONES                                    ######


# LINK TO STUDY ABSTRACT:   https://www.elen.ucl.ac.be/Proceedings/esann/esannpdf/es2013-84.pdf


########## R setup pre-processing steps###############################

##  1. install downloader package
install.packages("downloader")

## 2. add downloader to library
library(downloader)

## 1. Get source data from UCI Machine Learning Repository
temp <- tempfile()
fileURL = "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download(fileURL, dest=".//data/UCIHARdataset.zip", mode="wb")
unzip(".//data/UCIHARdataset.zip", exdir=".//data")

## 2. Create data frames for each of the 8 relevant text files
## FILES ARE CATEGORIZED BELOW

## TEST (w/ SUBJECTS) 
testdata <- read.table(".//data/UCI HAR Dataset/test/x_test.txt")
testdatalabels <- read.table(".//data/UCI HAR Dataset/test/y_test.txt")
testsubjects <- read.table(".//data/UCI HAR Dataset/test/subject_test.txt")

## TRAIN (w/ SUBJECTS) 
traindata <- read.table(".//data/UCI HAR Dataset/train/x_train.txt")
traindatalabels <- read.table(".//data/UCI HAR Dataset/train/y_train.txt")
trainsubjects <- read.table(".//data/UCI HAR Dataset/train/subject_train.txt")

## ACTIVITY
activity <- read.table(".//data/UCI HAR Dataset/activity_labels.txt")

## FEATURES
features <- read.table(".//data/UCI HAR Dataset/features.txt")

## 3. Join the y data to the activity labels for the test and train data frames
library(plyr)
test_activity <- join(testdatalabels, activity, by = 'V1')
train_activity <- join(traindatalabels, activity, by = 'V1')

## 4. Add the features as column names to the test and training data 
colnames(testdata) <- features[,2]
colnames(traindata) <- features[,2]

## 5. Subset each data frame to keep only the mean() and std() columns
subset_testdata <- testdata[grepl("[Mm]ean\\(\\)|std\\(\\)", names(testdata))]
subset_traindata <- traindata[grepl("[Mm]ean\\(\\)|std\\(\\)", names(traindata))]

## 6.  Add the activity column to the test and train data sets
testdata_activity <- cbind(subset_testdata, activity=test_activity$V2)
traindata_activity <- cbind(subset_traindata, activity=train_activity$V2)

## 7. Add the subjects column to the test and train data sets
testdata_final<- cbind(testdata_activity, subject=testsubjects$V1)
traindata_final<- cbind(traindata_activity, subject=trainsubjects$V1)

## 8.  Combine test and train into one dataset
final_dataset <- rbind(testdata_final, traindata_final)

## 9.  Create a tidy data set with the average of each variable for each activity and each subject
library(dplyr)
tidy_dataset <- final_dataset %>% group_by(activity,subject) %>% summarise_each(funs(mean))

## 10.  Use write.table to output the data set to a text file.   Use row.names = F
write.table(tidy_dataset, file="C:/Users/Mmappes/Dropbox/Coursera/R/data/tidy_dataset.txt", row.names = FALSE)



