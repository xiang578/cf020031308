# 社交媒体不会说谎：用多种在线信息预测2016台湾选举

@caoj

* 应用
  - 选举预测
* 数据
  - 社交媒体
  - 民意调查
  - 搜索引擎
  - 网站流量
* 方法
  - 卡尔曼滤波器
  - 金融事件模型

## 摘要

在线媒体的流行吸引了来自不同领域的研究人员探索人类行为并做出有趣的预测。在本研究中，我们利用从各种在线平台收集的各种各样的数据来预测台湾2016年大选。与大多数现有研究相反，我们把各类信息看做“信号”，采用卡尔曼滤波器，将多个信号融合到候选人的每日投票预测中。我们还根据源自金融研究领域的事件研究模型，考虑以定量方式影响选举的事件。我们获得了以下有趣的发现。首先，在预测能力和及时性方面，在线媒体的民意在台湾选举预测中占主导地位。但线下民意调查仍可以减小在线意见的样本偏见。其次，尽管在选举日临近时在线信号趋同，但简单的Facebook“赞”仍然是选举结果的最强指标。第三，大多数有影响力的事件与两岸关系有很强的联系，选举前一天道歉视频随后的周慈钰旗事件增加了蔡英文的投票份额3.66％。这项研究证明了网络媒体在政治中的预测能力和信息融合的优势。结合使用卡尔曼滤波器和事件研究方法有助于数据驱动的政治分析范例，用于预测和归因目的。

## 主要内容

### 数据和度量方法 

数据和度量信号来自于Facebook、Twitter、Google、候选人的竞选主页，对每类度量信号使用30天移动平均线（moving average）避免过度波动。

#### （1）Facebook

第一个signal：某个候选人的每个帖子的每日平均点赞（like）次数：

$$ S^c_{k,FAL} = \frac{1}{m} \sum^{m-1}_{j=0} \frac{\sum_i like^c_{k-j,i} / n^c_{k-j,FA}}{\sum_c \sum_i like^c_{k-j,i} / n^c_{k-j,FA}} $$
其中，$like^c_{k,i}$表示候选人c在k日发布的帖子i的点赞数，$n^c_{k,FA}$表示该候选人的帖子总数量，m是移动平均的窗口长度。

第二个signal，计算每个帖子下的日平均评论数：

$$ S^c_{k,FAC} = \frac{1}{m} \sum^{m-1}_{j=0} \frac{\sum_i Comment^c_{k-j,i} / n^c_{k-j,FA}}{\sum_c \sum_i Comment^c_{k-j,i} / n^c_{k-j,FA}} $$
$Comment^c_{k,i}$表示候选人c在k日发布的帖子i 下的评论数。

#### （2）Twitter

提及某候选人（繁体中文、简体中文）的tweets数量：
$$ S^c_{k,TW} = \frac{1}{m} \sum^{m-1}_{j=0} \frac{tw^c_{k-j}}{\sum_c tw^c_{k-j}} $$
$tw^c_k$表示第k天关于候选人c的推文数量。

#### （3）搜索引擎

根据候选人名字（繁体中文、简体中文）文数量搜索，限制搜索源在台湾，搜索索引率（search index ratio）
$$ S^c_{k,GO} = \frac{1}{m} \sum^{m-1}_{j=0} \frac{search^c_{k-j}}{\sum_c search^c_{k-j}} $$
$search^c_k$是候选人c在k日的关键字索引搜索总数。

#### （4）竞选首页

收集了来自Alexa的候选人竞选主页数据的每日流量，并使用“IP流量比率”作为意见衡量标准：
$$ S^c_{k,IP} = \frac{1}{m} \sum^{m-1}_{j=0} \frac{IP^c_{k-j}}{\sum_c IP^c_{k-j}} $$
$IP^c_k$候选人c在k日的竞选首页的IP流量

#### （5）民意调查

由19位权威民意调查员发布的线下民意调查。假设直到新的民意调查结果发布，上一次的结果意见不变。

### 模型

#### 卡尔曼滤波器

（1）选举预测的目标是为了推断各个候选者的投票份额。选择卡尔曼滤波器。卡尔曼滤波器会根据当前状态"测量值" 、上一刻的 "预测量" 和 "误差",计算得到当前的最优量， 再预测下一刻的量。
$$ \mathbf{s}^c_k = \mathbf{h}_k x^c_k + \mathbf{r}^c_k \text{,  } \mathbf{r}^c_k \sim N(0, \mathbf{R}^c_k) $$
$$ x^c_k = f_k x^c_{k-1} + q^c_k \text{,  } q^c_k \sim N(0, \sigma^2_{c,k}) $$
$$ x^c_0 \sim N(m^c_0, p^c_0) $$
$\mathbf{h}_k$是一个向量，映射了候选人c的隐藏状态$x^c_k$，用来观察$\mathbf{s}^c_k$中的多个信号，$f_k$是状态转移系数，$x^c_0$是隐藏状态的初始值，$\mathbf{r}^c_k \text{ and } q^c_k$是两个独立的高斯噪声。

在本例中，$x^c_k$是候选人c在k日的真实投票份额，$$ \mathbf{s}^c_k = (s^c_{k,GO}, s^c_{k,FAL}, s^c_{k,TW}, s^c_{k,IP})^T $$包含了被观察的多个信号。设置$f_k = 1 \text{ and } \mathbf{h}_k = 1$，初始的投票数$m^c_0$被设置为最近民调结果的平均值，并设置$p^c_0=1$的波动范围。请注意，我们将初始投票数$m^c_0$置为每个候选人影响信号的平均值和一个相等的值$m^c_0=1/3$，状态变量分别设置为$p^c_0=0$和$p^c_0=1$，当时间序列足够长时，最终预测结果对初始值不敏感。这组方程背后的逻辑是在线的度量信号是有缺陷的，真实的投票状态由混合噪声的均值表示。该模型的目标是融合有缺陷的信号以估计每日状态，并进一步将估计转移到第二天进行预测。

（2）然后估计噪声的参数$\mathbf{R}^c_k \text{ and } \sigma^2_{c,k}$，为了降低模型的复杂度，假设$\mathbf{R}^c_k=\mathbf{R}_k \text{ and } \sigma^2_{c,k}=\sigma^2_k, \forall c$，可以通过最大化条件密度函数来获得参数的最大后验估计：
$$ \mathcal{J}
   = p(x^{tsai}_{1:k}, x^{chu}_{1:k}, x^{soong}_{1:k}, \sigma^2_k, \mathbf{R}_k | \mathbf{s}^{tsai}_{1:k}, \mathbf{s}^{chu}_{1:k}, \mathbf{s}^{soong}_{1:k})
   \propto \prod_c p(x^c_0) \prod^k_{j=1} p(\mathbf{s}^c_j|x^c_j, \mathbf{R}_k) p(x^c_j|x^c_{j-1}, \sigma^2_k) p(\sigma^2_k, \mathbf{R}_k) $$
$$ \sum_c x^c_k = 1 \text{ and } \sum_c \mathbf{s}^c_k = \mathbf{I}_{4 \times 1} $$
 
得到：
$$ \hat{\sigma}^2_k = \frac{1}{3k} \sum_c \sum^k_{j=1} (\hat{x}^c_{j|j} - f_j \hat{x}^c_{j-1|j-1})^2 $$
$$ \mathbf{\hat{R}}_k = \frac{1}{3k} \sum_c \sum^k_{j=1} ((\mathbf{s}^c_j - \mathbf{h}_j\hat{x}^c_{j|j-1}) (\mathbf{s}^c_j - \mathbf{h}_j\hat{x}^c_{j|j-1})^T - (\mathbf{h}_j\hat{p}_{j|j-1}{\mathbf{h}_j}^T)) $$
$\hat{x}^c_{k|k-1}$是在给定k-1日的信号的条件下，对候选人c在日期k的投票状态预测，$\hat{x}^c_{k|k}$是在给定到k日的信号的条件下，更新后的投票预测。$p^c_{k|k-1} \text{ and } p^c_{k|k}$是预测的协方差和更新后的估计协方差。

为了在时间k递归估计每日投票状态，$\hat{x}^c_{k|k-1}$（投票份额预测）为：
$$ \hat{x}^c_{k|k-1} = f_k \hat{x}^c_{k-1|k-1} $$
$$ \text{(9) } p^c_{k|k-1} = f^2_k p^c_{k-1|k-1} + \hat{\sigma^2_k} $$

同时，通过把$s^c_k$注入到预测值$\hat{x}^c_{k|k-1}$，去更新状态估计$\hat{x}^c_{k|k}$。

使用加权函数来表示状态和信号的组合：
$$ \hat{x}^c_{k|k} = f_k \hat{x}^c_{k|k-1} + \mathbf{k}^c_k (\mathbf{s}^c_k - \mathbf{h}_k \hat{x}^c_{k|k-1}) $$
$$ p^c_{k|k} = p^c_{k|k-1} - \mathbf{k}^c_k \mathbf{h}^c_k p^c_{k|k-1} $$
其中$\mathbf{k}^c_k$称为卡尔曼增益系数，是用来衡量状态预测和预测中的变化信号。

通过最小化更新状态估计的误差$x^c_k - \hat{x}^c_{k|k}$，得到卡尔曼增益的表达式：
$$ \mathbf{k}^c_k = p^c_{k|k-1} \mathbf{h}^T_k (\mathbf{h}_k p^c_{k|k-1}\mathbf{h}^T_k + \mathbf{\hat{R}}^c_k)^{-1} $$
 
当获得更新的估计时，可以利用（9）式去预测第二天的投票份额。

（3）据台湾的互联网使用情况报告，一个超过90％的年龄在20至45岁之间的台湾居民的已访问互联网自2015年五月这个比例是在80％以上年龄在45岁至55岁之间的人口。相比之下，55岁以上的居民中只有49.5％在同一时期使用过互联网。因此，我们将在线数据融合结果作为年龄在20至50岁之间的群体的代表。关于民意调查机构采用的年龄调整抽样方法，我们将50至60岁，60至70岁和70岁以上群体的民意调查结果作为相应年龄组的投票份额估算，每日投票份额的预测$y^c_k$利用如下加权公式：

$$ y^c_k = w_{20~50} \hat{x}^c_{k|k-1} + w_{50~60} z^c_{50~60,k} + w_{60~70} z^c_{60~70,k} + w_{70} z^c_{>70,k} $$
$w_i$是年龄组i的人口比例，可以从台湾内政部获得。$z^c_{i,k}$是第k天候选人c在年龄组i中的民意调查结果。

#### 突发事件检测

Twitter作为一个在线平台，在竞选期间汇总了不同候选人的信息。通过分析2015年10月Twitter的情况，我们发现超过80％的检索推文都是新闻。由于大多数台湾主流媒体都在Twitter上设置账号，推文的波动性能够发出有影响力的事件。三步检测方法设计如下：

（1）根据大量推文感知事件。$tw^c_k$是之前定义的候选人c在k日的相关推文数量。我们通过跟上限$u^c_{k+1} = \bar{n} + \frac{s}{\sqrt{m}} t_{\alpha / 2} (m - 1)$比较，跟踪最近30日内的该数据，$\bar{n}$是$tw^c_k$在m天内的平均值，s是标准差。基于具有$\alpha$重要性的t-test，如果$tw^c_{k+1} \text{ surpasses } u^c_{k+1}$，存在一个影响事件。我们假设每次爆发只有一个新事件占主导地位，这对于政治运动来说是合理的。

（2）估计事件的时间窗口。每个候选人的每日推文首先被整合到一个文件中; 然后，文档中的术语由TF-IDF方法加权。TF-IDF是一个数字统计量，旨在反映一个单词对语料库集合中文档的重要程度。在TF-IDF值的增加比例随着次数的词出现在文档中，但往往是由主体，这有助于调整有些字一般更频繁出现的事实这个词的频率偏移。TF-IDF计算如下：

$$ tf(t, d^c_k) = \frac{f_{t,d^c_k}}{\sum_t f_{t,d^c_k}} $$
$$ idf(t, D^c) = \log \frac{N^c}{1 + |d^c_k \in D^c : t \in d^c_k|} $$
$$ tf-idf(t, d^c_k, D^c) = tf(t, d^c_k) idf(t, D^c) $$

$f_{t,d^c_k}$是单词t在关于候选人c在k日的一篇推文$d^c_k$中的计数，$D^c$是候选人c的所有推文，$N^c=|D^c|, \text{ and } |d^c_k \in D^c : t \in d^c_k|$，是单词t出现的文档数量。选择突发事件中权重最高的前30个词作为该事件的典型词，然后我们继续检查爆发日前后5天的典型单词的重复，具有非零重叠的第一天被认为是事件的开始日，并且具有非零重叠的最后一天是结束日，并且定义了事件时间窗口。删除可疑事件的时间窗口只有一天。

（3）衡量事件对公众舆论的影响。$\hat{x}^c_{k|k-1}$表示前一天的估计，$\hat{x}^c_{k|k}$表示最终的估计。直观的说，$\hat{x}^c_{k|k}$吸收了所有相关事件信号的信息，因此$\hat{x}^c_{k|k-1}$到$\hat{x}^c_{k|k}$的变化表示事件信号的影响。为了描述事件信号的影响，使用事件学习模型如下:

$$ \hat{x}^c_{k|k} = a + \hat{x}^c_{k-1|k-1} + \sum^{J^c}_{j=1} \gamma^c_j D^c_{j,k} + \epsilon $$
$D^c_{j,l}$是候选人c在k日的一个虚拟变量，如果在事件j的时间窗内，它等于1，否则等于0.  
$J^c$是检测事件的总数，a是回归常数。如果事件$J^c$对公众舆论有显着影响，则事件j通过t检验，$\gamma^c$是对事件j的影响的估计。通过这种方式，我们可以识别实际影响选举的事件。

## 备注 

* [t检验](https://baike.baidu.com/item/t检验/9910799?fr=aladdin)
* [最大后验估计](https://blog.csdn.net/bitcarmanlee/article/details/81417151)
* [卡尔曼滤波器](https://blog.csdn.net/phker/article/details/48468591)
* [卡尔曼滤波器](https://blog.csdn.net/lybaihu/article/details/54943545)
* [相关数据下载](https://figshare.com/articles/Social_media_would_not_lie_prediction_of_2016_taiwan_election_rar/6014159)