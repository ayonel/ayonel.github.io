---
title: 使用R语言进行多元混合效应逻辑回归
url: 110.html
id: 110
tags:
  机器学习
date: 2016-08-26 11:56:03
---

&#160; &#160; &#160; &#160;多元混合效应逻辑回归（Mixed Effects Logistic Regression）是什么：
&#160; &#160; &#160; &#160;混合效应逻辑回归是一种二分类模型，其输出是一组预测变量(自变量)的线性组合，但是样本不是简单地独立的，而是集群式分布，也即某个群体之间存在内部关联。
> Mixed effects logistic regression is used to model binary outcome variables, in which the log odds of the outcomes are modeled as a linear combination of the predictor variables when data are clustered or there are both fixed and random effects.）
 
举个例子：
> A large HMO wants to know what patient and physician factors are most related to whether a patient's lung cancer goes into remission after treatment as part of a larger study of treatment outcomes and quality of life in patients with lunger cancer.

一个医疗机构想要知道对于骨瘤是否最终治愈，医生和病人之间哪种属性是影响最大的。比如医生的属性：从业经验，学历等等。病人的属性有骨瘤大小，患病时间，性别等等。 我们现在就会有一组样本：

```
s_cured    doctor  experience    education    patient    tumour_size    tumour_time    patient_sex
   1         Bob      10            1          jav          3              2                 0    
   0         Bob      10            1          hex          4              1                 1
   1         Bob      10            1          lancy        6              4                 0
   1         Jack     5             3          joogh        1              2                 0    
   0         Jack     5             3          silla        4              1                 1
   0         Jack     5             3          grecy        6              4                 0
   1         Jack     5             2          lori         2              2                 0    
   0         Bob      10            2          carl         5              1                 1
   1         Bob      10            2          sharun       6              4                 0
```
我们现在有这组数据，我们现在要预测一个骨瘤病人能否治愈。
普通的多元线性逻辑回归会把每一组样本看成独立的，也就是样本之间是完全独立的，但是实际中我们可以看到，这些样本之间其实是聚集分布的，它们并不是完全独立。同为Bob医生的样本之间肯定存在某种关联，同为Jack医生的样本之间也肯定存在关联。多元混合效应逻辑回归就是加入一个随机效应(上面这个例子中就是doctor),弥补了普通线性回归视样本之间并不存在关联的不足。

混合效应线性回归更加贴近于真实的世界环境。UCLA有十分棒的[教程](http://www.ats.ucla.edu/stat/r/dae/melogit.htm)，大家可以去读一读。
那么如何在R里面使用混合效应回归呢？我们以混合效应逻辑为例。
* 读入数据：
`>data <- csv.read("data.csv")`
对数据中的计数类型（比如年龄啊，从业经验啊等等）进行正规化。为什么需要正规化，请自行谷歌。
`>pvar <- c("experience","education","tumour_size,tumour_time") >datsc -< data >datasc[pvar] <- lapply(datasc[pvar], scale) //正规化`

* 引入我们的核心计算包lme4:
`>require(lme4)`

* 进行计算：
`>result <- glmer(is_cured ~ experience+education+tumour_size+tumour_time+(1|doctor),data=datsc,family=binomial,control=glmerControl(optimizer="bobyqa")) //随机变量doctor放在(1|doctor),其余变量为固定变量，optimizer需要加上，防止不收敛`

* 输出结果：
`>summary(result)`