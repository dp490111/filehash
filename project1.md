# Project 2

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

1. Code for reading in the dataset and/or processing the data

2. Histogram of the total number of steps taken each day

3. Mean and median number of steps taken each day

4. Time series plot of the average number of steps taken

5. The 5-minute interval that, on average, contains the maximum number of steps

6. Code to describe and show a strategy for imputing missing data

7. Histogram of the total number of steps taken each day after missing values are imputed

8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

9. All of the R code needed to reproduce the results (numbers, plots, etc.) in the report


## Loading and Preprocessing the Data

The code below shows the process the read and process the data. The date variable changed from format "character" to format "date". An initial summary is used to generate an overview of the data.  


```r
setwd("/drives/home/d.parsons/JHU DSC/5 Reproducible Research") #set working directory
data <- read.csv("activity.csv") #read in data

data$date <- as.Date(data$date)

summary(data)
```

```
##      steps             date               interval     
##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
##  Median :  0.00   Median :2012-10-31   Median :1177.5  
##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
##  NA's   :2304
```


## What is the mean total number of steps taken per day?

The data is grouped by date and then summarised by steps to determine the mean and median number of steps per day. The histogram shows the frequency of the average number of steps per day. 


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
stepday <- group_by(data, date) #Group steps by date

stepday <- summarize(stepday, steps = sum(steps)) #Sum steps by date

#head(stepday) 

hist(stepday$steps, main = "Steps Per Day", xlab = "Steps", col = "lightblue") #Make histogram of steps per day. 
```


![Graph1](project1_files/figure-html/section2-1.png)




```r
avg_spd <- mean(stepday$steps, na.rm = TRUE) #Calculate average steps per day

med_spd <- median(stepday$steps, na.rm = TRUE) #Calculate median steps per day
```
The average number of steps per day is 1.0766189\times 10^{4} and the median number of steps per day is 10765.

## What is the average daily activity pattern?


```r
library(dplyr)

intstep <- group_by(data, interval) #Group steps by date

intstep <- summarize(intstep, mean  = mean(steps, na.rm = TRUE)) #Sum steps by date

#head(intstep)

plot(intstep$interval, intstep$mean, type = "l", main = "Average steps by interval", xlab = "Interval (minutes)", ylab = "Steps", col = "blue")
```

![Graph2](/project1_files/figure-html/section3-1.png)


```r
max_int <- intstep$interval[which(intstep$mean == max(intstep$mean))] #Find interval of max steps
```

The data is grouped by interval and then summarized by step. The plot above shows average steps per day for each time interval. The average maximum number of steps was 206.1698113 and occured at interval 835. 

## Imputing Missing Values


```r
library(tidyr)

num_NA <- sum(complete.cases(data)==FALSE)

#https://stackoverflow.com/questions/4862178/remove-rows-with-all-or-some-nas-missing-values-in-data-frame
#https://stackoverflow.com/questions/64502525/how-to-find-index-of-a-specific-value-in-a-dataframe-r
#https://www.r-bloggers.com/2021/07/countif-function-in-r/

data2 <- data

data2[is.na(data2)] <- mean(intstep$mean)
#https://stackoverflow.com/questions/8161836/how-do-i-replace-na-values-with-zeros-in-an-r-dataframe

#data2 <- replace_na(data2$steps, mean(intstep$mean)) #Changes dataframe to list
#https://tidyr.tidyverse.org/reference/replace_na.html


stepday2 <- group_by(data2, date) #Group steps by date

stepday2 <- summarize(stepday2, steps = sum(steps)) #Sum steps by date

hist(stepday2$steps, main = "Steps Per Day", xlab = "Steps", col = "lightblue") #Make histogram of steps per day. 
```

![Graph3](/project1_files/figure-html/section4-1.png)<!-- -->

```r
avg_spd2 <- mean(stepday2$steps) #Calculate average steps per day

med_spd2 <- median(stepday2$steps) #Calculate median steps per day
```

2304 rows had missing values. NAs were replaced with the average number of steps per interval. The average number of steps unique to each interval would provide a more accurate replacement for the NA values, but this was an unessecary level of fidelity for this application. The number of NAs was also small in comparison to the total data set. 

The plot above shows an update histogram of average steps per day using the data set with imputed values for NAs. There is very little change to the average and median steps per day. Average steps does not change, however, there is a negligible increase in median steps.

## Are there differences in activity patterns between weekdays and weekends?

Two additional columns were introduced. This process applied the weekday() function to the column "date" to extract the weekday. Then the weekday generated an additional factor "wday" to label each entry as "weekday" or "weekend". The data was grouped by wday and interval and then sumarized by steps. This process resulted in a plot of weekday vs weekend for average steps. The graph below shows that there is more activity on weekday mornings and less activity on weekday afternoons on average. There is also little activity at the very beginning and very end of each day for both weekdays and weekends. This is likely due to normal sleeping periods. 


```r
library(dplyr)

data2$day <- as.factor(weekdays(data2$date))

wkend <- c("Saturday", "Sunday")

data2$wday <- factor(data2$day %in% wkend, levels=c(FALSE, TRUE), labels=c('weekday', 'weekend'))

#https://stackoverflow.com/questions/28893193/creating-factor-variables-weekend-and-weekday-from-date

intstep2 <- group_by(data2, wday, interval) #Group steps by date
intstep2 <- summarize(intstep2, mean  = mean(steps)) #Sum steps by date
```

```
## `summarise()` has grouped output by 'wday'. You can override using the `.groups` argument.
```

```r
#https://stackoverflow.com/questions/36893812/using-dplyr-to-summarize-by-multiple-groups

library(ggplot2)

g1 <- ggplot(intstep2, aes(x = interval, y = mean, color = wday)) + geom_line() #plot base graph
  
g1 + labs(title = "Weekday vs. Weekend Average Steps", x = "Interval (minutes)", y = "Average Steps", color = "Day") #add titles
```

![Graph4](/project1_files/figure-html/section5-1.png)<!-- -->

```r
#https://r-graphics.org/recipe-line-graph-multiple-line
#https://ggplot2.tidyverse.org/reference/labs.html
#https://stackoverflow.com/questions/14622421/how-to-change-legend-title-in-ggplot


#**************************************************************************************************
# Alternate approach for data that I didn't use. 
#intstep_wkday <- filter(data2, data2$wday == "weekday")
#intstep_wkend <- filter(data2, data2$wday == "weekend")

#intstep_wkday <- group_by(intstep_wkday, interval) #Group steps by date
#intstep_wkday <- summarize(intstep_wkday, mean  = mean(steps)) #Sum steps by date

#intstep_wkend <- group_by(intstep_wkend, interval) #Group steps by date
#intstep_wkend <- summarize(intstep_wkend, mean  = mean(steps)) #Sum steps by date
```

## Markdown references

https://rmarkdown.rstudio.com/lesson-4.html

https://bookdown.org/yihui/rmarkdown-cookbook/r-code.html

https://stackoverflow.com/questions/41483287/how-to-produce-both-html-and-md-from-an-rmd-rmarkdown-and-rename-them


