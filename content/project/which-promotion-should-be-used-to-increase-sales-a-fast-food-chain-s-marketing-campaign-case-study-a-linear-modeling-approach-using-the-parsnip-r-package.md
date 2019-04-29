+++
date = "2019-04-09"
lastmod = "2019-04-09"
tags = ["project"]
title = "Which promotion should be used to increase sales? A fast-food chain’s marketing campaign case study"

+++
_The original post is published at TowardsDataScience. Check it out_ [_here_](https://towardsdatascience.com/which-promotion-should-be-used-to-increase-sales-c1c91b4ffb34)_._

In introducing a new product to the market, one of the common questions being what promotion has the greatest effect on sales. A fast-food chain tries to test this by introducing the new product at locations in several randomly selected market. A different promotion is used at each location and the weekly sales are recorded for the first four weeks.

I collected the data used from the IBM Watson website via this [link](https://www.ibm.com/communities/analytics/watson-analytics-blog/marketing-campaign-eff-usec_-fastf/). The key question of the following analysis is: **What drives sales in thousands?** To answer this question, we will start by exploring the data and finalize by applying a multiple linear regression to see which promotion or market size have the most ROI.

This post will make use of some modern R packages including parsnip for modeling, _cowplot_ for combining plots, _yardstick_ for evaluating model performance and _recipe_ for feature engineering.

#### The description of the data set

Our data set consists of 548 entries including:

* AgeOfStores: Age of store in years (1–28). The mean age of a store is 8.5 years.
* LocationID: Unique identifier for store location. Each location is identified by a number. The total number of stores is 137.
* Promotion: One of three promotions that were tested (1, 2, 3). We don’t really know the specifics of each promotion.
* Sales in Thousands: Sales amount for a specific LocationID, Promotion and week. The mean amount of sales are 53.5 thousand dollars.
* Market size: there are three types of market size: small, medium and large.
* Week: One of four weeks when the promotions were run (1–4).

All of these variables are beneficial for the modeling process. I will set sales in thousands as our target variable and analyze to see what variables contribute the most to sales.

First, I imported the dataset, then installed or loaded packages if needed.

    marketing <- read_csv("WA_Fn-UseC_-Marketing-Campaign-Eff-UseC_-FastF.csv",              col_types = cols(MarketID = col_skip())) # removed MarketID because it's not needed.
    
    library(simpleSetup)
    
    # Load packages, install if neededpackages <- c('tidyverse',              'readr',              "recipes",              "magrittr",              "rsample",              "viridis",              "caret",              "parsnip",              "yardstick",              "broom")library_install(packages)

#### Exploratory Visualisation

This visualization phase is essential to see patterns in the dataset and make assumptions to test for the modeling part.

![](https://cdn-images-1.medium.com/max/1600/1*aobijGZmmcwIMEw0EWV3YQ.png)

The density plots of numeric values in the dataset

Density plots are similar to histograms because they all show the distribution of variables. The difference here is the y-axis represents the probability per unit on the x-axis. Yet, here, I prefer to use [density plots](https://towardsdatascience.com/histograms-and-density-plots-in-python-f6bda88f5ac0) since they produce a smooth version of histograms, making them look more aesthetics and more readable.

These two plots show the density distribution of the age of stores and sales in thousands. Most stores are quite young, ranging from 1 to 10 years, while sales have the highest distribution at 50 thousand dollars per week.

![](https://cdn-images-1.medium.com/max/1200/1*2tCMUMieaKjgV43DNMASWw.png)

The violin plots of categorical variables in the dataset. Every point is also visualized to make it easier to see the pattern.

The violin plots here show the distributions of _Market Size, Week and Promotions_, which are factor variables in relation to _Sales_.

_Promotion 1_ has a higher mean sales and more outliers signifying higher sales compared to other promotions. _Large market size_ also shows higher sales while the distribution is wider. _Week_ does not show any particular pattern of higher sales, but the fatter distribution shows more people buy food in the first week.

From these visualizations, I can make this following assumption:

**Large market size and promotion 1 are the driving factors in increasing sales.**

The modeling process will help us to confirm this.

Here is the code for the two plots above:

    marketing_trnformed <- marketing %>%  mutate(    Promotion = as.character(Promotion),    Promotion_factor = as_factor(Promotion),    week = as.character(week),    Week_factor = as_factor(week)  ) %>%   select(-c("week", "Promotion", "LocationID"))
    
    # plot violinmarketing_trnformed %>%   select(Promotion_factor, Week_factor, MarketSize, SalesInThousands) %>%   distinct() %>%   group_by(Promotion_factor, SalesInThousands, MarketSize, Week_factor) %>%   summarise(sales = sum(SalesInThousands)) %>%  ungroup() %>%   select(-SalesInThousands) %>%   gather(x, y, Promotion_factor:Week_factor) %>%   ggplot(aes(x = y, y = sales)) +  facet_wrap(~ x, ncol = 1, nrow = 3, scales = "free") +   geom_violin(aes(x = y, fill = sales), alpha = 0.5) +  coord_flip() +  geom_jitter(width = 0.1, alpha = 0.5, color = "#2c3e50") +  ylab("Sales in Thousands") + xlab(NULL) +  theme_bw() <- promotion_plot
    
    #continuous variables: ageinstores and sales
    
    num_vars_plot <- marketing_trnformed %>%   select(AgeOfStore, SalesInThousands) %>%  gather(x, y, AgeOfStore:SalesInThousands) %>%   ggplot(aes(x = y)) +  facet_wrap(~ x, ncol = 2, nrow = 1, scales = "free") +   geom_density(color = "skyblue", fill = "skyblue", alpha = 0.5) +  theme_bw()

### Modeling

This post uses linear regression for our modeling process since the features are simple and only need small tweaks to make them suitable for modeling. Here are some important points to note about linear regression:

* Linear Regression aims to predict an outcome Y from one or multiple inputs X. The assumption behind this algorithm being there is a linear relationship between X and Y.
* The linear model, or lm, tries to produce a best fit linear relationship by minimizing the least squares criterion. The best fit line is found by finding the smallest sum of squared errors.
* A small p-value indicates that it’s unlikely to observe a substantial relationship due to random chance. So if a small p-value is observed, we can be pretty confident that the relationship between X and Y exists.

To illustrate this multiple linear regression model, we define the linear modeling equation as following:

> _y = Intercept + c1 x Promotion 1+ c2 x Promotion 3 + c3 x MarketSize_Medium +c4 x MarketSize_Large_

Where the:

* **Intercept:** Is what all predictions start out. In this case, it is 50 thousand dollars.
* **Model Coefficients (c1 through c4):** Estimates that adjust the features weight in the model.

#### Feature engineering

To make our modeling process more efficient, we need to transform the data into appropriate formats. Here we outline some of the transformation steps:

1. _Promotion_ is now a numeric variable and needed to be dummied into columns for each type of promotion.
2. Market Size is transformed using one-hot coding for each type of market size.

This process also makes the linear regression model interpretable provided that these explanatory variables are categorical. To do so, [one-hot encoding](https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science) is used.

The code below does just this. I first transformed variables _Promotion_ and _week_ into factors and used the package _rsample_ to split the dataset, 70% of the data will be for training and the other 30% is for testing. The package _recipe_ is also used for the feature engineering process described above.

    set.seed(3456)
    
    split <- rsample::initial_split(marketing_trnformed, prop = 0.7)
    
    split %>% training()
    
    split %>% testing()
    
    train <- training(split)
    test  <- testing(split)
    
    recipe <- recipe(SalesInThousands ~ Promotion_factor + MarketSize, data = train) %>%  step_string2factor(MarketSize) %>%  step_dummy(MarketSize, Promotion_factor, one_hot = TRUE) %>%   prep(data = train)
    
    recipe
    
    train_recipe <- bake(recipe, new_data = train)
    test_recipe  <- bake(recipe, new_data = test)

Once the feature engineering process is completed, the modeling process uses the **parsnip** package, which simplifies the modeling process and makes it easy to analyze and visualize.

    linearModel <- linear_reg(mode = "regression") %>%  
    				set_engine("lm") %>%  
                    fit(SalesInThousands ~ Promotion_factor_X1 +                        
                           				   Promotion_factor_X3 +            
    									   Promotion_factor_X2 +                         
                                           MarketSize_Large +                         
                                           MarketSize_Medium +                          
                                           MarketSize_Small, data = train_recipe)

#### Evaluation

To evaluate how the model fit the data, we measure the goodness-of-fit. Using yardstick the important metrics are specified:

    linearModel %>%   
    	predict(new_data = test_recipe) %>%
        bind_cols(test_recipe %>% 
        select(SalesInThousands)) %>%  
        yardstick::metrics(truth = SalesInThousands, estimate = .pred)
    
    ## .metric .estimator .estimate
    ##  rmse    standard   10.5  
    ##  rsq     standard   0.579
    ##  mae     standard   8.69

Where the:

* rsq (Rsquared): this shows how much variance is explained by our inputs. In this case, our inputs can explain 58% of the variability in the dataset.
* rmse (root mean squared error): this quantifies the average squared residuals _(i.e. the difference between actual and predicted values)_. This model shows rmse equal to 10.5, while the range of sales is from 20–100.
* mae (mean absolute error): The issue with rmse is that since it squares the residuals, it will be affected by large residuals. Mae instead averages the absolute value of the residuals. In this case the model shows mae equals to 8.69.

Another way to measure the business impact of the model is visualizing the feature importance of each variable of the model and evaluate how much it contributes to sales:

    linearModel$fit %>%  
    	broom::tidy() %>%  
        arrange(p.value) %>%  
        mutate(term = as_factor(term) %>% 
        		fct_rev()) %>%    
                ggplot(aes(x = estimate, y = term)) +  
                geom_point() +  
                ggrepel::geom_label_repel(aes(label = scales::dollar(estimate, accuracy = 1)), size = 3) +  
                scale_x_continuous(labels = scales::dollar_format()) +  
                labs(title = "Linear Regression: Feature Importance",       
                subtitle = "Looks like large market size, promotion 1 and 2 contribute most to sales") +  
                theme_bw()

![](https://cdn-images-1.medium.com/max/1600/1*3OzMM49ZctIaZbp98oEZAA.png)

The graph below shows the list of important features arranged from the highest p-value (Promotion_factor_X1) to the lowest p-value (MarketSize_Large). Promotion 1 (11 thousand dollars) and large market size (14 thousand dollars) have the most contributions to sales, as expected from our exploratory visualization shown above.

We can then interpret the model by adding the coefficients:

> _y = 50+ 14 x (1) + 11x (1) = 75_

So if we use Promotion 1 in large market size, we can earn 75 thousand dollars/week on average. That’s the highest sales compared to combining other variables.

#### Limitations

The dataset is small with only 508 observations. This makes some variables such as _Promotion_factor_X2 and MarketSize_Small_ not included in our model result. When the number of observations is small, the model decided not to include these variables.

#### Conclusion

To increase sales, the fast food chain should use Promotion 1 and target on large market size.

Linear regression has given us the power of interpretation because of its simplicity. The feature importance plot is able to show the business impact of each variable, which makes it easy and relevant to stakeholders or non-technical people. The lesson here is simple models are preferred over complex models if the dataset is not too complex.