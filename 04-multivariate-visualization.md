# Multivariate Visualization   


```r
library(tidyverse)
library(ggformula)
library(statthink)

# Import data
colleges <- read_csv(
  file = "https://raw.githubusercontent.com/lebebr01/statthink/master/data-raw/College-scorecard-clean.csv", 
  guess_max = 10000
  )
```

Real world data are never as simple exploring a distribution of a single variable, particularly when trying to understand individual variation. In most cases things interact, move in tandem, and many phenomena help to explain the variable of interest. For example, when thinking about admission rates, what may be some important factors that would explain some of the reasons why higher education institutions differ in their admission rates? Take a few minutes to brainstorm some ideas.
 


```r
gf_histogram(~ adm_rate, data = colleges, bins = 30, fill = ~ preddeg) %>%
  gf_labs(x = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          fill = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Often density plots are easier to visualize when there are more than one group. To plot more than one density curve, we need to specify the color argument instead of fill.


```r
gf_density(~ adm_rate, data = colleges, color = ~ preddeg) %>%
  gf_labs(x = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          color = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-2-1.png" width="672" />


```r
gf_density(~ adm_rate, data = colleges, fill = ~ preddeg) %>%
  gf_labs(x = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          fill = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-3-1.png" width="672" />


```r
gf_density(~ adm_rate, data = colleges, fill = ~ preddeg, color = ~ preddeg) %>%
  gf_labs(x = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          color = "Primary Deg",
          fill = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-4-1.png" width="672" />


```r
gf_density(~ adm_rate, data = colleges, color = ~ preddeg, fill = 'gray85', size = 1) %>%
  gf_labs(x = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          color = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-5-1.png" width="672" />
## Violin Plots
 
Violin plots are another way to make comparisons of distributions across groups. Violin plots are also easier to show more groups on a single graph. Violin plots are density plots that are mirrored to be fully enclosed. Best to explore with an example.ArithmeticError


```r
gf_violin(adm_rate ~ preddeg, data = colleges) %>%
  gf_labs(y = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          x = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Aesthetically, these figures are a bit more pleasing to look at if they include a light fill color. This is done similar to the density plots shown above with the `fill = ` argument.ArithmeticError


```r
gf_violin(adm_rate ~ preddeg, data = colleges, fill = 'gray85') %>%
  gf_labs(y = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          x = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Adding quantiles are useful to aid in the comparison with the violin plots. These can be added with the `draw_quantiles` argument.


```r
gf_violin(adm_rate ~ preddeg, data = colleges, fill = 'gray85', draw_quantiles = c(.1, .5, .9)) %>%
  gf_labs(y = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          x = "Primary Deg")
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-8-1.png" width="672" />
### Violin Plots with many groups
 
Many groups are more easily shown in the violin plot framework.

With many groups, it is often of interest to put the long x-axis labels representing each group on the y-axis so that it reads the correct direction and the labels do not run into each other. This can be done with the `gf_refine()` function with `coord_flip()`.


```r
gf_violin(adm_rate ~ region, data = colleges, fill = 'gray80', draw_quantiles = c(.1, .5, .9)) %>%
  gf_labs(y = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          x = "US Region") %>%
  gf_refine(coord_flip())
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-9-1.png" width="672" />

## Facetting
 
Facetting is another way to explore distributions of two or more variables.


```r
gf_violin(adm_rate ~ region, data = colleges, fill = 'gray80', draw_quantiles = c(.1, .5, .9)) %>%
  gf_labs(y = 'Admission Rate (in %)',
          title = 'Multivariate distribution of higher education admission rates by degree type',
          x = "US Region") %>%
  gf_refine(coord_flip()) %>%
  gf_facet_wrap(~ preddeg)
```

<img src="04-multivariate-visualization_files/figure-html/unnamed-chunk-10-1.png" width="672" />



<!-- ## Considering Groups -->
<!-- We've spent a lot of time trying to reason about other variables that may be important in explaining variation in our variable of interest. So far we have only explored the variable without considering other variables, in practice that is not that useful. -->

<!-- Instead, it is common to compute conditional statistics based on other characteristics in the data. An example may help to show the idea more clearly. -->

<!-- ```{r} -->
<!-- colleges %>% -->
<!--   df_stats(adm_rate ~ region, median) -->
<!-- ``` -->

<!-- Presented above are the conditional medians for the higher education institutions in different areas of the country. More specifically, the data are essentially split into subgroups and the median is computed for each of those subgroups instead of pooling all institutions into a single data frame. The formula syntax is now `outcome ~ grouping` where the variable of interest (i.e. commonly a numeric variable) and the variable to the right of the `~` is the grouping variable. This syntax is similar to the violin plots that were created earlier. -->

<!-- Can you see differences in the admission rates across the regions? -->

<!-- One thing that is useful to add in when computing conditional statisics, is how many data points are in each group. This is particularly useful when the groups are different sizes, which is common. To do this, we can add another function to the `df_stats()` function. -->

<!-- ```{r} -->
<!-- colleges %>% -->
<!--   df_stats(adm_rate ~ region, median, length) -->
<!-- ``` -->

<!-- This adds another columns which represents the number of observations that went into the median calculation for each group. The syntax above also shows that you can add additional functions separated by a comma in the `df_stats()` function and are not limited to a single function. We will take advantage of this feature later on. -->

<!-- ### Adding additional groups -->
<!-- What if we thought more than one variable was important in explaining variation in the outcome variable? These can also be added to the `df_stats()` function for additional conditional statistics. The key is to add another variable to the right-hand side of the formula argument. More than one variable are separated with a `+` symbol. -->

<!-- ```{r} -->
<!-- colleges %>% -->
<!--   df_stats(adm_rate ~ region + preddeg, median, length) -->
<!-- ``` -->

<!-- ## Other statistics of center -->
<!-- So far we have been discussing the median. The median attempts to provide a single number summary for the center of the distribution. It is a robust statistic, but likely isn't the most popular statistic to provide a location for the center of a distribution. The mean is often more commonly used as a measure of the center of a distribution. Part of this is due to the usage of the mean in common statistical methods and the mean also uses the values of all the data in the calculation. The median only considers the values of the middle score or scores, therefore this statistic is less sensitive to extreme values than the mean. I like to look at both statistics and this can provide some insight into the distribution of interest. We can add the mean using the `df_stats()` function by adding the function `mean`. -->

<!-- ```{r} -->
<!-- stats_compute <- colleges %>% -->
<!--   df_stats(adm_rate ~ region, median, mean, length) -->
<!-- stats_compute -->
<!-- ``` -->

<!-- Do you notice any trends in the direction the mean and median typically follow? More specifically, is the mean typically larger than the median or vice versa? -->

<!-- Let's visualize them. -->

<!-- ```{r} -->
<!-- gf_histogram(~ adm_rate, data = colleges, bins = 30) %>% -->
<!--   gf_facet_wrap(~ region) %>% -->
<!--   gf_vline(color = 'blue', xintercept = ~ median_adm_rate, data = stats_compute, size = 1) %>% -->
<!--   gf_vline(color = 'lightblue', xintercept = ~ mean_adm_rate, data = stats_compute, size = 1) -->
<!-- ``` -->

<!-- What is different about the distributions that have larger differences in the mean and median? -->

<!-- ## Measures of Variation -->

<!-- So far we have focused primarily on applying functions to columns of data to provide a single numeric summary for where the center of the distribution may lie. The center of the distribution is important, however the primary goal in research and with statistics is to try to understand the variation in the distribution. -->

<!-- One crude measure of variation that is intuitive is the range of a variable. The range is the difference between the smallest and the largest number in the data. We can compute this with the `df_stats()` function. -->

<!-- ```{r} -->
<!-- colleges %>% -->
<!--   df_stats(~ adm_rate, range) -->
<!-- ``` -->

<!-- The details of the `df_stats()` function are in the previous course notes. The output for this computation returns two values, the minimum and maximum value in the data and unsurprisingly, is 0 and 1 respectively.  -->

<!-- ### Robust measure of variation -->
<!-- The idea behind the IQR representing differences in percentiles allows us to extend this to different percentiles that may be more directly interpretable for a given situation. For example, suppose we wanted to know how spread out the middle 80% of the distribution is. We can do this directly by computing the 90th and 10th percentiles and finding the difference between the two. -->

<!-- ```{r} -->
<!-- mid_80 <- colleges %>% -->
<!--   df_stats(~ adm_rate, quantile(c(0.1, 0.9)), nice_names = TRUE) -->
<!-- mid_80 -->
<!-- ``` -->

<!-- As you can see, once you extend the amount of the distribution contained, the distance increases, now to 0.555 or 55.5% the the range of the middle 80% of the admission rate distribution. We can also visualize what this looks like. -->

<!-- ```{r} -->
<!-- gf_histogram(~ adm_rate, data = colleges, bins = 30, color = 'black') %>% -->
<!--   gf_vline(color = 'blue', xintercept = ~ value, data = gather(mid_80), size = 1) -->
<!-- ``` -->

<!-- We can also view the exact percentages using the empirical cumulative density function. -->

<!-- ```{r} -->
<!-- gf_ecdf(~ adm_rate, data = colleges) %>% -->
<!--   gf_vline(color = 'blue', xintercept = ~ value, data = gather(mid_80), size = 1) -->
<!-- ``` -->

<!-- ### Variation by Group -->
<!-- These statistics can also be calculated by different grouping variables similar to what was done with statisitcs of center. Now the variable of interest is on the left-hand side of the equation and the grouping variable is on the right hand side. -->

<!-- ```{r} -->
<!-- iqr_groups <- colleges %>% -->
<!--   df_stats(adm_rate ~ region, IQR, quantile(c(0.25, 0.75)), nice_names = TRUE) -->
<!-- iqr_groups -->
<!-- ``` -->

<!-- This can also be visualized to see how these statistics vary across the groups. -->

<!-- ```{r} -->
<!-- gf_histogram(~ adm_rate, data = colleges, bins = 30, color = 'black') %>% -->
<!--   gf_vline(color = 'blue', xintercept = ~ value,  -->
<!--      data = filter(pivot_longer(iqr_groups, IQR_adm_rate:'X75.'), name %in% c('X25.', 'X75.')), size = 1) %>% -->
<!--   gf_facet_wrap(~ region) -->
<!-- ``` -->

