Lab 09 - Grading the professor, Pt. 1
================
Lindsey Wilson
4/24/23

### Load packages and data

``` r
library(tidyverse) 
library(tidymodels)
library(openintro)
```

``` r
evals <- evals
```

## Part 1

### Exercise 1

The distribution of `score` is visualized below:

``` r
ggplot(evals,
       aes(x = score)) + 
  geom_histogram(stat = "count",  fill = "Grey") +
  labs(title = "Professor Evaluation Scores",
       subtitle = "Average evaluation represented by solid black line") + 
  geom_vline(xintercept = mean(evals$score), color = "Black")
```

    ## Warning in geom_histogram(stat = "count", fill = "Grey"): Ignoring unknown
    ## parameters: `binwidth`, `bins`, and `pad`

![](lab-09_files/figure-gfm/score-dist-1.png)<!-- --> It looks like we
have some pretty substantial negative skew, which is what I expected to
see. Students don’t really give negative course/professor evals unless
they had an extremely negative experience in the class.

### Exercise 2

Next let’s visualize the relationship between average evaluation score
and average beauty rating:

``` r
ggplot(evals,
       aes( x = bty_avg,
            y = score)) +
  geom_point() + 
  geom_smooth() +
  labs(title = "Average Professor Evaluation over Average Beauty Rating")
```

![](lab-09_files/figure-gfm/score-bty_avg-correlation-1.png)<!-- -->

``` r
cor(evals$bty_avg, evals$score)
```

    ## [1] 0.1871424

It looks like there’s a weak positive correlation between average beauty
rating and average professor evaluation. More attractive professors get
better ratings, but the effect isn’t huge.

### Exercise 3

Let’s replot the above data using `geom_jitter()` instead of
`geom_point()`:

``` r
ggplot(evals,
       aes( x = bty_avg,
            y = score)) +
  geom_jitter() + 
  geom_smooth() +
  labs(title = "Average Professor Evaluation over Average Beauty Rating")
```

![](lab-09_files/figure-gfm/score-bty_avg-jitter-1.png)<!-- -->

This is likely a better representation of the relationship between
beauty and professor evals, because it adds a bit of random variation
that wasn’t present with `geom_point()`. Individuals only had a certain
number of discrete options (1-5) to indicate their ratings of the
professor, so the averages of those ratings were constrained to only be
able to take on certain values. `geom_jitter()` fixes that problem by
adding randomness to the average points in the dataset. That’s why the
points don’t fall into neat rows and columns like they did in Exercise
2.

## Part 2

### Exercise 4

Let’s fit a linear model to the relationship between mean beauty rating
and professor evaluations:

``` r
m_bty <- linear_reg() %>%
  set_engine("lm") %>%
  fit(score ~ bty_avg, data = evals)

m_bty
```

    ## parsnip model object
    ## 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ bty_avg, data = data)
    ## 
    ## Coefficients:
    ## (Intercept)      bty_avg  
    ##     3.88034      0.06664

Based on this output,the formula for the linear model is: `score` =
0.067(`bty_avg`) + 3.88

### Exercise 5

Below I’ve added the line we found above to the plot from Exercise 3

``` r
ggplot(evals,
       aes( x = bty_avg,
            y = score)) +
  geom_jitter() + 
  geom_smooth(method = "lm", se = FALSE, color = "Orange", fullrange = TRUE) +
  labs(title = "Average Professor Evaluation vs. Average Beauty Rating")
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](lab-09_files/figure-gfm/Ex-3-replot-1.png)<!-- -->

### Exercise 6

In context, the slope of our linear regression means that, on average,
an increase of 1 point in average beauty rating is associated with a .06
point increase in a professor’s average evaluation

### Exercise 7

In context, our intercept means that a professor with a mean beauty
rating of zero would be predicted to have an average evaluation of 3.88
if we considered nothing else. This number isn’t really super
meaningful; it’s pretty unlikely anyone would ever actually get a zero
for mean beauty score, and even if they did, professor evaluation is
presumably determined by factors other than just attractiveness.

### Exercise 8

Our r-squared is calculated below:

``` r
glance(m_bty)$r.squared
```

    ## [1] 0.03502226

This means that about 3.5% of the variance in `score` is explained by
variance in `bty_avg`

## Part 3

### Exercise 9

Below is a linear regression model that predicts score from gender:

``` r
m_gen <- linear_reg() %>%
  set_engine("lm") %>%
  fit(score ~ gender, data = evals) %>%
  tidy()

m_gen
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>   <dbl>
    ## 1 (Intercept)    4.09     0.0387    106.   0      
    ## 2 gendermale     0.142    0.0508      2.78 0.00558

This output produces the following equation: `score` =
0.1415(`gender`) + 4.0928, where a gender code of 0 means female and a
gender code of 1 mean male. This means that female professors on average
recieve evaluations of 4.0928, and male professors on average score
0.1415 points higher than that.

### Exercise 10

The equation for female professors is: `score` = 4.09 The equation for
male professors is: `score` = 4.23

### Exercise 11

``` r
m_rank <- linear_reg() %>%
  set_engine("lm") %>%
  fit(score ~ rank, data = evals) %>%
  tidy()

m_rank
```

    ## # A tibble: 3 × 5
    ##   term             estimate std.error statistic   p.value
    ##   <chr>               <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)         4.28     0.0537     79.9  1.02e-271
    ## 2 ranktenure track   -0.130    0.0748     -1.73 8.37e-  2
    ## 3 ranktenured        -0.145    0.0636     -2.28 2.28e-  2

Based on this, it looks like the average evaluation for a teaching track
professor is 4.28 and that this decreases by 0.13 points on average for
tenure track professors and 0.15 points on average for tenured
professors. The overall equation is `score` = -0.1297(`tenure track`)
-0.1452(`tenured`) + 4.2843.

### Exercise 12 + 13

Let’s adjust create a new rank variable where “tenure track” is the new
baseline level, and use it to create a new linear model:

``` r
evals <- evals %>%
  mutate(rank_relevel = relevel(evals$rank, ref = 2 ))

m_rank_relevel <- linear_reg() %>%
  set_engine("lm") %>%
  fit(score ~ rank_relevel, data = evals)

m_rank_relevel
```

    ## parsnip model object
    ## 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ rank_relevel, data = data)
    ## 
    ## Coefficients:
    ##          (Intercept)  rank_relevelteaching   rank_releveltenured  
    ##               4.1546                0.1297               -0.0155

``` r
glance(m_rank_relevel)$r.squared
```

    ## [1] 0.01162894

Based on this output, it looks like the predicted score for a tenure
track professor is 4.15, which increases by 0.13 if you’re a teaching
professor and decreases by .015 if you’re a tenured professor. The
overall equation would be `score` = 0.1297(`teaching`) -
0.0155(`tenured`) + 4.1546.

The R-squared for this model is 0.0116, meaning that rank explains just
over 1% of the variance in evaluations

### Exercise 14

Here we’ll create a new variable called `tenure_eligible` that takes the
value of “no” for teaching professors and “yes” for tenure and tenure
track professors:

``` r
evals <- evals %>%
  mutate(tenure_eligible = case_when(evals$rank == "teaching" ~ "no",
                                     evals$rank == "tenured" ~ "yes",
                                     evals$rank == "tenure track" ~ "yes"))
```

### Exercise 15

And we can now use this new variable in a linear regression to predict
score:

``` r
m_tenure_eligible <- linear_reg() %>%
  set_engine("lm") %>%
  fit(score ~ tenure_eligible, data = evals)

tidy(m_tenure_eligible)
```

    ## # A tibble: 2 × 5
    ##   term               estimate std.error statistic   p.value
    ##   <chr>                 <dbl>     <dbl>     <dbl>     <dbl>
    ## 1 (Intercept)           4.28     0.0536     79.9  2.72e-272
    ## 2 tenure_eligibleyes   -0.141    0.0607     -2.32 2.10e-  2

``` r
glance(m_tenure_eligible)$r.squared
```

    ## [1] 0.01149589

Based on this output, it looks like the average score for a professor
without tenure eligibility is 4.28, which decreases by 0.14 once they
become eligible for tenure. The overall equation is `score` =
-0.1405(`tenure_eligible`) + 4.2843.

Also, R-squared is 0.0115, which means that tenure eligibility explains.
1.15% of evaluation variance. This is just slightly less than the
R-squared for `m_rank_relevel`, meaning that knowing whether you’re a
tenured vs. tenure track professor does allow us to predict score
slightly more accurately, but not by much.
