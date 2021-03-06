---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

The data archive is unzipped and the included raw csv file is read.


```r
unzip('activity.zip','activity.csv')
data <- read.csv('activity.csv')
```

## What is mean total number of steps taken per day?

The total steps summed up for each day.


```r
aggregatedData <- aggregate(data$steps, by=list(data$date), FUN=sum)
colnames(aggregatedData) <- c('date','steps')
plot(aggregatedData)
```

![plot of chunk stepsperday](figure/stepsperday-1.png) 

Mean and median are computed. Any missing values are just ignored.


```r
meanDailySteps <- mean(aggregatedData$steps, na.rm=TRUE)
medianDailySteps <- median(aggregatedData$steps, na.rm=TRUE)
```

The mean number of steps per day is 10766.188679.

The median number of steps per day is 10765.

## What is the average daily activity pattern?

Aggregate all data with respect to the time interval.

```r
dailyPattern <- aggregate(data$steps, by=list(data$interval), 
                          FUN= function(v) mean(v, na.rm=TRUE))
colnames(dailyPattern) <- c('time','steps')
plot(dailyPattern, type='l')
```

![plot of chunk dailypattern](figure/dailypattern-1.png) 

Find the most active interval.


```r
mostActiveTimeIndex <- which.max(dailyPattern$steps)
mostActiveTime <- dailyPattern$time[mostActiveTimeIndex]
```

The person was taking most steps in the time interval from 835 to 840.

## Inserting missing values


```r
totalNAs <- sum(is.na(data$steps))
```

There were 2304 missing entries in the dataset.

The missing data is filled up with the rounded average for that particular time interval.


```r
dataClean <- data
for (rowIndex in which(is.na(data$steps))) {
  dataClean[rowIndex, 'steps'] <- 
    round(dailyPattern[dailyPattern$time==data[rowIndex,'interval'],'steps'])
}
```

Replot the daily pattern after filling in missing values.


```r
aggregatedData <- aggregate(dataClean$steps, by=list(dataClean$date), FUN=sum)
colnames(aggregatedData) <- c('date','steps')
plot(aggregatedData)
```

![plot of chunk stepsperdayclean](figure/stepsperdayclean-1.png) 

## Are there differences in activity patterns between weekdays and weekends?

Aggregate the cleaned data accorting to interval and weekend-weekday.


```r
dataClean$isWeekend <- format(as.POSIXct(data$date), format='%w') %in% c(0,6)
dailyPattern <- aggregate(dataClean$steps, 
                          by=list(dataClean$interval, dataClean$isWeekend), 
                          FUN= function(v) mean(v, na.rm=TRUE))
colnames(dailyPattern) <- c('time','isWE', 'steps')
```

Plotting the results.


```r
par(mfrow = c(2, 1))
par(cex = 0.6)
par(mar = c(3, 3, 3, 3), oma = c(4, 4, 0.5, 0.5))
par(tcl = -0.25)
par(mgp = c(2, 0.6, 0))
plot(dailyPattern$time[dailyPattern$isWE], dailyPattern$steps[dailyPattern$isWE], 
     main="Weekend", xlab='interval', ylab='steps', type='l')
plot(dailyPattern$time[!dailyPattern$isWE], dailyPattern$steps[!dailyPattern$isWE], 
     main="Weekdays", xlab='interval', ylab='steps', type='l')
```

![plot of chunk weeklypattern](figure/weeklypattern-1.png) 

