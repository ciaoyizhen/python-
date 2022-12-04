# 第十二章 利用RFM分析对用户进行分类

顾客是上帝，但是他们也是最善变的，不知道什么时候就离你而去。那我们应该如何去分析顾客，了解他们呢？RFM模型隶属于用户价值模型，有两个方向：一个是基于用户生命周期，也就是时间和用户在产品内的成长路径进行的生命周期模型的搭建；另一个就是基于用户关键行为进行搭建。其中，RFM模型是最经典的基于用户关键行为的模型，它是衡量用户价值和用户创利能力的一个重要工具和手段，被广泛应用在各个行业内。

## RFM分析简介

### RFM模型简述

+ R：最近一次消费(Recency)，代表用户距离当前最后一次消费的时间。

+ F：消费频次(Frequency)，用户在一段时间内，在产品内的消费频次
+ M：消费金额(Monetary)，代表用户的价值贡献。



有了RFM模型之后，对客户进行运营时就可以决定发送短信时，对那些用户加上前缀“尊敬的VIP用户”，对那些用户前缀“好久不见”。

这个模型还可以判断企业哪些用户有异动，有流失的预兆。



| 客户类别     | R    | F    | M    |
| ------------ | ---- | ---- | ---- |
| 重要价值客户 | 高   | 高   | 高   |
| 重要挽留客户 | 低   | 低   | 高   |
| 一般价值客户 | 中   | 低   | 低   |
| 一般发展客户 | 高   | 中   | 中   |
| ...          | .... | ...  | ...  |



有了RFM模型之后，运营人员就可以就基于模型的评分来更好的指导运营。

因为RFM告诉了我们：

+ 谁是最好的客户？
+ 哪些客户正处于流失的边缘？
+ 谁有可能转化为更有利可图的客户？
+ 谁是不需要关注的无价值客户？
+ 必须保留哪些客户？
+ 谁是忠实客户？
+ 哪些客户最有可能对当前的营销动作做出回应？



### 理解RFM

#### R——最近一次消费

对不同的企业，R有不同的含义。对电商而言，R指的是客户在店铺最近一次消费和当前的时间间隔，理论上R值越小的客户价值越高。而对社交网站、在线视频播放而言，R可能就是最近一次登录时间、最近一次发帖时间、最近一次投资时间、最近一次观看时间。以电商为例，目前网购便利，顾客已经有了更多的购买选择和更低的购买成本，去除地域的限制因素，客户非常容易流失，因此，要提高回购率和留存率，要时刻警惕R值。



#### F——消费频率

消费频率是客户再固定时间内（如1年，1个月）的购买次数。不同的行业，用户购买频率有很大的区别，如用户可能每个月都会购买纸尿布，而电子产品可能一年内也才消费一次。因此，消费频率取决于产品和行业。我们在构建RFM模型时，有时把F值的时间范围去掉，替换成累计购买次数。影响复购的核心因素是商品，因此对复购不适合做跨类目比较。例如食品类和美妆类：食品类属于“半标品”，产品的标品化程度越高，客户背叛的难度就越小，越难形成忠实用户；但是相对于美妆，食品又属于易耗品，消费周期短，购买频率高，相对容易产生重复购买。因此跨类目复购并不具有可比性。



#### M——消费金额

M值是RFM模型最 具有价值的指标。大家熟知的“二八定律”（又名“帕累托法则”）曾给出过这样的解释：公司的80%的收入来自20%的用户。理论上M值和F值是一样的，都带有时间范围，指的是一段时间（通常是1年或1个月）内的消费金额。有的情况下，我们还会考虑客单价，对于客单价高的用户，我们考虑是要如何提高他的购物频次。对客单价低的客户，我们要考虑是否可以想办法提高客单价。





## RFM实战

仍对同一个数据进行分析。

#### R、F、M值的计算

读入数据+缺失值处理

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
import seaborn as sns
import datetime
import warnings

warnings.filterwarnings('ignore')
sns.set_style('whitegrid')
plt.rcParams['font.sans-serif'] = 'SimHei'
plt.rcParams['axes.unicode_minus'] = False

%matplotlib inline

colors = sns.color_palette()

file = r'../data/OnlineRetail.csv'
online = pd.read_csv(file, encoding='Unicode_escape', parse_dates=['InvoiceDate'])
online.head()

# 处理缺失值, 这里通过掩码的方式获取
mask = online['CustomerID'].isnull()
mask

online_rfm = online[~mask]
online_rfm.head()
```

确定当前日

```python
online_rfm['Total'] = online_rfm['UnitPrice'] * online_rfm['Quantity']
online_rfm.head()

current_date = max(online_rfm['InvoiceDate']) + datetime.timedelta(days=1)
current_date  # 假设这一天为当前日
```

计算RFM

```python
df = online_rfm.groupby('CustomerID').agg({'InvoiceDate': lambda x:(current_date - x.max()).days,
                                          'InvoiceNo': 'count',
                                          'Total': 'sum'})
df.sample(3)

# 对列名进行重命名
df.rename(columns={'InvoiceDate':'Recency', 
                  'InvoiceNo': 'Frequency',
                  'Total': 'Monetary'}, inplace=True)
```

如此，df数据中，InvoiceDate表示了该用户当前日期与上次购物日期的间隔时间，InvoiceNo，表示了该用户的购物频次，Total表示了该用户的购物总金额

虽然现在已经得到了R,F,M值，但是由于这些值的分布很广，不利于后续的分析，因此对他们进行再加工。

```python
# 具体的数值不好分析,因此转为labels的方式,根据其值分为四个等级

r = range(4, 0, -1)  # 因为距离越近越好,所以值越小,反而要更大
r_quartiles = pd.cut(df['Recency'], 4, labels=r)
r_quartiles

f = range(1, 5)
m = range(1, 5)
f_quartiles = pd.qcut(df['Frequency'], 4, labels=f)
m_quartiles = pd.qcut(df['Monetary'], 4, labels=m)
```

把数据进行组合

```python
df = df.assign(R=r_quartiles)  # df['F'] = f_quartiles  assign方法不没有原地操作
df = df.assign(F=f_quartiles)
df = df.assign(M=m_quartiles)
```

如此，就在df中有了全部的数据。

#### 利用RFM模型对客户进行细分

建立分数指标

```python
def concat_rfm(df):
    return str(df['R'])+str(df['F'])+str(df['M'])

df['RFM_Segment'] = df.apply(concat_rfm, axis=1)
df.sample(10)

def rfm_sum(df):
    return df['R']+df['F']+df['M']
df['RFM_score'] = df.apply(rfm_sum, axis=1)
df.sample(10)
```

RFM_score现在是三列的求和，可以通过这一列将用户分为金牌客户，银牌客户，铜牌客户，进一步分析各细分群组。

```python
# 查看每个组合的数目
df.groupby('RFM_Segment').size().sort_values(ascending=False)[:10]
```

结果分析，444客户群代表的是最近购物时间距当前日期短，购物频率高且同时购物金额大的群体。通过分组统计可以发现这个群体是最大的，同时433、422这两个客户群也排在第2位和第3位，这说明网站大量客户的购物日期距当前日期都很近，网站的客户留存做得很好。不过从输出441群体也比较多，那么这部分客户购物日期距当前日期近，但是购物频率和购物金额都较小，要考虑这是不是由于是新客户导致？

RFM分析中用得最广泛的就是客户细分，下面基于RFM_score将客户分为三类。需要说明的是，前面的RFM_score的计算采用简单求和得到，实际分析中经常考虑赋予不同权重给各指标，为简单起见，这里没有考虑这个问题。

```python
# 根据分数，将其客户分为三个类别
def segment(series):
    if series >= 10:
        return 'Gold'
    elif series >= 6 and series <= 9:
        return 'Silver'
    else:
        return 'Bronze'
    
    
df['Segment'] = df['RFM_score'].map(segment)
df.sample(5)
```

查看每个类别的RFM详细值的平均值

```python
df.groupby('Segment').agg({'Recency': 'mean',
                           'Frequency': 'mean',
                           'Monetary': ['mean', 'count']}).round(1)
```

