---
title: "2023 MAY 23 - Squirrels in Central Park"
date: 2023-05-23
output: html_document
---



------------------------------------------------------------------------

The goal of this work is to apply my understanding of data towards real world projects using R and the tidyverse. The data used here is from the TidyTuesday project.

## Check Out This Week's Data


```r
# load R packages for analysis
library(tidyverse)
library(tidytuesdayR)
```


```r
# Download the weekly data and make available in the tt object.
# Using the last_tuesday function gives us the latest TidyTuesday data from today's date

tues <- last_tuesday("2023-5-23")

tt <- tt_load(tues)
```

```
## 
## 	Downloading file 1 of 1: `squirrel_data.csv`
```


```r
# Check out the available data
tt
```

The data this week comes from the [2018 Central Park Squirrel Census](https://data.cityofnewyork.us/Environment/2018-Central-Park-Squirrel-Census-Squirrel-Data/vfnx-vebw). [The Squirrel Census](https://www.thesquirrelcensus.com/) is a multimedia science, design, and storytelling project focusing on the Eastern gray (Sciurus carolinensis). They count squirrels and present their findings to the public. The dataset contains squirrel data for each of the 3,023 sightings, including location coordinates, age, primary and secondary fur color, elevation, activities, communications, and interactions between squirrels and with humans.\
\
\
The following are the included variables:

| **Variable**                               | **Class** | **Description**                                                                                                                                                                                                                          |
|------------------|------------------|------------------------------------|
| X                                          | double    | Longitude coordinate for squirrel sighting point                                                                                                                                                                                         |
| Y                                          | double    | Latitude coordinate for squirrel sighting point                                                                                                                                                                                          |
| Unique Squirrel ID                         | character | Identification tag for each squirrel sightings. The tag is comprised of `Hectare ID` + `Shift` + `Date` + `Hectare Squirrel Number.`                                                                                                     |
| Hectare                                    | character | ID tag, which is derived from the hectare grid used to divide and count the park area. One axis that runs predominantly north-to-south is numerical (1-42), and the axis that runs predominantly east-to-west is roman characters (A-I). |
| Shift                                      | character | Value is either `AM` or `PM,` to communicate whether or not the sighting session occurred in the morning or late afternoon.                                                                                                              |
| Date                                       | double    | Concatenation of the sighting session day and month.                                                                                                                                                                                     |
| Hectare Squirrel Number                    | double    | Number within the chronological sequence of squirrel sightings for a discrete sighting session.                                                                                                                                          |
| Age                                        | character | Value is either `Adult` or `Juvenile.`                                                                                                                                                                                                   |
| Primary Fur Color                          | character | Primary Fur Color - value is either `Gray`, `Cinnamon` or `Black`.                                                                                                                                                                       |
| Highlight Fur Color                        | character | Discrete value or string values comprised of `Gray`, `Cinnamon` or `Black`.                                                                                                                                                              |
| Combination of Primary and Highlight Color | character | A combination of the previous two columns; this column gives the total permutations of primary and highlight colors observed.                                                                                                            |
| Color notes                                | character | Sighters occasionally added commentary on the squirrel fur conditions. These notes are provided here.                                                                                                                                    |
| Location                                   | character | Value is either `Ground Plane` or `Above Ground`. Sighters were instructed to indicate the location of where the squirrel was when first sighted.                                                                                        |
| Above Ground Sighter Measurement           | character | For squirrel sightings on the ground plane, fields were populated with a value of `FALSE`.                                                                                                                                               |
| Specific Location                          | character | Sighters occasionally added commentary on the squirrel location. These notes are provided here.                                                                                                                                          |
| Running                                    | logical   | Squirrel was seen running.                                                                                                                                                                                                               |
| Chasing                                    | logical   | Squirrel was seen chasing another squirrel.                                                                                                                                                                                              |
| Climbing                                   | logical   | Squirrel was seen climbing a tree or other environmental landmark.                                                                                                                                                                       |
| Eating                                     | logical   | Squirrel was seen eating.                                                                                                                                                                                                                |
| Foraging                                   | logical   | Squirrel was seen foraging for food.                                                                                                                                                                                                     |
| Other Activities                           | character | Other activities squirrels were observed doing.                                                                                                                                                                                          |
| Kuks                                       | logical   | Squirrel was heard kukking, a chirpy vocal communication used for a variety of reasons.                                                                                                                                                  |
| Quaas                                      | logical   | Squirrel was heard quaaing, an elongated vocal communication which can indicate the presence of a ground predator such as a dog.                                                                                                         |
| Moans                                      | logical   | Squirrel was heard moaning, a high-pitched vocal communication which can indicate the presence of an air predator such as a hawk.                                                                                                        |
| Tail flags                                 | logical   | Squirrel was seen flagging its tail. Flagging is a whipping motion used to exaggerate squirrel's size and confuse rivals or predators. Looks as if the squirrel is scribbling with tail into the air.                                    |
| Tail twitches                              | logical   | Squirrel was seen twitching its tail. Looks like a wave running through the tail, like a breakdancer doing the arm wave. Often used to communicate interest, curiosity.                                                                  |
| Approaches                                 | logical   | Squirrel was seen approaching human, seeking food.                                                                                                                                                                                       |
| Indifferent                                | logical   | Squirrel was indifferent to human presence.                                                                                                                                                                                              |
| Runs from                                  | logical   | Squirrel was seen running from humans, seeing them as a threat.                                                                                                                                                                          |
| Other Interactions                         | character | Sighter notes on other types of interactions between squirrels and humans.                                                                                                                                                               |
| Lat/Long                                   | character | Latitude and longitude                                                                                                                                                                                                                   |

## Initial Thoughts and Questions

Based on the variables provided, there are likely a few interesting relationships to explore with regards to the 3,023 recorded squirrel sightings, including `location`, `date`, `activity`, and `shift`.

-   Are more squirrels observed during the AM or PM?

-   What kind of activities are the squirrels engaging in?

## Data Wrangling and Exploratory Data Analysis

Let's create a series of smaller data frames to explore whether these variables have some sort of relationship that we can further explore


```r
# Extract squirrel data from tt 
squirrel <- tt$squirrel_data

squirrel %>% skimr::skim()
```


Table: <span id="tab:Extract"></span>Table 1: Data summary

|                         |           |
|:------------------------|:----------|
|Name                     |Piped data |
|Number of rows           |3023       |
|Number of columns        |31         |
|_______________________  |           |
|Column type frequency:   |           |
|character                |14         |
|logical                  |13         |
|numeric                  |4          |
|________________________ |           |
|Group variables          |None       |


**Variable type: character**

|skim_variable                              | n_missing| complete_rate| min| max| empty| n_unique| whitespace|
|:------------------------------------------|---------:|-------------:|---:|---:|-----:|--------:|----------:|
|Unique Squirrel ID                         |         0|          1.00|  13|  14|     0|     3018|          0|
|Hectare                                    |         0|          1.00|   3|   3|     0|      339|          0|
|Shift                                      |         0|          1.00|   2|   2|     0|        2|          0|
|Age                                        |       121|          0.96|   1|   8|     0|        3|          0|
|Primary Fur Color                          |        55|          0.98|   4|   8|     0|        3|          0|
|Highlight Fur Color                        |      1086|          0.64|   4|  22|     0|       10|          0|
|Combination of Primary and Highlight Color |         0|          1.00|   1|  27|     0|       22|          0|
|Color notes                                |      2841|          0.06|   3| 153|     0|      135|          0|
|Location                                   |        64|          0.98|  12|  12|     0|        2|          0|
|Above Ground Sighter Measurement           |       114|          0.96|   1|   5|     0|       41|          0|
|Specific Location                          |      2547|          0.16|   4| 102|     0|      304|          0|
|Other Activities                           |      2586|          0.14|   4| 132|     0|      307|          0|
|Other Interactions                         |      2783|          0.08|   2| 106|     0|      197|          0|
|Lat/Long                                   |         0|          1.00|  38|  45|     0|     3023|          0|


**Variable type: logical**

|skim_variable | n_missing| complete_rate| mean|count                |
|:-------------|---------:|-------------:|----:|:--------------------|
|Running       |         0|             1| 0.24|FAL: 2293, TRU: 730  |
|Chasing       |         0|             1| 0.09|FAL: 2744, TRU: 279  |
|Climbing      |         0|             1| 0.22|FAL: 2365, TRU: 658  |
|Eating        |         0|             1| 0.25|FAL: 2263, TRU: 760  |
|Foraging      |         0|             1| 0.47|FAL: 1588, TRU: 1435 |
|Kuks          |         0|             1| 0.03|FAL: 2921, TRU: 102  |
|Quaas         |         0|             1| 0.02|FAL: 2973, TRU: 50   |
|Moans         |         0|             1| 0.00|FAL: 3020, TRU: 3    |
|Tail flags    |         0|             1| 0.05|FAL: 2868, TRU: 155  |
|Tail twitches |         0|             1| 0.14|FAL: 2589, TRU: 434  |
|Approaches    |         0|             1| 0.06|FAL: 2845, TRU: 178  |
|Indifferent   |         0|             1| 0.48|FAL: 1569, TRU: 1454 |
|Runs from     |         0|             1| 0.22|FAL: 2345, TRU: 678  |


**Variable type: numeric**

|skim_variable           | n_missing| complete_rate|        mean|       sd|          p0|         p25|         p50|         p75|        p100|hist  |
|:-----------------------|---------:|-------------:|-----------:|--------:|-----------:|-----------:|-----------:|-----------:|-----------:|:-----|
|X                       |         0|             1|      -73.97|     0.01|      -73.98|      -73.97|      -73.97|      -73.96|      -73.95|▅▇▅▆▂ |
|Y                       |         0|             1|       40.78|     0.01|       40.76|       40.77|       40.78|       40.79|       40.80|▇▇▃▅▆ |
|Date                    |         0|             1| 10119487.40| 42466.71| 10062018.00| 10082018.00| 10122018.00| 10142018.00| 10202018.00|▇▂▇▂▃ |
|Hectare Squirrel Number |         0|             1|        4.12|     3.10|        1.00|        2.00|        3.00|        6.00|       23.00|▇▂▁▁▁ |


```r
# Create a new data frame to explore observed squirrels depending on time of day and age

df1 <- squirrel %>%
  select(`Shift`, `Hectare Squirrel Number`, `Age`)

df1 %>% 
  group_by(Shift, Age) %>%
  summarise(`# Observed` = n(), 
            `Min Squirrels per Hectare` = min(`Hectare Squirrel Number`), 
            `Max Squirrels per Hectare` = max(`Hectare Squirrel Number`), 
            .groups = "keep")
```

```
## # A tibble: 8 × 5
## # Groups:   Shift, Age [8]
##   Shift Age      `# Observed` `Min Squirrels per Hectare` Max Squirrels per He…¹
##   <chr> <chr>           <int>                       <dbl>                  <dbl>
## 1 AM    ?                   2                           3                      5
## 2 AM    Adult            1152                           1                     23
## 3 AM    Juvenile          152                           1                     16
## 4 AM    <NA>               41                           1                     14
## 5 PM    ?                   2                           2                      5
## 6 PM    Adult            1416                           1                     17
## 7 PM    Juvenile          178                           1                     15
## 8 PM    <NA>               80                           1                      9
## # ℹ abbreviated name: ¹​`Max Squirrels per Hectare`
```


```r
df1 %>% 
  group_by(Shift, Age) %>%
  summarise(`# Observed` = n(), 
            .groups = "keep") %>% 
  ggplot(aes(x = `Shift`, y = `# Observed`, fill = `Age`)) + 
  geom_col(position = "dodge") +
  labs(x = "Shift (AM/PM)", 
       y = "# of Squirrels Observed", 
       title = "More Squirrels Observed in the PM")
```

<img src="/post/2023-05-23-TidyTuesday/2023 MAY 23 - tidy_tuesday_files/figure-html/Plot DF1-1.png" width="672" />



Let's see what happens to the squirrel data frame if we drop the `NA`'s.


```r
df2 <- squirrel %>% 
  drop_na()
```

Only 1 row of data remains, indicating at least one observation is missing in each variable across nearly every single squirrel sighted, suggesting dropping `NA`'s immediately is not a good choice for our data set.\


```r
df2 <- squirrel %>% 
  select(Running, Chasing, Climbing, Eating, Foraging) %>%
  gather(key = "Activity", value = "Observed") %>%
  group_by(Activity) %>%
  count(Observed)

df2 %>%
  ggplot(aes(x = Activity, y = n, fill = Observed)) + 
  geom_col()
```

<img src="/post/2023-05-23-TidyTuesday/2023 MAY 23 - tidy_tuesday_files/figure-html/unnamed-chunk-4-1.png" width="672" />

It appears that we have so mand `NA`'s because a squirrel cannot be doing some of these activities simultaneously.\



## Future Work

1.  TBD
