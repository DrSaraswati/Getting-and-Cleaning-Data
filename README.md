# Getting-and-Cleaning-Data
The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set.
Footnote: I fully understand why data scientists spend most of their time retrieving and "cleaning" data. 

The information is the same as my run-analysis in describing what each code does in R.
Thank you!
#Load everything I **think** I'm going to need. 
library(data.table)
library(reshape2)
library(dplyr)
library(tidyr)

# Download File
setwd("C:/Getting and Cleaning Data/Week 4")
if(!file.exists("./data")){dir.create("./data")}
dataTableurl <-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(dataTableurl,destfile="./data/Dataset.zip",method="curl")

#Unzip File
unzip(zipfile="./data/Dataset.zip",exdir="./data")

#Read data into variable
f <-"C:/Getting and Cleaning Data/Week 4/UCI HAR Dataset"

#Read Subject Files into R
subTrain <- tbl_df(read.table(file.path(f, "train", "subject_train.txt")))
subTest  <- tbl_df(read.table(file.path(f, "test" , "subject_test.txt" )))

#Read Y Files (Activity) into R
yActiveTrain <- tbl_df(read.table(file.path(f, "train", "Y_train.txt")))
yActiveTest  <- tbl_df(read.table(file.path(f, "test" , "Y_test.txt" )))

#Read X data files into R
xDataTrain <- tbl_df(read.table(file.path(f, "train", "X_train.txt" )))
xDataTest  <- tbl_df(read.table(file.path(f, "test" , "X_test.txt" )))

#Row Bind Subject Train and Test Files Together and set Names
totalSubject <- rbind(subTrain, subTest)
setnames(totalSubject, "V1", "subject")

#Row Bind Y Train and Test Files Together and set names
totalYActivity<- rbind(yActiveTrain, yActiveTest)
setnames(totalYActivity, "V1", "activityNum")

#Row Bind the X data files
dataTable <- rbind(xDataTrain, xDataTest)

#Read “Features” into R (tbl_df worked the best in terms of not experiencing argument errors).  
dataFeatures <- tbl_df(read.table(file.path(f, "features.txt")))
setnames(dataFeatures, names(dataFeatures), c("featureNum", "featureName"))
colnames(dataTable) <- dataFeatures$featureName

#Read Activity Lables into R and set names.
activityLabels<- tbl_df(read.table(file.path(f, "activity_labels.txt")))
setnames(activityLabels, names(activityLabels), c("activityNum","activityName"))

# Merge columns
totalSubandYAct<- cbind(totalSubject, totalYActivity)
dataTable <- cbind(totalSubandYAct, dataTable)

# Extracting mean and standard deviation
dataFeaturesMeanStd <- grep("mean\\(\\)|std\\(\\)",dataFeatures$featureName,value=TRUE) #var name

#Unite Mean Std.Features and subset into data table
dataFeaturesMeanStd <- union(c("subject","activityNum"), dataFeaturesMeanStd)
dataTable<- subset(dataTable,select=dataFeaturesMeanStd) 

##Merge Activity Labels into Data Table
dataTable <- merge(activityLabels, dataTable , by="activityNum", all.x=TRUE)
dataTable$activityName <- as.character(dataTable$activityName)

##Conjure dataTable with variable means sorted by subject and Activity
dataTable$activityName <- as.character(dataTable$activityName)
dataAggr<- aggregate(. ~ subject - activityName, data = dataTable, mean) 
dataTable<- tbl_df(arrange(dataAggr,subject,activityName))

#View to check the data visually.
head(str(dataTable),2)

#Replaces string match with name in data table
names(dataTable)<-gsub("std()", "SD", names(dataTable))
names(dataTable)<-gsub("mean()", "MEAN", names(dataTable))
names(dataTable)<-gsub("^t", "time", names(dataTable))
names(dataTable)<-gsub("^f", "frequency", names(dataTable))
names(dataTable)<-gsub("Acc", "Accelerometer", names(dataTable))
names(dataTable)<-gsub("Gyro", "Gyroscope", names(dataTable))
names(dataTable)<-gsub("Mag", "Magnitude", names(dataTable))
names(dataTable)<-gsub("BodyBody", "Body", names(dataTable))

#View to check the data visually.
head(str(dataTable),6)

#Tidy Data! Write data to csv (I also wrote to a txt file for the course requirements, but in real time, I'd write it to a csv for my own viewing.
write.csv(dataTable, "TidyData1.csv")
