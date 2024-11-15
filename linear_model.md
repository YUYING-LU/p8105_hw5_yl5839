p8015_hw5_yl5839
================
Yuying Lu
2024-11-13

``` r
library(tidyverse)
knitr::opts_chunk$set(
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)

theme_set(theme(legend.position = "bottom"))

options(
  ggplot2.continuous.colour = "viridis",
  ggplot2.continuous.fill = "viridis"
)

scale_colour_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

# Problem 1

## Set function

``` r
birth_share = function(group_size){
  birth_select = sample(1:365,group_size,replace=TRUE)
  return(length(unique(birth_select))<group_size)
}
```

Running this function for 10000 times for each group size between 2 and
50, calculate the probability of birthday sharing and make the plot.

``` r
set.seed(1)

sim_results_df = 
  expand_grid(
    group_size = 2:50,
    iter = 1:10000
  ) |> 
  mutate(
    birth_share = map(group_size, birth_share)
  ) |> 
  unnest(birth_share) |> 
  group_by(group_size) |> 
  summarise(probability = mean(birth_share))

sim_results_df |>
  mutate(
    group_size = str_c(group_size),
    group_size = fct_inorder(group_size)) |> 
  ggplot(aes(x = group_size, y = probability))+
  geom_point()+
  geom_line()
```

<img src="linear_model_files/figure-gfm/unnamed-chunk-3-1.png" width="90%" />

**Comment:** According to the picture based on the 10000 times
simulation, we can see the probability that at least two people in the
group will share a birthday increases as the group size increases. And
the probability will converges to 1 when the group size is large.

# Problem 2

Generate 5000 datasets, each with $n=30$ samples from
$N(\mu=0,\sigma=5)$ and or each data set, calculate the sample mean
$\hat{\mu}$ and p-value of the hypothesis test with $\alpha = 0.05$.

``` r
set.seed(2)
norm_df = 
  expand_grid(
    mu_true = 0,
    iter = 1:5000
  ) |> 
  mutate(
    norm_sample = map(mu_true, \(mu) rnorm(n = 30, mean = mu, sd = 5))
  ) |> 
  mutate(
  test_result = map(norm_sample, \(df) broom::tidy(t.test(df, mu = 0)))
) |> 
  unnest(test_result) |> 
  select(mu_true, iter, estimate, p.value)

norm_df 
```

    ## # A tibble: 5,000 × 4
    ##    mu_true  iter estimate p.value
    ##      <dbl> <int>    <dbl>   <dbl>
    ##  1       0     1    1.14    0.295
    ##  2       0     2   -0.175   0.868
    ##  3       0     3   -1.23    0.226
    ##  4       0     4    0.901   0.381
    ##  5       0     5    0.337   0.735
    ##  6       0     6   -0.180   0.845
    ##  7       0     7   -0.464   0.587
    ##  8       0     8    0.495   0.634
    ##  9       0     9    0.865   0.320
    ## 10       0    10    0.666   0.526
    ## # ℹ 4,990 more rows

The column named `estimate` represents $\hat{\mu}$.

Then repeat the above process for $\mu = \{1,2,3,4,5,6\}$:

``` r
set.seed(3)
new_norm_df = 
  expand_grid(
    mu_true = 1:6,
    iter = 1:5000
  ) |> 
  mutate(
    norm_sample = map(mu_true, \(mu) rnorm(n = 30, mean = mu, sd = 5))
  ) |> 
  mutate(
  test_result = map(norm_sample, \(df) broom::tidy(t.test(df, mu = 0)))
) |> 
  unnest(test_result) |> 
  select(mu_true, iter, estimate, p.value)
```

Next we plot the proportion of times the null was rejected

``` r
new_norm_df |> 
  mutate(reject = p.value < 0.05) |> 
  group_by(mu_true) |> 
  summarise(reject_proportion = mean(reject)) |> 
  ggplot(aes(x = mu_true, y = reject_proportion))+
  geom_point()+
  geom_line()+
  xlab(expression(mu))+
  scale_x_continuous(breaks = 1:6)
```

<img src="linear_model_files/figure-gfm/unnamed-chunk-6-1.png" width="90%" />

**Association:** When $\mu \in \{1,...,6\}$, a larger $\mu$ has a higher
reject proportion.

Showing average $\mu$ and the average $\mu$ when null was rejected. v.s.
true $\mu$

``` r
new_norm_df |> 
  mutate(reject_mu = ifelse(p.value < 0.05, estimate, NA)) |> 
  group_by(mu_true) |> 
  summarise(average_mu = mean(estimate),
            average_mu_rejected = mean(reject_mu,na.rm = TRUE)) |> 
  pivot_longer(
    average_mu:average_mu_rejected,
    names_to = "type",
    values_to = "hat_mu"
  ) |> 
  ggplot(aes(x = mu_true, y = hat_mu, color = type))+
  geom_point(alpha = 0.5)+
  geom_abline()+
  xlab(expression(mu))+
  ylab(expression(hat(mu)))+
  scale_x_continuous(breaks = 1:6)+
  scale_y_continuous(breaks = 1:6)
```

<img src="linear_model_files/figure-gfm/unnamed-chunk-7-1.png" width="90%" />

From the plot, the sample average of $\hat{\mu}$ across tests for which
the null is rejected isn’t approximately equal to the true value of
$\mu$, when the true $\mu$ is close to $0$. The reason is when
calculating the average of the estimate $\hat{\mu}$ in the sample that
the null was rejected (we call it `average_mu_rejected`), we actually
excluded the estimate $\hat{\mu}$ that is approximate to 0. By the Law
of Large Number theorem, the average of all the estimates $\hat{\mu}$
(we call it `average_mu`) converges to the true $\mu$. However, when we
exclude the estimates close to 0, the rest average increases, leading to
a estimating bias. Additionally, when the true $\mu$ is close to 0,
excluding the estimate that was rejected can result in a more
significant deviation of the `average_mu_rejected` from the true $\mu$.
This exclusion creates a larger bias, as illustrated in the figure.

# Problem 3

``` r
url = "https://raw.githubusercontent.com/washingtonpost/data-homicides/master/homicide-data.csv"

homi_df = read_csv(url)
```
