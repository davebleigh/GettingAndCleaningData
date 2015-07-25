#Getting and Cleaning Data Course Assignment

This is the source file for the Coursera course "Getting and Cleaning Data" Week 4 Project Assignment

The source data was downloaded from here:

  https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

and was unzipped into the current directory.  All file paths will be relative references from that parent folder.

Here are the course instructions:

You should create one R script called run_analysis.R (this file) that does the following. 

    1. Merges the training and the test sets to create one data set.
    2. Extracts only the measurements on the mean and standard deviation for each measurement. 
    3. Uses descriptive activity names to name the activities in the data set
    4. Appropriately labels the data set with descriptive variable names. 
    5. From the data set in step 4, creates a second, independent tidy data set with 
       the average of each variable for each activity and each subject.


# Reading the Data Files
First we will need to read in the Training and Test data sets.  Both sets of data are split into 3 files: subject, activity and features.  6 files in all.
```R
trnSubject  <- read.table ( "UCI HAR Dataset/train/subject_train.txt", header = FALSE )
trnActivity <- read.table ( "UCI HAR Dataset/train/y_train.txt",       header = FALSE )
trnFeatures <- read.table ( "UCI HAR Dataset/train/X_train.txt",       header = FALSE )

tstSubject  <- read.table ( "UCI HAR Dataset/test/subject_test.txt", header = FALSE )
tstActivity <- read.table ( "UCI HAR Dataset/test/y_test.txt",       header = FALSE )
tstFeatures <- read.table ( "UCI HAR Dataset/test/X_test.txt",       header = FALSE )
```
# Step 1
Now we can perform step 1 of the course instructions, merging the testing and training data sets into 3 combined data sets for Subject, Activity and Features
```R
cmbSubject  <- rbind ( tstSubject, trnSubject )
cmbActivity <- rbind ( tstActivity, trnActivity )
cmbFeatures <- rbind ( tstFeatures, trnFeatures )
```
Before we move on, it will make things easier if we set the columns from the description file features.txt.  We will also set names for the activity and subject data
```
featureColumns <- read.table("UCI HAR Dataset/features.txt")
colnames ( cmbFeatures ) <- t( featureColumns[2] )

colnames ( cmbSubject )  <- "Subject"
colnames ( cmbActivity ) <- "Activity"
```
Now we can merge the data into a single data set using cbind()
```
cmbData <- cbind ( cmbSubject, cmbActivity, cmbFeatures )
```
# Step 2
Now we can perform Step 2 of the course instructions, trimming the data set down to just the Mean and Std Deviation columns (plus the activity and subject which are columns 1 and 2)
```
allColnames <- names ( cmbData )
colnamesToKeep <- grep ( ".*mean.*|.*std.*", allColnames, ignore.case = TRUE )

columnsToKeep <- c ( 1, 2, colnamesToKeep )

cmbData <- cmbData[, columnsToKeep]
```
# Step 3
Now we can perform Step 3 of the course instructions, replacing the activity field which is 1, 2, 3 etc with "Walking", "Walking Upstairs", "Walking Downstairs" etc
```
cmbData$Activity[cmbData$Activity==1] <- "Walking"
cmbData$Activity[cmbData$Activity==2] <- "Walking Upstairs"
cmbData$Activity[cmbData$Activity==3] <- "Walking Downstairs"
cmbData$Activity[cmbData$Activity==4] <- "Sitting"
cmbData$Activity[cmbData$Activity==5] <- "Standing"
cmbData$Activity[cmbData$Activity==6] <- "Laying"
```
# Step 4
Now we can perform Step 4 of the course instructions, giving the combined, filtered data set descriptive variable names.  I take this to mean replacing "t" with "Time" and "f" with "Frequency", along with expanding "Acc" into "Accelerometer" etc.  I added to the list until I was satisfied with the final set of names...
```
names ( cmbData ) <- gsub ( "^t", "Time",      names ( cmbData ) )
names ( cmbData ) <- gsub ( "^f", "Frequency", names ( cmbData ) )

names ( cmbData ) <- gsub ( "Acc",     "Accelerometer", names ( cmbData ) )
names ( cmbData ) <- gsub ( "Gyro",    "Gyroscope",     names ( cmbData ) )
names ( cmbData ) <- gsub ( "Mag",     "Magnitude",     names ( cmbData ) )
names ( cmbData ) <- gsub ( "angle",   "Angle",         names ( cmbData ) )
names ( cmbData ) <- gsub ( "gravity", "Gravity",       names ( cmbData ) )

names ( cmbData ) <- gsub ( "BodyBody",  "Body", names ( cmbData ) )

names ( cmbData ) <- gsub ( "-mean\\(\\)",  "Mean", names ( cmbData ) )
names ( cmbData ) <- gsub ( "-std\\(\\)",   "Std", names ( cmbData ) )

names ( cmbData ) <- gsub ( "-meanFreq\\(\\)",  "MeanFrequency", names ( cmbData ) )

names ( cmbData ) <- gsub ( "tBody", "TimeBody", names ( cmbData ) )

names ( cmbData ) <- gsub ( "AccelerometerJerkMean\\)",  "AccelerometerJerkMean", names ( cmbData ) )
```
# Step 5
Now we can perform Step 5 of the course instructions, creating a tidy data set with the average of each variable for each activity and each subject.
```
tidyData <- aggregate ( . ~ Activity + Subject, cmbData, mean )
```
# Writing the Results
All that is left is to write out the data so we can upload for marking.  The instructions requested no row mames.
```
write.table ( tidyData, "tidydata.txt", row.names = FALSE )
```
