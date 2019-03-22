# CorrelationAnalysis

##Background
这是一个为生产线出现不良产品时而进行的根因分析，即发现导致高不良率的机台或机台组合，再细化到发现机台上的不良根因参数。
##Data type

设备进出表，每个站点代表某种工序，每个工序可能存在并行的机台a,b..n都能完成该工序。这是典型的名义数据，数据形式如下。

**product_id**： 产品id

**station0n：**产品经过站点0n时的加工的机台

**label:**二类标签，1不良，0合格，通常样本不平衡约（1：20），后面我们会使用一些方法来尝试改善他

| product_id | station01  | station02  | station03  | station04  | label |
| ---------- | ---------- | ---------- | ---------- | ---------- | ----- |
| a0b1a22a   | machine_1a | machine_2c | machine_3b | machine_4c | 0     |
| a0b1a22b   | machine_1b | machine_2c | machine_3b | machine_4c | 1     |
| c0b1b21a   | machine_1b | machine_2c | machine_3b | machine_4c | 0     |
| c0b1b19b   | machine_1c | machine_2c | machine_3b | machine_4c | 0     |

## Framework

####Single machine location

我们使用多个相关性指标先进行不良机台定位，Infomation value，卡方统计量，做出置信度由高到低的Rank后，使用公式$\sqrt[n]{\prod_{i=1}^{n}Rank_i}$确定不良机台的置信度。

卡方统计量函数处于utility.py中

information value 是利用的别人的库，但对连续数据进行分箱的操作会使得结果对这个处理过分依赖，所以应该尽量多次尝试分箱的数量，选择最优。离散的数据处理方式可以更简单，因此后续会更新更简单的方式。

**example文件夹**中是使用该方法进行分析测试的notebook文件，但是是使用的station_machine为列名的01进出表测试的，即产品通过该站点该机台，则该特征的值为1，否则0。最后能在top2置信度发现不良机台。

#### Path analysis

因为导致不良产品的不一定完全是单个机台导致的，即有可能是机台的组合使得不良发生。因此单机台的分析不太合理。加上路径的分析作为补充。

使用CART决策树进行路径分析，用监督的方式学习站点设备进出表，最后生成的树的总是到label=1叶子节点路径是被怀疑的路径，进行重复的路径高频统计，1叶子节点总是经过的高频路径，是最可能的不良机台组合路径。

###Decision tree by information value

CART使用gini指数作为分裂指标，对不平衡数据可能并不能很好的处理。

因此实现了以**IV作为分裂指标的决策树。**

DT_IV3.py是多叉IV树，即根据站点下的机台数量，三台机台有三条分支

DT_IV2.py是二叉IV树，即和CART的基尼指数分裂方式一致，若为该特征走左，否则右。

DecisionTreebyIV.ipynb是DT_IV2测试文件，

DT_IV3.ipynb是DT_IV3测试文件，

#### result of IV

使用实际生产数据：iv树比cart准确度少平均3个点

使用公开数据集heart disease：iv树比cart准确度高平均5个点，但召回率一致。

不是数学大佬，只能以后做更多实验，逐步补充情况。



## 若有帮助,加个star憋 ^.^


