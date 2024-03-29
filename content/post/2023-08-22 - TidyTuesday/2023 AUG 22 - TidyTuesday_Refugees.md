---
title: "TidyTuesday - Refugees"
author: "Eugene Zhou"
date: "2023-08-22"
output: html_document
---



------------------------------------------------------------------------

The goal of this work is to apply my understanding of data towards real world projects using R and the tidyverse. The data used here is from the TidyTuesday project.

## Check Out This Week's Data

```{r Load , message=FALSE}
# load R packages for analysis
library(tidyverse)
library(tidytuesdayR)
```

```{r Get and load data, message=FALSE}
# Download the weekly data and make available in the tt object.
# Using the last_tuesday function gives us the latest TidyTuesday data from today's date

tt <- tt_load(last_tuesday())

# Check out the available data
tt
```

The data this week comes from PopulationStatistics {refugees} R package.

> {refugees} is an R package designed to facilitate access to the United Nations High Commissioner for Refugees (UNHCR) Refugee Data Finder. It provides an easy-to-use interface to the database, which covers forcibly displaced populations, including refugees, asylum-seekers, internally displaced people, stateless people, and others over a span of more than 70 years.

This package provides data from three major sources:

-   Data from UNHCR's annual statistical activities dating back to 1951.

-   Data from the United Nations Relief and Works Agency for Palestine Refugees in the Near East (UNRWA), specifically for registered Palestine refugees under UNRWA's mandate.

-   Data from the Internal Displacement Monitoring Centre (IDMC) on people displaced within their country due to conflict or violence.

The {refugees} package includes eight datasets. We're working with `population` with data from 2010 to 2022.

### The following are the included variables:

| **Variable**      | **Class** | **Description**                                                 |
|-----------------|-----------------|-------------------------------------|
| year              | double    | The year.                                                       |
| coo_name          | character | Country of origin name.                                         |
| coo               | character | Country of origin UNHCR code.                                   |
| coo_iso           | character | Country of origin ISO code.                                     |
| coa_name          | character | Country of asylum name.                                         |
| coa               | character | Country of asylum UNHCR code.                                   |
| coa_iso           | character | Country of asylum ISO code.                                     |
| refugees          | double    | The number of refugees.                                         |
| asylum_seekers    | double    | The number of asylum-seekers.                                   |
| returned_refugees | double    | The number of returned refugees.                                |
| idps              | double    | The number of internally displaced persons.                     |
| returned_idps     | double    | The number of returned internally displaced persons.            |
| stateless         | double    | The number of stateless persons.                                |
| ooc               | double    | The number of others of concern to UNHCR.                       |
| oip               | double    | The number of other people in need of international protection. |
| hst               | double    | The number of host community members.                           |

: Look's like we should different country of origin (COO) and country of asylum (COA). Let's extract the population data set from our `tt` list.

```{r}
ref_pop <- tt$population

# how does the data look?
ref_pop %>% glimpse()
```

## Questions:

1.  Which country has the most refugees fleeing the country?

## Data Cleaning:

Let's stick with ISO codes only.

```{r}
# use select to remove the UNHCR-coded fields
df <- ref_pop %>%
  select(-coo, -coa,)
```

## Exploratory Data Analysis

This section was originally confusing and I thought I was using the `group_by` and `summarise` functions wrong, but it looks like we have a collection of refugees from an **origin** country to **asylum** countries each year.

```{r}
df %>%
  group_by(coo_name, coo_iso, coa_name, coa_iso) %>%
  summarise(year, refugees)
```

Let's try making this data a little easier to work with by summing the refugees in each year first.

```{r}
df2 <- df %>%
  group_by(year, coo_name, coo_iso) %>%
  summarise(coo_refugees = sum(refugees))

# let's do a quick sanity check 
df2 %>%
  arrange(desc(coo_refugees))

```

This seems unreal. From 2018-2022, there were more than 6.5 million refugees out of Syria each year?

```{r}
df %>%
  filter(coo_iso == "SYR") %>%
  group_by(coa_iso) %>%
  ggplot() + 
  aes(x = year, y = refugees, fill = coa_iso) + 
  geom_col() + 
  theme(legend.position = "none") + 
  labs(title = "Syrian Refugees Per Year", 
       x = "Year", 
       y = "# of Refugees")
```
<img src="/post/2023-08-22 - TidyTuesday/2023%20AUG%2022%20-%20TidyTuesday_Refugees_files/figure-html/unnamed-chunk-5-1.png" width="672" />
