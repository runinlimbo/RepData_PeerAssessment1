# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Data has been read into R to analyse using the following code:


```r
URL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
if(!file.exists("~/Documents/DataScience/Course 5/Programming Assignment 1/DATA"))
{dir.create("~/Documents/DataScience/Course 5/Programming Assignment 1/DATA")}
download.file(URL,destfile = 
                "~/Documents/DataScience/Course 5/Programming Assignment 1/DATA/FitnessDevice_Dataset.zip")
unzip(zipfile=
        "~/Documents/DataScience/Course 5/Programming Assignment 1/DATA/FitnessDevice_Dataset.zip"
      ,exdir="~/Documents/DataScience/Course 5/Programming Assignment 1/DATA")
list.files("~/Documents/DataScience/Course 5/Programming Assignment 1/DATA")
```

```
## [1] "activity.csv"              "FitnessDevice_Dataset.zip"
```

```r
#  "activity.csv"              "FitnessDevice_Dataset.zip"

# Read Data into R
setwd("~/Documents/DataScience/Course 5/Programming Assignment 1/DATA")
FIT_DATA <- read.table("activity.csv",sep = ",",header = TRUE)
head(FIT_DATA)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
names(FIT_DATA)
```

```
## [1] "steps"    "date"     "interval"
```

Notice missing entries in the steps field.  Missing entries will be addressed later in the project.

Confirm fields are of proper class:

```r
class(FIT_DATA$steps)
```

```
## [1] "integer"
```

```r
class(FIT_DATA$date)
```

```
## [1] "factor"
```

```r
class(FIT_DATA$interval)
```

```
## [1] "integer"
```

Adjusting Date:

```r
FIT_DATA$date <- as.Date(FIT_DATA$date,format="%Y-%m-%d")
class(FIT_DATA$date) #DATE
```

```
## [1] "Date"
```


## What is mean total number of steps taken per day?
First, aggregate the steps.  The na.rm=True statement will ignore all NA fields:

```r
Steps_Sum <-aggregate(steps ~ date, data=FIT_DATA, sum, na.rm=TRUE)
```
Next, draw a histogram, which displays the total number of steps taken each day

```r
     hist(Steps_Sum$steps,
     main = "Histogram of Total Steps
     Taken Each Day",
     xlab = "Steps in a Day",
     ylab = "Day Count",
     col= "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

The output appears to be normal.  The mean and median are as follows:

```r
mean(Steps_Sum$steps)
```

```
## [1] 10766.19
```

```r
median(Steps_Sum$steps)
```

```
## [1] 10765
```
#### The mean total number of steps taken per day is 10,766.19 and the median is 10,765.


## What is the average daily activity pattern?
To address the average daily activity pattern, look at the average number of steps taken over each respective 5-minute interval within the study period.
 First, take the average of the steps over each interval (excluding NA entries):

```r
 Steps_Mean <- aggregate(steps~interval,data=FIT_DATA, mean, na.rm=TRUE)
```
 
 Using ggplot2, plot the average step activity throughout the day:

```r
library(ggplot2)
ggplot(data = Steps_Mean,
       aes(x=interval,y=steps)) +
  geom_line(stat="identity",col="blue")+
  ggtitle("Average Steps for each 5-Minute Interval within a Day over Study Period")+
  labs(x="Daily 5-Min Interval", y = "Average Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

It appears that the daily steps are the greatest between the 750 and 1000 minute intervals.  Using R code, the 5-minute interval that, on average, contains the maximum number of steps equals:

```r
Most_Steps_Interval <- Steps_Mean[which.max(Steps_Mean$steps),]
```

#### On average, the most step activity occurs during time interval 835, at 206.17 steps.


## Imputing missing values

To address missing values, first determine what fields have missing values:

```r
Missing_Steps <-sum(is.na(FIT_DATA$steps))
Missing_Date <-sum(is.na(FIT_DATA$date))
Missing_Interval<-sum(is.na(FIT_DATA$interval))
missings <- c(Missing_Steps, Missing_Date, Missing_Interval)
names(missings) <- c("steps","date","interval")
missings
```

```
##    steps     date interval 
##     2304        0        0
```
Only steps have missing values (2,304 total "NAs").

Missing values represent approximately 13% of total values:

```r
Mean_Missing <- mean(is.na(FIT_DATA$steps))
```

Replace these missing values with the average daily steps calculated previously for each daily 5-minute interval.

Step 1: Create an adjusted step column with no missing values

```r
FIT_DATA2<- merge(x = FIT_DATA, y = Steps_Mean, by = "interval", all.x=TRUE)
```
Step 2: Create adjusted steps column

```r
FIT_DATA2$steps_adj <- FIT_DATA2$steps.x
```
Step 3: Replace missings (NA) with Interval Mean

```r
# Subset Missings
FIT_DATA_NA <- FIT_DATA2[which(is.na(FIT_DATA2$steps.x)),]
# Assign Average Interval Steps
FIT_DATA_NA$steps_adj = FIT_DATA_NA$steps.y
# Subset NonMissings
FIT_DATA <- FIT_DATA2[which(!(is.na(FIT_DATA2$steps.x))),]
# Re-Combine data
FIT_DATA <-rbind(FIT_DATA,FIT_DATA_NA)
```
Step 4: Check that adjustments are accurate

```r
CHECK_ADJ <-FIT_DATA[which(is.na(FIT_DATA$steps.x)),]
CHK1 <- which(!(CHECK_ADJ$steps.y == CHECK_ADJ$steps_adj)) ##N = 0
CHECK_ADJ <-FIT_DATA[which(!(is.na(FIT_DATA$steps.x))),]
CHK2 <- which(!(CHECK_ADJ$steps.x == CHECK_ADJ$steps_adj)) ##N = 0
CHK3 <- sum(is.na(FIT_DATA$steps_adj)) #N = 0
```
Step 5: Clean up data frame

```r
# Remove Average Daily Interval
FIT_DATA <- subset(FIT_DATA,,-c(steps.y)) 
# Rename orignal steps as "steps"
names(FIT_DATA)[names(FIT_DATA)=="steps.x"] <- "steps"
```
The data is now ready to analyze with the adjustments made to the missing values.  The original step data can be found in the steps field.  The adjusted values can be found in the steps_adj field:

```r
head(FIT_DATA)
```

```
##   interval steps       date steps_adj
## 2        0     0 2012-11-23         0
## 3        0     0 2012-10-28         0
## 4        0     0 2012-11-06         0
## 5        0     0 2012-11-24         0
## 6        0     0 2012-11-15         0
## 7        0     0 2012-10-20         0
```


Histogram of the total number of steps taken each day after missing values are imputed:


```r
# Aggregate adjusted steps
Steps_Sum_Adj <- aggregate(steps_adj~date,data=FIT_DATA,sum)
# Draw histogram
hist(Steps_Sum_Adj$steps_adj,
     main = "Histogram of Total Steps Taken Each Day
     Adjusted for Missing Data",
     xlab = "Steps in a Day",
     ylab = "Day Count",
     col= "red")
```

![](PA1_template_files/figure-html/unnamed-chunk-18-1.png)<!-- -->

```r
AdjustedMean <- mean(Steps_Sum_Adj$steps_adj)
AdjustedMedian <- median(Steps_Sum_Adj$steps_adj)
```
#### The adjustments have minimal impact to the mean number of steps per day (10,766.19), but increased the median number of steps per day (10,766.19 vs. 10,765 excluding NAs).


## Are there differences in activity patterns between weekdays and weekends?
Depending on weekly schedules, it is possible that one may behave differently on the weekends compared to the weekdays.  To visualize weekend vs. weeday daily step patterns, develop a weekend vs. weekday panel plot of average daily steps over the 5-minute intervals. 

Create a Weeday Indicator, Summarize Data, and Plot results

```r
# Assign numeric values to days
# 1 = Sunday,....7 = Saturday
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
FIT_DATA$Day <- wday(FIT_DATA$date)

# Categorize Weekend vs. Weekday
FIT_DATA$Weekend <-"Weekday"
FIT_DATA$Weekend[FIT_DATA$Day==1]<-"Weekend"
FIT_DATA$Weekend[FIT_DATA$Day==7]<-"Weekend"
head(FIT_DATA)
```

```
##   interval steps       date steps_adj Day Weekend
## 2        0     0 2012-11-23         0   6 Weekday
## 3        0     0 2012-10-28         0   1 Weekend
## 4        0     0 2012-11-06         0   3 Weekday
## 5        0     0 2012-11-24         0   7 Weekend
## 6        0     0 2012-11-15         0   5 Weekday
## 7        0     0 2012-10-20         0   7 Weekend
```

```r
# Aggregate data by interval and weekend category
Weekend_Mean <-aggregate(steps_adj ~ interval+Weekend, data=FIT_DATA, mean)

# Plot the results
library(grid)
library(gridExtra)
ggplot(data = Weekend_Mean,
          aes(x=interval,
              y=steps_adj, #Sum
              fill=Weekend,
              colour = factor(Weekend))) +
  geom_line(stat="identity") +
  facet_grid(Weekend ~.)+
  labs(x = "5-Min Interval within a Day", y = "Average Steps")  +
  guides(fill=FALSE)+
  ggtitle("Panel Plot describing the average daily 5-min interval 
          steps on Weekdays vs. Weekends")+
  theme(legend.position="none")
```

![](PA1_template_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

#### Results show that there is generally earlier activity on the weekdays, followed by generally the same step activity during the middle of the day, and generally lower step activity in the evenings compared to weekend activity.  One could assume that during the weekdays, one would need to get up earlier to get to work in the morning.  It would appear that the individual monitored for activity has relatively the same amount of activity during the 9-5 workday schedule as the individual does on the weekends.  However, the individual is slightly more active in the evenings on the weekend.






