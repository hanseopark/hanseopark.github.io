---
layout: post
comments: true
title: 파이썬을 이용한 미국주식의 가격, 시세, 종가등을 얻기
permalink: /2021-07-07-stock-price/
---

### Stock's price

이번 포스트에서 파이썬 (Python) API를 이용해서 미국의 단위주식의 가격을 소개할 예정입니다.

주로 사용하는 API로는 두가지가 있습니다.

- pandas_datareader API

대표적으로 널리 사용되는 API이입니다. 사용법은 구글위에 다양한 정보가 있으니 참고해주시길 바랍니다.

> pip install pandas-datareader

- yahoo_fin API

제가 자주 이용하는  API 입니다. 공식적이 API가 아니라 안정성이 떨어지지만 직접적으로 야후 파이넨셜 사이트에서 정보를 가져오기 떄문에 알기쉽게 데이터를 크롤링 할 수 있습니다.

> pip install yahoo_fin



2가지 전부 평가가 좋은 API입니다. 사용에 따라 두가지를 적절히 이용해주시기 바랍니다.



### Code to get stock's price

> ```python
> import pandas as pd
> import pandas_datareader as pdr
> import yahoo_fin.stock_info as yfs
> 
> from tqdm import tqdm
> import datetime
> 
> def main(stock_list, start, end):
> 
>     etf_list = yfs.tickers_dow()
>     if stock_list == 'dow':
>         etf_list = yfs.tickers_dow()
>         filename = 'dow'
>     elif stock_list == 'sp500':
>         filename = 'sp500'
>         etf_list = yfs.tickers_sp500()
>     elif stock_list == 'nasdaq':
>         filename = 'nasdaq'
>         etf_list = yfs.tickers_nasdaq()
>     elif stock_list == 'other':
>         filename = 'other'
>         etf_list = yfs.tickers_other()
>     elif stock_list == 'selected':
>         filename = 'selected'
>         url = '/Users/hanseopark/Work/stock/data_ForTrading/selected_ticker.json'
>         temp_pd = pd.read_json(url)
>         temp_pd = temp_pd['Ticker']
>         etf_list = temp_pd.values.tolist()
> 
>     print(etf_list)
> 
>     df_recent = pd.DataFrame({'Recent_price': []})
>     etf_values = {}
>     error_symbols = []
>     for ticker in tqdm(etf_list):
>         try:
>             df = pdr.DataReader(ticker,'yahoo', start_day, today)
>             df_recent.loc[ticker, 'Recent_price'] = yfs.get_live_price(ticker)
>             etf_values[ticker] = df
>         except:
>             error_symbols.append(ticker)
> 
>     combined_value = pd.concat(etf_values)
>     combined_value = combined_value.reset_index()
>     combined_value= combined_value.rename(columns={'level_0': 'Ticker'})
> 
>     print(combined_value)
>     print(df_recent)
> 
>     url = '/Users/hanseopark/Work/stock/data_origin/FS_{0}_Value'.format(filename)
>     combined_value.to_json(url+'.json')
>     combined_value.to_csv(url+'.csv')
> 
>     url_recent = '/Users/hanseopark/Work/stock/data_origin/FS_{0}_Recent_Value'.format(filename)
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
>     main(stock_list=s, start= start_day, end = today)
> 
> ```
>
> 



원하는 인덱스(Index)에 대해 yahoo_fin API를 이용하여 각 주식(Ticker)을 리스트로 부릅니다.

각각의 Ticker에 대해 지정하 날로부터 오늘까지의 종가를 포함한 각 시간열에 대해 가격 정보를 얻어옵니다. 또한, 현재 가격만을 따로 데이터프레임으로 저장합니다. 추후 데이터 해석을 위해 미리 뽑아옵니다. Nasdaq과 other의 인덱스는 꽤 시간이 걸리므로 주의 해 주시길 바랍니다.



### Result

위 코드를 진행할 경우 밑과 같은 결과가 나타납니다. yahoo_fin API에 의한 원시 데이터는 대표적으로 다우 인덱스 대해 뽑아봤습니다.

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

이렇게 미국 주식의 시간열에 대한 주식 가격(시세, 종가) 및 현재가를 얻어봤습니다.

만약, 실제로 이 코드를 실행하고 싶으시다면, 폴더 Url을 변경해서 사용해주세요.

이 코드는 구글 위에 다양한 사람들에게 도움을 받았습니다. 대단히 감사합니다.

만약 이 포스트가 만족스럽다면 [Github](https://github.com/hanseopark/Stock/tree/master/ChoiceTicker) 의 **Star** 부탁드립니다 :)

