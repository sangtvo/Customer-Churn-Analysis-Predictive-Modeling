# Project 02: Customer Churn Analysis
> This study analyzes the customer data to identify why customers are leaving and the potential indicators that are causing them to leave. The data is derived from a telecommunications company through Kaggle and will be using three different predictive modeling: (1) binary logistic regression (2) decision trees and (3) random forest.

Table of Contents
---
1. [General Information](#general-information)
2. [Summary](#summary)
3. [Tech Stack](#tech-stack)
4. [Data Preprocessing/Cleaning](#data-preprocessingcleaning)
5. [Data Visualization](#data-visualization)
6. [Data Analysis](#data-analysis)
7. [Modeling](#modeling)
8. [Solution](#solution)
9. [Key Takeaways](#key-takeaways)
10. [References](#references)

<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#general-information"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#summary"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#tech-stack"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#data-preprocessingcleaning"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#data-visualization"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#data-analysis"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#modeling"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#solution"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#key-takeaways"/>
<a name="https://github.com/sangtvo/Seattle-PD-Funding-Eligibility#references"/>

General Information
---
The project is part of a graduate course (_Data Mining and Analytics II_) at Western Governor's University. The raw data set contains 7,043 observations and 21 features. To expand the project even further (originally binary logistic regression), decision trees and random forest analysis are added.


Summary
---
The best fitted linear regression model is y = 1.491x + 21.914 which means that for every additional incident reported (x), you can expect officers on site to increase by an average of 1.491. The overall mean of the data is 1.889 which is below the 2.5 threshold and is not qualified for additional funding. In order to increase their eligibility for funding, the Seattle PD must focus on zones with average officers at scene that is less than or equal to 2 and the largest reported incident zones. There are 12 zones that need assistance and can lead to funding eligibility.

Tech Stack
---
* R Studio
    * plyr
    * tidyverse
    * ggplot2
    * ggcorrplot
    * randomForest
    * caret
    * cowplot
    * RColorBrewer
    * pROC
    * rpart
    * rpart.plot

Data Preprocessing/Cleaning
---
#### Irrelevant:
Removed customerID variable since it is not necessary for the purpose of this analysis.
```r
cdf$customerID <- NULL
```

#### Recoding: 
Recode some of the categorical variables for simplicity.
```r
cdf$SeniorCitizen <- as.factor(mapvalues(cdf$SeniorCitizen, from=c("0","1"), to=c("No", "Yes")))
cdf$MultipleLines <- as.factor(mapvalues(cdf$MultipleLines, from=c("No phone service"), to=c("No")))

for (i in 9:14){
  cdf[,i] <- as.factor(mapvalues(cdf[,i], from=c("No internet service"), to=c("No")))
}
```

Recode the dependent variable as a factor in the clean data frame instead of characters.
```r
cdf[, 'Churn'] <- as.factor(cdf[, 'Churn'])
```

#### Missing Data:
Checking for missing data.
```r
sapply(df, function(x) sum(is.na(x)))
```
```
      customerID           gender    SeniorCitizen          Partner       Dependents           tenure     PhoneService    MultipleLines  InternetService 
               0                0                0                0                0                0                0                0                0 
  OnlineSecurity     OnlineBackup DeviceProtection      TechSupport      StreamingTV  StreamingMovies         Contract PaperlessBilling    PaymentMethod 
               0                0                0                0                0                0                0                0                0 
  MonthlyCharges     TotalCharges            Churn 
               0               11                0 
```

Calculating the percentage of missing values on TotalCharges variable.
```r
sum(is.na(df$TotalCharges))/nrow(df)
```
```
[1] 0.001561834
```

Since the data has 0.16% missing data in the TotalCharges variable, a new data frame is created to remove the missing values.
```r
cdf <- na.omit(df)
```

### The final clean data frame: 
```r
summary(cdf)
```
```
    gender          SeniorCitizen   Partner           Dependents            tenure      PhoneService       MultipleLines InternetService   
 Length:7032        No :5890      Length:7032        Length:7032        Min.   : 1.00   Length:7032        No :4065      Length:7032       
 Class :character   Yes:1142      Class :character   Class :character   1st Qu.: 9.00   Class :character   Yes:2967      Class :character  
 Mode  :character                 Mode  :character   Mode  :character   Median :29.00   Mode  :character                 Mode  :character  
                                                                        Mean   :32.42                                                      
                                                                        3rd Qu.:55.00                                                      
                                                                        Max.   :72.00                                                      
 OnlineSecurity OnlineBackup DeviceProtection TechSupport StreamingTV StreamingMovies   Contract         PaperlessBilling   PaymentMethod     
 No :5017       No :4607     No :4614         No :4992    No :4329    No :4301        Length:7032        Length:7032        Length:7032       
 Yes:2015       Yes:2425     Yes:2418         Yes:2040    Yes:2703    Yes:2731        Class :character   Class :character   Class :character  
                                                                                      Mode  :character   Mode  :character   Mode  :character  
                                                                                                                                              
                                                                                                                                              
                                                                                                                                              
 MonthlyCharges    TotalCharges    Churn     
 Min.   : 18.25   Min.   :  18.8   No :5163  
 1st Qu.: 35.59   1st Qu.: 401.4   Yes:1869  
 Median : 70.35   Median :1397.5             
 Mean   : 64.80   Mean   :2283.3             
 3rd Qu.: 89.86   3rd Qu.:3794.7             
 Max.   :118.75   Max.   :8684.8             
```

For the full notebook, please check out Customer Churn Analysis.Rmd

Exploratory Data Analysis
---

<table>
  <tr><th colspan=2>Univariate Analysis</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_1.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_2.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_3.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bar_4.png?raw=true"> </td></tr>
</table>

* ***Gender*** and ***Partner*** are almost evenly split 50/50 and that 83.76% of the data are not ***SeniorCitizens***. 
* 90.33% of the data have single ***PhoneService*** lines, but in terms of extra services, there are roughly 70% of the customers that do not have ***OnlineSecurity*** and ***TechSupport***. 
* While ***InternetService*** is a pretty common add-on with phone lines, **fiber optic** is a favorable internet service which accounts for 44.03% of the customers. 
* Most customers are on a **month-to-month** ***Contract*** (55.11%) and 33.63% of the customers pay their bill with an **electronic check**.
* The ***Churn*** rate of the data is 26.58%.


<table>
  <tr><th colspan=2>Bivariate Analysis</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_1.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_2.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_3.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_4.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bivar_5.png?raw=true"> </td>
</table>

* Gender percentages are almost similar and therefore, may not have influence on churn. 
have hnand customers with phone service are more likely to churn than those who don’t. 
* Online security and tech support have very similar percentages and perhaps they are somewhat correlated whether a customer is churning or not.
* Online backup and device protection are similar indicating that they might be correlated with one another.
* Month-to-month contract customers is higher than longer contracts, which makes sense because there is no obligation to stay longer if it was a month-to-month basis and can leave at any time. 
* Customers with internet service are more likely to churn than those that don't have internet.
  * Fiber optic churn rate is 3x higher than DSL and 9x without internet service.
* Churn percentage is higher for customers who utilizes paperless billing option.
* Customers who use electronic checks are almost 5x as high to churn compared to other payment method options.


<table>
  <tr><th colspan=2>Distribution of Continuous Independent Variables</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/distribution_MonthlyCharges.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/distribution_TotalCharges.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/distribution_tenure.png?raw=true"> </td><td>
</table>

![Correlation](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/correlation.png?raw=true)

<table>
  <tr><th colspan=2>Boxplots of Continuous Independent Variables</th></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bp_MonthlyCharges.png?raw=true"> </td><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bp_TotalCharges.png?raw=true"> </td></tr>
  <tr><td> <img src="https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/bp_tenure.png?raw=true"> </td><td>
</table>

Logistic Regression (Base) ROC Curve             |  Step-wise Logistic Regression ROC Curve
:-------------------------:|:-------------------------:
![BaseLR](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/base_lr_roc.png?raw=true)  |  ![StepLR](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/step_roc.png?raw=true)


![Dtree](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/dtree.png?raw=true)

![DtreeROC](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/dtree_roc.png?raw=true)

![Top10](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/Top10_Variables.png?raw=true)

![rfROC](https://github.com/sangtvo/Customer-Churn-Analysis/blob/main/images/rf_roc.png?raw=true)



Modeling
---
A linear regression model is created by using Microsoft Excel Data Analysis Toolpak. The model provides a population regression parameter where the slope is β<sub>1</sub> = 1.491 and a y-intercept of β<sub>0</sub> = 21.914. The linear regression model displays an upward trend and indicates that the number of incidents in a specific district are closely related to the numbers of officers responding to the scene. However, there are two outliers that is skewing the regression line, (1,1) and (125,165) which may show that it is not necessarily the best fit.

By calculating R<sup>2</sup> for the model with and without outliers, we can see how much variability of the response data around its mean. Ideally, the higher the R<sup>2</sup>, closer to 1, the better the model fits the data. However, there are some limitations with R<sup>2</sup> and sometimes does not indicate whether a regression model is adequate. Therefore, we must also consider sum of absolute error (SAE) calculation as it shows how far the regression line is from the actual data points. The lower the SAE, the better the fit.

Linear Regression             |  Residual Plot
:-------------------------:|:-------------------------:
![LR](https://github.com/sangtvo/Seattle-PD-Funding-Eligibility/blob/main/images/LR_outlier.gif?raw=true)  |  ![Residual](https://github.com/sangtvo/Seattle-PD-Funding-Eligibility/blob/main/images/Residual_outlier.gif?raw=true)

![ANOVA](https://github.com/sangtvo/Seattle-PD-Funding-Eligibility/blob/main/images/ANOVA.GIF?raw=true)

x             |  E(y) |  y |  E(y) - y = e
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
83	| 145.667	| 158	| -12.333
125	| 208.289| 	165	| 43.289
37	| 77.081| 	86| 	-8.919
64| 	117.338| 	131| 	-13.662
60	| 111.374| 	121| 	-9.626
31| 	68.135| 	72	| -3.865
52	| 99.446| 	96	| 3.446
60| 	111.374	| 124| 	-12.626
44| 	87.518| 	82| 	5.518
1	| 23.405	| 1	| 22.405
41	| 83.045| 	77| 	6.045
62	| 114.356| 	120| 	-5.644
38	| 78.572	| 72| 	6.572
44	| 87.518	| 76| 	11.518
91	| 157.595	| 176| 	-18.405
53	| 100.937| 	117| 	-16.063
35	| 74.099| 	68| 	6.099
39	| 80.063| 	76| 	4.063
86	| 150.14	| 158	| -7.86
| | | SAE= | 0.048

The linear regression model with outliers demonstrate a R<sup>2</sup> = 0.8795 and SAE = 0.048 while the linear regression model without outliers have a R<sup>2</sup> = 0.9591 and SAE = 68.007 (not shown). Even though the linear regression model without outliers has a higher R<sup>2</sup>, the SAE is not minimized.

The residual plot above shows that the two outliers seem to be pulling data points away from the trend line (y=0). However, when removing the outliers in the residual plot (not shown) becomes much more randomly dispersed and the “mean of zero” assumption almost holds true, meaning the mean is approximately zero all the way across the plot. Despite this and the limited observations, the best fit for linear regression model is with outliers. 

When looking at the ANOVA table, the model demonstrates that the F value is less than 0.05 and therefore statistically significant.

<table>
  <tr><th colspan=2>Descriptive Statistics</th></tr>
  <tr><td>Mean</td><td>1.889952153</td></tr>
  <tr><td>Standard Error</td><td>0.036811154</td></tr>
  <tr><td>Median</td><td>2</td></tr>
  <tr><td>Mode</td><td>1</td></tr>
  <tr><td>Standard Deviation</td><td>1.189974282</td></tr>
  <tr><td>Sample Variance</td><td>1.416038791</td></tr>
  <tr><td>Kurtosis</td><td>-0.93853082</td></tr>
  <tr><td>Skewness</td><td>0.151756917</td></tr>
  <tr><td>Range</td><td>4</td></tr>
  <tr><td>Minimum</td><td>0</td></tr>
  <tr><td>Maximum</td><td>4</td></tr>
  <tr><td>Sum</td><td>1975</td></tr>
  <tr><td>Count</td><td>1,045</td></tr>
</table>

The current threshold to receive additional funding for the Seattle Police Department is 2.5, however, they do not meet the expected threshold. When calculating the mean of the clean data, the mean is 1.889 which is below the 2.5 threshold (1.904 on graph due to one zone not removed yet). This means that the department averages 1.889 officers at the scene per incident and is ineligible for additional funding. However, there is a limitation of this study. Since the study is only for a span of 3 days, it is best to collect data for the whole month or quarter as a better estimate. 

Solution
---
Since the Seattle PD is not eligible for funding due to a mean of 1.889, the department should focus on zones with less average officer per site and zones with the largest reported incidents. By targeting zones with average officers that have a mean of 2 or less and reach a mean score of 2+, it is possible it will bring the overall mean to 2.5. If it is not reached still, then targeting the highest reported incidents zone can also bring it up by mandating 3 officers per incident. If the police department is understaffed, that would mean that officers are working over-time and arriving at multiple scenes to keep up with the incoming calls. This inefficiency will continue to drive down (or similar amount) the mean of officers at scene and the department will never reach that 2.5 threshold to receive any funding in future years. If the police department hired a human resources analyst, this person can detect how many officers are needed and analyze their work schedules to be more effective in order to meet demand.

Key Takeaways
---
* The number of incidents on Sunday (March 27, 2016) is 2x as high compared to Friday and Saturday that same weekend due to Easter Sunday.
* The top 3 incidents are disturbances, traffic related calls, and suspicious circumstances which are 3x more common than other incidents.
* At least 2 officers arrive onsite and show up more often in the outskirts of downtown Seattle. 
* The W zone has the lowest incidents reported due to Burley and Bethey districts having less population and more deserted, but the highest mean of officers onsite (2.324).
* The H zone which is downtown Seattle has the highest reported incidents, but the lowest mean of officers onsite of 1.32. 

References
---
Balk, G. (2019, April 4). Is Seattle 'dying'? Crime rates tell a different story. Retrieved June 12, 2020, from http://www.seattletimes.com/seattle-news/data/is-seattle-dying-not-if-you-look-crime-rates-from-the-80s-and-90s/
