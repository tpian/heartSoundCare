# heartSoundCare

介绍自己研究生期间有关心音的研究内容

## 1.主要研究内容

- 心音分割
- 多听诊区的心脏杂音检测

&emsp;&emsp;意义：心血管疾病是目前全球范围内的主要死亡原因之一，而心音听诊是一种简单、无创、低成本的心血管疾病筛查方法，但心音听诊对医生的听诊技能要求较高，且受到环境噪声的干扰，因此，心音自动化诊断技术的研究具有重要的临床意义，心音分割和心脏杂音检测是心音自动化诊断的两个重要环节。

## 2.心音介绍

&emsp;&emsp;一个基础心音周期由 S1、收缩期、S2 以及舒张期组成，相应的 PCG 如图 1 所示。
![心音周期](/figure/PCG.png)
<div align='center'>
  <font size=12>
    图1 心音周期示意图
  </font>
</div>

<br>

- 心音分割的任务是定位心音信号中 4 种基本心音的起止时刻，是其他心音分析任务的基础。
- 心脏杂音是心脏疾病的重要体征，心脏杂音检测的任务是检测受试者心音信号中是否存在心脏杂音。

## 3.心音分割

### 数据集

&emsp;&emsp;2016 心音挑战赛的开源数据集。【数据集链接】：https://www.physionet.org/content/challenge-2016/1.0.0/


![心音分割数据集](/figure/SegDataset.png)

### 方法

<div align='center'><img src=/figure/SegMethod.png width=80%></div>
<div align='center'>
  <font size=12>
    图2 心音分割方法框图
  </font>
</div>

<br/>
<br/>
&emsp;&emsp;主要步骤：

- 预处理：裁剪、滤波
- 特征提取： 梅尔谱图、MFCC、谱熵、多种包络
- 特征降维：主成分分析
- 网络训练：对异常样本进行过采样，平衡数据集中正常样本和异常样本的比例；使用交叉熵损失函数，网络结构如图 3 所示。
- 结果译码： Viterbi 算法，校正不合理的状态变化，如 S2->收缩期等。

<br/>
<br/>
<div align='center'><img src=/figure/SegNet.png width=80%></div>
<div align='center'>
  <font size=12>
    图3 心音分割网络结构
  </font>
</div>

### 结果

a. 心音分割 F1 值到达 96.84%，超过当前 SOTA 水平；

b. 在与训练集样本分布差异较大的数据子集上，F1 值仍能达到 96.4%，表明模型具有较好的泛化能力；

c. 能实现 4 种基本心音状态的事件检测，F1 值均在 96%以上，说明模型兼具良好的心音事件检测能力。

## 4.心脏杂音检测

### 数据集

&emsp;&emsp;2022 年心音挑战赛的开源数据集。【数据集链接】：https://www.physionet.org/content/challenge-2022/1.0.0/
![心脏杂音检测数据集](/figure/DetecDataset.png)

### 方法

<div align='center'><img src=/figure/DetecMethod.png width=80%></div>
<div align='center'>
  <font size=12>
    图4 心脏杂音检测方法框图
  </font>
</div>

<br/>

主要步骤：

- 预处理：
  - 心音：裁剪、滤波
  - 临床信息：MICE 方法填充缺失值
- 特征提取：梅尔谱图、MFCC、谱熵、多种包络
- Stage1 ： 使用图 3 网络完成杂音事件检测
- Stage2 ： 使用传统机器学习方法，如 SVM、RF、KNN、XGBoost、LightGBM 等，根据 Stage1 的结果与受试者临床信息，对单听诊区的心脏杂音检测结果进行判决。
- Stage3 ： 使用传统机器学习方法，根据多个听诊区的心脏杂音检测结果，对受试者最终的心脏杂音检测结果进行判决。

### 结果

a. Stage1 F1 值达到 72.94%，目前没有其他相关工作的结果可供比较，在杂音等级较高的心音记录上模型性能更好，如图 5 所示。
![Stage1结果](/figure/Stage1Res.png)
<div align='center'>
  <font size=12>
    图5 Stage1 结果示意图
  </font>
</div>

<br/>

b. Stage2 加权准确率达到 74.46%，Stage3 加权准确率达到 76.69%，接近当前 SOTA 水平。

c. 方法多种输出结合心音分割网络的结果，可实现受试者心脏杂音检测的自动化，如图 6 所示，这是其他相关工作所不具备的。

<div align='center'><img src=/figure/Diagnose.png width=80%></div>
<div align='center'>
  <font size=12>
    图6 最终结果示意图
  </font>
</div>

<br/>
&emsp;&emsp;图6中受试者的心脏杂音检测结果为：存在心脏杂音，杂音主要发生在AV、PV、TV听诊区，表现为收缩期杂音。可能存在二尖瓣关闭不全情况。
