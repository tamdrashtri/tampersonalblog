+++
date = "2019-04-09"
lastmod = "2019-04-09"
tags = ["project"]
title = "Maximizing Business Impact with Churn Prediction Using Machine Learning: A Practical Approach"

+++
#### The telecom industry and customer churn

Nowadays, most telecommunication market is almost saturated with high competitions since different providers sell the same services (i.e wireless, phone plans). Often the fights are between very large companies, each has about 15-30% of the market share.

Because of this saturated market, the cost of acquiring new customers can be 50 times higher than keeping a customer. In terms of wireless subscription, the acquisition cost is still increasing over time. On the customer side, it is very easy to switch to a new provider. In Canada, Bell and Telus reported an average customer acquisition cost of 521$ while the retention cost for each subscriber is about 11$.

Small improvements in reducing churn can generate a significant increase in profits and decreases in costs. A report by McKinsey estimated that reducing churn could increase the earnings of a typical wireless carrier in the US by about 9.9%.

This study aims to predict churn using Automated Machine Learning (Auto ML) and measure the expected value of the model to maximize business value for the telecommunication industry. The entire project uses R as a programming language to conduct data preparation, modeling, evaluation. I also aim to communicate results effectively using data visualization, as it is usually the best way to understand and gain insights.

#### The description of the data set

Our data set consists of 7043 profiles of telecom customers and is available via Kaggle. The data contains two main types of variables:

* Customers’ characteristics such as their gender, whether they have a partner, their tenure status, whether they are senior citizens.
* There usage behaviors such as phone service, internet service, online security, tech support, streaming TV, streaming movies, contract, payment method and their month charges.

In churn prediction cases, studies often found attributes such as customer bills, call duration details, customer demographics, age’s of customer smartphone and the average cost as important predictors of churn. In this dataset, we do not have call duration details and very few details on customer demographics such as where they live or what are their income profiles. Therefore, we cannot test all of these variables and can only focus on some variables relevant to the dataset.

Despite these limited number of attributes, both of these types of variables are beneficial for our predictive analysis. It helps us to detect whether their decisions to churn is based on personal characteristics or their service usage behaviors.

#### The formula for the cost of churn

The other thing needs to be considered is the cost of churn in the telecom industry for each customer. The formula to calculate the cost of churn is formulated as followed:

> Total cost of churn = (Customer Lifetime - The number of months customer stays until they churn) * Gross profit telecom companies earn from each customer * Number of customers

Specifically, customer lifetime in the telecommunication industry is usually 52 months. Customers usually churn in 19 months. Gross margin in the telecom industry is 47%. We can calculate gross profit by multiplying the gross margin with the average monthly charge, which equals 35.1 dollars. Therefore, the total cost of each churner is about 1158.3$.

#### Modeling

The H2O.ai framework is used for the modeling process. H2O is an open-source machine learning framework, which provides us the ability to implement machine learning algorithms without investing in much of the details and technicalities. Particularly, we use H2O AutoML, which aims to automate the machine learning workflow and train different algorithms within a specific time-frame.

Using the programming language R, the entire dataset is split into two different datasets, one is for training and the other is for testing. The split up is 70% for the training set and 30% for the testing set. H2O AutoML is applied to train with a maximum of 7 models. Cross-validation is applied to mitigate the effect of imbalanced data and overfitting problems (e.g. cross-validation trains several models on subsets of the input data and evaluate them on the other subset of the data). The maximum training time is set to 10 minutes.

![The list of models implemented and their performance](https://cdn-images-1.medium.com/max/1600/1*U5P62jS0aa3cZRGtPHbwYA.png)

The leaderboard is ordered by AUC, which is the Area under the Receiver Operating Characteristics Curve. This refers to a metric that is used to measure the performance of a model. Intuitively, it provides an estimate of the probability of a random set of observations of churners is ranked correctly and higher than the selected observations in the class of non-churners. In other words, it is the probability that a churner is assigned a higher probability to churn compared to that of a non-churn. In figure 5, the best performing model has an AUC of 0.85 and has a smaller mean per class error. Please note that on the graph the metric of mean per class error shows 0.23, which is the same. However, the difference is not shown since 0.23 is rounded. In further decimal points, the differences are shown.

Ensemble Stacking is ranked as the best performing model. There are more than one types of the same algorithm (one is StackEnsemble with All Models versus Stack Ensemble Best of Family). Other algorithms being implemented and ranked on the top are default Random Forest (DRF), an Extremely Randomized Forest (XRT), and three pre-specified XGBoost GBM (Gradient Boosting Machine) models. To explain how Stack Ensembles works in detail, it is a type of machine learning methods that combine multiple algorithms. It is a supervised machine learning algorithm (i.e. need data to train) that finds the optimal combination among many prediction algorithms called stacking. Stacking is a way to combine multiple algorithms by applying models by the results of other models. From a learned model, a new model uses the result of this model to train new models. There are different types of models tackling different spaces of the problem. The final model is stacked on top of other models which tackle each part of the modeling process. This is said to improve the overall performance of the model.

### Evaluation

#### Confusion Matrix

Once the modeling process is completed, several metrics are derived to assess the performance of the model. The first metric is the confusion matrix, which is a two by two matrix categorizing the number of correct and incorrect predictions for both positive and negative class. Positives mean customers being classified as churn, and negatives mean customers classified as staying. The true negatives and true positives are the correctly classified observations for the positive and negative classes. Likewise, the false negatives and the false positives represent the errors the model incorrectly classified. Usually, incorrectly predicted observations have different importance, meaning false negatives can cost more than false negatives.

![](https://cdn-images-1.medium.com/max/1600/1*7tNP5SbxJyHK4nXXCly3uw.png)

Confusion Matrix Illustration. The red color in the table shows the most costly rate in predicting churn

For this illustration above, the highlighted box shows the importance of false negatives, the most costly classification error of the model. This is when the model predicts customers stay while customers actually churn. In other words, when the model predicts customers stay, there is no need to give discount incentives for these customers. The company thereby loses profits by making them leave without doing any action for them to stay. Whereas, in terms of false positives (the model predicts customers churn while they stay), the company reduces profits by offering discounts, but still have some revenue from these returning customers. It is important, therefore, to minimize our false negative rate (or minimize costs) while maximizing true positives rate (maximize profits).

![](https://cdn-images-1.medium.com/max/1600/1*3oib1ovhNv7JQC_QtH4BkQ.png)

Confusion Matrix of the Stack Ensemble model.

This table shows the confusion matrix results from the Stack Ensemble model. From the table, we can derive that the number of false negatives is 159, which consists of 10% of the negative rate. Likewise, the number of false positives is 263. The total error rate is 0.20. We see no significant error rates across the confusion matrix. Though the error rate is not low, it is also not very high, so it is acceptable. This confusion matrix will be further used in the later part of the model when we discuss the expected value framework.

#### ROC

The other most commonly used ways to measure model performance is the Receiver Operating Characteristic curve (ROC), which shows the false positive rate (customers that stay the model incorrectly identify as leaving) on the x-axis and the true positive rate (customers the model correctly identify as leaving) on the y-axis. The closer the curve reaching point 1 on the top left of the curve, the better the model is since we maximize correct predictions and minimize incorrect ones.

![](https://cdn-images-1.medium.com/max/1600/1*m6Em86XB2Aei90UlROQoyQ.png)

The ROC curve of the best performing model — H2O Stack Ensemble. The horizontal line shows the false positive rate and the vertical line shows the true positive rate.

Our ROC curve shows a high performance of the H2O Stack Ensemble model with the AUC score equals 0.852, as shown in the leaderboard. From most of the cases, if the AUC score is over 0.80, it is rated as a very good performing model. As shown in the graph, the model performs much better than a random average model, showing it has more true positives rate than false positives rate. The model, therefore, has a good performance overall. From this graph, we can calculate the summary statistics for this ROC curve and calculate the optimal threshold value to maximize profits.

#### Gain and Lift

Gain and Lift are important metrics especially for people who focus on the direct benefits of the model, one of the groups of stakeholders that might benefit from this is business people. The gain chart, specifically, measures what can be gained by using the model. As shown in the graph, the horizontal line represents the cumulative data fraction and the vertical line represents the gain. Based on the yellow gain line in the graph, we can interpret that targeting 37.5% of the high probability customers (cumulative data fraction) can potentially yield 75% of the potential positive response. This is a potential result from the model, showing the model is performing well and it can be used to target customers with high benefits.

![](https://cdn-images-1.medium.com/max/1200/1*CnFK9ox8uBY2N0MOT4Kj2Q.png)

![](https://cdn-images-1.medium.com/max/1200/1*5AyfOHKZrB_1o1A2Xcmt6w.png)

Gain and Lift charts of Stack Ensembles

Lift Chart goes hand in hand with Gain Chart, showing the result of the modeling approach versus targeting people at random. Basically, lift is used to measure the prediction of random guess vs using a model. Such improvement of prediction from random guess is called Lift. It is shown that there is a direct connection between profitability and lift. Thus, life has been used in many churn prediction studies as a performance criterion.

For this lift chart, it is shown that if we targeted 25% of people with high probability of churning people (cumulative data fraction), the model performed 2.5 times better targeting ability compared to targeting randomly (lift). This can potentially reduce cost by about 2.5 times versus random selection because we only need to offer discount incentives to customers with high probability to churn.

In various churn prediction studies, the average top decile lift is about 2.1 to 1, meaning customers in the top decile lift were 2.1 times likely to churn than average. Compared to our model’s lift graph, the top decile lift is about 3.0 to 1. This shows the performance of Stack Ensemble is more accurate than other models on average.

#### Expected Value Framework

In this section, we will use the expected value framework to calculate the potential profits if implemented the model. The expected value framework is introduced in the book Data Science for Business (2013), aiming at combining the model result with the expected value of the model based on the confusion matrix.

To do this calculation, it is necessary to have the expected rates and the cost/benefit of each rate, for the formula is as follows:

![](https://cdn-images-1.medium.com/max/1600/1*ZNHQFD_1AeRn6Gr5YHLqZA.png)

* p(p) is the probability of actual yes in the confusion matrix
* p(n) is the probability of actual no in the confusion matrix
* p(Y, p) is the True Positive Rate (tpr)
* p(N, p) is the False Negative Rate (fnr)
* p(N, n) is the True Negative Rate (tnr)
* p(Y, n) is the False Positive Rate (fpr)
* b(Y, p) is the benefit of true positive
* c(N, p) is the cost of false negative
* b(N, n) is the benefit of true negative
* c(Y, n) is the cost of false positive

In terms of cost/benefit matrix for each expected rates, the table below shows the revenue and cost of each rate:

![](https://cdn-images-1.medium.com/max/1600/1*AoDwflWRxRcILsWSVCZg6Q.png)

The cost and revenue break down of each classification rate based on the confusion matrix.

The break down of each rate is explained as follows:

* True Positives (Revenue): this is when we give discount to customers we predicted as churn and in response, they are predicted as churn and in response, they no longer churn. We earn 33 months of revenue minus the 15% discount. The total profit is equal 1718.72$
* True Negatives: this is the rate when we predict customers do not churn, so we do not give discounts to them. We earn profits as usual.
* False Positives: this is when we predict they churn and they actually stay. We did give discount to them so we lose profits of the discount. The amount lost is 368.28$.
* False Negatives: this is the rate where we lose 33 months of revenue when we predicted customers stay while they actually churn. 33 months of revenue would be lost since we did not give discount to them, while they can actually change their decision to stay. The total cost of this rate is 2087$.

At which point should we base our calculation on the expected value? We need to find the point in which the best combination of the confusion matrix is chosen. In this case, F1 score does just this:

![](https://cdn-images-1.medium.com/max/1600/1*37NAFKSSBBZwktF4piFZzw.png)

The number of each calculation needed for the expected value formula based on the maximum F1 value.

The F1 score is selected at its maximum score as the baseline and the confusion matrix rates are followed by this. The reason we choose F1 score is that it is used to balance the precision and recall rate. Precision is the total positive predicted observations, while recall is the correctly predicted positive observations to all observations in actual class. They are both useful metrics, but usually, when one increases precision, recall will decrease so there is a tradeoff. When F1 score is used, it is to make sure that the best combination of confusion matrix rates is chosen, thus minimizing the classification errors.

Once we have all the numbers for the calculation, using the formula of the expected profit in figure 14, we calculate the profit for each customer and the calculation is shown below:

> (0.72 * ((0.716 * 1719) + (0.284 * -2087))) + (0.17 * (-368 * 0.17)) = 452$

This shows 452$ as the potential profit per person. Multiply this by the total number of customers in the dataset, we have:

> 452 * 7032 = 3.178.464$

The potential profit by implementing the model alone can generate 3.18 million dollars of the total 7032 persons.

#### Business Recommendations

The model shows that short contract terms and monthly charges are significant factors affecting churn. Most importantly, these factors are what is controllable and can be improved by offering incentives to customers. From this example of expected value framework, we introduced one option, which is to give discounts to customers who are predicted to churn based on the confusion matrix. This results in a profit of 3.18 million dollars out of 7032 people in the dataset. Note that each company can have over 1 million people or much more than that, so the profit to mitigate churn when added up can be huge.

Therefore, telecom companies could solve this potential issue of churn by offering special discounts to products or services that are least likely to churn (i.e. phone or data plans that require customers to subscribe with a long term contract with discounts compared to a monthly plan or high monthly charges. The higher the charges, the more customers intend to leave and change to more affordable options. This is time for companies to consider when is best to offer discounts to maximize profits. Personalized recommendations based on customers’ profiles are one of the best ways for companies to make sure their churn rate can be mitigated.

Therefore, it is necessary for firms to establish a data infrastructure that has information about customers and their detailed profiles (i.e. professions, age, country of residence, monthly incomes, etc). This can give firms a competitive advantage to develop good predictive models and provide the most suited offer for every customer. The more firms can create a segmentation of customers based on their profiles, the higher the chance of offering personalized products that can mitigate customers’ propensity to churn since they would satisfy with firms’ offers.

#### Note:

* This is only a summary of my bachelor thesis. You can read the full text and references [here](https://www.slideshare.net/TamNguyen689/tam-nguyenchurnfinalmarch21). For the code, check out my Github page of the thesis [here](https://github.com/tamdrashtri/data-analysis-final-project).