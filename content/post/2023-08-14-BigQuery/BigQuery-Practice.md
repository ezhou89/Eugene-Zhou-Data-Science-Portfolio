---
title: "Practicing BigQuery, SQL, and R"
date: 2023-08-14
output: html_document
editor_options:
  chunk_output_type: inline
---

<script src="/rmarkdown-libs/htmlwidgets/htmlwidgets.js"></script>
<script src="/rmarkdown-libs/jquery/jquery.min.js"></script>
<link href="/rmarkdown-libs/leaflet/leaflet.css" rel="stylesheet" />
<script src="/rmarkdown-libs/leaflet/leaflet.js"></script>
<link href="/rmarkdown-libs/leafletfix/leafletfix.css" rel="stylesheet" />
<script src="/rmarkdown-libs/proj4/proj4.min.js"></script>
<script src="/rmarkdown-libs/Proj4Leaflet/proj4leaflet.js"></script>
<link href="/rmarkdown-libs/rstudio_leaflet/rstudio_leaflet.css" rel="stylesheet" />
<script src="/rmarkdown-libs/leaflet-binding/leaflet.js"></script>
<script src="/rmarkdown-libs/htmlwidgets/htmlwidgets.js"></script>
<script src="/rmarkdown-libs/jquery/jquery.min.js"></script>
<link href="/rmarkdown-libs/leaflet/leaflet.css" rel="stylesheet" />
<script src="/rmarkdown-libs/leaflet/leaflet.js"></script>
<link href="/rmarkdown-libs/leafletfix/leafletfix.css" rel="stylesheet" />
<script src="/rmarkdown-libs/proj4/proj4.min.js"></script>
<script src="/rmarkdown-libs/Proj4Leaflet/proj4leaflet.js"></script>
<link href="/rmarkdown-libs/rstudio_leaflet/rstudio_leaflet.css" rel="stylesheet" />
<script src="/rmarkdown-libs/leaflet-binding/leaflet.js"></script>
<script src="/rmarkdown-libs/leaflet-providers/leaflet-providers_1.13.0.js"></script>
<script src="/rmarkdown-libs/leaflet-providers-plugin/leaflet-providers-plugin.js"></script>

# Purpose

I’ve recently been learning and practicing SQL, and given the fact that Google BigQuery is such a big deal now, I wanted to find a way to implement practice SQL by pulling from BigQuery’s public data sets and then visualizing in R.

Luckily, I found [this tutorial](https://youtu.be/6j27h_17C1Q) on YouTube from John Little that acts as a tutorial/walk-through. We will be querying [JHU’s Covid19](https://github.com/CSSEGISandData/COVID-19) public dataset for practice.

``` r
# Install packages if not already installed
#install.packages(c("DBI", "bigrquery"))
```

## Let’s load up the packages.

``` r
# load tidymodels and tidyverse packages
library(tidymodels)
library(tidyverse)

# bigrquery is a package to access BigQuery via R
library(bigrquery)

# DBI is a package that interfaces relational databases and R
library(DBI)
```

## Establish a database connection

First create a new GCP project in the GCP. The `dbConnect()` argument, `billing`, should have the value of a GCP **project ID**

GPC \> Home \> Dashboard GPC \> BigQuery

``` r
con <- dbConnect(
  bigquery(),
  project = "bigquery-public-data",
  dataset = "covid19_jhu_csse",
  billing = "fleet-furnace-316922"
)
con
```

    ## <BigQueryConnection>
    ##   Dataset: bigquery-public-data.covid19_jhu_csse
    ##   Billing: fleet-furnace-316922

Now, from **within the R console**, authenticate with GCP

    bigrquery::bq_auth()

## Create a pointer to a Google BQ database table

Here we query the JHU COVID19 data, specifically the summary table available in the BigQuery data (bigquery-public-data.covid19_jhu_csse.summary), where **summary** after the name of the data set denotes the specific table in the relational database. While we receive a warning message about `bigrquery` and `dbplyr` being incompatible, the package and functions appear to still work. Let’s try to set `message=FALSE`.

``` r
my_db_pointer <- tbl(con, "summary")
```

    ## Warning: <BigQueryConnection> uses an old dbplyr interface
    ## ℹ Please install a newer version of the package or contact the maintainer
    ## This warning is displayed once every 8 hours.

Now we can use [dplyr verbs](https://dplyr.tidyverse.org/) to explore the db table. For example, the `glimpse()` function is fairly similar to the following SQL query:

``` sql
SELECT *
FROM bigquery-public-data.covid19_jhu_csse.summary
LIMIT 10;
```

We see here that `glimpse()` returns all the variables or fields in the table, as well as the data type.

``` r
glimpse(my_db_pointer) 
```

    ## Rows: ??
    ## Columns: 13
    ## Database: BigQueryConnection
    ## $ province_state <chr> "South Dakota", "South Carolina", "Michigan", "Florida"…
    ## $ country_region <chr> "US", "US", "US", "US", "US", "US", "US", "US", "US", "…
    ## $ date           <date> 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 2020-0…
    ## $ latitude       <dbl> 43.20660, 32.82488, 45.37790, 26.90131, 37.01910, 38.03…
    ## $ longitude      <dbl> -98.58856, -79.96512, -85.19614, -81.92949, -78.66314, …
    ## $ location_geom  <wk_wkt> POINT (-98.58856 43.2066), POINT (-79.96512 32.82488…
    ## $ confirmed      <int> 8, 501, 13, 367, 11, 71, 26, 3, 353, 495, 19, 15, 4, 45…
    ## $ deaths         <int> 0, 10, 1, 40, 0, 2, 1, 0, 14, 24, 0, 2, 0, 4, 2, 0, 1, …
    ## $ recovered      <chr> "0", "0", "0", "0", "0", "0", "0", "0", "0", "0", "0", …
    ## $ active         <int> 8, 491, 12, 327, 11, 69, 25, 3, 339, 471, 19, 13, 4, 41…
    ## $ fips           <chr> "46023", "45019", "26029", "12015", "51037", "51540", "…
    ## $ admin2         <chr> "Charles Mix", "Charleston", "Charlevoix", "Charlotte",…
    ## $ combined_key   <chr> "Charles Mix, South Dakota, US", "Charleston, South Car…

Here we see that `count()` is the equivalent as the following SQL query to count the lines in the table.

``` sql
SELECT count(*)
FROM summary;
```

``` r
count(my_db_pointer) 
```

    ## # Source:   SQL [1 x 1]
    ## # Database: BigQueryConnection
    ##         n
    ##     <int>
    ## 1 4264080

## SQL

Using the `DBI` package, we can write SQL queries directly in R. For example, rather than using `glimpse`, we can use the following query.

``` r
#DBI::dbGetQuery(con, 
#"SELECT * 
#FROM `bigquery-public-data.covid19_jhu_csse.summary`
#LIMIT 10;")
```

### Examples of a more complex SQL query

``` r
jhu_covid19_nc_counties_10 <- dbGetQuery(con,
           "SELECT
              admin2, province_state, confirmed
            FROM
              `bigquery-public-data.covid19_jhu_csse.summary` 
            WHERE 
              country_region = 'US'
              AND date = '2020-06-30'
              AND province_state = 'North Carolina'
            LIMIT 10;"
)
jhu_covid19_nc_counties_10
```

    ## # A tibble: 10 × 3
    ##    admin2    province_state confirmed
    ##    <chr>     <chr>              <int>
    ##  1 Alamance  North Carolina      1146
    ##  2 Alexander North Carolina        91
    ##  3 Alleghany North Carolina        31
    ##  4 Anson     North Carolina       128
    ##  5 Ashe      North Carolina        54
    ##  6 Avery     North Carolina        12
    ##  7 Beaufort  North Carolina        83
    ##  8 Bertie    North Carolina       145
    ##  9 Bladen    North Carolina       371
    ## 10 Brunswick North Carolina       510

``` r
jhu_covid19_nc_counties <- dbGetQuery(con,
                                      "SELECT
                                        admin2, province_state, confirmed
                                      FROM
                                        `bigquery-public-data.covid19_jhu_csse.summary`
                                      WHERE
                                        country_region = 'US' 
                                        AND date = '2020-06-30'
                                        AND province_state = 'North Carolina';")

jhu_covid19_nc_counties
```

    ## # A tibble: 101 × 3
    ##    admin2    province_state confirmed
    ##    <chr>     <chr>              <int>
    ##  1 Alamance  North Carolina      1146
    ##  2 Alexander North Carolina        91
    ##  3 Alleghany North Carolina        31
    ##  4 Anson     North Carolina       128
    ##  5 Ashe      North Carolina        54
    ##  6 Avery     North Carolina        12
    ##  7 Beaufort  North Carolina        83
    ##  8 Bertie    North Carolina       145
    ##  9 Bladen    North Carolina       371
    ## 10 Brunswick North Carolina       510
    ## # ℹ 91 more rows

Here, we can see the function `bq_table_fields` provides us with all the fields, as well as their data types.

``` r
bigrquery::bq_table_fields("bigquery-public-data.austin_bikeshare.bikeshare_trips")
```

    ## <bq_fields>
    ##   trip_id <STRING>
    ##   subscriber_type <STRING>
    ##   bike_id <STRING>
    ##   bike_type <STRING>
    ##   start_time <TIMESTAMP>
    ##   start_station_id <INTEGER>
    ##   start_station_name <STRING>
    ##   end_station_id <STRING>
    ##   end_station_name <STRING>
    ##   duration_minutes <INTEGER>

## Using dplyr and dbplyr

`dplyr` can be used as a way to query data without needing to know SQL. `dbplyr` within the `dplyr` will be used to access data, and does not need to be called separate from the `tidyverse`.

First *connect to a table* using the `tbl()` function. This assumes you have already used `DBI::dbConnect()` to broker the database connection – as we did above.

``` r
covid_data_DBplyr_style <- tbl(con, "summary") 
```

> Note: `collect()` will run a query that has been assigned to an object.

`collect()` will activate your SQL query and store in local RAM.

``` r
# running collect on the list assigned to the variable will query the summary table from BigQuery

covid_data_DBplyr_style %>% 
  collect()   # this will gather all the data from the summary table
```

Above will pull the entire data table down into local RAM. A better approach is to leverage dplyr. Let dplyr/dbplyr translate queries into SQL and send those to the SQL engine. This allows use to use the RDMBS for summarizing data while using local RAM and CPU for retrieving only the data you want to manipulate.

``` r
# we can use dplyr verbs here to clean the data using tidy principles which are then translated into a SQL query

jhu_covid19_DBP_nc_counties <- covid_data_DBplyr_style %>% 
  filter(province_state == "North Carolina", date == '2020-06-30')  %>% 
  select(province_state, country_region, date, latitude, 
         longitude, confirmed, deaths, #location_geom,
         recovered, active, fips, admin2, combined_key) 

jhu_covid19_DBP_nc_counties 
```

    ## # Source:   SQL [?? x 12]
    ## # Database: BigQueryConnection
    ##    province_state country_region date       latitude longitude confirmed deaths
    ##    <chr>          <chr>          <date>        <dbl>     <dbl>     <int>  <int>
    ##  1 North Carolina US             2020-06-30     36.0     -79.4      1146     37
    ##  2 North Carolina US             2020-06-30     35.9     -81.2        91      1
    ##  3 North Carolina US             2020-06-30     36.5     -81.1        31      0
    ##  4 North Carolina US             2020-06-30     35.0     -80.1       128      2
    ##  5 North Carolina US             2020-06-30     36.4     -81.5        54      1
    ##  6 North Carolina US             2020-06-30     36.1     -81.9        12      0
    ##  7 North Carolina US             2020-06-30     35.5     -76.8        83      0
    ##  8 North Carolina US             2020-06-30     36.1     -77.0       145      4
    ##  9 North Carolina US             2020-06-30     34.6     -78.6       371      3
    ## 10 North Carolina US             2020-06-30     34.1     -78.2       510      5
    ## # ℹ more rows
    ## # ℹ 5 more variables: recovered <chr>, active <int>, fips <chr>, admin2 <chr>,
    ## #   combined_key <chr>

### show_query()

If you want to see the SQL

``` r
jhu_covid19_DBP_nc_counties %>% 
  show_query()
```

    ## <SQL>
    ## SELECT
    ##   `province_state`,
    ##   `country_region`,
    ##   `date`,
    ##   `latitude`,
    ##   `longitude`,
    ##   `confirmed`,
    ##   `deaths`,
    ##   `recovered`,
    ##   `active`,
    ##   `fips`,
    ##   `admin2`,
    ##   `combined_key`
    ## FROM `summary`
    ## WHERE (`province_state` = 'North Carolina') AND (`date` = '2020-06-30')

See Also [Writing SQL with dbplyr](https://dbplyr.tidyverse.org/articles/sql.html)  
See Also dbplyr \> Articles \> Verb translation  
See Also dbplyr \> Articles \> Function translation

### Variations

The following are some initial variations using dplyr verbs to create SQL queries, and connect with the RDMBS in a client/server fashion.

``` r
glimpse(my_db_pointer)
```

    ## Rows: ??
    ## Columns: 13
    ## Database: BigQueryConnection
    ## $ province_state <chr> "South Dakota", "South Carolina", "Michigan", "Florida"…
    ## $ country_region <chr> "US", "US", "US", "US", "US", "US", "US", "US", "US", "…
    ## $ date           <date> 2020-05-15, 2020-05-15, 2020-05-15, 2020-05-15, 2020-0…
    ## $ latitude       <dbl> 43.20660, 32.82488, 45.37790, 26.90131, 37.01910, 38.03…
    ## $ longitude      <dbl> -98.58856, -79.96512, -85.19614, -81.92949, -78.66314, …
    ## $ location_geom  <wk_wkt> POINT (-98.58856 43.2066), POINT (-79.96512 32.82488…
    ## $ confirmed      <int> 8, 501, 13, 367, 11, 71, 26, 3, 353, 495, 19, 15, 4, 45…
    ## $ deaths         <int> 0, 10, 1, 40, 0, 2, 1, 0, 14, 24, 0, 2, 0, 4, 2, 0, 1, …
    ## $ recovered      <chr> "0", "0", "0", "0", "0", "0", "0", "0", "0", "0", "0", …
    ## $ active         <int> 8, 491, 12, 327, 11, 69, 25, 3, 339, 471, 19, 13, 4, 41…
    ## $ fips           <chr> "46023", "45019", "26029", "12015", "51037", "51540", "…
    ## $ admin2         <chr> "Charles Mix", "Charleston", "Charlevoix", "Charlotte",…
    ## $ combined_key   <chr> "Charles Mix, South Dakota, US", "Charleston, South Car…

``` r
my_search <- my_db_pointer %>% 
  filter(province_state == "North Carolina", date == '2020-06-30')  %>% 
  select(province_state, country_region, date, latitude, 
         longitude, confirmed, deaths, #location_geom,
         recovered, active, fips, admin2, combined_key)
```

``` r
my_search %>% show_query()
```

    ## <SQL>
    ## SELECT
    ##   `province_state`,
    ##   `country_region`,
    ##   `date`,
    ##   `latitude`,
    ##   `longitude`,
    ##   `confirmed`,
    ##   `deaths`,
    ##   `recovered`,
    ##   `active`,
    ##   `fips`,
    ##   `admin2`,
    ##   `combined_key`
    ## FROM `summary`
    ## WHERE (`province_state` = 'North Carolina') AND (`date` = '2020-06-30')

``` r
my_search  
```

    ## # Source:   SQL [?? x 12]
    ## # Database: BigQueryConnection
    ##    province_state country_region date       latitude longitude confirmed deaths
    ##    <chr>          <chr>          <date>        <dbl>     <dbl>     <int>  <int>
    ##  1 North Carolina US             2020-06-30     36.0     -79.4      1146     37
    ##  2 North Carolina US             2020-06-30     35.9     -81.2        91      1
    ##  3 North Carolina US             2020-06-30     36.5     -81.1        31      0
    ##  4 North Carolina US             2020-06-30     35.0     -80.1       128      2
    ##  5 North Carolina US             2020-06-30     36.4     -81.5        54      1
    ##  6 North Carolina US             2020-06-30     36.1     -81.9        12      0
    ##  7 North Carolina US             2020-06-30     35.5     -76.8        83      0
    ##  8 North Carolina US             2020-06-30     36.1     -77.0       145      4
    ##  9 North Carolina US             2020-06-30     34.6     -78.6       371      3
    ## 10 North Carolina US             2020-06-30     34.1     -78.2       510      5
    ## # ℹ more rows
    ## # ℹ 5 more variables: recovered <chr>, active <int>, fips <chr>, admin2 <chr>,
    ## #   combined_key <chr>

``` r
# finally stores my_search into local RAM 

my_search %>% 
  collect()
```

    ## # A tibble: 101 × 12
    ##    province_state country_region date       latitude longitude confirmed deaths
    ##    <chr>          <chr>          <date>        <dbl>     <dbl>     <int>  <int>
    ##  1 North Carolina US             2020-06-30     36.0     -79.4      1146     37
    ##  2 North Carolina US             2020-06-30     35.9     -81.2        91      1
    ##  3 North Carolina US             2020-06-30     36.5     -81.1        31      0
    ##  4 North Carolina US             2020-06-30     35.0     -80.1       128      2
    ##  5 North Carolina US             2020-06-30     36.4     -81.5        54      1
    ##  6 North Carolina US             2020-06-30     36.1     -81.9        12      0
    ##  7 North Carolina US             2020-06-30     35.5     -76.8        83      0
    ##  8 North Carolina US             2020-06-30     36.1     -77.0       145      4
    ##  9 North Carolina US             2020-06-30     34.6     -78.6       371      3
    ## 10 North Carolina US             2020-06-30     34.1     -78.2       510      5
    ## # ℹ 91 more rows
    ## # ℹ 5 more variables: recovered <chr>, active <int>, fips <chr>, admin2 <chr>,
    ## #   combined_key <chr>

### Collecting all the data into a local tibble

``` r
# let's gather all covid data for Durham, NC as our data set of interest
# tidy format gets converted to SQL query 
jhu_covid19_durham_sinceApril <- covid_data_DBplyr_style %>% 
  filter(province_state == "North Carolina", 
         admin2 == "Durham",
         date >= '2020-04-01')  %>% 
  select(date, confirmed, deaths, recovered, active, 
         fips, admin2, combined_key) 

my_local_tbl <- jhu_covid19_durham_sinceApril %>% 
  collect()

my_local_tbl
```

    ## # A tibble: 1,073 × 8
    ##    date       confirmed deaths recovered active fips  admin2 combined_key       
    ##    <date>         <int>  <int> <chr>      <int> <chr> <chr>  <chr>              
    ##  1 2020-05-15       968     37 0            931 37063 Durham Durham, North Caro…
    ##  2 2022-05-26     77503    337 <NA>          NA 37063 Durham Durham, North Caro…
    ##  3 2023-02-13    102135    395 <NA>          NA 37063 Durham Durham, North Caro…
    ##  4 2022-02-16     67823    295 <NA>          NA 37063 Durham Durham, North Caro…
    ##  5 2021-05-26     25490    224 <NA>          NA 37063 Durham Durham, North Caro…
    ##  6 2020-04-13       290      1 0            289 37063 Durham Durham, North Caro…
    ##  7 2021-10-06     33405    253 <NA>          NA 37063 Durham Durham, North Caro…
    ##  8 2021-08-25     28893    242 <NA>          NA 37063 Durham Durham, North Caro…
    ##  9 2022-07-14     85557    345 <NA>          NA 37063 Durham Durham, North Caro…
    ## 10 2022-10-03     94466    365 <NA>          NA 37063 Durham Durham, North Caro…
    ## # ℹ 1,063 more rows

``` r
# arrange conducted at the database as part of the query
jhu_covid19_durham_sinceApril %>% 
  arrange(date) 
```

    ## # Source:     SQL [?? x 8]
    ## # Database:   BigQueryConnection
    ## # Ordered by: date
    ##    date       confirmed deaths recovered active fips  admin2 combined_key       
    ##    <date>         <int>  <int> <chr>      <int> <chr> <chr>  <chr>              
    ##  1 2020-04-01       126      0 0              0 37063 Durham Durham, North Caro…
    ##  2 2020-04-02       147      0 0              0 37063 Durham Durham, North Caro…
    ##  3 2020-04-03       159      0 0              0 37063 Durham Durham, North Caro…
    ##  4 2020-04-04       182      1 0              0 37063 Durham Durham, North Caro…
    ##  5 2020-04-05       186      1 0              0 37063 Durham Durham, North Caro…
    ##  6 2020-04-06       191      1 0              0 37063 Durham Durham, North Caro…
    ##  7 2020-04-07       219      1 0              0 37063 Durham Durham, North Caro…
    ##  8 2020-04-08       236      1 0              0 37063 Durham Durham, North Caro…
    ##  9 2020-04-09       243      1 0              0 37063 Durham Durham, North Caro…
    ## 10 2020-04-10       259      1 0              0 37063 Durham Durham, North Caro…
    ## # ℹ more rows

``` r
# arranged at the local level
my_local_tbl %>% 
  arrange(date)
```

    ## # A tibble: 1,073 × 8
    ##    date       confirmed deaths recovered active fips  admin2 combined_key       
    ##    <date>         <int>  <int> <chr>      <int> <chr> <chr>  <chr>              
    ##  1 2020-04-01       126      0 0              0 37063 Durham Durham, North Caro…
    ##  2 2020-04-02       147      0 0              0 37063 Durham Durham, North Caro…
    ##  3 2020-04-03       159      0 0              0 37063 Durham Durham, North Caro…
    ##  4 2020-04-04       182      1 0              0 37063 Durham Durham, North Caro…
    ##  5 2020-04-05       186      1 0              0 37063 Durham Durham, North Caro…
    ##  6 2020-04-06       191      1 0              0 37063 Durham Durham, North Caro…
    ##  7 2020-04-07       219      1 0              0 37063 Durham Durham, North Caro…
    ##  8 2020-04-08       236      1 0              0 37063 Durham Durham, North Caro…
    ##  9 2020-04-09       243      1 0              0 37063 Durham Durham, North Caro…
    ## 10 2020-04-10       259      1 0              0 37063 Durham Durham, North Caro…
    ## # ℹ 1,063 more rows

### Visualize

Creating visualizations requires 100% of the data. the [dbplot package](https://db.rstudio.com/dbplot) provides helper functions that automate the aggregation and plotting steps. `dbplot` is an alternative visualization approach to assist in the best practices of *transforming* data in the database, then *plotting* in R. Please also see this fuller [discussion on creating visualizations](https://db.rstudio.com/best-practices/visualization/) from databases using R.

``` r
# data is cleaned in cloud
jhu_covid19_durham_sinceApril %>% 
  arrange(date) %>% 
  mutate(daily_count = deaths - lag(deaths)) %>% 
  filter(daily_count > -1) %>% 
  mutate(scare = case_when(
    daily_count >= 2 ~ "high",
    daily_count == 1 ~ "medium",
    daily_count == 0 ~ "low"
  )) %>% 
# piped into ggplot at the local level  
  ggplot(aes(date, daily_count, "low", "medium")) +
  geom_jitter(aes(color = fct_relevel(scare, "low", "medium"))) +
  geom_smooth(method = lm, se = FALSE) +
  geom_smooth(color = "red", se = FALSE) +
  scale_color_manual(values = c("forestgreen", "goldenrod", "firebrick")) +
  ylim(0, 5) +
  guides(color = FALSE) +
  labs(title = "Covid Deaths", subtitle = "daily count",
       y = "", x = "") +
  theme_classic()
```

    ## Warning: Duplicated aesthetics after name standardisation:

    ## Warning: ORDER BY is ignored in subqueries without LIMIT
    ## ℹ Do you need to move arrange() later in the pipeline or use window_order() instead?

    ## Warning: The `<scale>` argument of `guides()` cannot be `FALSE`. Use "none" instead as
    ## of ggplot2 3.3.4.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## `geom_smooth()` using formula = 'y ~ x'

    ## Warning: Removed 9 rows containing non-finite values (`stat_smooth()`).

    ## `geom_smooth()` using method = 'gam' and formula = 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 9 rows containing non-finite values (`stat_smooth()`).

    ## Warning: Removed 452 rows containing missing values (`geom_point()`).

    ## Warning: Removed 6 rows containing missing values (`geom_smooth()`).

<img src="/post/2023-08-14-BigQuery/BigQuery-Practice_files/figure-html/unnamed-chunk-20-1.png" width="672" />

``` r
jhu_covid19_durham_sinceApril %>% 
  arrange(date) %>% 
  mutate(daily_count = confirmed - lag(confirmed)) %>% 
  mutate(scare = case_when(
    daily_count > 59 ~ "extreme",
    daily_count <= 59 & daily_count > 36  ~ "high",
    daily_count <= 36 & daily_count > 19 ~ "medium",
    daily_count <= 19 ~ "low"
  )) %>% 
  ggplot(aes(date, daily_count, "low", "medium")) +
  geom_point(aes(color = fct_relevel(scare, "low", "medium", "high", "extreme"))) +
  geom_smooth(method = lm, se = FALSE) +
  geom_smooth(color = "red", se = FALSE) +
  scale_color_manual(values = c("forestgreen", "goldenrod", "darkorange", "firebrick")) +
  scale_x_date(breaks = "1 month", date_labels = "%b") +
  guides(color = FALSE) +
  labs(title = "COVID19 Cases in Durham County, NC", 
       x = "", y = "daily county",
       caption = "Source: JHU dataset") +
  theme_classic()
```

    ## Warning: Duplicated aesthetics after name standardisation:

    ## Warning: ORDER BY is ignored in subqueries without LIMIT
    ## ℹ Do you need to move arrange() later in the pipeline or use window_order() instead?

    ## `geom_smooth()` using formula = 'y ~ x'

    ## Warning: Removed 1 rows containing non-finite values (`stat_smooth()`).

    ## `geom_smooth()` using method = 'gam' and formula = 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 1 rows containing non-finite values (`stat_smooth()`).

    ## Warning: Removed 1 rows containing missing values (`geom_point()`).

<img src="/post/2023-08-14-BigQuery/BigQuery-Practice_files/figure-html/unnamed-chunk-21-1.png" width="672" />

``` r
jhu_covid19_DBP_nc_counties %>% 
  select(admin2, fips, province_state, confirmed, deaths) %>% 
  rename(county = admin2, state = province_state, cases = confirmed) %>% 
  arrange(-deaths)
```

    ## # Source:     SQL [?? x 5]
    ## # Database:   BigQueryConnection
    ## # Ordered by: -deaths
    ##    county      fips  state          cases deaths
    ##    <chr>       <chr> <chr>          <int>  <int>
    ##  1 Mecklenburg 37119 North Carolina 11170    150
    ##  2 Guilford    37081 North Carolina  2812    114
    ##  3 Durham      37063 North Carolina  3679     63
    ##  4 Henderson   37089 North Carolina   610     50
    ##  5 Wake        37183 North Carolina  5178     47
    ##  6 Chatham     37037 North Carolina   950     43
    ##  7 Rowan       37159 North Carolina  1205     42
    ##  8 Cumberland  37051 North Carolina  1180     41
    ##  9 Orange      37135 North Carolina   669     41
    ## 10 Buncombe    37021 North Carolina   591     40
    ## # ℹ more rows

## SQL – MORE

Here we can use the connection created at the top to run an SQL query in an RMarkdown code chunk

``` sql
# this code chunk is saved as special_df

SELECT `province_state`, `country_region`, `date`, 
       `latitude`, `longitude`, `confirmed`, `deaths`, 
       `recovered`, `active`, `fips`, `admin2`, `combined_key`
FROM `summary`
WHERE ((`province_state` = 'North Carolina') AND (`date` = '2020-06-30'))
```

Above code chunk created the tibble `special_df` as identified in output.var

``` r
special_df %>% 
  ggplot(aes(confirmed, deaths)) +
  geom_jitter() +
  geom_text(data = . %>% filter(deaths > 100), 
            aes(confirmed, deaths, label = admin2, hjust = 1.15))
```

<img src="/post/2023-08-14-BigQuery/BigQuery-Practice_files/figure-html/unnamed-chunk-24-1.png" width="672" />

## Example 2a: DBplyr and Austin Bikeshare trips

Make the connection to the `bigquery-public-data.austin_bikeshare` database.

``` r
con2_bq_bikes <- dbConnect(
  bigrquery::bigquery(),
  project = "bigquery-public-data",
  dataset = "austin_bikeshare",
  billing = "fleet-furnace-316922"
)
con2_bq_bikes
```

    ## <BigQueryConnection>
    ##   Dataset: bigquery-public-data.austin_bikeshare
    ##   Billing: fleet-furnace-316922

Set the query pointer to `bigquery-public-data.austin_bikeshare.bikeshare_trips` table.

``` r
bikeshare_trips <- tbl(con2_bq_bikes, "bikeshare_trips") 
```

``` r
austin_bikeshare_all <- bikeshare_trips %>% 
  mutate(trip_id = as.double(trip_id)) %>% 
  collect()
```

``` r
austin_bikeshare_all
```

    ## # A tibble: 1,975,266 × 10
    ##     trip_id subscriber_type      bike_id bike_type start_time         
    ##       <dbl> <chr>                <chr>   <chr>     <dttm>             
    ##  1 22813735 24 Hour Walk Up Pass 209     classic   2020-09-13 18:26:08
    ##  2 17187153 Local365             2095    classic   2018-04-25 15:25:04
    ##  3 26978346 Student Membership   2034    classic   2022-06-21 16:27:28
    ##  4 26882016 Local31              21543   electric  2022-06-10 17:02:10
    ##  5 26963117 Local31              19177   electric  2022-06-19 19:34:58
    ##  6 26962857 Local31              19177   electric  2022-06-19 19:08:46
    ##  7 26930959 Local31              18181   electric  2022-06-16 15:43:06
    ##  8 26920468 Local31              19955   electric  2022-06-15 14:06:39
    ##  9 26966012 Student Membership   21800   electric  2022-06-20 09:28:47
    ## 10 26871697 Local31              19479   electric  2022-06-09 13:27:30
    ## # ℹ 1,975,256 more rows
    ## # ℹ 5 more variables: start_station_id <int>, start_station_name <chr>,
    ## #   end_station_id <chr>, end_station_name <chr>, duration_minutes <int>

``` r
glimpse(austin_bikeshare_all)
```

    ## Rows: 1,975,266
    ## Columns: 10
    ## $ trip_id            <dbl> 22813735, 17187153, 26978346, 26882016, 26963117, 2…
    ## $ subscriber_type    <chr> "24 Hour Walk Up Pass", "Local365", "Student Member…
    ## $ bike_id            <chr> "209", "2095", "2034", "21543", "19177", "19177", "…
    ## $ bike_type          <chr> "classic", "classic", "classic", "electric", "elect…
    ## $ start_time         <dttm> 2020-09-13 18:26:08, 2018-04-25 15:25:04, 2022-06-…
    ## $ start_station_id   <int> 3793, 2497, 7189, 7189, 7189, 7189, 7189, 7189, 718…
    ## $ start_station_name <chr> "28th/Rio Grande", "Capitol Station / Congress & 11…
    ## $ end_station_id     <chr> NA, NA, "7189", "7189", "7189", "7189", "7189", "71…
    ## $ end_station_name   <chr> "Stolen", "Stolen", "28th/Rio", "28th/Rio", "28th/R…
    ## $ duration_minutes   <int> 1016, 1463, 15, 39, 36, 26, 76, 51, 35, 46, 9, 15, …

### Visualize

[How far is too far to bike to work?](https://mobilitylab.org/2017/02/27/how-far-bike-work/)

``` r
austin_bikeshare_all %>% 
  drop_na(subscriber_type) %>% 
  filter(duration_minutes < 100) %>%
  mutate(travel_type = case_when(
    duration_minutes <= 10 ~ "Commuter",
    duration_minutes >  10 ~ "Tourist"
  )) %>% 
  ggplot(aes(duration_minutes, fct_reorder(fct_lump(subscriber_type, prop = 0.01), duration_minutes))) +
  geom_boxplot(aes(fill = travel_type)) +
  geom_vline(xintercept = 10, linetype = "dashed", color = "grey60") +
  scale_x_log10() +
  labs(title = "Bike Share Trip times", 
       subtitle = "Austin BikeShare Program",
       y = "Type of bike rental pass",
       x = "Duration of ride in minutes",
       caption = "Source: Public datasets > Google BigQuery > Austin Bikeshare Trips",
       fill = "") 
```

<img src="/post/2023-08-14-BigQuery/BigQuery-Practice_files/figure-html/unnamed-chunk-30-1.png" width="672" />

``` r
summary(austin_bikeshare_all)
```

    ##     trip_id         subscriber_type      bike_id           bike_type        
    ##  Min.   : 1954839   Length:1975266     Length:1975266     Length:1975266    
    ##  1st Qu.:12476112   Class :character   Class :character   Class :character  
    ##  Median :18389413   Mode  :character   Mode  :character   Mode  :character  
    ##  Mean   :17984246                                                           
    ##  3rd Qu.:25532763                                                           
    ##  Max.   :30074287                                                           
    ##                                                                             
    ##    start_time                     start_station_id start_station_name
    ##  Min.   :2013-12-12 16:48:46.00   Min.   :1001     Length:1975266    
    ##  1st Qu.:2016-10-20 14:28:29.75   1st Qu.:2547     Class :character  
    ##  Median :2018-08-29 14:25:37.50   Median :2575     Mode  :character  
    ##  Mean   :2019-01-29 00:26:20.29   Mean   :3163                       
    ##  3rd Qu.:2021-10-27 19:42:21.25   3rd Qu.:3794                       
    ##  Max.   :2023-06-30 23:47:17.00   Max.   :7341                       
    ##                                   NA's   :4447                       
    ##  end_station_id     end_station_name   duration_minutes  
    ##  Length:1975266     Length:1975266     Min.   :    2.00  
    ##  Class :character   Class :character   1st Qu.:    6.00  
    ##  Mode  :character   Mode  :character   Median :   12.00  
    ##                                        Mean   :   29.97  
    ##                                        3rd Qu.:   28.00  
    ##                                        Max.   :34238.00  
    ## 

## Example 2b: Austin BikeShare

List the top 5 percent of locations (stations) where bike-share trips begin and end.

Then, use `left_join()` to affect a database SQL join or table merge.

``` r
left_tbl <- bikeshare_trips %>% 
  count(start_station_name) %>% 
  slice_max(order_by = n, prop = .05)
left_tbl
```

    ## # Source:   SQL [10 x 2]
    ## # Database: BigQueryConnection
    ##    start_station_name                         n
    ##    <chr>                                  <int>
    ##  1 21st/Speedway @ PCL                    72127
    ##  2 21st & Speedway @PCL                   71145
    ##  3 Dean Keeton/Speedway                   47765
    ##  4 Zilker Park                            46722
    ##  5 Riverside/South Lamar                  32112
    ##  6 21st/Guadalupe                         31865
    ##  7 26th/Nueces                            31240
    ##  8 Rainey/Cummings                        30835
    ##  9 Guadalupe/West Mall @ University Co-op 29990
    ## 10 Dean Keeton/Whitis                     28322

``` r
right_tbl <- bikeshare_trips %>% 
  count(end_station_name) %>% 
  slice_max(order_by = n, prop = .05)
right_tbl
```

    ## # Source:   SQL [10 x 2]
    ## # Database: BigQueryConnection
    ##    end_station_name              n
    ##    <chr>                     <int>
    ##  1 "21st/Speedway @ PCL"     73330
    ##  2 "21st & Speedway @PCL"    73312
    ##  3 "Dean Keeton/Speedway"    51873
    ##  4 "Zilker Park"             50774
    ##  5 "21st/Guadalupe"          31708
    ##  6 "Riverside/South Lamar"   31544
    ##  7 "Dean Keeton & Speedway " 30131
    ##  8 "Rainey/Cummings"         29861
    ##  9 "2nd/Lavaca @ City Hall"  29565
    ## 10 "26th/Nueces"             29424

``` r
left_join(left_tbl, right_tbl, by = c("start_station_name" = "end_station_name")) # %>% show_query()
```

    ## # Source:   SQL [10 x 3]
    ## # Database: BigQueryConnection
    ##    start_station_name                       n_x   n_y
    ##    <chr>                                  <int> <int>
    ##  1 21st/Speedway @ PCL                    72127 73330
    ##  2 21st & Speedway @PCL                   71145 73312
    ##  3 Dean Keeton/Speedway                   47765 51873
    ##  4 Zilker Park                            46722 50774
    ##  5 Riverside/South Lamar                  32112 31544
    ##  6 21st/Guadalupe                         31865 31708
    ##  7 26th/Nueces                            31240 29424
    ##  8 Rainey/Cummings                        30835 29861
    ##  9 Guadalupe/West Mall @ University Co-op 29990    NA
    ## 10 Dean Keeton/Whitis                     28322    NA

## Example 3: NYC tree census

Inspired by [Edgar Ruiz’s dbplyr presentation](https://rstudio.com/resources/rstudioconf-2019/databases-using-r-the-latest/)

[DBplot](https://db.rstudio.com/dbplot/) leverages dplyr to process the calculations of a plot inside a database.

``` r
con_ny_trees <- dbConnect(
  bigrquery::bigquery(),
  project = "bigquery-public-data",
  dataset = "new_york_trees",
  billing = "fleet-furnace-316922"
)
con_ny_trees
```

    ## <BigQueryConnection>
    ##   Dataset: bigquery-public-data.new_york_trees
    ##   Billing: fleet-furnace-316922

Set the query pointer to `bigquery-public-data.ustin_bikeshare.bikesahre_trips` table

``` r
nytrees <- tbl(con_ny_trees, "tree_census_2015") 
```

### Visualize

``` r
library(dbplot)
library(leaflet)
```

``` r
glimpse(nytrees)
```

    ## Rows: ??
    ## Columns: 41
    ## Database: BigQueryConnection
    ## $ tree_id    <int> 80548, 449489, 449293, 449153, 449148, 68192, 68347, 68393,…
    ## $ block_id   <int> 502982, 503216, 503196, 503188, 503152, 503778, 503349, 503…
    ## $ created_at <date> 2015-07-20, 2015-11-12, 2015-11-12, 2015-11-12, 2015-11-12…
    ## $ tree_dbh   <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    ## $ stump_diam <int> 16, 4, 5, 10, 5, 3, 25, 2, 4, 3, 5, 8, 3, 4, 7, 6, 15, 12, …
    ## $ curb_loc   <chr> "OnCurb", "OnCurb", "OnCurb", "OnCurb", "OnCurb", "OnCurb",…
    ## $ status     <chr> "Stump", "Stump", "Stump", "Stump", "Stump", "Stump", "Stum…
    ## $ health     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ spc_latin  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ spc_common <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ steward    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ guards     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ sidewalk   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ user_type  <chr> "Volunteer", "NYC Parks Staff", "NYC Parks Staff", "NYC Par…
    ## $ problems   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ root_stone <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ root_grate <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ root_other <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ trunk_wire <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ trnk_light <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ trnk_other <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ brch_light <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ brch_shoe  <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ brch_other <chr> "No", "No", "No", "No", "No", "No", "No", "No", "No", "No",…
    ## $ address    <chr> "192 EAST 164 STREET", "1007 SUMMIT AVENUE", "80 WEST 163 S…
    ## $ zipcode    <int> 10451, 10452, 10452, 10452, 10452, 10452, 10452, 10452, 104…
    ## $ zip_city   <chr> "Bronx", "Bronx", "Bronx", "Bronx", "Bronx", "Bronx", "Bron…
    ## $ cb_num     <int> 204, 204, 204, 204, 204, 204, 204, 204, 204, 204, 204, 204,…
    ## $ borocode   <int> 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2,…
    ## $ boroname   <chr> "Bronx", "Bronx", "Bronx", "Bronx", "Bronx", "Bronx", "Bron…
    ## $ cncldist   <int> 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8,…
    ## $ st_assem   <int> 77, 77, 84, 84, 84, 84, 77, 77, 84, 84, 84, 77, 77, 77, 77,…
    ## $ st_senate  <int> 32, 29, 29, 29, 29, 29, 29, 29, 29, 29, 29, 29, 29, 29, 29,…
    ## $ nta        <chr> "BX14", "BX26", "BX26", "BX26", "BX26", "BX26", "BX26", "BX…
    ## $ nta_name   <chr> "East Concourse-Concourse Village", "Highbridge", "Highbrid…
    ## $ boro_ct    <int> 2018301, 2018900, 2018900, 2018900, 2018900, 2018900, 20189…
    ## $ state      <chr> "New York", "New York", "New York", "New York", "New York",…
    ## $ latitude   <dbl> 40.82914, 40.83350, 40.83205, 40.83309, 40.83284, 40.82999,…
    ## $ longitude  <dbl> -73.91993, -73.93010, -73.92889, -73.92845, -73.92951, -73.…
    ## $ x_sp       <dbl> 1006410, 1003594, 1003927, 1004049, 1003757, 1003159, 10047…
    ## $ y_sp       <dbl> 241366.5, 242953.0, 242422.2, 242803.5, 242711.3, 241672.5,…

``` r
nytrees %>% 
  count(curb_loc)
```

    ## # Source:   SQL [2 x 2]
    ## # Database: BigQueryConnection
    ##   curb_loc            n
    ##   <chr>           <int>
    ## 1 OnCurb         656896
    ## 2 OffsetFromCurb  26892

#### dbplot_raster()

Above, we **subset the data at the database** to *good and healthy trees*, *offset from the curb*. Now use the `dbpot_raster()` function to **plot in R**

``` r
locations <- nytrees %>% 
  filter(curb_loc == "OffsetFromCurb") %>% 
  filter(health == "Good") %>% 
  dbplot_raster(longitude, latitude, resolution = 30)
  # db_compute_raster(longitude, latitude, resolution = 30)
locations
```

<img src="/post/2023-08-14-BigQuery/BigQuery-Practice_files/figure-html/unnamed-chunk-37-1.png" width="672" />

#### db_compute_raster()

``` r
nytrees %>% 
  filter(curb_loc == "OffsetFromCurb") %>% 
  filter(health == "Good") %>% 
  count()
```

    ## # Source:   SQL [1 x 1]
    ## # Database: BigQueryConnection
    ##       n
    ##   <int>
    ## 1 20877

``` r
locations_tbl <- nytrees %>% 
  filter(curb_loc == "OffsetFromCurb") %>% 
  filter(health == "Good") %>% 
  db_compute_raster(longitude, latitude, resolution = 30)
locations_tbl
```

    ## # A tibble: 364 × 3
    ##    longitude latitude `n()`
    ##        <dbl>    <dbl> <dbl>
    ##  1     -73.8     40.8    16
    ##  2     -73.8     40.7   281
    ##  3     -73.9     40.7    60
    ##  4     -73.8     40.7    49
    ##  5     -73.8     40.7    34
    ##  6     -73.8     40.7    46
    ##  7     -73.7     40.7    27
    ##  8     -73.8     40.6     5
    ##  9     -74.1     40.6   129
    ## 10     -74.0     40.6    60
    ## # ℹ 354 more rows

#### Tidy evaluation functions

Tidy eval function [by Ruiz (timestamp 13:47)](https://rstudio.com/resources/rstudioconf-2019/databases-using-r-the-latest/)

``` r
size <- function(df, field) {
  field <- enquo(field)
  df %>% 
    arrange(!! field) %>% 
    mutate(diff = !! field - lag(!! field)) %>% 
    filter(diff > 0) %>% 
    summarise(min(diff)) %>% 
    pull()
}
```

``` r
lon_size <- locations_tbl %>% 
  size(longitude)

lon_size
```

    ## [1] 0.01839551

``` r
lat_size <- locations_tbl %>% 
  size(latitude)

lat_size
```

    ## [1] 0.01373925

``` r
sq <- locations_tbl %>% 
  mutate(lon1 = longitude,
         lon2 = longitude + lon_size,
         lat1 = latitude,
         lat2 = latitude + lat_size,
         of_max = `n()` / max(`n()`)
  )
```

#### Leaflet map

``` r
leaflet() %>% 
  addTiles() %>% 
  addRectangles(
    sq$lon1, sq$lat1, sq$lon2, sq$lat2
  )
```

<div class="leaflet html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-1" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"https://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"https://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addRectangles","args":[[40.81487422999999,40.73243873,40.73243873,40.70496023,40.69122098,40.71869948,40.65000323,40.58130698,40.62252472999999,40.63626398,40.59504623,40.75991723,40.75991723,40.75991723,40.51261073,40.54008923,40.55382848,40.54008923,40.82861347999999,40.80113498,40.88357048,40.77365648,40.71869948,40.67748173,40.59504623,40.60878547999999,40.54008923,40.74617798,40.70496023,40.66374248,40.74617798,40.78739573,40.55382848,40.88357048,40.58130698,40.63626398,40.77365648,40.69122098,40.88357048,40.56756773,40.60878547999999,40.67748173,40.59504623,40.56756773,40.60878547999999,40.74617798,40.75991723,40.74617798,40.69122098,40.69122098,40.66374248,40.67748173,40.65000323,40.56756773,40.69122098,40.70496023,40.70496023,40.62252472999999,40.84235272999999,40.73243873,40.62252472999999,40.60878547999999,40.63626398,40.66374248,40.67748173,40.60878547999999,40.60878547999999,40.49887148,40.89730973,40.74617798,40.86983123,40.56756773,40.84235272999999,40.78739573,40.69122098,40.89730973,40.62252472999999,40.60878547999999,40.63626398,40.69122098,40.84235272999999,40.84235272999999,40.82861347999999,40.82861347999999,40.73243873,40.67748173,40.67748173,40.70496023,40.59504623,40.67748173,40.73243873,40.82861347999999,40.81487422999999,40.51261073,40.51261073,40.52634998,40.52634998,40.77365648,40.65000323,40.71869948,40.59504623,40.55382848,40.74617798,40.55382848,40.59504623,40.75991723,40.71869948,40.60878547999999,40.73243873,40.69122098,40.58130698,40.75991723,40.84235272999999,40.78739573,40.60878547999999,40.62252472999999,40.73243873,40.65000323,40.51261073,40.81487422999999,40.81487422999999,40.74617798,40.70496023,40.63626398,40.63626398,40.77365648,40.74617798,40.73243873,40.65000323,40.58130698,40.63626398,40.80113498,40.59504623,40.59504623,40.60878547999999,40.71869948,40.56756773,40.73243873,40.67748173,40.62252472999999,40.58130698,40.69122098,40.59504623,40.85609198,40.75991723,40.66374248,40.67748173,40.70496023,40.71869948,40.69122098,40.62252472999999,40.56756773,40.71869948,40.77365648,40.78739573,40.62252472999999,40.58130698,40.58130698,40.80113498,40.75991723,40.65000323,40.66374248,40.62252472999999,40.73243873,40.59504623,40.51261073,40.88357048,40.85609198,40.77365648,40.62252472999999,40.81487422999999,40.58130698,40.49887148,40.75991723,40.67748173,40.56756773,40.58130698,40.60878547999999,40.58130698,40.54008923,40.82861347999999,40.49887148,40.86983123,40.88357048,40.88357048,40.86983123,40.70496023,40.58130698,40.69122098,40.66374248,40.67748173,40.65000323,40.59504623,40.60878547999999,40.70496023,40.73243873,40.80113498,40.54008923,40.82861347999999,40.84235272999999,40.85609198,40.85609198,40.80113498,40.77365648,40.70496023,40.66374248,40.66374248,40.69122098,40.60878547999999,40.59504623,40.71869948,40.70496023,40.66374248,40.56756773,40.54008923,40.59504623,40.70496023,40.80113498,40.73243873,40.58130698,40.58130698,40.71869948,40.60878547999999,40.80113498,40.58130698,40.89730973,40.75991723,40.81487422999999,40.75991723,40.75991723,40.71869948,40.71869948,40.71869948,40.62252472999999,40.52634998,40.55382848,40.74617798,40.77365648,40.74617798,40.71869948,40.74617798,40.73243873,40.59504623,40.63626398,40.56756773,40.55382848,40.80113498,40.82861347999999,40.63626398,40.81487422999999,40.65000323,40.60878547999999,40.89730973,40.59504623,40.56756773,40.70496023,40.84235272999999,40.89730973,40.85609198,40.82861347999999,40.74617798,40.73243873,40.74617798,40.71869948,40.74617798,40.66374248,40.70496023,40.66374248,40.65000323,40.66374248,40.65000323,40.63626398,40.69122098,40.74617798,40.85609198,40.59504623,40.56756773,40.56756773,40.81487422999999,40.77365648,40.67748173,40.65000323,40.62252472999999,40.63626398,40.82861347999999,40.70496023,40.65000323,40.62252472999999,40.65000323,40.62252472999999,40.58130698,40.86983123,40.59504623,40.67748173,40.67748173,40.70496023,40.75991723,40.65000323,40.77365648,40.86983123,40.85609198,40.81487422999999,40.77365648,40.78739573,40.73243873,40.69122098,40.71869948,40.63626398,40.60878547999999,40.78739573,40.80113498,40.78739573,40.55382848,40.69122098,40.59504623,40.78739573,40.58130698,40.58130698,40.66374248,40.66374248,40.52634998,40.75991723,40.73243873,40.67748173,40.62252472999999,40.80113498,40.86983123,40.84235272999999,40.84235272999999,40.75991723,40.73243873,40.69122098,40.69122098,40.70496023,40.63626398,40.60878547999999,40.60878547999999,40.58130698,40.71869948,40.74617798,40.77365648,40.62252472999999,40.63626398,40.63626398,40.62252472999999,40.74617798,40.67748173,40.66374248,40.67748173,40.77365648,40.75991723,40.63626398,40.52634998,40.66374248,40.58130698,40.59504623,40.88357048,40.78739573,40.56756773,40.60878547999999,40.78739573,40.58130698,40.63626398,40.65000323],[-73.81176120000001,-73.79336569,-73.88534324,-73.77497018000001,-73.77497018000001,-73.84855222,-73.73817916,-73.75657467000001,-74.05090283,-74.03250731999999,-73.95892528,-73.9957163,-73.95892528,-73.97732078999999,-74.21646242,-74.17967139999999,-74.21646242,-74.16127589,-73.94052977,-73.94052977,-73.84855222,-73.83015671,-73.92213426000001,-73.94052977,-73.9957163,-74.08769384999999,-74.19806690999999,-73.75657467000001,-73.90373875,-73.95892528,-73.9957163,-73.9957163,-74.19806690999999,-73.92213426000001,-73.84855222,-74.05090283,-73.77497018000001,-73.75657467000001,-73.83015671,-73.84855222,-74.03250731999999,-74.03250731999999,-73.92213426000001,-74.08769384999999,-73.83015671,-73.84855222,-73.88534324,-73.92213426000001,-73.83015671,-73.90373875,-73.73817916,-73.88534324,-73.95892528,-73.95892528,-74.03250731999999,-74.03250731999999,-73.9957163,-74.08769384999999,-73.83015671,-73.77497018000001,-73.92213426000001,-74.10608936,-74.12448487,-73.83015671,-73.79336569,-74.17967139999999,-74.16127589,-74.25325343999999,-73.90373875,-73.95892528,-73.88534324,-73.86694773000001,-73.79336569,-73.79336569,-73.73817916,-73.84855222,-74.01411181,-73.9957163,-73.84855222,-73.88534324,-73.90373875,-73.94052977,-73.88534324,-73.86694773000001,-73.92213426000001,-73.75657467000001,-73.77497018000001,-73.83015671,-73.75657467000001,-73.97732078999999,-73.97732078999999,-73.95892528,-73.94052977,-74.19806690999999,-74.25325343999999,-74.19806690999999,-74.16127589,-73.84855222,-73.92213426000001,-73.9957163,-74.08769384999999,-74.14288037999999,-73.71978365000001,-74.12448487,-74.10608936,-73.75657467000001,-73.73817916,-74.12448487,-73.71978365000001,-73.84855222,-73.90373875,-73.94052977,-73.84855222,-73.92213426000001,-74.06929834,-74.19806690999999,-73.84855222,-73.81176120000001,-74.17967139999999,-73.90373875,-73.88534324,-73.94052977,-73.75657467000001,-73.88534324,-73.95892528,-73.9957163,-73.79336569,-73.86694773000001,-73.90373875,-74.01411181,-74.17967139999999,-73.86694773000001,-74.03250731999999,-74.17967139999999,-74.14288037999999,-73.75657467000001,-73.97732078999999,-73.90373875,-73.9957163,-74.10608936,-73.79336569,-73.86694773000001,-73.81176120000001,-73.86694773000001,-73.73817916,-73.75657467000001,-73.81176120000001,-73.88534324,-73.88534324,-73.97732078999999,-74.03250731999999,-74.01411181,-74.01411181,-73.95892528,-73.94052977,-74.17967139999999,-74.16127589,-74.10608936,-73.83015671,-73.84855222,-73.84855222,-74.01411181,-73.9957163,-74.01411181,-74.12448487,-74.23485792999999,-73.81176120000001,-73.88534324,-73.86694773000001,-73.94052977,-73.95892528,-74.08769384999999,-74.23485792999999,-73.81176120000001,-73.92213426000001,-74.16127589,-73.83015671,-73.77497018000001,-74.12448487,-74.23485792999999,-73.79336569,-74.21646242,-73.90373875,-73.90373875,-73.88534324,-73.86694773000001,-73.73817916,-73.81176120000001,-73.95892528,-73.97732078999999,-73.90373875,-73.88534324,-73.94052977,-73.95892528,-74.01411181,-73.9957163,-73.95892528,-74.14288037999999,-73.83015671,-73.88534324,-73.92213426000001,-73.90373875,-73.88534324,-73.75657467000001,-73.79336569,-73.77497018000001,-73.81176120000001,-73.92213426000001,-73.94052977,-74.19806690999999,-73.86694773000001,-73.84855222,-73.92213426000001,-73.94052977,-74.21646242,-73.79336569,-73.92213426000001,-73.90373875,-73.73817916,-73.94052977,-74.06929834,-73.81176120000001,-74.01411181,-73.84855222,-74.19806690999999,-73.92213426000001,-74.01411181,-73.92213426000001,-73.79336569,-73.77497018000001,-73.79336569,-73.90373875,-73.94052977,-74.14288037999999,-74.23485792999999,-74.17967139999999,-73.77497018000001,-73.81176120000001,-73.88534324,-73.83015671,-73.90373875,-73.94052977,-73.77497018000001,-73.9957163,-73.9957163,-74.10608936,-73.92213426000001,-73.90373875,-73.92213426000001,-73.84855222,-73.77497018000001,-74.05090283,-73.88534324,-74.14288037999999,-74.12448487,-73.94052977,-73.95892528,-73.86694773000001,-73.83015671,-73.84855222,-73.81176120000001,-73.83015671,-73.86694773000001,-73.77497018000001,-73.73817916,-73.79336569,-73.97732078999999,-73.94052977,-73.9957163,-73.9957163,-73.97732078999999,-73.97732078999999,-74.01411181,-74.01411181,-73.94052977,-74.06929834,-74.14288037999999,-74.10608936,-73.83015671,-73.90373875,-73.86694773000001,-74.01411181,-73.95892528,-74.14288037999999,-73.92213426000001,-73.81176120000001,-73.94052977,-73.97732078999999,-73.75657467000001,-73.88534324,-73.86694773000001,-73.83015671,-73.97732078999999,-73.83015671,-73.73817916,-73.86694773000001,-73.86694773000001,-73.86694773000001,-73.92213426000001,-73.84855222,-73.84855222,-73.86694773000001,-73.79336569,-73.81176120000001,-73.81176120000001,-73.9957163,-73.95892528,-74.01411181,-73.97732078999999,-73.97732078999999,-73.97732078999999,-73.95892528,-74.16127589,-73.81176120000001,-74.01411181,-73.84855222,-73.9957163,-74.17967139999999,-73.84855222,-73.88534324,-74.17967139999999,-73.83015671,-73.95892528,-73.84855222,-73.90373875,-73.81176120000001,-73.92213426000001,-73.86694773000001,-73.92213426000001,-73.90373875,-73.75657467000001,-73.79336569,-73.94052977,-73.95892528,-73.90373875,-73.90373875,-73.92213426000001,-73.97732078999999,-73.97732078999999,-73.97732078999999,-73.97732078999999,-74.16127589,-74.08769384999999,-74.10608936,-74.12448487,-73.83015671,-73.95892528,-73.86694773000001,-74.01411181,-73.94052977,-73.92213426000001,-73.94052977,-74.21646242,-73.90373875,-73.95892528,-74.16127589,-73.86694773000001,-73.83015671,-74.17967139999999,-73.75657467000001,-73.86694773000001,-73.77497018000001,-74.16127589,-74.03250731999999],[40.82861347999999,40.74617797999999,40.74617797999999,40.71869947999999,40.70496022999999,40.73243872999999,40.66374247999999,40.59504622999999,40.63626397999999,40.65000322999999,40.60878547999999,40.77365647999999,40.77365647999999,40.77365647999999,40.52634997999999,40.55382847999999,40.56756772999999,40.55382847999999,40.84235272999999,40.81487422999999,40.89730972999999,40.78739572999999,40.73243872999999,40.69122097999999,40.60878547999999,40.62252472999999,40.55382847999999,40.75991722999999,40.71869947999999,40.67748172999999,40.75991722999999,40.80113497999999,40.56756772999999,40.89730972999999,40.59504622999999,40.65000322999999,40.78739572999999,40.70496022999999,40.89730972999999,40.58130697999999,40.62252472999999,40.69122097999999,40.60878547999999,40.58130697999999,40.62252472999999,40.75991722999999,40.77365647999999,40.75991722999999,40.70496022999999,40.70496022999999,40.67748172999999,40.69122097999999,40.66374247999999,40.58130697999999,40.70496022999999,40.71869947999999,40.71869947999999,40.63626397999999,40.85609197999999,40.74617797999999,40.63626397999999,40.62252472999999,40.65000322999999,40.67748172999999,40.69122097999999,40.62252472999999,40.62252472999999,40.51261072999999,40.91104897999999,40.75991722999999,40.88357047999999,40.58130697999999,40.85609197999999,40.80113497999999,40.70496022999999,40.91104897999999,40.63626397999999,40.62252472999999,40.65000322999999,40.70496022999999,40.85609197999999,40.85609197999999,40.84235272999999,40.84235272999999,40.74617797999999,40.69122097999999,40.69122097999999,40.71869947999999,40.60878547999999,40.69122097999999,40.74617797999999,40.84235272999999,40.82861347999999,40.52634997999999,40.52634997999999,40.54008922999999,40.54008922999999,40.78739572999999,40.66374247999999,40.73243872999999,40.60878547999999,40.56756772999999,40.75991722999999,40.56756772999999,40.60878547999999,40.77365647999999,40.73243872999999,40.62252472999999,40.74617797999999,40.70496022999999,40.59504622999999,40.77365647999999,40.85609197999999,40.80113497999999,40.62252472999999,40.63626397999999,40.74617797999999,40.66374247999999,40.52634997999999,40.82861347999999,40.82861347999999,40.75991722999999,40.71869947999999,40.65000322999999,40.65000322999999,40.78739572999999,40.75991722999999,40.74617797999999,40.66374247999999,40.59504622999999,40.65000322999999,40.81487422999999,40.60878547999999,40.60878547999999,40.62252472999999,40.73243872999999,40.58130697999999,40.74617797999999,40.69122097999999,40.63626397999999,40.59504622999999,40.70496022999999,40.60878547999999,40.86983122999999,40.77365647999999,40.67748172999999,40.69122097999999,40.71869947999999,40.73243872999999,40.70496022999999,40.63626397999999,40.58130697999999,40.73243872999999,40.78739572999999,40.80113497999999,40.63626397999999,40.59504622999999,40.59504622999999,40.81487422999999,40.77365647999999,40.66374247999999,40.67748172999999,40.63626397999999,40.74617797999999,40.60878547999999,40.52634997999999,40.89730972999999,40.86983122999999,40.78739572999999,40.63626397999999,40.82861347999999,40.59504622999999,40.51261072999999,40.77365647999999,40.69122097999999,40.58130697999999,40.59504622999999,40.62252472999999,40.59504622999999,40.55382847999999,40.84235272999999,40.51261072999999,40.88357047999999,40.89730972999999,40.89730972999999,40.88357047999999,40.71869947999999,40.59504622999999,40.70496022999999,40.67748172999999,40.69122097999999,40.66374247999999,40.60878547999999,40.62252472999999,40.71869947999999,40.74617797999999,40.81487422999999,40.55382847999999,40.84235272999999,40.85609197999999,40.86983122999999,40.86983122999999,40.81487422999999,40.78739572999999,40.71869947999999,40.67748172999999,40.67748172999999,40.70496022999999,40.62252472999999,40.60878547999999,40.73243872999999,40.71869947999999,40.67748172999999,40.58130697999999,40.55382847999999,40.60878547999999,40.71869947999999,40.81487422999999,40.74617797999999,40.59504622999999,40.59504622999999,40.73243872999999,40.62252472999999,40.81487422999999,40.59504622999999,40.91104897999999,40.77365647999999,40.82861347999999,40.77365647999999,40.77365647999999,40.73243872999999,40.73243872999999,40.73243872999999,40.63626397999999,40.54008922999999,40.56756772999999,40.75991722999999,40.78739572999999,40.75991722999999,40.73243872999999,40.75991722999999,40.74617797999999,40.60878547999999,40.65000322999999,40.58130697999999,40.56756772999999,40.81487422999999,40.84235272999999,40.65000322999999,40.82861347999999,40.66374247999999,40.62252472999999,40.91104897999999,40.60878547999999,40.58130697999999,40.71869947999999,40.85609197999999,40.91104897999999,40.86983122999999,40.84235272999999,40.75991722999999,40.74617797999999,40.75991722999999,40.73243872999999,40.75991722999999,40.67748172999999,40.71869947999999,40.67748172999999,40.66374247999999,40.67748172999999,40.66374247999999,40.65000322999999,40.70496022999999,40.75991722999999,40.86983122999999,40.60878547999999,40.58130697999999,40.58130697999999,40.82861347999999,40.78739572999999,40.69122097999999,40.66374247999999,40.63626397999999,40.65000322999999,40.84235272999999,40.71869947999999,40.66374247999999,40.63626397999999,40.66374247999999,40.63626397999999,40.59504622999999,40.88357047999999,40.60878547999999,40.69122097999999,40.69122097999999,40.71869947999999,40.77365647999999,40.66374247999999,40.78739572999999,40.88357047999999,40.86983122999999,40.82861347999999,40.78739572999999,40.80113497999999,40.74617797999999,40.70496022999999,40.73243872999999,40.65000322999999,40.62252472999999,40.80113497999999,40.81487422999999,40.80113497999999,40.56756772999999,40.70496022999999,40.60878547999999,40.80113497999999,40.59504622999999,40.59504622999999,40.67748172999999,40.67748172999999,40.54008922999999,40.77365647999999,40.74617797999999,40.69122097999999,40.63626397999999,40.81487422999999,40.88357047999999,40.85609197999999,40.85609197999999,40.77365647999999,40.74617797999999,40.70496022999999,40.70496022999999,40.71869947999999,40.65000322999999,40.62252472999999,40.62252472999999,40.59504622999999,40.73243872999999,40.75991722999999,40.78739572999999,40.63626397999999,40.65000322999999,40.65000322999999,40.63626397999999,40.75991722999999,40.69122097999999,40.67748172999999,40.69122097999999,40.78739572999999,40.77365647999999,40.65000322999999,40.54008922999999,40.67748172999999,40.59504622999999,40.60878547999999,40.89730972999999,40.80113497999999,40.58130697999999,40.62252472999999,40.80113497999999,40.59504622999999,40.65000322999999,40.66374247999999],[-73.79336569000002,-73.77497018000001,-73.86694773000001,-73.75657467000002,-73.75657467000002,-73.83015671000001,-73.71978365000001,-73.73817916000002,-74.03250732000001,-74.01411181,-73.94052977000001,-73.97732079000001,-73.94052977000001,-73.95892528,-74.19806691000001,-74.16127589,-74.19806691000001,-74.14288038000001,-73.92213426000001,-73.92213426000001,-73.83015671000001,-73.81176120000001,-73.90373875000002,-73.92213426000001,-73.97732079000001,-74.06929834,-74.1796714,-73.73817916000002,-73.88534324000001,-73.94052977000001,-73.97732079000001,-73.97732079000001,-74.1796714,-73.90373875000002,-73.83015671000001,-74.03250732000001,-73.75657467000002,-73.73817916000002,-73.81176120000001,-73.83015671000001,-74.01411181,-74.01411181,-73.90373875000002,-74.06929834,-73.81176120000001,-73.83015671000001,-73.86694773000001,-73.90373875000002,-73.81176120000001,-73.88534324000001,-73.71978365000001,-73.86694773000001,-73.94052977000001,-73.94052977000001,-74.01411181,-74.01411181,-73.97732079000001,-74.06929834,-73.81176120000001,-73.75657467000002,-73.90373875000002,-74.08769385000001,-74.10608936000001,-73.81176120000001,-73.77497018000001,-74.16127589,-74.14288038000001,-74.23485793,-73.88534324000001,-73.94052977000001,-73.86694773000001,-73.84855222000002,-73.77497018000001,-73.77497018000001,-73.71978365000001,-73.83015671000001,-73.99571630000001,-73.97732079000001,-73.83015671000001,-73.86694773000001,-73.88534324000001,-73.92213426000001,-73.86694773000001,-73.84855222000002,-73.90373875000002,-73.73817916000002,-73.75657467000002,-73.81176120000001,-73.73817916000002,-73.95892528,-73.95892528,-73.94052977000001,-73.92213426000001,-74.1796714,-74.23485793,-74.1796714,-74.14288038000001,-73.83015671000001,-73.90373875000002,-73.97732079000001,-74.06929834,-74.12448487,-73.70138814000002,-74.10608936000001,-74.08769385000001,-73.73817916000002,-73.71978365000001,-74.10608936000001,-73.70138814000002,-73.83015671000001,-73.88534324000001,-73.92213426000001,-73.83015671000001,-73.90373875000002,-74.05090283000001,-74.1796714,-73.83015671000001,-73.79336569000002,-74.16127589,-73.88534324000001,-73.86694773000001,-73.92213426000001,-73.73817916000002,-73.86694773000001,-73.94052977000001,-73.97732079000001,-73.77497018000001,-73.84855222000002,-73.88534324000001,-73.99571630000001,-74.16127589,-73.84855222000002,-74.01411181,-74.16127589,-74.12448487,-73.73817916000002,-73.95892528,-73.88534324000001,-73.97732079000001,-74.08769385000001,-73.77497018000001,-73.84855222000002,-73.79336569000002,-73.84855222000002,-73.71978365000001,-73.73817916000002,-73.79336569000002,-73.86694773000001,-73.86694773000001,-73.95892528,-74.01411181,-73.99571630000001,-73.99571630000001,-73.94052977000001,-73.92213426000001,-74.16127589,-74.14288038000001,-74.08769385000001,-73.81176120000001,-73.83015671000001,-73.83015671000001,-73.99571630000001,-73.97732079000001,-73.99571630000001,-74.10608936000001,-74.21646242,-73.79336569000002,-73.86694773000001,-73.84855222000002,-73.92213426000001,-73.94052977000001,-74.06929834,-74.21646242,-73.79336569000002,-73.90373875000002,-74.14288038000001,-73.81176120000001,-73.75657467000002,-74.10608936000001,-74.21646242,-73.77497018000001,-74.19806691000001,-73.88534324000001,-73.88534324000001,-73.86694773000001,-73.84855222000002,-73.71978365000001,-73.79336569000002,-73.94052977000001,-73.95892528,-73.88534324000001,-73.86694773000001,-73.92213426000001,-73.94052977000001,-73.99571630000001,-73.97732079000001,-73.94052977000001,-74.12448487,-73.81176120000001,-73.86694773000001,-73.90373875000002,-73.88534324000001,-73.86694773000001,-73.73817916000002,-73.77497018000001,-73.75657467000002,-73.79336569000002,-73.90373875000002,-73.92213426000001,-74.1796714,-73.84855222000002,-73.83015671000001,-73.90373875000002,-73.92213426000001,-74.19806691000001,-73.77497018000001,-73.90373875000002,-73.88534324000001,-73.71978365000001,-73.92213426000001,-74.05090283000001,-73.79336569000002,-73.99571630000001,-73.83015671000001,-74.1796714,-73.90373875000002,-73.99571630000001,-73.90373875000002,-73.77497018000001,-73.75657467000002,-73.77497018000001,-73.88534324000001,-73.92213426000001,-74.12448487,-74.21646242,-74.16127589,-73.75657467000002,-73.79336569000002,-73.86694773000001,-73.81176120000001,-73.88534324000001,-73.92213426000001,-73.75657467000002,-73.97732079000001,-73.97732079000001,-74.08769385000001,-73.90373875000002,-73.88534324000001,-73.90373875000002,-73.83015671000001,-73.75657467000002,-74.03250732000001,-73.86694773000001,-74.12448487,-74.10608936000001,-73.92213426000001,-73.94052977000001,-73.84855222000002,-73.81176120000001,-73.83015671000001,-73.79336569000002,-73.81176120000001,-73.84855222000002,-73.75657467000002,-73.71978365000001,-73.77497018000001,-73.95892528,-73.92213426000001,-73.97732079000001,-73.97732079000001,-73.95892528,-73.95892528,-73.99571630000001,-73.99571630000001,-73.92213426000001,-74.05090283000001,-74.12448487,-74.08769385000001,-73.81176120000001,-73.88534324000001,-73.84855222000002,-73.99571630000001,-73.94052977000001,-74.12448487,-73.90373875000002,-73.79336569000002,-73.92213426000001,-73.95892528,-73.73817916000002,-73.86694773000001,-73.84855222000002,-73.81176120000001,-73.95892528,-73.81176120000001,-73.71978365000001,-73.84855222000002,-73.84855222000002,-73.84855222000002,-73.90373875000002,-73.83015671000001,-73.83015671000001,-73.84855222000002,-73.77497018000001,-73.79336569000002,-73.79336569000002,-73.97732079000001,-73.94052977000001,-73.99571630000001,-73.95892528,-73.95892528,-73.95892528,-73.94052977000001,-74.14288038000001,-73.79336569000002,-73.99571630000001,-73.83015671000001,-73.97732079000001,-74.16127589,-73.83015671000001,-73.86694773000001,-74.16127589,-73.81176120000001,-73.94052977000001,-73.83015671000001,-73.88534324000001,-73.79336569000002,-73.90373875000002,-73.84855222000002,-73.90373875000002,-73.88534324000001,-73.73817916000002,-73.77497018000001,-73.92213426000001,-73.94052977000001,-73.88534324000001,-73.88534324000001,-73.90373875000002,-73.95892528,-73.95892528,-73.95892528,-73.95892528,-74.14288038000001,-74.06929834,-74.08769385000001,-74.10608936000001,-73.81176120000001,-73.94052977000001,-73.84855222000002,-73.99571630000001,-73.92213426000001,-73.90373875000002,-73.92213426000001,-74.19806691000001,-73.88534324000001,-73.94052977000001,-74.14288038000001,-73.84855222000002,-73.81176120000001,-74.16127589,-73.73817916000002,-73.84855222000002,-73.75657467000002,-74.14288038000001,-74.01411181],null,null,{"interactive":true,"className":"","stroke":true,"color":"#03F","weight":5,"opacity":0.5,"fill":true,"fillColor":"#03F","fillOpacity":0.2,"smoothFactor":1,"noClip":false},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[40.49887148,40.91104897999999],"lng":[-74.25325343999999,-73.70138814000002]}},"evals":[],"jsHooks":[]}</script>

``` r
fancy <- leaflet() %>% 
  addTiles() %>% 
  addProviderTiles(providers$CartoDB.Positron) %>% 
  addRectangles(
    sq$lon1, sq$lat1, sq$lon2, sq$lat2,
    fillOpacity = sq$of_max,
    fillColor = "forestgreen",
    stroke = FALSE,
    popup = glue::glue('Trees {sq$`n()`}')
  )

fancy
```

<div class="leaflet html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-2" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"https://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"https://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addProviderTiles","args":["CartoDB.Positron",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addRectangles","args":[[40.81487422999999,40.73243873,40.73243873,40.70496023,40.69122098,40.71869948,40.65000323,40.58130698,40.62252472999999,40.63626398,40.59504623,40.75991723,40.75991723,40.75991723,40.51261073,40.54008923,40.55382848,40.54008923,40.82861347999999,40.80113498,40.88357048,40.77365648,40.71869948,40.67748173,40.59504623,40.60878547999999,40.54008923,40.74617798,40.70496023,40.66374248,40.74617798,40.78739573,40.55382848,40.88357048,40.58130698,40.63626398,40.77365648,40.69122098,40.88357048,40.56756773,40.60878547999999,40.67748173,40.59504623,40.56756773,40.60878547999999,40.74617798,40.75991723,40.74617798,40.69122098,40.69122098,40.66374248,40.67748173,40.65000323,40.56756773,40.69122098,40.70496023,40.70496023,40.62252472999999,40.84235272999999,40.73243873,40.62252472999999,40.60878547999999,40.63626398,40.66374248,40.67748173,40.60878547999999,40.60878547999999,40.49887148,40.89730973,40.74617798,40.86983123,40.56756773,40.84235272999999,40.78739573,40.69122098,40.89730973,40.62252472999999,40.60878547999999,40.63626398,40.69122098,40.84235272999999,40.84235272999999,40.82861347999999,40.82861347999999,40.73243873,40.67748173,40.67748173,40.70496023,40.59504623,40.67748173,40.73243873,40.82861347999999,40.81487422999999,40.51261073,40.51261073,40.52634998,40.52634998,40.77365648,40.65000323,40.71869948,40.59504623,40.55382848,40.74617798,40.55382848,40.59504623,40.75991723,40.71869948,40.60878547999999,40.73243873,40.69122098,40.58130698,40.75991723,40.84235272999999,40.78739573,40.60878547999999,40.62252472999999,40.73243873,40.65000323,40.51261073,40.81487422999999,40.81487422999999,40.74617798,40.70496023,40.63626398,40.63626398,40.77365648,40.74617798,40.73243873,40.65000323,40.58130698,40.63626398,40.80113498,40.59504623,40.59504623,40.60878547999999,40.71869948,40.56756773,40.73243873,40.67748173,40.62252472999999,40.58130698,40.69122098,40.59504623,40.85609198,40.75991723,40.66374248,40.67748173,40.70496023,40.71869948,40.69122098,40.62252472999999,40.56756773,40.71869948,40.77365648,40.78739573,40.62252472999999,40.58130698,40.58130698,40.80113498,40.75991723,40.65000323,40.66374248,40.62252472999999,40.73243873,40.59504623,40.51261073,40.88357048,40.85609198,40.77365648,40.62252472999999,40.81487422999999,40.58130698,40.49887148,40.75991723,40.67748173,40.56756773,40.58130698,40.60878547999999,40.58130698,40.54008923,40.82861347999999,40.49887148,40.86983123,40.88357048,40.88357048,40.86983123,40.70496023,40.58130698,40.69122098,40.66374248,40.67748173,40.65000323,40.59504623,40.60878547999999,40.70496023,40.73243873,40.80113498,40.54008923,40.82861347999999,40.84235272999999,40.85609198,40.85609198,40.80113498,40.77365648,40.70496023,40.66374248,40.66374248,40.69122098,40.60878547999999,40.59504623,40.71869948,40.70496023,40.66374248,40.56756773,40.54008923,40.59504623,40.70496023,40.80113498,40.73243873,40.58130698,40.58130698,40.71869948,40.60878547999999,40.80113498,40.58130698,40.89730973,40.75991723,40.81487422999999,40.75991723,40.75991723,40.71869948,40.71869948,40.71869948,40.62252472999999,40.52634998,40.55382848,40.74617798,40.77365648,40.74617798,40.71869948,40.74617798,40.73243873,40.59504623,40.63626398,40.56756773,40.55382848,40.80113498,40.82861347999999,40.63626398,40.81487422999999,40.65000323,40.60878547999999,40.89730973,40.59504623,40.56756773,40.70496023,40.84235272999999,40.89730973,40.85609198,40.82861347999999,40.74617798,40.73243873,40.74617798,40.71869948,40.74617798,40.66374248,40.70496023,40.66374248,40.65000323,40.66374248,40.65000323,40.63626398,40.69122098,40.74617798,40.85609198,40.59504623,40.56756773,40.56756773,40.81487422999999,40.77365648,40.67748173,40.65000323,40.62252472999999,40.63626398,40.82861347999999,40.70496023,40.65000323,40.62252472999999,40.65000323,40.62252472999999,40.58130698,40.86983123,40.59504623,40.67748173,40.67748173,40.70496023,40.75991723,40.65000323,40.77365648,40.86983123,40.85609198,40.81487422999999,40.77365648,40.78739573,40.73243873,40.69122098,40.71869948,40.63626398,40.60878547999999,40.78739573,40.80113498,40.78739573,40.55382848,40.69122098,40.59504623,40.78739573,40.58130698,40.58130698,40.66374248,40.66374248,40.52634998,40.75991723,40.73243873,40.67748173,40.62252472999999,40.80113498,40.86983123,40.84235272999999,40.84235272999999,40.75991723,40.73243873,40.69122098,40.69122098,40.70496023,40.63626398,40.60878547999999,40.60878547999999,40.58130698,40.71869948,40.74617798,40.77365648,40.62252472999999,40.63626398,40.63626398,40.62252472999999,40.74617798,40.67748173,40.66374248,40.67748173,40.77365648,40.75991723,40.63626398,40.52634998,40.66374248,40.58130698,40.59504623,40.88357048,40.78739573,40.56756773,40.60878547999999,40.78739573,40.58130698,40.63626398,40.65000323],[-73.81176120000001,-73.79336569,-73.88534324,-73.77497018000001,-73.77497018000001,-73.84855222,-73.73817916,-73.75657467000001,-74.05090283,-74.03250731999999,-73.95892528,-73.9957163,-73.95892528,-73.97732078999999,-74.21646242,-74.17967139999999,-74.21646242,-74.16127589,-73.94052977,-73.94052977,-73.84855222,-73.83015671,-73.92213426000001,-73.94052977,-73.9957163,-74.08769384999999,-74.19806690999999,-73.75657467000001,-73.90373875,-73.95892528,-73.9957163,-73.9957163,-74.19806690999999,-73.92213426000001,-73.84855222,-74.05090283,-73.77497018000001,-73.75657467000001,-73.83015671,-73.84855222,-74.03250731999999,-74.03250731999999,-73.92213426000001,-74.08769384999999,-73.83015671,-73.84855222,-73.88534324,-73.92213426000001,-73.83015671,-73.90373875,-73.73817916,-73.88534324,-73.95892528,-73.95892528,-74.03250731999999,-74.03250731999999,-73.9957163,-74.08769384999999,-73.83015671,-73.77497018000001,-73.92213426000001,-74.10608936,-74.12448487,-73.83015671,-73.79336569,-74.17967139999999,-74.16127589,-74.25325343999999,-73.90373875,-73.95892528,-73.88534324,-73.86694773000001,-73.79336569,-73.79336569,-73.73817916,-73.84855222,-74.01411181,-73.9957163,-73.84855222,-73.88534324,-73.90373875,-73.94052977,-73.88534324,-73.86694773000001,-73.92213426000001,-73.75657467000001,-73.77497018000001,-73.83015671,-73.75657467000001,-73.97732078999999,-73.97732078999999,-73.95892528,-73.94052977,-74.19806690999999,-74.25325343999999,-74.19806690999999,-74.16127589,-73.84855222,-73.92213426000001,-73.9957163,-74.08769384999999,-74.14288037999999,-73.71978365000001,-74.12448487,-74.10608936,-73.75657467000001,-73.73817916,-74.12448487,-73.71978365000001,-73.84855222,-73.90373875,-73.94052977,-73.84855222,-73.92213426000001,-74.06929834,-74.19806690999999,-73.84855222,-73.81176120000001,-74.17967139999999,-73.90373875,-73.88534324,-73.94052977,-73.75657467000001,-73.88534324,-73.95892528,-73.9957163,-73.79336569,-73.86694773000001,-73.90373875,-74.01411181,-74.17967139999999,-73.86694773000001,-74.03250731999999,-74.17967139999999,-74.14288037999999,-73.75657467000001,-73.97732078999999,-73.90373875,-73.9957163,-74.10608936,-73.79336569,-73.86694773000001,-73.81176120000001,-73.86694773000001,-73.73817916,-73.75657467000001,-73.81176120000001,-73.88534324,-73.88534324,-73.97732078999999,-74.03250731999999,-74.01411181,-74.01411181,-73.95892528,-73.94052977,-74.17967139999999,-74.16127589,-74.10608936,-73.83015671,-73.84855222,-73.84855222,-74.01411181,-73.9957163,-74.01411181,-74.12448487,-74.23485792999999,-73.81176120000001,-73.88534324,-73.86694773000001,-73.94052977,-73.95892528,-74.08769384999999,-74.23485792999999,-73.81176120000001,-73.92213426000001,-74.16127589,-73.83015671,-73.77497018000001,-74.12448487,-74.23485792999999,-73.79336569,-74.21646242,-73.90373875,-73.90373875,-73.88534324,-73.86694773000001,-73.73817916,-73.81176120000001,-73.95892528,-73.97732078999999,-73.90373875,-73.88534324,-73.94052977,-73.95892528,-74.01411181,-73.9957163,-73.95892528,-74.14288037999999,-73.83015671,-73.88534324,-73.92213426000001,-73.90373875,-73.88534324,-73.75657467000001,-73.79336569,-73.77497018000001,-73.81176120000001,-73.92213426000001,-73.94052977,-74.19806690999999,-73.86694773000001,-73.84855222,-73.92213426000001,-73.94052977,-74.21646242,-73.79336569,-73.92213426000001,-73.90373875,-73.73817916,-73.94052977,-74.06929834,-73.81176120000001,-74.01411181,-73.84855222,-74.19806690999999,-73.92213426000001,-74.01411181,-73.92213426000001,-73.79336569,-73.77497018000001,-73.79336569,-73.90373875,-73.94052977,-74.14288037999999,-74.23485792999999,-74.17967139999999,-73.77497018000001,-73.81176120000001,-73.88534324,-73.83015671,-73.90373875,-73.94052977,-73.77497018000001,-73.9957163,-73.9957163,-74.10608936,-73.92213426000001,-73.90373875,-73.92213426000001,-73.84855222,-73.77497018000001,-74.05090283,-73.88534324,-74.14288037999999,-74.12448487,-73.94052977,-73.95892528,-73.86694773000001,-73.83015671,-73.84855222,-73.81176120000001,-73.83015671,-73.86694773000001,-73.77497018000001,-73.73817916,-73.79336569,-73.97732078999999,-73.94052977,-73.9957163,-73.9957163,-73.97732078999999,-73.97732078999999,-74.01411181,-74.01411181,-73.94052977,-74.06929834,-74.14288037999999,-74.10608936,-73.83015671,-73.90373875,-73.86694773000001,-74.01411181,-73.95892528,-74.14288037999999,-73.92213426000001,-73.81176120000001,-73.94052977,-73.97732078999999,-73.75657467000001,-73.88534324,-73.86694773000001,-73.83015671,-73.97732078999999,-73.83015671,-73.73817916,-73.86694773000001,-73.86694773000001,-73.86694773000001,-73.92213426000001,-73.84855222,-73.84855222,-73.86694773000001,-73.79336569,-73.81176120000001,-73.81176120000001,-73.9957163,-73.95892528,-74.01411181,-73.97732078999999,-73.97732078999999,-73.97732078999999,-73.95892528,-74.16127589,-73.81176120000001,-74.01411181,-73.84855222,-73.9957163,-74.17967139999999,-73.84855222,-73.88534324,-74.17967139999999,-73.83015671,-73.95892528,-73.84855222,-73.90373875,-73.81176120000001,-73.92213426000001,-73.86694773000001,-73.92213426000001,-73.90373875,-73.75657467000001,-73.79336569,-73.94052977,-73.95892528,-73.90373875,-73.90373875,-73.92213426000001,-73.97732078999999,-73.97732078999999,-73.97732078999999,-73.97732078999999,-74.16127589,-74.08769384999999,-74.10608936,-74.12448487,-73.83015671,-73.95892528,-73.86694773000001,-74.01411181,-73.94052977,-73.92213426000001,-73.94052977,-74.21646242,-73.90373875,-73.95892528,-74.16127589,-73.86694773000001,-73.83015671,-74.17967139999999,-73.75657467000001,-73.86694773000001,-73.77497018000001,-74.16127589,-74.03250731999999],[40.82861347999999,40.74617797999999,40.74617797999999,40.71869947999999,40.70496022999999,40.73243872999999,40.66374247999999,40.59504622999999,40.63626397999999,40.65000322999999,40.60878547999999,40.77365647999999,40.77365647999999,40.77365647999999,40.52634997999999,40.55382847999999,40.56756772999999,40.55382847999999,40.84235272999999,40.81487422999999,40.89730972999999,40.78739572999999,40.73243872999999,40.69122097999999,40.60878547999999,40.62252472999999,40.55382847999999,40.75991722999999,40.71869947999999,40.67748172999999,40.75991722999999,40.80113497999999,40.56756772999999,40.89730972999999,40.59504622999999,40.65000322999999,40.78739572999999,40.70496022999999,40.89730972999999,40.58130697999999,40.62252472999999,40.69122097999999,40.60878547999999,40.58130697999999,40.62252472999999,40.75991722999999,40.77365647999999,40.75991722999999,40.70496022999999,40.70496022999999,40.67748172999999,40.69122097999999,40.66374247999999,40.58130697999999,40.70496022999999,40.71869947999999,40.71869947999999,40.63626397999999,40.85609197999999,40.74617797999999,40.63626397999999,40.62252472999999,40.65000322999999,40.67748172999999,40.69122097999999,40.62252472999999,40.62252472999999,40.51261072999999,40.91104897999999,40.75991722999999,40.88357047999999,40.58130697999999,40.85609197999999,40.80113497999999,40.70496022999999,40.91104897999999,40.63626397999999,40.62252472999999,40.65000322999999,40.70496022999999,40.85609197999999,40.85609197999999,40.84235272999999,40.84235272999999,40.74617797999999,40.69122097999999,40.69122097999999,40.71869947999999,40.60878547999999,40.69122097999999,40.74617797999999,40.84235272999999,40.82861347999999,40.52634997999999,40.52634997999999,40.54008922999999,40.54008922999999,40.78739572999999,40.66374247999999,40.73243872999999,40.60878547999999,40.56756772999999,40.75991722999999,40.56756772999999,40.60878547999999,40.77365647999999,40.73243872999999,40.62252472999999,40.74617797999999,40.70496022999999,40.59504622999999,40.77365647999999,40.85609197999999,40.80113497999999,40.62252472999999,40.63626397999999,40.74617797999999,40.66374247999999,40.52634997999999,40.82861347999999,40.82861347999999,40.75991722999999,40.71869947999999,40.65000322999999,40.65000322999999,40.78739572999999,40.75991722999999,40.74617797999999,40.66374247999999,40.59504622999999,40.65000322999999,40.81487422999999,40.60878547999999,40.60878547999999,40.62252472999999,40.73243872999999,40.58130697999999,40.74617797999999,40.69122097999999,40.63626397999999,40.59504622999999,40.70496022999999,40.60878547999999,40.86983122999999,40.77365647999999,40.67748172999999,40.69122097999999,40.71869947999999,40.73243872999999,40.70496022999999,40.63626397999999,40.58130697999999,40.73243872999999,40.78739572999999,40.80113497999999,40.63626397999999,40.59504622999999,40.59504622999999,40.81487422999999,40.77365647999999,40.66374247999999,40.67748172999999,40.63626397999999,40.74617797999999,40.60878547999999,40.52634997999999,40.89730972999999,40.86983122999999,40.78739572999999,40.63626397999999,40.82861347999999,40.59504622999999,40.51261072999999,40.77365647999999,40.69122097999999,40.58130697999999,40.59504622999999,40.62252472999999,40.59504622999999,40.55382847999999,40.84235272999999,40.51261072999999,40.88357047999999,40.89730972999999,40.89730972999999,40.88357047999999,40.71869947999999,40.59504622999999,40.70496022999999,40.67748172999999,40.69122097999999,40.66374247999999,40.60878547999999,40.62252472999999,40.71869947999999,40.74617797999999,40.81487422999999,40.55382847999999,40.84235272999999,40.85609197999999,40.86983122999999,40.86983122999999,40.81487422999999,40.78739572999999,40.71869947999999,40.67748172999999,40.67748172999999,40.70496022999999,40.62252472999999,40.60878547999999,40.73243872999999,40.71869947999999,40.67748172999999,40.58130697999999,40.55382847999999,40.60878547999999,40.71869947999999,40.81487422999999,40.74617797999999,40.59504622999999,40.59504622999999,40.73243872999999,40.62252472999999,40.81487422999999,40.59504622999999,40.91104897999999,40.77365647999999,40.82861347999999,40.77365647999999,40.77365647999999,40.73243872999999,40.73243872999999,40.73243872999999,40.63626397999999,40.54008922999999,40.56756772999999,40.75991722999999,40.78739572999999,40.75991722999999,40.73243872999999,40.75991722999999,40.74617797999999,40.60878547999999,40.65000322999999,40.58130697999999,40.56756772999999,40.81487422999999,40.84235272999999,40.65000322999999,40.82861347999999,40.66374247999999,40.62252472999999,40.91104897999999,40.60878547999999,40.58130697999999,40.71869947999999,40.85609197999999,40.91104897999999,40.86983122999999,40.84235272999999,40.75991722999999,40.74617797999999,40.75991722999999,40.73243872999999,40.75991722999999,40.67748172999999,40.71869947999999,40.67748172999999,40.66374247999999,40.67748172999999,40.66374247999999,40.65000322999999,40.70496022999999,40.75991722999999,40.86983122999999,40.60878547999999,40.58130697999999,40.58130697999999,40.82861347999999,40.78739572999999,40.69122097999999,40.66374247999999,40.63626397999999,40.65000322999999,40.84235272999999,40.71869947999999,40.66374247999999,40.63626397999999,40.66374247999999,40.63626397999999,40.59504622999999,40.88357047999999,40.60878547999999,40.69122097999999,40.69122097999999,40.71869947999999,40.77365647999999,40.66374247999999,40.78739572999999,40.88357047999999,40.86983122999999,40.82861347999999,40.78739572999999,40.80113497999999,40.74617797999999,40.70496022999999,40.73243872999999,40.65000322999999,40.62252472999999,40.80113497999999,40.81487422999999,40.80113497999999,40.56756772999999,40.70496022999999,40.60878547999999,40.80113497999999,40.59504622999999,40.59504622999999,40.67748172999999,40.67748172999999,40.54008922999999,40.77365647999999,40.74617797999999,40.69122097999999,40.63626397999999,40.81487422999999,40.88357047999999,40.85609197999999,40.85609197999999,40.77365647999999,40.74617797999999,40.70496022999999,40.70496022999999,40.71869947999999,40.65000322999999,40.62252472999999,40.62252472999999,40.59504622999999,40.73243872999999,40.75991722999999,40.78739572999999,40.63626397999999,40.65000322999999,40.65000322999999,40.63626397999999,40.75991722999999,40.69122097999999,40.67748172999999,40.69122097999999,40.78739572999999,40.77365647999999,40.65000322999999,40.54008922999999,40.67748172999999,40.59504622999999,40.60878547999999,40.89730972999999,40.80113497999999,40.58130697999999,40.62252472999999,40.80113497999999,40.59504622999999,40.65000322999999,40.66374247999999],[-73.79336569000002,-73.77497018000001,-73.86694773000001,-73.75657467000002,-73.75657467000002,-73.83015671000001,-73.71978365000001,-73.73817916000002,-74.03250732000001,-74.01411181,-73.94052977000001,-73.97732079000001,-73.94052977000001,-73.95892528,-74.19806691000001,-74.16127589,-74.19806691000001,-74.14288038000001,-73.92213426000001,-73.92213426000001,-73.83015671000001,-73.81176120000001,-73.90373875000002,-73.92213426000001,-73.97732079000001,-74.06929834,-74.1796714,-73.73817916000002,-73.88534324000001,-73.94052977000001,-73.97732079000001,-73.97732079000001,-74.1796714,-73.90373875000002,-73.83015671000001,-74.03250732000001,-73.75657467000002,-73.73817916000002,-73.81176120000001,-73.83015671000001,-74.01411181,-74.01411181,-73.90373875000002,-74.06929834,-73.81176120000001,-73.83015671000001,-73.86694773000001,-73.90373875000002,-73.81176120000001,-73.88534324000001,-73.71978365000001,-73.86694773000001,-73.94052977000001,-73.94052977000001,-74.01411181,-74.01411181,-73.97732079000001,-74.06929834,-73.81176120000001,-73.75657467000002,-73.90373875000002,-74.08769385000001,-74.10608936000001,-73.81176120000001,-73.77497018000001,-74.16127589,-74.14288038000001,-74.23485793,-73.88534324000001,-73.94052977000001,-73.86694773000001,-73.84855222000002,-73.77497018000001,-73.77497018000001,-73.71978365000001,-73.83015671000001,-73.99571630000001,-73.97732079000001,-73.83015671000001,-73.86694773000001,-73.88534324000001,-73.92213426000001,-73.86694773000001,-73.84855222000002,-73.90373875000002,-73.73817916000002,-73.75657467000002,-73.81176120000001,-73.73817916000002,-73.95892528,-73.95892528,-73.94052977000001,-73.92213426000001,-74.1796714,-74.23485793,-74.1796714,-74.14288038000001,-73.83015671000001,-73.90373875000002,-73.97732079000001,-74.06929834,-74.12448487,-73.70138814000002,-74.10608936000001,-74.08769385000001,-73.73817916000002,-73.71978365000001,-74.10608936000001,-73.70138814000002,-73.83015671000001,-73.88534324000001,-73.92213426000001,-73.83015671000001,-73.90373875000002,-74.05090283000001,-74.1796714,-73.83015671000001,-73.79336569000002,-74.16127589,-73.88534324000001,-73.86694773000001,-73.92213426000001,-73.73817916000002,-73.86694773000001,-73.94052977000001,-73.97732079000001,-73.77497018000001,-73.84855222000002,-73.88534324000001,-73.99571630000001,-74.16127589,-73.84855222000002,-74.01411181,-74.16127589,-74.12448487,-73.73817916000002,-73.95892528,-73.88534324000001,-73.97732079000001,-74.08769385000001,-73.77497018000001,-73.84855222000002,-73.79336569000002,-73.84855222000002,-73.71978365000001,-73.73817916000002,-73.79336569000002,-73.86694773000001,-73.86694773000001,-73.95892528,-74.01411181,-73.99571630000001,-73.99571630000001,-73.94052977000001,-73.92213426000001,-74.16127589,-74.14288038000001,-74.08769385000001,-73.81176120000001,-73.83015671000001,-73.83015671000001,-73.99571630000001,-73.97732079000001,-73.99571630000001,-74.10608936000001,-74.21646242,-73.79336569000002,-73.86694773000001,-73.84855222000002,-73.92213426000001,-73.94052977000001,-74.06929834,-74.21646242,-73.79336569000002,-73.90373875000002,-74.14288038000001,-73.81176120000001,-73.75657467000002,-74.10608936000001,-74.21646242,-73.77497018000001,-74.19806691000001,-73.88534324000001,-73.88534324000001,-73.86694773000001,-73.84855222000002,-73.71978365000001,-73.79336569000002,-73.94052977000001,-73.95892528,-73.88534324000001,-73.86694773000001,-73.92213426000001,-73.94052977000001,-73.99571630000001,-73.97732079000001,-73.94052977000001,-74.12448487,-73.81176120000001,-73.86694773000001,-73.90373875000002,-73.88534324000001,-73.86694773000001,-73.73817916000002,-73.77497018000001,-73.75657467000002,-73.79336569000002,-73.90373875000002,-73.92213426000001,-74.1796714,-73.84855222000002,-73.83015671000001,-73.90373875000002,-73.92213426000001,-74.19806691000001,-73.77497018000001,-73.90373875000002,-73.88534324000001,-73.71978365000001,-73.92213426000001,-74.05090283000001,-73.79336569000002,-73.99571630000001,-73.83015671000001,-74.1796714,-73.90373875000002,-73.99571630000001,-73.90373875000002,-73.77497018000001,-73.75657467000002,-73.77497018000001,-73.88534324000001,-73.92213426000001,-74.12448487,-74.21646242,-74.16127589,-73.75657467000002,-73.79336569000002,-73.86694773000001,-73.81176120000001,-73.88534324000001,-73.92213426000001,-73.75657467000002,-73.97732079000001,-73.97732079000001,-74.08769385000001,-73.90373875000002,-73.88534324000001,-73.90373875000002,-73.83015671000001,-73.75657467000002,-74.03250732000001,-73.86694773000001,-74.12448487,-74.10608936000001,-73.92213426000001,-73.94052977000001,-73.84855222000002,-73.81176120000001,-73.83015671000001,-73.79336569000002,-73.81176120000001,-73.84855222000002,-73.75657467000002,-73.71978365000001,-73.77497018000001,-73.95892528,-73.92213426000001,-73.97732079000001,-73.97732079000001,-73.95892528,-73.95892528,-73.99571630000001,-73.99571630000001,-73.92213426000001,-74.05090283000001,-74.12448487,-74.08769385000001,-73.81176120000001,-73.88534324000001,-73.84855222000002,-73.99571630000001,-73.94052977000001,-74.12448487,-73.90373875000002,-73.79336569000002,-73.92213426000001,-73.95892528,-73.73817916000002,-73.86694773000001,-73.84855222000002,-73.81176120000001,-73.95892528,-73.81176120000001,-73.71978365000001,-73.84855222000002,-73.84855222000002,-73.84855222000002,-73.90373875000002,-73.83015671000001,-73.83015671000001,-73.84855222000002,-73.77497018000001,-73.79336569000002,-73.79336569000002,-73.97732079000001,-73.94052977000001,-73.99571630000001,-73.95892528,-73.95892528,-73.95892528,-73.94052977000001,-74.14288038000001,-73.79336569000002,-73.99571630000001,-73.83015671000001,-73.97732079000001,-74.16127589,-73.83015671000001,-73.86694773000001,-74.16127589,-73.81176120000001,-73.94052977000001,-73.83015671000001,-73.88534324000001,-73.79336569000002,-73.90373875000002,-73.84855222000002,-73.90373875000002,-73.88534324000001,-73.73817916000002,-73.77497018000001,-73.92213426000001,-73.94052977000001,-73.88534324000001,-73.88534324000001,-73.90373875000002,-73.95892528,-73.95892528,-73.95892528,-73.95892528,-74.14288038000001,-74.06929834,-74.08769385000001,-74.10608936000001,-73.81176120000001,-73.94052977000001,-73.84855222000002,-73.99571630000001,-73.92213426000001,-73.90373875000002,-73.92213426000001,-74.19806691000001,-73.88534324000001,-73.94052977000001,-74.14288038000001,-73.84855222000002,-73.81176120000001,-74.16127589,-73.73817916000002,-73.84855222000002,-73.75657467000002,-74.14288038000001,-74.01411181],null,null,{"interactive":true,"className":"","stroke":false,"color":"#03F","weight":5,"opacity":0.5,"fill":true,"fillColor":"forestgreen","fillOpacity":[0.04892966360856269,0.8593272171253823,0.1834862385321101,0.1498470948012232,0.1039755351681957,0.1406727828746177,0.08256880733944955,0.01529051987767584,0.3944954128440367,0.1834862385321101,0.1926605504587156,0.4464831804281346,0.09785932721712538,0.4678899082568808,0.253822629969419,0.3730886850152905,0.1529051987767584,0.2996941896024465,0.327217125382263,0.2629969418960245,0.1559633027522936,0.2048929663608563,0.1192660550458716,0.2996941896024465,0.1590214067278287,0.1712538226299694,0.4862385321100918,0.1743119266055046,0.1896024464831804,0.1529051987767584,0.03058103975535168,0.1651376146788991,0.1651376146788991,0.1131498470948012,0.03363914373088685,0.04281345565749235,0.0581039755351682,0.07033639143730887,0.07033639143730887,0.03058103975535168,0.09480122324159021,0.03975535168195719,0.01529051987767584,0.006116207951070336,0.01834862385321101,0.2048929663608563,0.3211009174311927,0.2048929663608563,0.1987767584097859,0.2844036697247707,0.07339449541284404,0.1957186544342508,0.382262996941896,0.09785932721712538,0.01834862385321101,0.4067278287461774,0.5290519877675841,0.1070336391437309,0.02752293577981652,0.1773700305810398,0.2691131498470948,0.2293577981651376,0.1681957186544343,0.1804281345565749,0.09785932721712538,0.05504587155963303,0.09480122324159021,0.1681957186544343,0.07951070336391437,0.1284403669724771,0.1100917431192661,0.03669724770642202,0.03363914373088685,0.003058103975535168,0.03669724770642202,0.02752293577981652,0.07951070336391437,0.01529051987767584,0.003058103975535168,0.003058103975535168,0.2599388379204893,0.1620795107033639,0.1437308868501529,0.2935779816513762,0.1681957186544343,0.3149847094801223,0.2324159021406728,0.2966360856269113,0.1651376146788991,0.5718654434250765,0.2935779816513762,0.4801223241590214,0.400611620795107,0.04587155963302753,0.2324159021406728,0.2018348623853211,0.1467889908256881,0.2201834862385321,0.2018348623853211,0.345565749235474,0.3149847094801223,0.2599388379204893,0.06422018348623854,0.1712538226299694,0.2507645259938838,0.1009174311926606,0.1559633027522936,0.1039755351681957,0.1253822629969419,0.1039755351681957,0.03363914373088685,0.1651376146788991,0.0764525993883792,0.03058103975535168,0.01834862385321101,0.003058103975535168,0.01529051987767584,0.003058103975535168,0.003058103975535168,0.3241590214067278,0.3058103975535168,0.2171253822629969,0.1406727828746177,1,0.1376146788990826,0.5657492354740061,0.1865443425076453,0.4342507645259939,0.3058103975535168,0.07033639143730887,0.03363914373088685,0.05504587155963303,0.03363914373088685,0.05504587155963303,0.1559633027522936,0.1773700305810398,0.1009174311926606,0.1100917431192661,0.07951070336391437,0.1100917431192661,0.0764525993883792,0.01223241590214067,0.003058103975535168,0.1498470948012232,0.1284403669724771,0.3792048929663608,0.2629969418960245,0.1498470948012232,0.5902140672782875,0.2415902140672783,0.1345565749235474,0.06422018348623854,0.4097859327217125,0.327217125382263,0.1100917431192661,0.1100917431192661,0.1865443425076453,0.2966360856269113,0.04892966360856269,0.1712538226299694,0.0856269113149847,0.1406727828746177,0.0581039755351682,0.1651376146788991,0.09480122324159021,0.04892966360856269,0.006116207951070336,0.1834862385321101,0.07033639143730887,0.1865443425076453,0.1743119266055046,0.08868501529051988,0.04281345565749235,0.0764525993883792,0.1192660550458716,0.0581039755351682,0.1100917431192661,0.009174311926605505,0.003058103975535168,0.01834862385321101,0.009174311926605505,0.04587155963302753,0.7920489296636085,0.06727828746177369,0.09785932721712538,0.2629969418960245,0.07951070336391437,0.3333333333333333,0.2844036697247707,0.3547400611620795,0.1559633027522936,0.3425076452599388,0.4648318042813456,0.1896024464831804,0.5137614678899083,0.3761467889908257,0.2415902140672783,0.08868501529051988,0.1162079510703364,0.1437308868501529,0.1651376146788991,0.1284403669724771,0.1009174311926606,0.09785932721712538,0.0581039755351682,0.3547400611620795,0.2415902140672783,0.08868501529051988,0.3425076452599388,0.02140672782874618,0.1681957186544343,0.1529051987767584,0.4281345565749236,0.06422018348623854,0.345565749235474,0.006116207951070336,0.06422018348623854,0.08868501529051988,0.1192660550458716,0.02446483180428135,0.003058103975535168,0.1406727828746177,0.09174311926605505,0.04892966360856269,0.02446483180428135,0.02446483180428135,0.006116207951070336,0.2813455657492355,0.5932721712538226,0.1620795107033639,0.4495412844036697,0.2110091743119266,0.01834862385321101,0.3241590214067278,0.2171253822629969,0.1345565749235474,0.1345565749235474,0.2629969418960245,0.1284403669724771,0.2599388379204893,0.1743119266055046,0.08256880733944955,0.1773700305810398,0.1314984709480122,0.1529051987767584,0.05198776758409786,0.1406727828746177,0.1223241590214067,0.1162079510703364,0.06422018348623854,0.1009174311926606,0.1223241590214067,0.02752293577981652,0.08256880733944955,0.2079510703363914,0.02752293577981652,0.006116207951070336,0.06116207951070336,0.02752293577981652,0.1590214067278287,0.1100917431192661,0.3180428134556575,0.1712538226299694,0.2018348623853211,0.345565749235474,0.4159021406727829,0.1987767584097859,0.3119266055045872,0.2232415902140673,0.1834862385321101,0.4892966360856269,0.3608562691131498,0.08256880733944955,0.05198776758409786,0.1467889908256881,0.08256880733944955,0.2232415902140673,0.1437308868501529,0.1896024464831804,0.09785932721712538,0.3241590214067278,0.09174311926605505,0.03975535168195719,0.09480122324159021,0.1559633027522936,0.1070336391437309,0.1498470948012232,0.1314984709480122,0.04587155963302753,0.0764525993883792,0.003058103975535168,0.08868501529051988,0.0764525993883792,0.01529051987767584,0.0581039755351682,0.03363914373088685,0.009174311926605505,0.03669724770642202,0.01223241590214067,0.2110091743119266,0.2752293577981652,0.3058103975535168,0.04281345565749235,0.3608562691131498,0.2079510703363914,0.3516819571865443,0.2140672782874618,0.0856269113149847,0.08868501529051988,0.9021406727828746,0.8899082568807339,0.6391437308868502,0.09480122324159021,0.3302752293577982,0.1192660550458716,0.08256880733944955,0.0856269113149847,0.09174311926605505,0.1162079510703364,0.1651376146788991,0.2018348623853211,0.2079510703363914,0.1009174311926606,0.07033639143730887,0.1009174311926606,0.003058103975535168,0.1162079510703364,0.1712538226299694,0.1590214067278287,0.3914373088685015,0.1559633027522936,0.3058103975535168,0.1681957186544343,0.1773700305810398,0.1467889908256881,0.1253822629969419,0.1498470948012232,0.5137614678899083,0.2568807339449541,0.2293577981651376,0.6666666666666666,0.253822629969419,0.3730886850152905,0.04587155963302753,0.308868501529052,0.3149847094801223,0.2691131498470948,0.4311926605504587,0.1773700305810398,0.03363914373088685,0.1376146788990826,0.3180428134556575,0.1804281345565749,0.2691131498470948,0.2507645259938838,0.01834862385321101,0.1559633027522936,0.0764525993883792,0.07033639143730887,0.03363914373088685,0.03058103975535168,0.003058103975535168,0.01223241590214067,0.003058103975535168],"smoothFactor":1,"noClip":false},["Trees 16","Trees 281","Trees 60","Trees 49","Trees 34","Trees 46","Trees 27","Trees 5","Trees 129","Trees 60","Trees 63","Trees 146","Trees 32","Trees 153","Trees 83","Trees 122","Trees 50","Trees 98","Trees 107","Trees 86","Trees 51","Trees 67","Trees 39","Trees 98","Trees 52","Trees 56","Trees 159","Trees 57","Trees 62","Trees 50","Trees 10","Trees 54","Trees 54","Trees 37","Trees 11","Trees 14","Trees 19","Trees 23","Trees 23","Trees 10","Trees 31","Trees 13","Trees 5","Trees 2","Trees 6","Trees 67","Trees 105","Trees 67","Trees 65","Trees 93","Trees 24","Trees 64","Trees 125","Trees 32","Trees 6","Trees 133","Trees 173","Trees 35","Trees 9","Trees 58","Trees 88","Trees 75","Trees 55","Trees 59","Trees 32","Trees 18","Trees 31","Trees 55","Trees 26","Trees 42","Trees 36","Trees 12","Trees 11","Trees 1","Trees 12","Trees 9","Trees 26","Trees 5","Trees 1","Trees 1","Trees 85","Trees 53","Trees 47","Trees 96","Trees 55","Trees 103","Trees 76","Trees 97","Trees 54","Trees 187","Trees 96","Trees 157","Trees 131","Trees 15","Trees 76","Trees 66","Trees 48","Trees 72","Trees 66","Trees 113","Trees 103","Trees 85","Trees 21","Trees 56","Trees 82","Trees 33","Trees 51","Trees 34","Trees 41","Trees 34","Trees 11","Trees 54","Trees 25","Trees 10","Trees 6","Trees 1","Trees 5","Trees 1","Trees 1","Trees 106","Trees 100","Trees 71","Trees 46","Trees 327","Trees 45","Trees 185","Trees 61","Trees 142","Trees 100","Trees 23","Trees 11","Trees 18","Trees 11","Trees 18","Trees 51","Trees 58","Trees 33","Trees 36","Trees 26","Trees 36","Trees 25","Trees 4","Trees 1","Trees 49","Trees 42","Trees 124","Trees 86","Trees 49","Trees 193","Trees 79","Trees 44","Trees 21","Trees 134","Trees 107","Trees 36","Trees 36","Trees 61","Trees 97","Trees 16","Trees 56","Trees 28","Trees 46","Trees 19","Trees 54","Trees 31","Trees 16","Trees 2","Trees 60","Trees 23","Trees 61","Trees 57","Trees 29","Trees 14","Trees 25","Trees 39","Trees 19","Trees 36","Trees 3","Trees 1","Trees 6","Trees 3","Trees 15","Trees 259","Trees 22","Trees 32","Trees 86","Trees 26","Trees 109","Trees 93","Trees 116","Trees 51","Trees 112","Trees 152","Trees 62","Trees 168","Trees 123","Trees 79","Trees 29","Trees 38","Trees 47","Trees 54","Trees 42","Trees 33","Trees 32","Trees 19","Trees 116","Trees 79","Trees 29","Trees 112","Trees 7","Trees 55","Trees 50","Trees 140","Trees 21","Trees 113","Trees 2","Trees 21","Trees 29","Trees 39","Trees 8","Trees 1","Trees 46","Trees 30","Trees 16","Trees 8","Trees 8","Trees 2","Trees 92","Trees 194","Trees 53","Trees 147","Trees 69","Trees 6","Trees 106","Trees 71","Trees 44","Trees 44","Trees 86","Trees 42","Trees 85","Trees 57","Trees 27","Trees 58","Trees 43","Trees 50","Trees 17","Trees 46","Trees 40","Trees 38","Trees 21","Trees 33","Trees 40","Trees 9","Trees 27","Trees 68","Trees 9","Trees 2","Trees 20","Trees 9","Trees 52","Trees 36","Trees 104","Trees 56","Trees 66","Trees 113","Trees 136","Trees 65","Trees 102","Trees 73","Trees 60","Trees 160","Trees 118","Trees 27","Trees 17","Trees 48","Trees 27","Trees 73","Trees 47","Trees 62","Trees 32","Trees 106","Trees 30","Trees 13","Trees 31","Trees 51","Trees 35","Trees 49","Trees 43","Trees 15","Trees 25","Trees 1","Trees 29","Trees 25","Trees 5","Trees 19","Trees 11","Trees 3","Trees 12","Trees 4","Trees 69","Trees 90","Trees 100","Trees 14","Trees 118","Trees 68","Trees 115","Trees 70","Trees 28","Trees 29","Trees 295","Trees 291","Trees 209","Trees 31","Trees 108","Trees 39","Trees 27","Trees 28","Trees 30","Trees 38","Trees 54","Trees 66","Trees 68","Trees 33","Trees 23","Trees 33","Trees 1","Trees 38","Trees 56","Trees 52","Trees 128","Trees 51","Trees 100","Trees 55","Trees 58","Trees 48","Trees 41","Trees 49","Trees 168","Trees 84","Trees 75","Trees 218","Trees 83","Trees 122","Trees 15","Trees 101","Trees 103","Trees 88","Trees 141","Trees 58","Trees 11","Trees 45","Trees 104","Trees 59","Trees 88","Trees 82","Trees 6","Trees 51","Trees 25","Trees 23","Trees 11","Trees 10","Trees 1","Trees 4","Trees 1"],null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[40.49887148,40.91104897999999],"lng":[-74.25325343999999,-73.70138814000002]}},"evals":[],"jsHooks":[]}</script>

``` r
library(mapview)
```

    ## The legacy packages maptools, rgdal, and rgeos, underpinning the sp package,
    ## which was just loaded, will retire in October 2023.
    ## Please refer to R-spatial evolution reports for details, especially
    ## https://r-spatial.org/r/2023/05/15/evolution4.html.
    ## It may be desirable to make the sf package available;
    ## package maintainers should consider adding sf to Suggests:.
    ## The sp package is now running under evolution status 2
    ##      (status 2 uses the sf package in place of rgdal)

``` r
mapshot(fancy, file = "images/tree_cover.png")
```

    ## PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.

## Resources

- [Databases using R](https://db.rstudio.com/)
- [library(DBI)](https://dbi.r-dbi.org/reference/)
- [library(bigrquery)](https://bigrquery.r-dbi.org/)
- [library(dbplyr)](https://dbplyr.tidyverse.org/)
- [RStudio Conf 2019, 15 min. Recording](https://rstudio.com/resources/rstudioconf-2019/databases-using-r-the-latest/)
