# Abstract & Introduction & Related Work
一个树状层级的粗粒度到细粒度的模型，拥抱了本体论结构（？这是什么）
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d9e73cfd5e84bce97bddba78438b692.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
- 研究任务
hierarchical entity classification
- 已有方法和相关工作：FET的进展主要集中在以下方面
1. 更好的mention表示
2. 融入层次结构
- 面临挑战
研究人员提出了替代的FET方法，其类型不在类型层次中形成。
- 创新思路
提出了一种新的方法，将明确的本体结构考虑在内，通过多层次的学习排名方法，以给定的实体提法为条件对候选类型进行排名，直观地说，较粗的类型更容易，而较细的类型则更难分类：我们通过在排名模型的每一级允许不同的边际来捕捉这一直觉

- 实验结论
achieves state-of-the-art results according to multiple measures across various commonly used datasets.

# Method
## Mention Representation
首先，使用mention中的词的表征，得出一个mention表征。我们在线性转换后，在mention的顶部应用一个maxpool层

![在这里插入图片描述](https://img-blog.csdnimg.cn/c41fd0ee898547ab8c1d1645f1bd0d63.png)
然后使用注意力机制得到mention to context的表示
![在这里插入图片描述](https://img-blog.csdnimg.cn/4a61b13b23a04f0fbcd536b7838f0d54.png)
最后将mention的representation和context vector concat起来
![在这里插入图片描述](https://img-blog.csdnimg.cn/975b88201a6d48cea734c614832cc629.png)
##  Type Scorer
用一个两层的fc，并与y的 type embedding 来学习一个 mention 与类型 y 之间的 embedding
![在这里插入图片描述](https://img-blog.csdnimg.cn/878b89be1133458981a7cca34a927b42.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)
## Hierarchical Learning-to-Rank
本文提出了一个novel的 hierarchical learning-to-rank loss
1.  允许进行自然的多标签分类
2. 考虑层次化的本体

首先从多类hinge loss开始，将正样本排在负样本上面

这是用ranking SVM，模型学会去将正样本的y的排名高于一个 $\xi$ 的间隔，如果是线性SVM，这个间隔必须设置为1，依赖L2正则化来限制type embedding
![在这里插入图片描述](https://img-blog.csdnimg.cn/bddec81be48b4121ae9b1c15cfdbab32.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)
### Multi-level Margins 

然而，这种方法认为所有的候选类型都是 flat，而不是 hierarchical ,所有的类型都被给予同样的处理，没有任何关于它们在类型层次中的**相对位置的先验**。

我们通过以下方式对这一直觉进行编码：
（1）学习只在类型树的**同一层次**上对类型进行排序
（2）对**不同层次**的排序模型设置不同的间隔参数
![在这里插入图片描述](https://img-blog.csdnimg.cn/a5b25a3205ea49928f9b7f9ddade641a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)
更上层的间隔应该要比下层的大，因为我们的模型应该能够在更容易的配对之间学习更大的间隔：我们表明，这比在我们的实验中使用单一的间隔要好
![在这里插入图片描述](https://img-blog.csdnimg.cn/d90c0ffa182a4b9ea14d4c4a24ee3af0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)
### Flexible Threshold
给定父节点的情况下，模型应该学习到接上一个**正确的子节点的置信度要高于其他的负样本**

总结就是："一个正子样本应该比它的父类型排名高，而它的父类型应该比它的负子样本排名高"
![在这里插入图片描述](https://img-blog.csdnimg.cn/aa532d3227084b29931bd0d3ce972e64.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0412e1ae65a847dda7698b35545571c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)
由于我们在中间插入了父节点，我们把间隔分成两部分
![在这里插入图片描述](https://img-blog.csdnimg.cn/22348e63ebde44ca885890b01d98e22f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/536812ac6ede43c9acdad597de5d3b2c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)
1. 正样本的的分数应该比所有的父节点高
2. 父节点的分数应该比所有负兄弟样本节点高
3. 正>负

分别由以下系数决定：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9fe13867e1dd4645826058e7a3bdfb01.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)

##  Decoding
![在这里插入图片描述](https://img-blog.csdnimg.cn/76d97eb502ab403c9349a172b9e8f80f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)

##  Subtyping Relation Constraint
本体中的每个类型y∈Y都被赋予一个类型嵌入y∈R dt。我们注意到类型上的二元子类型关系 " <: "⊆Y×Y的类型。Trouillon等人（2016）提出了关系嵌入方法ComplEx，它能很好地处理反对称和传递性关系，如子类型。它之前已经被运用于FET中--在Murty等人（2018）中，ComplEx被添加到损失中以调节类型嵌入。ComplEx在复数空间中操作--我们使用实数空间和复数空间之间的自然同构，将类型嵌入映射到复数空间中（嵌入向量的前一半为实数部分，后一半为虚数部分)
![在这里插入图片描述](https://img-blog.csdnimg.cn/86310da1e86045f5a20098ab91f3421c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/05258a23c8c04b91ab0c57e581e01e73.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
给定实例$(x,Y)$，对于每个正类型 $y∈Y$，我们学习以下关系：
![在这里插入图片描述](https://img-blog.csdnimg.cn/8a0fa2b9a2564aeca2decc8c2c610b21.png)


将这些关系约束转化为原始SVM下的二元分类问题（"是或不是一个子类型"），我们得到一个hinge loss
![在这里插入图片描述](https://img-blog.csdnimg.cn/717a7c32702e4771bd2cde6c29703840.png)
比交叉熵损失函数效果更好，是由于训练对的结构决定的

训练对的结构：我们使用兄弟姐妹和父母的兄弟姐妹作为负样本（这些类型比较接近于的类型），因此在训练中使用了更有竞争力的负样本。

## Training and Validation
我们的最终损失是分层排名损失和子类型关系约束损失的组合，并使用L2正则化
![在这里插入图片描述](https://img-blog.csdnimg.cn/55d7c8766bf1407d8dd3d8f0c72ede42.png)
优化器使用AdamW，在L2正则化的情况下比原始Adam更优

# Experiments
![在这里插入图片描述](https://img-blog.csdnimg.cn/dec421e62aea4b628c9271fce784d6e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f031e3c08ee94602a38709002b672e56.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb701da8648041afb73eae976aa75c8f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
#  Conclusions
我们提出了(i)一种新的多层次学习到损失函数，在类型树上操作。和(ii)一个伴随的从粗到细的解码器
以充分接受本体论结构的类型的本体结构，进行分层实体打字。我们的方法在各种数据集上取得了sota，并在严格的准确性上取得了实质性的改进（4-8%）

此外，我们主张仔细研究部分类型路径：它们的解释依赖于数据的注释方式，并反过来影响类型的性能

# Remark
第一感觉就是，这篇paper真的很抽象很难读懂，一字一句读了很久，感觉自己还是理解的不到位，属于是我目前读过的paper里面最复杂的，method部分满满的三页

方法我觉得并不novel，用树形结构从粗粒度到细粒度，但是能实现出来并让它work确实很厉害，这就是约翰霍普金斯大学的含金量吗orz