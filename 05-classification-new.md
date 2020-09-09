# Classification   

Classification is a task that tries to predict group membership using the data attributes available. For example, one could try to predict if an individual is left or right handed based on their preferences to music or hobbies. In this situation, the classification technique would look for patterns in the music or hobby preferences to help differentiate between those that are left or right handed. Perhaps the data would show that left handed individuals would be more likely to be artistic, therefore those that rated more highly artistic tasks would be more likely to be classified as left handed.

To perform the classification tasks in this chapter, we are going to consider a group of **statistical models** called decision trees, or more specifically in this case, classification trees. Statistical models are used to help us as humans understand patterns in the data and estimate uncertainty. Uncertainty comes from the variation in the data. For example, those that are left handed are likely not all interested or like artistic hobbies or tasks, but on average maybe they are more likely to enjoy these tasks compared to right handed individuals. Statistical models help us to understand if the differences shown in our sample of data are due to signal (true differences) or noise (uncertainty). 

In the remaining sections of this chapter, we will build off of this idea of statistical models to understanding how these work with classification trees to classify. Furthermore, we will aim to develop heuristics to understand if our statistical model is practically useful. That is, does our model help us to do our classification task above just randomly guessing. We will use a few additional packages to perform the classification tasks, including `rpart`, `rpart.plot`, and `rsample`. The following code chunk loads all the packages that will be used in the current chapter. 


```r
library(tidyverse)
library(ggformula)
library(statthink)
library(rpart)
library(rpart.plot)
library(rsample)

remotes::install_github("grantmcdermott/parttree")
library(parttree)

# Add plot theme
theme_set(theme_statthinking())

us_weather <- mutate(us_weather, snow_factor = factor(snow))
```

## Topic: Decision Trees 

We will continue to use the United States weather data introduced in Chapter 4. Given that this data was for the winter months in the United States, the classification task we will attempt to perform is to correct predict if it will snow on a particular day that precipitation occurs. To get a sense of how often it rains vs snows in these data, we can use the `count()` function to do this. For the `count()` function, the first argument is the name of the data, followed by the attributes we wish to count the number of observations in the unique values of those attributes.


```r
count(us_weather, rain, snow)
```

```
## [90m# A tibble: 4 x 3[39m
##   rain  snow      n
##   [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<int>[39m[23m
## [90m1[39m No    No     [4m1[24m571
## [90m2[39m No    Yes     728
## [90m3[39m Yes   No      821
## [90m4[39m Yes   Yes     280
```

The following table counts the number of times that it rains or snows in the data. You may notice that there are days in which it does not rain or snow as shown by the row with No for both the rain and snow columns. There are also days in which it both rains and snows as shown in the row with Yes in both the rain and snow columns. Not surprisingly, a majority of the days it does not rain or snow, occurring about 46% of the time ($1571 / (1571 + 728 + 821 + 280) = 46.2%$). Using similar logic, about 8% of the days in the data have both snow and rain. 

<!--
For the current classification, we will focus on days in which it either rains or snows instead of both or none. To do this, we will filter or restrict that data to those cases only, a data task that can be done with the `filter()` function. The `filter()` function works by selecting rows of data that match specific situations. This is similar to how a search engine, such as Google, works. In a search engine, you type is search criteria and the search engine gives you back matches that meet those criteria. The `filter()` function works similarly, we specify the criteria with which we hope to have data match to keep. 

In this example, we hope to keep the rows where it either rains or snows, but rows where it doesn't or both rains or snows. To do this, we will use the `|` and `&` operators which can be translated into "or" or "and" respectively. In English, the cases we want to keep are days in which it does not rain **and** it does snow *or* days in which it does rain **and** it does not snow. Turning this into data language that `filter()` can understand, we can substitute the `|` or `&` operators into places where or/and are in the English version along with the data attribute names. Therefore we could write in code, `(rain & snow) | (rain & snow)`. The last piece we need to add to the code version, is the values we want to retain for the attributes. For example, in the data, "No" means that the event did not occur and "Yes" means the event did occur. Therefore, translating the English version into code with data values would look like: `(rain == 'No' & snow == 'Yes') | (rain =='Yes' & snow == 'No')`. In R, the `==` means literal values, therefore the code `rain == 'No'` means the literal word "No" within the rain attribute. 


```r
us_weather_rs <- us_weather %>%
  filter((rain == 'No' & snow == 'Yes') | (rain == 'Yes' & snow == 'No'))

count(us_weather_rs, rain, snow)
```

```
## [90m# A tibble: 2 x 3[39m
##   rain  snow      n
##   [3m[90m<chr>[39m[23m [3m[90m<chr>[39m[23m [3m[90m<int>[39m[23m
## [90m1[39m No    Yes     728
## [90m2[39m Yes   No      821
```

We can further check if the filtering command worked by using the `count()` function. Using the `count()` function on the filtered data shows that we only retained rows of the data for the combinations that we wanted, namely days in which it there is some form of precipitation, but only if it snowed or rained, not both.
-->

### Fitting a Classification Tree

Let's class_tree our first classification tree to predict whether it snowed on a particular day. For this, we will use the `rpart()` function from the *rpart* package. The first argument to the `rpart()` function is a formula where the outcome of interest is specified to the left of the `~` and the attributes that are predictive of the outcome are specified to the right of the `~` separated with `+` signs. The second argument specifies the method for which we want to run the analysis, in this case we want to classify days based on the values in the data, therefore we specify `method = 'class'`. The final argument is the data element, in this case `us_weather`.

Before we fit the model, what attributes do you think would be predictive of whether it will rain or snow on a particular day during the winter months? Take a few minutes to brainstorm some ideas.

In this example, a handful of attributes to explore, including the average, minimum, and maximum temperature for the day. These happen to be all continuous attributes, meaning that these attributes can take many data values. The model is not limited to those types of data attributes, but that is where we will start the classification journey. 

Notice that the fitted model is saved to the object, `class_tree`. This will allow for easier interaction with the model results later. Then after fitting the model, the model is visualized using the `rpart.plot()` function. The primary argument to the `rpart.plot()` function is the fitted model object from the `rpart()` function, here that would be `class_tree`. The additional arguments passed below adjust the appearance of the visualization. 


```r
class_tree <- rpart(snow_factor ~ drybulbtemp_min + drybulbtemp_max, 
   method = 'class', data = us_weather)

rpart.plot(class_tree, roundint = FALSE, type = 3, branch = .3)
```

<div class="figure">
<img src="05-classification-new_files/figure-html/first-class-tree-1.png" alt="Classification tree predicting whether it will snow or rain" width="672" />
<p class="caption">(\#fig:first-class-tree)Classification tree predicting whether it will snow or rain</p>
</div>

The visualization shown in Figure \@ref(fig:first-class-tree) produces the decision rules for the classification tree. The decision rules start from the top of the tree and proceed down the branches to the leaf nodes at the bottom that highlight the predictions. By default, the `rpart()` algorithm assumes that each split should go in two directions. For example, the first split occurs with the maximum temperature is less than 42 degrees Fahrenheit or greater than or equal to 42 degrees Fahrenheit. If the maximum temperature for the day is greater than or equal to 42 degrees Fahrenheit, the first split in the decision tree follows the left-most branch and proceeds to the left-most leaf node. This results in the prediction for those days as being days in which it does not snow (i.e., a category prediction of "No"). The numbers below the "No" label indicate that the probability of it snowing on a day where the maximum temperature was greater than or equal to 42 degrees Fahrenheit is 0.09 or about 9%. Furthermore, this category represents about 53% of the total number of data cases inputted. 

Following the right-hand split of the first decision, which occurs for days when the maximum temperature is less than 42 degrees, we come to another split. This split is again for the maximum temperature, but now the split comes at 36 degrees Fahrenheit. In this case, if the temperature is greater than or equal to 36 degrees Fahrenheit, the decision leads to the next leaf node and a prediction that it will not snow that day. For this leaf node, there is more uncertainty in the prediction, where on average the probability of it snowing would be 0.42 or about 42%. This value is less than 50%, therefore the "No" category is chosen. This occurs for about 16% of the data. 

For days in which the maximum temperature is less than 36 degrees Fahrenheit, the decision tree moves to the right further and comes to another split. The third split in the decision tree is for the minimum daily temperature and occurs at 23 degrees Fahrenheit. For days where the minimum temperature is greater than 23 degrees Fahrenheit (but also had a maximum temperature less than 36 degree Fahrenheit), the right-most leaf node is predicted. For these data cases, about 8% of the total data, the prediction is that it will snow (i.e., "Yes" category) and the probability of it snowing in those conditions is about 71%. 

Finally, if the minimum temperature is less than 23 degrees Fahrenheit (but also had a maximum temperature less than 36 degree Fahrenheit), then one last split occurs on the maximum temperature at 29 degrees Fahrenheit. This leads to the last two leaf node in the middle of Figure \@ref(fig:first-class-tree). One prediction states it will snow, for maximum temperature less than 29 degrees and one predicting it will not snow, for those greater than or equal to 29 degrees. Both of these leaf nodes have more uncertainty in the predictions, being close to 50% probability.

Note, that the average daily temperature was included in the model fitting procedure, but was not included in the results shown in Figure \@ref(fig:first-class-tree). Why do you think this happened? The model results show the attributes that were helpful in making the prediction of whether it snowed or not. For this task, the model found that the maximum and minimum temperature attributes were more useful and adding the average daily temperature did not appreciably improve the predictions. For this reason, it did not show up in the decision tree. Furthermore, the attributes that are most informative in making the prediction are at the top of the decision tree. In the results shown in Figure \@ref(fig:first-class-tree), the maximum daily temperature was the most helpful attribute in making the snow or not prediction.

The decision tree rules can also be requested in text form using the `rpart.rules()` function and are shown below. The rows in the output are the leaf nodes from \@ref(fig:first-class-tree) and the columns represent the probability of it snowing, the decision rules that are applicable, and the percentage of data found in each row. For example, for the first row, it is predicted to snow about 9% of the time when the maximum temperature for the day is greater than 42 and this occurred in 53% of the original data. Since the probability is less than 50%, the prediction would be that it would not snow on days with those characteristics. In rows where there are `&` symbols, these separate different data attributes that are useful in the classification model.


```r
rpart.rules(class_tree, cover = TRUE)
```

```
##  snow_factor                                                            cover
##         0.09 when drybulbtemp_max >=       42                             53%
##         0.42 when drybulbtemp_max is 36 to 42                             16%
##         0.45 when drybulbtemp_max is 29 to 36 & drybulbtemp_min <  23      9%
##         0.63 when drybulbtemp_max <  29       & drybulbtemp_min <  23     14%
##         0.71 when drybulbtemp_max <  36       & drybulbtemp_min >= 23      8%
```

#### Visualizing Results

To get another view of what the classification model is doing in this scenario, we will visualize the study results. First, the `gf_point()` function is used to create a scatterplot where the maximum temperature is shown on the x-axis and the minimum temperature is shown on the y-axis, shown in Figure \@ref(fig:scatter-usweather). There is a positive relationship between maximum and minimum temperatures and on days with lower maximum temperatures are where it tends to snow. However, there is not perfect separation, meaning that there are days that have similar minimum and maximum temperatures where it does snow and other where it does not snow. 


```r
temperature_scatter <- gf_point(drybulbtemp_min ~ drybulbtemp_max, 
                                color = ~ snow_factor,
                                alpha = .75,
                                data = us_weather) %>%
  gf_labs(x = "Maximum Temperature (in F)",
          y = "Minimum Temperature (in F)",
          color = "Snow?")

temperature_scatter
```

<div class="figure">
<img src="05-classification-new_files/figure-html/scatter-usweather-1.png" alt="Scatterplot of the minimum and maximum daily temperatures and if it snows or not" width="672" />
<p class="caption">(\#fig:scatter-usweather)Scatterplot of the minimum and maximum daily temperatures and if it snows or not</p>
</div>

The next figure will make use of the *parttree* R package to visualize what the classification model is doing. The `geom_parttree()` function is used where the primary argument is the saved classification model object that was save earlier, named `class_tree`. The other two arguments to add are the fill aesthetic that is the outcome of the classification tree and to control how transparent the backgroud fill color is. In this example, this is set using `alpha = .25` where the transparency is set at 75% (i.e., 1 - 0.25 = 0.75). Setting a higher alpha value would reduce the amount of transparency, whereas setting a smaller value would increase the transparency. 

Figure \@ref(fig:predict-usweather) gives a sense as to what the classification model is doing to the data. The classification model breaks the data into quadrants and makes a single uniform prediction for those quadrants. For example, the areas of the figure that are shaded as red are days in which the model predicts **it will not snow** whereas the blue/green color are days in which the model predicts **it will snow**. The data points are the real data cases, therefore there are instances inside each of the quadrants in which the model did not correctly predict or classify the case. Each of the quadrants in the figure represent different leaf nodes shown in \@ref(fig:first-class-tree) and each represent a different likelihood of it snowing. 


```r
temperature_scatter + 
  geom_parttree(data = class_tree, aes(fill = snow_factor), alpha = .25) + 
  scale_fill_discrete("Snow?")
```

<div class="figure">
<img src="05-classification-new_files/figure-html/predict-usweather-1.png" alt="Showing the predictions based on the classification tree with the raw data" width="672" />
<p class="caption">(\#fig:predict-usweather)Showing the predictions based on the classification tree with the raw data</p>
</div>

<!--
### Pruning Trees

One downside of decision trees, is that they can tend to overfit the data and capitalize on chance variation in our sample that we can not generalize to another sample. This means that there are features in the current sample that would not be present in another sample of data. There are a few ways to overcome this, one is to prune the tree to only include the attributes that are most important and improve the classification accuracy. One measure of this can be used is called the complexity parameter (CP) and this statistic attempts to balance the tree complexity related to how strongly the levels of the tree improve the classification accuracy. We can view these statistics with the printcp() and plotcp() functions where the only argument to be specified is the classification tree computation that was saved in the previous step.


```r
printcp(class_tree)
plotcp(class_tree)
```


```r
prune_class_tree <- prune(class_tree, cp = .02)
rpart.plot(prune_class_tree, roundint = FALSE, type = 3, branch = .3)
```


### Accuracy


```r
titanic_predict <- titanic %>%
  mutate(tree_predict = predict(prune_class_tree, type = 'class')) %>%
  cbind(predict(prune_class_tree, type = 'prob'))
head(titanic_predict, n = 20)
```


```r
titanic_predict %>%
  count(survived, tree_predict)
```


```r
gf_bar(~ survived, fill = ~tree_predict, data = titanic_predict)
```


```r
gf_bar(~ survived, fill = ~tree_predict, data = titanic_predict, position = "fill") %>%
  gf_labs(y = 'Proportion') %>%
  gf_refine(scale_y_continuous(breaks = seq(0, 1, .1)))
```


```r
titanic_predict %>%
  mutate(same_class = ifelse(survived == tree_predict, 1, 0)) %>%
  df_stats(~ same_class, mean, sum)
```



### Comparison to Baseline


```r
titanic_predict <- titanic_predict %>%
  mutate(tree_predict_full = predict(class_tree, type = 'class'))

titanic_predict %>%
  count(survived, tree_predict_full)
```


```r
gf_bar(~ survived, fill = ~tree_predict_full, data = titanic_predict, position = "fill") %>%
  gf_labs(y = "proportion") %>%
  gf_refine(scale_y_continuous(breaks = seq(0, 1, .1)))
```


```r
titanic_predict %>%
  mutate(same_class = ifelse(survived == tree_predict_full, 1, 0)) %>%
  df_stats(~ same_class, mean, sum)
```

#### Absolute vs Relative Comparison


### Training/Test Data

So far we have used the entire data to make our classification. This is not best practice and we will explore this is a bit more detail. First, take a minute to hypothesize why using the entire data to make our classification prediction may not be the best?

It is common to split the data prior to fitting a classification/prediction model into a training data set in which the model makes a series of predictions on the data, learns which data attributes are the most important, etc. Then, upon successfully identifying a useful model with the training data, test these model predictions on data that the model has not seen before. This is particularly important as the algorithms to make the predictions are very good at understanding and exploiting small differences in the data used to fit the model. Therefore, exploring the extent to which the model does a good job on data the model has not seen is a better test to the utility of the model. We will explore in more detail the impact of not using the training/test data split later, but first, let's refit the classification tree to the titanic data by splitting the data into 70% training and 30% test data. Why 70% training and 30% test? This is a number that is sometimes used as the splitting, an 80/20 split is also common. The main idea behind the making the test data smaller is so that the model has more data to train on initially to understand the attributes from the data. Secondly, the test data does not need to be quite as large, but we would like it to be representative. Here, the data are not too large, about 1000 passengers with available survival data, therefore, withholding more data helps to ensure the test data is representative of the 1000 total passengers.
Splitting the data into training/test

This is done with the rsample package utilizing three functions, initial_split(), training(), and test(). The initial_split() function helps to take the initial random sample and the proportion of data to use for the training data is initially identified. The random sample is done without replacement meaning that the data are randomly selected, but can not show up in the data more than once. Then, after using the initial_split() function, the training() and test() functions are used on the resulting output from initial_split() to obtain the training and test data respectively. It is good practice to use the set.seed() function to save the seed that was used as this is a random process. Without using the set.seed() function, the same split of data would likely not be able to be recreated in the code was ran again.

Let's do the data splitting.
 

```r
titanic <- bind_rows(titanic_train, titanic_test) %>% 
  mutate(survived = ifelse(Survived == 1, 'Survived', 'Died')) %>% 
  drop_na(survived)

set.seed(2019)
titanic_split <- initial_split(titanic, prop = .7)
titanic_train <- training(titanic_split)
titanic_test <- testing(titanic_split)
```
 

```r
class_tree <- rpart(survived ~ Pclass + Sex + Age + Fare + Embarked + SibSp + Parch, 
   method = 'class', data = titanic_train)

rpart.plot(class_tree, roundint = FALSE, type = 3, branch = .3)
```


```r
prune_class_tree <- prune(class_tree, cp = .02)

rpart.plot(prune_class_tree, roundint = FALSE, type = 3, branch = .3)
```

This seems like a reasonable model. Let's check the model accuracy.


```r
titanic_predict <- titanic_train %>%
  mutate(tree_predict = predict(prune_class_tree, type = 'class'))
titanic_predict %>%
  mutate(same_class = ifelse(survived == tree_predict, 1, 0)) %>%
  df_stats(~ same_class, mean, sum)
```

 This is actually slightly better accuracy compared to the model last time, about xxx compared to about xxx prediction accuracy. But, let's test the model out on the test data to see the prediction accuracy for the test data, the real test.



```r
titanic_predict_test <- titanic_test %>%
  mutate(tree_predict = predict(prune_class_tree, newdata = titanic_test, type = 'class'))
titanic_predict_test %>%
  mutate(same_class = ifelse(survived == tree_predict, 1, 0)) %>%
  df_stats(~ same_class, mean, sum)
```

For the test data, prediction accuracy was quite a bit lower, about xxx.

### Introduction to resampling/bootstrap

To explore these ideas in more detail, it will be helpful to use a statistical technique called resampling or the bootstrap. We will use these ideas a lot going forward in this course. In very simple terminology, resampling or the bootstrap can help us understand uncertainty in our estimates and also allow us to be more flexible in the statistics that we run. The main drawback of resampling and bootstrap methods is that they can be computationally heavy, therefore depending on the situation, more time is needed to come to the conclusion desired.

Resampling and bootstrap methods use the sample data we have and perform the sampling procedure again treating the sample we have data for as the population. Generating the new samples is done with replacement (more on this later). This resampling is done many times (100, 500, 1000, etc.) with more in general being better. As an example with the titanic data, let's take the titanic data, assume this is the population of interest, and resample from this population 1000 times (with replacement) and each time we will calculate the proportion that survived the disaster in each sample. Before we write the code for this, a few questions to consider.

1. Would you expect the proportion that survived to be the same in each new sample? Why or why not?
2. Sampling with replacement keeps coming up, what do you think this means?
3. Hypothesize why sampling with replacement would be a good idea?

Let's now try the resampling with the calculation of the proportion that survived. We will then save these 1000 survival proportions and create a visualization.


```r
resample_titanic <- function(...) {
    titanic %>%
        sample_n(nrow(titanic), replace = TRUE) %>%
        df_stats(~ Survived, mean)
}

survival_prop <- map(1:1000, resample_titanic) %>% 
  bind_rows()

gf_density(~ mean_Survived, data = survival_prop)
```

#### Bootstrap variation in prediction accuracy

We can apply these same methods to evaluate the prediction accuracy based on the classification model above. When using the bootstrap, we can get an estimate for how much variation there is in the classification accuracy based on the sample that we have. In addition, we can explore how different the prediction accuracy would be for many samples when using all the data and by splitting the data into training and test sets.
Bootstrap full data.

Let's first explore the full data to see how much variation there is in the prediction accuracy using all of the data. Here we will again use the sample_n() function to sample with replacement, then fit the classification model to each of these samples, then calculate the prediction accuracy. First, I'm going to write a function to do all of these steps one time.


```r
calc_predict_acc <- function(data) {
  rsamp_titanic <- titanic %>%
    sample_n(nrow(titanic), replace = TRUE)

  class_model <- rpart(survived ~ Pclass + Sex + Age + Fare + SibSp + Parch, 
        method = 'class', data = rsamp_titanic, cp = .02)

  titanic_predict <- rsamp_titanic %>%
    mutate(tree_predict = predict(class_model, type = 'class'))
  titanic_predict %>%
    mutate(same_class = ifelse(survived == tree_predict, 1, 0)) %>%
    df_stats(~ same_class, mean, sum)
}

calc_predict_acc()
```


 To do the bootstrap, this process can be replicated many times. In this case, I'm going to do 500. In practice, we would likely want to do a few more.



```r
predict_accuracy_fulldata <- map(1:2000, calc_predict_acc) %>%
  bind_rows()

gf_density(~ mean_same_class, data = predict_accuracy_fulldata)
```


```r
calc_predict_acc_split <- function(data) {
  titanic_split <- initial_split(titanic, prop = .7)
  titanic_train <- training(titanic_split)
  titanic_test <- testing(titanic_split)

  class_model <- rpart(survived ~ Pclass + Sex + Age + Fare + SibSp + Parch, 
        method = 'class', data = titanic_train, cp = .02)

  titanic_predict <- titanic_test %>%
    mutate(tree_predict = predict(class_model, newdata = titanic_test, type = 'class'))
  titanic_predict %>%
    mutate(same_class = ifelse(survived == tree_predict, 1, 0)) %>%
    df_stats(~ same_class, mean, sum)
}

calc_predict_acc_split()
```


```r
predict_accuracy_traintest <- map(1:2000, calc_predict_acc_split) %>%
  bind_rows()

gf_density(~ mean_same_class, data = predict_accuracy_traintest)
```


```r
bind_rows(
  mutate(predict_accuracy_fulldata, type = "Full Data"),
  mutate(predict_accuracy_traintest, type = "Train/Test")
) %>%
  gf_density(~ mean_same_class, color = ~ type, fill = NA, size = 1.25)
```


### Cross-validation

-->