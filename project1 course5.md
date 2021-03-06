---
title: "Reproducible Research: Peer Assessment 1"
author: "Surekha Padmam"
date: "may 10,2020"
output: html_document
---



Setting global option to set echo true   and to turn off warnings:

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
knitr::opts_chunk$set(warning=FALSE)
```

## Loading and preprocessing the data

```{r}
library(ggplot2)
activity<-read.csv("activity.csv",sep=",")
activity$date<-as.POSIXct(activity$date,"%Y-%m-%d")
weekday<-weekdays(activity$date)
activity<-cbind(activity,weekday)
summary(activity)
```



## What is mean total number of steps taken per day?
```{r}
activity_total_steps <- with(activity, aggregate(steps, by = list(date), FUN = sum, na.rm = TRUE))
names(activity_total_steps) <- c("date", "steps")
hist(activity_total_steps$steps, main = "Total number of steps taken per day", xlab = "Total steps taken per day", col = "darkblue", ylim = c(0,20), breaks = seq(0,25000, by=2500))
```
finding mean and median:
```{r}
mean(activity_total_steps$steps)
median(activity_total_steps$steps)
```


## What is the average daily activity pattern?
```{r}
average_daily_activity <- aggregate(activity$steps, by=list(activity$interval), FUN=mean, na.rm=TRUE)
names(average_daily_activity) <- c("interval", "mean")
plot(average_daily_activity$interval, average_daily_activity$mean, type = "l", col="darkblue", lwd = 2, xlab="Interval", ylab="Average number of steps", main="Average number of steps per intervals")
```
average daily activity is:
```{r}
average_daily_activity[which.max(average_daily_activity$mean), ]$interval
```

## Imputing missing values
```{r}
sum(is.na(activity$steps))
imputed_steps <- average_daily_activity$mean[match(activity$interval,average_daily_activity$interval)]
activity_imputed <- transform(activity, steps = ifelse(is.na(activity$steps), yes = imputed_steps, no = activity$steps))
total_steps_imputed <- aggregate(steps ~ date, activity_imputed, sum)
names(total_steps_imputed) <- c("date", "daily_steps")
hist(total_steps_imputed$daily_steps, col = "darkblue", xlab = "Total steps per day", ylim = c(0,30), main = "Total number of steps taken each day", breaks = seq(0,25000,by=2500))
```
finding mean and  median of steps taken per day
```{r}
mean(total_steps_imputed$daily_steps)
median(total_steps_imputed$daily_steps)
```

## Are there differences in activity patterns between weekdays and weekends?

```{r}
activity$date <- as.Date(strptime(activity$date, format="%Y-%m-%d"))
activity$datetype <- sapply(activity$date, function(x) {
        if (weekdays(x) == "S�bado" | weekdays(x) =="Domingo") 
                {y <- "Weekend"} else 
                {y <- "Weekday"}
                y
        })
        
 activity_by_date <- aggregate(steps~interval + datetype, activity, mean, na.rm = TRUE)
plot<- ggplot(activity_by_date, aes(x = interval , y = steps, color = datetype)) +
       geom_line() +
       labs(title = "Average daily steps by type of date", x = "Interval", y = "Average number of steps") +
       facet_wrap(~datetype, ncol = 1, nrow=2)
print(plot)
       
```