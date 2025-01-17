selective search算法详解
=====================

# 一. 背景

> 这篇论文是J.R.R. Uijlings发表在2012 IJCV上的一篇文章，主要介绍了选择性搜索（Selective Search）的方法。物体识别（Object Recognition），在图像中找到确定一个物体，并找出其为具体位置，经过长时间的发展已经有了不少成就。之前的做法主要是基于穷举搜索（Exhaustive Search），选择一个窗口（window）扫描整张图像（image），改变窗口的大小，继续扫描整张图像。这种做法是比较“原始的”，改变窗口大小，扫描整张图像，直观上就给人一种非常耗时，结果太杂的印象。这里是使用算法从多个维度对找到图片中可能的区域目标，减少目标碎片，提升物体检测效率。

# 二. 介绍

> 图像（Image）包含的信息非常的丰富，其中的物体（Object）有不同的形状（shape）、尺寸（scale）、颜色（color）、纹理（texture），要想从图像中识别出一个物体非常的难，还要找到物体在图像中的位置，这样就更难了。下图给出了四个例子，来说明物体识别（Object Recognition）的复杂性以及难度。

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/1.png)


> （a）中的场景是一张桌子，桌子上面放了碗，瓶子，还有其他餐具等等。比如要识别“桌子”，我们可能只是指桌子本身，也可能包含其上面的其他物体。这里显示出了图像中不同物体之间是有一定的层次关系的，或者说一种嵌套的关系——勺子在锅里面，锅在桌子上。
> （b）中给出了两只猫，可以通过纹理（texture）来找到这两只猫，却又需要通过颜色（color）来区分它们。
> （c）中变色龙和周边颜色接近，可以通过纹理（texture）来区分。
> （d）中的车辆，我们将车轮归类成车的一部分，既不是因为颜色相近，也不是因为纹理相近，而是因为车轮附加在车的上面。

# 三. 算法特点

> selective search算法特点如下：

## (一) 适用所有尺寸(Capture All Scales)

> 目标可以以任意尺寸出现在图片中，甚至有些目标和其他目标的边界并不明显。穷举搜索（Exhaustive Selective）通过改变窗口大小来适应物体的不同尺度，而选择搜索（Selective Search）算法采用了图像分割（Image Segmentation）以及使用一种层次算法（Hierarchical Algorithm）来对所有的目标尺寸进行记录，如下图所示：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/2.png)

## (二) 多样化(Diversification)
> 单一的策略无法应对多种类别的图像。使用颜色（color）、纹理（texture）、大小（size）等多种策略对区域（region）进行合并。

## (三) 计算速度快(Fast to Compute)
> 算法，就像功夫一样，唯快不破！

# 四. 算法流程

> 选择性算法使用的是按层次合并算法（Hierarchical Grouping），基本思路如下：首先使用论文“Efficient Graph-Based Image Segmentation”中的方法生成一些起始的小区域，之后使用贪心算法将区域归并到一起：先计算所有临近区域间的相似度（通过颜色，纹理，吻合度，大小等相似度），将最相似的两个区域归并，然后重新计算临近区域间的相似度，归并相似区域直至整幅图像成为一个区域。这种方式不需要设定滑动窗口，滑动格子，可以适应于任何目标的尺寸。具体流程如下：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/3.png)

```
· 输入：彩色图片（三通道）
· 输出：物体位置的可能结果L

使用 Efficient Graph-Based Image Segmentation的方法获取原始分割区域R={r1,r2,…,rn}；
初始化相似度集合S=∅；
计算两两相邻区域之间的相似度，将其添加到相似度集合S中；

从相似度集合S中找出相似度最大的两个区域 ri 和rj，
将其合并成为一个区域 rt，
从相似度集合S中除去原先与ri和rj相邻区域之间计算的相似度，
计算rt与其相邻区域（原先与ri或rj相邻的区域）的相似度，将其结果添加到相似度集合S中，
同时将新区域rt添加到区域集合R中；

在集合R中所有区域获取每个区域的Bounding Boxes，这个结果就是物体位置的可能结果L。
```

# 五. 多样化策略

> 论文给出了三方面的多样化策略：颜色空间多样化、相似度多样化和初始化区域多样化。具体如下：

## (一) 颜色空间多样化

> 考虑场景以及光照条件等，作者采用了8种不同的颜色空间，这个策略主要应用于图像分割算法中原始区域的生成。主要使用的颜色空间有：（1）RGB，（2）灰度I，（3）Lab，（4）rgI（归一化的rg通道加上灰度），（5）HSV，（6）rgb（归一化的RGB），（7）C（8）H（HSV的H通道）。具体如下：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/4.png)

## (二) 相似度多样化
> 论文介绍了4种相似度的计算方法，具体如下：

### 1. 颜色相似度

> 使用L1-norm归一化获取图像每个颜色通道的25 bins的直方图，这样对于RGB影像，每个区域都可以得到一个75维的向量，区域之间颜色相似度通过下面的公式计算：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/5.png)

> 区域合并后，新的区域直方图计算如下：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/6.png)

### 2. 纹理相似度

> 纹理采用SIFT-Like特征，具体做法是对每个颜色通道的8个不同方向计算方差σ=1的高斯微分（Gaussian Derivative），每个通道每个颜色获取10 bins的直方图（L1-norm归一化），这样就可以获取到一个240维的向量，区域之间纹理相似度计算方式和颜色相似度计算方式类似，合并之后新区域的纹理特征计算方式和颜色特征计算相同。

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/7.png)

### 3. 大小相似度
> 大小是指区域中包含像素点的个数，使用如下公式计算ri和rj区域大小占的比例共同决定大小相似度，计算方式是总体减去两个像素和占全图像像素比例，这样可以让小区域优先级高，尽量让小的区域先合并，避免对大区域关注度过高，或者说是避免某个大区域对周围小区域进行吞并。

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/8.png)

### 4. 填充相似度

> 填充相似度主要是为了衡量两个区域是否更加“吻合”，其指标是合并后的区域的Bounding Box（能够框住区域的最小矩形（没有旋转））越小，其吻合度越高，计算公式如下：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/9.png)

### 5. 总相似度

> 将上述所有相似度写到一起，得到如下的计算公式：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/10.png)

## (三) 初始化区域多样化

> 最后是初始化起始区域，用"Efficient GraphBased Image Segmentation"得到的初始化区域可以根据阈值k得到不同的结果。

# 六. 区域排序

> 经过上述步骤，得到目标假设区域（object hypothesis），但是区域较多，不能全部展现，需要对其进行排序，在数量和质量之间做一个平衡。论文采取的策略如下：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/11.png)

# 七. 目标识别

> 论文采用的目标识别方法是传统的“特征+SVM“方法，具体如下：

##  (一) 特征

> 论文采用了color-SIFT特征和spatial pyramid divsion方法，在一个尺度下σ=1.2下抽样提取特征。使用SIFT、Extended OpponentSIFT、RGB-SIFT特征，在四层金字塔模型 1×1、2×2、3×3、4×4，提取特征，可以得到一维的特征向量。

## (二) 训练过程

> 训练方法采用SVM。首先选择包含真实结果（ground truth）的物体窗口作为正样本（positive examples），选择与正样本窗口重叠20%~50%的窗口作为负样本（negative examples）。在选择样本的过程中剔除彼此重叠70%的负样本，这样可以提供一个较好的初始化结果。在重复迭代过程中加入hard negative examples（得分很高的负样本）。其训练过程框图如下：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/12.png)

## (三)评估

> 通过算法计算得到的包含物体的Bounding Boxes与真实情况（ground truth）的窗口重叠越多，那么算法性能就越好。这是使用的指标是平均最高重叠率ABO（Average Best Overlap）。对于每个固定的类别c，ABO的公式表达为：

![image](https://github.com/ShaoQiBNU/Selective_Search/blob/master/images/13.png)


> 上面结果给出的是一个类别的ABO，对于所有类别下的性能评价就是使用所有类别的ABO的平均值MABO（Mean Average Best Overlap）来评价。

# 八. 目标识别

> 具体使用实例见：https://github.com/ShaoQiBNU/selectivesearch
