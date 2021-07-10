---
layout: post
comments: true
title: Preprocessing with financial statements
permalink: /2021-07-10-pre-finance/
---

### Preprocessing with financial statements

I have already gotten the financial statement of [post](https://hanseopark.github.io/2021-07-04-financial-statements/).

It is enough from person to person. However I will analyze for meachine learning for regression.

I hope to get and change numeric data in statements.

If you want to understand what it is mean in financialk statements, you should know and search the defintion for google. It is not major of finance for me and already explained more easily than me for world.

I have used config file for root_url which is setted by myself and get index list using API of [yahoo_fin](http://theautomatic.net/yahoo_fin-documentation/#get_balance_sheet).

And, I utilized [class](https://github.com/hanseopark/Stock/blob/master/Strategy/class_Strategy.py) made by me. It is useful to preprocess and multiple strategy that it is updated for performance improvement curruntly.

- **yahoo_fin API**

> pip install yahoo_fin



You have to make directory(<root_url>/data_preprocessing) before run below code.

### Code to get preprocessing stock's financial statements

> ```python
> import pandas as pd
> import pandas_datareader as pdr
> import yahoo_fin.stock_info as yfs
> 
> import json
> 
> def main(url='', index_list = ['AAPL'], index_name = 'dow', EACH = True, ALL = True, SHOW = True, SAVE = True):
> 
>     ## Configuration directory url
>     url_data = url+'data_origin/'
>     url_pre  = url+'data_preprocessing/'
> 
>     strategy = LongTermStrategy(url, filename) # Select Long term strategy
> 
>     ###################
>     ## Preprocessing ##
>     ###################
> 
>     if EACH == True:
>         print('Preprocessing each Financial Statements')
>         # Basic stats
>         dict_basic = {}
>         basic_fac = ['PER', 'PSR', 'PBR', 'PEG', 'forPER', 'Cap']
>         print('\nEach factor preprocessing in Basic stats ('+ ' '.join(s for s in basic_fac), end=')\n')
>         for fac in basic_fac:
>             if fac == 'PER':
>                 df = strategy.get_PER()
>             elif fac == 'PSR':
>                 df = strategy.get_PSR()
>             elif fac == 'PBR':
>                 df = strategy.get_PBR()
>             elif fac == 'PEG':
>                 df = strategy.get_PEG()
>             elif fac == 'forPER':
>                 df = strategy.get_FORPER()
>             elif fac == 'Cap':
>                 df = strategy.get_CAP()
> 
>             if SAVE == True:
>                 print('\n----------------------------- SAVE DATA  {} ---------------------------------\n'.format(fac))
>                 df.to_json(url_pre+'/{0}_{1}.json'.format(index_name, fac))
>                 df.to_csv(url_pre+'/{0}_{1}.csv'.format(index_name, fac))
> 
>             if SHOW == True:
>                 print(df)
>             dict_basic[fac] = df
> 
>         # Additional stats
>         dict_add = {}
>         basic_fac = ['Beta', 'DivRate', 'ROE', 'ROA', 'PM', 'Cash', 'Debt']
>         print('\nEach factor preprocessing in Additional stats ('+ ' '.join(s for s in basic_fac), end=')\n')
>         for fac in basic_fac:
>             if fac == 'Beta':
>                 df = strategy.get_Beta()
>             elif fac == 'DivRate':
>                 df = strategy.get_DivRate()
>             elif fac == 'ROE':
>                 df = strategy.get_ROE()
>             elif fac == 'ROA':
>                 df = strategy.get_ROA()
>             elif fac == 'PM':
>                 df = strategy.get_PM()
>             elif fac == 'Cash':
>                 df = strategy.get_Cash()
>             elif fac == 'Debt':
>                 df = strategy.get_Debt()
> 
>             if SAVE == True:
>                 print('\n----------------------------- SAVE DATA  {} ---------------------------------\n'.format(fac))
>                 df.to_json(url_pre+'/{0}_{1}.json'.format(index_name, fac))
>                 df.to_csv(url_pre+'/{0}_{1}.csv'.format(index_name, fac))
> 
>             if SHOW == True:
>                 print(df)
>             dict_add[fac] = df
> 
>         # Balance sheets
>         dict_bal = {}
>         basic_fac = ['TA']
>         print('\nEach factor preprocessing in Balance sheets ('+ ' '.join(s for s in basic_fac), end=')\n')
>         for fac in basic_fac:
>             if fac == 'TA':
>                 df = strategy.get_TA()
> 
>             if SAVE == True:
>                 print('\n----------------------------- SAVE DATA  {} ---------------------------------\n'.format(fac))
>                 df.to_json(url_pre+'/{0}_{1}.json'.format(index_name, fac))
>                 df.to_csv(url_pre+'/{0}_{1}.csv'.format(index_name, fac))
> 
>             if SHOW == True:
>                 print(df)
>             dict_bal[fac] = df
> 
>         # Income sheets
>         dict_income = {}
>         basic_fac = ['TR']
>         print('\nEach factor preprocessing in Income sheets ('+ ' '.join(s for s in basic_fac), end=')\n')
>         for fac in basic_fac:
>             if fac == 'TR':
>                 df = strategy.get_TR()
> 
>             if SAVE == True:
>                 print('\n----------------------------- SAVE DATA  {} ---------------------------------\n'.format(fac))
>                 df.to_json(url_pre+'/{0}_{1}.json'.format(index_name, fac))
>                 df.to_csv(url_pre+'/{0}_{1}.csv'.format(index_name, fac))
> 
>             if SHOW == True:
>                 print(df)
>             dict_income[fac] = df
> 
>         # Cash flow
>         dict_flow = {}
>         basic_fac = ['DIV', 'ISS']
>         print('\nEach factor preprocessing in Cash flow ('+ ' '.join(s for s in basic_fac), end=')\n')
>         for fac in basic_fac:
>             if fac == 'DIV':
>                 df = strategy.get_DIV()
>             elif fac == 'ISS':
>                 df = strategy.get_ISS()
> 
>             if SAVE == True:
>                 print('\n----------------------------- SAVE DATA  {} ---------------------------------\n'.format(fac))
>                 df.to_json(url_pre+'/{0}_{1}.json'.format(index_name, fac))
>                 df.to_csv(url_pre+'/{0}_{1}.csv'.format(index_name, fac))
> 
>             if SHOW == True:
>                 print(df)
>             dict_flow[fac] = df
> 
>     if ALL == True:
>         stats_name = ['stats', 'addstats', 'balsheets', 'income', 'flow']
>         print('\nPreprocessing element in ('+ ' '.join(s for s in stats_name), end=')\n')
>         for stat in stats_name:
>             if stat == 'stats':
>                 df = strategy.get_stats(preprocessing = True)
>             elif stat == 'addstats':
>                 df = strategy.get_addstats(preprocessing = True)
>             elif stat == 'balsheets':
>                 df = strategy.get_balsheets_element(index_list)
>             elif stat == 'income':
>                 df = strategy.get_income_element(index_list)
>             elif stat == 'flow':
>                 df = strategy.get_flow_element(index_list)
> 
>             if SAVE == True:
>                 print('\n----------------------------- SAVE DATA  {} ---------------------------------\n'.format(stat))
>                 print('')
>                 df.to_json(url_pre+'/{0}_{1}_element.json'.format(index_name, stat))
>                 df.to_csv(url_pre+'/{0}_{1}_element.csv'.format(index_name, stat))
> 
>             if SHOW == True:
>                 print(df)
> 
> if __name__=='__main__':
>     from class_Strategy import LongTermStrategy
>     with open('config/config.json', 'r') as f:
>         config = json.load(f)
>     root_url = config['root_dir']
>     dow_list = yfs.tickers_dow()
>     filename = ''
>     s= input("Choice of stock's list (dow, sp500, nasdaq, other, all, selected): ")
>     if s == 'dow':
>         dow_list = yfs.tickers_dow()
>         filename = 'dow'
>     elif s == 'sp500':
>         filename = 'sp500'
>         dow_list = yfs.tickers_sp500()
>     elif s == 'nasdaq':
>         filename = 'nasdaq'
>         dow_list = yfs.tickers_nasdaq()
>     elif s == 'other':
>         filename = 'other'
>         dow_list = yfs.tickers_other()
>     elif s == 'all':
>         filename = 'all'
>         dow_list_1 = yfs.tickers_nasdaq()
>         dow_list_2 = yfs.tickers_other()
>         dow_list = dow_list_1 + dow_list_2
>     elif s == 'selected':
>         filename = 'selected'
>         url = 'data_ForTrading/selected_ticker.json'
>         temp_pd = pd.read_json(url)
>         temp_pd = temp_pd['Ticker']
>         dow_list = temp_pd.values.tolist()
> 
>     main(url=root_url, index_list = dow_list, index_name = filename, EACH=True, ALL=True, SAVE=True, SHOW=False)
> 
> else:
>     pass
> ```
>
> 

It is needed what to set various factor like PER PBR and so on.

And, we can choose parameter what you want in main funtion.



### Result

It is example for dow index not to show dataframes which are setted as parameter is False.

> ```
> Choice of stock's list (dow, sp500, nasdaq, other, all, selected): dow
> Preprocessing each Financial Statements
>    
> Each factor preprocessing in Basic stats (PER PSR PBR PEG forPER Cap)
> 
> ----------------------------- SAVE DATA  PER ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  PSR ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  PBR ---------------------------------
> 
> 
>    ----------------------------- SAVE DATA  PEG ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  forPER ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  Cap ---------------------------------
> 
> 
> Each factor preprocessing in Additional stats (Beta DivRate ROE ROA PM Cash Debt)
> 
> ----------------------------- SAVE DATA  Beta ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  DivRate ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  ROE ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  ROA ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  PM ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  Cash ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  Debt ---------------------------------
> 
> 
> Each factor preprocessing in Balance sheets (TA)
> 
> ----------------------------- SAVE DATA  TA ---------------------------------
> 
> 
> Each factor preprocessing in Income sheets (TR)
> 
> ----------------------------- SAVE DATA  TR ---------------------------------
> 
> 
> Each factor preprocessing in Cash flow (DIV ISS)
> 
> ----------------------------- SAVE DATA  DIV ---------------------------------
> 
> 
> ----------------------------- SAVE DATA  ISS ---------------------------------
> 
> 
> Preprocessing element in (stats addstats balsheets income flow)
> 
> ----------------------------- SAVE DATA  stats ---------------------------------
> 
> 
> 
> ----------------------------- SAVE DATA  addstats ---------------------------------
> 
> 
> For balance sheets
> 100%|████████████████████████████████████████████████████████████████████| 30/30 [00:01<00:00, 28.21it/s]
> 
> ----------------------------- SAVE DATA  balsheets ---------------------------------
> 
> 
> For income statements
> 100%|████████████████████████████████████████████████████████████████████| 30/30 [00:00<00:00, 45.14it/s]
> 
> ----------------------------- SAVE DATA  income ---------------------------------
> 
> 
> For cash flow
> 100%|████████████████████████████████████████████████████████████████████| 30/30 [00:00<00:00, 47.32it/s]
> 
> ----------------------------- SAVE DATA  flow ---------------------------------
> 
> ```
> 



### Conclusion

Let us know how to preprocess stock's financi statements in index.

If you want to use this code, I'm very sorry that you should change code and make directory for data a little bit.

I should appreciate and refer for many blog on google. Thanks a lot.

If you satisfied this post you should check [Github](https://github.com/hanseopark/Stock/tree/master/Strategy) and please **Star** :)

