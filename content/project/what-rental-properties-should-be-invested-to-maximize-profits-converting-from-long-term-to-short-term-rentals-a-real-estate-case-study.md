+++
date = "2019-06-05"
lastmod = "2019-06-08"
tags = []
title = "What rental properties should be invested to maximize profits? Converting from long-term to short-term rentals: A Real Estate Case Study"

+++
![](/uploads/Screen Shot 2019-06-05 at 7.55.49 AM.png)

_This blog's intended audience is the real estate company's stakeholders. For more details, you can read the white paper containing the technical details of this post via_ [_here_](https://www.slideshare.net/secret/bR26yHPFK2k7gU)_. The link of the Tableau interactive dashboard is_ [_here_](https://public.tableau.com/views/Whatshort-termrentalpropertiesmakethemostprofitACaseStudy-Ver2/finaldashboard?:embed=y&:display_count=yes&publish=yes&:origin=viz_share_link)_._

  
Given about $500,000, how should Watershed spend its money to maximize their ROI? By conducting modeling and sensitivity analysis, I recommend that Watershed can focus their initial investment on **_15 specific rental properties that generate at least $30,000 yearly profit and are located densely in Miami, Austin, San Diego, Palo Alto, and New York_. The first stage focuses on 2 bedroom houses since they are the most profitable ones. The later stage can be spent on converting 1 bedroom houses and other types of apartments.**

For my sensitivity analysis, I set all variables as unchanged except the profitability cutoff. The reason I do not change other variables being they are already informed numbers from different stakeholders. Therefore, investing Watershed’s initial cash on the variable that can be changed and has the most potential ROI, profitability cutoff, can help us focus on properties that are most profitable given the limited cash given, $500,000.

 ![](/uploads/Screen Shot 2019-06-04 at 3.54.25 PM.png)

In the dashboard, for the parameter control, I increase the profitable cutoff from $6000 to $30000. This has reduced the total cash needed from $1.8 million to $450,000. This half a million dollar meets our initial condition set by the manager, who requires $500,000 as the maximum initial investment Watershed can spend to convert from long-term to short-term rentals.

As a result, only 15 properties are able to meet this profitability threshold. You can see the distribution of each of these properties in this box plot. Most properties have a yearly profit of about $35000 to $65000. Then a few scatter properties have the most profit at $90000 to $138000. At this point, we can ask if there is any pattern in these 15 properties. In other words, do they share some similarities?

![](/uploads/Screen Shot 2019-06-04 at 4.10.33 PM.png)

The first similarity is observed by location. Most of these properties are located in Miami and Austin, while the rest are scattered in Palo Alto, New York, and San Diego. Then I highlight what bedroom and types have the most presence: the two-bedroom house. The second most popular is the one-bedroom house. The analysis shows that apartments are not as profitable as houses.

![](/uploads/Screen Shot 2019-06-04 at 4.16.30 PM.png)

The left image shows the location of 15 profitable properties. The right one highlight the location and number of the 2 bedroom houses.![](/uploads/Screen Shot 2019-06-04 at 2.16.26 PM.png)

The yearly profit of this model is $900,000 for the first year and $1.0 million for later years. This is double the initial investment in which we only have $450,000.

![](/uploads/Screen Shot 2019-06-04 at 4.41.50 PM.png)

The robustness of this model has been tested by testing all variable parameters. This one profitability cutoff is the most reasonable one Watershed can focus on given the limited budget.