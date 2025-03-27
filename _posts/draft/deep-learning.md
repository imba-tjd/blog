https://www.youtube.com/watch?v=hfMk-kjRv4c

## 原理

* 输入和输出维度是业务决定的，不能随意更改。但隐藏层可以随意改
* 特征提取
  * onehot编码：对于每一个值都编码一个维度。如 有无 分别是 [1,0] 和 [0,1]。会导致共线性Collinearity，对于不含惩罚项的线性模型有问题。实现：sklearn.preprocessing.OneHotEncoder、pandas的get_dummy
* 单个神经元做不出XOR门
* 国内的凹函数：国外称为convex，翻译是凸函数；与之相对的是concave
* 常见的超参数：学习速率eta、批量大小batch size、迭代次数epoch
* 多分类：如果最终结果有3类，则最后输出3个神经元，分别对应每个类别，正确类别的label是1，错误的是0。正向传播时看哪个的值最大（原始输出值，不在[0,1]），或再加argmax/softmax
  * ArgMax：如[1.5, -1, 0.4] -> [1,0,0]。容易解释，但不能用于反向传播优化，因为一种实现是大于等于第一大的值设为1，小于的设为0，导致导数恒为0
* 权重可理解为关系的强弱，每层中的一部分神经元可理解为捕获了某种模式
* cost/loss：可理解为 输入是所有NN参数，输出是一个数值（代价），它的参数是训练样本。梯度下降：改动哪个参数能使得cost减小得更快

### 激活函数（非线性变换）

* 使用激活函数后，bias的含义不再是平移直线，而是一种触发激活函数的阈值
* ReLU：x >= 0 ? x : 0，或max(0,x)；负区间导数为0，可能导致神经元“死亡”；0点导数可人为设成0或1。Leaky ReLU：x<0时用x/100
* Sigmoid（实际是logistic函数）：1 / ( 1+ e^(-x) )，S形曲线，将x压缩到0到1。二分类中常用，在隐藏层中较少使用
* SoftMax：多分类中的最后一层使用，有几个输入就有几个输出，f(i) = e^i / sum(e^j)，类似于概率，值域在[0,1]，各类值之和为1。有用于反向传播的导数。Sigmoid是它的特例
* tanh：也是S形的，且关于原点对称，值域-1到1
* SiLU：x / ( 1+ e^(-x) )。形状整体类似于ReLU，但可导。0点=0，x小于0时y小于但接近0
* SoftPlus：log(1+e^x)。形状整体类似于e^x，只是大于0时线性上升；也可以说类似于平滑ReLU
* 当损失不下降时，一种原因是梯度消失（梯度在反向传播过程中逐渐趋近于零，导致靠近输入层的参数更新缓慢）。当使用Sigmoid或Tanh等饱和型激活函数时，其导数在输入值较大或较小时趋近于零，导致反向传播时梯度迅速衰减，无法有效学习。解决办法是ReLU

### 一种Layer的表示方法

* weight = 二维数组[输入维度,本层的神经元数或输出维度]，bias = [神经元数]
* 单个神经元就接收一组维度的输入，对应相乘求和，加上偏置，传给激活函数，才能输出一个值。即单个神经元具有 w=List[输入维度] 和 b，一层具有List[神经元数]；但若按这种方法表示，相乘前要转置
* 后一层的输入维度就是前一层的输出维度
* 当输入数据为batch时，X = [样本数][特征] 即每一行是一个样本，列是对应的特征。做矩阵乘法

### 反向传播backpropagation

#### 单个神经元（MCP perceptron感知机模型）学习规则

1. 对于一次输入，具有一组x，正向传播act(sum(wi * xi)+b)，得到一个输出值，减去真实值得到Error。
2. 应用链式法则，（如用ReLU）乘以激活函数的斜率的绝对值。乘以学习率如0.01，得到调整值。
3. b -= 它，wi -= 它*xi。

#### 调整w和b的原理

* 增加还是减少：如果Error为正，说明计算值偏大，w和b减去正数变小。如果Error为负，说明计算值偏小，要调大w和b，减去负数等于加上正数
* 调整程度：如果Error很大，则调整幅度就大，这样允许先处理更差的。乘以激活函数的斜率是因为可能一点改变就产生很大区别
  * 梯度爆炸：在RNN中，随着输入，参数不断相乘以指数形式增长，若大于1，最终会产生一个巨大值，导致step size巨大，无法良好梯度下降；类似于学习率太大。若小于1，相乘后又会趋于0，每一步太小，最后达到迭代次数，称为梯度消失
* 迭代次数：线性可分的可以证明一定能收敛，表现为一轮迭代后不更新任何参数。若不可分则会一直不收敛，一直有参数更新，要指定迭代次数，或更新值很小
* 对于原始仅用于分类的感知机：线性计算后经过一个阈值函数，如z>0?1:0，才算作模型的输出，再与target相减，是“01误差（误分类）”。而Adaline模型的输出是连续值，计算误差未经过阈值函数；如果用于分类预测也是再用阈值函数

#### 梯度下降+MSE（Adaline算法）调整w和b的原理

假定x是一维的（特征为1），有多组x,y（batch size > 1）。

1. z = ((w*x + b) - y)^2，将w和b视为变量，分别求 ∂z/∂w、∂z/∂b 导函数，代入x,y数据集求平均。得到的值称为weight/bias slope
  * 不求导函数，近似计算：取h=0.01，算 (f(xj+h) - f(xj)) / h
2. w和b -= 学习率 * 上面算出的slope，称作step size

当有多维和多组数据：

```
w: [n_features]
X: [n_examples, n_features]
y: [n_examples]
out = X @ w + b -> [n_examples]
errors = out - y

b -= eta*2.0 * errors.mean()

w -= eta*2.0 * X.T @ errors / X.shape[0]
等价于 w[i] -= eta* (2.0 * X[:,i] * errors).mean() 对于第i个feature，计算从所有样本产生的损失的均值；对于单个样本，必须对应从它产生的error 数值相乘（不是@）
```

即使数据并非完全线性可分，仍可产生线性决策边界并收敛。

优化前面的权重：SSR对前面的权重求偏导，先对y'求偏导，具体值与在优化后面的权重时一样但实际是同时计算的；再将中间的w视为常量，因为变量是y''，求导得w；最前面的w是变量，求导得x即输入。

##### 随机梯度下降法(SGD)

* 在分布式训练中表示每次迭代只使用一个数据（batch size为1）。更新过程中损失会略有波动（噪声）
* 原意表示每一轮使用数据的随机子集。每个epoch要shuffle样本，减少噪声
* 如果一次把数据集全算完再更新，叫全批量。还有小批量
* 从一组数据计算梯度更稳定，但太大可能爆内存、可能收敛到不太好的局部最小值。小批量噪音大，收敛效果可能好，也可能难收敛
* 可以在线学习 partial_fit

## 常见模型

* GAN：生成与给定数据集相似的新数据样本，用于图像合成、风格迁移和数据增强

### CNN 卷积神经网络

用于图像和视频。

输入：样本数 x 通道数 x 长 x 宽。

卷积核：如一个3x3的矩阵（每个元素是可学习的参数），在原图像上按 对应相乘再求和 输出一个值。\
卷积层：多个（如5个）卷积核，输出多个“通道”。\
卷积操作：往右移动1步或多步，一般是重叠的。

池化：降维操作。如取2x2的范围中的平均值或最大值。没有参数，不可学习。一般不重叠

现代卷积神经网络：AlexNet、ResNet、DenseNet。

### RNN 循环神经网络

MCP和CNN对于输入都是固定的，而RNN可以处理非固定长度的序列，且输入样本的顺序是有关的，如NLP、语音。\
序列建模：多对一，如情感分析输入文本，输出类别。一对多，如输入图像，输出文字描述。多对多，可以是每个输入都对应一个输出；也可以等输入全部完成后再输出，如翻译。

结构：对于一个输入x1，计算 `z = act(w1*x1+b)`。此时若继续传播到下一个神经元w3 b3，即预测输出。它还可以与新数据一起投入下一次的输入，乘以w2，与 w1和下一个数据 求和，再加偏置，称为反馈循环：`z2 = act(w1*x2 + w2*z + b)`，其中w1 w2 w3对于所有输入数据都是共享的。\
但不太适用于长序列，当w2>1时会梯度爆炸，w2<1会梯度消失。

x可以是向量，若w1具有


### LSTM 长短期记忆神经网络

* 宏观上：RNN对于很久以前的事件和最近的事件，使用同一个反馈循环，来预测明天。而LSTM使用两条独立的路径预测明天。长期记忆是累加而非连乘，避免了梯度爆炸和消失
* 定义 长期记忆M（又称为Cell State） 短期记忆m（又称为Hidden State）。就是数字；也可能是向量，维度数是超参数，M和m的维度相同。m的值域为[-1,1]
* 一个计算百分比的单元：`P = sigmoid(w1*x + w2*m + b)`，x是输入的数据，m是短期记忆。以下每步中用到的P的参数不同
* 一个激活函数为tanh，其余与P相同的单元T

1. “遗忘门”：M *= P，表示长期记忆还留下百分之多少。
2. “输入门”：M += P * T，表示创建多少百分比的“潜在长期记忆”。
3. “输出门”：m = P * tanh(M)，即更改短期记忆，且也是LSTM的输出。tanh(M)称为“潜在短期记忆”

在输入时，LSTM会展开(Unroll)成按顺序处理的单元。

### Word Embedding、Word2Vec

已经分词了，要对token编号。如果直接按顺序编，或者随机分配浮点数，意思相近的词不会聚集在一起。本模型解决此问题。\
首先有一个词汇表（如长度几千），有一些句子。目标是把所有单个词转换成固定长度（如128）的list[float]，且近义词的距离小。

先对词表做OneHot编码，每个词变为 长len(词表) 只有自己为1 其他都为0 的向量；也可理解为按顺序编码每个词，作为下标。\
创建一个单隐藏层神经网络：输入 上一行 向量；隐藏层的神经元权重就是最终需要的嵌入，共有 len(词表) x 128 个权重，每个词有128个权重，没有偏置项；输出长度又是 len(词表)，再加softmax。\
训练时：依次输入句子中的每个词，target为句子中的下一个词。即期待预测下一个词。\
如句子里有A is good, B is good，则训练后A B的嵌入向量距离小。

负采样优化：因为输入向量极其稀疏，其余词 0 x w 结果都是0，可以不用算。在未训练时模型会输出 len(词表) 的随机向量，而target只有1位为1，可随机选择一些词（如20个）只正向传播计算和反向传播优化它们。

Word2Vec：使用“上下文”信息。①连续词袋：输入时向量变为 [1, 0, 1] 其中一个1是当前词，另一个1代表待预测词的下一个词，用来预测中间词。②跳跃模型：用中间词预测周围词。

### Encoder-Decoder、Seq2Seq

Encoder将一整个句子或文章转换成一个固定长度的“上下文向量”，Decoder将向量解码成句子。两个句子之间长度可以不同，关键是可以具有不同的词表，如用于翻译。

二者都包含多个LSTM堆叠起来。第一层(Layer)LSTM的m不仅作为下一个时间的m，还作为第二层**在相同时间步**的输入。不同层的LSTM有不同的参数，但每个LSTM本身的参数对于时间序列输入来说是复用的。

Encoder输入时，先将句子中的词，逐个编码成Embedding向量，按顺序输入；最后输入EOS。\
Encoder最后一层的最后一个的m是输出，称为“上下文向量”。连接Decoder时也将所有M和m称为上下文向量，对应传给Decoder的层。\
即Encoder处理整个文章，得到上下文向量，后续就和它无关了，与Decoder解耦。

Decoder的第一个输入是EOS，此时间经过Decoder计算，最后一层的m为输出，用一个全连接层（len(ctx_vec) x len(输出词表)）和softmax转换成词汇。该词汇再编码成Embedding作为下一个时间的Decoder的输入，直到输出EOS。\
训练时，不将Decoder的输出作为下一时的输入，而是输入“正确内容”；输出仅用于计算损失。且如果到了该输出EOS时未输出，直接停止训练。这称为Teacher Forcing。

### Attention

最初加在LSTM上，后来只用Attention了。

输入的Token并行编码成Embedding，此时同一个词的向量是一样的，经过Attention层后根据上下文“调整”了。\
每个Attention层有三个矩阵QKV可学习参数，QK的大小为

Cross Attention：翻译任务时，输入输出词表不同，每个输出要考虑每个输入。
Self Attention：自回归预测下一个词时
多头Attention：多个Attention层并行计算

TODO:位置编码


## numpy

* 创建后不能更改大小，元素类型相同（同构），不能是锯齿形的
* dimension和axis：如[[1,2,3]可能表示三维空间中的点，但它只有两个轴
* 其他与py原生list的区别：a+b 原生是扩展，np是运算。arange步长允许小数

```
创建：
arr=np.array([[1,2],[3,4]])
ones/zeros((2,3))  创建2行3列，值为1.0/0.0。指定其他值：full(shape, v)。不初始化：empty
ones_like(lst)  与参数相同形状且值都为1.0
指定整数类型：dtype=np.int32或'int32'或'i'。转换类型：arr.astype(int)、tensor.to(不支持字符串)
np.random.xxx  包括生成 整数或浮点 和 指定范围和个数 或 某种shape，可以生成各种分布。推荐用 rng=np.random.default_rng(42)。如果直接用，不要用rand randn randint。可以用random(可选长度)范围[0.0, 1)
np.linspace(0,1,5)  将[0,1]划分5份：[0, .25, .5, .75, 1]
np.arange(15).reshape(3, 5)

属性：
arr.shape 各维度元素个数，一定返回元组
arr.ndim 单个数字几维
arr.size 总元素个数，即所有维度数相乘。与torch不同，t.size(n)返回某一维度数

取索引：
arr[(arr>3) & (arr%2==0)]、np.where(mask, arr, mask为False时要填充的值)  前者会取值去空成一维，后者能保留原形状。获取符合mask的序号：np.argwhere
arr[:,0]  取第一列组成新的数组。其中:表示取那一维所有，...表示省略指定中间几维
arr[(0,1,2),(0,1,2)]  代表取0,0 1,1 2,2
普通范围索引是view，高级索引是copy

运算：
广播运算。或如果维度不匹配，会扩展（复制值）到相同尺寸，条件：其中一个矩阵必须有一个维度为1、两个矩阵必须有一个相同的维度
[1,2,3] + 1 -> [2,3,4]
[1,2,3] + [1,2,3] -> [2,4,6]
[[1], [2], [3]] + 1 -> [[2], [3], [4]]
[[1], [2], [3]] + [1] -> [[2], [3], [4]]
[[1], [2], [3]] + [1,2] -> [ [2,3], [3,4], [4,5] ]
[[1,2,3]] + [[1], [2], [3]] -> [[1,2,3],[1,2,3],[1,2,3]] + [[1,1,1],[2,2,2],[3,3,3]]

@ == np.matmul 二维矩阵乘法。np.dot 当只有1维时与*相同，是对应位置元素相乘，点乘；当二维时与@相同。一维数组可看作[1,n]和[n,1]，在@的左右两边自适应
mat.T 矩阵转置
arr[0][0].item() 转换回py类型
np.multiply(arr, 2, out=arr) # [[2,4],[6,8]] 设置+返回
np.sum() min() max() mean() median() var() std() 对于多维数组，如果指定axis=0，会聚合每一列（[a[0,0]+a[1,0],a[0,1]+a[1,1]]），=1聚合每一行（a[0,0]+a[0,1]）
np.argmax(arr) 返回值最大的那一项的index
np.log(arr) np.pow(arr,2) np.sqrt(arr) 对每一项运算
np.unique()
np.concatenate((a1,a2))  对于二维数组，它的 axis=0 等于 np.vstack()，axis=1 等于 np.hstack()
np.insert(a1, ndx, a2) np.delete(arr, ndx)
添加新轴：[1,2,3][:, None] -> [[1],[2],[3]]  此处None等价于np.newaxis

arr.argsort()  返回arr排序后对应原数组中的索引。如[3,1,2].argsort() -> [1,2,0]，表示原数组排序后的结果为 arr[1],arr[2],arr[0]。再加一次argsort()得到排名[2,0,1]。手动实现：将原数组转化为(元素值, 索引)，按值排序，返回索引

numpy out has performance benefits？

固定长度的字符串：dtype='<U10'
```

## Pytorch

* https://pytorch.ac.cn/
* https://pytorch.org/hub/ https://modelzoo.co/

### 安装

* Linux版默认有cuda，Win默认为CPU。Linux装CPU版：conda install pytorch-cpu；或-i https://download.pytorch.org/whl/cpu TODO:conda要-c conda_forge？
* 相关项目
  * intel_extension_for_pytorch(IPEX)：对于支持avx512的CPU能加速训练。仅linux或WSL2。https://huggingface.co/docs/transformers/main/en/perf_train_cpu
  * 优化超参数的框架，支持ML和DL框架：https://optuna.org/
  * https://pytorch.org/tnt training tools and utilities
* 环境
  * torch.cuda.is_available()、watch -n1 nvidia-smi、nvidia-smi stats、nvtop、nvitop。2.5支持intel的xpu，有xpu-smi
  * torch.set_default_device('cuda')，否则默认为CPU，要用if torch.cuda.is_available(): t=t.to('cuda') 或创建t时指定device
  * torch.manual_seed(42)

### tensor

```py
torch.tensor(lst)、from_numpy(np_array);t.numpy()二者共享底层
torch.rand(shape) torch.rand_like()
torch.cat([tensor, tensor, tensor], dim=1)
原地改变：x.copy_(y), x.t_()
t.view(1, -1) # 转换为shape[1, sequence_length]  np也有view且效果完全不同，是改变dtype
```

### Module 和 正向传播

* F.relu()是调用函数。nn.ReLU是一个类(Module)，调用后创建函数实例
* nn.Linear 创建后会 自动随机初始化值、requires_grad=True

```py
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim

class LinearModel(nn.Module):
    def __init__(self, input_size, output_size):
        super().__init__()
        self.linear = nn.Linear(input_size, output_size)
    def forward(self, in):
        out = self.linear(in)
        return out

使用：
input_size = 10
model = LinearModel(input_size,5)
model.linear.weight; model.linear.bias
input_data = torch.randn(32, input_size)  # 32是batch size
output_data = model(input_data)  # 调用forward
print(output_data.shape) # (32,5)

其它方式创建：
class LinearModel2(nn.Module):
    def __init__(self, input_size, output_size):
        super().__init__()
        l1 = nn.Linear(input_size, 5) # 注意不是self.l1
        l2 = nn.Linear(5, output_size)
        self.module_list = nn.ModuleList([l1, l2])
    def forward(self, x):
        for f in self.module_list:
            x = f(x)
        return x

LinearModel3 = nn.Sequentail(nn.Linear(2,4), nn.ReLU(), nn.Linear(4, 1), nn.Sigmoid())

手动创建单个神经元：
w = nn.Parameter(torch.tensor(1.), requires_grad=False)
out = F.relu(in * w + b)
```

### 自动微分autograd

1. 创建tensor时设置 requires_grad=True 或 t.requires_grad_() 会创建计算图，跟踪记录对它的操作。nn.Xxx默认True
2. 当模型计算出output_data后，调用.backward() （实际一般对loss调用），会触发autograd引擎以相反的顺序遍历计算图，计算出梯度，储存在那个需要梯度的tensor的 .grad 中

* 计算原理：将输出tensor看作因变量，对想要的那个输入tensor链式求偏导
* 如果参数是固定的，则设为False表示不需要优化
* 获得共享底层参数但去掉梯度：t.detach()。画图时可能用到

### 优化器

```py
optimizer = optim.Adam(model.parameters(), lr=0.001) # 还有SGD
loss_fn = nn.MSELoss()
for epoch in range(num_epochs):
    loss_sum = 0
    for batch_idx, (data, labels) in enumerate(dataloader):
        outputs = model(inputs)
        loss = loss_fn(outputs, labels)
        loss.backward()  # 在tensor中累积梯度
        loss_sum += loss  # 按loss大小终止，不会自动累积
    if loss_sum < 1e-4: return
    optimizer.step()  # 一批统一调整
    optimizer.zero_grad()

自动调整学习率：
from torch.optim.lr_scheduler import StepLR
scheduler = StepLR(optimizer, step_size=30, gamma=0.1)  # Reduce LR by 0.1 every 30 epochs
for epoch in range(num_epochs): scheduler.step()
```

### 数据集

* 数据：垃圾 -> Dataset 提供一种方式获取数据及其label -> Dataloader 为后面的网络提供不同的数据形式，基本就是划分batch
* 来源
  * paperswithcode.com/datasets
  * universe.roboflow.com
  * data.mendeley.com/research-data
  * kaggle
  * datasetsearch.research.google.com

```py
from torch.utils.data import Dataset, Dataloader

class MyDS(Dataset):
    def __init__(self, root):
        self.root = root
        self.data = os.listdir(root)
    def __getitem__(self, ndx):
        return self.data[ndx]
    def __len__(self):
        return len(self.data)
    父类自带了add，可以把两个ds相加

dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

TensorDataset(inputs, labels)  组装已有的tensor
```

保存：torch.save(model.state_dict(), 'my_model.pth')
加载：model.load_state_dict(torch.load('my_model.pth'))





加载图片：
from PIL import Image; import cv2（包名是opencv-python）
from torchvision import transforms
im = Image.open(path) = cv2.imread(path)  im.shape 0->(高,宽,通道)
im_tensor = transforms.ToTensor()(im)
Image.fromarray(nparr.astype('uint8')).show()
降采样，用步长：im[::10,::10,:]。翻转：im[::-1]。裁剪：切片 im[a:b,c:d]

transforms：
含有许多“工具”。一般用Compose([创建多个工具类实例])创建可复用的处理流，再(调用)

from torchvision import datasets
training_data = datasets.FashionMNIST( 预定义的数据集，Each example comprises a 28×28 grayscale image and an associated label from one of 10 classes
    root="data", 储存路径
    train=True,
    download=True,
    transform=ToTensor() # 修改feature。target_transform修改label
)
DataLoader(training_data, batch_size=64, shuffle=True, pin_memory=True)


OpenCV的默认通道是BGR颜色模型
opencv-python-headless


model.eval()
inputs = tokenizer(…)
with torch.no_grad(): # 整个函数：@torch.no_grad
    outputs = model(**inputs)


简单的CNN模型：
class  MyNet(nn.Module):
    def __init__(self):
        super(MyNet, self).__init__()
        self.conv1 = nn.Conv2d(in_channels=1, out_channels=32, kernel_size=3, padding=1) # Convolution卷积
        self.dropout1 = nn.Dropout(p=0.25) # 一种正则化方法，随机失活。其他正则化方法还有L2
        self.conv2 = nn.Conv2d(in_channels=32, out_channels=64, kernel_size=3, padding=1)
        self.dropout2 = nn.Dropout(p=0.25)
        self.fc1 = nn.Linear(64 * 7 * 7, 128) # 全连接层
        self.fc2 = nn.Linear(128, 64)
        self.fc3 = nn.Linear(64, 10)

    def forward(self, x):
        x = self.conv1(x)
        x = self.dropout1(x)
        x = F.relu(x)
        x = self.conv2(x)
        x = self.dropout2(x)
        x = F.max_pool2d(x,2)
        x = torch.flatten(x,1)
        x = self.fc1(x); x = F.relu(x)
        x = self.fc2(x); x = F.relu(x)
        x = self.fc3(x)
        output = F.log_softmax(x, dim=1) # 产生probabilities(可能性,概率)，用于分类
        return output


torch.compile，又叫dynamo
实际上支持任何函数，也可以用作装饰器。会递归编译，一般在顶层使用，排除不兼容的或用某上下文管理器关闭；也可以从底层开始测试
在执行时将模型编译成优化的内核，多次执行才有优化效果
mode=默认"reduce-overhead"，另一个选项是max-autotune
model的grad_fn可以看到是否compile过
只有V100 A100 H100才能看到明显效果
torchtriton：好像能使得在gpu上运行compile


分布式理论：
https://github.com/PacktPublishing/Distributed-Machine-Learning-with-Python
数据并行（训练）：数据加载带宽和模型训练带宽之间不匹配
解决：拆分（不相交）数据集到多GPU上。
新问题：如何同步。解决：①随机梯度下降SGD优化器（且初始化时用相同的种子）。②模型同步（有多种方案）。③超参调整：batch_size、学习率
模型同步：
①采用一个(组)中心节点作为“服务器”，工作结点拉取参数，训练后提交。缺点：服务器带宽瓶颈；如果用多个服务器，则会变复杂。
②All-Reduce架构：只有工作节点。一轮训完全汇总到一个，再广播。Ring All-Reduce（NV NCCL是其理论的实现）：将所有节点视为环，每个节点接收上家发来的，与自己的合并，发给下家，走完一圈后最后的节点拥有总和，再走一圈同步
③：All-Gather：每个节点都广播自己的值，并且接收所有其他节点。数据传输量远超All-Reduce

多GPU数据并行训练：if torch.cuda.device_count() > 1: model = nn.DataParallel(model)
模型并行（LLM推理）


Lightning：
import lightning as L
class NN(L.LightningModule):
    init和forward不变
    def configure_optimizers(self): return SGD(self.parameters(), lr=self.learning_rate)
    def train_step(self, batch, ndx):
        in, label = batch
        out = self.forward(in)
        loss = loss_fn(out, label)
        return loss
trainer = L.Trainer(max_epochs=99, accelerator='auto', devices='auto')
model.learning_rate = trainer.tuner.lr_find(...).suggestion()
trainer.fit(model, dataloader)


GPU资源：
https://console.cloud.intel.com/home?region=us-region-2 



https://zh.d2l.ai/chapter_introduction/index.html
循环神经网络：RNN,GRU,LSTM,seq2seq。优化算法：SGD,Momentum,Adam
《动手学深度学习》实体书
深度学习入门 : 基于Python的理论与实现     9787115485588


https://learn.microsoft.com/zh-cn/training/paths/get-started-with-artificial-intelligence-on-azure/
https://learn.microsoft.com/zh-cn/collections/r47ni8gp1eqgqp?ref=collection&listId=1nx4c0zqyoodor&sharingId=6A9F03F25E12DA9E&wt.mc_id=aisc25_landingpage_wwl
https://learn.microsoft.com/zh-cn/training/browse/?wt.mc_id=aiml-7486-cxa&subjects=artificial-intelligence
https://learn.microsoft.com/zh-cn/training/browse/?expanded=azure&roles=ai-edge-engineer%2Cai-engineer
