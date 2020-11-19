---
typora-copy-images-to: ./image
---

# YOLACT

参考[https://blog.csdn.net/sinat_37532065/article/details/89415374](https://blog.csdn.net/sinat_37532065/article/details/89415374)，写的很好

### 摘要

本文提出一种FCNs型实时实例分割模型YOLACT。 在MS COCO数据集上，mAP达到了29.8，使用单块Titan XP推理速度达到了33fps。

YOLACT将实例分割拆分成两个并行的子任务：一个分支去生成一系列的原型mask；另一个分支预测每个实例对应的mask系数。

由于在YOLACT没有进行repooling类操作，所以得到的分割mask更为精细。

此外，本文还提出了一种Fast NMS算法，在轻微降低精度的同时，显著提升了NMS过程的计算速度。

### 介绍

Mask R-CNN是实例分割模型的典范，它基于Faster-RCNN改进，奠定了Two-stage型实例分割的基调，同样Mask R-CNN的重点也是在于精度的提升。受此影响，作者希望基于One-stage目标检测器设计出一种One-stage实例分割模型。

SSD/YOLO等通过移除第二个stage，并以其他方式弥补性能缺失来加速Faster R-CNN。然而这种思想不易直接应用在实例分割任务上，因为实例分割为了产生mask，严重依赖于特征定位。在Two-stage模型中，通过repooling型操作（如ROI align、ROI pooling）来将特征映射到包围框中，这种做法从逻辑上就是串行的，所以很难加速。FCIS虽然将这些操作进行并行化，但由于后处理步骤太多，同样很难达到加速的目的。

基于以上考虑，本文提出了YOLACT，它摒弃了隐含的特征定位步骤，将实例分割任务拆分成两个并行的子任务：（1）生成一系列覆盖全图的原型mask；（2）对每个实例，预测一系列的线性组合系数。最后，在进行推理时，对每一个实例，使用其对应预测出的mask系数，与原型mask简单乘加，然后根据bounding box裁剪，阈值化，即得到每个实例对应的mask。

经过实验分析，作者认为YOLACT通过原型mask去自适应的学习如何定位实例目标。原型mask的数量与类别数目无关，也就是说每张原型mask都包含了跨类别的信息，YOLACT旨在学习一种分布的表示方法，这样每个实例可以通过该表示方法对多张原型mask进行线性组合，来得到自己的mask。通过训练，每张原型mask会学习到去抽象输入图像的一些细节信息，比如边缘信息、位置信息，或对特定区域响应的信息。

YOLACT有三个显著的优势：

1）速度快，因为one-stage；

2）mask质量高，因为不包含repooling类操作；

3）普适性强，这种生成原型mask和mask系数的思路可以应用在目前很多留下的检测器上。

与人类视觉系统相比，YOLACT也很直观：线性系数组合和检测分支更像是在解决“是什么”的问题，原型mask的生成像是在解决“在哪里”的问题。

### YOLACT

类比Mask R-CNN之于Faster R-CNN，YOLACT旨在现有的one-stage型检测器上添加一个mask分支来达到实例分割的目的，但这一过程中不希望引入特征定位步骤。

YOLACT通过添加两个并行的分支来完成该任务：**第一个分支使用FCN去产生一系列独立于单一实例的原型mask；第二个分支在检测分支上添加额外的头去预测mask系数，以用于编码一个实例在原型mask空间的表示。**最后，在NMS步骤后，通过将两分支的输出结果进行线性组合来得到最后的预测结果。

YOLACT的网络结构如下图所示。

![YOLACT](/Users/ruoningsong/Downloads/YOLACT.png)

**可行性分析**

因为分割任务的目标是得到mask，而mask的特点是存在天然的空间联系，所以YOLACT采用了上述组织形式。从NN的角度来说，Conv层天然利用了空间相关性，但FC层不会。这就导致了一个问题，因为大多数One-stage检测器通过FC层预测box参数和所属类别。Two-stage通过ROI Align等特征定位步骤保留了空间信息，同时使用Conv层输出mask，但是这些操作都必须等待RPN来完成，极大地影响了效率。

在YOLACT中，**FC层负责预测语义标签，Conv层负责预测原型mask和mask系数。**两分支并行，最后通过矩阵乘法组装，这样一来既保留了空间的相关性，又保持了One-stage的模型结构，速度极快。 

#### 原型生成

称生成原型的网络分支为protonet。protonet基于FCN实现，最后会输出k个通道，每个通道可以视作一张原型mask。protonet的作用有些类似语义分割模型，不同之处在于protonet部分的训练不单独设置loss，只在整个网络最后输出的mask上进行监督。

![YOLACT-protonet](/Users/ruoningsong/Downloads/YOLACT-protonet.png)

protonet的设计上有两个取舍：

1）从深的backbone中获取protonet可以产生更稳当的mask；

2）高分辨率的原型mask有利于提高分割精度和对小目标的效果。

YOLACT使用了FPN作为backbone，同时上采样到原图尺寸的1/4以改善对小目标的分割效果。

#### mask系数

典型的基于Anchor的检测模型会为每个Anchor预测4个值用于表征box信息，和C个值用于表征类别得分，共(4+C)个值。

YOLACT为 每个Anchor预测(4+C+k)个值，额外k个值即为mask系数。

另外作者认为，为了能够通过线性组合来得到最终想要的mask，能够从最终的mask中减去原型mask是很重要的。换言之就是，mask系数必须有正有负。所以，在mask系数预测时使用了tanh函数进行非线性激活，因为tanh函数的值域是(-1,1).

#### mask合成

通过基本的矩阵乘法配合sigmoid函数来处理两分支的输出，从而合成mask。



 其中，P是h×w×k的原型mask集合，C是n×k的系数集合，代表有n个通过NMS和阈值过滤的实例，每个实例对应有k个mask系数。

**Loss设计**

Loss由分类损失、框回归损失和mask损失三部分组成，其中分类损失和框回归损失同SSD，mask损失为预测mask和ground truth mask的逐像素二进制交叉熵。

**Mask裁剪**

为了改善小目标的分割效果，在推理时会首先根据检测框进行裁剪，再阈值化。而在训练时，会使用ground truth框来进行裁剪，并通过除以对应ground truth框面积来平衡loss尺度。

### Backbone Detector

因为预测一组原型mask和mask系数是一个相对比较困难的任务，需要更丰富更高级的特征，所以在网络设计上，作者希望兼顾速度和特征丰富度。因此，YOLACT的主干检测器设计遵循了RetinaNet的思想，同时更注重速度。

YOLACT使用ResNet-101结合FPN作为默认主干网络，默认输入图像尺寸为550×550.

与原版RetinaNet相比，YOLACT检测头（如下图）的设计更轻量，速度更快。

使用**平滑-L1 loss**训练bounding box参数，并且采用了和SSD中相同的bounding box参数编码方式。

使用**softmax交叉熵**训练分类部分，共**(C+1)**个类别。同时，使用**OHEM**方式选取训练样本，正负样本比例设为1:3.

注意一点，**没有像RetinaNet一样采用focal loss**。