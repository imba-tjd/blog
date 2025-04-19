# 经典机器学习
    * 隐马尔科模型HLM：文本任务，用深度学习

* 数据挖掘任务：①预测：分类、回归。②描述：关联（相似度计算）、聚类、异常
* 缺点：无法完全准确、难以纠正错误（一般只能改数据，即使调参，也难以评估是否会对正确的部分产生影响）、难以解释原理（尤其是神经网络）

---

## 术语

* 均值 EX、μ、mean
* 方差 variance σ²：一种测量分散程度(variation)的方式，低方差为集中。DX=EX²-(EX)²。协方差：conv(X,Y)
* 标准差 Standard Deviation σ：又称均方差、均方根值（RMS），是方差的算术平方根。在统计学上，S是样本的标准差，σ是总体（真实）的标准差
* SS(mean) - sum of squares around the mean 均值周围平方和：计算y的平均值m，再将每个y计算(y-m)^2，求和
  * SSR - sum of the squared residuals。也称为L2损失
* Var(mean) - variance around the mean 方差 = SS(mean) / n
  * 概率论上是除以n-1，是每个样本值与全体样本值的平均数之差的平方值的平均数，也叫样本方差、抽样方差、无偏估计、无偏方差
* MSE, Mean Squared Error 均方误差, Var(residual)：是常见的损失函数。也叫总体方差、有偏估计。再开方称为均方根误差
  * （在线性回归中）MSE对于离群值的损失计算更大，为了减小损失，模型相对于MAE会更偏向于离群值，即会受异常值影响
* MAE 平均绝对误差：不可导，无法梯度下降，DL中不用。不除以n称为L1损失
* 残差 residual：对于某个点，预测值到真实值的距离。只有线性模型才有此概念
* 偏差 bias：宏观上表示简化的ML模型无法捕捉数据的真实关系。如用直线拟合曲线关系则具有高bias
* bias 和 variance：前者表示在训练集上的拟合程度，后者表示在测试集上的拟合程度。低bias+高variance为过拟合overfitting，高bias+低variance为欠拟合underfitting
  * underfitting 训练和验证都有较大误差。overfitting 能非常好地匹配训练，但验证却有很大误差
  * early stop：训练过程中，训练集的loss不断下降，但验证的loss上升，说明模型过拟合。此时停止训练，选择最好的模型
* 斜率：slope。截距：intercept
* Nominal标称 和 Ordinal有序：值的顺序是否有意义
* Mode众数：出现频率最高的值
* Discrete Data 离散数据：定性 qualitative，如各个城市的名字。连续数据：定量 quantitative，数字，可测量。预处理数据时一般要把二者分开
* CARTs：分类和回归树
* TODO: 除以n和除以n-1的区别

## 数据预处理

* TODO：顺序。是不是有些训练和测试都要用，有些只用于训练
* pipe = sklearn.pipeline.make_pipeline(预处理器, 转换器, 可选模型)：pipe.fit(X_train, y_train)自动对于数据先fit再transform，对于模型仅fit。y_pred = pipe.predict(X_test)自动依次transform，最后predict。能用统一的方式处理训练数据和测试数据
* 对不同列使用不同的转换方法：make_column_transformer

### 清理

#### 缺失值

* 直接删除：如果一列缺太多可以删列，一行如果只缺一个一般不删
* 代替法：如用0填充
* 插值法SimpleImputer：平均数（非数值如str用不了）、众数、先聚类再平均。KNNImputer
* 模型预测法IterativeImputer：用其他属性预测。但实际上如果其他属性和缺失值无关，则预测结果毫无意义；如果预测得准，说明这个缺失的属性没必要纳入数据集中
* 有的模型如LR不接受空值：Input contains NaN, infinity or a value too large。有的模型内置支持但很少

```py
查看：df.isna().sum() 或 df.info()
一行只有3个及以下有效数据时删除：df.dropna(thresh=4) 当指定列存在NaN时删除那些行：subset=['C']
用均值填充：df.fillna(df.mean())，用前后值填充：method='ffill或bfill'。插值填充：df.interpolate(method=默认'linear')
df.drop_duplicates(keep=默认'first')  # TODO: 处理重复值，还可以聚合（如求和、平均）。现笔记应该只有一整行完全相同时才行，可能还要排序

from sklearn.impute import SimpleImputer
imputed_X_train = imputer.fit_transform(X_train)
imputed_X_valid = imputer.transform(X_valid)
imputed_X_train_pd = pandas.DataFrame(imputed_X_train, columns=X_train.columns) # 如果转回pd，加上列名
```

#### 异常值（偏离值、离群点）

* 检测：箱线图、Z-score、IQR法。孤立森林sklearn.ensemble.IsolationForest，假定异常样本的比例低
* 处理：删除、修正为阈值

#### 其它处理

```py
df.select_dtypes(include=['object']).columns # 查看非数字列的列名
df.select_dtypes(exclude=['object']) # 去掉非数字列
```

### 变换

#### 编码分类变量

* 线性模型假设特征之间存在线性关系，要将分类变量转换为数值 。如果使用标签编码Label Encoding，如性别编码为0 1，模型可能错误地认为“1比0大”
* 无序分类变量用独热编码One-Hot Encoding，有序用标签编码（如教育程度：小学<中学<大学）
* 类别数量极大时，独热编码会导致维度爆炸。此时需选择其他方法：目标编码Target Encoding（用类别目标均值替代）、嵌入Embedding、频率编码
* 独热编码的多重共线性问题：如3个值会创建3个feature分别是[1,0,0],[0,1,0],[0,0,1]，但其实用2列就能表达，分别是[0,0],[1,0],[0,1]。解决办法就是删掉一列，pd用drop_first=true，sklearn用drop='first'
* 树模型不需要转换，或模型支持指定参数自动转换，如categorical_feature、enable_categorical=True

```py
dummies_C = pd.get_dummies(df['C'], prefix='C') # TODO: 直接pd.get_dummies(df)是什么效果
df = pd.concat([df, dummies_C], axis=1)
df.drop(columns=['C'], inplace=True)

from sklearn.preprocessing import OrdinalEncoder # 可能存在X_train和X_valid里有不同值的情形，此时要drop掉差异部分，太复杂略
label_X_train[object_cols] = ec.fit_transform(X_train[object_cols])
label_X_valid[object_cols] = ec.transform(X_valid[object_cols])
ec.inverse_transform 转换回原始形式
```

#### 离散化

将连续变量分箱。

目的一：如年龄对于购买力的影响，25和26不应有显著区别。可分为0-18 18-40 40-60 60-100四类再OneHot编码。但边界处会突变，可改为0-18 15-40 35-60 55-100

#### 样本不均衡

* 过抽样(上采样)增加少数类样本数量，包括简单复制、SMOTE算法添加随机噪声和kNN原理合成、GMM高斯混合模型生成同分布的
* 欠抽样(下采样)减少多数类样本，可能丢失重要信息
* 模型法：有的算法能自动调整
* sklearn的resample实现了过采样和欠采样，通过n_samples参数控制

#### 平滑（阶梯状）

贝塞尔曲线变换、窗口卷积。简单算法：分箱后用某个统计量代替

#### 标准化

* 归一化Normalization
  * 缩放到[0,1]等固定区间。常用 MinMaxScaler = (x-min) / (max-min)、sigmoid、tanh
  * 对异常值敏感
  * 基于距离的算法用到，如KNN、某些聚类算法
  * 当已知数据有固定范围时可用，如像素在[0,255]
* 标准化Standardization
  * 将数据转换为均值0，标准差1的标准正态分布
  * z-score变换：(x-mean)/std
  * 值范围不定，如果处理后>2说明存在异常值
  * 不适合稀疏数据集（有很多0）
  * 一般用于原数据基本符合正态分布的情况。某些模型也假设输入数据是标准化的，如 线性模型, SVM(RBF核), 梯度下降, AdaBoost
* 先划分出测试集再变换，否则训练集就影响了测试集。如果测试集的范围超过了训练集，transform的结果会>1，正常
* 再sc=sklearn.preprocessing.StandardScaler();sc.fit_transform(X_train);sc.transform(X_test)。对于线性回归，y要另创建对象，输出值要逆变换还原到原区间 inverse_transform(y_pred)
* 决策树不需要处理，因为每次只考虑一个条件，不受其他不同量级的影响，关心变量的分布和变量之间的条件概率
* 使用中位数和分位数缩放：RobustScaler，适合多异常值的小数据集，减少过拟合

### 特征工程

* 特征选择
  * 过滤法：基于统计指标（如方差、卡方检验、相关系数）选择重要特征
  * 包裹法：通过模型评估特征重要性（如随机森林、LASSO）。RFC拟合后访问feature_importances_
  * 嵌入法：模型内置特征选择（如XGBoost的特征得分）
  * 序贯特征选择算法SBS：对于所有特征依次删1个，评测看哪个得分更高，就决定删那个，重复直到指定数量；属于贪心思想
  * 能减少过拟合
* 特征创建：如 总收入=工资+奖金、面积×房价

#### 降维（特征提取）

* 减少特征数或数据集的维度且保持结果良好。一般处理相关性高的（冗余），将两个feature合并为一个，如减少图片像素用池化
* PCA主成分分析
  * 无监督学习
  * 将数据集绘制成点，用类似于线性回归的方式找到一个方向，称为PC，保留了最大数据方差。二维下与之正交的就可以去掉，多维下就寻找正交且继续具有最大方差的
  * 会“旋转坐标轴”，产生新坐标
  * 使用前应对数据放缩
  * 只能线性降维，需要数据基本线性可分
  * sklearn.decomposition.PCA 用户指定处理后的维数或百分比或mle算法自动猜测
* 因子分析：是前者的扩展。还能分类
* 线性判别分析LDA
* 非线性方法：流形学习算法t-SNE（也能用于聚类，常用于将复杂数据进行二维或三维可视化，但不可解释）、UMAP
* TF-IDF、Word2Vec
    
### 样本切分

训练集(train)用于拟合模型。验证集(verify)用于调整超参数，由训练集划分出来；调超参实际上也是一种拟合，逐渐由无偏估计变为有偏。测试集(test)用于最终模型的无偏评估。

```py
# 把源数据分成 训练 和 测试 两部分。此处0.1表示10%。shuffle默认True。stratify=y进行分层
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.1, random_state=42)
```

### 模型评价

```py
from sklearn.metrics import *

y_pred = model.predict(X_test)
print("MSE: %.2f" % mean_squared_error(y_test, y_pred))
print("Coefficient of determination: %.2f" % r2_score(y_test, y_pred)) 

# 以上是分开predict和获得score。还有一种通用的集成方式。对于不同模型，分数的意义不同，如回归模型的score就是R^2，分类模型就是accuracy
model.score(X_test, y_test)
```

#### 超参选择 交叉验证

* 常见超参：SVM的C、kernel、gamma
* holdout交叉验证：把训练集划分出验证集，反复调整超参，以在验证集上获得更高分数。一般不用
* K折 K-Fold：避免验证集过拟合。把训练集分成K份，每次取其中一份作为验证集，其余训练。一共训练K次，得到K个模型超参相同及其在验证集上的得分；调整超参后重新训练K次
* 留一法 Leave One Out：每一份测试集的大小是1条数据，即上面的K=数据量n
* sklearn
  * 支持设定n_jobs=-1在所有核心上并行处理
  * 一部分线性模型内置，自动寻找超参：LogisticRegressionCV。好像仅处理正则化强度

```py
# 自动K折验证：cv就是K，默认5。返回分数列表，一般再计算mean和std。当用于寻找超参时，需在前面手动循环超参创建model
sklearn.model_selection.cross_val_score(model或pipe, X_train, y_train, cv=5, n_jobs=-1)

# 手动K折验证
kfold = sklearn.model_selection.StratifiedKFold(n_splits=5).split(X_train, y_train) # 返回值表示一些下标
for k, (train, vrf) in enumerate(kfold): model.fit(X_train[train], y_train[train]); print(model.score(X_train[vrf], y_train[vrf]))

# 查看超参
model.get_params()

# 自动调整（给定的）超参，训练多个模型取最好的。其中Randomized要提供分布，GridSearch提供多个具体值。最后得到的model可直接predict，不用按最佳超参手动重新训练
from sklearn.model_selection import RandomizedSearchCV
param_distributions = {'n_estimators': randint(1, 5), 'max_depth': randint(5, 10)}
search = RandomizedSearchCV(未设定某些超参的model, param_distributions, n_iter=5, random_state=42)
search.fit(X_train, y_train)
search.best_params_

param_grid = [ # 最外层list并列，更改核函数。里面的按笛卡儿积
  {'C': [1, 10, 100, 1000], 'kernel': ['linear']},
  {'C': [1, 10, 100, 1000], 'gamma': [0.001, 0.0001], 'kernel': ['rbf']},
]
```

#### 混淆矩阵 Confusion Matrix

```
      预测值Predicted
       -----------
真实值  | TP | FN |
Actual |---------|
 True  | FP | TN |
        -----------
```
        
* 又称可能性矩阵、错误矩阵。用于评估分类任务
* 其中Actual和Predicted的组合就是符合直觉的那种。但在某些时候，ML中，用True来存放模型的输出，Predicted存放验证数据
* True Positive(TP)、FN（预测没有实际有，又叫二类错误，如漏诊）、FP（预测有实际没有，又叫一类错误，如误诊）、TN（预测没有实际也没有）。对角线是预测正确的
* 样例总数 M = TP + FP + TN + FN。MP = TP + FN，MN = FP + TN
* 准确率(Accuracy, ACC)：识别正确的个数/样本总个数 (TP+TN) / n。表示在所有样本上的预测良好程度；若样本类别不平衡，单纯提高它，不利于发现少数类低正确率
* 精确率(Precision) / PPV：预测为正类别的样本中，真正的正类别是多少 TP / (TP+FP)。表示阳性是否可靠，当FP代价高时要增加它。提高阈值导致二者数量都减少，但比例增加
* 召回率(Recall) / 敏感性/灵敏度(Sensitivity) / TPR / 查全率：在实际正类别中，模型能预测出(预测为正)多少 TP / (TP+FN)。将所有数据全预测为正（降低阈值）就能达到100%；会导致FP增加，但能降低FN。在推荐系统中，尽可能将正类包括，避免遗漏，之后再排序
* 特异性(Specificity) / TNR：在实际为负类别的样本中，模型能够正确预测为负类别的比例 TN / (FP+TN)
* F1分数(F1-score)：精确率和召回率的调和平均数。取值范围[0, 1]，越接近1表示模型的性能越好
* ROC图：依次调整分类阈值（超参），达到增加TP（纵轴）、减少FP（横轴）的目的。纵横轴值域都为1，一开始阈值设为0表示模型都预测为P，导致TP和FP都为1，随着调整逐步往左上角移动，之后再往下移动直到TP和FP都为0即都预测为N。左上角的点代表较好的阈值
* 曲线下面积(AUC)：将ROC的点连接起来，与横轴之间的面积就是AUC，更换不同的模型算法，也绘制ROC和计算AUC，AUC更大代表更好。AUC的值也能通过计算得到：依次各取一个P和一个N，进行 MP × MN 次比较，将P类的模型得分高于N类模型的次数记为H，则AUC = H / (MP × MN)。如果AUC=1，则可以完美区分；如果=0.5，则没有任何区分度；如果<0.5，则比随机猜还差
* sklearn.metrics.confusion_matrix(y_test,y_pred)、classification_report(y_pred,y_test)、roc_auc_score

## 线性回归 Linear Regression

* 用于预测连续值。也能用于判断变量之间是否相关。也能用于回归：加一个阈值函数映射到类别，如z>0表示类别1，<0表示类别0，但太“硬”
* 用最小二乘法(least squares method, LSM)将数据拟合到直线。数学上可以求导得到正规方程直接得到参数值，但工程上一般用迭代的方式
* R^2 = ( Var(mean) - Var(fit) ) / Var(mean) = 1 - Var(fit)/Var(mean)，是MSE的标准化版本
  * Var(mean)是按y的平均值计算方差，Var(fit)是按拟合的直线的取值作为期望算的方差，因为分母一样，其实可以不用除以n，即SS
  * 假设结果是0.6，就可以说：当我们把x考虑进来后，y的方差有60%的减少；或x"解释(explain)"了y的方差的60%；60%的方差可用x-y的关系解释。越接近1表示强相关
  * R：相关性的度量标准，量化mean和fit的区别。R^2比R更符合直觉
  * 计算p值。用于决定R^2的关系有多可靠
* 在推荐系统，CTR点击率预估领域，还是以LR为主

### 岭回归 Ridge Regression / L2正则化 Regularization

* 当训练数据量很少时，即使训练集拟合得好，测试集也不好，称作低bias高variance。岭回归增加bias提升泛化能力
* 训练目标变为：最小化 SS(residual) + λ * (斜率)^2，后半部分称为岭回归惩罚。增大λ将导致拟合的斜率减小，对feature不敏感，相当于将feature值（包括“最优解”时的值）往0压缩。对于多特征，是λ*sum(各特征^2)
* 需要对输入数据 中心化和放缩
* Lasso回归 / L1正则化：λ * abs(斜率)
  * 效果上可以把一些系数缩小到0，可以理解为一种特征选择。如果无用特征多（稀疏），用Lasso；如果有用特征多，Ridge稍好
* 弹性网络回归：L1和L2的组合
* 稳健回归：用于减少异常值的影响。包括huber回归和RANSAC随机采样一致性迭代算法

## 逻辑回归 Logistic Regression

* 虽然叫回归，但其实仅用于分类
* sigmoid函数：形状像S的函数。logistic函数：一种典型的sigmoid函数，f(x) = 1 / ( 1 + e ^ -z )
  * z为线性输出，数学上是 k(x-x0)，ML中是 w1x1 + w2x2 + b
  * z又称为 对数几率比(log odds ratio, logit)，是logistic函数的反函数。从fx中解出 z = log( y / (1-y) )，其中y和1-y就是两种概率输出
* 值域在[0,1]，表示概率
* 对数损失函数（0-1损失函数的平滑版）：sum(log( e^(-y*z) + 1 ))。逻辑回归模型的变化率不是常量，如果用MSE，当输出越来越接近0和1时，精度不够
* L2正则化：如果不做，在模型具有大量特征的情况下，逻辑回归的渐近性质会不断将损失推向0。一般还配合限制训练次数（Early stop）
  * sklearn：C默认=1.0 与λ成反比
* “逻辑回归是线性的，但Logistic函数不是线性的”：它的核心模型、决策边界在特征空间中是线性的，但为了输出概率值，使用了一个非线性变换

### Softmax回归

* 又称为 多项式逻辑回归，用于多元分类。另一种方法：OvR(One versus Rest) 为每个类别训练一个分类器，此类别为正，其余所有类别都是负；最终判断时一个样本可以属于多个分类。sklearn现只用multinomial
* 相当于只有一个输入、多个神经元（数量等于类别）的单层神经网络，训练一组 wi * x + bi，传给softmax函数，得到“概率”
* 当K=2时退化为二元逻辑回归
* 使用交叉熵损失Cross-Entropy Loss = -log(p) 其中p是模型对于目标类的输出值，对每个样本计算求和。此函数对于p=0产生大值，而MSE最大只有1
* model.predict_proba 对于多分类，依次返回各个类的概率（二分类就会返回2个）

## 其它算法原理

### KNN K最临近

* 非参数算法，不需要训练模型，或称为懒惰学习法：给定训练数据时，只是简单存储。对于一个新数据，对其的预测是周围K个的平均值。可用于回归和分类
* K的取值是超参数，过大会欠拟合，过小会过拟合
* 距离度量：连续型有闵可夫斯基距离、余弦相似度、皮尔逊相似系数。离散型有汉明(编辑)距离、杰卡德Jaccard相似系数（= 1 - 二者相同的项/总项）
* 样本多时计算花费大。替代：局部敏感哈希LSH、树结构如KD树
* 容易因为“维度灾难”而过拟合：随着feature维数的增加，样本空间变得稀疏，即使最邻近的值也很远。解决：特征选择、降维

### SVM 支持向量机

* 在数据点之间绘制决策边界，将数据尽量分开
* 所谓的“支持向量”指的是靠近边界的样本
* 远离边界的正确分类点不会参与拟合（Hinge损失的0-loss区域）
* 核函数：将数据投影到更高维，处理线性不可分。核技巧：用某些简单的式子来表示距离，不用真的计算出高维投影结果再处理
  * 如两组按圆周分布、半径不同的数据，投影到三维，将外圈数据z轴值变大，就可以用一个平面将二者划分
* 如果数据量不够DL，可以选用
* 分类器称为SVC

### 朴素贝叶斯分类器 Naive Bayes

* 假定（现实世界中难以存在）不同特征出现的概率相互独立
* 贝叶斯定理：P(A|B) = P(B|A) * P(A) / P(B)。P(A|B,C) = P(B|A) * P(C|A) * P(A) / (P(B)*P(C))
  * A是分类，B是特征（如某个单词的出现次数(count)）
  * 左边是待计算的（给定新特征B要求计算分类A），右边是模型参数（训练数据样本）
  * 其中分母不知道也不必计算，对于A的每个取值，计算分子，比大小即可
* 拉普拉斯修正：如果某个条件概率不存在于训练集中，会导致概率乘积计算为0。修正各个概率为：对于每一种取值，假定先存在一个样本。如P(yes) = (count(yes)+1) / (count(*)+2)，P(x|yes) = (count(yes and x)+1) / (count(yes)+distinct(X))
* feature和target都要是离散化的
* 用途：垃圾邮件过滤

### 决策树 Decision Tree

* 基本上是一系列Yes or No（或者if else）问题，叶子结点就是分类结果
* ID3算法：优先划分具有最大化信息增益（熵增、基尼杂质）的feature
* C4.5算法：ID3的改进，能够处理连续型属性、有缺失值的数据、使用信息增益率作为属性选择的标准（不再偏向分支多即取值值多的属性）
* 预剪枝：当某一分支的样本数量小于阈值，或达到预定深度时，不再划分。后剪枝：构建完整的决策树之后，合并某些结点。sklearn有一个自动且复杂的后剪枝功能，没有手动的
* 当无法再进行分类时，如特征已用完或剪枝，确定叶子的类别：一般用多数表决法
* 只能产生正交决策边界：如果把feature画在坐标轴上，决策树只能产生垂直于坐标轴的线。在每个节点只考虑一个特征进行分割
* 超参：深度（越深越overfit）、max_leaf_nodes
* 分类误差 是修剪决策树的标准，但不建议用于构建

### 集成方法

* 组合多个简单算法，用一个元模型聚合输出
* 投票分类器：在同一个数据集上训练多个 不同算法 模型，选择出现次数最多那个类别（众数）。如果是回归，一般用均值
* bagging / 自举聚合BootstrapAggregation
  * 将一个样本数据集*有放回*抽样出多个子集（也可能数量等于原样本），称为bootstrap，分别（并行）训练相同基线的算法模型
    * 随机森林RF：有放回地随机选某些feature，算法为决策树
      * sklearn.ensemble.RandomForestRegressor/RandomForestClassifier
  * 每个独立的模型过拟合，集成后再减小
  * 包外OOB估计：因为样本可重复选，可能存在样本从未被选中，称作OOB样本，天然作为测试集
* 提升boosting：每个模型接受上一个模型的输出，修复它的问题（减少欠拟合/训练误差），顺序
  * AdaBoost：每个预测器不断改变样本的权重
  * GradientBoost：没有调整样本权重，而是使用前一个预测器的残差作为标签进行训练
  * XGBoost：能并行，能处理异常值和缺失值等。实测完全不调参与RF差不多。后期出了hist版内存占用更小。https://neptune.ai/blog/xgboost-everything-you-need-to-know
  * CatBoost Yandex：有序提升、对称树、巧妙地处理类别特征，不适合稀疏数据集。训练速度慢
  * LightGBM 微软：原理类似hist版的XGB，但内存占用更小，速度更快，效果也不错。但有人测试比Cat和XGB分数差
  * sklearn的HistGradientBoostingClassifier。hist是将大量连续数据先分箱，适合样本数>10000，比普通GBT和RF的快很多。实测class_weight设为balanced会更差，且不能与categorical_features同用
* stacking：组合不同算法
* 如果不进行超参调节，RF比GBM好。RF只要调max_depth, n_estimators, class_weight。GBM峰值性能好，但要调的参数多，也相对容易过拟合。min_samples_leaf当数据集小时应减小

### 聚类（寻找物体之间的自然分组）

* 硬聚类、软聚类（一个点可以属于多个集群）
* 测量相似度：对于点，用欧几里得距离。对于向量，用余弦距离。对于两个簇的距离：如果数值型，用平均欧几里得距离，或“重心”之间的距离。如果平均没有意义，也有选择最近的点或最远的
* 简单度量指标：纯度 = 簇中占主导地位的类 / 簇的大小。缺点：当分成n类每类1个时能达到100%
* 可能用于解决线性不可分：先聚类，再每个簇训练一个分类器

#### 分层（树）

* 凝聚：AGNES算法，自下而上，每次合并距离最小的两个类
* 分割：DIANA算法，自上而下，先将距离其他点距离最大的一个点划出来，再遍历其他所有点，看离哪个簇更近
* 需要终止条件，因为可以把所有点都分成一个类，也可以每个点都各自分一个类。可指定聚类个数K，也可以指定其他阈值如一个类的半径
* 缺点：一个对象一旦划分，就无法撤销。时间复杂度大
* 其他算法：BIRCH 使用 CF 树并逐步调整子聚类的质量。CURE 从聚类中选择分散良好的点，然后将它们向聚类中心收缩指定的分数

#### 分区（划分）

重定位、K均值、k-medoids(PAM)就是集群由集群中的一个对象表示。

KMeans：先随机选k个中心，将各个数据点分配给离它最近的中心，再对于每个类的数据均值重新计算中心，反复直到稳定。是EM算法的思想。\
选择K的方式：当K增加时，Loss（集群距离总和）一开始会迅速减小，之后减小的幅度平缓了，取拐点的K。\
仅适合具有凸形状的聚类，无法处理两种数据按同心圆半径不同分布。特征维度不宜过高。\

KMeans++：初始随机选一个中心，各点按到中心的距离归类（初始只有1类），计算簇内点到中心的距离，取最远的作为下一个中心，直到K个，再将它们作为普通kmeans的初始中心。

#### 基于密度

* 可以发现任意形状的簇，对异常值不敏感
* DBSCAN：给定一个半径从一个点开始画圈，如果圈内点的个数大于给定阈值，则此点称为中心点，圈内的点称为直接密度可达；如果小于阈值，但自身在另一个中心点圈内，称为边界点；否则称为噪声点。如果a->b->c，则称为密度可达。如果a->b a->c，则abc称为密度相连；都连在一起，就是一个簇
* OPTICS、DENCLUE

#### 其它算法

* 基于网格：速度与数据对象个数无关
* 基于模型：神经网络（Self-Organizing Map、Autoencoder）、统计（混合高斯模型）

#### 需求

* 可扩展性（在时间和空间方面）
* 能够处理不同的数据类型
* 对确定输入参数的领域知识要求最低
* 能够处理噪声和异常值
  * ①计算完簇内距离平均值后，如果偏离太远就剔除。②用中位数，不受异常值影响
* 对输入记录的顺序不敏感
* 合并用户指定的约束
* 可解释性和可用性

## 问题构建

* 用非ML术语回答“我要达成什么目标？”
* 先验证当前的非ML解决方案。如果没有，可以用“启发式”(heuristic)，看起来基本就是按经验选几个feature，用固定方式计算，不保证全局最优
* 比较ML在质量上的改进、费用和维护
* 用于ML的数据：大量、一致、可信（来源）、正确、代表性。标签与特征之间具有相关性
* 确定需要的输出
  * 预测性ML
    * 二分类、多类别单标签（对图片中的动物进行分类） 、多类别多标签（对图片中的所有动物进行分类）
    * 一维回归、多维回归（预测多个标签）
  * 生成式AI
    * 文本：总结、翻译、分析
    * 图片：生成、生成变体、应用视觉效果
    * 音频、视频
  * 假设您始终拥有正确的答案。您会如何将其用于自己的产品？
* 代理标签：如用 用户是否愿意分享或点赞 作为衡量用户是否认为视频有用
* 定义成效指标：提升了 具体的百分之多少。再决定是否有改进模型的可能，以及代价
* 实现模型：从简单模型开始作为基准（也可用预训练模型），设置好数据流水线，监控训练，部署，训练-应用偏差
* 安全考量：https://learn.microsoft.com/zh-cn/windows/ai/rai

---

## [pandas](https://pandas.pydata.org/docs/)

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
pd.DataFrame([ [1,3], [2,4] ], columns=['A','B'])  一般是 按行导入 原无列名 的外部数据
pd.DataFrame({'A':[1,2], 'B':[3,4]})  两列AB，两行，第0行数据是13。指定行名：index=['row1', 'row2']
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
df[0:2]/[1:]  选取一定范围的行，一定要是slice；仍为DataFrame，即使结果只有一行。可被iloc完全替代，原本设计类似py切片，不支持numpy那种，最好不用
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



fig, ax = plt.subplots(ncols=2, figsize=(10, 5), sharex=True, sharey=True)

ax[0].scatter(X_train, y_train, label="Train data points")
ax[0].plot(
    X_train,
    regressor.predict(X_train),
    linewidth=3,
    color="tab:orange",
    label="Model predictions",
)
ax[0].set(xlabel="Feature", ylabel="Target", title="Train set")
ax[0].legend()

ax[1].scatter(X_test, y_test, label="Test data points")
ax[1].plot(X_test, y_pred, linewidth=3, color="tab:orange", label="Model predictions")
ax[1].set(xlabel="Feature", ylabel="Target", title="Test set")
ax[1].legend()

fig.suptitle("Linear Regression")

plt.show()





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

其他可视化库：Seaborn(基于matplotlib，用起来更简单，但只支持2D) bokeh plotly功能最多可以画地图 plotly/dash(基于plotly.js，用于构建网页) altair Plotnine pyecharts mlxtend（能直接对整个df画各feature的图）

## scikit-learn

* 教程
  * 中文文档：https://scikit-learn.org.cn/lists/8.html 教程 https://scikit-learn.org.cn/lists/2.html 用户指南 https://sklearn.apachecn.org 另一个中文站
  * https://zhuanlan.zhihu.com/p/88729124 https://zhuanlan.zhihu.com/p/103136609 https://zhuanlan.zhihu.com/p/99618155 https://zhuanlan.zhihu.com/p/29649128 https://zhuanlan.zhihu.com/p/190049765
* 序列化持久保存：pickle、joblib.dump(m, 'filename')第三方二进制序列化库内部基于pickle格式加载时会用mmap、treelite编译决策树的库
* 其他库：yellowbrick图形化，mlxtend工具类，dtreeviz可视化，scikit-optimize，m2cgen把模型转换为其它语言，featuretools，Sacred能保存各种参数用于复现
  * 自动机器学习，自动调超参数：https://auto.gluon.ai。https://github.com/nidhaloff/igel
* solver求解器：使用哪种算法寻找参数。可能涉及功能（有的不支持多分类，是否支持正则化），可能涉及性能（大数据集），有的在其他二进制库中实现（一般自带了）

```py
y = data.Price # 选择一个列作为预测目标target。小数则为回归，整数或其它离散量则为分类
X = data[['col1','col2']] # 选择一些列作为“features”。另一种选择方式：去掉不要的drop(columns=['Price'])。如[[1,2,3],[4,5,6]]表示2个sample，3个feature

# 内置了一些数据集，小型的自带，大型的使用时会联网下
from sklearn.datasets import load_iris
X, y = load_iris(return_X_y=True)
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

## 传统NLP

* 任务：垃圾邮件、短信识别。文本相似性判断，用于主题聚类或信息检索。情感分析。质量评估。主题提取
  * 命名实体识别NER：人名、机构名、地名。可以基于规则或统计
  * 词性标注POS, Part of Speech Tagging：与NER相比都是类别识别、都是序列标注，不同点在于POS更细
* 基于规则的NLP：需要了解语法Grammar、词性POS、构词法Morphologic
* 流程
  * 文本预处理管道：词元化Tokenization，包括转换成小写、删除停用词、拆分。词干提取running->run。词形还原Lemmatization，如spoken->speak
  * 删除停用词Stop-Words Removal：就是无意义的词，如“的”、“the”
  * 数据清洗：如删除所有HTML标记 `re.sub('<[^>]*>', '', text)`
* 分词
  * 基于字典和字符串匹配：正向最大匹配MM从左到右。逆向RMM。最少切分。双向最大匹配BMM若正向逆向不同则取最少切分。最佳匹配OM，就是给词典排序时按词频，对提高分词效果无帮助
  * 基于理解：又称基于人工智能，在分词的同时进行句法、语义分析来处理歧义，包括神经网络和专家系统和二者集成
  * 基于统计：无字典（实际一般与基于词典的结合）。N元语法N-Gram模型、隐马尔可夫HiddenMarkov模型、最大熵模型MEM
* 文本的数值表示
  * OneHot：任意两个词之间的距离相同。矩阵稀疏。没有考虑上下文
  * 词袋模型(Bag of Words, BOW)
  * Word2Vec：基于DL的无监督学习模型
* 主题聚类：超参指定k个主题个数，对于一些无label的文章，将它们归到k个类中，能输出每个类中最关键的n个单词。常用算法：LDA潜在狄利克雷分配

### 词袋模型

1. 词汇表：dict{单词:序号} 按字母顺序
2. 特征向量：对于每个文章或句子，将其中的词汇聚合生成 list[(序号，出现次数或称为频率)] 或 隐含序号list[频率]，长度等于词表长度，称为MultiHot

缺点：不考虑词序特征、文法、句法特征。顺序丢失（无聊不好玩 与 好玩不无聊 的BoW完全相同）。忽略语义（苹果手机 与 吃苹果 的苹果意义不同）。忽略单词之间的距离（手机 互联网 苹果 前两者距离应更近）。也比较稀疏

sklearn：feature_extraction.text.CounterVectorizer

n-gram：上面的Bow称为1-gram或unigram。2-gram对于分词后的结果，两两按顺序组合编码。如原文ABCD，若仅按观察到的作为特征，则创建(AB, BC, CD)；若基于所有可能的，则创建(AA, AB, BA, ...) 共4x4=16个，非常稀疏。

单词频率(词频)-逆文本频率TF-IDF矩阵：某些词在各个文章中经常出现，则它通常不包含有判别性的信息。本方法相当于对某些单词加权，更突出“领域”词。\
公式：

其中TF就是BoW里的，IDF是log(语料库中文档总数/(包含词𝑤的文档数+1))，把它乘以TF。
        例如：某文章，“且”和“机器学习”出现次数相同，则它们的TF相同。再看语料库，如果每篇文章都出现“且”，则它的IDF是log1等于0

### 库

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
