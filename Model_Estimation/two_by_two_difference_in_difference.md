---
title: 2x2 Difference in Difference 
parent: Category
has_children: false
nav_order: 1
mathjax: true ## Switch to false if this page has no equations or other math rendering.
---


## INTRODUCTION

Causal inference with cross-sectional data is fundamentally tricky.

1. People, firms, etc. are different from one another in lots of ways.
2. Can only get a clean comparison when you have a (quasi-)experimental setup, such as an experiment or an regression discontinuity.

Difference-in-difference makes use of a treatment that was applied to one group at a given time but not another group. It compares how each of those groups changed over time (comparing them to themselves to eliminate between-group differences) and then compares the treatment group difference to the control group difference (both of which contain the same time gaps, eliminating differences over time).

## KEEP IN MIND

- For Difference-in-differences to work, *parallel trends* must hold. That is, *nothing else should be changing the gap between treated and control states at the same time as the treatment*. While it is not a formal test of parallel trends, researchers often look at whether the gap between treated and control states is constant in pre-treatment years.
- Suppose in $t = 0$ ("Pre-period"),  and $t = 1$ ("Post-period"). We want to estimate $\tau = Post - Pre$, or $Y(post)-Y(pre)= Y(t=1)-Y(t=0)=\tau$.

## ALSO CONSIDER 

- This page discusses "2x2" difference-in-difference design, meaning there are two groups, and treatment occurs at a single point in time. Many difference-in-difference applications instead use many groups, and treatments that are implemented at different times (a "rollout" design). Traditionally these models have been estimated using fixed effects for group and time period, i.e. ["two-way" fixed effects](https://lost-stats.github.io/Model_Estimation/fixed_effects_in_linear_regression.html). However, this approach with difference-in-difference can heavily bias results if treatment effects differ across groups, and alternate estimators are preferred. See [Goodman-Bacon 2018](https://www.nber.org/papers/w25018) and [Callaway and Sant'anna 2019](https://papers.ssrn.com/sol3/Papers.cfm?abstract_id=3148250).


# IMPLEMENTSTIONS

## R

In this case, we need to discover whether legalized marijuana could change the murder rate. Some states legalized marijuana in 2014. So we measure the how the murder rate changes from before 2014 to after between legalized states and states without legalization. 

Step 1:
* First of all, we need to load Data and Package, we call this data set "DiD".
```{r}
library(readr)
library(tidyverse)
library(broom)
library(here)
DiD <- read_csv(here("Data/DiD_crime.csv"))
```

Step 2:

Secondly, we create the indicator variable called `after` to indicate whether it is after the year of 2014 (1), or between 2010-2013 (0). The variable `treat` indicates that the state legalizes marijuana in 2014. Notice that `treat = 1` in these states even before 2014.

If the year is after 2014 **and** the state decided to legalize marijuana, the indicator variable "treatafter" is "1" .

```{r}
DiD <- mutate(DiD, after = year >= 2014,
                   treat = year>=2010 & year<=2013) %>%
       mutate(treatafter = after*treat)
```

Step 3:

Then we need to plot the graph to visualize the impact of legalize marijuana on murder rate by using `ggplot`.

```{r}
mt <-ggplot(DiD,aes(x=year, y=murder, color = treat)) +
        geom_point(size=3)+geom_line() + 
        geom_vline(xintercept=2014,lty=4) + 
        labs(title="Murder and Time", x="Year", y="Murder Rate")
mt
```
![Diff-in-Diff](https://github.com/zuzhangjin/lost-stats.github.io/blob/source/Model_Estimation/Images/dif%20in%20dif.jpg)

It looks like, before the legalization occurred, murder rates in treated and untreated states were very similar, lending plausibility to the parallel trends assumption.

Step 4:

We need to measure the impact of impact of legalize marijuana. If we include `treat`, `after`, and `treatafter` in a regression, the coefficient on `treatafter` can be interpreted as "how much bigger was the before-after difference for the treated group?" which is the DiD estimate. 

```{r}
reg<-lm(murder ~ treat+treatafter+after, data = DiD)
summary(reg)
```

After legalization, the murder rate dropped by 0.3% more in treated than untreated states, suggesting that legalization reduced the murder rate.
