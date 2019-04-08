---
title: "Power and Sample Size in R"
author: "your name here"
date: "4/9/2019"
output: 
  html_document:
    keep_md: yes
---

This workshop will demonstrate how to estimate power and sample size for a variety of statistical tests. We will also learn about the relationships between power, sample size, and effect size.

Structure and content outline are based on a power and sample size workshop by Clay Ford (UVA libraries).

### Set-up

First step is to install the pwr package


```r
#install.packages("pwr")
library(pwr)
```

## pwr.p.test

One-sample test for proportions  (ES=h) 

This type of analysis is where you want to compare an observed proportion or percentage to some theoretical proportion / percentage

Name tag example: We think people place name tags on the left side of their chest 90% percent of the time and we want to know if this differs significantly from chance.

Ok, let's use the package to calculate a sample size. What sample size do we need to show this assuming a significance level of 0.05 and a desired power of 0.80?


```r
# first calculate the effect size h
h <- ES.h(p1 = 0.90, p2 = 0.50) #p1 = alternative hypothesis, p2 = null
```

The effect size is calculated using the difference in the arcsined proportions. Check out `?ES.h` for the formula. The arcsine transform is able to capture the magnitude of differences. For example 0.65-0.50 AND 0.16-0.01 both = 0.15 but in the first case 0.65 is 1.3x higher than 0.50 and in the second case 0.16 is 16x higher than 0.01. The arcsine captures this difference.

Conventional effect sizes: small = 0.2, medium = 0.5, and large = 0.8

Now we'll use the effect size we calculated to calculate the sample size.

```r
pwr.p.test(h = h, sig.level = 0.05, power = 0.80, alternative = "greater")
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.9272952
##               n = 7.190053
##       sig.level = 0.05
##           power = 0.8
##     alternative = greater
```
About 8 people; always round up.

Interpretation: if the effect really is this large (h = 0.927), then n=7.19 will give us an 80% chance of correctly rejecting a false null hypothesis (an 80% chance of finding the effect)

Above, we assumed a one-sided test: H0: p = 0.5, H1: p > 0.5 

We recommend that you take the conservative route and calculate sample size based on a two-sided (non-directional) test. That is H0: p = 0.5, H1: p != 0.5. The default alternative in pwr is two.sided.


```r
#now two-sided test
pwr.p.test(h = h, sig.level = 0.05, power = 0.80)
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.9272952
##               n = 9.127904
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
```
Notice we need more people now that we calculated a two-sided test. 

Say we think people place name tags on the left 70% percent of the time instead of 50%. Now how many people do we need to achieve power = 0.80


```r
h <- ES.h(p1 = 0.70, p2 = 0.50)
pwr.p.test(h = h, sig.level = 0.05, power = 0.80)
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.4115168
##               n = 46.34804
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
```
Due to the change in effect size, we now need 47 people!

Let's flip it around and consider what the power would be given a certian n. Let's assume that we are interested in the difference in 70% and 50%. What is the power of our test if we survey 30 people provided we accept a significance level of 0.05? (Notice we can use the ES.h() function in pwr.p.test)


```r
pwr.p.test(h = ES.h(p1 = 0.70, p2 = 0.50), n = 30, sig.level = 0.05)
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.4115168
##               n = 30
##       sig.level = 0.05
##           power = 0.6156361
##     alternative = two.sided
```

Again, notice that the default is to run a two.sided test. This means we're simply testing that the alternative proportion is not 0.5. It could be lower or higher. 

Assuming alternative = "greater" increases the power

```r
pwr.p.test(h = ES.h(p1 = 0.70, p2 = 0.50), n = 30, sig.level = 0.05, alternative = "greater")
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.4115168
##               n = 30
##       sig.level = 0.05
##           power = 0.7287765
##     alternative = greater
```

We can save the result of a sample size estimatation and plot it

```r
pout <- pwr.p.test(h = ES.h(p1 = 0.70, p2 = 0.50), power = 0.80, sig.level = 0.05)
plot(pout)
```

![](PowerSampleSize_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

> Note: Your plot may look different from mine depending on whether the graphics package ggplot2 is installed.

## YOUR TURN
Let's say 16% of the general population has a disease. You would like to know if your patients, 25% of whom show the disease, exhibit the disease significantly more than the general population. Find what sample size you will need to achieve 80% power.

```r
pwr.p.test(h = ES.h(p1 = 0.25, p2 = 0.16), power = 0.80, sig.level = 0.05)
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.2241639
##               n = 156.198
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
```

```r
pwr.p.test(h = ES.h(p1 = 0.25, p2 = 0.16), power = 0.80, sig.level = 0.05, alternative = "greater")
```

```
## 
##      proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.2241639
##               n = 123.0373
##       sig.level = 0.05
##           power = 0.8
##     alternative = greater
```

## pwr.p2n.test
Two-sample test for proportions (ES=h)

This type of analysis is where you want to compare two observed proportions or percentages to each other.

Example: Let's say I want to randomly sample brain cells in mice of 2 different genotypes to see if an equal proportion of cells are activated in each genotype. My null hypothesis is no difference in the proportion of activated cells in each genotype. My alternative hypothesis is that there is a difference. (two-sided; one region has a higher proportion, but I don't know which.) I'd like to detect a difference as small as 15%. 

How many animals do I need to sample in each group if we want 80% power and a significance level of 0.05?


```r
# Effect size depends on what the two proportions are:
ES.h(p1 = 0.85, p2 = 0.70)
```

```
## [1] 0.3638807
```

```r
ES.h(p1 = 0.65, p2 = 0.50)
```

```
## [1] 0.3046927
```

```r
ES.h(p1 = 0.45, p2 = 0.30)
```

```
## [1] 0.3113494
```

```r
#now calculate sample sizes (PER GROUP)
# 85% vs. 70%
pwr.2p.test(h = ES.h(p1 = 0.85, p2 = 0.70), sig.level = 0.05, power = .80)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.3638807
##               n = 118.5547
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

```r
# 65% vs. 50%
pwr.2p.test(h = ES.h(p1 = 0.65, p2 = 0.50), sig.level = 0.05, power = .80)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.3046927
##               n = 169.0879
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

```r
# 45% vs. 30%
pwr.2p.test(h = ES.h(p1 = 0.45, p2 = 0.30), sig.level = 0.05, power = .80)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.3113494
##               n = 161.9349
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

```r
# Sample size is PER GROUP. Always round sample size up.
```

If we don't have any preconceived estimates of proportions or don't feel comfortable making estimates, we can use the conventional effect sizes of 0.2, 0.5, or 0.8 (small, medium, large)

```r
pwr.2p.test(h = .2, sig.level = 0.05, power = .80)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.2
##               n = 392.443
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

```r
pwr.2p.test(h = .5, sig.level = 0.05, power = .80)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.5
##               n = 62.79088
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

```r
pwr.2p.test(h = .8, sig.level = 0.05, power = .80)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.8
##               n = 24.52769
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

```r
# Sample size is PER GROUP. Always round sample size up.
```

## YOUR TURN
Let's say we are only able to sample 20 mice per group but our proportions of activated cells were 90% and 60%. What power did we achieve?

```r
pwr.2p.test(h = ES.h(p1 = 0.90, p2 = 0.60), sig.level = 0.05, n = 20)
```

```
## 
##      Difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.7259373
##               n = 20
##       sig.level = 0.05
##           power = 0.6314435
##     alternative = two.sided
## 
## NOTE: same sample sizes
```

## pwr.2p2n.test

Two-sample test for proportions with unequal n (ES=h)

This type of analysis is where you want to compare two observed proportions or percentages to each other but your 2 samples have unequal n.

Example: Let's stick with the example about the mouse brain activation in 2 different genotypes. Let's say that the wild type genotype is far easier to acquire so we will have more of those as compared to the mutant genotype that we have to specially breed or pay alot for.

It turns out we were able to survey 54 WT mice and 15 mutants. What's the power of our test if we're interested in being able to detect a "large" effect size.

```r
pwr.2p2n.test(h = .8, n1 = 54, n2 = 15, sig.level = 0.05)
```

```
## 
##      difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.8
##              n1 = 54
##              n2 = 15
##       sig.level = 0.05
##           power = 0.7826086
##     alternative = two.sided
## 
## NOTE: different sample sizes
```

Let's say I previously surveyed my 54 WT mice and found that p% of brain cells were activated. I'd like to survey some mutant mice and see if a significantly different proportion of brain cells are activated. How many mutants do I need to sample to detect a large effect size (0.8) in either direction with 90% power and a significance level of 0.05?

```r
pwr.2p2n.test(h = .8, n1 = 54, power = .9, sig.level = 0.05)
```

```
## 
##      difference of proportion power calculation for binomial distribution (arcsine transformation) 
## 
##               h = 0.8
##              n1 = 54
##              n2 = 23.59001
##       sig.level = 0.05
##           power = 0.9
##     alternative = two.sided
## 
## NOTE: different sample sizes
```

# pwr.t.test

One-sample or Two-sample t-test of means (ES=d)

These types of analysis look at means. A one-sample t-test is where you are comparing one observed mean to a theoretical populaton mean. A two-sample t-test is where you want to compare two observed means to each other.

Examples: 

One-sample t-test: You want to test whether blood lead levels in refugee children seen at UVA is the same as the national average for children in the US (2 ug/dl)

Two-sample t-test: You would like to know whether diabetics or non-diabetics show higher mean cholesterol levels. Let's say I randomly observe 30 diabetics and 30 non-diabetics. I would consider a difference of 0.4 mmol/L to be significant and the pooled standard deviation is 1.1 mmol/L. How powerful is this experiment if I want to conduct a two-tailed analysis?


```r
?pwr.t.test
# d = difference in means / pooled SD

myd <- 0.4/1.1
pwr.t.test(n = 30, d = myd, sig.level = 0.05)
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 30
##               d = 0.3636364
##       sig.level = 0.05
##           power = 0.2830956
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

```r
#need way more people to assess this effect size
```

In the above scenario, how many people would be enough to achieve 80% power?

```r
pwr.t.test(power = .8, d = myd, sig.level = 0.05)
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 119.682
##               d = 0.3636364
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

```r
#need 120 people per group
```

Let's say we don't have preliminary data to help inform our estimate of pooled SD. One rule of thumb is to take the difference between the maximum and minimum values and divide by 4 (or 6). Let's say max is 12 mmol/dl and min is 1.5 mmol/dl. So our guess at a standard deviation is: 

```r
(12-1.5)/4
```

```
## [1] 2.625
```

So our estimated d would be:

```r
myd <- 0.4/2.625 # 0.152
pwr.t.test(d = myd, power = 0.80, sig.level = 0.05)
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 677.0061
##               d = 0.152381
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

For any of the pwr functions we can provide multiple sample size values to get multiple power estimates. For example, let's see how power changes as we let n go from 25 to 150 by 25 using an effect size of 0.33.


```r
pwr.t.test(n = seq(from = 25, to = 200, by = 25), d = 0.33, sig.level = 0.05)
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 25, 50, 75, 100, 125, 150, 175, 200
##               d = 0.33
##       sig.level = 0.05
##           power = 0.2080648, 0.3723258, 0.5190757, 0.6413925, 0.7385261, 0.8129181, 0.8682965, 0.9085803
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

We can also save the result of a pwr.t.test sample size estimate and plot it.

```r
pout <- pwr.t.test(d = 0.33, power = 0.80, sig.level = 0.05)
plot(pout)
```

![](PowerSampleSize_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

For a one-sample t-test, use `type = "one.sample"`

```r
?pwr.t.test
```
Notice that there is also a `type = "paired"` available for a two-sample paired t-test

A paired t-test is used when the two samples are related. A before-after design is the quintessential paired t-test design.

For example, 24 high school boys are put on an intensive weight-lifting program. Does this increase their testosterone levels? We'll measure their testosterone before the program and after. We'll use a paired t-test to see if the difference in hormone level is less than 0 (before - after = negative). A average difference in 20 ng/dl is the smallest difference we care to detect and based on what we know about Testosterone, the standard deviation is around 110 ng/dl for this population. How powerful is the test to detect a difference of 20 with 0.05 significance?


```r
pwr.t.test(n = 24, d = -20 / 110, type = "paired", alternative = "less")
```

```
## 
##      Paired t test power calculation 
## 
##               n = 24
##               d = -0.1818182
##       sig.level = 0.05
##           power = 0.2176304
##     alternative = less
## 
## NOTE: n is number of *pairs*
```

## YOUR TURN
Using the pwr.t.test, create a plot to show how many participants we would need in the situation above to achieve 80% power.

```r
pout <- pwr.t.test(power = .8, d = -20 / 110, type = "paired", alternative = "less")
plot(pout)
```

![](PowerSampleSize_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

# pwr.t2n.test

This function is used for a two-sample t test for means with unequal sample sizes (ES=d) 

Find power for a t-test with 28 in one group and 35 in the other group and a medium effect size. (sig.level defaults to 0.05.)


```r
pwr.t2n.test(n1 = 28, n2 = 35, d = 0.5)
```

```
## 
##      t test power calculation 
## 
##              n1 = 28
##              n2 = 35
##               d = 0.5
##       sig.level = 0.05
##           power = 0.4924588
##     alternative = two.sided
```

Find n1 sample size when other group has 35, desired power is 0.80, effect size is 0.5 and significance level is 0.05:

```r
pwr.t2n.test(n2 = 35, d = 0.5, power = 0.8)
```

```
## 
##      t test power calculation 
## 
##              n1 = 321.7572
##              n2 = 35
##               d = 0.5
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
```

Oooh that is alot of people. What if we could deal with only being able to detect a large effect?

```r
pwr.t2n.test(n2 = 35, d = 0.8, power = 0.8)
```

```
## 
##      t test power calculation 
## 
##              n1 = 19.99172
##              n2 = 35
##               d = 0.8
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
```

This is all we have time to cover today. You should know that the pwr package also has functions for calculating power and sample size for Chi Square, correlation, ANOVA, and linear models.

## pwr.chisq.test
Chi-squared tests (ES=w)

## pwr.r.test
correlation test (ES=r)

# pwr.anova.test
balanced one-way ANalysis Of VAriance (ES = f)

## pwr.f2.test
Linear model (ES = f2)

### Further resources
If it appeals to you, also check out the free software program [G* power](http://www.gpower.hhu.de/) with its associated [user guide manual](http://www.gpower.hhu.de/fileadmin/redaktion/Fakultaeten/Mathematisch-Naturwissenschaftliche_Fakultaet/Psychologie/AAP/gpower/GPowerManual.pdf)

