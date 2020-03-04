This repositaory contains the assignment of Getting and Cleaning Data.


---------------------------------------------------------------------
files contained in this repositary:
README.md
run_analysis.R: the script for producing the analysis required.
Codebook.pdf: a pdf file, in which the variables names are explained.
---------------------------------------------------------------------

Explaination of the script:

### create a file and set a working directory
if(!file.exists('./Getting and Cleaning Data/Assignment')){
        dir.create('./Getting and Cleaning Data/Assignment')
}
setwd('../')

### download the data set
Url <- 'https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip'
download.file(Url, './accelerometer_Samsung.zip', method = 'curl')

### unzip the file
unzip('./accelerometer_Samsung.zip')

### load the data into r
train_sub <- read.table('./UCI HAR Dataset/train/subject_train.txt')
train_set <- read.table('./UCI HAR Dataset/train/X_train.txt')
train_label <- read.table('./UCI HAR Dataset/train/y_train.txt')

test_set <- read.table('./UCI HAR Dataset/test/X_test.txt')
test_label <- read.table('./UCI HAR Dataset/test/y_test.txt')
test_sub <- read.table('./UCI HAR Dataset/test/subject_test.txt')

feature <- read.table('./UCI HAR Dataset/features.txt', stringsAsFactors = F)
Act_label <- read.table('.//UCI HAR Dataset/activity_labels.txt', stringsAsFactors = F)


### naming the columns by passing the elements in the file 'feature' to the column names of both data set
colnames(test_set) <- feature$V2
train_set <- `colnames<-`(train_set, feature$V2)


### merge the Subject and Activity columns to the main data, so that the data can be grouped by there two variable 
### afterwards
withSub_test <- cbind.data.frame('Subject' = test_sub$V1, 
                                 'Activity' = test_label$V1, test_set)
withSub_train <- cbind('Subject' = train_sub$V1, 'Activity' = train_label$V1, train_set)

### join the train and test data
binded <- rbind.data.frame(withSub_train, withSub_test)

### searching for the indexes, in which columns contain the phrase 'mean()' or 'std()' in the binded file,
### so that the columns containing there phrase could be subsetted.
index_mean_and_std <- grep('mean[()]|std[()]', names(binded))
needed_col <- c(1, 2, index_mean_and_std)               ## 1st and 2nd columns from Subject and Activity also included

### selecting the columns needed
mean_std <- binded[, needed_col]

### making descriptive labels for 'Activity' column, using the package 'dplyr'
library(dplyr)
mean_std <- mean_std %>% tbl_df
## using mutate function to turn the 'Activity' column into factor with the corresponding labels
mean_std <- mutate(mean_std, Activity = factor(Activity, labels = Act_label$V2))

### group the data on the basis of Subject and Activity to analyse the mean of each variables
grouped <- group_by(mean_std, Subject, Activity)
mean_grouped <- grouped %>% summarise_all(.funs = mean, na.rm = T)
## naming the summary columns
colnames(mean_grouped)[3:length(names(mean_grouped))]<- 
        paste('mean of', names(mean_grouped)[3:length(names(mean_grouped))])

---------------------------------------------------------------------------------------------------

Code for reading the output into R:

read.table('./step5_output.txt', check.names = FALSE)
