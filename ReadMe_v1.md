
# General Description of the Repository

**This repository contains the project for Module 17 of the AI course.**

Dataset comes from the UCI Machine Learning repository  [link](https://archive.ics.uci.edu/ml/datasets/bank+marketing). The data is from a Portugese banking institution and is a collection of the results of multiple marketing campaigns. We will make use of the article accompanying the dataset [here](docs/CRISP-DM-BANK.pdf) for more information on the data and features.

The data are related to 17 campaigns that occurred between May 2008 and November 2010, corresponding to a total of 79354 contacts

### Repository Structure:
- A **data** folder with the aforementioned file.
- An **images** folder containing generated image files.

The notebook can be accessed via the following link: [View the notebook](progetto_17_1_v2.ipynb).

The project is also available on GitHub: **https://github.com/GPSMAS/BankInvestmentCampaign.git**.

### Project Overview:
The project involves analyzing the dataset to identify the best model for classifying the data on target feature incating yes/no for innvestment sign, allowing for the optimization of signed investments for the bank.
The Dataset contain 20 features and one target.
more information can be read [here](data/bank-additional-names.txt)

### Business Understanding

**The goal** is to increase investments, or in other words, improve the success rate of the marketing campaign. This requires identifying the features that have the greatest impact on the target audience, allowing us to reduce the number of contacts needed to achieve higher investment levels.

**Our approach can be described** as maximizing true positives (TP) on "yes" responses. However, since the cost of making a contact is relatively low compared to the potential return on investment, the best strategy might be to prioritize contacts predicted as true positives, even if it means sacrificing some precision. In this context, maximizing recall is important, even with a slightly lower precision rate, because making an extra phone call isn't a major issue if it helps capture a larger share of the investment opportunities.

### Data Understanding

1) **null/NaN value**: the result is no null or NaN value
2) **analyze target variable with a frequency plot** [Target Frequency Plot](images/FreqTarget.jpg):from the image we can see that the dataset is really unbalanced

3) **analyze data by visualization with frequency plot of features** 
- For numerical features [Features Numerical Frequency Plot](images/FreqFeaturesNumerical.jpg);
- For categorical features [Features Categorical Frequency Plot](images/FreqFeaturesCategorical.jpg);

3.1) a lot of record classified as "not existent information" for features previous_contact; poutcome.
3.2) the customers are mostly new customers; a small percentage is from past campaigns.
3.3) the age is distributed across a wide range, but most are under 60, but the signed investment percentage upper 60 is higher.
3.4) Classification as "unknown" value present in the features job, marital, education, default, housing, loan.
3.4.1) features with "unknown" value and its impact on target = y - found that it impacts less than 30% of the dataset and does not impact "yes" for the target, so we consider it can be deleted.
Again, the highest number of non-existent values is for default, but default have just "no" and "unknown" value meaning that if the customer have category default as "yes" is not selected for this compaign, it can be a constraint before of analisys, but we have not data for this, so for our analisys default add just complexity but not value added, we assume that can be dropped.
3.5) from features description in project overview we find out that feature named "duration" is good to use just for benchmarking but not to predict, so maybe it can be disregarded.


### Data Preparation
To reduce the complexity for the first model we select just bank features that in our df are prefixed with client_
Drop record with "unknown" values.
Drop default features.
Apply: StandardScaler for numeric features and OneHotEncoder (dropfirst) for categorical.

### Baseline Modeling 

To start we go to analize if we can define a baseline using frequency or apply a simple Logistic Regression with standard parameter. 
We found that accuracy for "yes" is just 13% due the fact that dataset is really umbalanced, also we have zero for all other score, also for recall that is scoring of our interest, as already said in Business understanding section.

We try to apply a **simple model**  to define a baseline on our needs. To do this we apply Logistic Regression with class_weight='balanced'. 

We find that now the scoring are acceptable:
- recall and other measure are scored;
- no big difference between test and traing prediction;
- good result for recall scoring, and also for accuracy.

The results can consulted in [this plot](images/ScoreEvaLogRegBalanced.jpg). 

### Model Comparison

In this section we try to apply more models and create a comparison between them in terms of performance, focusing on the Logistic Regression model, KNN, Decision Tree, and SVM - all models setted with default parameters, except for class balancing.

The results can be consulted [here](images/ModelComparison.jpg)

**From the image we can find** that:
- Decision Tree suffer about overfitting, we see a big difference between train and test scoring.
- Logistic Regression and SVM have the best result for recall 0.5 and 0.49 --- with low precision (this answer to our objectives to opimize recall).
- KNN remains less effective due to its very low recall (0.08 on the test set), while Decision Tree suffers from overfitting

**What actions we plan to do** :

Logistic Regression should be prioritized, as it has the highest recall (0.50) and avoids overfitting. It's a simple model that is effective in capturing positive cases even with imbalanced data.
SVM is still a competitive option, with only a slightly lower recall (0.49), and may be worth tuning further. we can experiment with adjusting the C parameter to generalize more  balance recall and precision.

**Traing time analysis**:
In the plot [here](images/SimpleModelsTimeTrain.jpg) we can see how is longer SVM to train the same dataset.

### Improving the Model

To improve the model we try to set more detailed analysis on all features as follow:
1) correlation analysis for numerical features whit [heatmap](images/corr_NumericalFeat.jpg), on which we select only strong relation with values > 0.65.
2) Cramér's V for chategorical features with [heatmap](images/CramerCategoricalFeat.jpg), on which we select only strong relation with values > 0.65.
3) **Actions to do**:
3.1) for numerical features: we mantain only: context_emp_var_rate; context_cons_conf_idx. *We drop*: context_cons_price_idx; context_euribor3m; context_nr_employed.
3.2) for categorical features: finding strong relation between housing and loan, so we *drop housing* and mantain more generic information with loan.
3.3) As already said *duration can be dropped* for our scope.
3.4) We *drop records with "unknowns"* information;
3.5) We *drop default* column.

4) **Finding optimal model application**: From the results of section Model Comparison we continue to evaluate as model: Logistic Regression and SVM, looking for optimal combination of hyperparametes with grid search cv:
4.1) Logistic Regression: try to set classifier__C': [0.1, 1, 10, 100]
4.2) SVM, to generalize more we try for classifier__C': [1,3,5]

5) **Results**
- we find Logistic Regression with C = 0.1 as optimal solution, the acceptable value can be consulted [here](images/OptimalModel.jpg).
- we can read that no overfitting are showed and good result for recall at 62%.


### Deployment

To use this results we want to examinate the first 10 coeffincients so that as the more important features to take in mind:

**Contact Months** : The months of contact (particularly March, October, September, and July) seem to be key factors in the decision to sign for the investment. This could be linked to seasonal variables or specific marketing or promotional initiatives carried out during those months.

**Outcome of Previous Campaigns**: If clients have not participated in previous campaigns or had a successful outcome, they are more likely to sign. New clients or those with positive experiences are probably less skeptical about new investments.

**Occupation**: Occupations such as "students" and "retirees" appear more likely to sign for an investment. This may reflect a greater openness among these categories, possibly looking to build their capital or manage it more securely.

**Education Level**: People with a university degree have a higher probability of investing, as one might expect, since they may be more familiar with investment opportunities. However, the positive probability for "illiterate" individuals could suggest that other factors are at play (such as the support of financial advisors).
