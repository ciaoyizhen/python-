# 第五章 开始利用Pandas进行数据分析

在开始正式的数据分析之前，数据分析人员通常都会进行探索性数据分析。

这一工作的目标是对要分析的数据有一个初步了解：各项数据的定义如何、数据有无缺失、是否有异常数据、数据的分布如何。在探索性数据分析过程中，数据分析人员还经常会对数据进行描述性统计、简单的数据可视化等操作以发现数据中的模式，帮助他们提出假设。



#### 元数据

元数据（Metadata），又称为中介数据、中继数据，它是描述数据的数据（Dataabout Data），主要是妙手数据属性（Property）的信息。

Pandas数据分析中所指的元数据主要包括了数据维度、数据类型、大小以及数据字典等。

```python
# 数据大小
DataFrame.shape
# 数据总数
DataFrame.size  # 也就是行乘列，三维就是全乘
# 数据维度
DataFrame.ndim

# 获取数据元数据信息的类型、占用内存大小信息。
DataFrame.info()
```

有的数据文件会提供一个数据字典,该数据字典通常会提供各数据列的名称、说明、数据类型、取值范围等信息。（即就是某一列里面文字描述这一个信息）



#### 数据类型转换

数据分析人员通常会把数据分为连续变量和离散变量。

然而pandas有自己的一套系统类型(dtypes)

|  pandas类型   | python类型 |    numpy类型     |           用途           |
| :-----------: | :--------: | :--------------: | :----------------------: |
|    object     |    str     |      string      | 文本或数字和非数字混合值 |
|     int64     |    int     | int系列,uint系列 |           整型           |
|    float64    |   float    |    float系列     |          浮点型          |
|     bool      |    bool    |      bool_       |        True/False        |
|  datetime64   |     NA     |  datetime64[ns]  |       时间和日期值       |
| timedelta[ns] |     NA     |        NA        |         时间差值         |
|   category    |     NA     |        NA        |      有限的文本列表      |

通常在数据分析时,会对数据所占内存进行精确统计

```python
# 获取每一列的内存使用情况，以字节数返回，可以通过/（1024*1024）转化为兆字节
DataFrame.memory_usage(index=True, deep=False)
"""
index=True, 则对列, index=False,则对行
deep=False, 是估计, deep=True, 则是真实计算
"""
```



对各列数据类型、所占内存大小有了了解后，就需要根据具体需求来调整数据的类型。

:red_circle:**unique与nunique**

```python
# unique是统计不同值有什么.这个方法只有Series可以使用,DataFrame不允许
retail_data['Country'].unique()  # 该方法没有内置参数,无法排除None值

# nunique是统计不同值有多少个.这个方法Series和DataFrame都可以使用
retail_data[['Country']].nunique()  # 可以通过传入dropna=True来排除None值
retail_data['Country'].nunique()
```







例如，Country列的可能取值并不多，完全可以转换为Pandas中的category变量以减少存储空间

```python
# 通过调用astype进行类型转化
retail_data['Country'] = retail_data['Country'].astype('category')
# category是pandas中定义的一个数据类型
# 可以对特点的类型数据进行按照自己的意愿进行排序，特别是我们在处理数据是需要对字符串进行排序时.

# 有内置函数
"""
to_datetime
to_numeric
"""
retail_data['InvoiceDate'] = pd.to_datetime(retail_data['InvoiceDate'])
retail_data[['InvoiceDate']].dtypes
```





### 缺失数据与异常处理

理想很丰满，现实很骨感。数据也是一样，真实数据分析中的数据经常会出现各种问题，要么有缺失，要么有重复，有的情况下数据还有错误，有时数据需要进行变换才能用于分析。



#### 缺失值

**:red_circle:统计缺失数据**

```python
# isnull()函数会对数据中的所有数据是否存在缺失值(NaN)进行判断，她返回了一个与原数据维度大小相同的DataFrame，而取值是由该处是否有缺失值决定。
# notnull与isnull功能正好相反。

# DataFrame使用is
retail_data.isnull().sum()  # 对一个DataFrame使用sum,则会返回Series
retail_data.isnull().sum().sum()  # 对一个Series使用sum,则会得到一个值
```



然而，现实中的缺失值并不总是以NaN的形式存在，一旦开始数据分析，就会发现各种五花八门的代表缺失值的方式，如0代表确实、-999代表确实等。那么该怎么处理？Pandas中的数据读入函数已经充分考虑了各种可能性。

```python
file = r'../data/OnlineRetail.csv'
retail_data_na = pd.read_csv(file, na_values=[0, 'United Kingdom'])  # na_values中的值就是将数据中的该值对应到NaN
```



#### 重复值

```python
# Series.duplicated()  DataFrame是针对整个行

# 第一次出现的值为False,第二次出现的值为True

retail_data.duplicated().sum()  # 该方法可以统计重复的数据有多少个

retail_data[((retail_data['InvoiceNo']=='536409')&(retail_data['StockCode']=='21866'))]  # 给出重复的数据
# 上述代码会给出两行一样的数据
```

出现两行一样的样本，要考虑是不是用户购买了两件相同的物品，还是因为数据本身被重复记录了两次。





#### 处理缺失数据

**:red_circle:删除缺失值**

```python
# dropna()方法
"""
默认axis=0, 若缺失,则删除一整行
how  默认是有缺失,就删除这一行,采用'all'方式的话,则为全为缺失,才删除这一行
thresh  阈值,要求多少列数据不是缺失值时才输出
"""
import numpy as np
import pandas as pd

df = pd.DataFrame(np.random.randint(0, 100, 15).reshape(5, 3),
                 index=['a', 'b', 'c', 'd', 'e'],
                 columns=['c1', 'c2', 'c3'])
df['c4'] = np.nan
df.loc['f'] = np.arange(10, 14)
df.loc[:, 'c5'] = np.nan
df.loc['g'] = np.nan
df['c4']['a'] = 18


df.dropna(axis=0, thresh=df.shape[0]*0.5)
```



**:red_circle:Numpy与pandas的不同之处**

> + numpy的NaN是不能进行计算,计算就会返回NaN
> + pandas的Nan会自动被忽略计算



:red_circle:缺失值填充

```python
# fillna()方法  直接传入值,使用method方法, limit参数表示只填充几次

"""
df.fillna(df.mean())  # 直接传入均值填充
df.fillna(0)  # 直接用0进行填充

时序数据一般需要使用前后值来进行插值
df.fillna(method='ffill')  # 用前面的值进行填充
df.fillna(methon='bfill')  # 用后面的值进行填充
"""
df.fillna(method='ffill')
df.fillna(df.mean())

# 某些特殊情况,要手工补齐指定的缺失值,可以使用下面的方法
fill_values = pd.Series([1, 2], index=['b', 'c'])
df['c4'].fillna(fill_values)  # 这种方法其实可以直接一个值一个值的插入

# 还可以使用插值函数
s = pd.Series([1, 2, np.nan, 5, np.nan, 9])
s.interpolate()

# 插值函数的使用,若索引不是时间的插值,计算的时候只是根据数据的值来进行,然而索引是时间的话,插值就会考虑时间的间隔。
ts = pd.Series([1, np.nan, 2],
              index=[datetime.datetime(2016, 1, 1),
                     datetime.datetime(2016, 2, 1),
                     datetime.datetime(2016, 4, 1)])
ts.interpolate()  # 忽略时间影响
ts.interpolate(method='time')  # 计算时间影响
ts.interpolate(methon='values')  # 根据索引值进行插值


```



#### 处理重复数据]

```python
# data.drop_duplicates()  删除重复值数据,可以选择保留前面还是后面的数据,还可以根据输入的参数选择删除指定检查的列

data = pd.DataFrame({'a':['x']*3 + ['y']*4,
                    'b':[1, 1, 2, 3, 3, 4, 4]})
data.drop_duplicates()
data['c'] = np.arange(7)
data.drop_duplicates(['a', 'b'])  # 只判断这两列是否重复,其他列重复不算
```



#### 异常值

数据分析中，除了重复值和缺失值，还有异常值，异常值的判断一方面来自常识（如天气温度不可能是100℃，人的寿命不会是200岁等），另一方面是数据字典给出的取值范围，此外，还可以通过数据的标准差来判断，例如，认为两个标准差以外的数据需要特别进行观察。

``` python
mask = np.abs(df['Data']-df['Data'].mean())>=(2*df['Data'].std())  # 获取异常值标签
df[mask] = df.mean()  # 用均值填充
```



#### 描述性统计

假设所有的缺失数据、重复数据、异常值都已经处理完毕，那么数据分析的下一步工作就是对数据进行描述性统计了。

描述性统计可以对数据分布有更深入的了解。

```python
# DataFrame.describe()  对数值类型输出均值信息,对于其他类型,输出计数、不同值个数、频率等
# 通过include参数调用

retail_data.describe(include=[np.number])
retail_data.describe(include=[np.object])
```



对基本统计信息有了一定了解后，通常数据分析人员还会查看一定数量的最大值、最小值。

```python
# nlargest与nsmallest返回前n个最大值或最小值
retail_data.nlargest(50, 'TotalPrice')

# 更常用的是通过sort_values()排序的方式得到
retail_data.sort_values('TotalPrice', ascending=False)  # 降序

# 可以采用多列排序
retail_data.sort_values(['TotalPrice', 'Country'], ascending=False)  # 先排序总价,总价相同才考虑国家
```

