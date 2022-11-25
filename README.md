# 高頻股票數據的罕見事件分析-以APPLE公司為例


## References：Rare Events Analysis for High‐Frequency Equity Data

>Author：Dragos Bozdog, Ionut¸ Florescu, Khaldoun Khashanah, and Jim Wang \
>July 2011Wilmott 2011(54):74 - 81 \
>DOI:10.1002/wilm.10016 \
>Stevens Institute of Technology, e-mail: ifloresc@stevens.edu \
>Project: Rare Events

## Abstract
本論文提出一種檢測罕見事件的方法，這些事件被定義為相對於交易量來說的較大的價格變動。\
我們在檢測到這些罕見事件後進行分析，並提供校準交易規則的方法，針對特定的交易規則進行說明。\
本論文應用這個方法在五天內對5,369支股票的數據進行分析，為了得出全面的結論，作者將股票分等並計算每個類別在這些罕見事件過後價格恢復的機率。\
本論文開發的方法基於非參數統計，不對研究中隨機變數 (Random Variable)的分佈做任何假設。

## Aim
• 開發一種即時檢測罕見事件的方法，其中價格變動較大且交易量相對較小的情況。\
• 分析這些罕見事件後的價格行為並研究價格回升的可能性。包含如果在檢測到的觀察值下進行交易，預期報酬率是多少？

## Methodology
本程式碼使用"Rare Events Analysis for High‐Frequency Equity Data"中的方法，並使用APPLY公司近一年的股票資料進行高頻股票數據的罕見事件分析。\
在價量曲線上，以一個固定視窗去分析出買進股票的時機點，當時機點發生在當價格急速下降的時候，再去分析買進的股票的報酬與時間的關係。

## programming environment
>google colaboratory

## Sampling Method: Rare Event Detection

A thorough investigation of the distribution of price changes, conditional on cumulative trading window, would involve the evaluation of all observations for each equity (S<sub>n</sub>-S<sub>j</sub> | V<sub>k</sub>+V<sub>k+1</sub>+...+V<sub>n</sub><V<sub>0</sub>) for k ≤ j ≤ n where n runs through all the trades, S<sub>n</sub> is the price, and V<sub>n</sub> is the volume associated with trade n. Although the distribution would be accurate, such a task would involve significant computational effort considering the large database we use. Because we are interested only in the rare events, we chose to select the extreme observations in this sequence and subsequently analyze only a percentage of these observations at the tails of the distribution Δp | V<V<sub>n</sub>. \
Specifically, we construct the sequence of consecutive trades S<sub>k</sub>,S<sub>k+1</sub>,...,S<sub>n</sub> and their associated volumes V<sub>k</sub>,V<sub>k,1</sub>,...,V<sub>n</sub>such that V<sub>k</sub>+V<sub>k+1</sub>+...+V<sub>n</sub><V<sub>0</sub> and we consider Δp<sub>n</sub>=max{S<sub>n</sub>-S<sub>k</sub>,S<sub>n</sub>-S<sub>k+1</sub>,S<sub>n</sub>-S<sub>n-1</sub>}

**公式(1):**

$$ Δpn = max\{S_{n} – S_{k}, S_{n} – S_{k+1}, . . ., S_{n} – S_{n-1}\} $$

>指在「一連串的連續交易之下」的最大價差 。

---
**公式(2):**

$$Q_{a}^{+}\left ( x \right ) = \lbrace \{ x:prob\left ( \Delta_{P}< x\right )< \alpha \ or\ prob\left ( \Delta_{P}>  x\right )> 1 - \alpha  \rbrace \}$$



>$Q_{a}^{+}\left ( x \right )$ :所有觀察值(所有罕見事件發生時的價差的總集合)\
>prob ( Δp < x ) < α :「價差（Δp）<自定義的價差（ｘ）間的機率是小於 α 」或\
>prob (Δp > x ) > 1 – α :「價差（Δp）>自定義的價差（ｘ）間的機率是大於1 – α 」

---
## process

><strong>Step1:install yfinance</strong> 
```python
# https://pypi.org/project/yfinance/

!pip install yfinance
```
```python
import yfinance as yf

msft = yf.Ticker("AAPL")

# get stock info
msft.info

# get historical market data
hist = msft.history(period="1y")
```
><strong>Step2:definition V0</strong> \
>因為事後視窗為3倍，因此將V0定義為累積交易量的1/4。
```python
V_Total = hist["Volume"].sum()
V0 = V_Total/4

V0

```
><strong>Step3:依據公式(1)找出最大價差的金額與發生的時間點</strong>\
>找出最大價差（價差由大到小排序後的最大值）是負的，代表我股價處於一直下跌的情況。\
dp< x代表的含義就是，我要從價格下降幅度最大的罕見事件買進，並進行事後分析。
```python
import pandas as pd

l_total_event = len(hist) # 所有事件的長度
day_index = hist.index # 撈出日期

max_dp = [] # 最大價差(等同於論文公式中的Q集合)
max_day = [] # 最大價差的天數
max_data_n = [] # 最大價差的日期(n)
max_data_k = [] # 最大價差的日期(k)

for n in range(1,l_total_event):
  V_sum = 0 # 累積交易量的初始化
  dp = [] # 價差的集合
  dp_day = [] # 價差的天數的集合
  Sn = hist["Close"][n] # 後項的價格
  
  for k in range(n):
    V_sum += hist["Volume"][k] # 計算累積交易量 ( Vk + ... + Vn-1 )
    if (V_sum<V0) :
      Sk = hist["Close"][k] # 前項的價格
      dp.append(Sn-Sk) # 新增價差(Sn-Sk)至dp
      dp_day.append([n,k]) # 新增價差的天數至dp_date
  
  dic1 = {
      "價差":dp, "發生的天數[n,k]":dp_day # n:後項的日期； k:前項的日期
  }
  
  df1 = pd.DataFrame(dic1)
  sort_df1 = df1.sort_values(by="價差", ascending=False) # 做完由大至小排序後的df1
  max_dp.append(sort_df1.iloc[0,0]) # 抓出由大至小排序後的df1的"價差"，並新增至max_dp
  max_day.append(sort_df1.iloc[0,1]) # 抓出由大至小排序後的df1的"發生的日期"，並新增至max_data
  max_data_n.append(day_index[max_day[n-1][0]]) # 抓出最大價差的日期(n) 
  max_data_k.append(day_index[max_day[n-1][1]]) # 抓出最大價差的日期(k) 

dic2 = {
      "最大價差":max_dp, "最大價差發生的天數[n,k]":max_day, "最大價差發生的日期[n]":max_data_n, "最大價差發生的日期[k]":max_data_k,  # n:後項的日期； k:前項的日期
  }
Q_df = pd.DataFrame(dic2)

Q_df


```
