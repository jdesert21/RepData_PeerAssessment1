---
title: "Module 5 week 1 Project"
output: 
  html_document: 
    keep_md: yes
  md_document:
        variant:markdown_github
---
Load the data into a dataframe named data0

```r
library(dplyr)
library(ggplot2)
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
data <- read.csv(unz(temp, "activity.csv"),header=TRUE, stringsAsFactors = FALSE)
unlink(temp)
data%>%mutate(date2=as.Date(data$date,format="%Y-%m-%d"))%>%select(-date)->data0
```
##Histogram of the number of steps taken per day  
Main difference between a histogram and a barplot :  
With bar charts, each column represents a group defined by a categorical variable; and with histograms, each column represents a group defined by a continuous, quantitative variable. So, even though it's built using barplot, it's a histogram.

```r
library(dplyr)
library(ggplot2)
data0%>%group_by(date2)%>%summarise(stepsbydate=sum(steps,na.rm=TRUE))->data1
barplot(setNames(data1$stepsbydate,data1$date2),space=0)
```

![](Module5week1project_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
summary(data1$stepsbydate)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6778   10400    9354   12810   21190
```

##Average daily activity pattern
The number of steps is on average bigger at 8:35 am

```r
library(dplyr)
library(ggplot2)
aggregate(data0,by=list(data0$interval),FUN=mean,na.rm=TRUE)->data2
data2%>%select(-c(date2,Group.1))->data2
ggplot(data2,aes(interval,steps))+geom_line()
```

![](Module5week1project_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

##Imputing missing values  
###NA number

```r
sum(is.na(data0$steps))
```

```
## [1] 2304
```

###Strategy to fill in all the missing values :  
We'll use the mean by interval to fill in the missing values


```r
filldata<-data0
gmean<-aggregate(x=filldata,by=list(gint=filldata$interval),FUN=mean,na.rm=TRUE)
for (i in 1:length(filldata[,1])){
      if(is.na(filldata[i,1])){
            filldata[i,1]<-as.numeric(filter(gmean, gint==filldata[i,2])[2])
      }
      
}
```
###Histogram of number of steps by day


```r
filldata%>%group_by(date2)%>%summarise(stepsbydate=sum(steps))->data4
ggplot(data4,aes(x=date2,y=stepsbydate))+geom_bar(stat="identity")+theme(axis.text.x = element_text(angle = 90, hjust = 1))+labs(x="Dates",y="Number of steps")
```

![](Module5week1project_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
###Mean and median of the number of steps by day
As a reminder, we'll first display the median and mean with the missing values and then the median and mean after the strategy to fill in missing values

```r
summary(data1$stepsbydate)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6778   10400    9354   12810   21190
```

```r
summary(data4$stepsbydate)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10770   10770   12810   21190
```
##Differences in activity pattern between weekdays and weekends

```r
filldata%>%mutate(ndate=weekdays(date2))->data5
for (i in 1:length(data5$ndate)){
      if(data5[i,4]%in% c("samedi","dimanche")){
            data5[i,5]<-"weekend"     
      }else data5[i,5]<-"weekday"
}
data5%>%group_by(interval,V5)%>%summarise(stepsbyinterval=sum(steps))->data6
ggplot(data6,aes(interval,stepsbyinterval))+geom_line()+facet_wrap(~V5,nrow = 2)+labs(y="Number of steps")+theme_bw()
```

![](Module5week1project_files/figure-html/unnamed-chunk-8-1.png)<!-- -->
