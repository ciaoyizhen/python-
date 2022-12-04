# 第三章 Pandas基础

#### 设置显示数据

```python
import pandas as pd
pd.set_option('display.max_rows', 7)  # 设置最多显示7行
pd.set_option('display.max_columns', 6)  # 设置最多显示6列
```



#### DataFrame信息查看

DataFrame.index的类型为RangeIndex。

DataFrame.columns的类型为Index。

DataFrame.values的类型为ndarray。

```
DataFrame.dtypes  # 查看列表中的数据类型
DataFrame.value_counts()  # 统计DataFrame中不同的值出现的次数

DataFrmae.dtypes.value_counts()  # 结合一下,可以得到DataFrame中不同的数据类型各有几列
```



#### 生成Series

从DataFrame中提取Series的方法：

1. retail_data[‘Country’]
2. retail_data.Country                                (不建议使用)



> Series数据，支持大部分python运算操作符（四则运算等）
>
> 在数据分析中，会大量利用bool值进行数据筛选

```python
# Series运算操作符实例

# 
retail_data['UnitPrice'] + 1
retail_data['UnitPrice'] * 2
retail_data['UnitPrice'] > 2  # 返回一列的True或False
retail_data['UnitPrice'] == 'France'  # 返回一列的True或False
```



#### 链式方法

链式方法就是通过‘.’操作的顺序来调用对象方法的方式。(略)



#### 缺失值处理

```
# isnull()方法
retail_data['UnitPrice'].isnull()  # 返回缺失值位置为True

# 通过sum方法可以获取缺失值的个数
retail_data['UnitPrice'].isnull().sum()  # 对每个True进行求和,即得到缺失值个数
retail_data['UnitPrice'].isnull().mean()  # 对整个数据取均值,即得到缺失值在总数据中的比例

# fillna()函数对缺失数据进行填充
retail_data['UnitPrice'].fillna(0)  # 用0填充缺失值
```





> **Pandas中的DataFrame与Seires的所有操作都是生成新的DataFrame与Series，要对原始数据采用修改，只要加上参数inplace=True即可**



#### 索引与列

```python
# 设置索引列的方法
pd.read_csv(file, index_col='列名')  # 通过index_col传列名或者传位置索引都可以

DataFrame.set_index('列名')  # 这个只能传列名  注意!!! 这里只有操作,没有改变原始,要考虑inplace或者赋值

DataFrame.reset_index()  # 索引恢复  若你用index_col读取进行的那列会消失!!!

# 获取位置索引的方法
DataFrame.columns.get_loc('列名')  # 即可获取列名对应的位置索引
```

#### 修改索引内的名字与列的名字

```python
# 使用DataFrame.rename()的方法可以实现索引名与列名的改变
# 将索引中的afg变更为Afghanistan, 将列名中的time变更为date
idx = {'afg':'Afghanistan'}
col = {'time':'date'}
data = gapminder.rename(index=idx, columns=col)
data.head()

# 此外可以通过转换为列表对象,将一个新列表传入来完成修改
index = gaminder.index
columns = gaminder.columns
index_list = index.tolist()
columns_list = columns.tolist()
index_list[0] = 'Afghanistan'
columns_list[0] = 'date'
gapminder.index = index_list
gapminder_index = columns_list
```



#### 添加/修改/删除列

```python
# 添加新的一列,名为'Total_price', 默认添加在最后一列
retail_data['Total_price'] = retail_data['UnitPrice'] * retail_data['Quantity']
retail_data[['Total_price', 'UnitPrice', 'Quantity']]

# 若要在中间增加列,采用insert()方法
totalPrice_index = retail_data.columns.get_loc('CustomerID') + 1  #  获取'CustomerID'后面一个位置索引
retail_data.insert(loc=totalPrice_index,
                  column='New_totalPrice',
                  value=retail_data['Total_price'])

# 若要删除某列,使用drop()方法
retail_data.drop('New_totalPrice', axis='columns')  # axis必须要传,告诉程序要删的是行还是列,可以使用位置索引

# 删除某列也可以直接使用 del
del retail_data['Total_price']  # 这种方法删除,会在原来的DataFrame上进行修改
```



#### 选择多列

```python
# 通过索引直接拿到对应的列, 若使用[[..]] 则返回DataFrame, [..]则为Series
retail_data[['Total_price', 'UnitPrice', 'Quantity']]

# 选择数据中对应的数据类型 select_dtypes(include=['数据类型'])
retail_data.select_dtypes(include=['object'])  # 模型参数就为include, 也可以不使用[], 当列表里有多个数据类型,则返回同时满足该数据类型的数据.

# 选择包含某个字段的列名
"""
filter三个参数(互斥,即只能使用一个):
like:返回包含该字符串的列名
regex='正则表达式'  # 使用正则表达式提取列名
items=['列名1', '列名2'] # 与之前[[]]不同的是,如果里面没有这项目不会报错
"""
retail_data.filter(like='Price')
retail_data.filter(regex='_')
retail_data.filter(items=['Total_price', 'UnitPrice', 'wrong'])


# 将列名进行分组处理
price_info = ['UnitPrice', 'Quantity', 'Total_price']
invoice_info = ['InvoiceNo', 'InvoiceDate', 'StockCode', 'Description']
customer_info = ['CustomerID', 'Country']
columns_new = price_info + invoice_info + customer_info

set(columns_new) == set(retail_data.columns)  # 先判断两个列名有没有遗漏

retail_data_new = retail_data[columns_new]  # 通过导入列名顺序 将列名顺序替换为新的DataFrame
```





#### 
