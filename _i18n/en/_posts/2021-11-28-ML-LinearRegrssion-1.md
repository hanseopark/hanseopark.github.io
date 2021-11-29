---
layout: post
comments: true
title: The prediction of stock's price using Linear Regression for Machine Learning (1)
permalink: /2021-11-28-ML-LinearRegression-1/
---

## Outline

[1. Introduction](#1.-Introduction)



## 1. Introduction

This is purpose is prediction of stock's price using ML with financial statements.

I have already gotten financial statements up to yahoo's fianance web.This method have alreday been explained by [this link](https://hanseopark.github.io/2021-07-04-financial-statements/). If you want how to get, you should check it. And this full idea has been already explained for [kaggle.](https://www.kaggle.com/hanseopark/prediction-of-price-for-ml-with-finance-stats)

The overall method will be carried out in the following ...

1. [Dats Preprocessing](#data-preprocessing)
2. Correlation for features
3. Modeling
4. Conclusion



### Data Preprocessing

It is imfortant to analyze data for ML. I only use the recent value and numeric datas. Some example are B (bilion) -> 10^9 and etc... [Link](https://hanseopark.github.io/2021-07-10-pre-finance/). And then, I load preprocessed data.

- [Previous Data Preprocessing](#previous-data-preprocessing)
  1. [Load data](#load-financial-statements-of-stock-preprocessed)
  2. [Merge dataframe](#merge-dataframe)
  3. [Check numeric datasets](#check-numeric-datasets)
  4. [Split data and test for correlation](#split-data-and-test-for-correlation)
  5. [Correlation for features and Heatmap](#correlation-for-features-and-heatmap)

- Additional Data Preprocessing





### Previous Data Preprocessing



#### Load Financial statements of stock preprocessed

```python
df_stats = pd.read_json(url+'/data_preprocessing/{0}_stats_element.json'.format(index_name))
df_addstats = pd.read_json(url+'/data_preprocessing/{0}_addstats_element.json'.format(index_name))
df_balsheets = pd.read_json(url+'/data_preprocessing/{0}_balsheets_element.json'.format(index_name))
df_income = pd.read_json(url+'/data_preprocessing/{0}_income_element.json'.format(index_name))
df_flow = pd.read_json(url+'/data_preprocessing/{0}_flow_element.json'.format(index_name))
```



#### Merge dataframe

``` python
df = pd.concat([df_stats, df_addstats, df_balsheets, df_income, df_flow], axis=1)
```



#### Check numeric datasets

```python
from pandas.api.types import is_numeric_dtype
num_cols = [is_numeric_dtype(dtype) for dtype in df.dtypes]
```



#### Split data and test for correlation

```python
from sklearn.model_selection import train_test_split
train_df_corr, test_df_corr = train_test_split(df, test_size=0.2)
```



#### Correlation for features and Heatmap

```python
corrmat = train_df_corr.corr()
top_corr_features = corrmat.index[abs(corrmat['marketCap'])>0]
plt.figure(figsize=(13,10))
plt_corr = sns.heatmap(train_df_corr[top_corr_features].corr(), annot=True)
```

We can think how to consider features for changing the degree of correlation. 

It is a value that does not take into account any degree. 



![Heat map](../images/corrHeatmap_sp500.png)



There are many values that do not matter if viewed simply because there are tickers that contain insufficient information.Therefore, we need additional preprocessing to process in sufficient information.

