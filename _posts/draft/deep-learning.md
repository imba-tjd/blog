https://www.youtube.com/watch?v=hfMk-kjRv4c

## 方向

* https://paperswithcode.com/sota
* https://pytorch.org/examples/

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
* 矩阵相乘ABC，(AB)C与A(BC)的运算量不同，先计算中间矩阵更小的那个，乘法次数更少

### 激活函数（非线性变换）

* 使用激活函数后，bias的含义不再是平移直线，而是一种触发激活函数的阈值
* ReLU：x >= 0 ? x : 0，或max(0,x)。负区间导数为0，可能导致神经元“死亡”；0点导数可人为设成0或1。Leaky ReLU：x<0时用x/100
* Sigmoid：二分类中使用，隐藏层中较少使用。SoftMax：多分类中使用
* tanh：也是S形的，且关于原点对称，值域-1到1。公式=2*Sigmoid(2x)-1
* SiLU：x / ( 1+ e^(-x) )。形状整体类似于ReLU，但可导。0点=0，x小于0时y小于但接近0
* SoftPlus：log(1+e^x)。形状整体类似于e^x，只是大于0时线性上升；也可以说类似于平滑ReLU

当损失不下降时，一种原因是梯度消失（梯度在反向传播过程中逐渐趋近于零，导致靠近输入层的参数更新缓慢）。\
当使用Sigmoid或Tanh等饱和型激活函数时，其导数在输入值较大或较小时趋近于零，导致反向传播时梯度迅速衰减，无法有效学习。解决办法是ReLU。\
ReLU对于某些输入也会输出零，称作“假死亡”，因为对于其它一些输入是正常的，这反而是一种Dropout。但如果初始化时随机数选得不好，导致大多数输出零，则无法有效学习。

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

### CNN 卷积(Convolution)神经网络

用于图像和视频。

输入：样本数 x 通道数 x 长 x 宽。

卷积核：如一个3x3的矩阵（每个元素是可学习的参数），在原图像上按 对应相乘再求和 输出一个值。\
卷积层：多个（如5个）卷积核，输出多个“通道”。\
卷积操作：往右移动1步或多步，一般是重叠的。
输出称作“特征图”

池化：降维操作，减少计算量，提升泛化能力。如取2x2的范围中的平均值或最大值。没有参数，不可学习。一般不重叠

现代卷积神经网络：AlexNet、ResNet、DenseNet。

### RNN 循环神经网络

MCP和CNN对于输入都是固定的，而RNN可以处理非固定长度的序列，且输入样本的顺序是有关的，如NLP、语音。\
序列建模：多对一，如情感分析输入文本，输出类别。一对多，如输入图像（固定），输出文字描述（不定长）。多对多，可以是每个输入都对应一个输出；也可以等输入全部完成后再输出，如翻译。\
输入形状：一般为 (batch_size, sequence_len, feature_len)。以下先假设不存在batch_size，且feature_len为1，即序列为单个数字。

结构：对于一个输入x1，计算 `z = act(w1*x1+b)`。此时若继续传播到下一个神经元Wo即预测输出。\
它还有另一权重w2，与新数据一起投入下一次的输入，称为**反馈循环**：`z2 = act(w1*x2 + w2*z + b)`，其中w1 w2 w3对于所有输入数据都是共享的。

若输入是向量（feature_len > 1），设长度为x，隐藏层长度为h。则w1形状为hx，w2形状为hh，b长h。\
组合w1 w2为[Whx:Whh]即(h, h+x)，一维连接输入和上一时刻的输出[x:h]即(x+h)。

缺点：不太适用于长序列，当w2>1时会梯度爆炸，w2<1会梯度消失。

多层RNN：第一层在某一时刻的输出，作为第二层**在相同时间步**的输入。不同层的参数不同，但对于不同时刻是复用的。

除了隐藏层循环连接，还有输出层循环连接，可以投入下一时刻的隐藏层或输出层。

TODO: GRU门控循环单元

### LSTM 长短期记忆神经网络

* 宏观上：RNN对于很久以前的事件和最近的事件，使用同一个反馈循环，来预测明天。而LSTM使用两条独立的路径预测明天。长期记忆是累加而非连乘，避免了梯度爆炸和消失
* 定义 长期记忆M（又称为Cell State） 短期记忆m（又称为Hidden State）。就是数字，m的值域为[-1,1]
* 一个计算百分比的单元：`P = sigmoid(w1*x + w2*m + b)`，x是输入的数据，m是短期记忆。以下每步中用到的P的参数不同。
* 一个激活函数为tanh、其余与P相同的单元T

1. “遗忘门”：M *= P，表示长期记忆还留下百分之多少。
2. “输入门”：M += P * T，表示创建多少百分比的“潜在长期记忆”。
3. “输出门”：m = P * tanh(M)，即更改短期记忆，且也是LSTM的输出。tanh(M)称为“潜在短期记忆”

在输入时，LSTM会展开(Unroll)成按顺序处理的单元。

M和m还有x其实也可以是向量；M和m和b的维度数相同，是超参数Hidden Size，不必与x的维度相同。\
为了做`W1*x + W2*m`，令m的维度为h，x的维度为i。W2的形状为hh，W1的形状为hi，m和x两个向量看作h1和i1的矩阵，相乘得到形状h1即h的向量。\
实际运算会将W1和W2横向拼起来，变为(h, h+i)；x和m拼起来，变为(h+i, 1)。\
sigmoid(向量)应理解为对每一项依次用sigmoid，因为它本身只接受标量。

### Word Embedding、Word2Vec

已经分词了，要对token编号。如果直接按顺序编，或者随机分配浮点数，意思相近的词不会聚集在一起。本模型解决此问题。\
首先有一个词汇表（如长度几千），有一些句子。目标是把所有单个词转换成固定长度（如128）的list[float]，且近义词的距离小，且类似于father-mother≈man-woman。

先对词表做OneHot编码，每个词变为 长len(词表) 只有自己为1 其他都为0 的向量；也可理解为按顺序编码每个词，作为下标。\
创建一个单隐藏层神经网络：输入 上一行 向量；隐藏层的神经元权重就是最终需要的嵌入，共有 len(词表) x 128 个权重，每个词有128个权重，没有偏置项；输出长度又是 len(词表)，再加softmax。\
训练时：依次输入句子中的每个词，target为句子中的下一个词。即期待预测下一个词。\
如句子里有A is good, B is good，则训练后A B的嵌入向量距离小。

负采样优化：①因为输入向量极其稀疏，其余词 0 x w 结果都是0，可以不用算。\
②在未训练时模型会输出 len(词表) 的随机向量，而target只有1位为1，可随机选择一些词（如20个）只正向传播计算和反向传播优化它们。

Word2Vec 使用“上下文”信息：①连续词袋：输入时向量变为 [1, 0, 1] 其中一个1是当前词，另一个1代表待预测词的下一个词，用来预测中间词。②跳跃模型：用中间词预测周围词。

### 原始Encoder-Decoder、Seq2Seq

Encoder将一整个句子或文章转换成一个固定长度的“上下文向量”，Decoder将向量解码成句子。两个句子之间长度可以不同，关键是可以具有不同的词表，如用于翻译。

二者都包含多层LSTM，第一层(Layer)LSTM的m不仅作为下一个时间的m，还作为第二层**在相同时间步**的输入。\
每层LSTM具有多个Unit，但似乎就是单个LSTM输出向量，视为多个输出标量的叠加，而非输出向量的叠加。

Encoder输入时，先将句子中的词，逐个编码成Embedding向量，按顺序输入；最后输入EOS。\
Encoder最后一层的最后一个的m（最终隐状态）是输出，称为“上下文向量”。连接Decoder时也将所有M和m称为上下文向量，对应传给Decoder的层。\
即Encoder处理整个文章，得到上下文向量，后续就和它无关了，与Decoder解耦。

Decoder的第一个输入是EOS，此时间经过Decoder计算，最后一层的m为输出，用一个全连接层（len(ctx_vec) x len(输出词表)）和softmax转换成词汇。该词汇再编码成Embedding作为下一个时间的Decoder的输入，直到输出EOS。\
训练时，不将Decoder的输出作为下一时的输入，而是输入“正确内容”；输出仅用于计算损失。且如果到了该输出EOS时未输出，直接停止训练。这称为Teacher Forcing。

使用LSTM作为Encoder-Decoder的缺点：
1. 编码阶段，所有输入信息都储存在一个状态中，描述能力有限。
2. 长距离衰减，句子靠前部分的影响降低。
3. 解码阶段，编码信息仅在第一时刻输入，随着序列推移，编码信息越来越弱。

TODO：gemini的说法，Encoder输入最后有EOS。Decoder输入先输入SOS，最后直到输出EOS结束。statequest视频没有说Encoder最后输入EOS，且Decoder最初输入SOS。

### 原始(标准)Attention

加在LSTM上，且是不可训练的。

LSTM可能会遗忘最初的序列，如 不要xxx 变为 要xxx 区别很大。Attention的思路是对于每个原始输入都增加了一条路径到当前输入。

Encoder的架构不变，但保留每个时间的输出（记为Eo）。

在Decoder中，单次输出（记为Do）与之前所有Eo依次点乘计算相似度（实际原理是余弦相似度，但分母仅用于归一，所以就省略了），\
再SoftMax一下变到[0,1]表示比例，再与各Eo对应相乘，表示优先考虑（加权）与Do最相似的Eo。\
再对于每一维将所有处理结果对应位置相加（不是将向量求和，向量长度始终保持Hidden Size不变）。\
再与Do一维拼接，放到全连接层解码。

### Transformer

#### Encoder

目的是将输入的与上下文无关的向量调整为相关的；每个输入的token都会产生一个向量：

1. 将Token编码为Embedding。假设长度为d。此时同一个词的向量是一样的
2. 位置编码：创建i个周期不同的sin/cos函数，传入当前token是第几个，与1相加。这样不同顺序向量就不同。RNN和LSTM天然隐式包含顺序信息，是时间上递归；而Transformer是非循环结构，本身不包含顺序信息
3. 自注意力层
  1. 创建三个权重矩阵Wq Wk Wv，形状一样都为dd，都与2点乘求和。注意此处是将向量求和，但因为又做了d次，又产生了长为d的向量；假设记为qkv
  2. 对于一个单词，将q与其他各个单词的k点乘，向量求和（即q*单个k -> 1个数），可选除以根号d进行缩放，一起组成向量传给softmax，得到与其他单词的相似度
  3. 加权：将所有单词的v对应与3.2的相似度相乘，再对应位置相加。得到AttentionOutput
4. 残差连接：2 + 3。或者另一个角度看3得到的是Δ2，即2的调整值。实际还会在2 3后加一个行归一层，二者都是使得训练更容易，减少梯度消失问题。

对于一份输入，所有Token都可以并行做1234。

#### QKV矩阵

* 有可能指Wqkv权重矩阵，也可能指权重与Embedding计算后的值
* qk的长度一般小于Embedding的长度d，如128。理解为将原始Embedding映射到一个低维空间
* q是某种询问，当q和k的方向对齐（点乘大）时，它们就匹配，称作那些k对应词的嵌入“注意到了”q对应词的嵌入。一个q可以查询多个k，得到的相似度
* 一般输入矩阵形状为(n,d)，其中n是token数（相当于样本）。Wq的形状为(d,128)，Wk一样。与Embedding计算后QK大小为(n, 128)。实际可以用一个Q查询所有K，即QKT大小为(i,j)，V为(j,)。V必须与输入一样(n,d)，但也可以低秩分解
* QKT -> 每一行是一个q对其余各个k的相似度，此表格称为“注意力模式”。之后再softmax理解为按行处理
* Wqkv对于所有输入都是相同的，称作自注意力单元
* 多头注意力MHA：简单理解为将qkv拆分成几份，分别计算。重点在于对于不同token并行拆分后是将第i份先连起来，之后再经过Wo
* 多查询注意力MQA：对于多头，Wq不变，Wkv共享一个。分组查询注意力GQA，每个组内共享Wkv

#### Decoder

先输入EOS。经过与Encoder完全相同的流程（参数不同）变成向量。\
再经过Encoder-Decoder Attention交叉注意力层，就是将Eo也考虑进来，经过KV，当前Decoder暂时的输出经过Q。\
得到的向量再经过残差连接，再全连接层解码。\
下一个时间，**输入之前的所有输出**（不同于LSTM的标准Attention），仅解码最后一个token。

##### Decoder训练和掩码

对于一个Token序列，将source设为BOS+它，target设为它+EOS。\
经过Attention层时，为了使后面词语的q只检查前面词语的k，将qk矩阵的下三角（不含对角线，即要处理自己的qk）设为-∞（原矩阵 + 下三角为-math.inf的矩阵），这样softmax后就变为了0。\
使得输出的Token可以一次性与target进行误差比较，且每个Token只会关注之前的输入，就好似逐字生成（推理时）一样。\
交叉注意力部分没有掩码。

#### 其它技术

* kv cache：对于Transformer架构的Decoder-Only和Encoder-Decoder模型的生成阶段，当前token之前的kv已经算过了，可以缓存。但Encoder-Only不需要，它一次处理所有序列；标准Attention也不需要
* LoRA：观察发现微调时，模型权重的变化往往是“低秩”的。假设要调整W，不直接调整，而是创建一个ΔW，固定W不变与其相加；再将ΔW拆分成两个矩阵AB，大小分别为(h,r) (r,h)，r远小于h。缺点：前向和反向传递的速度大约是原来的两倍
* 解决上下文窗口太小
  1. 训练
    1. 在训练阶段使用更长的序列数据
    2. 改进注意力机制：稀疏注意力、线性注意力
    3. 新的模型架构：Mamba
  2. 应用层面：RAG、滑动窗口或分块处理、摘要链、关键信息抽取（先用非LLM处理）。Qwen的RoPE、YaRN缩放技术（如果ctx小于3万不要启用）
* 生成策略
  * 贪心搜索的主要缺点：它错过了隐藏在低概率词后面的高概率词
  * 波束搜索：在每个时间步保留最可能的num_beams个词，最终选概率最高的序列，仍然是确定性的。在翻译中可以考虑使用
    * 使用：num_beams=5, early_stopping=True
  * 采样：非确定性，根据当前条件概率分布随机选词。选中低概率词后可能导致生成内容语义不连贯。
    * （降低）温度：增大原本概率高的概率
    * Top-K：先保留K个概率最大的，归一化，再采样
    * Top-P：在累计概率超过p的最小单词集中进行；当容易预测时，它会保留较少词
  * 其它生成选项：min_length 强制在达到它之前不生成EOS。num_return_sequences：返回多个结果，对于波束搜索各结果区别不大
* 分词器未登录词(out of vocabulary, OOV)问题
  * 基于词Word、基于字符Character：遇到不存在的会变为 UNK Token。如果Character包含所有Unicode字符，则太大
  * 基于子词Subword：Byte Pair Encoding (BPE), WordPiece, SentencePiece。如果一个完整的词语不在词汇表中，分词器会尝试分解成已知的子词单元组合。如 tokenization 可能分解为 token 和 ization。是Word和Character的中间形态。先分解到character(或byte)级别，再按merges.txt中的顺序合并，再按vocab.json映射成数值索引。不适合中文
  * 字节级别BPE：从根本上解决了任何OOV问题。简单来说就是以Byte的256种可能作为Fallback
  * Ċ和Ġ分别表示空格和换行符
  * tokenizer.json：前两个文件是它的子集，分别在model.vocab和model.merges。它还包括额外token、填充截断策略等。tokenizer_config.json：好像是tokenizer.json不包含词表的部分。tokenizer.model：好像也是必要的，有时没有是用了其他模型的
* 估算内存
  * bf16每个参数用2字节，训练时需要8字节，合计(2+8) * 7B = 70GB
  * Lora：1B的参数在整个微调过程中占大约1.4GB

#### GPT

因果语言模型(causal Language Models)，也被称为自回归语言模型(autoregressive language models)或仅解码器语言模型(decoder-only language models)。

自监督学习(Self-supervised Learning)：训练时使用下一个Token；实际并行输入，用掩码注意力。而Seq2Seq虽然也有Teacher Forcing，但它是监督学习。

#### Bert

双向（Bidirectional）Transformer编码器，能对输入序列进行双向理解。\
更擅长 自然语言理解NLU 任务，如文本分类（情感分析）、抽取式问答（填空题）。

两个预训练任务：
1. 掩码语言模型MLM，随机遮盖（替换为`[MASK]`）输入序列中的一部分（15%）词语，训练模型去预测这些被遮盖的词语。一个MASK只生成一个Token。
2. Next Sentence Prediction (NSP)：给出两个句子A B，让模型判断B是否是A的下一句话。目的在于学习句子级别的关系，比如问答或句子排序任务。输入格式为`[CLS] A [SEP] B [SEP]`，以CLS的状态二分类IsNext还是NotNext。

由于需要未来上下文，它们无法实时生成顺序输出。

生成Embedding的原理：CLS的最终状态。\
分类任务的原理：添加一个线性层，从CLS的隐藏状态映射到类别。\
MLM的训练原理：添加一个线性层，从MASK的隐藏状态隐射到Token。其交叉熵损失仅对于MASK计算。

#### MoE层

1. 门控网络 (Gate Network) 或路由网络 (Router): 这是一个小型的前馈网络或线性层，它接收输入，并决定将输入“路由”给哪些专家（通常是选择排名靠前的 k 个专家）以及如何组合这些专家的输出。这个门控网络有自己的参数。
2. 多个专家网络 (Expert Networks): 包含多个独立的子网络（即“专家”）。每个专家通常是一个前馈网络（例如两个线性层加一个激活函数）。所有的专家通常具有相同的架构，但它们内部的权重是独立训练的。

### Stable Diffution

是一个潜在扩散模型(Latent Diffusion Model)

核心组件：

1. Text Encoder文本编码器 将文本转换为Embedding。常用 CLIP (Contrastive Language–Image Pre-training) 的文本部分
  * CLIP：通过对比学习(Contrastive Learning)的方式，在大规模的图片-文本对数据集上进行预训练，学习 图像与其对应的文本描述（而非预定义类别），使得模型
理解图片和文本之间的语义关联。架构为 双编码器，包括图片编码器（ResNet和Vision Transformer）和文本编码器，将它们映射到同一个共享的嵌入空间。训练完成后就可计算文本和图片之间的相似度
2. U-Net:核心。执行图像的去噪过程。一种具有编码器-解码器结构的卷积神经网络。接收带噪声的潜在表示、当前的时间步信息（表示去噪的进度）以及文本嵌入作为输入，然后预测出添加到潜在表示中的噪声（或者预测去噪后的潜在表示）
3. 变分自编码器 (Variational Autoencoder, VAE)
  * 编码器 (VAE Encoder): 将原始图像（或初始的随机噪声）压缩到维度更低的潜在空间中。让复杂的扩散过程在计算成本更低的潜在空间中进行
  * 解码器 (VAE Decoder): 将在潜在空间中经过 U-Net 处理和去噪后的最终潜在表示，解码回高分辨率的像素空间图像

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
np.arange(15).reshape((3, 5))  用-1表示推断。不改变原来元素的数量和值，只按新维度依次填入

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
np.concatenate/concat((a1,a2))  对于二维数组，它的 axis=0 等于 np.vstack()，axis=1 等于 np.hstack()；torch还能用cat，维度用dim
np.insert(a1, ndx, a2) np.delete(arr, ndx)
添加新维度：[1,2,3][:, None] -> [[1],[2],[3]]; [1,2,3][None, :] -> [[1,2,3]]  此处None等价于np.newaxis

arr.argsort()  返回arr排序后对应原数组中的索引。如[3,1,2].argsort() -> [1,2,0]，表示原数组排序后的结果为 arr[1],arr[2],arr[0]。再加一次argsort()得到排名[2,0,1]。手动实现：将原数组转化为(元素值, 索引)，按值排序，返回索引

numpy out has performance benefits？

固定长度的字符串：dtype='<U10'
```

## Pytorch

* https://pytorch.ac.cn/
* https://pytorch.org/hub/ https://modelzoo.co/

### 安装

* pip install torch --find-links https://mirrors.aliyun.com/pytorch-wheels/cu130
  * CPU版：cu130改为cpu。Win默认为CPU，Linux默认为cuda
  * conda install pytorch-cpu 不知道要不要-c conda_forge
  * cuda版是自包含的，不需要装cuda toolkit，但需要装显卡驱动。用nvidia-smi查看支持的最高cuda版本
  * 不会传递装numpy
* 相关项目
  * intel_extension_for_pytorch(IPEX)：对于支持avx512的CPU能加速训练。仅linux或WSL2。不再更新，理由是它的内容已经集成进了pytorch 2.8
  * 优化超参数的框架，支持ML和DL框架：https://optuna.org/
  * https://pytorch.org/tnt training tools and utilities
* 环境
  * torch.cuda.is_available()、watch -n1 nvidia-smi、nvidia-smi dmon（每秒显示功耗温度等。原stats命令废弃了。还有一个pmon查看进程占用，普通GeForce卡用不了，必须不用于显示才行，即不能是WDDM模式，要TCC模式）、nvtop、nvitop。2.5支持intel的xpu，有xpu-smi
  * torch.set_default_device('cuda')，否则默认为CPU，要用if torch.cuda.is_available(): t=t.to('cuda') 或创建t时指定device
    * 通用：if torch.accelerator.is_available(): t.to(torch.accelerator.current_accelerator())
  * torch.manual_seed(42)

### tensor

```py
torch.tensor(lst)、from_numpy(np_array);t.numpy()二者共享底层
torch.rand(shape) torch.rand_like()
原地改变：x.copy_(y), x.t_()  要计算梯度时不能用
t.view(1, -1) # 转换为shape[1, sequence_length]  np也有view且效果完全不同，是改变dtype
```

#### 分页和非阻塞

* 分页：默认可以使用页面文件。创建时加pin_memory=True锁定在内存中
* 非阻塞：移到device时默认non_blocking=False，相当于每次调用后自动再torch.cuda.synchronize()。如果循环移动一批tensor，可设为True，之后手动执行同步
* 二者结合：已经pin的tensor再加非阻塞移动，加速很明显；未pin的不要t.pin_memory().to()否则会更慢；仅CPU->GPU且不修改原有tensor才不会损坏数据
* 单纯非阻塞效果不明显。单纯已经pin的效果还可以

### Module 和 正向传播

* F.relu()是调用函数。nn.ReLU是一个类(Module)，调用后创建函数实例
* nn.Linear 创建后会 自动随机初始化值、requires_grad=True
* 输入只支持mini_batch，如果只有一个输入，用t.unsqueeze(0)添加虚假的第0 batch维

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
2. 当模型计算出output_data后，调用.backward() （实际一般对loss调用），会触发autograd引擎以相反的顺序遍历计算图，计算出梯度，储存在那个需要梯度的tensor的 .grad 中。只有叶结点有，默认只能计算一次

* 计算原理：将输出tensor看作因变量，对想要的那个输入tensor链式求偏导。DAG在pytorch里是动态的，每次迭代重新生成，可配合控制流语句
* 如果参数是固定的，则设为False表示不需要优化（冻结参数）
* 获得共享底层参数但去掉梯度：t.detach()。画图时可能用到。另一种方式：with torch.no_grad()或函数上@torch.no_grad，前向传播时用到

### 优化器

* SGD：最基本的优化器。实现简单。收敛过程可能震荡，容易陷入局部最优或鞍点。对学习率敏感
* Adam：维护梯度的指数加权平均（一阶矩，提供动量方向）和平方梯度的指数加权平均（二阶矩，提供自适应缩放），进行早期阶段的偏差修正。从而自动调整学习率。AdamW：变种，推荐使用

```py
optimizer = optim.Adam(model.parameters(), lr=0.001)
loss_fn = nn.MSELoss() # 还有CrossEntropyLoss
for epoch in range(num_epochs): # 一般以下封装在train()，一个epoch先train()再test()，test里用model.eval()和no_grad
    loss_sum = 0
    for batch_idx, (data, labels) in enumerate(dataloader):
        outputs = model(inputs)
        loss = loss_fn(outputs, labels)
        loss.backward()  # 在tensor中累积梯度
        loss_sum += loss  # 按loss大小终止，不会自动累积
    if loss_sum < 1e-4: return
    optimizer.step()  # 一批统一调整
    optimizer.zero_grad()
    if batch % 100 == 0:
        print("loss: {:>7f}  [{:>5d}/{:>5d}]", loss.item(), (batch + 1) * len(X), len(dataloader.dataset))

按规则调整学习率：
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
  * datasetsearch.research.google.com 支持搜索其他平台
  * 国内：OpenDataLab、ModelScope
  * 注意许可协议

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

### 持久化

* 保存：torch.save(model.state_dict(), 'my_model.pth')
* 加载：model.load_state_dict(torch.load('my_model.pth', weights_only=True))
* 导出onnx：pip install onnx onnxscript; o=torch.onnx.export(model, example_inputs, dynamo=True); o.optimize(); o.save('model.onnx')

### torch.compile(dynamo)

* 需要g++。使用后要空生成一次编译内核，速度很慢
* 实际上支持任何函数，也可以用作装饰器。会递归编译，一般在顶层使用，排除不兼容的或用某上下文管理器关闭；也可以从底层开始测试
* 在执行时将模型编译成优化的内核，多次执行才有优化效果
* mode=可选reduce-overhead和max-autotune，默认二者平衡
* fullgraph=默认False，设为True如果失败会抛异常
* model的grad_fn可以看到是否compile过
* 只有V100 A100 H100才能看到明显效果
* 保存编译结果
  * 无法按模型级别保存。但有一些全局缓存
  * 先调用一次触发编译。torch.compiler.save_cache_artifacts()，把bytes持久化保存，之后torch.compiler.load_cache_artifacts()
  * 在本机上：TORCHINDUCTOR_FX_GRAPH_CACHE=1 TORCHINDUCTOR_AUTOGRAD_CACHE=1 默认存放在 /tmp/torchinductor_username

### 量化

* API：未来优先用PyTorch 2 Export量化，不要用FX Graph Mode量化，老版方法是Eager Mode量化
* 类型：训练后(PTQ)动态（权重量化，激活浮点，最简单，适合NLP）、训练后静态（权重和激活都量化，训练需要校准，适合CNN）、静态量化感知训练（AWQ，最准）
* torch.quantization.quantize_dynamic(model, {nn.Linear 要量化的层}, dtype=torch.qint8)
* 后端：不同后端适合不同设备，支持的算子不同，如arm cpu用qnnpack，还要再对模型设定一下。目前支持x86和arm cpu，gpu的tensorrt在beta
* 输入的tensor也要量化。一般修改模型，init加self.quant = torch.ao.quantization.QuantStub()，forward第一句x=self.quant(x)

## torchvision

* 预训练模型：m = models.densenet121(weights='IMAGENET1K_V1'); m.eval() 但是输出结果映射回类别要另外下一个json

### datasets 预定义的数据集

```py
from torchvision import datasets
training_data = datasets.FashionMNIST(
    root="data", # 储存路径
    train=True,  # 指定False返回测试数据
    download=True,
    transform=ToTensor() # 每次取单条数据时对feature执行的转换
    target_transform=Lambda(lambda y: torch.zeros(10).scatter_(0, torch.tensor(y), 1)) # 转换label，此处为OneHot
)
DataLoader(training_data, batch_size=64, shuffle=True, pin_memory=True)  # 不推荐改num_workers
```

### transforms

含有许多“图片处理工具”。一般用Compose([创建多个工具类实例])创建可复用的处理流，再(调用)

```py
from torchvision import transforms
trans=Compose([Resize(255),CenterCrop(224),ToTensor(),Normalize(mean=[0.5, 0.5, 0.4],std=[0.2, 0.2, 0.2])])
```

### 加载图片

```
from PIL import Image; import cv2（包名是opencv-python-headless）

im = Image.open(path) = cv2.imread(path)  im.shape 0->(高,宽,通道)
im_tensor = transforms.ToTensor()(im)
Image.fromarray(nparr.astype('uint8')).show()
降采样，用步长：im[::10,::10,:]。翻转：im[::-1]。裁剪：切片 im[a:b,c:d]

OpenCV的默认通道是BGR颜色模型，转换成RGB：img[:, :, [2, 1, 0]]

摄像头：
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 224); cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 224); cap.set(cv2.CAP_PROP_FPS, 36)
ret, img = cap.read()

显示图片：
npimg = img.numpy()
plt.imshow(np.transpose(npimg, (1, 2, 0)))
```

## 示例CNN模型

```py
class MyCNN(nn.Module):
    def __init__(self):
        super(MyNet, self).__init__()
        self.conv1 = nn.Conv2d(in_channels=1, out_channels=32, kernel_size=3, padding=1) inchannel输入通道数，灰色为1；outchannel输出多少个特征图，也代表有多少个卷积层；卷积核大小还可以设成=(3,3)
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
        x = torch.flatten(x,1) # 保持第0维不变，从1到-1维 展平连接。因为全连接层只接受1维数据（第0维是batch数）
        x = self.fc1(x); x = F.relu(x)
        x = self.fc2(x); x = F.relu(x)
        x = self.fc3(x)
        output = F.log_softmax(x, dim=1) # 产生probabilities(可能性,概率)，用于分类
        return output

self.pool1 = MaxPool2d((2,2), stride=(2,2))
kaiming_uniform_(self.conv1.weight, nonlinearity='relu') 一种初始化方法
xavier_uniform_ 用于初始化全链接层
```

## 分布式理论

https://github.com/PacktPublishing/Distributed-Machine-Learning-with-Python

* 数据并行：不同设备处理不同数据样本，但模型结构相同
* 张量并行：拆分模型内部张量（如矩阵乘法）到多个设备。用于模型层太大，一张卡放不下
* 流水线并行：拆分模型层到多个设备，一个GPU处理一部分层，数据样本“流动”处理。如同一时间GPU1处理样本1的1-4层，另一个GPU处理样本2的5-8层

### 数据并行（训练）

* 数据加载带宽和模型训练带宽之间不匹配
* 解决：拆分（不相交）数据集到多GPU上
* 新问题：如何同步。解决：①随机梯度下降SGD优化器（且初始化时用相同的种子）。②模型同步（有多种方案）。③超参调整：batch_size、学习率
* 实现：if torch.cuda.device_count() > 1: model = nn.DataParallel(model)

#### 模型同步

1. 采用一个(组)中心节点作为“服务器”，工作结点拉取参数，训练后提交。缺点：服务器带宽瓶颈；如果用多个服务器，则会变复杂。
2. All-Reduce架构：只有工作节点。一轮训完全汇总到一个，再广播。Ring All-Reduce（NV NCCL是其理论的实现）：将所有节点视为环，每个节点接收上家发来的，与自己的合并，发给下家，走完一圈后最后的节点拥有总和，再走一圈同步
3. All-Gather：每个节点都广播自己的值，并且接收所有其他节点。数据传输量远超All-Reduce

### 模型并行（LLM推理）

TODO

## 混合精度训练

* 结合FP32和FP16。参数和Loss仍用FP32，正向和反向传播运算用FP16
* 如果全用FP16训练，容易出现溢出、梯度爆炸等问题

```py
from torch.cuda.amp import autocast, GradScaler # Automatic Mixed Precision
scaler = GradScaler()

for data, target in dataloader:
    data, target = data.cuda(), target.cuda()
    optimizer.zero_grad()
    with autocast(): # 训练时用，自动将运算转换为FP16
        output = model(data)
        loss = loss_fn(output, target)

    scaler.scale(loss).backward() # 放大loss避免反向梯度过小下溢
    scaler.step(optimizer) # 它先unscale（缩小），判断是否溢出，如果梯度值不是inf或nan，再optimizer.step()
    scaler.update()
```

## Lightning

```py
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
```

## 其他项目

* 可视化模型（各层和图）：https://github.com/lutzroeder/netron
* torchtriton：torch.compile的同类
* triton：类似py的DSL，不用懂CUDA也可以GPU编程，只优化operator算子，基于MLIR
* tvm：深度学习编译器，支持模型格式，编译到可执行代码，高低层都优化


https://zh.d2l.ai/chapter_introduction/index.html
循环神经网络：RNN,GRU,LSTM,seq2seq。优化算法：SGD,Momentum,Adam
《动手学深度学习》实体书
深度学习入门 : 基于Python的理论与实现     9787115485588


https://learn.microsoft.com/zh-cn/training/paths/get-started-with-artificial-intelligence-on-azure/
https://learn.microsoft.com/zh-cn/collections/r47ni8gp1eqgqp?ref=collection&listId=1nx4c0zqyoodor&sharingId=6A9F03F25E12DA9E&wt.mc_id=aisc25_landingpage_wwl
https://learn.microsoft.com/zh-cn/training/browse/?wt.mc_id=aiml-7486-cxa&subjects=artificial-intelligence
https://learn.microsoft.com/zh-cn/training/browse/?expanded=azure&roles=ai-edge-engineer%2Cai-engineer
