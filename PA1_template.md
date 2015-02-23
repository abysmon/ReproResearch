---
title: 'Reproducible Research: Human Activity monitoring Project'
author: "Kay Dee"
date: "Thursday, February 12, 2015"
output:
  html_document:
    highlight: kate
    keep_md: yes
    self_contained: no
    theme: flatly
    toc: yes
---

***
### Background 

This project makes use of data from a personal activity monitoring device. The device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

***
### About the Data

The data for can be downloaded from [here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:
  
  1. steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
  2. date: The date on which the measurement was taken in YYYY-MM-DD format
  3. interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

***
### Loading and preprocessing the data


```r
url = 'https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip'
download.file(url, 'activity.zip')
fname = unzip('activity.zip', list = T)$Name
data = read.csv(fname, stringsAsFactors = F)
options(max.print = 100, scipen = 999)
```




***
### Central measure of total number of steps taken per day


```r
daily = data %>% 
          group_by(date) %>% 
          summarize(Total.Steps = sum(steps))

daily = as.data.frame(daily)
daily = daily[complete.cases(daily), ]
```

###### Distribution of total number of steps walked on daily basis


```r
ggplot(daily, aes(date)) + 
  geom_histogram(aes(Total.Steps), stat = 'bin', 
                 fill = 'lightblue4', color = 'white') + 
  theme_bw()
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

###### Average of total steps walked on daily basis is 10766
###### Median of total steps walked on daily basis is 10765

***
### Average daily activity pattern


```r
stepwise = data %>% 
            group_by(interval) %>% 
            summarize(Avg.Steps = mean(steps, na.rm = T))

stepwise = as.data.frame(stepwise)
ggplot(stepwise, aes(x = interval, y = Avg.Steps, color = 'red')) + 
  geom_line() + theme_bw() + guides(color = F)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

###### The 5-min interval that has the maximum number of steps on average across all the days, starts from




```r
paste0(substr(tmax,1,2),":",substr(tmax,3,4))
```

```
## [1] "08:35"
```

***
### Imputing missing values

###### Total number of missing values in the dataset 2304

###### Filling in the missing values in the dataset with the median for the corresponding 5-minute interval


```r
fdata = ddply(data, 
              .(interval), 
              transform, 
              steps = ifelse(is.na(steps), median(steps, na.rm = T), steps))
fdata = as.data.frame(fdata)
# fdata is the dataset with the missing values filled
```

###### Total steps walked for each day


```r
fdaily = fdata %>% 
  group_by(date) %>% 
  summarize(Total.Steps = sum(steps))

fdaily = as.data.frame(fdaily)
```

###### Distribution of total steps taken each day, of the missing values filled dataset

```r
ggplot(fdaily, aes(date)) + 
  geom_histogram(aes(Total.Steps), stat = 'bin', 
                 fill = 'firebrick1', color = 'white') + 
  theme_bw() + ylab('Daily total steps taken') + xlab('5min periods')
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

###### Average of total steps walked on daily basis is 9503.8688525.

###### Median of total steps walked on daily basis is 10395.

With missing values imputed the mean and median for the dataset comes down or in another words, with missing values, mean/median gets over-estimated.

***
### Differences in activity patterns between weekdays and weekends?

###### Creating a separate variable for indicating weekday/weekends


```r
fdata.ts = fdata
fdata.ts[ ,2] = as.POSIXct(strptime(x = fdata.ts[ ,2], format = "%Y-%m-%d"))
fdata.ts$wday = weekdays(fdata.ts[ ,2])

fdata.ts$wdayflag = 'Weekday'
fdata.ts[(fdata.ts$wday=="Sunday")|(fdata.ts$wday=="Saturday"), 5] = 'Weekend'
fdata.ts$wdayflag = factor(fdata.ts$wdayflag)
fdata.ts = fdata.ts[ ,-4]
```

###### Average steps walked for each 5-minute interval weekday/weekends wise 


```r
fweekday = fdata.ts %>% 
  group_by(interval, wdayflag) %>% 
  summarize(AvgSteps = mean(steps))

fweekday = as.data.frame(fweekday)
```

Plot of average steps walked for each 5-minute interval being split over weekdays vs weekends


```r
fweekday$hours = as.POSIXct(strptime(sprintf("%04d",fweekday$interval), '%H%M'))
fweekday$hours = as.numeric(fweekday$hours - trunc(fweekday$hours, "days"))
class(fweekday$hours) = "POSIXct"

ggplot(fweekday, aes(x = hours, y = AvgSteps, color = wdayflag)) + 
  geom_line() + 
  theme_bw() + 
  scale_x_datetime(breaks = date_breaks("2 hour"), 
                   labels = date_format("%H:%M")) +
  guides(color = FALSE) + 
  facet_wrap(~wdayflag, nrow = 2)
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 

***