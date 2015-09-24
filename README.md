---
title: "README.md"
author: "Yanan_Zhao"
date: "September 23, 2015"
output: html_document
---

1. Merges the training and the test sets to create one data set.

```r
## Read test files
test_subject_test <- read.table("./test/subject_test.txt")
test_X_test <- read.table("./test/X_test.txt")
test_y_test <- read.table("./test/y_test.txt")


## Read train files
train_subject_train <- read.table("./train/subject_train.txt")
train_X_train <- read.table("./train/X_train.txt")
train_y_train <- read.table("./train/y_train.txt")

## combine test and train dataset
combine_subject <- rbind(train_subject_train,test_subject_test)
combine_subject <- factor(combine_subject[,1])
combine_x_test <- rbind(train_X_train,test_X_test)
combine_mean <- colMeans(combine_x_test)
combine_std <- apply(combine_x_test,2,sd)
combine_y_test <- rbind(train_y_train,test_y_test)
```

Here I will do the part 3 first. Part 2 is after part 3.

3. Uses descriptive activity names to name the activities in the data set


```r
## convert numeric activity name to the corresponding activity name
library(plyr)
combine_y_test <- as.factor(combine_y_test[,1])
combine_y_test <- revalue(combine_y_test,c("1" = "WALKING", 
                                           "2" = "WALKING_UPSTAIRS",
                                           "3" = "WALKING_DOWNSTAIRS",
                                           "4" = "SITTING",
                                           "5" = "STANDING",
                                           "6" = "LAYING"))

## Assigne the full data frame col nanmes by using features file
features <- read.table("features.txt")
names(combine_x_test) <- features[,2]
```

2. Extracts only the measurements on the mean and standard deviation for each measurement. 


```r
## find the col names with keywords "mean" and "std"
mean_measurement_index <- grepl("mean",tolower(features[,2]))
std_measurement_index <- grepl("std",tolower(features[,2]))

## extract the col with "mean" and "std" variables
mean_std_measurment <- as.data.frame(c(combine_x_test[,mean_measurement_index],combine_x_test[,std_measurement_index]))
```

4. Appropriately labels the data set with descriptive variable names. 


```r
## Appropriately labels the data set with descriptive variable names.
temp_names <- colnames(mean_std_measurment)
temp_names <- gsub("tBody","Time_domain_Body_",temp_names)
temp_names <- gsub("fBody","Frequency_domain_Body",temp_names)
temp_names <- gsub("tGravity","Time_domain_Gravity_",temp_names)
temp_names <- gsub("Acc","Accleration_",temp_names)
temp_names <- gsub("Mag","Magnitude_",temp_names)
temp_names <- gsub("Gyro","Gyroscope_",temp_names)
colnames(mean_std_measurment) <- temp_names

## Add the Activity and Subject to the data frame
mean_std_measurment$Activity <- combine_y_test
mean_std_measurment$Subject <- combine_subject
```

5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.


```r
## generate tidy data set with the average of each variable for each activity and each subject
results <- aggregate(.~Activity+Subject,mean_std_measurment,FUN=mean)
write.table(results,file = "results.txt",row.names = FALSE)
```
