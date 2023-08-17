---
title: "2023 August 15 - Spam Email"
date: 2023-08-15
output: html_document
---



------------------------------------------------------------------------

The goal of this work is to apply my understanding of data towards real world projects using R and the tidyverse. The data used here is from the TidyTuesday project.

# Check Out This Week's Data


```r
# load R packages for analysis
library(tidyverse)
library(tidytuesdayR)
```


```r
# Download the weekly data and make available in the tt object.
# Using the last_tuesday function gives us the latest TidyTuesday data from today's date

tues <- last_tuesday("2023-08-15")

tt <- tt_load(tues)
```

```
## 
## 	Downloading file 1 of 1: `spam.csv`
```


```r
# Check out the available data
tt
```

The data this week comes from Vincent Arel-Bundock's Rdatasets package(<https://vincentarelbundock.github.io/Rdatasets/index.html>).

> Rdatasets is a collection of 2246 datasets which were originally distributed alongside the statistical software environment R and some of its add-on packages. The goal is to make these data more broadly accessible for teaching and statistical software development.

We're working with the [spam email](https://vincentarelbundock.github.io/Rdatasets/doc/DAAG/spam7.html) dataset. This is a subset of the [spam e-mail database](https://search.r-project.org/CRAN/refmans/kernlab/html/spam.html).

This is a dataset collected at Hewlett-Packard Labs by Mark Hopkins, Erik Reeber, George Forman, and Jaap Suermondt and shared with the [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/94/spambase). The dataset classifies 4601 e-mails as spam or non-spam, with additional variables indicating the frequency of certain words and characters in the e-mail.

## The following are the included variables:

| **Variable** | **Class** | **Description**                                                          |
|--------------------|------------------|----------------------------------|
| crl.tot      | double    | Total length of uninterrupted sequences of capitals                      |
| dollar       | double    | Occurrences of the dollar sign, as percent of total number of characters |
| bang         | double    | Occurrences of \'!\', as percent of total number of characters           |
| money        | double    | Occurrences of \'money\', as percent of total number of characters       |
| n000         | double    | Occurrences of the string \'000\', as percent of total number of words   |
| make         | double    | Occurrences of \'make\', as a percent of total number of words           |
| yesno        | character | Outcome variable, a factor with levels 'n' not spam, 'y' spam            |

# Initial Thoughts and Questions

Based on the context of this data set and variables provided, we understand that it was probably used to teach classification algorithms to recognize spam email:

-   Do the oldest individuals on the planet were living in the same general location or country?

-   Who tends to live longer? Men or Women?

From what I've previously heard regarding age statistics, Japan tends to have the longest life expectancy and women tend to live longer than men, so let's explore if that's still the case.

# Data Wrangling and Exploratory Data Analysis

## Extract the data

Luckily, the spam data set only contains one data frame, so we don't need to create some table joins. We can just extract the data frame to explore the variables.


```r
# Extract data from tt 
spam <- tt$spam

spam %>% glimpse()
```

```
## Rows: 4,601
## Columns: 7
## $ crl.tot <dbl> 278, 1028, 2259, 191, 191, 54, 112, 49, 1257, 749, 21, 184, 26…
## $ dollar  <dbl> 0.000, 0.180, 0.184, 0.000, 0.000, 0.000, 0.054, 0.000, 0.203,…
## $ bang    <dbl> 0.778, 0.372, 0.276, 0.137, 0.135, 0.000, 0.164, 0.000, 0.181,…
## $ money   <dbl> 0.00, 0.43, 0.06, 0.00, 0.00, 0.00, 0.00, 0.00, 0.15, 0.00, 0.…
## $ n000    <dbl> 0.00, 0.43, 1.16, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.19, 0.…
## $ make    <dbl> 0.00, 0.21, 0.06, 0.00, 0.00, 0.00, 0.00, 0.00, 0.15, 0.06, 0.…
## $ yesno   <chr> "y", "y", "y", "y", "y", "y", "y", "y", "y", "y", "y", "y", "y…
```

### How many spam emails do we have vs not spam?

It looks like we have a 40/60 split of spam and not spam emails in this data, as seen below.


```r
# Group data by gender and count via summarize
spam %>%
  group_by(yesno) %>%
  summarise(n())
```

```
## # A tibble: 2 × 2
##   yesno `n()`
##   <chr> <int>
## 1 n      2788
## 2 y      1813
```

### Let's try to gather the variables


```r
spam2 <- spam %>%
  gather(key = "variable", value = "value", -yesno)
spam2
```

```
## # A tibble: 27,606 × 3
##    yesno variable value
##    <chr> <chr>    <dbl>
##  1 y     crl.tot    278
##  2 y     crl.tot   1028
##  3 y     crl.tot   2259
##  4 y     crl.tot    191
##  5 y     crl.tot    191
##  6 y     crl.tot     54
##  7 y     crl.tot    112
##  8 y     crl.tot     49
##  9 y     crl.tot   1257
## 10 y     crl.tot    749
## # ℹ 27,596 more rows
```


```r
ggplot(spam2, 
       aes(x = yesno, y = value, fill = yesno)) + 
  geom_boxplot(notch = TRUE) + 
  scale_y_log10() + 
  facet_wrap(vars(variable), scales = "free") + 
  coord_flip() + 
  labs(title = "Email Words and Spam", y = "Percentages", x = "Classification") + 
  theme(legend.position = "none")
```

```
## Warning: Transformation introduced infinite values in continuous y-axis
```

```
## Warning: Removed 16880 rows containing non-finite values (`stat_boxplot()`).
```

<img src="/post/2023-08-15-TidyTuesday/2023 AUG 15 - tidy_tuesday_files/figure-html/Plot Data-1.png" width="672" />

### Let's replace "y" and "n" with 1 and 0 to see if we can set up a model using Log Regression

Let's use `str_replace` to convert the "y" and "n" text to "0" and "1". Since the converted `yesno` variable is still a string, let's use `as.numeric` to convert the column to a numeric.


```r
spam$yesno <- str_replace(string = spam$yesno, pattern = "n", replacement = "0")
spam$yesno <- str_replace(string = spam$yesno, pattern = "y", replacement = "1")
spam$yesno <- spam$yesno %>% as.numeric()
```

Let's keep `crl.tot` separate from the other variables since the values don't appear to be recorded as percentages.


```r
spam3 <- spam %>%
  gather(key = "variable", value = "value", -yesno, -crl.tot)
spam3
```

```
## # A tibble: 23,005 × 4
##    crl.tot yesno variable value
##      <dbl> <dbl> <chr>    <dbl>
##  1     278     1 dollar   0    
##  2    1028     1 dollar   0.18 
##  3    2259     1 dollar   0.184
##  4     191     1 dollar   0    
##  5     191     1 dollar   0    
##  6      54     1 dollar   0    
##  7     112     1 dollar   0.054
##  8      49     1 dollar   0    
##  9    1257     1 dollar   0.203
## 10     749     1 dollar   0.081
## # ℹ 22,995 more rows
```

### Let's try to create a Logistic Regression Model


```r
model1 <- glm(yesno ~ ., data = spam, family = "binomial")
```

```
## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred
```

```r
summary(model1) 
```

```
## 
## Call:
## glm(formula = yesno ~ ., family = "binomial", data = spam)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -8.4904  -0.6153  -0.5816   0.4439   1.9323  
## 
## Coefficients:
##               Estimate Std. Error z value Pr(>|z|)    
## (Intercept) -1.700e+00  5.361e-02 -31.717  < 2e-16 ***
## crl.tot      6.917e-04  9.745e-05   7.098 1.27e-12 ***
## dollar       8.013e+00  6.175e-01  12.976  < 2e-16 ***
## bang         1.572e+00  1.115e-01  14.096  < 2e-16 ***
## money        2.142e+00  2.418e-01   8.859  < 2e-16 ***
## n000         4.149e+00  4.371e-01   9.492  < 2e-16 ***
## make         1.698e-02  1.434e-01   0.118    0.906    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 6170.2  on 4600  degrees of freedom
## Residual deviance: 4058.8  on 4594  degrees of freedom
## AIC: 4072.8
## 
## Number of Fisher Scoring iterations: 16
```

It looks like our model was successful. Let's try to test out plotting the model as well, although a binary outcome probably doesn't create the best visualization.


```r
ggplot(spam3, 
       aes(x = value, y = yesno, color = variable)) + 
  geom_point() + 
  geom_smooth(method = glm, method.args = list(family = "binomial")) + 
  labs(title = "Log Plot", 
       x = "Values", 
       y = "Spam / Not Spam")
```

```
## `geom_smooth()` using formula = 'y ~ x'
```

```
## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred

## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred

## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred

## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred
```

<img src="/post/2023-08-15-TidyTuesday/2023 AUG 15 - tidy_tuesday_files/figure-html/Log Regression Plot-1.png" width="672" />