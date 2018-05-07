# 天池美年AI双高疾病风险预测第一赛季

## 赛题描述

双高疾病风险预测，要求参赛者通过双高人群的体检数据来预测人群的高血压和高血脂程度，以血压、血脂的具体数值为指标，设计高精度，高效，且解释性强的算法来挑战双高精准预测这一科学难题。

## 赛题数据

两个特征文件 data_part1 和 data_part2，每个文件第一行是字段名，之后每一行代表某个指标的检查结果（指标含义已脱敏）。每个文件各包含3个字段，分别表示病人id、体检项目id 和体检结果，部分字段在部分人群中有缺失。其中，体检项目id字段，数值相同表示体检的项目相同。 体检结果字段有数值和字符型，部分结果以非结构化文本形式提供。

标签文件 train.csv，是训练数据的答案：包含六个字段，第一个字段为病人id，与上述特征文件的病人id有对应关系，之后五个字段依次为收缩压、舒张压、甘油三酯、高密度脂蛋白胆固醇和低密度脂蛋白胆固醇。

## 评估标准

参赛选手需要提交对每个个体的收缩压，舒张压，甘油三酯，高密度脂蛋白胆固醇和低密度脂蛋白胆固醇五项指标预测结果，以小数形式表示，保留小数点后三位。该结果将与个体实际检测到的数值进行对比。对于第j项指标，计算公式如下：
$$e_j=\frac{1}{m}\sum_{i=1}^{m}\left ( log\left ( {y_i}'+1 \right )-log\left ( y_i+1 \right ) \right )^{2}$$
其中m为总人数，yi'为选手预测的第i个人的指标j的数值，yi为第i个人的指标j的实际检测值。最后的评价指标是五个预测指标评估结果之和：
$$E=\frac{1}{5}\sum_{n=1}^{5}e_n$$

## 数据预处理

### 数据拼接与重构

由于比赛官方给出的两个特征文件的数据结构是vid,table_id,value的表结构，不适合对每一项特征进行处理，因此需要先解决数据结构的问题。首先拼接两个特征文件，并重塑数据表结构，将vid以及具体的特征名作为column。代码详见preProcess.py

### 数据缺失

由于医疗领域的特殊性，会有相当多的的检查项item缺失，我们选择删去少于400条记录的特征。接下来针对数值型数据与文本数据分别进行预处理。

### 文本数据处理

本题中文本数据主要来源于data_part1，主要包括体检检查报告，影像学报告。由于data_part1中大部分数据比较工整，我们使用正则去人工筛选了与双高可能相关的共61项类别特征。具体代码与特征处理方法详见preProcess.py

### 数值数据处理

本题中的数值数据特征占了绝大多数，经过初步的缺失特征的筛选，数值型特征剩余约420个。这些数值特征的数据中也包含了大量的数据噪声，比如"70-80"，这里需要将其清洗为标准数值型数据。这里针对具体的噪声数据制作了映射字典，具体字典以及噪声处理代码详见preProcess.py

## 模型

这里使用了LightGBM进行回归建模。本题需要对5个指标进行回归，5个指标的损失共同作为评价标准。这里针对每个指标分别建模，因为子问题之间是相互独立的，只需要对5个指标分别寻优即可。
考虑到甘油三酯以及低密度胆固醇的长尾效应比较严重，模型甘油三脂和低密度胆固醇的高值的拟合效果不佳，这里使用了log trick，即在输入之前将甘油三酯和低密度胆固醇取自然对数log，最后在模型输出后再使用自然指数恢复。
在最后一次提交前，对训练集使用5折交叉验证进行调参与模型的调整。在最后一次提交时，我们将整个训练集全部作为训练使用。
