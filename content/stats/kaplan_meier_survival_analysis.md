+++
title = 'Kaplan-Meier Survival Curves'
date = 2024-04-27T14:30:02-04:00
draft = false
comments = true
+++



===========================================================================================

## Survival or Time-to-event analysis

A frequent analysis is a time-to-event, or survival analysis. As the
name states, this can include analyzing survival data (time to death
after treatment, surgery, diagnosis) or this can the analysis of many
different scenarios such as:  
time to progression/relapse  
time to heart attack  
time from unemployment to employment  
time to replacement of a certain part  
employee turnover – time from hiring to quitting   
and the list goes on…   
Actually, one of the inventors of the method, Edward Kaplan, was interested in this type of method out of interest in analyzing the lifetime of light bulbs and
vacuum tubes

The main goal in this type of analysis is to analyze the time from the
starting point until an event occurs. 
For this type of analysis you need
to be able to clearly define what the event is, when the time period
starts and when the time period ends.  
As an example, in a survival analysis, the start time could be when a
patient enters a trial, the end point could be the end of the clinical
trial and the event is death

Special tools have been developed for this kind of analysis as
time-to-event data often includes censoring.

### Censoring:

Censoring occurs when the time to event cannot be determined. This can
happen for several reasons such as a patient dropping out of the study,
a patient or subject is lost during follow-up, or the event of interest
has not occurred by the time the study ends. In all of these cases, the
time to event cannot be determined. And in that case, the patient or
subject is said to be censored.  
**Censored** data: the exact time to event is UNKNOWN. This includes patients alive at the end of the study  
**Non-censored** data: the exact time to the event is KNOWN. The event occurred during the length of the study

![png](../images/survival_curve_trial_design.png)

### Kaplan-Meier (KM) survival curves:

One such tool to work with time-to-event data is called the Kaplan-Meier
Method (or KM curve, or plot). The Kaplan-Meier curve was originally
published by Edward L. **Kaplan** and Paul **Meier** in 1958 and is an
extremely commonly used statistical analysis.

(as a side note, if you search for ‘Kaplan-Meier OR KM-curve’ in PubMed
you get 163,716 results, and that only includes papers which mention
Kaplan-Meier in the abstract! In 2014 it was the 11th most cited paper
in the Web of Science Top 100 papers (van Noorden et al. 2014))

#### What is it?  
The KM survival curve provides a visual representation of survival
probabilities over time. Survival probabilities refer to the estimated
probability that a subject survives (or is event free) beyond a certain
time point. This may make more sense as we look at the examples below.

We can use the lung dataset that comes with the survival package to plot
a survival curve

Load in packages
``` r
library(survival)
library(survminer)
library(tidyverse)
```

``` r
# This code will be explained later
# the lung dataset comes with the survival package

ggsurvplot(survfit(Surv(time, status) ~ 1, data = lung), 
     xlab = "Days", 
     ylab = "Overall survival probability")
```

![png](../images/lung_dataset_curve.png)
You can see we start at 100% survival probability, all patients are
alive at day 0 and at day 1000 we have a very low survival probability
around 5-10%.  
So there is only a 5-10% probability a patient will survive
beyond 1000 days in this study  
The drops in the plot show when an event
has occurred  
The plus signs on the plot indicate the censored patients

But how are these survival probabilities actually calculated? Here is
the formula, and then we’ll look at it step by step  
S(t) = S(t-previous) x (1 - (d(t)/n(t)))

S(t) = Survival probability at time t  
S(t-previous) = survival probability just before time t  
d(t) = events at time t  
n(t) = number of subjects at risk just before time t

We’ll break that equation down below using some sample data.

First let’s create an example dataset of 10 patients

``` r
# create an example dataset to use for our time to event analysis:

data <- data.frame(subject = paste0("subject", "_", 1:10),
                   days = c(5, 10, 15, 12, 10, 20, 26, 30, 26, 29),
                   status = c(1, 1, 0, 0, 1, 1, 1, 1, 0, 0)) # 1=event occurred, 0=censored

head(data)
```

    ##     subject days status
    ## 1 subject_1    5      1
    ## 2 subject_2   10      1
    ## 3 subject_3   15      0
    ## 4 subject_4   12      0
    ## 5 subject_5   10      1
    ## 6 subject_6   20      1

In our example data above, each row is one subject The days column shows
the number of days to the event for each subject  
The status column shows whether the event occurred for each patient (1) or if the patient is
censored (0)

Now we can calculate the survival probabilities by hand:

First we sort the patients from shortest survival time to longest,
regardless of the censoring status

``` r
test <- data # create new dataframe so our original dataframe stays clean

test <- test[order(test$days), ]

test
```

    ##       subject days status
    ## 1   subject_1    5      1
    ## 2   subject_2   10      1
    ## 5   subject_5   10      1
    ## 4   subject_4   12      0
    ## 3   subject_3   15      0
    ## 6   subject_6   20      1
    ## 7   subject_7   26      1
    ## 9   subject_9   26      0
    ## 10 subject_10   29      0
    ## 8   subject_8   30      1

Next, we add a column to show how many patients were at risk, BEFORE
each time point  
For example, at our first time point of 5 days, all patients were alive
and in the study, so 10 patients were at risk  
Then at our next time point, 10 days, 9/10 patients were alive and in the study  
If a patient is censored at a time point, they are no longer considered at risk as
they are no longer in the study and those patients are removed from the
“at risk” number  
For example, at time point day 15, we would say there
are 6 patients at risk (3 had an event occur (one at day 5 and two at
day 10) and 1 was censored (day 12))

``` r
# use a for loop to get the at-risk patients at each time point

test$risk_before_time <- 999 # placeholder, make very large so we can spot mistakes in our for loop output

for (i in 1:nrow(test)){
  event_time <- test$days[i] # time point
  test$risk_before_time[i]  = nrow(test) - length(test$days[test$days < event_time]) # total patients - patients lost before that time point
}

test
```

    ##       subject days status risk_before_time
    ## 1   subject_1    5      1               10
    ## 2   subject_2   10      1                9
    ## 5   subject_5   10      1                9
    ## 4   subject_4   12      0                7
    ## 3   subject_3   15      0                6
    ## 6   subject_6   20      1                5
    ## 7   subject_7   26      1                4
    ## 9   subject_9   26      0                4
    ## 10 subject_10   29      0                2
    ## 8   subject_8   30      1                1

We can do some quick QC checks here – looking at the data, our for loop
output seems correct.  
At day 12, 7 patients remained in the study and at
day 30 only 1 patient remained in the study

Next we add a column to show how many events occurred at each time point  
This value only includes patients with an event NOT censored patients -
this is an important distinction

``` r
test$event_at_timepoint <- 999 # placeholder, make very large so we can spot mistakes in our for loop output

for (i in 1:nrow(test)){
  event_time <- test$days[i] # time point
  test$event_at_timepoint[i]  = length(test$days[test$days == event_time & test$status == 1]) # number of events at that time point
}

test
```

    ##       subject days status risk_before_time event_at_timepoint
    ## 1   subject_1    5      1               10                  1
    ## 2   subject_2   10      1                9                  2
    ## 5   subject_5   10      1                9                  2
    ## 4   subject_4   12      0                7                  0
    ## 3   subject_3   15      0                6                  0
    ## 6   subject_6   20      1                5                  1
    ## 7   subject_7   26      1                4                  1
    ## 9   subject_9   26      0                4                  1
    ## 10 subject_10   29      0                2                  0
    ## 8   subject_8   30      1                1                  1

Looking at the table above, at the 26 days time point, because 1 patient
had an event and the other was censored, we say there was only 1 event
at that time point  
But 2 patients are removed from the at risk value at
the next time point (29 days) as both those patients are no longer in
the study

Now that we’ve calculated our number of at risk patients and number of
events at each time point, let’s reduce our dataframe to keep the
distinct rows for each time point (remove rows when we have duplicated
time points, like at days 10 and 26)

``` r
# remove the subjects and status column
test <- test %>% select(-c(subject, status))

# only keep the distinct rows
test <- distinct(test)
test
```

    ##   days risk_before_time event_at_timepoint
    ## 1    5               10                  1
    ## 2   10                9                  2
    ## 3   12                7                  0
    ## 4   15                6                  0
    ## 5   20                5                  1
    ## 6   26                4                  1
    ## 7   29                2                  0
    ## 8   30                1                  1

Next we calculate the fraction of at-risk patients that survive past
each time point  
This is calculated as: 1 - (number of events at each time point/number
of at risk patients)  
In other words, we’re looking at the fraction of patients that survive
at a given time point

``` r
test$frac_surv <- 1 - (test$event_at_timepoint/test$risk_before_time)
test
```

    ##   days risk_before_time event_at_timepoint frac_surv
    ## 1    5               10                  1 0.9000000
    ## 2   10                9                  2 0.7777778
    ## 3   12                7                  0 1.0000000
    ## 4   15                6                  0 1.0000000
    ## 5   20                5                  1 0.8000000
    ## 6   26                4                  1 0.7500000
    ## 7   29                2                  0 1.0000000
    ## 8   30                1                  1 0.0000000

After the first event at day 5, 9/10 patients, or 90% of the patients
are still alive  
After day 10, 7 of the 9 remaining patients are still
alive (77.8% of the at-risk patients)  
After day 12, we consider the
survival fraction to be 100% because we only lost a patient due to
censoring, so 0 events occurred at day 12 and all remaining patients are
still alive

Next, we calculate the survival probability at each time point   
We can think of this as the probability of surviving up to each time
point X the probability of surviving that time point

To calculate the survival probability at each time point, we multiple
the previous time point survival probability x the current time point
fraction of survival  

Let’s break that down a bit more:  
We start with a survival probability
of 1 (all patients are alive at the beginning of the trial)

``` r
# prev surv probability = 1
# current survival fraction at day 5 = 0.9
# survival probability at time point 5 days = 1 x 0.90 

# Next timepoint (day 10)
# previous survival probability = 0.9
# day 10 survival fraction = 0.77
0.9 * 0.77 # 0.693
```

    ## [1] 0.693

``` r
# Next timepoint (day 12):
# previous survival probability = 0.693 (calculated in previous line)
# day 12 frac_surv = 1
0.693 * 1 # 0.0693
```

    ## [1] 0.693

Use a for loop to compute these values

``` r
# create new column for survival probabilities and assign as 0.9 (the starting surv probability for row 1)
test$surv_prob <- 0.9

# loop through rows 2 to the last row
for (i in 2:nrow(test)){
  prev_surv_prob <- test$surv_prob[i-1] # previous survival probability
  test$surv_prob[i]  = signif(prev_surv_prob * test$frac_surv[i], 3) # previous survival prob x current time point fraction survival
}

test
```

    ##   days risk_before_time event_at_timepoint frac_surv surv_prob
    ## 1    5               10                  1 0.9000000      0.90
    ## 2   10                9                  2 0.7777778      0.70
    ## 3   12                7                  0 1.0000000      0.70
    ## 4   15                6                  0 1.0000000      0.70
    ## 5   20                5                  1 0.8000000      0.56
    ## 6   26                4                  1 0.7500000      0.42
    ## 7   29                2                  0 1.0000000      0.42
    ## 8   30                1                  1 0.0000000      0.00

At the time points when patients were only censored (event_at_timepoint
= 0), you can see the survival probability does not change  
The censored patients only affect the survival probability at the next
time point when an event occurs because the number of at risk patients
is decreased with removal of the censored patient(s)

And there we have it, we’ve now calculated our survival probabilities

Let’s see if our values match those calculated in R using the survival
package

``` r
# use the survfit function from the survival package in R
# formula: survfit(Surv(time, censoring status) ~ grouping variable, data source)
## Make sure your censoring values match what R is expecting (1 = event and 0 = censored)

check <- survfit(Surv(days, status) ~ 1, # ~ 1 indicates no groupings
  data = data
)

summary(check)
```

    ## Call: survfit(formula = Surv(days, status) ~ 1, data = data)
    ## 
    ##  time n.risk n.event survival std.err lower 95% CI upper 95% CI
    ##     5     10       1     0.90  0.0949        0.732        1.000
    ##    10      9       2     0.70  0.1449        0.467        1.000
    ##    20      5       1     0.56  0.1706        0.308        1.000
    ##    26      4       1     0.42  0.1763        0.184        0.956
    ##    30      1       1     0.00     NaN           NA           NA

It looks like it matches!  
At 5 days, both methods show 10 patients at risk, 1 event and a survival
probability of 0.9  
At 10 days both methods show 9 patients at risk, 2
events and a survival probability of 0.7  
At 20 days, both methods show 5
patients at risk, 1 event and a survival probability of 0.56  
Time 12, 15
and 29 are not shown in the output of the survfit method as those were
time points when patients were only censored and no events occurred

Great! It looks like we now understand how the survival probabilities
are calculated!


To do this all of this in R and create a nice survival curve:

``` r
fit <- survfit(Surv(days, status) ~ 1, data = data)

# plot survival curve
ggsurvplot(fit, # survival curve as calculated above
           data = data, # source of data
           surv.median.line = "hv", # show median survival
           censor.shape="|", # censor shape
           censor.size = 4, # censor size
           conf.int = FALSE, # Add confidence interval
           xlab = "Time in days",   # x-axis label
           break.time.by = 5,     # How to break the x-axis
           palette = c("#355070"), # change color of survival curve
           ggtheme = theme_bw() # Change ggplot2 theme
 )
```

![png](../images/km_curve_test_data.png)
We can now clearly see how our data is represented on the survival
curve:  
On the plot above, each drop indicates an event which we know occurred
at time points: 5, 10, 20, 26 and 30  
The vertical lines indicate the
censored datapoints which we know occurred at time points 12, 15 and 29  
The dashed line shows the median survival (when the survival probability
is 50%), which in our case is at 26 days

There are many other options for customization of your survival curve:
<https://rpkgs.datanovia.com/survminer/reference/ggsurvplot.html>

Two quick notes:  
-- Median survival is generally considered rather than
mean as long time-to-event values can skew the mean  
-- we can see how taking censoring into account is very important. If we failed to do that
our estimations could be way off  
If we ignored censoring and estimated the median survival either using
all patients or only those that died, we would either estimate 17.5 or
15 days, which is a shorter median survival than when we correctly
included the censored patients which was at 26 days

``` r
median(data$days)
```

    ## [1] 17.5

``` r
median(data$days[data$status == 1])
```

    ## [1] 15

In the next post, we’ll look at comparing survival curves between groups  
Now you're ready to plot all your time-to-event data!


**survival curve references:**  
Survival Analysis Part I: Basic concepts and first analyses:
<https://www.nature.com/articles/6601118>

video tutorial: <https://youtu.be/ZzFpFUeJ4E8>

A very interesting article that includes information about Edward Kaplan
of the Kaplan-Meier curve:  
Stalpers, L. J. A., & Kaplan, E. L. (2018). Edward L. Kaplan and the
Kaplan-Meier Survival Curve. BSHM Bulletin: Journal of the British
Society for the History of Mathematics, 33(2), 109–135.
