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
本程式碼使用"Rare Events Analysis for High‐Frequency Equity Data"論文中的方法，並使用APPLY公司近一年的股票資料進行高頻股票數據的罕見事件分析。\
在價量曲線上，以一個固定視窗去分析出買進股票的時機點，當時機點發生在當價格急速下降的時候，再去分析買進的股票的報酬與時間的關係。

## Programming Environment
>Google Colaboratory

## Sampling Method: Rare Event Detection

A thorough investigation of the distribution of price changes, conditional on cumulative trading window, would involve the evaluation of all observations for each equity (S<sub>n</sub>-S<sub>j</sub> | V<sub>k</sub>+V<sub>k+1</sub>+...+V<sub>n</sub><V<sub>0</sub>) for k ≤ j ≤ n where n runs through all the trades, S<sub>n</sub> is the price, and V<sub>n</sub> is the volume associated with trade n. Although the distribution would be accurate, such a task would involve significant computational effort considering the large database we use. Because we are interested only in the rare events, we chose to select the extreme observations in this sequence and subsequently analyze only a percentage of these observations at the tails of the distribution Δp | V<V<sub>n</sub>. \
Specifically, we construct the sequence of consecutive trades S<sub>k</sub>,S<sub>k+1</sub>,...,S<sub>n</sub> and their associated volumes V<sub>k</sub>,V<sub>k,1</sub>,...,V<sub>n</sub>such that V<sub>k</sub>+V<sub>k+1</sub>+...+V<sub>n</sub><V<sub>0</sub> and we consider Δp<sub>n</sub>=max{S<sub>n</sub>-S<sub>k</sub>,S<sub>n</sub>-S<sub>k+1</sub>,...,S<sub>n</sub>-S<sub>n-1</sub>}

**公式(1):**

$$ Δpn = max\lbrace \{ S_{n} – S_{k}, S_{n} – S_{k+1}, . . ., S_{n} – S_{n-1} \rbrace \} $$

>指在「一連串的連續交易之下」的最大價差 。

---
**公式(2):**

$$Q_{a}^{+}\left ( x \right ) = \lbrace \{ x:prob\left ( \Delta_{P}< x\right )< \alpha \ or\ prob\left ( \Delta_{P}>  x\right )> 1 - \alpha  \rbrace \}$$



>$Q_{a}^{+}\left ( x \right )$ :所有觀察值(所有罕見事件發生時的價差的總集合)\
>prob ( Δp < x ) < α :「價差（Δp）<自定義的價差（ｘ）間的機率是小於 α 」\
>prob (Δp > x ) > 1 – α :「價差（Δp）>自定義的價差（ｘ）間的機率是大於1 – α 」

---
## Process

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
><strong>Step2:definition V0</strong>
>
>因為事後視窗為3倍，因此將V0定義為累積交易量的1/4。\
>p.s V0為罕見事件發生的前數筆資料
```python
V_Total = hist["Volume"].sum()
V0 = V_Total/4

V0

```
><strong>Step3:依據公式(1)找出最大價差的金額與發生的時間點</strong>
>
>找出最大價差（價差由大到小排序後的最大值）是負的，代表我股價處於一直下跌的情況。\
dp< x代表的含義就是，我要從價格下降幅度最大的罕見事件買進，並進行事後分析。
```python
l_total_event = len(hist) # 所有事件的長度
day_index = hist.index # 撈出日期

max_dp = [] # 最大價差(等同論文公式中的Q集合)
max_day = [] # 最大價差的天數
max_data_n = [] # 最大價差的日期(n)
max_data_k = [] # 最大價差的日期(k)

for n in range(1,l_total_event):
  V_sum = 0 
  dp = [] # 價差的集合
  dp_day = [] # 價差的天數的集合
  Sn = hist["Close"][n] # 後項的價格
  
  for k in range(n):
    V_sum += hist["Volume"][k] # 計算累積交易量
    if (V_sum<V0) :
      Sk = hist["Close"][k] # 前項的價格
      dp.append(Sn-Sk)
      dp_day.append([n,k])
  
  dic1 = {
      "價差":dp, "發生的天數[n,k]":dp_day # n:後項的日期； k:前項的日期
  }
  
  df1 = pd.DataFrame(dic1)
  sort_df1 = df1.sort_values(by="價差", ascending=False)
  max_dp.append(sort_df1.iloc[0,0])
  max_day.append(sort_df1.iloc[0,1])
  max_data_n.append(day_index[max_day[n-1][0]])
  max_data_k.append(day_index[max_day[n-1][1]])


dic2 = {
      "最大價差":max_dp, "最大價差發生的天數[n,k]":max_day, "最大價差發生的日期[n]":max_data_n, "最大價差發生的日期[k]":max_data_k,  # n:後項的日期； k:前項的日期
  }
Q_df = pd.DataFrame(dic2)

Q_df

```
><strong>Step4:依據公式(2)找出罕見事件</strong>
>
>設alpha=0.05。\
>論文提及須找出一個x，使得prob(max_dp < x)時發生的機率<= alpha\
>這裡使用max_dp由小至大做排序，取前5%(alpha=0.05)的資料筆數，當作我的罕見事件
```python
alpha = 0.05

l = len(Q_df)
a = int(l * alpha)

print(a)

```
```python
Q_df1 = Q_df.sort_values(by="最大價差", ascending=True)
Q_df2 = Q_df1.iloc[:12,:].reset_index() # 取前5%(alpha=0.05)的資料筆數，當作我的罕見事件
Q_df2

```
><strong>Step5:分析事後視窗，並印出報酬率和機率的表格與圖表</strong>
>
>依據罕見事件發生的日期(共12筆)去畫圖\
>x軸為事後視窗大小(1, 2 ,3 倍)\
>Y軸為報酬率和機率
>
>使用定義2中的兩個交易規則:
>- 如果在事後窗口中出現有利的價格變動，我們可以使用可能的最佳報酬率平倉。
>- 如果在事後窗口內沒有發生有利的價格變動，我們會使用窗口內可能的最差報酬率來平倉。
```python
# 新增欄位

Q_df2.insert(5,column="Vae在1倍V0時，有利價格的機率",value=0)
Q_df2.insert(6,column="Vae在2倍V0時，有利價格的機率",value=0)
Q_df2.insert(7,column="Vae在3倍V0時，有利價格的機率",value=0)
Q_df2.insert(8,column="Vae在1倍V0時的報酬率",value=0)
Q_df2.insert(9,column="Vae在2倍V0時的報酬率",value=0)
Q_df2.insert(10,column="Vae在3倍V0時的報酬率",value=0)
Q_df2

```
```python

import matplotlib.pyplot as plt
import numpy as np 

for n in range(len(Q_df2)):
  happen = Q_df2["最大價差發生的天數[n,k]"][n][0] # 罕見事件發生的當天
  today_v = hist["Volume"][happen] # 罕見事件買進當天的交易量
  today_price = hist["Close"][happen] # 罕見事件買進當天的價格
  plt.figure(figsize= (28,6))

  for time in range(1,4): # 分析三倍
    Vae = 0 
    Vae_list = [] # 累積交易量的list
    judge_list = [] # 是否發生有利價格的機率，1是發生有利價格，0則是沒有發生
    favorable_price_list = [] # 有利價格的集合
    best_favorable_price = 0 # 最有利的價差
    worst_favorable_price = 0 # 最差的價差
    pass_day = 1 # 經過的天數

    while Vae < time*V0: # 當我的Vae < time倍的V0時
      next_happen = happen + pass_day
      if next_happen == len(hist):
        break
      next_v = hist["Volume"][next_happen] # 罕見事件幾天後的交易量
      next_price = hist["Close"][next_happen] # 罕見事件幾天後的價格

      Vae += next_v # 更新我的累積交易量為Vae
      Vae_list.append(Vae)
      

      favorable_price = next_price - today_price # 找出有利價格(favorable price)的價差
      pass_day += 1 


      if favorable_price > 0:
        judge_list.append(1) # 發生有利價差情況，新增1
        if favorable_price > best_favorable_price:
          favorable_price_list.append(favorable_price)
          best_favorable_price = favorable_price
        else:
          favorable_price_list.append(best_favorable_price) # 保留最有利的價差報酬
      else: 
        if best_favorable_price > 0:
          judge_list.append(1) # 發生有利價差情況，新增1
          favorable_price_list.append(best_favorable_price) # 保留最有利的價差報酬
        else:
          judge_list.append(0) # 發生不利價差情況，新增0
          if favorable_price < worst_favorable_price:
            favorable_price_list.append(worst_favorable_price)
            worst_favorable_price = favorable_price
          else:
            favorable_price_list.append(worst_favorable_price)
    

      # 以上使用定義2中的兩個交易規則
      # - 如果在事後窗口中出現有利的價格變動，我們可以使用可能的最佳報酬率平倉。
      # - 如果在事後窗口內沒有發生有利的價格變動，我們會使用窗口內可能的最差報酬率來平倉。
    
    Q_df2.iloc[n,time+4] = judge_list[-1] # 將Vae=1倍V0，2倍V0，3倍V0各別發生有利的機率放入表格中
    Q_df2.iloc[n,time+7] = favorable_price_list[-1]/(0.01*today_price) # 將Vae=1倍V0，2倍V0，3倍V0各別的報酬率放入表格中
 
    y = np.array(favorable_price_list) # 最好的價差
    x = np.array(Vae_list) # 累積交易量的事後視窗

    rare_date_after = str(Q_df2.iloc[n,3]).split()[0] # 切割出rare_date_after 罕見事件發生的當天的日期


    plt.subplot(1,4,time) # 繪製多張圖
    
    # 預期報酬率(%)圖
    plt.plot(x/V0, y/(0.01*today_price), 'bo-',label="return(%)")
    for i in range(len(x)):
        if i == 0:
            plt.text(x[i]/V0, (y[i]/today_price)*100-6, int((y[i]/today_price*100)), fontsize = 12)
            # 在每一筆資料，列印出相對應的的數值
        elif i == (len(x)-1):
            plt.text(x[i]/V0-0.16, y[i]/today_price*100-6, int((y[i]/today_price*100)), fontsize = 12)
            # 在每一筆資料，列印出相對應的的數值
        else:
            pass    
    
    # 有利價格變動的機率(%)圖
    judge_array = np.array(judge_list)
    plt.plot((x/V0), judge_array*100, 'ro-',label="prob(%)")
    for i in range(len(x)):
        if i == 0:
            plt.text(x[i]/V0, judge_array[i]*100+2, (judge_array[i]*100), fontsize = 12)
            # 在每一筆資料，列印出相對應的的數值
        elif i == (len(x)-1): # 尾
            plt.text(x[i]/V0-0.25, judge_array[i]*100+2, (judge_array[i]*100), fontsize = 12)
            # 在每一筆資料，列印出相對應的的數值
            pass
    
    plt.legend(loc = (0.75,0.75))
    plt.title("%s" % rare_date_after)
    plt.xlim(0,3) # 設定x軸的上下限
    plt.xlabel("Vae < %d*V0" % time)
    plt.ylabel("(%)")
    plt.ylim(-30,110) # 設定y軸的上下限
  plt.show()

# 因為 plt.text 只能針對當下去填寫資料內容，所以我才要跑回圈 

```
```python
Q_df2
```

## Conclusion
在我偵測到的罕見事件進行開倉(買進)的話，隨著事後視窗越大，我的有利價格的機率會越高，且報酬率會越佳。
