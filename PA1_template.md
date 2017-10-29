# Reproducible Research
Alejandra Ugarte  
28/10/2017  


## Read Activity Monitoring Data

Set working directory and download the file. Convert NAs to 0 so that you can transform the data.


```r
setwd("~/Documents/GitHub/RepData_PeerAssessment1")
df <- read.csv("activity.csv")
df[is.na(df)] <- 0
```

## What is mean total number of steps taken per day?

Aggregate the data to get the total number of steps by day - 
Create a Historgram to show this data and caluclate the mean and median number of steps. 


```r
steps <- aggregate(df$steps,by = list(df$date),sum)
colnames(steps) <- c("Date","Number of Steps")
steps$Date <- as.Date(steps$Date,format="%Y-%m-%d")
hist(steps$`Number of Steps`,col = "Red",
     main = "Total Number of Steps Each Day",xlab = "Number of Steps")
```

![](PA1_template_files/figure-html/histogram-1.png)<!-- -->

```r
mean(steps$`Number of Steps`)
```

```
## [1] 9354.23
```

```r
median(steps$`Number of Steps`)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

Aggregate the data to get the mean number of steps by interval - 
Plot this data on a line graph (type = 'l')


```r
time_series <- aggregate(df$steps,by = list(df$interval),mean)
colnames(time_series) <- c("Interval","Mean Number of Steps")
plot(time_series$Interval,time_series$`Mean Number of Steps`,
     type = "l", col="Red",xlab="Interval",
     ylab = "Avg. Number of Steps",
     main = "Average Number of Steps per Interval")
```

![](PA1_template_files/figure-html/Time Series Plot-1.png)<!-- -->

## Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

use the function which.max to find the row in the data set that contains the highest mean number of steps. Print this row to the console - interval 835 with number of steps 179.1311


```r
which.max(time_series$`Mean Number of Steps`)
```

```
## [1] 104
```

```r
time_series[104,]
```

```
##     Interval Mean Number of Steps
## 104      835             179.1311
```

## Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)

Read in the dataframe again and use sum on the is.na function.


```r
setwd("~/Documents/GitHub/RepData_PeerAssessment1")
df <- read.csv("activity.csv")
colnames(df) <- c("Steps","Date","Interval")
sum(is.na(df$Steps))
```

```
## [1] 2304
```

## Devise a strategy for filling in all of the missing values in the dataset. Create a new dataset that is equal to the original dataset but with the missing data filled in.

Use the average number of steps per interval to fill in the NAs in the file. Merge the dataframe *df* with the *time_series* aggregated data which will create a dataset with interval, steps (including NAs), the date, and the average number of steps per interval. Subset to just the NA data. With the subsetted data copy across the average number of steps per interval to the steps column (therefore getting rid of NAs). Remove the NAs from the original data set and merge with the subset data. Remove the extra column.


```r
df2 <- merge(df,time_series)
NA_stuff <- subset(df2,is.na(df2$Steps))
NA_stuff$Steps <- NA_stuff$`Mean Number of Steps`
df3 <- na.omit(df2)
final <- rbind(df3,NA_stuff)
final$`Mean Number of Steps`<- NULL
```

## Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

As before, aggregate the data to get the total number of steps per day. Create a histrogram of the data and calculate the mean and median. With a full data set, with NA's set to the average number of steps per interval - the mean has increased but the median remains the same. 


```r
final_aggs <- aggregate(final$Steps,by = list(final$Date),sum)
colnames(final_aggs) <- c("Date","Number of Steps")
final_aggs$Date <- as.Date(final_aggs$Date,format="%Y-%m-%d")
hist(final_aggs$`Number of Steps`, col = "Red", main = "Total Number of Steps each Day", xlab = "Number of Steps")
```

![](PA1_template_files/figure-html/Histogram with full data-1.png)<!-- -->

```r
mean(final_aggs$`Number of Steps`)
```

```
## [1] 10581.01
```

```r
median(final_aggs$`Number of Steps`)
```

```
## [1] 10395
```
## Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
final$Date <- as.Date(final$Date,format="%Y-%m-%d")
final$Weekday <- weekdays(final$Date)
for (i in 1:nrow(final)){
  if(final$Weekday[i] %in% c("Saturday","Sunday")){
      final$Weektype[i] <-  "Weekend"
    } else {
      final$Weektype[i] <- "Weekday"
    }
}

Weekday <- subset(final, final$Weektype=="Weekday")
Weekday <- aggregate(Weekday$Steps,by = list(Weekday$Interval),mean)
colnames(Weekday) <- c("Interval","Avg. Steps")
Weekend <- subset(final,final$Weektype=="Weekend")
Weekend <- aggregate(Weekend$Steps,by = list(Weekend$Interval),mean)
colnames(Weekend) <- c("Interval","Avg. Steps")
```

## Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

Plot both weekday and weekend datasets on a line graph using the par(mfrow) function to create a panel. 


```r
par(mfrow = c(2,1))
plot(Weekend$Interval,Weekend$`Avg. Steps`,
     type = "l", col="Red",xlab="Interval",
     ylab = "Avg. Number of Steps",
     main = "Weekend")

plot(Weekday$Interval,Weekday$`Avg. Steps`,
     type = "l", col="Red",xlab="Interval",
     ylab = "Avg. Number of Steps",
     main = "Weekday")
```

![](PA1_template_files/figure-html/panel plot-1.png)<!-- -->

