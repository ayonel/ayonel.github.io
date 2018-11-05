---
title: 用weka进行多项式朴素贝叶斯增量学习
url: 73.html
id: 73
tags:
  - 机器学习
date: 2016-08-12 15:14:41
---

&#160; &#160; &#160; &#160;有时候我们进行贝叶斯分类时，由于数据量太大导致内存溢出或者对模型的训练有着特殊要求(比如用第一个月的数据预测第二个月，再将第二个月的数据加入已经训练好的模型，去预测第三个月...)，这时普通的贝叶斯分类不行了。我们需要使用贝叶斯来进行增量学习（incremental learning）, weka的贝叶斯分类器即存在增量式的贝叶斯分类器。下文以多项式朴素贝叶斯增量学习为例，weka中的包为：
`weka.classifiers.bayes.NaiveBayesMultinomialUpdateable` 
&#160; &#160; &#160; &#160;下文是核心代码：
```
 //获取第一个月的数据用于建立模型
        ArffLoader loader = new ArffLoader();
        loader.setFile(new File("month1.arff"));
        loader.getStructure();//这句话必须有，不然会报错

        //定义分类器 并用第一个月的数据做训练
        Instances originDataSet = loader.getDataSet();//获取第一个月的实例
        originDataSet.setClassIndex(originDataSet.numAttributes()-1);//定义最后一个属性作为分类目标
        NaiveBayesMultinomialUpdateable nb = new NaiveBayesMultinomialUpdateable();//定义模型
        nb.buildClassifier(originDataSet);//建立模型

        //对从第二个月起的每个月进行预测和训练,一直到第100个月,(2,100只是用于示例)
        for (int month = 2; month <= 100; month++) {
            //读取测试集
            ArffLoader testLoader = new ArffLoader();
            testLoader.setFile(new File("month"+i+".arff"));
            testLoader.getStructure();

            Instances testDataSet = testLoader.getDataSet();//获取测试集实例
            //进行分类，并且输出
            for (int i = 0; i< testDataSet.numInstances(); i++) {
                FileWriter fileWritter = new FileWriter("output.txt", true);
                BufferedWriter bufferWritter = new BufferedWriter(fileWritter);
                bufferWritter.write(nb.distributionForInstance(testDataSet.instance(i))[1]+'\n');//这一行输出的是分类中分到第二类(下标为1)的概率
                bufferWritter.flush();
            }

            // 再把这次的测试集当做训练集进行训练
            for (int i = 0; i< testDataSet.size(); i++) {
                testDataSet.setClassIndex(testDataSet.numAttributes()-1);
                nb.updateClassifier(testDataSet.get(i));
            }
```