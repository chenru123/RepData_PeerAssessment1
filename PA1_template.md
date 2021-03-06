---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document
---

## 0. Data Source

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken

## 1. Loading and preprocessing the data

* Read the data into a data frame

```r
data <- read.csv("activity.csv",header=TRUE)
```

## 2. What is mean total number of steps taken per day?

**2.1** Before making a histogram of the total number of steps taken each day, we'll need to sum all the steps per day (ignore all the missing values)

```r
steps <- tapply(data$steps,data$date,na.rm=TRUE,FUN = sum)
```

Run the summary command to list the min/max, median and mean values of the data set

```r
summary(steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6778   10400    9354   12810   21190
```

**2.2** Make a histogram of the total number of steps taken each day

```r
hist(steps, breaks=10, ylim = c(0,20), xlim = c(0,25000), 
     xlab = "Total # of Steps", main = "Histogram Of The Total Number Of Steps Taken Each Day")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

**2.3** Calculate and report the **mean** and **median** total number of steps taken per day

```r
mean(steps, na.rm = TRUE)
```

```
## [1] 9354.23
```

```r
median(steps, na.rm = TRUE)
```

```
## [1] 10395
```

## 3. What is the average daily activity pattern?

**3.1** Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
dailyAvg <- aggregate(steps ~ interval, data = data,FUN = mean,na.rm=TRUE)

plot(dailyAvg$interval,dailyAvg$steps,type = "l", xlab = "5-minutes Interval",ylab = "Average # of Steps Taken", main = "Average Daily Activity Pattern")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

**3.2** Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
dailyAvg[which.max(dailyAvg$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

## 4. Imputing missing values

**4.1** Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

**4.2** Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

The strategy that I'll use is to fill in all the missing values with the mean for that 5-minute interval.

**4.2.1** Create a function looping thru the original dataset for missing value (steps = NA) and filling in with mean for that 5-minute interval


```r
fillValue <- function(steps, interval){
  if (!is.na(steps))
    filled <- steps
  else
    filled <- (dailyAvg[dailyAvg$interval==interval,"steps"])
}
```

**4.2.2** Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
newData <- data
newData$steps <- mapply(fillValue,newData$steps,newData$interval)
```

**4.3** Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Re-calcuate the total number of steps taken each day with filled missing value based on the strategy used above prior re-plotting the histogram


```r
newSteps <- tapply(newData$steps,newData$date,FUN = sum)

par(mfrow=c(1,2))
hist(steps, breaks=10, ylim = c(0,25), xlim = c(0,25000), 
     xlab = "Total # of Steps", main = "Histogram Of The Total Number Of Steps\n Taken Each Day (With Missing Value)")
hist(newSteps, breaks=10, ylim = c(0,25), xlim = c(0,25000), 
     xlab = "Total # of Steps", main = "Histogram Of The Total Number Of Steps\n Taken Each Day (Filled Missing Value)")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)

Calculate and report the **mean** and **median** total number of steps taken per day


```r
mean(newSteps)
```

```
## [1] 10766.19
```

```r
median(newSteps)
```

```
## [1] 10766.19
```

What is the impact of imputing missing data on the estimates of the total daily number of steps?

The "Mean" and "Median" values are higher after imputing missing data.  The behavior is caused by replacing missing "NA" steps value with filling in the mean for that 5-minute interval.  For the original data set, the mean and median calculation didn't account for those rows with NA in the steps column.

## 5. Are there differences in activity patterns between weekdays and weekends?

**5.1** Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
install.packages("lubridate")
```

```
## package 'lubridate' successfully unpacked and MD5 sums checked
## 
## The downloaded binary packages are in
## 	C:\Users\chenru\AppData\Local\Temp\RtmpMHiPxQ\downloaded_packages
```

```r
library(lubridate)

dayOfweek <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
        "weekend"
    } else {
        "weekday"
    }
}

newData$dayType <- as.factor(sapply(newData$date,dayofweek))
```

**5.2** Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using simulated data:


```r
par(mfrow=c(2,1))

newDailyAvgWeekend <- aggregate(steps ~ interval, data = newData, subset = (newData$dayType=="weekend"),FUN = mean)

plot(newDailyAvgWeekend,type="l", main="weekend", xlab="Interval",ylab="Number of Steps")

newDailyAvgWeekday <- aggregate(steps ~ interval, data = newData, subset = (newData$dayType=="weekday"),FUN = mean)

plot(newDailyAvgWeekday,type="l", main="weekday",xlab="Interval",ylab="Number of Steps")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png)
