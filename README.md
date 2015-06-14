# Reproducible Research: Assigment 1

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

## Loading data into R 
```{r, echo=TRUE}
dat <-  read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

- Calculate the total number of steps taken per day

```{r, echo=TRUE}
aggdat <- aggregate(dat$steps, by=list(dat$date), FUN=sum)
```

- Histogram of the total number of steps taken each day

```{r, echo=TRUE}
hist(aggdat$x, main="Histogram: Total Number of Steps by Day", xlab="Steps")
```

- Calculate and report the mean and median of the total number of steps taken per day
```{r, echo=TRUE}
median(aggdat$x, na.rm = TRUE)
mean(aggdat$x, na.rm = TRUE)
```


## What is the average daily activity pattern?

- Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r, echo=TRUE}
intdat <- aggregate(dat$steps, by=list(dat$interval), FUN=mean, na.rm=TRUE)
names(intdat) <- c("Interval", "Avg_steps")
plot(intdat, type = "l", xlab="Interval", ylab = "Steps", 
     main="Average number of steps taken by interval, averaged across all days")
```

- Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```{r, echo=TRUE}
intdat[intdat$Avg_steps==max(intdat$Avg_steps), ]
```

## Imputing missing values

- Calculate and report the total number of missing values in the dataset 

```{r, echo=TRUE}
sum(is.na(dat$steps))
```

- Impute missing values in the variable steps using the mean by Interval. 

```{r, echo=TRUE}
for(v in unique(dat$interval)){
        mean(dat[dat$interval==v, "steps"], na.rm=TRUE)
}
```

- Create a new dataset that is equal to the original dataset but with the missing data filled in. Using the dataframe with mean steps by inteval replace missign values with the average number of steps by interval.

```{r, echo=TRUE}
datimp <- merge(dat, intdat, by.x="interval", by.y="Interval", all=TRUE)
datimp$steps2 <- ifelse(is.na(datimp$steps)==TRUE, datimp$Avg_steps, datimp$steps) 
```

- Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r, echo=TRUE}
aaggimp <- aggregate(datimp$steps, by=list(datimp$date), FUN=sum)
hist(aggimp$x, main="Histogram: Total Number of Steps by Day", xlab="Steps", 
     sub="(Missing values imputed)")
mean(aggimp$x)
median(aggimp$x)
```

Because I used the average to imput missing values the mean does not change. On the other hand the median did changed.

## Are there differences in activity patterns between weekdays and weekends?

- Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```{r, echo=TRUE}
datimp$weekday <- weekdays(as.Date(datimp$date))
datimp$weekday <- ifelse( (datimp$weekday=="Saturday" | datimp$weekday=="Sunday"), 1, 0 )
datimp$weekday <- factor(datimp$weekday, labels=c("weekday", "weekend"))
```

- Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```{r, echo=TRUE}
aggimp_week <- aggregate(datimp$steps2, 
                         by=list(datimp$interval, datimp$weekday), 
                         FUN=mean, na.rm=TRUE)
names(aggimp_week) <- c("interval", "weekday", "steps")

xyplot(steps ~ interval | weekday, data=aggimp_week, layout=c(1,2), type = "l")
```






