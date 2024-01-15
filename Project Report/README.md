---
title: "Cyclistic Bike Study"
author: "Thuan"
date: "2023-12-18"
output:
  rmarkdown::github_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Introduction
Over the past months, I have been taking the Google Data Analytics courses on 
Coursera. Throughout the course there was a heavy emphasis on the analysis 
process: Ask, Prepare, Process, Analyze, Share and Act. A case study is a great 
way to showcase what I have learned and understanding from the course. Given a 
scenario and the data, with the analysis process in mind, below are my approach 
toward the case study, I'll be using R and Tableau for this report.

## Scenario
In 2016, Cyclistic launched a successful bike-share offering. Since then, the 
program has grown to a fleet of 5,824 bicycles that are geotracked and locked 
into a network of 692 stations across Chicago. The bikes can be unlocked from 
one station and returned to any other station in the system anytime.

You are a junior data analyst working in the marketing analyst team at 
Cyclistic, a bike-share company in Chicago. The director of marketing believes 
the company’s future success depends on maximizing the number of annual 
memberships. Therefore, your team wants to understand how casual riders and 
annual members use Cyclistic bikes differently. From these insights, your team 
will design a new marketing strategy to convert casual riders into annual 
members. But first, Cyclistic executives must approve your recommendations, so 
they must be backed up with compelling data insights and professional data 
visualizations.

Cyclistic’s marketing strategy relied on building general awareness and 
appealing to broad consumer segments. One approach that helped make these things 
possible was the flexibility of its pricing plans: single-ride passes, full-day 
passes, and annual memberships. Customers who purchase single-ride or full-day
passes are referred to as casual riders. Customers who purchase annual 
memberships are Cyclistic members.

Cyclistic’s finance analysts have concluded that annual members are much more 
profitable than casual riders. Although the pricing flexibility helps Cyclistic 
attract more customers, Moreno believes that maximizing the number of annual 
members will be key to future growth. Rather than creating a marketing campaign 
that targets all-new customers, Moreno believes there is a very good chance to 
convert casual riders into members. She notes that casual riders are already 
aware of the Cyclistic program and have chosen Cyclistic for their mobility 
needs.

## Business task 
Lily Moreno is the director of marketing and your manager. Moreno is responsible 
for the development of campaigns and initiatives to promote the bike-share 
program. These may include email, social media, and other channels.

Moreno has set a clear goal: Design marketing strategies aimed at converting 
casual riders into annual members. In order to do that, however, the marketing 
analyst team needs to better understand how annual members and casual riders 
differ, why casual riders would buy a membership, and how digital media could 
affect their marketing tactics. Moreno and her team are interested in analyzing 
the Cyclistic historical bike trip data to identify trends.

Moreno has establish three questions that will guide the future marketing 
programs: 
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become 
members?

## Prepare
We are given historical trip data from the company to analyze and identify 
trends. The data provide are from 2003 to 2023.
(Note: The datasets have a different name because Cyclistic is a fictional 
company. For the purposes of this case study, the datasets are appropriate and 
will enable you to answer the business questions. The data has been made 
available by Motivate International Inc. under this license.)

## Process
I'll be using R programming to load the data as spreadsheet cannot process the
large amount of data that are being use. I'll be using data from the year 2022.

## Installing package
Below are package that I'll be using to process the data.

```{r Loading Package, install first if havent}
library(tidyverse)
library(janitor)
library(lubridate)
```

## Importing data
After downloading the data provide, I'll be using read.csv to import the data into R.

```{r Import data}
Jan2022 <- read.csv("202201-divvy-tripdata.csv")
Feb2022 <- read.csv("202202-divvy-tripdata.csv")
Mar2022 <- read.csv("202203-divvy-tripdata.csv")
Apr2022 <- read.csv("202204-divvy-tripdata.csv")
May2022 <- read.csv("202205-divvy-tripdata.csv")
Jun2022 <- read.csv("202206-divvy-tripdata.csv")
Jul2022 <- read.csv("202207-divvy-tripdata.csv")
Aug2022 <- read.csv("202208-divvy-tripdata.csv")
Sep2022 <- read.csv("202209-divvy-tripdata.csv")
Oct2022 <- read.csv("202210-divvy-tripdata.csv")
Nov2022 <- read.csv("202211-divvy-tripdata.csv")
Dec2022 <- read.csv("202212-divvy-tripdata.csv")
```

# Looking over data
The step below give me a quick view of the data and the whether the formatting
for the columns are consistent.

```{r}
# The str() function provide the structure of the data which provide a quick
# summary of the data and it format
str(Jan2022)
str(Feb2022)
str(Mar2022)
#str(Apr2022)
#str(May2022)
#str(Jun2022)
#str(Jul2022)
#str(Aug2022)
#str(Sep2022)
#str(Oct2022)
#str(Nov2022)
#str(Dec2022)
```

# Merging data
In this step, I'm merging the data to create a new data set and remove any empty
columns and rows.

```{r Merge data into one dataframe using bind_row}
merge_df <- bind_rows(Jan2022, Feb2022, Mar2022, Apr2022, May2022, Jun2022, 
                       Jul2022, Aug2022, Sep2022, Oct2022, Nov2022, Dec2022)
```

```{r clean and remove any empty columns or rows using clean_name, remove_empty}
merge_df <- clean_names(merge_df)
remove_empty(merge_df, which = c())
```

From earlier using the str() functions that display the summary of the columns,
we can see that the columns started_at and ended_at are format as char instead
of date time format. Below, I'll convert char into date time.

```{r convert started_at and ended_at from char to date time format}
#as.POSIXct function let me convert date, time and time zones instead of as.Date() which only deals with date value
merge_df$started_at <- as.POSIXct(merge_df$started_at, format = "%Y-%m-%d %H:%M:%S", tz = "GMT")
merge_df$ended_at <- as.POSIXct(merge_df$ended_at, format = "%Y-%m-%d %H:%M:%S", tz = "GMT")
```

```{r}
str(merge_df)
```

# Insight
After a brief view of the data, with our current information, we would only be able to aggregate at ride-level, AKA popular stations, types of bikes used, percentages of casual members to annual members, etc.

Below, I'll be adding new columns that could help me answer the business task,
such as the day of week that customers rent bike, the month in order to see
which month is most popular, and the trip duration of each rental.

```{r}
merge_df$day_of_week <- wday(merge_df$started_at, label = T, abbr = T)
```

```{r}
merge_df$month <- month(merge_df$started_at)
```

```{r}
merge_df$trip_duration <- difftime(merge_df$ended_at, merge_df$started_at, units = 'sec')
```

```{r}
merge_df$starting_hour <- format(as.POSIXct(merge_df$started_at), '%H')
```

```{r}
str(merge_df)
```

After, adding the new columns, I'll clean the data one more by creating a new 
dataframe that would remove any trip duration that are below 0.

```{r}
clean_df <- merge_df[!(merge_df$trip_duration <= 0),]
```

# Analyze
During this analyze step, I would like to remind myself of the business task 
that need to solve. Which are: 


1. How do annual members and casual riders use Cyclistic bikes differently?


2. Why would casual riders buy Cyclistic annual memberships?


3. How can Cyclistic use digital media to influence casual riders to become 
members? 

Furthermore, I would like to add on to this by looking at the hours of bike
use between the casual and members, which time of the years where the bike 
rental peak, and between casual and member who spent more time using the bike 
over the years.

In order to answer these questions, I'll be using graph and plot to better 
observe the tasks that are being ask.

### Plot 1: Number of rides by member type

```{r}
#remove scientific notation
options(scipen = 999)
 
ggplot(data = clean_df) + aes(x = day_of_week, fill = member_casual) +
  geom_bar(position = 'dodge') +
  labs(x = 'Day of week', y = 'Number of rides', fill = 'Member Type',
       title = 'Number of rides by member type')
```

![Plot_1](https://github.com/pahtato/Google-Analytics-Case-Study/assets/149435656/77707981-9efe-4f21-863c-43fb9ebd985f)


### Plot 2: Number of rides per month

```{r}
ggplot(data = clean_df) + aes(x = month, fill = member_casual) +
   geom_bar(position = 'dodge') +
   labs(x = 'Month', y = 'Number of rides', fill = 'Member type', 
        title = 'Number of rides per month')
```

![Plot_2](https://github.com/pahtato/Google-Analytics-Case-Study/assets/149435656/cb9ebca5-a3fa-4fa5-ad2c-c6557aaf3de9)


### Plot 3: Bikes usage throughout the week

```{r}
ggplot(data = clean_df) + aes(x = starting_hour, fill = member_casual) +
   facet_wrap(~day_of_week) +
   geom_bar() +
   labs(x ='Starting hour', y = 'Number of rides', fill = 'Member Type', 
        title = 'Bikes use throughout the week by hour') +
   theme(axis.text = element_text(size = 5))
```

![Plot_3](https://github.com/pahtato/Google-Analytics-Case-Study/assets/149435656/85599b55-451e-48b1-b08d-45a3fe9f6e57)


### Table 1: Average trip duration of member types by day of weeks

```{r}
aggregate(clean_df$trip_duration ~ clean_df$member_casual + clean_df$day_of_week, FUN = mean)
```

### Plot 4: Average trip duration of member type

```{r}
ggplot(data = clean_df) +
  aes(x = member_casual, y = trip_duration, color = member_casual, 
      fill = member_casual) +
  geom_bar(stat = 'summary')
```

![Plot_4](https://github.com/pahtato/Google-Analytics-Case-Study/assets/149435656/69c23e1a-1e96-451f-bb70-5bb2a1ab457e)


### Most frequent stations by member type

```{r}
#count will let you count, group & sort the unique values from a dataframe

annual_member_df <- filter(clean_df, member_casual=='member')
count(annual_member_df, start_station_name, sort = T)
count(annual_member_df, end_station_name, sort = T)
```

```{r}
casual_member_df <- filter(clean_df, member_casual=='casual')
count(casual_member_df, start_station_name, sort = T)
count(casual_member_df, end_station_name, sort = T)
```

# Observation from the data
Below are hypothesis that I have gather after analyzing the plots and graph 
throughout the whole process of the analysis:

First, there's a high chance that the casual members are composed of tourists 
and/or families member who use the rental service as a mean to sightseeing and 
other leisure activities as the usage of casual members are mostly on weekends.
Also, some of the popular station that casual members frequent are mostly 
attractions location.

Second, there are a high usage of annual members in the weekdays compare to 
casual members, this information give us an idea that there a high possibility 
that the annual member are composed of working adults who are using the service 
as a mean of transportation for work.From Tableau, some of the most frequent 
station for the annual members doesn't compose of any location near any 
attraction.

Last and not least, there’s a possibility that the service could be exploit from
the single-ride pass, we are given information from the scenario that there are 
three price plan of single-ride, full-day, and annual, furthermore, the bike 
can be returned anytime. This could created a problem where customers could 
exploit it by just buying a single-ride passes.

# Recommendation
Below are a few suggestion that could provide solution for the business tasks:

In order to convert casual to annual, there should be an incentive to convert 
for the customers. What I suggest would be a mileage reward for annual members 
base on the how much there mileage are for the week, month, or year.

There should be prioritize advertisement on the weekends and from the months of 
June through August. Since there are peak casual riders on the weekend and June
through August. Also, I would suggest occasional offer discount to switch to an 
annual members. Another, suggestion is using location-based advertising to 
target the popular stations among the casual members.

### Constraint
There are a few constraint from the data that limit the analysis, such as casual
riders are a combination of single-ride and full-day passes. If this information
are given we could further pinpoint whether the customers are exploiting the 
single-ride passes. Also, having a unique IDs would give more information of 
individual who has used the service the most, which allows for a more specific 
promotion such as the milestone reward that are mention.

```{r}
#write.csv(clean_df, file = 'Cyclistic_df.csv')
```
