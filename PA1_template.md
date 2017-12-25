    knitr::opts_chunk$set(echo = TRUE, error = FALSE, warning = FALSE, message = FALSE)

Introduction
------------

In this assignment, the activity data set is explorerd and analyzed.

Data loading and preprocessing
------------------------------

The csv file is loaded along with the formatting of time variables.

    activity <- read.csv("activity.csv",head=TRUE)
    activity$date <- as.Date(activity$date,format="%Y-%m-%d")
    head(activity,5)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20

    str(activity)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

Mean total number of steps taken per day
----------------------------------------

### Aggregate the total number of steps per day

To obtain the daily total steps, the step data is aggregated for each
day.

The new data set is stored in activity-daily.

    library(dplyr)
    activity_daily <- aggregate(steps~date,activity,sum,na.rm=TRUE)

### Histogram of total number of steps per day

Now the histogram of total number of steps per day is plotted.

    hist(activity_daily$steps,breaks = 50, main = "Total number of steps per day", xlab = "Steps")

<img src="figure-html/Hist-1.png" style="display: block; margin: auto;" />

### Mean and median of total number of steps per day

The mean and median of the total number of steps per day are calculated
as below:

    mean(activity_daily$steps)

    ## [1] 10766.19

    median(activity_daily$steps)

    ## [1] 10765

average daily activity pattern
------------------------------

Now we investigate the daily activity pattern.

To do this we first aggregate the number of steps for each 5 min over
all days.

    activity_5min <- aggregate(steps~interval,activity,sum,na.rm=TRUE)

### Time series plot

Here is a time series plot presenting the number of steps for each 5 min
over all days.

    plot(activity_5min$interval,activity_5min$steps,type="l",main="Number of steps for each 5 min", xlab="Time", ylab="Steps")

<img src="figure-html/TS1-1.png" style="display: block; margin: auto;" />

### The interval with the maximum number of steps

The interval that has the maximal number of steps on average is
calculated as below:

    activity_5min %>% subset(steps == max(steps))

    ##     interval steps
    ## 104      835 10927

So the interval 835 has the maximal average number of steps.

Imputing missing values
-----------------------

### Number of missing values

    nrow(activity %>% filter(is.na(steps))) 

    ## [1] 2304

### Median Imputation

The imputation technique adopted here is the median imputation for each
interval.

    steps_median <- activity %>% aggregate(steps~interval,data=.,median,na.rm=TRUE)
    activity$steps_impute <- ifelse(is.na(activity$steps),steps_median[which(activity$interval==steps_median$interval),"steps"],activity$steps)

### Histogram of total number of steps per day

    activity_daily_impute <- aggregate(steps_impute~date,activity,sum,na.rm=TRUE)
    hist(activity_daily_impute$steps_impute,breaks=50,main = "Total number of steps per day", xlab = "Steps")

<img src="figure-html/Hist2-1.png" style="display: block; margin: auto;" />

### Mean and median of total number of steps per day

The mean and median of the total number of steps per day after
imputation are calculated as below:

    mean(activity_daily_impute$steps_impute)

    ## [1] 10587.94

    median(activity_daily_impute$steps_impute)

    ## [1] 10682.5

In comparison with the values before imputation, the imputed mean and
median go down because of the imputed values regressing to the median.

### Difference between weekdays and weekends

Determine weekdays and weekends of the imputed data set

    activity <- activity %>% mutate(date_wk=weekdays(date))
    activity$date_wk <- ifelse(activity$date_wk %in% c("Sunday","Saturday"),"Weekend","Weekday")
    activity$date_wk <- as.factor(activity$date_wk)

Plot the corresponding time series using lattice plot.

    library(lattice)
    xyplot(steps_impute ~ interval|date_wk, data=activity, type='l',layout=c(1,2),ylab="Steps", xlab="Time", main="Number of steps over time")

<img src="figure-html/TS2-1.png" style="display: block; margin: auto;" />

It can be observed that people start to walk earlier on weekdays than
weekends.
