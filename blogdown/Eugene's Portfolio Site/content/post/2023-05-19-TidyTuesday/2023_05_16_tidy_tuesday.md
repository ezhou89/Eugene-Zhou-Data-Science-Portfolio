---
title: "2023 MAY 19 - Tornadoes in the US"
date: 2023-05-19
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
tt <- tt_load(last_tuesday("2023-05-19"))
```

```
## 
## 	Downloading file 1 of 1: `tornados.csv`
```


```r
# Check out the available data
tt
```

This week's data comes from the [Severe Weather Maps, Graphics, and Data Page](https://www.spc.noaa.gov/wcm/#data) of the NOAA's National Weather Service Storm Prediction Center.

The following are the included variables:

| **Variable** | **Class** | **Description**                                                                                                                                                                      |
|----------|----------|-----------------------------------------------------|
| om           | integer   | Tornado number. Effectively an ID for this tornado in this year.                                                                                                                     |
| yr           | integer   | Year, 1950-2022.                                                                                                                                                                     |
| mo           | integer   | Month, 1-12.                                                                                                                                                                         |
| dy           | integer   | Day of the month, 1-31.                                                                                                                                                              |
| date         | date      | Date.                                                                                                                                                                                |
| time         | time      | Time.                                                                                                                                                                                |
| tz           | character | [Canonical tz database timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).                                                                                      |
| datetime_utc | datetime  | Date and time normalized to UTC.                                                                                                                                                     |
| st           | character | Two-letter postal abbreviation for the state (DC = Washington, DC; PR = Puerto Rico; VI = Virgin Islands).                                                                           |
| stf          | integer   | State FIPS (Federal Information Processing Standards) number.                                                                                                                        |
| mag          | integer   | Magnitude on the F scale (EF beginning in 2007). Some of these values are estimated (see fc).                                                                                        |
| inj          | integer   | Number of injuries. When summing for state totals, use sn == 1 (see below).                                                                                                          |
| fat          | integer   | Number of fatalities. When summing for state totals, use sn == 1 (see below).                                                                                                        |
| loss         | double    | Estimated property loss information in dollars. Prior to 1996, values were grouped into ranges. The reported number for such years is the maximum of its range.                      |
| slat         | double    | Starting latitude in decimal degrees.                                                                                                                                                |
| slon         | double    | Starting longitude in decimal degrees.                                                                                                                                               |
| elat         | double    | Ending latitude in decimal degrees.                                                                                                                                                  |
| elon         | double    | Ending longitude in decimal degrees.                                                                                                                                                 |
| len          | double    | Length in miles.                                                                                                                                                                     |
| wid          | double    | Width in yards.                                                                                                                                                                      |
| ns           | integer   | Number of states affected by this tornado. 1, 2, or 3.                                                                                                                               |
| sn           | integer   | State number for this row. 1 means the row contains the entire track information for this state, 0 means there is at least one more entry for this state for this tornado (om + yr). |
| f1           | integer   | FIPS code for the 1st county.                                                                                                                                                        |
| f2           | integer   | FIPS code for the 2nd county.                                                                                                                                                        |
| f3           | integer   | FIPS code for the 3rd county.                                                                                                                                                        |
| f4           | integer   | FIPS code for the 4th county.                                                                                                                                                        |
| fc           | logical   | Was the mag column estimated?                                                                                                                                                        |

## Initial Thoughts and Questions

Based on the variables provided, there are likely a few interesting relationships to explore with regards to recorded tornadoes, including magnitudes, number of states affected, estimated property loss, and number of injuries and fatalities. Furthermore, because climate change is a pressing concern for many, how has the severity of tornadoes changed over the years?

## Data Wrangling and Exploratory Data Analysis

Let's create a series of smaller data frames to explore whether these variables have some sort of relationship that we can further explore


```r
# Extract tornados.csv from tt 
tornadoes <- tt$tornados
```


```r
# Create a new data frame to explore tornado magnitudes through the years and the monetary damages caused 

df1 <- tornadoes %>%
  select(yr, mag, loss) 
  
df1 %>% skimr::skim()
```


Table: <span id="tab:unnamed-chunk-3"></span>Table 1: Data summary

|                         |           |
|:------------------------|:----------|
|Name                     |Piped data |
|Number of rows           |68693      |
|Number of columns        |3          |
|_______________________  |           |
|Column type frequency:   |           |
|numeric                  |3          |
|________________________ |           |
|Group variables          |None       |


**Variable type: numeric**

|skim_variable | n_missing| complete_rate|       mean|          sd|   p0|   p25|   p50|    p75|       p100|hist  |
|:-------------|---------:|-------------:|----------:|-----------:|----:|-----:|-----:|------:|----------:|:-----|
|yr            |         0|          1.00|    1991.85|       19.57| 1950|  1976|  1995|   2008|       2022|▃▅▆▇▇ |
|mag           |       756|          0.99|       0.78|        0.90|    0|     0|     1|      1|          5|▇▂▁▁▁ |
|loss          |     27170|          0.60| 2020898.22| 30395882.07|   50| 10000| 50000| 500000| 2800100000|▇▁▁▁▁ |

Out of 68,693 rows, there are 27,170 rows where values for loss are missing. Drop these rows from this first data frame.


```r
df1 <- df1 %>% 
  drop_na()

# Let's also clean up the original tornadoes df
tornadoes <- tornadoes %>%
  drop_na()
```

It makes sense that the amount of monetary damage a tornado incurs should increase with increasing magnitude. For this data frame, we can group_by and explore the number of tornadoes that occured at each magnitude, as well as the average damage cost and when a tornado at each magnitude last occurred.


```r
df1 %>%
  group_by(mag) %>%
  summarise(`No. of Events` = n(), 
            `Mean Loss (USD)` = mean(loss), 
            `Last Occured` = last(yr))
```

```
## # A tibble: 6 × 4
##     mag `No. of Events` `Mean Loss (USD)` `Last Occured`
##   <dbl>           <int>             <dbl>          <dbl>
## 1     0           11895            79761.           2022
## 2     1           18363           514475.           2022
## 3     2            8386          1866285.           2022
## 4     3            2290         10210050.           2022
## 5     4             530         43083575.           2022
## 6     5              53        219843019.           2013
```

## Visualize

Let's plot this first data frame by converting the magnitude to a factor, so we can graph the data as a boxplot. Use a log scale for the y-axis and set notch = TRUE to show separation between the IQR of each magnitude.


```r
#
df1 %>%
ggplot(aes(x = as_factor(mag), y = loss, fill = mag)) + 
  geom_boxplot(notch = TRUE) + 
  scale_y_log10() + 
  scale_fill_steps2(low = "red", 
                    mid = "white", 
                    high = "blue", 
                    midpoint = 2) + 
  labs(x = "Magnitude", 
       y = "Property Loss ($, USD)", 
       title = "Property Damage Increases with Tornado Magnitude")
```

```
## Notch went outside hinges
## ℹ Do you want `notch = FALSE`?
## Notch went outside hinges
## ℹ Do you want `notch = FALSE`?
```

<img src="/post/2023-05-19-TidyTuesday/2023_05_16_tidy_tuesday_files/figure-html/Visualize-1.png" width="672" />


```r
# This will save your most recent plot
ggsave(
  filename = "My TidyTuesday Plot.png",
  device = "png")
```

```
## Saving 7 x 5 in image
## Notch went outside hinges
## ℹ Do you want `notch = FALSE`?
## Notch went outside hinges
## ℹ Do you want `notch = FALSE`?
```


```r
df2 <- tornadoes %>%
    select(mag, ns, loss) 

df2 %>%
    ggplot((aes(x = as_factor(ns), y = loss))) + 
    geom_boxplot(notch = TRUE) + 
    scale_y_log10() + 
    labs(title = "Losses Do NOT Noticeably Increase Beyond Impacting 2 States", 
         x = "# of States Affected by Tornado", 
         y = "Property Loss ($, USD)")
```

```
## Notch went outside hinges
## ℹ Do you want `notch = FALSE`?
```

<img src="/post/2023-05-19-TidyTuesday/2023_05_16_tidy_tuesday_files/figure-html/unnamed-chunk-7-1.png" width="672" />

We understand that tornadoes typically occur in the middle of the US, as opposed to coastal states, but which states have had the most recorded?


```r
tornadoes %>%
  group_by(st) %>%
  summarise(mean(mag),
            n()) %>%
  #clean up variable names 
  transmute(`State` = st, 
         `Avg Magnitude` = `mean(mag)`,
            `No. of Tornadoes` = `n()`) %>%
  arrange(desc(`No. of Tornadoes`))
```

```
## # A tibble: 52 × 3
##    State `Avg Magnitude` `No. of Tornadoes`
##    <chr>           <dbl>              <int>
##  1 TX              1.04                4601
##  2 FL              0.654               2585
##  3 OK              1.27                2499
##  4 MS              1.12                2209
##  5 IA              1.14                1898
##  6 LA              1.08                1762
##  7 KS              1.19                1728
##  8 MO              1.12                1711
##  9 AL              1.16                1687
## 10 GA              1.02                1618
## # ℹ 42 more rows
```

It looks like Texas, Florida, and Oklahoma are the three states with the most tornadoes (a little surprising that Florida gets tornadoes in addition to hurricanes). What about the states with the tornadoes of the greatest magnitude?


```r
tornadoes %>%
  group_by(st) %>%
  summarise(mean(mag),
            n()) %>%
  #clean up variable names 
  transmute(`State` = st, 
         `Avg Magnitude` = `mean(mag)`,
            `No. of Tornadoes` = `n()`) %>%
  arrange(-`Avg Magnitude`)
```

```
## # A tibble: 52 × 3
##    State `Avg Magnitude` `No. of Tornadoes`
##    <chr>           <dbl>              <int>
##  1 AR               1.34               1310
##  2 OK               1.27               2499
##  3 SD               1.23                658
##  4 IN               1.22               1189
##  5 IL               1.21               1407
##  6 WI               1.21               1048
##  7 KS               1.19               1728
##  8 NH               1.19                 74
##  9 KY               1.17                904
## 10 AL               1.16               1687
## # ℹ 42 more rows
```

It looks like Arkansas, Oklahoma, and South Dakota have tornadoes of the greatest magnitude, on average.

## Can we model? 

As shown in the data earlier, we understand the strength of a tornado will have an affect on other variables in this data, such as property damage and injuries. From here, we will be following along with the steps used in Julia Silge's screencast, building a predictive model using XGBoost (although the pre-processed data will be slightly different)


```r
# Load tidymodels package
library(tidymodels)
```

```
## ── Attaching packages ────────────────────────────────────── tidymodels 1.1.0 ──
```

```
## ✔ broom        1.0.5     ✔ rsample      1.1.1
## ✔ dials        1.2.0     ✔ tune         1.1.1
## ✔ infer        1.0.4     ✔ workflows    1.1.3
## ✔ modeldata    1.2.0     ✔ workflowsets 1.0.1
## ✔ parsnip      1.1.0     ✔ yardstick    1.2.0
## ✔ recipes      1.0.7
```

```
## ── Conflicts ───────────────────────────────────────── tidymodels_conflicts() ──
## ✖ scales::discard() masks purrr::discard()
## ✖ dplyr::filter()   masks stats::filter()
## ✖ recipes::fixed()  masks stringr::fixed()
## ✖ dplyr::lag()      masks stats::lag()
## ✖ yardstick::spec() masks readr::spec()
## ✖ recipes::step()   masks stats::step()
## • Learn how to get started at https://www.tidymodels.org/start/
```

### **Let\'s set the data budgent by splitting the data for training and testing.**


```r
# Set seed
set.seed(123)

# Stratify data by magnitude to ensure that higher magnitude tornadoes with lower occurence are still in the training and testing data splits

tornado_split <- tornadoes %>%
  initial_split(strata = mag)

# Create a training and testing set of data

tornado_train <- training(tornado_split)
tornado_test <- testing(tornado_split)
```

### **Let\'s create some cross-validation re-samples.**


```r
# Set the seed
set.seed(234)

# Create 10 mini samples for feature engineering and model tuning when selecting optimal model

tornado_folds <- vfold_cv(tornado_train, strata = mag)
tornado_folds
```

```
## #  10-fold cross-validation using stratification 
## # A tibble: 10 × 2
##    splits               id    
##    <list>               <chr> 
##  1 <split [28021/3115]> Fold01
##  2 <split [28021/3115]> Fold02
##  3 <split [28021/3115]> Fold03
##  4 <split [28022/3114]> Fold04
##  5 <split [28023/3113]> Fold05
##  6 <split [28023/3113]> Fold06
##  7 <split [28023/3113]> Fold07
##  8 <split [28023/3113]> Fold08
##  9 <split [28023/3113]> Fold09
## 10 <split [28024/3112]> Fold10
```


```r
# Load embed package for the glm recipe function
library(embed)


# Supervised feature engineering using step_lencode_glm
tornado_recipe <- 
  recipe(mag ~ date + st + inj + len + wid, data = tornado_train) %>%
  # Encode by state and set outcome as magnitude. maps the state to the outcome
  step_lencode_glm(st, outcome = vars(mag)) %>%
  # Use month and year as features due to seasonality of tornadoes
  step_date(date, features = c("month", "year"), keep_original_cols = FALSE) %>%
  # dummy variable for all remaining categorical variables
  step_dummy(all_nominal_predictors())
  
tornado_recipe
```

```
## 
```

```
## ── Recipe ──────────────────────────────────────────────────────────────────────
```

```
## 
```

```
## ── Inputs
```

```
## Number of variables by role
```

```
## outcome:   1
## predictor: 5
```

```
## 
```

```
## ── Operations
```

```
## • Linear embedding for factors via GLM for: st
```

```
## • Date features from: date
```

```
## • Dummy variables from: all_nominal_predictors()
```


```r
# Learn which transformations need to occur from training data 
prep(tornado_recipe) %>% bake(new_data = NULL) %>%
  glimpse()
```

```
## Rows: 31,136
## Columns: 17
## $ st             <dbl> 1.1343931, 1.1304608, 1.2276498, 1.0012422, 1.0273574, …
## $ inj            <dbl> 0, 3, 0, 1, 32, 8, 2, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, …
## $ len            <dbl> 0.1, 2.0, 9.6, 0.1, 7.7, 0.2, 2.0, 18.1, 0.5, 0.1, 0.2,…
## $ wid            <dbl> 10, 37, 50, 10, 100, 10, 33, 27, 27, 10, 10, 100, 37, 1…
## $ mag            <dbl> 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
## $ date_year      <int> 1950, 1950, 1950, 1950, 1950, 1950, 1950, 1950, 1950, 1…
## $ date_month_Feb <dbl> 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Mar <dbl> 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Apr <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0…
## $ date_month_May <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1…
## $ date_month_Jun <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Jul <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Aug <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Sep <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Oct <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Nov <dbl> 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ date_month_Dec <dbl> 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
```

### **Now that we have prepared the data, let\'s build a model.**


```r
# Use xgboost model because we know variables within this large data set are correlated with each other

xgb_spec <- 
  boost_tree(
    trees = tune(), 
    min_n = tune(), 
    mtry = tune(), 
    learn_rate = 0.01
  ) %>%
  set_engine("xgboost") %>%
  set_mode("regression") 

# Create the XGBoost workflow
xgb_wf <- workflow(tornado_recipe, xgb_spec)
```

### **Let\'s tune the model**


```r
# Load the finetune package
library(finetune)
doParallel::registerDoParallel()

# Set seed for model reproducibility
set.seed(345)

# Try all hyperparameter combinations
# Use an ANOVA model to gauge different model hyperparameters
xgb_rs <- tune_race_anova(
  #will use an ANOVA to elminate tested models using cross-validation resamples
  xgb_wf, 
  resamples = tornado_folds, 
  grid = 15, 
  control = control_race(verbose_elim = TRUE)
)
```

```
## i Creating pre-processing data to finalize unknown parameter: mtry
```

```
## ℹ Racing will minimize the rmse metric.
## ℹ Resamples are analyzed in a random order.
## ℹ Fold10: 8 eliminated; 7 candidates remain.
## 
## ℹ Fold07: 2 eliminated; 5 candidates remain.
## 
## ℹ Fold03: 2 eliminated; 3 candidates remain.
## 
## ℹ Fold05: 0 eliminated; 3 candidates remain.
## 
## ℹ Fold09: 0 eliminated; 3 candidates remain.
## 
## ℹ Fold04: 0 eliminated; 3 candidates remain.
## 
## ℹ Fold06: 0 eliminated; 3 candidates remain.
```


```r
# Check out the racing results
xgb_rs
```

```
## # Tuning results
## # 10-fold cross-validation using stratification 
## # A tibble: 10 × 5
##    splits               id     .order .metrics          .notes          
##    <list>               <chr>   <int> <list>            <list>          
##  1 <split [28021/3115]> Fold01      2 <tibble [30 × 7]> <tibble [0 × 3]>
##  2 <split [28021/3115]> Fold02      3 <tibble [30 × 7]> <tibble [0 × 3]>
##  3 <split [28024/3112]> Fold10      1 <tibble [30 × 7]> <tibble [0 × 3]>
##  4 <split [28023/3113]> Fold07      4 <tibble [14 × 7]> <tibble [0 × 3]>
##  5 <split [28021/3115]> Fold03      5 <tibble [10 × 7]> <tibble [0 × 3]>
##  6 <split [28023/3113]> Fold05      6 <tibble [6 × 7]>  <tibble [0 × 3]>
##  7 <split [28023/3113]> Fold09      7 <tibble [6 × 7]>  <tibble [0 × 3]>
##  8 <split [28022/3114]> Fold04      8 <tibble [6 × 7]>  <tibble [0 × 3]>
##  9 <split [28023/3113]> Fold06      9 <tibble [6 × 7]>  <tibble [0 × 3]>
## 10 <split [28023/3113]> Fold08     10 <tibble [6 × 7]>  <tibble [0 × 3]>
```

### **Evaluate the model**


```r
# Check out best resulting model
collect_metrics(xgb_rs)
```

```
## # A tibble: 6 × 9
##    mtry trees min_n .metric .estimator  mean     n std_err .config              
##   <int> <int> <int> <chr>   <chr>      <dbl> <int>   <dbl> <chr>                
## 1     9  1631    27 rmse    standard   0.628    10 0.00283 Preprocessor1_Model09
## 2     9  1631    27 rsq     standard   0.530    10 0.00436 Preprocessor1_Model09
## 3    11  1273     6 rmse    standard   0.629    10 0.00285 Preprocessor1_Model10
## 4    11  1273     6 rsq     standard   0.530    10 0.00432 Preprocessor1_Model10
## 5    14  1877    37 rmse    standard   0.628    10 0.00299 Preprocessor1_Model13
## 6    14  1877    37 rsq     standard   0.530    10 0.00448 Preprocessor1_Model13
```


```r
# Check out tested hyperparameters and elimninated models
plot_race(xgb_rs)
```

<img src="/post/2023-05-19-TidyTuesday/2023_05_16_tidy_tuesday_files/figure-html/unnamed-chunk-19-1.png" width="672" />


```r
# Set and finalize the parameters
tornado_fit <- 
  xgb_wf %>%
  # Hyperparameters no longer tunable 
  finalize_workflow(select_best(xgb_rs, "rmse")) %>%
  # Fits to training data and then once to testing data
  last_fit(tornado_split)

tornado_fit
```

```
## # Resampling results
## # Manual resampling 
## # A tibble: 1 × 6
##   splits                id             .metrics .notes   .predictions .workflow 
##   <list>                <chr>          <list>   <list>   <list>       <list>    
## 1 <split [31136/10381]> train/test sp… <tibble> <tibble> <tibble>     <workflow>
```


```r
# Metrics collected from testing data
# Should indicate no overfitting of model to training data
collect_metrics(tornado_fit)
```

```
## # A tibble: 2 × 4
##   .metric .estimator .estimate .config             
##   <chr>   <chr>          <dbl> <chr>               
## 1 rmse    standard       0.635 Preprocessor1_Model1
## 2 rsq     standard       0.519 Preprocessor1_Model1
```

### **Does our model predict tornado magnitude?**


```r
# 
collect_predictions(tornado_fit) %>%
  ggplot(aes(.pred)) + 
  geom_histogram()
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="/post/2023-05-19-TidyTuesday/2023_05_16_tidy_tuesday_files/figure-html/unnamed-chunk-22-1.png" width="672" />


```r
collect_predictions(tornado_fit) %>%
  mutate(mag = as_factor(mag)) %>%
  ggplot(aes(x = mag, y = .pred, fill = mag)) + 
  geom_boxplot(alpha = 0.6, show.legend = FALSE) + 
  labs(title = "Predicting Tornado Magnitude", 
       x = "Actual Magnitude", 
       y = "Predicted Magnitude")
```

<img src="/post/2023-05-19-TidyTuesday/2023_05_16_tidy_tuesday_files/figure-html/unnamed-chunk-23-1.png" width="672" />

We see from this data that we have **over-predicted** the severity of tornadoes at the lower end of the spectrum. Next, let\'s see which variables were determined to be the most important with our xgboost model.


```r
# Load the vip library
library(vip)
```

```
## 
## Attaching package: 'vip'
```

```
## The following object is masked from 'package:utils':
## 
##     vi
```


```r
extract_workflow(tornado_fit) %>%
  extract_fit_parsnip() %>%
  vip(num_features = 10)
```

<img src="/post/2023-05-19-TidyTuesday/2023_05_16_tidy_tuesday_files/figure-html/unnamed-chunk-25-1.png" width="672" />

We see that injury, length, year, width, and state were most predictive for magnitude. This should make sense as the variables were already perceived to be correlated, although we may want to separate out variables like injury in the future, as injuries would be measured after a tornado and not before.

### **Deploy the model**


```r
library(vetiver)
```

```
## 
## Attaching package: 'vetiver'
```

```
## The following object is masked from 'package:tune':
## 
##     load_pkgs
```


```r
v <- extract_workflow(tornado_fit) %>%
  vetiver_model("tornado-xgb")

v
```

```
## 
## ── tornado-xgb ─ <bundled_workflow> model for deployment 
## A xgboost regression modeling workflow using 5 features
```

## **Future Work**

1.  Further explore the price outliers where only 1 state was affected by the tornado.

2.  Map visualization of tornado damage
