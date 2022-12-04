# Seaborn补充

#### 风格设置

+ set(style=‘’)
+ set_style(‘’)
+ axes_style()  # 这个是针对当前子图的

支持的风格有五种：

+ darkgrid  默认
+ whitegrid
+ dark
+ white
+ ticks

#### 去掉边框线

sns.despine(bottom=True, right=True)   # 就是原本四条线封闭你图形现在不了。



## 常见图

参考链接：[(135条消息) Seaborn常见绘图总结_不会写作文的李华的博客-CSDN博客](https://blog.csdn.net/qq_40195360/article/details/86605860)

### 关系图

#### 