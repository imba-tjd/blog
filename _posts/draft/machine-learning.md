# 经典机器学习

* 算法：线性/逻辑回归、决策树、随机森林、SVM支持向量机、隐马尔可夫模型、KNN(K近邻)、K-均值
* 任务：分类classification（一般有监督，目标是预测新的数据的标签）、回归regression、聚类clustering（一般无监督，就是区分一批数据）、降维、模型选择、预处理
* 适合表格型数据
* gradient-boosting梯度提升：XGBoost、LightGBM、CatBoost、AdaBoost https://neptune.ai/blog/when-to-choose-catboost-over-xgboost-or-lightgbm https://www.kaggle.com/code/faressayah/xgboost-vs-lightgbm-vs-catboost-vs-adaboost
* 自动机器学习
  * https://github.com/EpistasisLab/tpot2
  * https://github.com/automl/auto-sklearn 不更新了，还不支持sklearn 1.0。自动调超参数
  * https://github.com/microsoft/nni 不活跃，有中文文档
* nlp：https://github.com/explosion/spaCy NLTK

## pandas

* 大部分运算都是非原地的，且可指定inplace=True
* axis=0指行，1指列；许多函数有index和column的命名参数，优先用这个，除非想应用于所有
* Series具有广播特性：赋单个值就全变成该值，赋list/range/Series就依次改变。与比较运算符计算会产生值都为bool的Series，与另一个Series运算就依次处理，不会变成两列的df。但是不支持'xxx' in S1，要自己用map。长度不可变
* 如果是自动生成的行名，第0列不为行名
* pandas-bokeh：简单做出交互图
* modin：目的是作为pd的原地替代库，速度更快资源消耗更小。类似的库还有swifter pandarallel Dask Ray Vaex

```py
import pandas as pd
data = pd.read_csv('data.csv', index_col=0 指定第一列为行id, header=None若第一行不是列名, parse_dates=True)/excel/json/sql/sql_table/sql_query(sql语句, con)，编码默认u8，支持网络url
df.to_xxx()保存，索引无意义时一般指定index=False，其中to_sql(table_name('xxx'),con=c)能直接保存到数据库连接中，to_markdown(tablefmt="pipe")，to_pickle/feather以二进制格式保存；指定sep=None可自动检测分隔符，文件名以gz/xz/zst结尾可自动压缩保存且读取时自动识别
pd.DataFrame({'A': [1, 2], 'B': [3, 4]})  两列AB，两行，第0行数据是13；index=['row1', 'row2']指定行名
pd.DataFrame([[1,3], [2,4]], columns=['A', 'B'])  另一种创建方式，按行输入数据
pd.Series([1,3], index=['A','B'])  一行数据，AB是列名；但也可看作一列数据，AB是行id
pd.Series([1,2], name='A')  一列数据，A是列名，行id自动编号
columns=pd.MultiIndex.from_product([['one', 'two'], ['first', 'second']])  产生one下的两个和two下的两个，使用时用.loc[:, ('one', 'second')]
df.columns/index
df.shape行列数
df.head(n=5)/tail()  显示最开始/后面的几行
df.describe()  以8*n的表格显示各列的count、平均值、最大最小值等，只显示数字列
df.info()  显示所有列的名称类型占用空间

df.A/df['A']  获取一列，保留行名，再用[]能取出指定行的值；后一种方式适用于列名含空格
df.[['A','B']]  获取多列，仍为DataFrame，不能用小括号
df[0:2]/[1:]  获取一定范围的行，一定要是slice；可被iloc完全替代；仍为DataFrame，即使只有一行
df.iloc[0] / [1:3,0] / [:,0] / [(0,1,2),0]  第一个索引是行范围，用:就是选择所有行，第二个索引是选择列；单索引时类型为Series，且index变为原columns的内容因此可用.A
df.loc  若按数字索引访问，与iloc相比为闭区间。实际用于基于名称的范围选择，以及支持非数字的index范围：loc['A':'C']代替loc['A','B','C']。df.loc[df.Sex=='male', 'Height']取出所有男性的身高
df.A == 'xxx'  广播运算，返回一个全为True/False的长度相同的Series。还有A.isin([x,y])、A.notnull()、(...).any(...)进一步过滤。(data.A == 'xxx') & (data.B > 10) 逻辑或用|，一定要加括号
df.A + df.B;  df.A - df.B.mean();  df.A = Iterable对象
df.at/iat：取单个值性能更高

df1.append(df2)  合并行
pd.merge([df1, df2], on='key')  合并列
df1.join(df2, on='key')
pd.concat([rows1, rows2])  合并行，设定axis=1变为合并列
df.assign(new_col=df.A+df.B)  添加列

df.A.value_counts()  计算某列的唯一值及其出现次数，相当于groupby再size()或再.A.count()，再从大到小排序
df.sort_values(by = 'A')  默认ascending=True，by可以是[]；还有sort_index()在groupby后可能用到
df.A.str.xxx  把数据看作字符串广播使用对应的函数；splite()有个expand=True
df.A.unique()、isnull()/notnull()、max()、idxmax()返回最大值的index常见于loc中以获取那一行
df.A.map(lambda)
df.apply(lambda row: ..., axis=1)  用于整个df，默认按列，设定axis=1后按行，类型是Series，用.A可获取列的值。df.applymap处理单个元素
df.agg([max, min])  对每一列都调用对应的函数，产生以max和min为index的聚合结果；Series也适用
df.sum()  返回以列名为index的Series

df.filter(regex='^L')
df.rename(columns={'old': 'new'}, index=...)  也可df.index = [...]
df.dropna() 删除包含空值的行，设置how='all'只处理全为空的；fillna(x) 用x填充空值，drop_duplicates() 删除重复值
df.drop([xxx])  删除行；删列还能用del或axis=1，且是原地的
df.set_index(['col1','col2'], verify_integrity=True)
df.stack().rename_axis
df.index.set_levels(frame.index.levels[-1].astype(int), level=-1)

df.groupby(['A']).B.max()  按A分组后把对应范围的B聚合，产生以A为index的Series
groupby多个列时，会产生MultiIndex，一般用reset_index()去掉命名变成编号
groupby后的结果可看作含有df的Series，可.apply(lambda df: ...)，但必须返回一行或一个值，即需要聚合
df.pivot_table(index=col1,columns=col2,values=[col3,col4],aggfunc=max)  数据透视表，以col1为行，col2为列，取col3和col4的最大值

pd.set_option("display.max.columns", None)  列过多时不隐藏
df.plot(x='xxx',y=[...])  默认是折线图，plot.pie()画其它图。还要开%matplotlib inline
pd.to_numeric
A.str好像对本来是int的会报错
```

## matplotlib

* line折线图，hist直方图，pie饼图，scatter散点图，bar柱状图
  * 柱状图和直方图的区别：柱状图的x轴是分类，不同类可以随意交换；而直方图的x轴是数据的范围，且矩形条之间没有间隙

```py
%matplotlib inline
import matplotlib.pyplot as plt

plt.plot([xpoints], [ypoints], label='折线名')
plt.title()
plt.ylabel('Y轴名称'); plt.xlabel()
plt.rcParams['font.sans-serif']=['SimHei'] # 解决不显示中文
plt.show() # 终端里也能用，但会显示在窗口中
plt.savefig('img.svg') # 格式自动根据文件名的后缀设置

fig, ax = plt.subplots() # 在一个figure中绘制多个图
```

## scikit-learn

* https://scikit-learn.org.cn/lists/8.html https://scikit-learn.org.cn/lists/2.html https://sklearn.apachecn.org。https://zhuanlan.zhihu.com/p/88729124 https://zhuanlan.zhihu.com/p/103136609 https://zhuanlan.zhihu.com/p/99618155 https://zhuanlan.zhihu.com/p/29649128 https://zhuanlan.zhihu.com/p/190049765
* 树的层数太浅会导致underfitting，无论是训练还是验证都具有较大误差；层数太多会导致overfitting，能非常好的匹配训练，但验证却有很大误差；应处于中间，一种控制方法是创建model时设定max_leaf_nodes，另一种仅解决of的方法是指定regularization
* 损失函数：衡量模型的预测值与sample真实值的区别，一般就是相减再平方
* 另一种评判好坏的方法：bias(偏差)和variance(方差)，两者形成4种组合。高bias为离目标远，低bias为离目标近，高variance为分散，低variance为集中。低bias+高variance为overfitting，高bias+低variance为underfitting
* 缺点：无法完全准确、难以纠正错误（一般只能改数据，即使调参，也难以评估是否会对正确的部分产生影响）、难以解释原理（尤其是神经网络）
* 决策树：xgboost.XGBRegressor，实测不调任何参数时与RF随机森林差不多；后来出了hist版，减少了内存占用。微软出了LightGBM，原理类似hist版的XGB，但内存占用更小，速度更快，效果也不错。这类模型(GBDT)不需要归一化
* 归一化：概率模型（树形模型）不需要归一化，因为它们不关心变量的值，而是关心变量的分布和变量之间的条件概率，如决策树、RF。而像Adaboost、SVM、LR、Knn、KMeans之类的最优化问题就需要归一化。sklearn.preprocessing.StandardScaler().fit(X).transform(X)
* pipeline：把pre-processors和estimators连起来自动依次使用
* sklearn.tree.DecisionTreeRegressor：需要调参
* 部署：pickle、joblib.dump第三方二进制序列化库、treelite编译决策树的库
* 其他库：yellowbrick图形化，mlxtend工具类，dtreeviz可视化，scikit-optimize，m2cgen把模型转换为其它语言，featuretools，Sacred能保存各种参数用于复现

```py
y = data.Price # 选择一个列作为预测目标target。小数则为回归，整数或其它离散量则为分类，无监督学习不需要
X = data[['col1','col2']] # 选择一些列作为“features”。也支持用普通list，如[[1,2,3],[4,5,6]]表示2个sample，3个feature

from sklearn.model_selection import train_test_split # 把源数据分成训练的和验证的两部分，此处测试的占10%
train_X, val_X, train_y, val_y = train_test_split(X, y, test_size=0.1, random_state=0)

from sklearn.ensemble import RandomForestRegressor/RandomForestClassifier # 比单个决策树更精确且无需调整叶子参数，基本可以无脑替换普通决策树
dt_model = RandomForestRegressor(random_state=0) # 设定random_state使得每次运行结果一样
dt_model.fit(train_X, train_y) # 填充数据
val_predicted_prices = dt_model.predict(val_X) # 预测结果，类型是numpy.ndarray

from sklearn.metrics import mean_absolute_error, accuracy_score # MAE平均绝对误差，等于avg(abs(预测值-真实值))
mean_absolute_error(val_y, val_predicted_prices)

from sklearn.impute import SimpleImputer # 填充空值，用已有的数据模拟，当空值较少时可以使用，如果较多，应drop那一列
imputed_X_train = pd.DataFrame(imputer.fit_transform(X_train))
imputed_X_valid = pd.DataFrame(imputer.transform(X_valid))
imputed_X_train.columns = X_train.columns; imputed_X_valid.columns = X_valid.columns # Imputation会移除列名，此操作加回去

pd.get_dummies(data) # 自动把数据中非整数的离散值变成整数
data.select_dtypes(exclude=['object']) # 直接去掉非数字列
object_cols = [col for col in X_train.columns if X_train[col].dtype == "object"] # 提取非数字列的列名
from sklearn.preprocessing import OrdinalEncoder # 也是把非数字编码为数字，但可能存在X_train和X_valid里有不同值的情形，此时要drop掉差异部分，太复杂略。还有一种OneHotEncoder用起来太复杂了
label_X_train[object_cols] = ordinal_encoder.fit_transform(X_train[object_cols])
label_X_valid[object_cols] = ordinal_encoder.transform(X_valid[object_cols])

from sklearn.datasets import load_iris # 内置了一些数据集，小型的自带，大型的使用时会联网下
X, y = load_iris(return_X_y=True)

from sklearn.model_selection import RandomizedSearchCV # 自动调整超参数
param_distributions = {'n_estimators': randint(1, 5), 'max_depth': randint(5, 10)}
search = RandomizedSearchCV(estimator=RandomForestRegressor(random_state=0),n_iter=5,param_distributions=param_distributions,random_state=0)
search.fit(X_train, y_train)
search.best_params_

from sklearn.linear_model import LogisticRegression, LinearRegression
Input contains NaN, infinity or a value too large：存在空值
```

## BigQuery

```py
from google.cloud import bigquery
client = bigquery.Client()
dataset_ref = client.dataset("hacker_news", project="bigquery-public-data") # 描述要请求的内容
dataset = client.get_dataset(dataset_ref) # 进行请求获取数据
client.list_tables(dataset)

table_ref = dataset_ref.table("full")
table = client.get_table(table_ref)
table.schema
client.list_rows(table, max_results=5).to_dataframe() # 数据转df
```

## Streamlit

* 从纯py生成网页，主要是为机器学习设计的，自带托管平台
* 不支持32位，依赖pandas等一大堆
* 对于用户的每一次交互，整个脚本从头到尾执行一遍
* 中文文档：http://cw.hubwiz.com/card/c/streamlit-manual/
* streamlit run xxx.py/URL
* 其它项目：pynecone、gradio
