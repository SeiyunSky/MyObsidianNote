```python
#数据读取/写入
pd.read_csv()          # 读取CSV
pd.read_excel()        # 读取Excel
pd.read_sql()          # 读取SQL数据库
df.to_csv()            # 保存为CSV
df.to_excel()          # 保存为Excel

#数据查看与信息
df.head()              # 查看前n行
df.tail()              # 查看后n行
df.info()              # 数据概览（类型、内存等）
df.describe()          # 数值列统计描述
df.shape               # 数据维度（非方法，是属性）
df.columns             # 列名列表
df.dtypes              # 各列数据类型

#数据清洗
df.dropna()            # 删除缺失值
df.fillna()            # 填充缺失值
df.drop()              # 删除行/列
df.drop_duplicates()   # 删除重复行
df.replace()           # 值替换
df.rename()            # 重命名列

#数据选择与过滤
df[]                   # 列选择（df['列名']）
df.loc[]               # 标签索引
df.iloc[]              # 位置索引
df.query()             # 查询表达式
df.filter()            # 按列名过滤
df.isin()              # 是否在列表中

#数据排序与排名
df.sort_values()       # 按值排序
df.sort_index()         # 按索引排序
df.rank()              # 排名计算

#数据分组与聚合
df.groupby()           # 分组操作
df.agg()               # 多聚合函数
df.pivot_table()       # 数据透视表
df.crosstab()          # 交叉表

#数据变形与合并
pd.concat()            # 数据连接
pd.merge()             # 数据合并（类似SQL JOIN）
df.melt()              # 宽表转长表
df.pivot()             # 长表转宽表
df.stack()             # 列转行（堆叠）
df.unstack()           # 行转列（解堆叠）

#数值计算与统计
df.sum()               # 求和
df.mean()              # 平均值
df.median()            # 中位数
df.std()               # 标准差
df.min()/df.max()      # 最小/最大值
df.count()             # 非空计数
df.corr()              # 相关系数矩阵
df.cov()               # 协方差矩阵

#数据类型转换
df.astype()            # 类型转换
pd.to_numeric()        # 转换为数值
pd.to_datetime()       # 转换为日期时间
pd.get_dummies()       # One-Hot编码

#时间序列处理
pd.to_datetime()       # 转换为时间戳
df.resample()          # 重采样
df.shift()             # 数据偏移
df.rolling()           # 滚动窗口

#字符串处理
df.str.contains()      # 包含检测
df.str.replace()       # 字符串替换
df.str.split()         # 字符串分割
df.str.upper()/lower() # 大小写转换
```