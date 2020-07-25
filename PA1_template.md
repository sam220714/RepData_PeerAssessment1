Introduction
============

This is peer graded assignment 1 of Reproducible research. In this
assignment, students were asked to perform basic R programming and
obtain results whilst presenting them in a orderly fashion through usage
of R markdown.

Data
----

The activity data provided in the assignment has been gathered from a
personal activity monitoring device.

Assignment
==========

### Loading and pre-processing data

It is assumed that the csv file is in the operating directory of the
user

    activity <- read.csv("activity.csv")

### Mean total number of steps taken per day

    steps <- aggregate(activity$steps, list(activity$date), FUN = sum)
    colnames(steps) <- c("Date","Total")
    head(steps)

    ##         Date Total
    ## 1 2012-10-01    NA
    ## 2 2012-10-02   126
    ## 3 2012-10-03 11352
    ## 4 2012-10-04 12116
    ## 5 2012-10-05 13294
    ## 6 2012-10-06 15420

Creating histogram will help in further understanding:

    library(ggplot2)
    ggplot(data = steps, aes(Total)) + geom_histogram(col = "violet", fill = "lightblue", binwidth = 2500, boundary = 0) + scale_x_continuous(breaks = seq(0,25000,2500)) + scale_y_continuous(breaks = seq(0,18,2))

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](kachara_files/figure-markdown_strict/unnamed-chunk-3-1.png)

To report the mean and median of the total number steps taken per day,
mean() and median() function are put to use:

    mean(steps$Total, na.rm = TRUE)

    ## [1] 10766.19

    median(steps$Total, na.rm = TRUE)

    ## [1] 10765

### Average daily activity pattern

First a dataset is created of 5-minute intervals and the average number
of steps taken, averaged across all days

    time_int <- aggregate(activity$steps, by = list(activity$interval), mean, na.rm = TRUE)
    colnames(time_int) <- c("interval", "average")

Then a time series plot is drawn via ggplot2:

    ggplot(data = time_int, aes(interval, average)) + geom_line(col = "blue", size = 1) +ggtitle("Average daily activity pattern") + xlab("Time Intervals") + ylab("Average number of steps taken")

![](kachara_files/figure-markdown_strict/unnamed-chunk-6-1.png)

For the 5-minute interval containing the maximum number of steps,
which.max() function is used

    int_max_steps <- time_int[which.max(time_int$average),]$interval
    int_max_steps

    ## [1] 835

### Imputing missing values

Most of the raw data available contains missing values, in the
assignment, the number of NA values are determined as

    sum(is.na(activity$steps))

    ## [1] 2304

I have replaced all the missing values with the mean for that 5-minute
interval, for that another dataset was created

    imp_steps <- time_int$average[match(activity$interval, time_int$interval)]

A copy of the original file is made

    s <- read.csv("activity.csv")

Now to replace the NA values I have used a for loop

    for (i in 1:length(s$date))
    {
      if(is.na(s$steps[i]))
      {
        s$steps[i] <- imp_steps[i]
      }
    }

another dataset similar to ‘steps’ is created

    s_steps <- aggregate(s$steps, list(s$date), FUN = sum)
    colnames(s_steps) <- c("Date","Total")

Now for the histogram

    ggplot(data = s_steps, aes(Total)) + geom_histogram(col = "violet", fill = "lightblue", binwidth = 2500, boundary = 0) + scale_x_continuous(breaks = seq(0,25000,2500)) + scale_y_continuous(breaks = seq(0,30,2))

![](kachara_files/figure-markdown_strict/unnamed-chunk-13-1.png)

To report the mean and median of the total number steps taken per day,
mean() and median() functions are put to use:

    mean(s_steps$Total)

    ## [1] 10766.19

    median(s_steps$Total)

    ## [1] 10766.19

### Differences in activity patterns between weekdays and weekends

To accommodate a new factor variable, two additional columns were added
for weekday, type of weekday, whether it is a weekend or not sorts.

    s$weekday <- weekdays(as.POSIXlt(s$date))
    s$day_type <- ifelse(test = s$weekday == c("Saturday","Sunday"), yes = "Weekend", no = "Weekday")

After this, another dataset is made comprising of 5-minute intervals and
the average number of steps taken, averaged across all weekday days or
weekend days

    time_int2 <- aggregate(s$steps, by = list(s$interval, s$day_type), mean)
    names(time_int2) <- c("interval","Day_type","Average")

For the panel plot, I have used qplot() function

    qplot(data = time_int2, x = interval, y = Average, facets = Day_type~., geom="line") + ggtitle("Activity pattern in Weekdays and Weekends") + xlab("Time Intervals") + ylab("Average number of steps taken")

![](kachara_files/figure-markdown_strict/unnamed-chunk-17-1.png)
