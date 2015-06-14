---
title: "PA1_template.Rmd"
author: "DanielG"
date: "June 11, 2015"
output: html_document
---

# Reproducible Research: Assigment 1

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

## Loading data into R 

```r
dat <-  read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

- Calculate the total number of steps taken per day


```r
aggdat <- aggregate(dat$steps, by=list(dat$date), FUN=sum)
```

- Histogram of the total number of steps taken each day


```r
hist(aggdat$x, main="Histogram: Total Number of Steps by Day", xlab="Steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

- Calculate and report the mean and median of the total number of steps taken per day

```r
median(aggdat$x, na.rm = TRUE)
```

```
## [1] 10765
```

```r
mean(aggdat$x, na.rm = TRUE)
```

```
## [1] 10766.19
```


## What is the average daily activity pattern?

- Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
intdat <- aggregate(dat$steps, by=list(dat$interval), FUN=mean, na.rm=TRUE)
names(intdat) <- c("Interval", "Avg_steps")
plot(intdat, type = "l", xlab="Interval", ylab = "Steps", 
     main="Average number of steps taken by interval, averaged across all days")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

- Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?



```r
intdat[intdat$Avg_steps==max(intdat$Avg_steps), ]
```

```
##     Interval Avg_steps
## 104      835  206.1698
```

## Imputing missing values

- Calculate and report the total number of missing values in the dataset 


```r
sum(is.na(dat$steps))
```

```
## [1] 2304
```

- Impute missing values in the variable steps using the mean by Interval. 


```r
for(v in unique(dat$interval)){
        mean(dat[dat$interval==v, "steps"], na.rm=TRUE)
}
```

- Create a new dataset that is equal to the original dataset but with the missing data filled in. Using the dataframe with mean steps by inteval replace missign values with the average number of steps by interval.


```r
datimp <- merge(dat, intdat, by.x="interval", by.y="Interval", all=TRUE)
datimp$steps2 <- ifelse(is.na(datimp$steps)==TRUE, datimp$Avg_steps, datimp$steps) 
```

- Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
aggdate_imp <- aggregate(datimp$steps2, by=list(datimp$date), FUN = sum)
hist(aggdate_imp, main="Histogram: Total Number of Steps by Day", xlab="Steps", 
     sub="(Missing values imputed)")
```

```
## Error in hist.default(aggdate_imp, main = "Histogram: Total Number of Steps by Day", : 'x' must be numeric
```

```r
mean(aggdate_imp$x)
```

```
## [1] 10766.19
```

```r
median(aggdate_imp$x)
```

```
## [1] 10766.19
```

Because I used the average to imput missing values the mean does not change. On the other hand the median did changed.

## Are there differences in activity patterns between weekdays and weekends?

- Create a new factor variable in the dataset with two levels ‚Äì ‚Äúweekday‚Äù and ‚Äúweekend‚Äù indicating whether a given date is a weekday or weekend day.


```r
datimp$weekday <- weekdays(as.Date(datimp$date))
datimp$weekday <- ifelse( (datimp$weekday=="Saturday" | datimp$weekday=="Sunday"), 1, 0 )
datimp$weekday <- factor(datimp$weekday, labels=c("weekday", "weekend"))
```

- Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
aggimp_week <- aggregate(datimp$steps2, 
                         by=list(datimp$interval, datimp$weekday), 
                         FUN=mean, na.rm=TRUE)
names(aggimp_week) <- c("interval", "weekday", "steps")

xyplot(steps ~ interval | weekday, data=aggimp_week, layout=c(1,2), type = "l")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 






