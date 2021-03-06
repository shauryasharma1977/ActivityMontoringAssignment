---
title: "Reproducible Research: Peer Assessment 1"
author: "Shaurya"
date: "9/30/2020"
output: 
  html_document:
    keep_md: true
---

## Introduction
This is my submission towards the reporoducible Research assignment. This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

My goal in this assignment is to

- Load the data and clean it if required
- Analyze the dataset for key variables steps, intervals and find patterns in activity
- Address inadequacies of the dataset which may skew the analysis (NA values etc)
- Answer all the questions as requested in the assignment


## Loading and preprocessing the data
- 17568 rows are retrieved along with 3 variables steps (double), date (as date) and interval(double)
- Only steps variable has NA values which is expected as per the assignment.

```r
# Load data
if (!file.exists("./data/activity.csv") )
{
        dlurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"  
        download.file(dlurl,destfile='./data/activity.zip',mode='wb')  
        unzip("./data/activity.zip" , exdir = "./data")
}

## load the activity file
activityfile <- read_csv("./data/activity.csv")
## View the dataset
str(activityfile)
```

```{.bg-info}
## tibble [17,568 x 3] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ steps   : num [1:17568] NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date[1:17568], format: "2012-10-01" "2012-10-01" ...
##  $ interval: num [1:17568] 0 5 10 15 20 25 30 35 40 45 ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   steps = col_double(),
##   ..   date = col_date(format = ""),
##   ..   interval = col_double()
##   .. )
```
## What is mean total number of steps taken per day?
As per the histogram of total steps per day, the most frequent steps are in the 10000-15000 range.

```r
## Total steps per day, each observation is one day, 
total_steps_per_day <- aggregate(steps ~ date, data=activityfile, sum)

## Plot a histogram
hist(total_steps_per_day$steps,  
                xlab = "Number of Steps", 
                        main ="Total Steps Distribution", 
                                col = "skyblue1")
```

![](PA1_template_files/figure-html/totalstepsperday-1.png)<!-- -->

Calculate mean
The mean is 1.076619 and the median is 10765


```r
originalmean <- mean(total_steps_per_day$steps)
originalmean
```

```{.bg-info}
## [1] 10766.19
```
Calculate median

```r
originalmedian <- median(total_steps_per_day$steps)
originalmedian
```

```{.bg-info}
## [1] 10765
```
## What is the average daily activity pattern?
- Flat till 500 and activity increases from there on till 9.
- Peaks int he morning between 900 and 1000. Perhaps participants have got to their workplaces/schools and then activity reduces
- During Noon 1100-1230 activity increases again (perhaps people move about for lunch/coffee etc)
- Post 3000 we see peaks and troughs till 1900 post which activity reduces

```r
## mean steps per interval, 
mean_steps_per_interval <- aggregate(steps ~ interval, data=activityfile, mean)

## Plot a histogram
plot(x=mean_steps_per_interval$interval, 
        y=mean_steps_per_interval$steps, type="l",
                xlab="Interval", ylab="Average Number of Steps", 
                        main="Average Number of Steps per Interval", 
                                col = "blue")
```

![](PA1_template_files/figure-html/meanstepsperinterval-1.png)<!-- -->

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps
The maxiumum number of steps are contained in the 835 interval.

```r
##rowindex with max value
maxrow <- which.max(mean_steps_per_interval$steps)
## which interval has the max # of steps at an average
mean_steps_per_interval$interval[maxrow]
```

```{.bg-info}
## [1] 835
```
## Imputing missing values


```r
## check the number of NA values for steps
sum(!complete.cases(activityfile))
```

```{.bg-info}
## [1] 2304
```
Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, 
Create a new dataset that is equal to the original dataset but with the missing data filled in

```r
## Strategy to fill missing steps values
## update steps with mean of steps for that interval (across all days)
## only when mean is NA

## Step 1 calculate the mean steps per interval
## to be used in filling NA in our steps data
steps_by_interval <- activityfile %>%
        filter(!is.na(steps)) %>%
        group_by(interval) %>%
        summarize(meansteps = mean(steps))


## update steps and create new dataset
activityfilecleaned <-  activityfile %>%
                                inner_join(steps_by_interval, by = c("interval"="interval")) %>%
                                        mutate(steps = ifelse(is.na(steps), meansteps, steps))        


## We can drop the meansteps column
## This is a new dataset that is equal to the original 
## dataset but with the missing data filled in
activityfilecleaned <- 
        activityfilecleaned %>%
        select (steps, date, interval)
```
Report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? 

-The mean is the same between original and imputed dataset
-The median has a slight variance(Median Var:1.189) between original and imputed dataset


```r
## total number of steps each day
cleaned_total_steps_per_day <- aggregate(steps ~ date, data = activityfilecleaned, sum, na.rm = TRUE)

## calculate mean for the cleaned data set
mean(cleaned_total_steps_per_day$steps)
```

```{.bg-info}
## [1] 10766.19
```

```r
## Difference with original mean
print(paste0(round(originalmean - mean(cleaned_total_steps_per_day$steps), digits=3), " is the difference between mean of Original and  of Imputed data", collapse=" "))
```

```{.bg-info}
## [1] "0 is the difference between mean of Original and  of Imputed data"
```
Median

```r
## calculate median for the cleaned data set
median(cleaned_total_steps_per_day$steps)
```

```{.bg-info}
## [1] 10766.19
```

```r
print(paste0(round(originalmedian - median(cleaned_total_steps_per_day$steps), digits=3), " is the difference between median Original and Imputed data", collapse=" "))
```

```{.bg-info}
## [1] "-1.189 is the difference between median Original and Imputed data"
```
Make a histogram of the total number of steps taken each day.
What is the impact of imputing missing data on the estimates of the total daily number of steps?
Answer: The missing data is all within the 10k-15k steps/day range which means the missing data was for the days that already have the maximum number of steps. Approximately data for another 7-8 days was added.

```r
hist(cleaned_total_steps_per_day$steps, main = paste("Total Steps Each Day"), col="blue", xlab="Number of Steps")

##plot origibal total number of steps histogram on top of this one.
hist(total_steps_per_day$steps, main = paste("Total Steps Frequency"), 
        col="skyblue", xlab="Number of Steps", add=T)
                legend("bottom", c("Imputed", "Non-imputed"), col=c("blue", "skyblue"), lwd=5)
```

![](PA1_template_files/figure-html/newactivitytotalnumberofstepsplot-1.png)<!-- -->
## Are there differences in activity patterns between weekdays and weekends?
- The morning peaks on a weekday are higher than those on weekdays. This maybe du to to weekday activity of chosen participants. On weekend it is in excess of 220 where as week day is ~150 approx.
- Between 1000 ad 3000, the activity level(peaks) of weekend is relatively higher. Again this might be due to inactivity of participant (work on desk) during the day
- Between 3000 - 6000, we see elevated peaks on weekend.

Question:-Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
## add a variable to tag weekend/weekday
activityfilecleaned <- 
        activityfilecleaned %>%
                mutate(daytype = ifelse((weekdays(date)=="Sunday" | weekdays(date)=="Saturday"), "weekend", "weekday"))  

## convert into a factor variable
activityfilecleaned$daytype = factor(activityfilecleaned$daytype, levels = c("weekday","weekend"))
```


Make a panel plot containing a time series plot  of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

```r
steps_by_interval_by_daytype <- activityfilecleaned %>%
        group_by(daytype, interval) %>%
        summarize(steps = mean(steps))


## Plot it
library(lattice)
xyplot(steps_by_interval_by_daytype$steps ~ steps_by_interval_by_daytype$interval|steps_by_interval_by_daytype$daytype, 
       main="Average Steps by Interval (weekday vs weekend)",
        xlab="Interval", 
                ylab="Steps",layout=c(1,2), type="l")
```

![](PA1_template_files/figure-html/timeseriesplotnumberofsteps-1.png)<!-- -->
