# Health and Economic Impacts of Severe Weather Events in the US



## Objective

The objective of this assignment is to use National Weather Service's Storm Database to answer the following research questions: 

        1. Across the United States, which types of events are most harmful with respect to population health?
        2. Across the United States, which types of events have the greatest economic impact?

## Synopsis

The United States has been hit by numerous severe weather events over the years. These events are associated with health and economic impacts that can be measured using the National Weather Service's Storm Database for the years 1995 to 2011. The analyis revealed that tornadoes, thunderstorms, floods and damaging winds are the most harmful weather events with respect to people's health and in terms of economic impact. Taken together, these stroms have either injured or killed more than 50,000 people and have totalled 387 billion U.S. dollars in property or crop damages since 1995. Moreover, California, Florida and Texas have been consistently among the top five most affected states in the country.  

## Data Processing


#### 1. Load R packages needed for analysis


```r
library(ggplot2)
library(dplyr)
library(R.utils)
library(gridExtra)
library(grid)
library(scales)
library(plyr)
```


#### 2. Load data into R


```r
# Set working directory
setwd("~/Desktop/Rprogramming/RepData_PeerAssessment2")
```


```r
# Download file from URL
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", "StormData.csv.bz2")

# Unzip file
bunzip2("StormData.csv.bz2", "StormData.csv", remove=FALSE, skip=TRUE)
```

```
## [1] "StormData.csv"
## attr(,"temporary")
## [1] FALSE
```

```r
# Load data
StormData <- read.csv("StormData.csv")
```


#### 3. Check and Process data


```r
# Check data
colnames(StormData)
```

```
##  [1] "STATE__"    "BGN_DATE"   "BGN_TIME"   "TIME_ZONE"  "COUNTY"    
##  [6] "COUNTYNAME" "STATE"      "EVTYPE"     "BGN_RANGE"  "BGN_AZI"   
## [11] "BGN_LOCATI" "END_DATE"   "END_TIME"   "COUNTY_END" "COUNTYENDN"
## [16] "END_RANGE"  "END_AZI"    "END_LOCATI" "LENGTH"     "WIDTH"     
## [21] "F"          "MAG"        "FATALITIES" "INJURIES"   "PROPDMG"   
## [26] "PROPDMGEXP" "CROPDMG"    "CROPDMGEXP" "WFO"        "STATEOFFIC"
## [31] "ZONENAMES"  "LATITUDE"   "LONGITUDE"  "LATITUDE_E" "LONGITUDE_"
## [36] "REMARKS"    "REFNUM"
```

First, I created a dataset called `workingdata` in which I kept only the variables necessary to conduct the analysis. 


```r
# Keep selected variables for analysis
keep <- c("STATE", "EVTYPE", "FATALITIES", "INJURIES", "PROPDMG", "PROPDMGEXP", 
          "CROPDMG", "CROPDMGEXP")

# Create working dataset for analysis
workingdata <- StormData[keep]
```

Next, I created the variable `year` to store the date value from `BGN_DATE`. I also added a factor variable called `yearcat` with two levels to idendify "Historical" data prior to 1995, and "Recent" data since 1995. 


```r
# Format date variable
workingdata$year <- as.numeric(format(as.Date(StormData$BGN_DATE, 
                                              format="%m/%d/%Y %H:%M:%S"), "%Y"))

# Year category
workingdata$yearcat[workingdata$year<=1994] <- "Historical"
workingdata$yearcat[workingdata$year>=1995] <- "Recent"

# Take a look at data
head(workingdata, 1)
```

```
##   STATE  EVTYPE FATALITIES INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP
## 1    AL TORNADO          0       15      25          K       0           
##   year    yearcat
## 1 1950 Historical
```


#### 4. Reduce the number of levels in event type

The variable `EVTYPE` contains 189 levels to define each type of weather event in the dataset. However, many levels could be easily grouped into a single category. For this reason and to make the levels more manageable, I created a new variable called `event` that consists of 7 categories identifying all the types of events stored in `EVTYPE`. To name the new categories, I used a list provided by The National Severe Storms Laboratory (http://www.nssl.noaa.gov/education/svrwx101/).

`event` is a factor variable with 7 levels:

        1. Hail
        2. Damaging Winds
        3. Flood
        4. Tornadoes
        5. Thunderstorms
        6. Winter Weather
        7. Lightning


```r
# New event variable with fewer levels
workingdata[grepl("thunderstorms|thunderstorm|tstm|", 
                        workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Thunderstorms"

workingdata[grepl("tornado|waterspout|waterspouts|
                        tornadoes|tornados|gustnado", 
                        workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Tornadoes"

workingdata[grepl("flood|flash|flooding|heavy rain|
                        heavy rains|floods|tidal|stream|
                        prolonged rain", workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Flood"

workingdata[grepl("lightning", workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Lightning"

workingdata[grepl("hail", workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Hail"

workingdata[grepl("wind|wnd|gusty|hurricane|typhoon|
                        dust|saharan|strong|tropical|cyclones|
                        winds", workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Damaging Winds"

workingdata[grepl("cold|ice|icy|sleet|frost|freeze|
                        snow|winter|wintry|wintery|blizzard|
                        chill|freezing|avalanche|glaze|
                        patchy ice|hypothermia|
                        bitter|hyperthermia", workingdata$EVTYPE, 
                        ignore.case=TRUE), "event"] <- "Winter Weather"

# Re-order levels 
workingdata$event <- factor(workingdata$event, 
                            levels = c("Hail", 
                                       "Damaging Winds", 
                                       "Flood",
                                       "Tornadoes",
                                       "Thunderstorms",
                                       "Winter Weather", 
                                       "Lightning"))
```


#### 5. Check the frequency of events across time


```r
# Plot events per year
p0 <- ggplot(workingdata, aes(x=year, fill=event)) +
        geom_bar() + 
        labs(x="", y="Frequency") +
        scale_x_continuous(breaks=seq(1950, 2011, 10)) +
        ggtitle("Weather Events in the U.S., 1950-2011") +
        theme(plot.title = element_text(face="bold", size=15))
p0
```

![](PA2_US_Storms_final_files/figure-html/events-per-year-1.png)<!-- -->

```r
# Load function for crosstab
source("http://pcwww.liv.ac.uk/~william/R/crosstab.r")

# Crosstab - row percentages
crosstab(workingdata, row.vars="event", col.vars="yearcat", type="r")
```

```
##                yearcat Historical Recent    Sum
## event                                          
## Hail                        25.15  74.85 100.00
## Damaging Winds              28.50  71.50 100.00
## Flood                        3.69  96.31 100.00
## Tornadoes                   56.71  43.29 100.00
## Thunderstorms                4.79  95.21 100.00
## Winter Weather               1.79  98.21 100.00
## Lightning                    9.37  90.63 100.00
```

The above plot shows the number of severe weather events recorded in the U.S. from 1950 to 2011. More events have been recorded from the mid-1990s onward. The cross-table with row percentages, also shows that the majority of events in the database were recorded in more recent years. For example, across all hail storms recorded over time, 74.85 percent of them were recorded in the past 25 years compared to 25.15 percent in the years prior to 1995. Given the larger sample size, I've decided to conduct the analysis using a subset of the Storm Database covering the years 1995-2011.


```r
workingdata <- subset(workingdata, year>=1995)
```


#### 6. Calculate the value of damages in billions of U.S. dollars

Next, I calculated the dollar value of property and crop damages. The variables `PROPDMGEXP` and `CROPDMGEXP` contained the exponent but not the actual units per dollar. As such, both variables needed to be cleaned and converted into units of U.S. dollars. 


```r
# Checking property damage variable
unique(workingdata$PROPDMGEXP)
```

```
##  [1]   B M K m + 0 5 6 ? 4 2 3 7 H - 1 8
## Levels:  - ? + 0 1 2 3 4 5 6 7 8 B h H K m M
```

```r
# Converting letter exponents into integers
workingdata$PROPEXP[workingdata$PROPDMGEXP == "+"] <- 0
workingdata$PROPEXP[workingdata$PROPDMGEXP == "-"] <- 0
workingdata$PROPEXP[workingdata$PROPDMGEXP == "?"] <- 0

workingdata$PROPEXP[workingdata$PROPDMGEXP == "0"] <- 1
workingdata$PROPEXP[workingdata$PROPDMGEXP == "1"] <- 1e+01
workingdata$PROPEXP[workingdata$PROPDMGEXP == "2"] <- 1e+02
workingdata$PROPEXP[workingdata$PROPDMGEXP == "3"] <- 1e+03
workingdata$PROPEXP[workingdata$PROPDMGEXP == "4"] <- 1e+04
workingdata$PROPEXP[workingdata$PROPDMGEXP == "5"] <- 1e+05
workingdata$PROPEXP[workingdata$PROPDMGEXP == "6"] <- 1e+06
workingdata$PROPEXP[workingdata$PROPDMGEXP == "7"] <- 1e+07
workingdata$PROPEXP[workingdata$PROPDMGEXP == "8"] <- 1e+08

workingdata$PROPEXP[workingdata$PROPDMGEXP ==  ""] <- 1
workingdata$PROPEXP[workingdata$PROPDMGEXP == "h"] <- 1e+02
workingdata$PROPEXP[workingdata$PROPDMGEXP == "H"] <- 1e+02
workingdata$PROPEXP[workingdata$PROPDMGEXP == "K"] <- 1e+03
workingdata$PROPEXP[workingdata$PROPDMGEXP == "m"] <- 1e+06
workingdata$PROPEXP[workingdata$PROPDMGEXP == "M"] <- 1e+06
workingdata$PROPEXP[workingdata$PROPDMGEXP == "B"] <- 1e+09

# Calculating value of property damage in billions of U.S. dollars
workingdata$PROPDMGVAL <- (workingdata$PROPDMG * workingdata$PROPEXP) / 1e+09
```


```r
# Checking crop damage variable
unique(workingdata$CROPDMGEXP)
```

```
## [1]   M m K B ? 0 k 2
## Levels:  ? 0 2 B k K m M
```

```r
# Converting letter exponents into integers
workingdata$CROPEXP[workingdata$CROPDMGEXP == "?"] <- 0

workingdata$CROPEXP[workingdata$CROPDMGEXP == "0"] <- 1
workingdata$CROPEXP[workingdata$CROPDMGEXP == "2"] <- 1e+02

workingdata$CROPEXP[workingdata$CROPDMGEXP ==  ""] <- 1
workingdata$CROPEXP[workingdata$CROPDMGEXP == "k"] <- 1e+03
workingdata$CROPEXP[workingdata$CROPDMGEXP == "K"] <- 1e+03
workingdata$CROPEXP[workingdata$CROPDMGEXP == "m"] <- 1e+06
workingdata$CROPEXP[workingdata$CROPDMGEXP == "M"] <- 1e+06
workingdata$CROPEXP[workingdata$CROPDMGEXP == "B"] <- 1e+09

# Calculating value of crop damage in billions of U.S. dollars
workingdata$CROPDMGVAL <- (workingdata$CROPDMG * workingdata$CROPEXP) / 1e+09
```


## Results


#### 1. Health Impact


```r
# Aggregate by event
health <- ddply(workingdata, 
                .(event), 
                summarise,
                tot_fatalities=sum(FATALITIES, na.rm=TRUE),
                tot_injuries=sum(INJURIES, na.rm=TRUE))

# Convert to thousands
health$total <- (health$tot_fatalities+health$tot_injuries)/1000

# Plot1
p1 <- ggplot(data=health, aes(x=reorder(event, -total), y=total)) +
        geom_bar(fill="steelblue", stat="identity") +
        theme(axis.text.x=element_text(angle=45,hjust=1,vjust=1)) +
        labs(x="", y="Number of persons ('000)") +
        ggtitle("By event") +
        theme(plot.title = element_text(size=10.5))

# Aggregate by state
health_state <- ddply(workingdata, 
                      .(STATE, event), 
                      summarise, 
                      tot_fatalities=sum(FATALITIES, na.rm=TRUE),
                      tot_injuries=sum(INJURIES, na.rm=TRUE))

# Convert to thousands
health_state$total <- (health_state$tot_fatalities +
                                 health_state$tot_injuries)/1000

# Top 5 states
temp <- aggregate(health_state$total, list(STATE=health_state$STATE), sum)
temp <- temp[order(-temp$x), ][1:5, ]
top5_health <- merge(temp, health_state, by=c("STATE"))
levels(top5_health$event) <- gsub("^Damaging Winds","Wind", levels(top5_health$event))
levels(top5_health$event) <- gsub("^Thunderstorms","T-Storm",levels(top5_health$event))
levels(top5_health$event) <- gsub("^Winter Weather","Winter",levels(top5_health$event))
levels(top5_health$event) <- gsub("^Tornadoes","Tornado",levels(top5_health$event))
rm(temp)

# Plot2
p2 <- ggplot(data=top5_health, aes(x=STATE, y=total, fill=event)) +
        geom_bar(position="fill", stat="identity") +
        scale_y_continuous(labels=percent_format()) +
        labs(x="", y="") +
        ggtitle("By state") +
        theme(plot.title=element_text(size=10.5)) +
        theme(legend.text=element_text(size=8)) +
        theme(legend.position="bottom") +
        theme(legend.title = element_blank())

# Panel1
title1=textGrob("Health Impact of U.S. Weather Events, 1995-2011 1/", 
                gp=gpar(fontsize=15, fontface="bold"))

footnote1=textGrob("1/ Includes fatalities and injuries.", 
                  x=0, hjust=-0.1, vjust=0.1,
                  gp=gpar(fontsize=10, fontface="italic"))

grid.arrange(p1, p2, nrow=1, ncol=2, top=title1, bottom=footnote1)
```

![](PA2_US_Storms_final_files/figure-html/health-impact-1.png)<!-- -->

The weather events that have been most harmful to the U.S. population over the past 25 years are tornadoes, thunderstorms, floods and damaging winds. Together they have either injured or killed nearly 52,000 people since 1995. Looking at the states that have been hit the most by these events are Alabama, California, Florida, Missouri and Texas. In particular, tornadoes have been the most harmful to the population of Alabama, while thunderstoms have lead to more casualties in California.


#### 2. Economic Impact


```r
# Aggregate by event
economic <- ddply(workingdata, 
                  .(event), 
                  summarise, 
                  tot_propdmg=sum(PROPDMGVAL, na.rm=TRUE), 
                  tot_cropdmg=sum(CROPDMGVAL, na.rm=TRUE))

# Total economic damage by event
economic$total <- economic$tot_propdmg + economic$tot_cropdmg

# Plot3
p3 <- ggplot(data=economic, aes(x=reorder(event, -total), y=total)) +
        geom_bar(fill="#FF6666", stat="identity") +
        theme(axis.text.x=element_text(angle=45,hjust=1,vjust=1)) +
        labs(x="", y="Billions of USD") +
        ggtitle("By event") +
        theme(plot.title = element_text(size=10.5))

# Aggregate by state
economic_state <- ddply(workingdata, 
                        .(STATE, event), 
                        summarise, 
                        tot_propdmg=sum(PROPDMGVAL, na.rm=TRUE),
                        tot_cropdmg=sum(CROPDMGVAL, na.rm=TRUE))

# Total economic damage by state
economic_state$total <- economic_state$tot_propdmg + economic_state$tot_cropdmg

# Top 5 states
temp <- aggregate(economic_state$total, list(STATE=economic_state$STATE), sum)
temp <- temp[order(-temp$x), ][1:5, ]
top5_econ <- merge(temp, economic_state, by=c("STATE"))
levels(top5_econ$event) <- gsub("^Damaging Winds","Wind", levels(top5_econ$event))
levels(top5_econ$event) <- gsub("^Thunderstorms","T-Storm", levels(top5_econ$event))
levels(top5_econ$event) <- gsub("^Winter Weather","Winter", levels(top5_econ$event))
levels(top5_econ$event) <- gsub("^Tornadoes","Tornado", levels(top5_econ$event))
rm(temp)

# Plot4
p4 <- ggplot(data=top5_econ, aes(x=STATE, y=total, fill=event)) +
        geom_bar(position="fill", stat="identity") +
        scale_y_continuous(labels=percent_format()) +
        labs(x="", y="") +
        ggtitle("By state") +
        theme(plot.title=element_text(size=10.5)) +
        theme(legend.text=element_text(size=8)) +
        theme(legend.position="bottom") +
        theme(legend.title = element_blank())

# Panel2
title2=textGrob("Economic Impact of U.S. Weather Events, 1995-2011 1/", 
                gp=gpar(fontsize=15, fontface="bold"))

footnote2=textGrob("1/ Includes damages to property and crop.", 
                  x=0, hjust=-0.1, vjust=0.1,
                  gp=gpar(fontsize=10, fontface="italic"))

grid.arrange(p3, p4, nrow=1, ncol=2, top=title2, bottom=footnote2)
```

![](PA2_US_Storms_final_files/figure-html/economic-impact-1.png)<!-- -->

The weather events that have had the highest economic impact over the past 25 years are floods, damaging winds, thunderstorms and tornadoes. Looking at the states that have been the most impacted by these events in dollar terms are California, Florida, Louisiana, Mississippi and Texas. In particular, floods have caused the most damages in California, while damaging winds (hurricanes) have caused the most damages in Florida. In Gulf states like Louisiana, Mississippi and Texas, both damaging winds and thunderstorms have had large economic impacts.


