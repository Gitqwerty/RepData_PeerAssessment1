# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
The following code reads the activity.csv file located in the zip file. Note that it does not unzip the zip contents. It prepares the data by specifying the column classes, most importantly the Date class.

```r
act <- read.csv(unz("activity.zip", "activity.csv"), colClasses = c("numeric", "Date", "numeric"))
```


## What is mean total number of steps taken per day?

```r
total <- aggregate(act$steps, by=list(act$date), FUN=sum)

hist(total$x, xlab="Number of steps in a day", main="Total number of steps taken each day", labels=TRUE)
```

![plot of chunk TotalStepsHistogram](figure/TotalStepsHistogram.png) 

```r
mean(total$x, na.rm=TRUE)
```

```
## [1] 10766
```

```r
median(total$x, na.rm=TRUE)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

```r
inter <- aggregate(act$steps, by=list(act$interval), FUN=mean, na.rm=TRUE)
plot(inter$Group.1, inter$x, type="l", xlab="Time Interval", ylab="Average Number of Steps in Interval", main="Average Steps per Time Interval")
```

![plot of chunk AverageDailyStepsByInterval](figure/AverageDailyStepsByInterval.png) 

The following finds the interval, which on average, has the maximum number of steps.

```r
inter[which(inter$x==max(inter$x)),][1]
```

```
##     Group.1
## 104     835
```
The answer is the interval 835 (104 is the row in the "inter" table, and Group.1 is the column name for the intervals).


## Imputing missing values

The following stores the rows with missing values into the object "missingRows", it has 3 columns that correspond to the dataset:

```r
missingRows <- act[!complete.cases(act),]
dim(missingRows)
```

```
## [1] 2304    3
```
The missingRows table has 2304 rows. Which means 2304 rows have missing values.
summary() can also be used to find missing values.


The strategy used to fill in the missing data is to use the average steps for the time interval the missing value is in. A new dataset, called "filled", is created which has the missing data filled in.

```r
filled <- act
for (i in 1:length(filled$steps)) {
    if (is.na(filled$steps[i])) {
        filled$steps[i] = inter$x[which(inter$Group.1==filled$interval[i])]
    }
}
```

The following creates a new histogram that includes the filled in data and finds the mean and median

```r
total2 <- aggregate(filled$steps, by=list(filled$date), FUN=sum)
hist(total2$x, xlab="Number of steps in a day", main="Total number of steps taken each day", labels=TRUE)
```

![plot of chunk TotalStepsHistogramFilled](figure/TotalStepsHistogramFilled.png) 

```r
mean(total2$x)
```

```
## [1] 10766
```

```r
median(total2$x)
```

```
## [1] 10766
```
The mean value is the same as from the first part of the assignment. But the median value has changed from 10765 to 10766. Adding missing data increases the total daily number of steps. But only for the following 8 days:

```r
unique(missingRows$date)
```

```
## [1] "2012-10-01" "2012-10-08" "2012-11-01" "2012-11-04" "2012-11-09"
## [6] "2012-11-10" "2012-11-14" "2012-11-30"
```


## Are there differences in activity patterns between weekdays and weekends?

The following creates a new variable in the data set, at first it just lists the day of the week (Monday, Tuesday, etc.)

```r
actDay <- cbind(filled, weekdays(filled$date))
colnames(actDay) = c("steps","date","interval","day")
```

The following converts the names Monday, Tuesday, etc. to "weekday" or "weekend" factors using the levels function

```r
levels(actDay$day) <- list(weekend="Saturday", weekend="Sunday", weekday="Monday", weekday="Tuesday",weekday="Wednesday",weekday="Thursday",weekday="Friday")
```

The following finds the average steps per day, and then plots the results, one plot for weekend and the other for weekdays.

```r
internew <- aggregate(actDay$steps, by=list(actDay$interval, actDay$day), FUN=mean)

par(mfrow=c(2,1))
plot(internew$Group.1[which(internew$Group.2=="weekend")], internew$x[which(internew$Group.2=="weekend")], type="l", xlab="Time Interval", ylab="Average Number of Steps in Interval", main="Weekend Average Steps per Time Interval")
plot(internew$Group.1[which(internew$Group.2=="weekday")], internew$x[which(internew$Group.2=="weekday")], type="l", xlab="Time Interval", ylab="Average Number of Steps in Interval", main="Weekday Average Steps per Time Interval")
```

![plot of chunk WeekendvsWeekdayMeanSteps](figure/WeekendvsWeekdayMeanSteps.png) 
By examining the charts, it does seem there is more activity on the weekends.
