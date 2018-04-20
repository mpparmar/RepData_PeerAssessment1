---
title: "Reproducible Research Project1"
output:
  html_document:
    keep_md: true
---



## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

## Loading and processing data
Working directory set to correct folder
Load data and read file 


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
activity <- read.csv("activity.csv")
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
summary(activity)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

```r
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

Process data into suitable format for analysis
We need to remove missing values. Create another dataset without missing values


```r
act.complete <- na.omit(activity)
```

## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.
1.Calculate the total number of steps taken per day


```r
  act.day <- group_by(act.complete,date)
  act.day <- summarize(act.day,steps = sum(steps))
  summary(act.day)
```

```
##          date        steps      
##  2012-10-02: 1   Min.   :   41  
##  2012-10-03: 1   1st Qu.: 8841  
##  2012-10-04: 1   Median :10765  
##  2012-10-05: 1   Mean   :10766  
##  2012-10-06: 1   3rd Qu.:13294  
##  2012-10-07: 1   Max.   :21194  
##  (Other)   :47
```
2.If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
  qplot(steps,data = act.day)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/plot graph-1.png)<!-- -->

3.Calculate and report the mean and median of the total number of steps taken per day
Use mean() and median() function

```r
  mean(act.day$steps)
```

```
## [1] 10766.19
```

```r
  median(act.day$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
  1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken,        averaged across all days (y-axis)


```r
  act.int <- group_by(act.complete,interval)
  act.int <- summarize(act.int,steps = mean(steps))
```

Plot avg. daily steps vs intervals

```r
  ggplot(act.int, aes(interval, steps)) + geom_line()
```

![](PA1_template_files/figure-html/avgDailyAct_plot-1.png)<!-- -->

  2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
  act.int[act.int$steps==max(act.int$steps),]
```

```
## # A tibble: 1 x 2
##   interval steps
##      <int> <dbl>
## 1      835   206
```

## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
The total number of rows with NAs is equal to the difference between the number of rows in the raw data and the number of rows in the data with only complete cases:


```r
  nrow(activity)-nrow(act.complete)
```

```
## [1] 2304
```

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
  names(act.int)[2] <- "mean.steps"
  act.impute <- merge(activity, act.int)
```

3.Create a new dataset that is equal to the original dataset but with the missing data filled in.

If steps is NA, I replace the value with the mean number of steps for the interval:


```r
  act.impute$steps[is.na(act.impute$steps)] <- act.impute$mean.steps[is.na(act.impute$steps)]
```

4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
  act.day.imp <- group_by(act.impute, date)
  act.day.imp <- summarize(act.day.imp, steps=sum(steps))
  qplot(steps, data=act.day.imp)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/PlotHist-1.png)<!-- -->

## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
I convert the date variable to the date class, then use the weekdays() function to generate the day of the week of each date. I create a binary factor to indicate the two weekend days:


```r
  act.impute$dayofweek <- weekdays(as.Date(act.impute$date))
  act.impute$weekend <-as.factor(act.impute$dayofweek=="Saturday"|act.impute$dayofweek=="Sunday")
  levels(act.impute$weekend) <- c("Weekday", "Weekend")
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
First I create separate data frames for weekends and weekdays:


```r
  act.weekday <- act.impute[act.impute$weekend=="Weekday",]
  act.weekend <- act.impute[act.impute$weekend=="Weekend",]
  
  act.int.weekday <- group_by(act.weekday, interval)
  act.int.weekday <- summarize(act.int.weekday, steps=mean(steps))
  act.int.weekday$weekend <- "Weekday"
  act.int.weekend <- group_by(act.weekend, interval)
  act.int.weekend <- summarize(act.int.weekend, steps=mean(steps))
  act.int.weekend$weekend <- "Weekend"
  act.int <- rbind(act.int.weekday, act.int.weekend)
  act.int$weekend <- as.factor(act.int$weekend)
  ggplot(act.int, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)
```

![](PA1_template_files/figure-html/plotgraph-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.

