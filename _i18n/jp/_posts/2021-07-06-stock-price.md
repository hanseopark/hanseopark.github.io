---
layout: post
comments: true
title: アメリカの単元株価格を得る方法は?
permalink: /2021-07-06-stock-price/
---

### Stock's price

このページはパイソン（Python）APIを用いてアメリカの単元株価格をを得る方法について紹介します。

私が使うAPIに関しては多く2つあります。

- pandas_datareader API

代表的に皆さんがよく使うAPIです。このAPIは値段以外にもたくさんの情報を得ることができます。

> pip install pandas-datareader

- yahoo_fin API

私がより使うAPIはこれです。これはヤフーファイナンスの情報をクローリングして株式の情報を集めることです。これは公式的なAPIではないがただのクローリングとして情報を得るから特に問題ないと思います。

> pip install yahoo_fin



2つとも評価がいいAPIです。使い方法はこのページ以外にもGoogleの上に沢山あると思います。



### Code to get stock's price

> ```python
> import pandas as pd
> import pandas_datareader as pdr
> import yahoo_fin.stock_info as yfs
> 
> from tqdm import tqdm
> import datetime
> import json
> 
> def main(url, index_list, index_name, start, end, run_yfs=False, run_pdr=True):
> 
>     ## Configuration directory url
>     url_data = url+'data_origin/'
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
>             print('Error: ', error_symbols)
>             df_recent.loc[ticker, 'Recent_price'] = yfs.get_live_price(ticker)
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
>     url_value_data = url_data+'FS_{0}_Value'.format(index_name)
>     combined_value.to_json(url_value_data+'.json')
>     combined_value.to_csv(url_value_data+'.csv')
> 
>     url_recent_value_data = url_data+'FS_{0}_Recent_Value'.format(index_name)
>     df_recent.to_json(url_recent_value_data+'.json')
>     df_recent.to_csv(url_recent_value_data+'.csv')
> 
> if __name__ == '__main__':
>     with open('../config/config.json', 'r') as f:
>         config = json.load(f)
>     root_url = config['root_dir']
> 
>     filename= input("Choice of stock's list (dow, sp500, nasdaq, other, selected): ")
> 
>     dow_list = yfs.tickers_dow()
>     if filename == 'dow':
>         dow_list = yfs.tickers_dow()
>     elif filename == 'sp500':
>         dow_list = yfs.tickers_sp500()
>     elif filename == 'nasdaq':
>         dow_list = yfs.tickers_nasdaq()
>     elif filename == 'other':
>         dow_list = yfs.tickers_other()
>     elif filename == 'selected':
>         url = root_url+'/data_ForTrading/selected_ticker.json'
>         temp_pd = pd.read_json(url)
>         temp_pd = temp_pd['Ticker']
>         dow_list = temp_pd.values.tolist()
> 
>     print(dow_list)
> 
>     ## Define time of start to end ##
>     #td_1y = datetime.timedelta(weeks=52/2)
>     start_day = datetime.datetime(2010,1,1)
>     today = datetime.datetime.now()
>     #start_day = today-td_1y
> 
>     print('Price of stcok in {0} for date series and recent price'.format(filename))
>     main(url=root_url, index_list = dow_list, index_name=filename, start= start_day, end = today, run_yfs = True, run_pdr=False)
> 
> else:
>     pass
> ```
>
> 



願うIndexに関してyahoo_fin APIを用いてTickerをリストとして呼びます。

各Tickerに関して指定する日から今日までの単元株価格を測定します。さらに、現在の値段も別のデータフレイムとしてセーブします。NasdaqとotherのIndexは結構時間掛かるのでご注意ください。



### Result

代表的にDow indexに関する情報を下に表します。　

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

このようにアメリカの株式の情報を得ることができました。

もし、実際にこのコードを動いてみたい人は、ディレクトリのコードを直して動いてください。

このコードはクーグル上の沢山の人に手伝いました。本当にありがとうございます。

もしこのページに満足したら[Github](https://github.com/hanseopark/Stock/tree/master/ChoiceTicker) に行って **Star** お願いします :)

