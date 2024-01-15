# Working in R

## Below are the code that I use for the analysis from creating new columns to plot

### Installing package

```{r Loading Package, install first if havent}
library(tidyverse)
library(janitor)
library(lubridate)
```

### Importing data
I'll use read.csv to import the data (this section doesn't have the code, since 
the file is too large for free version of GitHub)

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

### Looking over data

```{r}
# The str() function provide the structure of the data which provide a quick
# summary of the data and it format
str(Jan2022)
str(Feb2022)
str(Mar2022)
str(Apr2022)
str(May2022)
str(Jun2022)
str(Jul2022)
str(Aug2022)
str(Sep2022)
str(Oct2022)
str(Nov2022)
str(Dec2022)
```

### Merging data

```{r Merge data into one dataframe using bind_row}
merge_df <- bind_rows(Jan2022, Feb2022, Mar2022, Apr2022, May2022, Jun2022, Jul2022, Aug2022, Sep2022, Oct2022, Nov2022, Dec2022)
```

```{r clean and remove any empty columns or rows using clean_name, remove_empty}
merge_df <- clean_names(merge_df)
remove_empty(merge_df, which = c())
```

```{r convert started_at and ended_at from char to date time format}
#as.POSIXct function let me convert date, time and time zones instead of as.Date() which only deals with date value

merge_df$started_at <- as.POSIXct(merge_df$started_at, format = "%Y-%m-%d %H:%M:%S", tz = "GMT")
merge_df$ended_at <- as.POSIXct(merge_df$ended_at, format = "%Y-%m-%d %H:%M:%S", tz = "GMT")
```

```{r}
str(merge_df)
```

### Creating new columns

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

```{r}
clean_df <- merge_df[!(merge_df$trip_duration <= 0),]
```

### Plot 1: Number of rides by member type

```{r}
#remove scientific notation
#options(scipen = 999)
 
ggplot(data = clean_df) + aes(x = day_of_week, fill = member_casual) +
  geom_bar(position = 'dodge') +
  labs(x = 'Day of week', y = 'Number of rides', fill = 'Member Type',
       title = 'Number of rides by member type')
```

### Plot 2: Number of rides per month

```{r}
ggplot(data = clean_df) + aes(x = month, fill = member_casual) +
   geom_bar(position = 'dodge') +
   labs(x = 'Month', y = 'Number of rides', fill = 'Member type', 
        title = 'Number of rides per month')
```

### Plot 3: Bikes usage throughout the week

```{r}
ggplot(data = clean_df) + aes(x = starting_hour, fill = member_casual) +
   facet_wrap(~day_of_week) +
   geom_bar() +
   labs(x ='Starting hour', y = 'Number of rides', fill = 'Member Type', 
        #title = 'Bikes use throughout the week by hour') +
   theme(axis.text = element_text(size = 5))
```

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


```{r}
#write.csv(clean_df, file = 'Cyclistic_df.csv')
```
