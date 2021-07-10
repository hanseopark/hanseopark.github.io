---
layout: post
comments: true
title: How to get stock's price? 
permalink: /2021-06-28-stock-price/
---

### Stock's price

This post introduce for us to get stock price using API with python.

- pandas_datareader API

it is very useful API for free to be able to get at any web site.

> pip install pandas-datareader

- yahoo_fin API

It is very useful API for free to be able to get only yahoo finance in additional to get financial statements.

> pip install yahoo_fin

I use this both API in condition.

This APIs have advantage for free but, it is not official for website like yahoo finance.

Therefore, they were sometimes blocked. For free user like us, we use according to the situation and participate updating library :)



### Code to get stock's price

> ```python
> import pandas as pd
> import pandas_datareader as pdr
> import yahoo_fin.stock_info as yfs
> 
> from tqdm import tqdm
> import datetime
> 
> def main(index_name, start, end, run_yfs=False, run_pdr=True):
> 
>     index_list = yfs.tickers_dow()
>     if index_name == 'dow':
>         index_list = yfs.tickers_dow()
>     elif index_name == 'sp500':
>         index_list = yfs.tickers_sp500()
>     elif index_name == 'nasdaq':
>         index_list = yfs.tickers_nasdaq()
>     elif index_name == 'other':
>         index_list = yfs.tickers_other()
>     elif index_name == 'selected':
>         url = '/Users/hanseopark/Work/stock/data_ForTrading/selected_ticker.json'
>         temp_pd = pd.read_json(url)
>         temp_pd = temp_pd['Ticker']
>         index_list = temp_pd.values.tolist()
> 
>     print(index_list)
> 
>     print('Example Apple inc')
>     if run_pdr:
>         df_aapl = pdr.DataReader('AAPL', 'yahoo', start_day, today)
> 
>     if run_yfs:
>         start_day.strftime('%m/%d/%y')
>         today.strftime('%m/%d/%y')
>         df_aapl = yfs.get_data('AAPL',start_date=start_day, end_date = today)
>         df_aapl = df_aapl.reset_index()
>         df_aapl = df_aapl.rename(columns={'open':'Open', 'index':'Date', 'high':'High','low':'Low','close':'Close','adjclose':'Adj Close','volume':'Volume','ticker':'Ticker'})
>         df_aapl = df_aapl[['Ticker','Date','High','Low','Open','Close','Volume','Adj Close']]
>     print(df_aapl)
> 
>     df_recent = pd.DataFrame({'Recent_price': []})
>     etf_values = {}
>     error_symbols = []
>     for ticker in tqdm(index_list):
>         try:
>             if run_pdr:
>                 df = pdr.DataReader(ticker,'yahoo', start_day, today)
>             if run_yfs:
>                 df = yfs.get_data(ticker,start_date=start_day, end_date = today)
>                 df = df.reset_index()
>                 df = df.rename(columns={'open':'Open', 'index':'Date', 'high':'High','low':'Low','close':'Close','adjclose':'Adj Close','volume':'Volume','ticker':'Ticker'})
>                 df = df[['Ticker','Date','High','Low','Open','Close','Volume','Adj Close']]
>             df_recent.loc[ticker, 'Recent_price'] = yfs.get_live_price(ticker)
>             etf_values[ticker] = df
>         except:
>             error_symbols.append(ticker)
>     print('Error: ', error_symbols)
> 
>     if run_pdr:
>         combined_value = pd.concat(etf_values)
>         combined_value = combined_value.reset_index()
>         combined_value= combined_value.rename(columns={'level_0': 'Ticker'})
>     if run_yfs:
>         combined_value = pd.concat(etf_values)
>         combined_value = combined_value.reset_index()
>         combined_value = combined_value.drop(['level_0','level_1'], axis=1)
> 
>     print(combined_value)
>     print(df_recent)
> 
>     url = '/Users/hanseopark/Work/stock/data_origin/FS_{0}_Value'.format(index_name)
>     combined_value.to_json(url+'.json')
>     combined_value.to_csv(url+'.csv')
> 
>     url_recent = '/Users/hanseopark/Work/stock/data_origin/FS_{0}_Recent_Value'.format(index_name)
>     df_recent.to_json(url_recent+'.json')
>     df_recent.to_csv(url_recent+'.csv')
> 
> if __name__ == '__main__':
>     s= input("Choice of stock's list (dow, sp500, nasdaq, other, selected): ")
> 
>     ## Define time of start to end ##
>     #td_1y = datetime.timedelta(weeks=52/2)
>     start_day = datetime.datetime(2010,1,1)
>     today = datetime.datetime.now()
>     #start_day = today-td_1y
> 
>     print('Price of stcok in {0} for date series and recent price'.format(s))
>     main(index_name=s, start= start_day, end = today, run_yfs = True, run_pdr=False)
> 
> else:
>     pass
> 
> ```
>
> 

We can get in condition of setting from start day to end day (today) and Recent price of live.

And, we're also able to get index of stock like Dow and S&P500 etc.



### Result

The example result of upper code at dow index.

>```
>      Ticker       Date        High         Low        Open       Close       Volume   Adj Close
>0       AAPL 2009-12-31    7.619643    7.520000    7.611786    7.526072  352410800.0    6.471693
>1       AAPL 2010-01-04    7.660714    7.585000    7.622500    7.643214  493729600.0    6.572423
>2       AAPL 2010-01-05    7.699643    7.616071    7.664286    7.656428  601904800.0    6.583786
>3       AAPL 2010-01-06    7.686786    7.526786    7.656428    7.534643  552160000.0    6.479064
>4       AAPL 2010-01-07    7.571429    7.466072    7.562500    7.520714  477131200.0    6.467088
>...      ...        ...         ...         ...         ...         ...          ...         ...
>84407    WMT 2021-06-21  136.750000  135.350006  135.660004  136.399994    6925000.0  136.399994
>84408    WMT 2021-06-22  137.720001  136.229996  136.229996  137.029999    6369000.0  137.029999
>84409    WMT 2021-06-23  136.990005  135.929993  136.500000  135.960007    6463500.0  135.960007
>84410    WMT 2021-06-24  137.240005  136.100006  136.169998  136.910004    7814700.0  136.910004
>84411    WMT 2021-06-25  138.860001  136.919998  137.130005  138.529999    9544400.0  138.529999
>
>[84412 rows x 8 columns]
>      Recent_price
>AAPL    133.110001
>AMGN    242.679993
>AXP     169.449997
>BA      248.380005
>CAT     216.309998
>CRM     241.869995
>CSCO     53.060001
>CVX     107.300003
>DIS     178.350006
>DOW      63.290001
>GS      368.769989
>HD      313.630005
>HON     218.740005
>IBM     146.839996
>INTC     55.910000
>JNJ     164.210007
>JPM     154.050003
>KO       54.320000
>MCD     232.419998
>MMM     194.750000
>MRK      77.199997
>MSFT    265.019989
>NKE     154.350006
>PG      134.919998
>TRV     151.720001
>UNH     404.950012
>V       237.320007
>VZ       56.380001
>WBA      52.160000
>WMT     138.529999
>```
>
>



### Conclusion

Let us know how to get stock's price in index.

If you want to use this code, I'm very sorry that you should change code and make directory for data a little bit.

I should appreciate and refer for many blog on google. Thanks a lot.

If you satisfied this post you should check [Github](https://github.com/hanseopark/Stock/tree/master/Strategy) and please **Star** :)
