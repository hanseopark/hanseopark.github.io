---
layout: post
comments: true
title: The prediction of stock's price using Linear Regression for Machine Learning (1)
permalink: /2021-11-28-ML-LinearRegression-2/
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

- [Previous Data Preprocessing](https://hanseopark.github.io/2021-11-28-ML-LinearRegression-1/)
- [Additional Data Preprocessing](#additional-data-preprocessing)
  1. [Remove index](#remove-index)
  2. [Remove columns](#remove-columns)
  3. [Filling numeric data](#filling-numeric-data)
  4. [Feature Engineering](#feature-engineering)



### Additional Data Preprocessing

We have gotten price and inpormation of stocks, however, there are remained more to correlate somthings which are included to remove index and columns that are leak imformation. So, We need to correlate for porper value by reducing NaN values and adding or changing to numeric data.

#### Remove index

```python
df = df[df['marketCap'].notna()]
df_index_null = pd.DataFrame(columns=['TotalNull', 'PercentOfNull'])
for ticker in df.index:
  temp_df = df.loc[ticker,:]
  count_null = temp_df.isnull().sum()
  percent_count_null = count_null/len(temp_df)
  df_index_null.loc[ticker, 'TotalNull'] = count_null
  df_index_null.loc[ticker, 'PercentOfNull'] = percent_count_null

remove_index = df_index_null[df_index_null['PercentOfNull']>0.5].index.tolist()
```

I hope to get the data over 50% about percent of null only. If you want to get value in wider range, you should just change 0.5 to 0.X.



``` python
unequal_in_index = [x for x in portfolio if x not in df.index.values.tolist()]
for l in unequal_in_index:
  portfolio.remove(l)
for tic in portfolio:
  if tic in remove_index:
    remove_index.remove(tic)
df = df.drop(remove_index, axis=0)
print('After removing index: ', len(df))
```

It should need to correlate same method for portfolio. Because portpolio as for valid train use individually full train wihch is index. 



#### Remove columns

``` python
nulltotal = df.isnull().sum().sort_values(ascending=False)
nullpercent = ( df.isnull().sum() / len(df) ).sort_values(ascending=False)
nullpoint = pd.concat([nulltotal, nullpercent], axis=1, keys=['Total number of null',     'Percent of null'])
remove_cols = nullpercent[nullpercent >= 0.5].keys()
df = df.drop(remove_cols, axis=1)
newtotal = df.isnull().sum().sort_values(ascending=False)
```



In our analysis, we analyze the numeric datas for 84 financial statements. It is also needed to drop statements for leak information.

I hope to get the data over 50% about percent of null only. If you want to get value in wider range, you should just change 0.5 to 0.X.



#### Filling numeric data

``` python
numeric_missed = newtotal.index
for feature in numeric_missed:
  df[feature] = df[feature].fillna(0)
```

The missed data should just add 0 value.



#### Feature Engineering

```python
from scipy.stats import norm, skew
numeric_feats = df.dtypes[df.dtypes != 'object'].index
skewed_feats = df[numeric_feats].apply(lambda x: skew(x)).sort_values(ascending=False)
high_skew = skewed_feats[abs(skewed_feats) > 0.5]
```

We could get features in high correlation.

