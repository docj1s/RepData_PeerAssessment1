---
title: "PA1_template"
author: "Rashaad Jones"
date: "September 17, 2017"
output: html_document
---

#Reproducible Research: Peer Assessment #1



##Loading and preprocessing the data

```r
  myData <<- read.csv("activity.csv")
```

##What is the mean total number of steps taken per day?
-**Calculate the total number of steps take per day**

```r
    library(dplyr) 
     histData<-aggregate(steps~date, myData, sum)
```

-**Make a histogram of the total number of steps taken each day**

```r
    hist(histData$steps, breaks=nrow(histData))
```

<img src="PA1_template_files/figure-html/hist-1.png" width="672" />

-**Calculate and report the mean and median of the total number of steps taken per day**  
The mean is 1.0766189\times 10^{4}.  
The median is 10765.
    
##What is the average daily activity pattern?    
-**Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)**

```r
 time_series <- with(myData, tapply(steps, interval, mean, na.rm = TRUE))

    plot(row.names(time_series), time_series, type = "l", xlab = "5-min interval", 
         ylab = "Average Steps Taken Across All Days", main = "Average Number of Steps Taken", 
         col = "red")
```

<img src="PA1_template_files/figure-html/timeplot-1.png" width="672" />
    
-**Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?**   
    835    
    
##Imputing missing values
-**Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)**

```r
    sum(is.na(myData))
```

```
## [1] 2304
```
Missing values is `r sum(is.na(myData))

-**Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc**

```r
fillData <- function(FUN=mean)
{
    stepsAvg <- summarise(group_by(myData, interval), mean(steps, na.rm=TRUE))
    
    names(stepsAvg) <- c("interval", "steps")
    
    filled <- numeric()
    for (i in 1:nrow(myData)) {
        obs <- myData[i, ]
        if (is.na(obs$steps)) {
            steps <- subset(stepsAvg, interval == obs$interval)$steps
        } else {
            steps <- obs$steps
        }
        filled <- c(filled, steps)
    }
    
    myDataFilled <- myData
    myDataFilled$steps <- filled
    
    myDataFilled
}
```

-**Create a new dataset that is equal to the original dataset but with the missing data filled in.**

```r
   myDataFilled <- fillData(myData)
```

-**Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?**

```r
    stepsTotal <- summarise(group_by(myDataFilled, date), sum(steps, na.rm=TRUE))
    names(stepsTotal) <- c("date", "sumSteps")
    
    hist(stepsTotal$sumSteps)
```

<img src="PA1_template_files/figure-html/histFill-1.png" width="672" />

Mean: 1.0766189\times 10^{4}    
Median: 1.0766189\times 10^{4}

The mean and the median has NOT changed


##Are there differences in activity patterns between weekdays and weekends?
-**Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.**

```r
    myDataFilled$date <- as.Date(myDataFilled$date)
    day <- weekdays(myDataFilled$date)
    dayLevel <- vector()
    
    for(i in 1:nrow(myDataFilled))
    {
        if(day[i] == "Saturday" | day[i] == "Sunday")
        {
            dayLevel[i] <- "Weekend"
        }
        else
            dayLevel[i] <- "Weekday"
    }
```

-**Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).**

```r
    library(lattice)
    
    myDataFilled$dayLevel <- as.factor(dayLevel)
    
    stepsByDay <- aggregate(steps ~ interval + dayLevel, data = myDataFilled, mean)
    names(stepsByDay) <- c("interval", "dayLevel", "steps")
    
    xyplot(steps ~ interval | dayLevel, stepsByDay, type = "l", layout = c(1, 2), 
           xlab = "Interval", ylab = "Number of steps")
```

<img src="PA1_template_files/figure-html/panelPlot-1.png" width="672" />

##ENTIRE CODE (with results hidden)

```r
openFile <- function()
{
    myData <<- read.csv("activity.csv")
}


createHist <- function()
{
    openFile()
    
    histData<-aggregate(steps~date, myData, sum)
    hist(histData$steps,  hist(histData$steps, breaks=nrow(histData)) )
    
    dev.copy(png, file="SumSteps.png", width=480, height=480)
    dev.off()
    
    print(mean(histData$steps))
    print(median(histData$steps))
}

computeAvgActivity <- function()
{
    openFile()
    
    time_series <- with(myData, tapply(steps, interval, mean, na.rm = TRUE))

    plot(row.names(time_series), time_series, type = "l", xlab = "5-min interval", 
         ylab = "Average Steps Taken Across All Days", main = "Average Number of Steps Taken", 
         col = "red")
    
    dev.copy(png, file="SumSteps.png", width=480, height=480)
    dev.off()
    
    max_interval <- which.max(time_series)
    names(max_interval)
}

imputMissingValues <- function()
{
    openFile()
    
    print(sum(is.na(myData)))
    
    myDataFilled <- fillData(myData)
    #print(myDataFilled)
    
    stepsTotal <- summarise(group_by(myDataFilled, date), sum(steps, na.rm=TRUE))
    names(stepsTotal) <- c("date", "sumSteps")
    
    hist(stepsTotal$sumSteps)
    
    print(mean(stepsTotal$sumSteps))
    print(median(stepsTotal$sumSteps))
}

fillData <- function(FUN=mean)
{
    stepsAvg <- summarise(group_by(myData, interval), mean(steps, na.rm=TRUE))
    
    names(stepsAvg) <- c("interval", "steps")
    
    filled <- numeric()
    for (i in 1:nrow(myData)) {
        obs <- myData[i, ]
        if (is.na(obs$steps)) {
            steps <- subset(stepsAvg, interval == obs$interval)$steps
        } else {
            steps <- obs$steps
        }
        filled <- c(filled, steps)
    }
    
    myDataFilled <- myData
    myDataFilled$steps <- filled
    
    myDataFilled
}

analyzeDayType<-function()
{
    library(lattice)
    
    openFile()
    myDataFilled <- fillData()
    myDataFilled$date <- as.Date(myDataFilled$date)
    day <- weekdays(myDataFilled$date)
    dayLevel <- vector()
    
    for(i in 1:nrow(myDataFilled))
    {
        if(day[i] == "Saturday" | day[i] == "Sunday")
        {
            dayLevel[i] <- "Weekend"
        }
        else
            dayLevel[i] <- "Weekday"
    }
    
    myDataFilled$dayLevel <- as.factor(dayLevel)
    
    stepsByDay <- aggregate(steps ~ interval + dayLevel, data = myDataFilled, mean)
    names(stepsByDay) <- c("interval", "dayLevel", "steps")
    
    xyplot(steps ~ interval | dayLevel, stepsByDay, type = "l", layout = c(1, 2), 
           xlab = "Interval", ylab = "Number of steps")
    
}
```
