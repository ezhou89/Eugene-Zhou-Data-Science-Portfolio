---
  title: "2023 MAY 30 - World's Oldest Individuals"
author: "Eugene Zhou"
date: 2023-05-30
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

tues <- last_tuesday("2023-5-30")

tt <- tt_load(tues)
```

```r
# Check out the available data
tt
```

The data this week comes from the [Wikipedia List of the verified oldest people](https://en.wikipedia.org/wiki/List_of_the_verified_oldest_people) via [frankiethull on GitHub](https://github.com/frankiethull/centenarians). These are lists of the 100 known verified oldest people sorted in descending order by age in years and days. The oldest person ever whose age has been independently verified is Jeanne Calment (1875--1997) of France, who lived to the age of 122 years and 164 days. The oldest verified man ever is Jiroemon Kimura (1897--2013) of Japan, who lived to the age of 116 years and 54 days. The oldest known living person is Maria Branyas of Spain, aged 116 years, 85 days. The oldest known living man is Juan Vicente Pérez of Venezuela, aged 114 years, 1 day. The 100 oldest women have, on average, lived several years longer than the 100 oldest men.

### The following are the included variables:

| **Variable**                | **Class** | **Description**                                                                                                                           |
  |--------------------|-------------------|---------------------------------|
  | rank                        | integer   | This person's overall rank by age.                                                                                                        |
| name                        | character | The full name of this person.                                                                                                             |
| birth_date                  | date      | This person's birth date.                                                                                                                 |
  | death_date                  | date      | This person's death date (or NA if still alive).                                                                                          |
| age                         | double    | The person's age, either on the day of their death or on the day when the dataset was extracted on 2023-05-25.                            |
  | place_of_death_or_residence | character | Where the person lives now or where they were when they died.                                                                             |
  | gender                      | character | Most likely actually the sex assigned to the person at birth (the source article does not specify).                                       |
  | still_alive                 | character | Either "alive" if the person was still alive at the time when the article as referenced, or "deceased" if the person was no longer alive. |
  
  ## Initial Thoughts and Questions
  
  Based on the variables provided, the most obvious question to ask would be:
  
  -   Do the oldest individuals on the planet were living in the same general location or country?
  
  -   Who tends to live longer? Men or Women?
  
  From what I've previously heard regarding age statistics, Japan tends to have the longest life expectancy and women tend to live longer than men, so let's explore if that's still the case.

## Data Wrangling and Exploratory Data Analysis

Let's create a series of smaller data frames to explore whether these variables have some sort of relationship that we can further explore

```r
# Extract data from tt 
cent <- tt$centenarians

cent %>% skimr::skim()
```

Luckily for us, the only variable that has missing data is `death_date`, because 12 of these individuals are currently alive. All other data is complete, making this a relatively straight forward and small set of data with 200 observations.

Lets first tackle the question of which gender tends to live longer through a quick summary.

```r
# Group data by gender and count via summarize

cent %>%
  group_by(gender) %>%
  summarise(n())

```

We see this data set collected 100 from each gender, suggesting we may not be able to determine which gender lives longer. However, we should be able to determine how much longer, on average, one gender lives compared to the other.

```{r}
cent %>%
  group_by(gender) %>%
  summarise(mean(age))
```

We see that the longest living women, on average, are still outliving the longest living men by 3.1 years.

Next lets see if we can determine which countries have the most long-lived individuals.

```r
# Create a new data frame to explore which countries have the highest number of individuals

df1 <- cent %>%
  group_by(place_of_death_or_residence) %>%
  summarise(`No. of Individuals` = n(), 
            `Avg Age` = mean(age),
            `Shortest` = min(age),  
            `Longest` = max(age)) 

df1 %>% 
  arrange(desc(`No. of Individuals`))
```

Oddly enough, it looks like the US has the greatest number of centenarians. How does that compare against the countries with the longest average lifespan? Which are the top 10 countries?
  
  ```{r}
cent %>%
  group_by(place_of_death_or_residence) %>%
  summarise(`Avg Age` = mean(age)) %>%
  transmute(`Country` = place_of_death_or_residence, `Avg Age`) %>%
  arrange(desc(`Avg Age`)) %>%
  head(n = 10)

```

According to the data, Jamaica has the highest average lifespan, which is unheard of from conventional knowledge. How is this possible? Lets double check how many individuals from these top countries are actually on this list.

```r
cent %>%
  group_by(place_of_death_or_residence) %>%
  summarise(`Avg Age` = mean(age), 
            `No. of Individuals` = n()) %>%
  transmute(`Country` = place_of_death_or_residence, `Avg Age`, `No. of Individuals`) %>%
  arrange(desc(`Avg Age`)) %>%
  head(n = 10)
```

Looks like these n of 1 data points are acting more like outliers to drive up the average age. What if we filter out the countries that only have 1 individual?

```r
cent %>%
  group_by(place_of_death_or_residence) %>%
  summarise(`Avg Age` = mean(age), 
            `No. of Individuals` = n()) %>%
  filter(`No. of Individuals` > 1) %>%
  transmute(`Country` = place_of_death_or_residence, `Avg Age`, `No. of Individuals`) %>%
  arrange(desc(`Avg Age`)) %>%
  head(n = 10)
```

Its still very surprising to see that Brazil is now at the top of the list, as ranked by the average oldest recorded lifespans in descending order. However, we now see that Japan and the US is now included in the top 10, with Japan ranking higher than the US, as expected.

Lets create a visualization thats essentially a heat map in the world map. To do so, we need to create a new data frame that contains the coordinates of the world map in addition to our centenarian data.

```r
# Load up the world map coordinates
library(maps)
world_map <- map_data("world")

# Replace United States with USA to join the data sets
cent$place_of_death_or_residence[cent$place_of_death_or_residence == "United States"] <- "USA"

# Merge centanarian data with world coordinates into a new data frame
df2 <- cent %>%
  group_by(place_of_death_or_residence) %>%
  summarise(`Avg Age` = mean(age), 
            `No. of Individuals` = n()) %>%
  transmute(`Country` = place_of_death_or_residence, `Avg Age`, `No. of Individuals`) %>%
  full_join(world_map, join_by("Country" == "region"))
```

Now we can visualize.

```r
ggplot(df2) +
  geom_map(
    dat = world_map, map = world_map, aes(map_id = region),
    fill = "white", color = "black", size = 0.25
  ) +
  geom_map(map = world_map, aes(map_id = Country, fill = `No. of Individuals`), size = 0.25) +
  scale_fill_gradient2(low = "white", high = "red", na.value = "white", name = "# of People") +
  expand_limits(x = world_map$long, y = world_map$lat) + 
  labs(title = "Countries with the Most Centenarians", 
       y = "Latitude", 
       x = "Longitude")
```

```r
# Save image
# This will save your most recent plot
image <- ggplot(df2) +
  geom_map(
    dat = world_map, map = world_map, aes(map_id = region),
    fill = "white", color = "black", size = 0.25
  ) +
  geom_map(map = world_map, aes(map_id = Country, fill = `No. of Individuals`), size = 0.25) +
  scale_fill_gradient2(low = "white", high = "red", na.value = "white", name = "# of People") +
  expand_limits(x = world_map$long, y = world_map$lat) + 
  labs(title = "Countries with the Most Centenarians", 
       y = "Latitude", 
       x = "Longitude")

image %>% ggsave(
  filename = "map.png",
  device = "png")

```

## Future Work

1.  TBD
