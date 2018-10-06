---
title: "Reproducible Research: Peer Assessment 1"
author: David Robinson
date: 06/10/2018
output: 
  html_document:
    keep_md: true
---



## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Packages Required
In order for the Markdown code to run successfully you will need to have the following package(s) installed:

- ggplot2

We will also set the global options value `scipen` to avoid output values in scientific format in the document.

```r
#LOAD PACKAGES
library(ggplot2)

options(scipen=999)
```


## Data

The data for this assignment can be downloaded from the course web site:

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)
* **date**: The date on which the measurement was taken in YYYY-MM-DD format
* **interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

You can also fork / clone the [Github Repository For this assignment](http://github.com/rdpeng/RepData_PeerAssessment1) to obtain data for this analysis.

## Loading and preprocessing the data
I have assumed that if running the markdown document the user has downloaded the data from either of the links above and set their working directory in R to be the folder which contains the downloaded, unzipped *"activity.csv"* file. This can be done using the `setwd()` function. See `?setwd()` for help on this function.

- Load the data using read.csv


```r
#LOAD ACTIVITY DATA
activity <- read.csv("activity.csv", stringsAsFactors = F)
```

- using `str()` we can see that the format of the date field is character. 

```r
#LOAD ACTIVITY DATA
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```
- To enable analysis later, we should convert the date field to a date format using the `as.POSIXct()` function. We run `str()` again to check this is successful. 

```r
#CONVERT DATE FIELD TO DATE, CURRENLY STRING
activity$date <- as.POSIXct(activity$date, format = "%Y-%m-%d")
#CHECK FORMAT HAS WORKED
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : POSIXct, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## What is mean total number of steps taken per day?
A histogram of the daily number of steps will give us an overview of the distribution of number of steps. As per the instructions in the project at this stage we've been asked to omit the NA entries. 

First create a data frame using the `aggregate()` function, and rename the columns so they are intuative. I have also run the min and max function on the total number of daily steps which will help us set up our breaks when plotting the histogram

```r
sum_by_date <- with(activity, aggregate(steps, by = list(date), FUN = sum, na.rm = T ))
#Names not intuative, so rename
names(sum_by_date) <- c("date", "total_steps")

min(sum_by_date$total_steps)
```

```
## [1] 0
```

```r
max(sum_by_date$total_steps)
```

```
## [1] 21194
```

Next we plot this histogram, I've used package `ggplot2()` for this. You may need to install this package as instructed earlier in the document. I have also plotted lines displaying both the <span style="color:red">mean</span> and the <span style="color:orange">median</span> values of the daily number of steps. These are also computed below so you can see their values. 

```r
g <- ggplot(data = sum_by_date, aes(sum_by_date$total_steps))
g + geom_histogram(breaks=seq(0, to =22000, 2000), col = "grey50", fill = "navy blue") +
    ggtitle("Distribution of Total Daily Steps", subtitle = "NA Values Removed") +
    xlab("Daily Total Number of Steps") +
    ylab("Frequency") +
    geom_vline(aes(xintercept = mean(sum_by_date$total_steps)),col='red',size=2) +
    geom_vline(aes(xintercept = median(sum_by_date$total_steps)),col='orange',size=2) +
    geom_text(aes(label="Mean",y=15,x=mean(sum_by_date$total_steps)),
            hjust=1.25,col='red',size=2.5) +
    geom_text(aes(label="Median",y=15,x=median(sum_by_date$total_steps)),
            hjust=-0.25,col='orange',size=2.5)
```

![](PA1_template_files/figure-html/hist_plot-1.png)<!-- -->

Calculating the mean and median explicitly we can see:

```r
mean(sum_by_date$total_steps)
```

```
## [1] 9354.23
```

```r
median(sum_by_date$total_steps)
```

```
## [1] 10395
```

So mean number of steps per day is **9354.23** and median number of steps per day is **10395** . 

## What is the average daily activity pattern?
In order to look at the average daily activity pattern, we'll first look at a time series plot of the 5 minute interval (x-axis) and the average number of steps taken, average across all days (y-axis)  

As a start point we need to aggregate the data by interval rather than by date as we did in the previous section, again we will use the `aggregate()` function. We will then create a very simple plot using `ggplot2()` package


```r
sum_by_int <- with(activity, aggregate(steps, by = list(interval), FUN = sum, na.rm = T ))
#Names not intuative, so rename
names(sum_by_int) <- c("interval", "daily_steps")

#PLOT
g <- ggplot(data = sum_by_int, aes(x= sum_by_int$interval, y = sum_by_int$daily_steps))
g + geom_line(color = "navy blue", show.legend = F) +
    ggtitle("Average number of Steps taken per 5 minute Interval", subtitle = "NA Values Removed") +
    xlab("Interval [5 minutes]") +
    ylab("Average Daily Steps") +
    geom_vline(aes(xintercept = sum_by_int[which(sum_by_int$daily_steps == max(sum_by_int$daily_steps)), 1],col='red'), show.legend = F) +
    geom_text(aes(label=paste0("Max Interval= ", sum_by_int[which(sum_by_int$daily_steps == max(sum_by_int$daily_steps)), 1] ),y=max(sum_by_int$daily_steps),x=sum_by_int[which(sum_by_int$daily_steps == max(sum_by_int$daily_steps)), 1]),
            hjust=1.05,col='red',size=4)
```

![](PA1_template_files/figure-html/steps_interval-1.png)<!-- -->

The interval number with the highest average daily number of steps is **835**.
This can be computed using the code

```r
sum_by_int[which(sum_by_int$daily_steps == max(sum_by_int$daily_steps)), 1]
```

```
## [1] 835
```

## Imputing missing values
One of the problems posed in this project is to dealing with missing values where data is coded NA. This may introduce some bias into the analysis.

First we should check how many missing values we have in the data and what percentage that is of the total data set.

```r
#NUMBER MISSING
sum(is.na(activity$steps))
```

```
## [1] 2304
```

```r
#PERCENTAGE OF TOTAL DATASET MISSING
round(sum(is.na(activity$steps)) / nrow(activity)*100,2)
```

```
## [1] 13.11
```
In total there are **2304** missing values or **13.11%** of the data. This is problematic and could bias any findings.

Now we will devise a simple strategy to fill in the blanks in the dataset. I am opting to replace NAs with the mean for the 5 minute interval from which the missing data originates from. While this is a simple approach it seems reasonable to assume that the individual wearing the tracker has a similar daily routine.

Create a copy of the original dataset, so as not to overwrite original data. Then we use the `ave()` function in conjunction with `replace()` to replace each NA value with the mean value for its corresponding interval. Finally plot the histogram of the data with NAs repalaced


```r
#CREATE COPY OF ORIGINAL ACTIVITY DATASET
activity_na <- activity
#USE AVE() TO REPLACE NA's WITH MEAN VALUE FOR SUBSET
activity_na$steps <- with(activity_na, ave(steps, interval, FUN = function(x) replace(x, is.na(x), mean(x, na.rm=TRUE))))
#NOW CREATE NEW AGGREGATED VALUES FOR PLOT
new_sum_by_date <- with(activity_na, aggregate(steps, by = list(date), FUN = sum, na.rm = T ))
#Names not intuative, so rename
names(new_sum_by_date) <- c("date", "total_steps")
#PLOT
g <- ggplot(data = new_sum_by_date, aes(new_sum_by_date$total_steps))
g + geom_histogram(breaks=seq(0, to =22000, 2000), col = "grey50", fill = "navy blue") +
  ggtitle("Distribution of Total Daily Steps", subtitle = "NA Values Replaced with Interval average") +
  xlab("Daily Total Number of Steps") +
  ylab("Frequency") +
  geom_vline(aes(xintercept = mean(new_sum_by_date$total_steps)),col='red',size=2) +
  geom_vline(aes(xintercept = median(new_sum_by_date$total_steps)),col='orange',size=2) +
  geom_text(aes(label=paste0("Mean = ", round(mean(new_sum_by_date$total_steps),0)),y=15,x=mean(new_sum_by_date$total_steps)),
            hjust=1.25,col='red',size=2.5) +
  geom_text(aes(label=paste0("Median", round(median(new_sum_by_date$total_steps),0)),y=15,x=median(new_sum_by_date$total_steps)),
            hjust=-0.25,col='orange',size=2.5)
```

![](PA1_template_files/figure-html/replace_na-1.png)<!-- -->

The mean is 10766 and the median is 10766 these are computed as follows:

```r
mean(new_sum_by_date$total_steps)
```

```
## [1] 10766.19
```

```r
median(new_sum_by_date$total_steps)
```

```
## [1] 10766.19
```
The mean value has increased from **9354** to **10766** while the median has increased from **10395** to **10766**. Since the mean is equal to the median and the median is a non integer value, it implies that the method of replacing NAs has left the mean and median equal ie. median value is an average value, this makes intuative sense since earlier we found that **13.11%** of the data was NAs. Also since the mean has increased this implied that the NA values occurred in intervals where typically the wearer of the tracker was more active.


## Are there differences in activity patterns between weekdays and weekends?
A further topic of interest is the average number of steps taken in 5 minute intervals across weekdays and weekends, to see if these differ. For this part of the analysis the `weekdays()` function will be used, and we will use the dataset with the NAs replaced with mean of the interval data. 

The first step is to create a new factor to identify if the record is a weekday or weekend. 

```r
activity_na$day_type <- ifelse(weekdays(activity_na$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")
#CHECK TO SEE IF SPLIT MAKES SENSE
table(activity_na$day_type)
```

```
## 
## weekday weekend 
##   12960    4608
```

Now we will make a panel plot using `ggplot2` to show the time series plot of the 5 minute intervals (x-axis) and average number of steps taken across all weekday days and weekend days (y-axis). We could use lattice here, which would probably be easier, but I'm 100% committed to ggplot! 

To prepare the data for the panel plot we will use the `aggregate()` function, however we want to aggregate by interval and day_type which indicate if we are looking at a weekend day or weekday day. The function we will be applying is the mean. In addition we will create an additional dataframe `avg_by_day_type` which taeks the mean number of steps by day_type only, this creates a 2x2 data frame which is later used for labelling the plots with the mean number of steps taken per interval to ease comparison. 


```r
#CREATE AGGREGATED DATASET & SUMMARY DATASET TO ALLOW LABELLING
sum_int_wday <- aggregate(steps ~ interval + day_type, data = activity_na, mean)
avg_by_day_type <- aggregate(steps ~ day_type, data = activity_na, mean)
#NOW PLOT
g <- ggplot(sum_int_wday, aes(interval, steps))
g+ geom_line(color = "navy blue") +
   facet_grid(day_type ~ .) +
   ggtitle("Average number of Steps taken per 5 minute Interval split by Weekday | Weekend", subtitle = "NA values replaced") +
   xlab("Interval [5 minutes]") +
   ylab("Average Daily Steps")  +
   geom_hline(data = avg_by_day_type, aes(yintercept = steps), size = 0.5, color = "red", linetype = "longdash")+
   geom_text(data = avg_by_day_type, mapping = aes(x=-Inf, y = avg_by_day_type$steps, label = paste0("mean = ", round(avg_by_day_type$steps,2))),
              hjust = -0.3, vjust = -0.3, color = "red", size = 3)
```

![](PA1_template_files/figure-html/panel_plot-1.png)<!-- -->

From this panel plot we can compare the activity at weekdays (top plot) and weekends. We have annotated the graphs with the mean, we can see here that the average steps per interval at the weekend is higher than during the remainder of the weeks. However this doesn't tell the whole story as if we look at more detail at the plots, we can see that during the weekends activity starts later in the day (since time increases from left to right), and during the remainder of the day the activity levels are much higher than during the remainder of the week. In conclusion it appears the user of this tracking device is more active at the weekend than during the week. 

