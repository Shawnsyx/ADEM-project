&emsp; **从Theano到Pytorch的迁移 （一）**

# 前言

&emsp;原项目Theano的github地址：[https://github.com/julianser/hed-dlg-truncated](https://github.com/julianser/hed-dlg-truncated "https://github.com/julianser/hed-dlg-truncated")

&emsp;论文地址: [https://arxiv.org/abs/1605.06069](https://arxiv.org/abs/1605.06069 "https://arxiv.org/abs/1605.06069")

&emsp;首先是对模型**ADEM**介绍以及对评估模型的学习。 ADEM，short for "automatic dialogue evaluation model”，对话自动评估模型。模型在利用RNN（recurrent neural network，循环神经网络）半监督式（semi-supervised)训练下来评分。

有关RNN的学习就不浪费时间整理已有的东西，重点精力放在我在此项目的总结与感受。有关RNN的开篇学习，我参考了这篇[https://blog.csdn.net/zhaojc1995/article/details/80572098](https://blog.csdn.net/zhaojc1995/article/details/80572098 "RNN")，markdown有关数学公式的编辑参考了这篇简书[https://www.jianshu.com/p/e74eb43960a1](https://www.jianshu.com/p/e74eb43960a1 "Markdown数学公式语法")

# The aim of the project

&emsp;NLP中重要一环是对“dialogue”的评价，而dialogue中包括reference（人工回复作为“参考”）和candidate（模型给出的答复），基本思路是利用一些评估标准来对比candidate和reference，并给出一个评分。本项目就是基于原有的评估思路上，给定一系列responses，得到一个准确的人工评分。responses要有一定的多样性，包含相关、不相关的，也需要包含连贯性强和连贯性较弱的内容。

&emsp;为了达到上述的“多样化的responses”，研究人员们采用了4种模型产生的candidate responses。
1. TF-IDF retrieval-based model
2. Dual Encoder(DE)
3. responses generated using the hierarchical recurrent  encoder-decoder (HRED) model 
4. human-generated models

# 评估方式

## Word-Overlap Metrics
&emsp;利用词语的覆盖程度来计算。其中一个最广为使用的当属BLEU和METEOR，两者广泛应用于评价机器翻译的方法当中。但是机器翻译指标与人类评估结果还存在较大差异，尤其在对话系统中，尽管一个对话的responses可以跟reference词语重叠很少（导致Word-Overlap评分很低），但可以被人类评价为一个“好的回复”。

&emsp;[BLEU](https://www.cnblogs.com/jiangxinyang/p/10523585.html "BLEU")尽管可以采用n-gram方法来优化评价方式，但在对话评价上有自己的局限性。机器翻译工作中词语选来选去大多不离那些candidate，所以bleu可以很好的被应用。但是在对话当中，与reference词语重叠程度小，也可以是好的回复，这里如果仅以BLEU评分作为评估就显得不合理了。而且回复也需要考虑前后的语境（context）。

## ADEM
&emsp;Automatic Dialogue Evaluation Model是为了弥补word-overlap metrics的缺陷。而且（1）能够捕捉到语义相似性 （2）挖掘语境和参考回答对于评分模型的作用。



