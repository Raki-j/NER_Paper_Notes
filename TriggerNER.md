# Abstract & Introduction & Related Work
- 研究任务
NER任务，如何以具有成本效益的方式获得监督
在标记数据数量有限的情况下，我们如何学习一个有效的NER模型？
- 已有方法和相关工作
	1. 为了实现低资源学习的NER，最近的工作主要集中在基于字典的远距离监督上。
	2.  一种方法专注于实例抽样和人类注释用户界面，要求工作人员首先注释最有用的实例
- 面临挑战
	1. 匹配句子的质量在很大程度上取决于字典的覆盖率和语料库的质量
所学的
	2. 学到的模型往往偏向于与字典中的实体有相似的表面形式。如果不在更好的监督下进一步调整，这些模型的召回率很低
	3. 最近的一项研究（Lowell等人，2019）认为，主动注释的数据在训练时几乎没有帮助
- 创新思路
引入了 “entity trigger” 来提高NER模型的标签效率，entity trigger 被定义为句子中的一组词，它有助于解释为什么人类会在句子中认出一个实体
- 实验结论
	1. 用了20%的 trigger-annotated sentences 达到了普通标注语句 70%的效果，高效

	2. 无论是基于字典的远程监督还是主动学习，都可以通过我们的框架在触发器增强的NER学习中使用。例如，我们可以使用高质量的语料库创建一个词典，然后通过要求人类注释者对为TMN设计的主动采样算法所选择的触发器进行注释来应用主动学习。我们相信，我们的工作为未来更经济有效地使用人类学习NER模型的研究提供了启示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab834e3a6c05400c9f07159c3a516f52.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_12,color_FFFFFF,t_70,g_se,x_16)


entity trigger的示意如下，我们将 "实体触发器"（或称触发器，为简单起见）定义为可以帮助解释同一句子中某一实体的识别过程的一组词语
![在这里插入图片描述](https://img-blog.csdnimg.cn/e96e3b98692c4dc7abd3d881a5d67efa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
entity trigger是必要的并且足够提示人去识别其相关实体，即使我们把实体用一个随机的词给mask掉
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d10fedb61d446e9bf2ec1017797d109.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
在众包数据集中，entity trigger的平均长度只有1.5-2 words
#  Trigger Matching Networks
句子表征和训练期间看到的触发器表征之间进行软匹配，就可以在推理时识别出无标签的句子。我们使用一个自我注意机制来进行这种软匹配

我们提出了一个简单而有效的框架，名为触发器匹配网络（TMN），由一个触发器编码器（TrigEncoder）、一个基于语义的触发器匹配模块（TrigMatcher）和一个基础序列标记器（SeqTagger）组成。我们的框架有两个学习阶段：第一阶段（第3.1节）联合学习TrigEncoder和TrigMatcher，第二阶段（第3.2节）使用触发器向量来学习NER标签。图3显示了这个pipeline。我们在第3.3节中介绍了推理的情况
![在这里插入图片描述](https://img-blog.csdnimg.cn/5895a504ef6846a6a28c2e33f70fa909.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
## Trigger Encoding & Semantic Matching
在第一阶段，我们建议联合训练触发器编码器（TrigEncoder）和基于注意力的触发器匹配模块（TrigMatcher）。编码器（TrigEncoder）和基于注意力的触发器匹配模块（TrigMatcher）使用一个共享的嵌入空间。

首先reformat每句话的格式，把每一个trigger都分离出来到一个单独的句子，对于每一个实例，我们把它输入到LSTM里面，得到每一个 $x_i$ 的上下文表示
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d2fde4d3fe54a62b49ea32f55798c6e.png)
为了得到基于注意力机制的表示，接下来
![在这里插入图片描述](https://img-blog.csdnimg.cn/16a0bfd82d124d2d9c38b2501e549f2e.png)
我们希望使用相关实体的类型作为监督来指导触发器的表示。因此，触发器向量 $g_t$ 被进一步送入一个多类分类器，以预测相关实体 $e$ 的类型（如PER、LOC等），我们用 $type(e)$ 来表示。触发器分类的损失如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/b4a6f1833b4a47adb5fdc813ce90e06f.png)
为了学会基于注意力的表征来匹配触发器和句子，我们使用对比损失，直觉来源于相似的trigger和句子应该有相似的表示。我们采样一些负样本，通过随机混合triggers和句子来训练，因为 trigmatcher同时需要正负样本来训练，对于负样本，我们期望跟正样本有一个距离为 m 的间隔

对比损失for软性匹配：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1b6e246f21fa44b28d2fe6cfd01e6e12.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/23b3f49cbdec41d9b4c65bd523d9f971.png)
## Trigger-Enhanced Sequence Tagging
我们将 entity trigger 作为注意力查询来训练用于NER的触发器增强序列标记器

如之前的图所示，将得到的 trigger vector跟整个句子做attention

Q， K分别是trigger vector和原句子，然后再用原句子作为V，得到trigger enhanced representation，最后将这个vector和原句子的vector concat起来

![在这里插入图片描述](https://img-blog.csdnimg.cn/adc96ba6809f40f5b203fb3e1ee69e05.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f022502454d74bdca8600477a5fb2ab5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_14,color_FFFFFF,t_70,g_se,x_16)
##  Inference on Unlabeled Sentences
对于不知道trigger的数据，使用注意力机制，对于每个未标记的句子x，我们首先计算它的自关注向量$g_s$ ，就像我们在训练 TrigMatcher 时做的那样。使用 $L2-norm$ 距离来计算对比损失，我们在句子和trigger vector 的共享嵌入空间中有效地检索出最相似的trigger

然后使用平均汇聚，得到 $g_t$
![在这里插入图片描述](https://img-blog.csdnimg.cn/50456893e22d43caa343662403abba6f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
# Experiments
![在这里插入图片描述](https://img-blog.csdnimg.cn/61420f0b7ae144d99918aea22289b3fb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/96290a7131af4d71ba689f9346c27dce.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8c7574704f3a4b00b39642f1e815b978.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix552h6KeJ55qEUmFraQ==,size_15,color_FFFFFF,t_70,g_se,x_16)
# Conclusion
在本文中，我们引入了 "实体触发器 "的概念，作为一种补充性的注释。单个实体注释提供了有限的明确监督。实体触发器注释加入了补充性的监督信号，从而帮助模型更有效地学习和概括。我们还在两个主流数据集上众包了触发器，并将把它们发布给社区。我们还提出了一个新的框架TMN，它可以联合学习触发器表征和具有自我注意力的软匹配模块，这样可以很容易地推广到未见过的句子，以标记命名实体。
TriggerNER的未来方向包括：
1）开发自动提取新型触发器的模型
2）将现有的实体触发器转移到低资源语言中
3）用更好的结构化归纳偏见来改进触发器建模（例如OpenIE）

# Remark
首先我觉得这个idea是很不错，但是，虽然说用20%的数据达到了别人70%的效果，但是最终用完整数据的对比没有放出来，大概率是效果没有提升，其次，说这样提高了效率，但是我想说的是，你众包搞那么多trigger难道就不麻烦了吗，我感觉比直接标更多相同质量的数据的代价要高，还有就是没有做跟bert crf的对比，作为一篇20年4月的paper，感觉有点不应该

终极总结，idea可以，但是是否work有待考究