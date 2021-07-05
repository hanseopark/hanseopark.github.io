---
layout: post
comments: true
title: 株式の財務諸表を得る方法は?
permalink: /2021-07-06-financial-statements/
---

### Stock's financial statements

株式は現在幾らかの価額も重要がこれよりも**財務諸表**(Financial statements)を知ることがより重要です。簡単に考えても株式を現在買う価額は意味なく時価総額が重要です。これは財務諸表の一種でこれ以外にも現在現金の流れななど、会社を評価する要素を多くあります。

代表的にPERまたはROAなどがあり、私は財務諸表を[yahoo finance site](https://finance.yahoo.com) 上を基盤として[yahoo_fin API](https://pypi.org/project/yahoo-fin/) をつかて測定してみます。具体的な使い方は[yahoo_fin documation](http://theautomatic.net/yahoo_fin-documentation/)を参考してください。

- **yahoo_fin API**

> pip install yahoo_fin



### Code to get stock's financial statements

> ```python
> 
> def main(stock_list):
>     # For check the table of one ticker like aapl
>     aapl_quote = yfs.get_quote_table('aapl')
>     aapl_val = yfs.get_stats_valuation('aapl')
>     aapl_ext = yfs.get_stats('aapl')
>     aapl_sheet = yfs.get_balance_sheet('aapl')
>     aapl_income = yfs.get_income_statement('aapl')
>     aapl_flow = yfs.get_cash_flow('aapl')
> 
>     print('*'*100)
>     print('quote')
>     print(aapl_quote)
>     print('*'*100)
>     print('basic')
>     print(aapl_val)
>     print('*'*100)
>     print('additional')
>     print(aapl_ext)
>     print('*'*100)
>     print('balance sheets')
>     print(aapl_sheet)
>     print(len(aapl_sheet.columns))
>     print('*'*100)
>     print('income statements')
>     print(aapl_income)
>     print('*'*100)
>     print('cash flow')
>     print(aapl_flow)
>     print('*'*100)
> 
>     # Get list of Dow tickers
>     dow_list = yfs.tickers_dow()
>     filename = ''
>     if stock_list == 'dow':
>         dow_list = yfs.tickers_dow()
>         filename = 'dow'
>     elif stock_list == 'sp500':
>         filename = 'sp500'
>         dow_list = yfs.tickers_sp500()
>     elif stock_list == 'nasdaq':
>         filename = 'nasdaq'
>         dow_list = yfs.tickers_nasdaq()
>     elif stock_list == 'other':
>         filename = 'other'
>         dow_list = yfs.tickers_other()
>     elif stock_list == 'selected':
>         filename = 'selected'
>         url = '/Users/hanseopark/Work/stock/data_ForTrading/selected_ticker.json'
>         temp_pd = pd.read_json(url)
>         temp_pd = temp_pd['Ticker']
>         dow_list = temp_pd.values.tolist()
> 
>     print(dow_list)
> 
>     # Get data in the current  olumn for each stock's valuation table
>     dow_stats = {}
>     dow_addstats = {}
>     dow_balsheets = {}
>     dow_income = {}
>     dow_flow = {}
> 
>     error_symbols = []
>     for ticker in tqdm(dow_list):
>         try:
>             # Getteing Summary
>             basic = yfs.get_stats_valuation(ticker)
>             basic =basic.iloc[:,:2]
>             basic.columns = ['Attribute', 'Recent']
>             dow_stats[ticker] = basic
> 
>             # Getting additioanl stats
>             add = yfs.get_stats(ticker)
>             add.columns = ['Attribute', 'Value']
>             dow_addstats[ticker] = add
> 
>             # Getting balance sheets
>             sheets = yfs.get_balance_sheet(ticker)
>             dow_balsheets[ticker] = sheets
> 
>             # Getting income statements
>             income = yfs.get_income_statement(ticker)
>             dow_income[ticker] = income
> 
>             # Getting cash flow statements
>             flow = yfs.get_cash_flow(ticker)
>             dow_flow[ticker] = flow
> 
>         except:
>             error_symbols.append(ticker)
>             print('Error ticker: ', ticker)
> 
>     print('error symol: ', error_symbols)
> 
>     for ticker in dow_balsheets.keys():
>         leng = len(dow_balsheets[ticker].columns)
>         dow_balsheets[ticker].columns = ['Before_'+str(i) for i in range(0,leng)]
>         dow_balsheets[ticker] = dow_balsheets[ticker].rename(columns={'Before_0': 'Recent'})
> 
>     for ticker in dow_income.keys():
>         leng = len(dow_income[ticker].columns)
>         dow_income[ticker].columns = ['Before_'+str(i) for i in range(0,leng)]
>         dow_income[ticker] = dow_income[ticker].rename(columns={'Before_0': 'Recent'})
> 
>     for ticker in dow_flow.keys():
>         leng = len(dow_flow[ticker].columns)
>         dow_flow[ticker].columns = ['Before_'+str(i) for i in range(0,leng)]
>         dow_flow[ticker] = dow_flow[ticker].rename(columns={'Before_0': 'Recent'})
> 
> 
>     combined_stats = pd.concat(dow_stats)
>     combined_stats = combined_stats.reset_index()
> 
>     combined_addstats = pd.concat(dow_addstats)
>     combined_addstats = combined_addstats.reset_index()
> 
>     combined_balsheets = pd.concat(dow_balsheets)
>     combined_balsheets = combined_balsheets.reset_index()
> 
>     combined_income = pd.concat(dow_income)
>     combined_income = combined_income.reset_index()
> 
>     combined_flow = pd.concat(dow_flow)
>     combined_flow = combined_flow.reset_index()
> 
>     del combined_stats['level_1']
>     del combined_addstats['level_1']
> 
>     print(combined_stats)
>     print(combined_addstats)
>     print(combined_balsheets)
>     print(combined_income)
>     print(combined_flow)
> 
>     list_stats = ['stats', 'addstats', 'balsheets', 'income', 'flow']
>     for s in list_stats:
>         url = '/Users/hanseopark/Work/stock/data_origin/FS_{0}_{1}'.format(filename, s)
>         if s == 'stats':
>             df = combined_stats
>         elif s == 'addstats':
>             df = combined_addstats
>         elif s == 'balsheets':
>             df = combined_balsheets
>         elif s == 'income':
>             df = combined_income
>         elif s == 'flow':
>             df = combined_flow
>         df.to_json(url+'.json')
>         df.to_csv(url+'.csv')
> 
> if __name__ == '__main__':
>     s= input("Choice of stock's list (dow, sp500, nasdaq, other, selected): ")
>     main(stock_list=s)
> 
> 
> ```
>
> 

まず、私たちはyahoo_fin APIを用いて生データを貰います。

続いて、データはのタイムスケールはTickerによって違いますが、簡単のため最近(Recent)と以前(Before_i)で指定して変換します。



### Result

アップル(AAPL)の生の財務諸表のデータは下のようになります。

> ```
> ****************************************************************************************************
> basic
>                              0      1
> 0      Market Cap (intraday) 5  2.34T
> 1           Enterprise Value 3   2.3T
> 2                 Trailing P/E  31.46
> 3                Forward P/E 1  26.16
> 4  PEG Ratio (5 yr expected) 1   1.45
> 5            Price/Sales (ttm)   7.18
> 6             Price/Book (mrq)  33.76
> 7   Enterprise Value/Revenue 3   7.07
> 8    Enterprise Value/EBITDA 7  23.05
> ****************************************************************************************************
> additional
>                                          Attribute         Value
> 0                                Beta (5Y Monthly)          1.21
> 1                                 52-Week Change 3           NaN
> 2                          S&P500 52-Week Change 3           NaN
> 3                                   52 Week High 3        145.09
> 4                                    52 Week Low 3         89.14
> 5                          50-Day Moving Average 3         128.8
> 6                         200-Day Moving Average 3        129.16
> 7                              Avg Vol (3 month) 3        82.31M
> 8                               Avg Vol (10 day) 3        64.61M
> 9                             Shares Outstanding 5        16.69B
> 10                    Implied Shares Outstanding 6           NaN
> 11                                           Float        16.67B
> 12                            % Held by Insiders 1         0.07%
> 13                        % Held by Institutions 1        58.69%
> 14                   Shares Short (May 27, 2021) 4       123.12M
> 15                    Short Ratio (May 27, 2021) 4          1.36
> 16               Short % of Float (May 27, 2021) 4         0.74%
> 17  Short % of Shares Outstanding (May 27, 2021) 4         0.74%
> 18       Shares Short (prior month Apr 29, 2021) 4        82.71M
> 19                  Forward Annual Dividend Rate 4          0.88
> 20                 Forward Annual Dividend Yield 4         0.66%
> 21                 Trailing Annual Dividend Rate 3          0.82
> 22                Trailing Annual Dividend Yield 3         0.60%
> 23                 5 Year Average Dividend Yield 4          1.34
> 24                                  Payout Ratio 4        18.34%
> 25                                 Dividend Date 3  May 12, 2021
> 26                              Ex-Dividend Date 4  May 06, 2021
> 27                             Last Split Factor 2           4:1
> 28                               Last Split Date 3  Aug 30, 2020
> 29                                Fiscal Year Ends  Sep 25, 2020
> 30                       Most Recent Quarter (mrq)  Mar 26, 2021
> 31                                   Profit Margin        23.45%
> 32                          Operating Margin (ttm)        27.32%
> 33                          Return on Assets (ttm)        16.90%
> 34                          Return on Equity (ttm)       103.40%
> 35                                   Revenue (ttm)       325.41B
> 36                         Revenue Per Share (ttm)         19.14
> 37                  Quarterly Revenue Growth (yoy)        53.60%
> 38                              Gross Profit (ttm)       104.96B
> 39                                          EBITDA        99.82B
> 40                  Net Income Avi to Common (ttm)        76.31B
> 41                               Diluted EPS (ttm)          4.45
> 42                 Quarterly Earnings Growth (yoy)       110.10%
> 43                                Total Cash (mrq)        69.83B
> 44                      Total Cash Per Share (mrq)          4.18
> 45                                Total Debt (mrq)       134.74B
> 46                         Total Debt/Equity (mrq)        194.78
> 47                             Current Ratio (mrq)          1.14
> 48                      Book Value Per Share (mrq)          4.15
> 49                       Operating Cash Flow (ttm)        99.59B
> 50                    Levered Free Cash Flow (ttm)        80.12B
> ****************************************************************************************************
> balance sheets
> endDate                    2020-09-26    2019-09-28    2018-09-29    2017-09-30
> Breakdown
> totalLiab                258549000000  248028000000  258578000000  241272000000
> totalStockholderEquity    65339000000   90488000000  107147000000  134047000000
> otherCurrentLiab          47867000000   43242000000   39293000000   38099000000
> totalAssets              323888000000  338516000000  365725000000  375319000000
> commonStock               50779000000   45174000000   40201000000   35867000000
> otherCurrentAssets        11264000000   12352000000   12087000000   13936000000
> retainedEarnings          14966000000   45898000000   70400000000   98330000000
> otherLiab                 46108000000   50503000000   48914000000   43251000000
> treasuryStock              -406000000    -584000000   -3454000000    -150000000
> otherAssets               33952000000   32978000000   22283000000   18177000000
> cash                      38016000000   48844000000   25913000000   20289000000
> totalCurrentLiabilities  105392000000  105718000000  115929000000  100814000000
> shortLongTermDebt          8773000000   10260000000    8784000000    6496000000
> otherStockholderEquity     -406000000    -584000000   -3454000000    -150000000
> propertyPlantEquipment    45336000000   37378000000   41304000000   33783000000
> totalCurrentAssets       143713000000  162819000000  131339000000  128645000000
> longTermInvestments      100887000000  105341000000  170799000000  194714000000
> netTangibleAssets         65339000000   90488000000  107147000000  134047000000
> shortTermInvestments      52927000000   51713000000   40388000000   53892000000
> netReceivables            37445000000   45804000000   48995000000   35673000000
> longTermDebt              98667000000   91807000000   93735000000   97207000000
> inventory                  4061000000    4106000000    3956000000    4855000000
> accountsPayable           42296000000   46236000000   55888000000   44242000000
> ****************************************************************************************************
> income statements
> endDate                              2020-09-26    2019-09-28    2018-09-29    2017-09-30
> Breakdown
> researchDevelopment                 18752000000   16217000000   14236000000   11581000000
> effectOfAccountingCharges                  None          None          None          None
> incomeBeforeTax                     67091000000   65737000000   72903000000   64089000000
> minorityInterest                           None          None          None          None
> netIncome                           57411000000   55256000000   59531000000   48351000000
> sellingGeneralAdministrative        19916000000   18245000000   16705000000   15261000000
> grossProfit                        104956000000   98392000000  101839000000   88186000000
> ebit                                66288000000   63930000000   70898000000   61344000000
> operatingIncome                     66288000000   63930000000   70898000000   61344000000
> otherOperatingExpenses                     None          None          None          None
> interestExpense                     -2873000000   -3576000000   -3240000000   -2323000000
> extraordinaryItems                         None          None          None          None
> nonRecurring                               None          None          None          None
> otherItems                                 None          None          None          None
> incomeTaxExpense                     9680000000   10481000000   13372000000   15738000000
> totalRevenue                       274515000000  260174000000  265595000000  229234000000
> totalOperatingExpenses             208227000000  196244000000  194697000000  167890000000
> costOfRevenue                      169559000000  161782000000  163756000000  141048000000
> totalOtherIncomeExpenseNet            803000000    1807000000    2005000000    2745000000
> discontinuedOperations                     None          None          None          None
> netIncomeFromContinuingOps          57411000000   55256000000   59531000000   48351000000
> netIncomeApplicableToCommonShares   57411000000   55256000000   59531000000   48351000000
> ****************************************************************************************************
> cash flow
> endDate                                 2020-09-26   2019-09-28   2018-09-29   2017-09-30
> Breakdown
> investments                             5335000000  58093000000  30845000000 -33542000000
> changeToLiabilities                    -1981000000  -2548000000   9172000000   8373000000
> totalCashflowsFromInvestingActivities  -4289000000  45896000000  16066000000 -46446000000
> netBorrowings                           2499000000  -7819000000    432000000  29014000000
> totalCashFromFinancingActivities      -86820000000 -90976000000 -87876000000 -17974000000
> changeToOperatingActivities              881000000   -896000000  30016000000  -8480000000
> issuanceOfStock                          880000000    781000000    669000000    555000000
> netIncome                              57411000000  55256000000  59531000000  48351000000
> changeInCash                          -10435000000  24311000000   5624000000   -195000000
> repurchaseOfStock                     -75992000000 -69714000000 -75265000000 -34774000000
> totalCashFromOperatingActivities       80674000000  69391000000  77434000000  64225000000
> depreciation                           11056000000  12547000000  10903000000  10157000000
> otherCashflowsFromInvestingActivities   -791000000  -1078000000   -745000000   -124000000
> dividendsPaid                         -14081000000 -14119000000 -13712000000 -12769000000
> changeToInventory                       -127000000   -289000000    828000000  -2723000000
> changeToAccountReceivables              6917000000    245000000  -5322000000  -2093000000
> otherCashflowsFromFinancingActivities   -126000000   -105000000   -105000000   -105000000
> changeToNetincome                       6517000000   5076000000 -27694000000  10640000000
> capitalExpenditures                    -7309000000 -10495000000 -13313000000 -12451000000
> ****************************************************************************************************
> ```
>
> 

解析のため変換したデータフレイムは下のようになります。

一つの例としてDow indexに属するTickerを測定しました。

>```
>    Ticker                    Attribute Recent
>0     AAPL      Market Cap (intraday) 5  2.34T
>1     AAPL           Enterprise Value 3   2.3T
>2     AAPL                 Trailing P/E  31.46
>3     AAPL                Forward P/E 1  26.16
>4     AAPL  PEG Ratio (5 yr expected) 1   1.45
>..     ...                          ...    ...
>265    WMT  PEG Ratio (5 yr expected) 1   3.26
>266    WMT            Price/Sales (ttm)   0.70
>267    WMT             Price/Book (mrq)   5.02
>268    WMT   Enterprise Value/Revenue 3   0.76
>269    WMT    Enterprise Value/EBITDA 7  10.78
>
>[270 rows x 3 columns]
>     Ticker                     Attribute   Value
>0      AAPL             Beta (5Y Monthly)    1.21
>1      AAPL              52-Week Change 3     NaN
>2      AAPL       S&P500 52-Week Change 3     NaN
>3      AAPL                52 Week High 3  145.09
>4      AAPL                 52 Week Low 3   89.14
>...     ...                           ...     ...
>1525    WMT       Total Debt/Equity (mrq)   74.74
>1526    WMT           Current Ratio (mrq)    0.95
>1527    WMT    Book Value Per Share (mrq)   27.93
>1528    WMT     Operating Cash Flow (ttm)  31.91B
>1529    WMT  Levered Free Cash Flow (ttm)  17.24B
>
>[1530 rows x 3 columns]
>    Ticker               Breakdown        Recent      Before_1      Before_2      Before_3
>0     AAPL               totalLiab  2.585490e+11  2.480280e+11  2.585780e+11  2.412720e+11
>1     AAPL  totalStockholderEquity  6.533900e+10  9.048800e+10  1.071470e+11  1.340470e+11
>2     AAPL        otherCurrentLiab  4.786700e+10  4.324200e+10  3.929300e+10  3.809900e+10
>3     AAPL             totalAssets  3.238880e+11  3.385160e+11  3.657250e+11  3.753190e+11
>4     AAPL             commonStock  5.077900e+10  4.517400e+10  4.020100e+10  3.586700e+10
>..     ...                     ...           ...           ...           ...           ...
>784    WMT       netTangibleAssets  4.704200e+10  3.839600e+10  3.551500e+10  5.962700e+10
>785    WMT          netReceivables  6.516000e+09  6.284000e+09  6.283000e+09  5.614000e+09
>786    WMT            longTermDebt  4.165000e+10  4.441000e+10  4.394800e+10  3.023100e+10
>787    WMT               inventory  4.494900e+10  4.443500e+10  4.426900e+10  4.378300e+10
>788    WMT         accountsPayable  4.914100e+10  4.697300e+10  4.706000e+10  4.609200e+10
>
>[789 rows x 6 columns]
>    Ticker                          Breakdown        Recent      Before_1      Before_2      Before_3
>0     AAPL                researchDevelopment   18752000000   16217000000   14236000000   11581000000
>1     AAPL          effectOfAccountingCharges          None          None          None          None
>2     AAPL                    incomeBeforeTax   67091000000   65737000000   72903000000   64089000000
>3     AAPL                   minorityInterest          None          None          None          None
>4     AAPL                          netIncome   57411000000   55256000000   59531000000   48351000000
>..     ...                                ...           ...           ...           ...           ...
>655    WMT                      costOfRevenue  420315000000  394605000000  385301000000  373396000000
>656    WMT         totalOtherIncomeExpenseNet   -6384000000   -1352000000  -10497000000   -5814000000
>657    WMT             discontinuedOperations          None          None          None          None
>658    WMT         netIncomeFromContinuingOps   13706000000   15201000000    7179000000   10523000000
>659    WMT  netIncomeApplicableToCommonShares   13510000000   14881000000    6670000000    9862000000
>
>[660 rows x 6 columns]
>    Ticker                              Breakdown        Recent      Before_1      Before_2      Before_3
>0     AAPL                            investments  5.335000e+09  5.809300e+10  3.084500e+10 -3.354200e+10
>1     AAPL                    changeToLiabilities -1.981000e+09 -2.548000e+09  9.172000e+09  8.373000e+09
>2     AAPL  totalCashflowsFromInvestingActivities -4.289000e+09  4.589600e+10  1.606600e+10 -4.644600e+10
>3     AAPL                          netBorrowings  2.499000e+09 -7.819000e+09  4.320000e+08  2.901400e+10
>4     AAPL       totalCashFromFinancingActivities -8.682000e+10 -9.097600e+10 -8.787600e+10 -1.797400e+10
>..     ...                                    ...           ...           ...           ...           ...
>539    WMT                      changeToInventory -2.395000e+09 -3.000000e+08 -1.311000e+09 -1.400000e+08
>540    WMT             changeToAccountReceivables -1.086000e+09  1.540000e+08 -3.680000e+08 -1.074000e+09
>541    WMT  otherCashflowsFromFinancingActivities -1.670000e+09 -1.463000e+09 -1.060000e+09 -4.018000e+09
>542    WMT                      changeToNetincome  3.440000e+09 -2.860000e+08  1.011000e+10  4.703000e+09
>543    WMT                    capitalExpenditures -1.026400e+10 -1.070500e+10 -1.034400e+10 -1.005100e+10
>
>[544 rows x 6 columns]
>```
>
>



### Conclusion

このようにアメリカの株式の情報を得ることができました。

もし、実際にこのコードを動いてみたい人は、ディレクトリのコードを直して動いてください。

このコードはクーグル上の沢山の人に手伝いました。本当にありがとうございます。

もしこのページに満足したら[Github](https://github.com/hanseopark/Stock/tree/master/ChoiceTicker) に行って **Star** お願いします :)

