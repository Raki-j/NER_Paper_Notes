# Abstract & Introduction
-  研究任务
flat 和 nested NER tasks

-  已有方法
	1. 序列标注模型，以CRF为骨干
	2.  解析树，他们做了一个假设，即当一个mention与另一个mention重叠时，它们完全被另一个mention所包含
	3.  mention hyper-graphs，用神经网络来学习超图表示
	4. 以分层的方式动态地堆叠平面NER层
	5. 通过对嵌套实体mention的头部驱动的短语结构进行建模和利用，形成锚点-区域网络（ARNs）架构
	6. 通过选择置信度最高的的实体跨度并将这些节点与信心加权的关系类型和核心关系联系起来，建立了一个跨度列举方法
	7.  基于BERT的模型，它首先将标记和/或实体合并成实体，然后给这些实体分配标签。
	8. 推理模型，从最外层的实体到内层的实体进行反复提取
	9. 将嵌套式NER视为一个序列-序列生成问题，其中输入序列是一个标记列表，目标序列是一个标签列表。
- 面临挑战
在嵌套NER任务中，无法很好的处理实体重叠的问题
- 创新思路
 把NER任务转化成机器阅读理解任务
- 实验结论
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a569a43f9c2493d83e30e8bd6129114.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)


# Method
##  Query Generation
 在本文中，将注释指南说明作为构建查询的参考。注释指南说明是由数据集建立者提供给数据集注释者的指南。它们是对标签类别的描述，这些描述尽可能地通用和精确，以便人类注释者可以在任何文本中对概念或提及的内容进行注释，而不会遇到歧义。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8772e697ee9a41029e824d6f652e92b8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_15,color_FFFFFF,t_70,g_se,x_16)
## model
### model backbone
使用bert作为backbone，将query concat到一起
![在这里插入图片描述](https://img-blog.csdnimg.cn/f4d9a5b119e64140be6218124f8e1f49.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
### span selection
每个token上有两个二元分类器，一个是预测每个标记是否是起始索引，另一个是预测每个标记是否是结束索引。
#### Start Index Prediction
鉴于BERT输出的表示矩阵E，模型首先预测每个标记是起始索引的概率，如下所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/d208c65ed0224a6ea27134d11a58599c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_15,color_FFFFFF,t_70,g_se,x_16)
#### End Index Prediction
跟start index的方法一模一样，用自己矩阵 $P_{end}$
#### Start-End Matching
由于实体之间存在重叠，使用匹配最近两个开始和结束的方法并不work，所以需要一个方法将start和end匹配起来
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2811065226647309f753d47df17f50c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
上标代表矩阵中的一行，给出一个start index和一个end index，训练一个二元分类器判断是否匹配
![在这里插入图片描述](https://img-blog.csdnimg.cn/b09803b638d544adbcd98c564e361a59.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)

## Train and Test
算三个交叉熵：起点，终点，起点和终点
![在这里插入图片描述](https://img-blog.csdnimg.cn/be9fa247518e4011ba9fbf5ad5f39431.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/69a83bd7cd4f49c39807c6e998d3a9d7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)
三个系数为超参数


...
# Experiment
##  Results
![在这里插入图片描述](https://img-blog.csdnimg.cn/f6c63bec2c6b4d43a8311dcb17b318ad.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b0fc26bfdc54932b5db1e0bfe1f5612.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)

#  Ablation studies
##  Improvement from MRC or from BERT
实验表明MRC模型对NER任务是有提升的
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8ec0555f7a64f6998c2d9c974f66674.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a26437564964453b097376e0695efdd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)


## How to Construct Queries
如何构建查询对最终结果有重大影响，实验表明最优的方法是Annotation guideline notes（注释准则说明）
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f51abb9a3cc42258ae2ee143076382d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_11,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e550425c21ea48acaafe4dffad187a76.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)
##  Zero-shot Evaluation on Unseen Labels
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ab659ab851645fdbf7a16eb21f27654.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
## Size of Training Data
![在这里插入图片描述](https://img-blog.csdnimg.cn/abdf2ec9c99e4049bd3726517a9dfc1f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)

# Conclusion
在本文中，将NER任务重新表述为MRC问题回答任务。这种形式化有两个关键优势。
(1)能够处理重叠或嵌套的实体；
(2)查询编码了关于要提取的实体类别的重要先验知识。
所提出的方法在 nested 和 flat NER数据集上都获得了SOTA结果

# Remark
用阅读理解的方式做NER task，对于我这个没见过世面的小菜鸟来说确实是很novel的方法，然后效果也work，在那个时间点达到sota，但是之后的sota模型就直接碾压了它好几个点，再者不知道这个方法的效率如何，但是方法总体来说比较简单，idea新颖，总之个人觉得是篇好paper。