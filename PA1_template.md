---
#Reproducible Research-Peer Assessment 1"
---
##**Loading and preprocessing the data**

1. Load the data and check the colums's classes and names


```r
setwd("C:/Users/toshiba/Documents")
data<-read.csv("activity.csv")
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

##**What is mean total number of steps taken per day?**

1. Calculate the total number of steps taken per day


```r
sum_steps<-tapply(data$steps,data$date,sum)
```

2. Make a histogram of the total number of steps taken each day

```r
hist(sum_steps)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day

```r
#mean
mean_steps<-mean(sum_steps,na.rm=TRUE)
print(mean_steps)
```

```
## [1] 10766.19
```

```r
#median
median_steps<-median(sum_steps,na.rm=TRUE)
print(median_steps)
```

```
## [1] 10765
```

##**What is the average daily activity pattern?**  
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)  
Use zoo object to create time series, "interval" is the x-axis variable, "avg_steps" is the y-axis variable.

```r
library(zoo)
#extract the levels of the factor variable "interval" as x
interval<-as.numeric(levels(as.factor(data$interval)))
#calculate the mean of average number of steps taken across all days as y
avg<-function(x)mean(x,na.rm=TRUE)
avg_steps<-tapply(data$steps,as.factor(data$interval),avg)
#plot x and y
ts<-zoo(avg_steps,interval)
plot(ts,type="l",xlab="Interval",ylab="Number of steps")
title("Time Series of Interval and Number of steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
library(dplyr)
#arrange the dataset in a descending order so that the first element of the interval contains the maximum number of steps
ds<-as.data.frame(cbind(interval=as.numeric(interval),avg=as.numeric(avg_steps)))
arg<-arrange(ds,desc(avg_steps))
arg$interval[1]
```

```
## [1] 835
```

##**Imputing missing values**

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
#(rows of the original data) - (rows of the data has NAs removed) = (num of NAs)
data2<-na.omit(data)
nrow(data)-nrow(data2)
```

```
## [1] 2304
```

2+3. when find out only the "steps"colum contains missing values(**using the is.na()function,TRUE=having NAs,FALSE=do not have NAs**), use the **mean** of colum "steps" to fill in the missing values and Create a new dataset that is equal to the original dataset but with the missing data filled in, and then check if the dataset has any missing value.

```r
#check which colum has NAs
table(is.na(data$steps))
```

```
## 
## FALSE  TRUE 
## 15264  2304
```

```r
table(is.na(data$date))
```

```
## 
## FALSE 
## 17568
```

```r
table(is.na(data$interval))
```

```
## 
## FALSE 
## 17568
```

```r
#fill in the NAs
data$steps[is.na(data$steps)]<-mean(as.numeric(data$steps),na.rm=TRUE)
#check if there is any NAs in the new data
table(is.na(data))
```

```
## 
## FALSE 
## 52704
```

4. Make a histogram of the total number of steps taken each day(`sum_steps2`)and Calculate and report the mean(`mean_steps2`) and median(`median_steps2`) total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
sum_steps2<-tapply(data$steps,data$date,sum)
hist(sum_steps2)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

```r
mean_steps2<-mean(sum_steps2)
print(mean_steps2)
```

```
## [1] 10766.19
```

```r
median_steps2<-median(sum_steps2)
print(median_steps2)
```

```
## [1] 10766.19
```
The differences are:  
1, the frequency of total number of steps taken each day is changed because missing values are filled in.  

2, the median of the variable is changed because I use the mean to fill in the missing values.  

##**Are there differences in activity patterns between weekdays and weekends?**  

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
(星期六=Saturday,星期日=Sunday)  


```r
#create week as the factor indicating whether it is weekday or weekend,initialize it with weekday.
w<-rep("weekday",nrow(data))
d<-mutate(data,week=factor(w,levels=c("weekday","weekend")))
#if the date of an observation is weekend,return the index of it and use the index to set its week factor to weekend
index<-which(as.character(weekdays(as.Date(d$date)))%in%c("星期六","星期日"))
d$week[index]<-"weekend"
#such result indicates that we have 12960 observations that are weekdays and 4608 of which are weekends
table(d$week)
```

```
## 
## weekday weekend 
##   12960    4608
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.  


```r
#split the original data into 2 parts:weekday and weekend
weekday_group<-split(d,d$week)$weekday
weekend_group<-split(d,d$week)$weekend
#the variable "interval" is same to above,so we only have to change the number of steps, use avg_steps2 and avg_steps2 to represent it.
avg_steps1<-tapply(weekday_group$steps,as.factor(weekday_group$interval),mean)
avg_steps2<-tapply(weekend_group$steps,as.factor(weekend_group$interval),mean)
#put x and y into zoo object
ts1<-zoo(avg_steps1,weekday_group$interval)
```

```
## Warning in zoo(avg_steps1, weekday_group$interval): some methods for "zoo"
## objects do not work if the index entries in 'order.by' are not unique
```

```r
ts2<-zoo(avg_steps2,weekend_group$interval)
```

```
## Warning in zoo(avg_steps2, weekend_group$interval): some methods for "zoo"
## objects do not work if the index entries in 'order.by' are not unique
```

```r
#set the pattern of printing the plot and plot it
par(mfrow=c(2,1))
plot(ts1,type="l",xlab="Intervals",ylab="Number of steps")
title(main="weekday")
plot(ts2,type="l",xlab="Intervals",ylab="Number of steps")
title(main="weekend")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 
