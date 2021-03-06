Reproducible Research: Peer Assessment 1
=========================================


## Loading and preprocessing the data
Assuming you havea 'activity.zip' in your working directory, following R code will unzip the file and load it as dataset.

data in date column is then transformed as Date class.

```r
unzip("activity.zip")
dataset <- read.csv("activity.csv")
dataset$date <- as.Date(dataset$date, "%Y-%m-%d")
```


## What is mean total number of steps taken per day?
### Creating histogram 
Take a look at histogram of the total number of steps taken each day.  
In following R code, I have grouped each day in date column, then applied that to calcurate sum of steps taken by each day.   
(I have decided to show histogram in 20 breaks)


```r
date_group <- factor(dataset$date)
StepsByDay_sum <- tapply(dataset$steps, date_group, sum)
hist(StepsByDay_sum, xlab = "Total steps by day", ylab = "Frequency (Days)", 
    main = "Histogram : Number of daily steps", breaks = 20)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 


### Getting mean and median
Following R code will calcurate mean and median of total number of steps taken per day.
* Mean of total number of steps taken per day

```r
StepsByDay_mean <- mean(StepsByDay_sum, na.rm = T)
StepsByDay_mean
```

```
## [1] 10766
```


* Median of total number of steps taken per day

```r
StepsByDay_median <- median(StepsByDay_sum, na.rm = T)
StepsByDay_median
```

```
## [1] 10765
```



## What is the average daily activity pattern?
### Creating a time series plot
In following R code, I have grouped each interval in interval column, then applied that to calcurate mean of steps taken in each interval across all days.  

```r
interval_group <- factor(dataset$interval)
StepsByInterval_mean <- tapply(dataset$steps, interval_group, mean, na.rm = T)
```


Then, using plot function, create a time series plot which shows the daily activity pattern.

```r
plot(StepsByInterval_mean, type = "l", xlab = "5 minute interval", ylab = "Average number of steps", 
    main = "Average number of steps ( by each 5 minute interval)")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 


### Which 5 minute interval contains the maximum number of steps?
Following R code find the interval which has the maximum number of average steps across all days.

```r
MaxStepsInterval <- dataset$interval[which.max(StepsByInterval_mean)]
MaxStepsInterval
```

```
## [1] 835
```

The intervavl with macimum number is : 835


## Imputing missing values
### Report the total number of missing values in the dataset

There are 2304 missing values. Following is how I get the number.

```r
sum(is.na(dataset$steps))
```

```
## [1] 2304
```


### Replace the missing values
Following R code replace the NA values with the average number of steps for the corresponding interval.  

1. Create data frame of average steps by 5 minute interval  
2. Merge original dataset and newly created average steps data frame to create new dataframe "dataset2"  
3. Replace NA values in steps column with value in StepsByInterval_mean column of collesponding row.  
4. delete StepsByInterval_mean column as it is not needed anymore  

```r
StepsByInterval_mean_DF <- as.data.frame(StepsByInterval_mean)

dataset2 = merge(dataset, StepsByInterval_mean_DF, by.x = "interval", by.y = "row.names")

dataset2$steps[is.na(dataset2$steps)] = dataset2$StepsByInterval_mean[is.na(dataset2$steps)]
dataset2$date <- as.Date(dataset2$date, "%Y-%m-%d")

dataset2$StepsByInterval_mean = NULL
```


### Using new dataset, make a histogram of the total number of steps taken each day
I have grouped each date in date column, then applied that to calcurate sum of steps taken in each date, then plot histgram.

```r
StepsByDay_sum2 <- tapply(dataset2$steps, date_group, sum)
hist(StepsByDay_sum2, xlab = "Total steps by day", ylab = "Frequency (Days)", 
    main = "Histogram : Number of daily steps", breaks = 20)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 


### Using new dataset, calculate and report the mean and median total number of steps taken per day. 

Following R code will calcurate mean and median of total number of steps taken per day.
* Mean of total number of steps taken per day

```r
StepsByDay_mean2 <- mean(StepsByDay_sum2, na.rm = T)
StepsByDay_mean2
```

```
## [1] 10766
```


* Median of total number of steps taken per day

```r
StepsByDay_median2 <- median(StepsByDay_sum2, na.rm = T)
StepsByDay_median2
```

```
## [1] 10352
```


Values does differ from the estimates from the first part of the assignment. Relacing NA values with mean created more days with 0 steps taken, also created days with more than 30000 steps which did not apper in the first part of the assignment.


## Are there differences in activity patterns between weekdays and weekends?

### Create a new factor variable in the dataset with two levels – “weekday” and “weekend”
Following R code adds new 'day' column to dataset 2

1. weekdays() function computes date value of corresponding row and adds day name
2. Based on condition of day value of each row replace the value with "weekday" or "weekend"


```r
dataset2$day <- weekdays(dataset2$date)
dataset2$day[dataset2$day == "Saturday" | dataset2$day == "Saturday"] <- "weekend"
dataset2$day[dataset2$day == "Monday" | dataset2$day == "Tuesday" | dataset2$day == 
    "Wednesday" | dataset2$day == "Thursday" | dataset2$day == "Friday"] <- "weekday"
```


### Make a panel plot containing a time series plot of the 5-minute intervaland the average number of steps taken, averaged across all weekday days or weekend days
For this task, first calculate the average steps in each 5 minute interval for weekdays and weekends.


```r
Average_StepsByInterval_Weekday = tapply(subset(dataset2, day == "weekday")$steps, 
    subset(dataset2, day == "weekday")$interval, mean, na.rm = TRUE)
Average_StepsByInterval_Weekend = tapply(subset(dataset2, day == "weekend")$steps, 
    subset(dataset2, day == "weekend")$interval, mean, na.rm = TRUE)
```


Then using base plotting system, create panel plot.


```r
par(mfrow = c(2, 1))
plot(Average_StepsByInterval_Weekday, type = "l", xlab = "5 minute interval", 
    ylab = "Average number of steps", main = "weekdays")
plot(Average_StepsByInterval_Weekend, type = "l", xlab = "5 minute interva", 
    ylab = "Average number of steps", main = "weekend")
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15.png) 

