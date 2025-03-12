# 经典机器学习

* 算法：线性/逻辑回归、决策树、随机森林、SVM支持向量机、隐马尔可夫模型、KNN(K近邻)、K-均值
  * 可学的：随机森林、Gradient Boosting Machines (GBM)、逻辑回归
    * 如果不进行超参调节，可能随机森林比GBM好。随机森林只要调max_depth, n_estimators, class_weight。GBM峰值性能好，但要调的参数多，也相对容易过拟合
  * 不学的
    * 支持向量机SVM：分类和回归都不行。需要选择核函数，如果选得好则有效，可以处理高维数据和非线性可分数据，但对于大量数据不行
    * 朴素贝叶斯分类器(Naive Bayes)：用于文本分类、垃圾邮件过滤。它假设特征之间相互独立，现实世界中往往不是
    * 线性回归和逻辑回归(Logistic Regression)：深度学习的基础，可理解为单层的神经网络。对于非线性可分(linearly separable)数据不行。在推荐系统，CTR点击率预估领域，还是以LR为主
    * 隐马尔科模型HLM：文本任务，用深度学习
    * KNN（K临近）：算法简单，但当数据量大时计算花费大。替代用局部敏感哈希LSH和树结构如KD树
    * Principal Component Analysis主成分分析：用于降维有点用
* 任务：分类classification（一般有监督，目标是预测新的数据的标签）、回归regression、聚类clustering（一般无监督，就是区分一批数据）、降维、模型选择、预处理
* 适合表格型数据
* gradient-boosting梯度提升：XGBoost、LightGBM、CatBoost、AdaBoost https://neptune.ai/blog/when-to-choose-catboost-over-xgboost-or-lightgbm https://www.kaggle.com/code/faressayah/xgboost-vs-lightgbm-vs-catboost-vs-adaboost
* 自动机器学习，自动调超参数：https://github.com/automl/auto-sklearn 不活跃，不支持sklearn 1.0，见 #1371
  * https://github.com/nidhaloff/igel
* CNN：用于图像和视频分析任务。RNN：顺序数据分析任务，自然语言处理、语音识别、时间序列分析。GAN：生成与给定数据集相似的新数据样本，用于图像合成、风格迁移和数据增强等任务。Transformer网络：自然语言处理
* 训练集(train)、验证集(verify)、测试集(test)：训练用于拟合模型。验证用于调整超参数，一般由训练集划分出来；调超参实际上也是一种拟合，逐渐由无偏估计变为有偏。测试用于最终模型的无偏评估
* early stop：训练过程中，训练集的loss不断下降，但验证的loss上升，说明模型过拟合。此时停止训练，选择最好的模型

## pandas

* https://pandas.pydata.org/docs/
* axis=0指行，1指列。许多函数有index和column的命名参数，优先用这个，除非想应用于所有
* Series具有广播特性：赋单个值就全变成该值，赋list/range/Series就依次改变。与比较运算符计算会产生值都为bool的Series，与另一个Series运算就依次处理，不会变成两列的df。但是不支持'xxx' in S1，要自己用map。长度不可变
* 如果是自动生成的行名，第0列不为行名
* 有许多处理时间序列的方法
* 其他项目：
  * modin 作为pd的原地替代库，速度更快资源消耗更小。类似的还有swifter pandarallel Dask Ray Vaex

```py
import pandas as pd
读取：
data = pd.read_csv('data.csv', index_col=0 指定第一列为行id, header=None若第一行不是列名, parse_dates=True, error_bad_lines=False遇到错误跳过但不包括空值) 编码默认u8，支持网络url
read_excel('file.xlsx', sheet_name=默认0支持str) / json / sql / sql_table / sql_query(sql语句, con)
保存：
df.to_csv('data.csv.gz', index=False)
to_sql('table', con=c, if_exists='默认fail可选append和replace') / to_markdown(tablefmt="pipe") / to_pickle/feather 二进制格式
df.to_numpy()  返回底层的np.array，是view。不推荐df.values
df.copy(deep=True)

创建：
pd.DataFrame({'A':[1,2], 'B':[3,4]})  两列AB，两行，第0行数据是13。指定行名：index=['row1', 'row2']
pd.DataFrame([ [1,3], [2,4] ], columns=['A','B'])  另一种创建方式，按行输入数据
pd.Series([1,3], index=['A','B'])  一行数据，AB是列名；但也可看作一列数据，AB是行id。也可传dict
pd.Series([1,2], name='A')  一列数据，A是列名，行id自动编号
columns = pd.MultiIndex.from_product([ ['one','two'], ['first','second'] ])  产生one下的两个和two下的两个，使用时用.loc[:, ('one', 'second')]
rows = pd.MultiIndex.from_tuples([ ('A', 1), ('A', 2), ('B', 1) ])

基本信息：
df.columns / index
df.shape  (行数, 列数)
df.size  元素总数
df.dtypes  各列数据类型
df.head(n=5)/tail()  显示最开始/后面的几行
df.info()  所有列的 名称 类型 占用空间，可选memory_usage='deep'
pd.set_option("display.max.columns", None)  列过多时不隐藏

选取(view)：
df.A/df['A']  选取一列，保留行名，再用[]能取出指定行的值
df[['A','B']]  选取多列，仍为DataFrame
df[0:2]/[1:]  选取一定范围的行，一定要是slice；可被iloc完全替代；仍为DataFrame，即使结果只有一行
df.iloc[0] / [[0,2]] / [1:] / [:,0]  第一个索引是行范围，用:选择所有行；第二个索引选择列。单索引时类型为Series，且index变为原columns的内容因此可用.A
  df.A.idxmax() 返回A里最大的那一行的index
df.loc  闭区间，一般不用数字访问
  基于标签的范围选择，条件过滤：df.loc[df.A > 5]、loc[df.Sex=='male', 'Height'] 取出所有男性的身高
    实际上是广播运算，返回一个全为True/False的长度相同的Series，又称为mask
    更多逻辑运算：(df.A == 'xxx') & (df.B > 10) 逻辑或用|，一定要加括号。A.isin([x,y])、A.isna()/notna()、any
    字符串方式：df.query('A > 5 and B < 10') 支持用`@val`访问变量，如果列名有空格用反引号
  非数字的index范围（假设为Series）：loc['A':'C'] 代替 loc['A','B','C']
df.iat[索引]/at[标签]：取单个值性能更高
根据标签的子串或正则选取：df.filter(like或regex, axis=默认1)
迭代：
  按行迭代整个df，返回副本：for i, row in df.iterrows()。按列：for col, series in df.items()
  for val in df.A

运算：基本都是非原地的，有些可指定inplace=True
df.A + df.B、df.A - df.B.mean()  广播运算
df.add(other, fill_value=0)
df.eval('C = A + B')
数据转换：
df.where(df > 0, other=0)  替换不满足条件的值。反向操作，替换满足条件的：mask(df < 0, other=0)
df.map(lambda 单个元素) / map({映射})  如果只想处理某一列就先取列。以前的applymap废弃了
df.apply(lambda row_series: ..., axis=1)  用于整个df，默认axis=0却是每次取一列，=1才是每次取一行
df.A.str.strip()  广播调用字符串函数
  split()有个expand=True拆分为多列
  contains('pattern', 默认regex=True)
  extract('reg_capture_pattern') 将捕获组拆分成多列
  replace(r'\s+', ' ', regex=True)
  cat(sep=',')
df.astype({'A': 'int32', 'B': 'float64'})、df.A.astype('string')  类型转换。另有pd.to_numeric(df.A)转换成float64或int64
df.replace({'A': {'old': 'new'}})
df.clip(lower=0, upper=100)  限制值范围
df.round(2)  四舍五入
将字符串代表的类别转换为序数：A.astype('category').cat.codes。哑编码(OneHot)：pd.get_dummies(data)；处理训练数据中不存在但生产中有的类别：配合pd.Categorical
合并：
pd.concat([df1, df2, rows], ignore_index=True)  合并行，ignore_index重置索引。设定axis=1变为合并列。之前的append废弃了
df.merge和join() on='列'  类数据库join，根据文档merge默认inner，suffixes=('_1','_2')指定重复列名后缀；join默认left
df1.combine_first(df2)  用df2填充df1的缺失值。update(df2)：用df2的非空值覆盖
重复行、空行：
df.dropna()  删除包含空值的行；只处理全为空的：how='all'
df.fillna(x)  用x填充空值，或method='ffill或bfill'用前后值填充。df.interpolate(method=默认'linear') 插值填充
df.drop_duplicates(keep=默认'first')
其他修改：
df.drop([xxx])  删除行；删列用axis=1或column=，还能用del且是原地的
df.columns = ['new_A', 'new_B']  重命名列。按映射重命名：df.rename(columns={'old': 'new'}, index=...)
df.set_index('col', verify_integrity=True检查重复值)  去掉数字index，将某一列改为index，原列里的值变为index的值。如果只想普通的重建数字：reset_index(drop=True)
df.stack(future_stack=True)  将df“压缩”成Series，原来的行列变成多维索引

统计：在df上调用聚合函数基本是返回以列名为index的Series
df.describe()  以8*n的表格显示各列的count、平均值、最大最小值等。默认只显示数字列，改变：include='all'
平均值mean() 中位数median() 分位数quantile([0.25, 0.75]) 标准差std() 方差var() 和sum() 累计和cumsum() 累计积cumprod() 众数mode()
df.A.value_counts()  某列的唯一值及其出现次数，相当于groupby再size()或再.A.count()，再从大到小排序。显示百分比而非次数：normalize=True。dropna默认True
df.agg(['max', 'min'])  对每一列都调用对应的函数，产生以max和min为index的聚合结果；Series也适用。不同列使用不同聚合：agg({'A':'max',...})
df.A.unique()唯一值列表、nunique()唯一值数量
df.rolling(window=3).mean()  滑动窗口
df.nlargest(5, 'A')  最大的n行。sample() 随机抽样
df.corr()  皮尔逊相关系数。假定前提：两个变量之间是线性关系，或正态分布

排序和分组：
df.sort_values('A', ascending=默认True) 多列：(['A','B'], ascending=[True,False])。na_position默认'last'，kind默认快排，改为稳定排序用'stable'
df.groupby(['A']).B.max()  按A分组后把对应范围的B聚合，产生以A为index的Series
  groupby多个列时，会产生MultiIndex，一般用reset_index()去掉命名变成编号
  groupby后的结果可看作含有df的Series，可.apply(lambda df: ...)，但必须返回一行或一个值，即需要聚合
  .filter(lambda x: len(x) > 2)、.transform(lambda x: x - x.mean())、.sort_index()
  高级操作：groupby('A').agg({ 'B':{'B_max':'max','B_min':'min'} }).flatten_names()、滚动计算.rolling(window=3).mean()、累计计算.expanding().sum()
df.pivot_table(index='col1',columns='col2',values=['col3','col4'],aggfunc='max'或{不同值对应的处理方式},margins=True汇总,fill_value=0)  数据透视表，以col1为行，col2为列，取col3和col4的最大值，聚合后还空的值填0

画图：要开%matplotlib inline
df.plot(x='xxx',y=[...])  默认折线图，kind='bar'改为其他图。.bar(stacked=True)堆叠条形图，scatter(x='A', y='B', c='C')，hist(bins=50)
```

## matplotlib

* line折线图，stack面积图，hist直方图，pie饼图，scatter散点图，bar柱状图，box箱形图
  * 柱状图和直方图的区别：柱状图的x轴是分类，不同类可以随意交换；y可以用不同颜色横向分出更多类，值表示汇总；另一种方式：每个x具有多个柱。而直方图的x轴是数据的范围，且矩形条之间没有间隙
  * 面积图：一种与折线图类似只是下方染色，另一种类似于地图占满整个图形

```py
%matplotlib inline
import matplotlib.pyplot as plt

plt.plot([xpoints], [ypoints], label='折线名')
plt.title()
plt.ylabel('Y轴名称'); plt.xlabel()

plt.show() # 终端里也能用，但会显示在窗口中
plt.savefig('img.svg' / bytesio) # 格式自动根据文件名的后缀设置

# 在一个figure中绘制多个图
fig, axs = plt.subplots(n) # 单参数为一列n行，(1, 2)为1行2列，行列都大于1时返回二维数组
axs[0].plot...

plt.tight_layout()

subplot2grid

fig = plt.figure()
subplot1 = fig.add_subplot(121)
subplot1.plot(x,y)
subplot2 = fig.add_subplot(122)
subplot2.plot(y,x)
fig.tight_layout()
```

解决英文系统上不显示中文的问题。实测无法用Noto Sans CJK SC(fonts-noto-cjk)。

```py
! apt-get install fonts-wqy-microhei -qq && rm -rf ~/.cache/matplotlib && fc-list :lang=zh family
from matplotlib import rcParams, font_manager
font_manager.fontManager.addfont('/usr/share/fonts/truetype/wqy/wqy-microhei.ttc')
font_manager.findfont('WenQuanYi Micro Hei', fallback_to_default=False)
rcParams['font.family']=['WenQuanYi Micro Hei']
#rcParams['axes.unicode_minus'] = False 已知wqy不需要此项，不知雅黑是否需要
```

其他可视化库：Seaborn(基于matplotlib，用起来更简单，但只支持2D) bokeh plotly功能最多可以画地图 plotly/dash(基于plotly.js，用于构建网页) altair Plotnine pyecharts

## scikit-learn

* 教程
  * 中文文档：https://scikit-learn.org.cn/lists/8.html 教程 https://scikit-learn.org.cn/lists/2.html 用户指南 https://sklearn.apachecn.org 另一个中文站
  * https://zhuanlan.zhihu.com/p/88729124 https://zhuanlan.zhihu.com/p/103136609 https://zhuanlan.zhihu.com/p/99618155 https://zhuanlan.zhihu.com/p/29649128 https://zhuanlan.zhihu.com/p/190049765
* 树的层数太浅会导致underfitting，无论是训练还是验证都具有较大误差；层数太多会导致overfitting，能非常好的匹配训练，但验证却有很大误差；应处于中间，一种控制方法是创建model时设定max_leaf_nodes，另一种仅解决of的方法是指定regularization
* 损失函数：衡量模型的预测值与sample真实值的区别，一般就是相减再平方
* 另一种评判好坏的方法：bias(偏差)和variance(方差)，两者形成4种组合。高bias为离目标远，低bias为离目标近，高variance为分散，低variance为集中。低bias+高variance为overfitting，高bias+低variance为underfitting
* 缺点：无法完全准确、难以纠正错误（一般只能改数据，即使调参，也难以评估是否会对正确的部分产生影响）、难以解释原理（尤其是神经网络）
* 决策树(DecisionTree)：xgboost.XGBRegressor，实测不调任何参数时与RF随机森林差不多；后来出了hist版，减少了内存占用。微软出了LightGBM，原理类似hist版的XGB，但内存占用更小，速度更快，效果也不错。这类模型(GBDT)不需要归一化
* 归一化：概率模型（树形模型）不需要归一化，因为它们不关心变量的值，而是关心变量的分布和变量之间的条件概率，如决策树、RF。而像Adaboost、SVM、LR、Knn、KMeans之类的最优化问题就需要归一化。sklearn.preprocessing.StandardScaler().fit(X_train).transform(X_train) 应只在训练集上放缩
* pipeline：把pre-processors和estimators连起来自动依次使用
* 序列化持久保存：pickle、joblib.dump(m, 'filename')第三方二进制序列化库内部基于pickle格式加载时会用mmap、treelite编译决策树的库
* 其他库：yellowbrick图形化，mlxtend工具类，dtreeviz可视化，scikit-optimize，m2cgen把模型转换为其它语言，featuretools，Sacred能保存各种参数用于复现

```py
y = data.Price # 选择一个列作为预测目标target。小数则为回归，整数或其它离散量则为分类，无监督学习不需要
X = data[['col1','col2']] # 选择一些列作为“features”。另一种选择方式：去掉不要的drop(columns=['Price'])。如[[1,2,3],[4,5,6]]表示2个sample，3个feature

from sklearn.model_selection import train_test_split # 把源数据分成 训练 和 测试 两部分，此处0.1表示10%
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.1, random_state=42)

from sklearn.ensemble import RandomForestRegressor/RandomForestClassifier # 比单个决策树更精确且无需调整叶子参数，基本可以无脑替换普通决策树
model = RandomForestRegressor(random_state=0) # 设定random_state使得每次运行结果一样
model.fit(X_train, y_train) # 填充数据
val_predicted_prices = model.predict(X_test) # 预测结果，返回类型是np.ndarray

from sklearn.metrics import mean_absolute_error
mean_absolute_error(y_test, val_predicted_prices)
model.score(X_test, y_test) # accuracy


# 填充空值，用已有的数据模拟，当空值较少时可以用；如果较多，应drop那一列
from sklearn.impute import SimpleImputer 
imputed_X_train = pd.DataFrame(imputer.fit_transform(X_train))
imputed_X_valid = pd.DataFrame(imputer.transform(X_valid))
imputed_X_train.columns = X_train.columns; imputed_X_valid.columns = X_valid.columns # Imputation会移除列名，此操作加回去

pd.get_dummies(data) # 自动把数据中非整数的离散值变成整数
data.select_dtypes(exclude=['object']) # 直接去掉非数字列
object_cols = [col for col in X_train.columns if X_train[col].dtype == "object"] # 提取非数字列的列名
from sklearn.preprocessing import OrdinalEncoder # 也是把非数字编码为数字，但可能存在X_train和X_valid里有不同值的情形，此时要drop掉差异部分，太复杂略。还有一种OneHotEncoder，转换后toarray()
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

## gradio

```py
gradio xxx.py 会自动reload
import gradio as gr

app = gr.Interface(fn=处理函数, inputs=[输入控件], outputs=[输出控件])
app.launch()

每个输入控件对应fn的参数
控件有对应的literal便捷使用默认样式，如'text'等价于gr.Textbox()
Audio和Video和Image可以网页采集或上传文件，后端接收默认是内存中的numpy数组，可改为服务器磁盘上的临时文件路径；Image还可以是PIL.Image。File可用'binary'表示bytes
控件基本上都有label参数

输出除了那些控件外，还可以配合pandas画图，且具有交互性而不是单纯的图片；也接受matplotlib对象。
data = pd.DataFrame({'a':[1,2,3], 'b':[4,5,6]})
gr.BarPlot(data, x='a', y='b')

gr.Json(dict)、gr.HTML(value='xxx')、gr.Markdown()
会话属性：inputs加gr.State(初始值)，fn处理后返回给outputs，fn里对它的使用就好像使用它包裹的对象

launch：
share=True 启动免费内网穿透
server_name 默认127.0.0.1。端口默认7860
auth=("username", "passwd")
title、description
allow_flagging='never' 默认有一个在服务器上保存输出内容的Flag按钮用于报告错误

交互基本都要靠事件触发回调

自定义界面：
with gr.Blocks() as app:
    c = gr.控件。默认竖向布局
    with gr.Row():
        横向布局

    btn.click(fn,inputs=[前面布局创建的控件变量],outputs)
with gr.Accordion('Advanced options', open=False): 相当于html的details

聊天机器人：
bot=gr.Chatbot(height=240)
msg=gr.Textbox()
def respond(msg, bot):
    for (usermsg, botmsg) in bot: 处理每一条历史纪录
    msg是当前输入，就是文本框控件的值。与历史记录一起送给模型
    bot.append((msg, ret)) bot是引用对象，往里面添加内容
    return '', bot 第一个返回值用于清空输入框，第二个将聊天内容输出
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
* 其它项目：pynecone

## milvus向量数据库

* sdk版本与服务端版本具有严格对应关系，必须看发行文档
* 其他向量数据库收集：https://cookbook.openai.com/examples/vector_databases/readme

```py
# 连接
from pymilvus import MilvusClient
client = MilvusClient(uri='http://host:port', token='user:passwd') # 可指定本地文件会自动创建，又称MilvusLite，有工具将数据导出方便迁移到独立版
# 也有一种数据库的概念，里面放集合，支持RBAC多租户。connections.connect()连接服务器，默认db_name='default'。但是似乎具有某些全局状态，感觉不太好

# Collections：一个Collection中的所有向量嵌入具有相同的维度和距离度量相似性
client.create_collection(collection_name="demo", dimension=384)
# 主键和向量字段使用默认名称（"id "和 "vector"），默认不自动递增。指定类型：MilvusClient.create_schema; schema.add_field
# list_collections()、has、drop、describe
# 有“加载”的概念，加载后读取到了内存中，之后不用了要释放。理论上应该是加载到服务器的内存里
# Collection.construct_from_dataframe

# 插入
doc = ['aaa', 'bbb']
vec = [[ np.random.uniform(-1, 1) for _ in range(384) ] for _ in range(len(docs)) ] # 演示用，随机生成embedding
data = [ {"id": i, "vector": vectors[i], "text": docs[i], "subject": "history"} for i in range(len(vectors)) ]
client.insert(collection_name="demo_collection", data=data)

# 搜索
res = client.search(
    collection_name="demo_collection",
    data=[vectors[0]], # query_vectors 按向量查询
    filter="subject == 'biology'", # 排除标量字段。默认无索引
    limit=2,
    output_fields=["text", "subject"], # 不加则默认只有id和distance
)
client.query( # 按标量查询。delete类似
    collection_name="demo_collection",
    filter="subject == 'history'", # 符合的。还可按ids=[0, 2]
    output_fields=["text", "subject"],
)
```

## NLP

* https://github.com/explosion/spaCy
* NLTK
* Gensim：实现了常用的主题模型，文档相似度计算
* CoreNLP：Java的
* 中文：哈工大LTP、中科院大学NLPIR、清华THULAC、北大pkuseg
* https://github.com/microsoft/nlp-recipes NLP Best Practices，不维护了
* https://github.com/hankcs/HanLP

## 声音

* https://github.com/babysor/MockingBird

## 其他项目

* https://www.ray.io/ 分布式

## 书签

```
https://www.youtube.com/watch?v=bmmQA8A-yUA
https://i.am.ai/roadmap/
https://github.com/microsoft/AI-For-Beginners
https://github.com/microsoft/ML-For-Beginners/blob/main/translations/README.zh-cn.md
https://microsoft.github.io/ai-edu/ 中文教程

pytorch和深度学习:
https://pytorch.org/get-started/ ；https://pytorch.apachecn.org/ 中文文档
用于语音、图像、文本(垃圾邮件)的识别、分类和预测(推荐系统)。容忍误差，有明确的输入和输出，有大量的数据集且不随时间快速变化（否则就要重新训练模型）。
https://mlelarge.github.io/dataflowr-web/ https://mlelarge.github.io/dataflowr-web/cea_edf_inria.html
https://github.com/amusi/PyTorch-From-Zero-To-One
https://zhuanlan.zhihu.com/p/66543791 60分钟快速入门 PyTorch
https://zhuanlan.zhihu.com/p/99318332 60题PyTorch简易入门指南
https://zhuanlan.zhihu.com/p/87263048
https://www.zhihu.com/question/55720139
https://zhuanlan.zhihu.com/c_1176098426973106176
https://github.com/PyTorchLightning/pytorch-lightning https://zhuanlan.zhihu.com/p/120331610 https://zhuanlan.zhihu.com/p/134291726
https://course.fast.ai/
https://www.zhihu.com/question/384519338
https://www.zhihu.com/question/388079431
https://tangshusen.me/Deep-Learning-with-PyTorch-Chinese/#/ 一本书
https://github.com/madewithml/basics
https://github.com/MLEveryday/100-Days-Of-ML-Code
https://github.com/scutan90/DeepLearning-500-questions
https://www.zhihu.com/question/375537442
https://zhuanlan.zhihu.com/p/30011154
https://github.com/explosion/thinc
https://github.com/awesomedata/awesome-public-datasets 各种数据源
https://github.com/recommenders-team/recommenders Best Practices on Recommendation Systems
https://github.com/AMAI-GmbH/AI-Expert-Roadmap
https://github.com/MorvanZhou/PyTorch-Tutorial
https://github.com/ShusenTang/Dive-into-DL-PyTorch
https://zhuanlan.zhihu.com/p/479795186
https://github.com/datawhalechina/thorough-pytorch
https://github.com/openxla/xla 加速编译的
https://github.com/d2l-ai/d2l-zh 动手学深度学习
https://github.com/lutzroeder/netron 神经网络可视化
https://github.com/Tencent/ncnn 前向推理框架
https://github.com/tinygrad/tinygrad

机器学习：
https://github.com/rasbt/python-machine-learning-book-3rd-edition 据说很简单
入门：https://zhuanlan.zhihu.com/p/24339995 https://zhuanlan.zhihu.com/p/29704017 https://www.zhihu.com/question/55949025
https://github.com/Yorko/mlcourse.ai
https://github.com/instillai/machine-learning-course
https://github.com/apachecn/AiLearning 中文
https://github.com/ML-course/master
https://github.com/datawhalechina/pumpkin-book
https://github.com/Jack-Cherish/Machine-Learning
polyaxon 机器学习平台
https://github.com/aialgorithm/Blog
https://github.com/ethen8181/machine-learning

机器学习的课程涉及了很多数学、统计概率、以及优化方向的知识，大概包括：
* 线性代数：矩阵/张量乘法、求逆，奇异值分解/特征值分解，行列式，范数等
* 统计与概率：概率分布，独立性与贝叶斯，最大似然(MLE)和最大后验估计(MAP)等
* 信息论：基尼系数，熵(Entropy)等
* 优化：线性优化，非线性优化(凸优化/非凸优化)以及其衍生的求解方法如梯度下降、牛顿法、基因算法和模拟退火等
* 数值计算：上溢与下溢，平滑处理，计算稳定性(如矩阵求逆过程)
* 微积分：偏微分，链式法则，矩阵求导等

numpy:
https://www.bilibili.com/video/BV19T4y127Z2
https://zhuanlan.zhihu.com/p/27624814
https://zhuanlan.zhihu.com/p/73785485
https://zhuanlan.zhihu.com/p/76186124
https://zhuanlan.zhihu.com/p/81815234
https://cs231n.github.io/python-numpy-tutorial/
https://mp.weixin.qq.com/s?__biz=MzIxMjM4MjkwMw==&mid=2247483920&idx=1&sn=96b11616cf48c83f54ac76c6687a20af
https://zhuanlan.zhihu.com/p/32242331
https://dafriedman97.github.io/mlbook/content/introduction.html


pandas:
https://www.zhihu.com/question/289788451
https://zhuanlan.zhihu.com/p/43018099
https://blog.csdn.net/matrix_laboratory/article/details/50704160
https://mp.weixin.qq.com/s?__biz=MzUwOTg0MjczNw==&mid=2247493035&idx=1&sn=c916f32b29555ac2acba839efeb205ee
NAN值的处理：https://mp.weixin.qq.com/s?__biz=MzUwOTg0MjczNw==&mid=2247485455&idx=1&sn=2107a2efb5aebd8797356b335e35196d
画图：https://mp.weixin.qq.com/s?__biz=MzUwOTg0MjczNw==&mid=2247491730&idx=1&sn=a01fe6c3292f8dd00f20cd890d2601d5

http://joyfulpandas.datawhale.club/Content/ch2.html
https://www.pypandas.cn/docs/getting_started/overview.html
https://github.com/hangsz/pandas-tutorial
https://pandas.liuzaoqi.com/intro.html
https://github.com/BrambleXu/pydata-notebook
https://www.machinelearningplus.com/python/101-pandas-exercises-python/ 汉化：https://mp.weixin.qq.com/s?__biz=MzUwOTg0MjczNw==&mid=2247490395&idx=1&sn=49215a3b51a6768802ba2eae3410e537 https://mp.weixin.qq.com/s?__biz=MzUwOTg0MjczNw==&mid=2247490736&idx=1&sn=69c1805418951ff115776730e5f9af54
https://github.com/guipsamora/pandas_exercises
https://realpython.com/learning-paths/pandas-data-science/
https://github.com/jvns/pandas-cookbook
https://realpython.com/learning-paths/pandas-data-science/

https://the-turing-way.netlify.app/ 数据科学的书，英文
https://jakevdp.github.io/PythonDataScienceHandbook/ numpy pandas Matplotlib sklearn
```
