# Course_Project


The aim of the project is to clean and extract usable data from the "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip" zip file. R script called run_analysis.R that does the following: 
- Merges the training and the test sets to create one data set. 
- Extracts only the measurements on the mean and standard deviation for each measurement. 
- Uses descriptive activity names to name the activities #n the data set 
- Appropriately labels the data set with descriptive variable names. 
- From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

Libraries Used:

library(dplyr)

Downloading the data:

if (!file.exists("data_proj")) {
  dir.create("data_proj")
}

fileURL <-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

temp <- tempfile()
download.file(fileURL,temp)
list.files <- unzip(temp,list=TRUE)
list.files

## Reading the data:

- List of all features:
data_features <- read.table(unz(temp, list.files$Name[2]))
head(data_features)

- Links the class labels with their activity name:
data_Activ_Names <- read.table(unz(temp, list.files$Name[1]))
head(data_Activ_Names)


TRAIN DATA

- Training set:
data_Training_set <- read.table(unz(temp, list.files$Name[31]))
head(data_Training_set)

- Training labels:
data_Training_Activ <- read.table(unz(temp, list.files$Name[32]))
head(data_Training_Activ)

- Subject who performed the Training activity:
data_Training_subject <- read.table(unz(temp, list.files$Name[30]))
head(data_Training_subject)


TEST data:

- Test set:
data_Test_set <- read.table(unz(temp, list.files$Name[17]))
head(data_Test_set)

- Test labels:
data_Test_Activ <- read.table(unz(temp, list.files$Name[18]))
head(data_Test_Activ)

- Subject who performed the Test activity:
data_Test_subject <- read.table(unz(temp, list.files$Name[16]))
head(data_Test_subject)


Step 1- Merges the training and the test sets to create one data set.

Subject <- rbind(data_Training_subject, data_Test_subject)
colnames(Subject) <- "Subject"

Step 3- Uses descriptive activity names to name the activities in the data set

Activity <- rbind(data_Training_Activ, data_Test_Activ)
Activity <- left_join(Activity, data_Activ_Names, by = "V1")
Activity2 <- rename(Activity, Activity = V2)
Activity <- Activity2["Activity"]

Features <- rbind(data_Training_set, data_Test_set)
colnames(Features) <- t(data_features[2])

UCI_DATA <- cbind(Subject, Activity, Features)


Step 2- Extracts only the measurements on the mean and standard deviation for each measurement.


UCI_DATA <- tbl_df(UCI_DATA)

variable.names(UCI_DATA)

- Because there are duplicated columns names and i dont need these columns
UCI_DATA <- UCI_DATA[ , !duplicated(colnames(UCI_DATA))]


UCI_DATA_MEAN_STD <- select(UCI_DATA, Subject, Activity, contains("mean"), contains("std"))
variable.names(UCI_DATA_MEAN_STD)

dim(UCI_DATA_MEAN_STD)


Step 4- Appropriately labels the data set with descriptive variable names.

names(UCI_DATA_MEAN_STD) <- gsub("Acc", "Accelerometer", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("Gyro", "Gyroscope", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("BodyBody", "Body", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("Mag", "Magnitude", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("^t", "Time", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("^f", "Frequency", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("tBody", "Time_Body", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("-mean()", "Mean", names(UCI_DATA_MEAN_STD), ignore.case = TRUE)
names(UCI_DATA_MEAN_STD) <- gsub("-std()", "STD", names(UCI_DATA_MEAN_STD), ignore.case = TRUE)
names(UCI_DATA_MEAN_STD) <- gsub("-freq()", "Frequency", names(UCI_DATA_MEAN_STD), ignore.case = TRUE)
names(UCI_DATA_MEAN_STD) <- gsub("angle", "Angle", names(UCI_DATA_MEAN_STD))
names(UCI_DATA_MEAN_STD) <- gsub("gravity", "Gravity", names(UCI_DATA_MEAN_STD))
variable.names(UCI_DATA_MEAN_STD)

Step 5- From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

UCI_DATA_MEAN_STD$Subject <- as.factor(UCI_DATA_MEAN_STD$Subject)
Tidy_UCI_DATA_MEAN <- aggregate(. ~Subject + Activity, UCI_DATA_MEAN_STD, mean)
Tidy_UCI_DATA_MEAN <- arrange(Tidy_UCI_DATA_MEAN, Subject, Activity)
View(Tidy_UCI_DATA_MEAN)

- Generating a data set as a txt file
write.table(Tidy_UCI_DATA_MEAN, file = "Tidy_UCI_DATA_MEAN.txt", row.names = FALSE)




