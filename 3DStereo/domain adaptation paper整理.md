

# 1 .offline domain adaptation

（1）**“Unsupervised adaptation for deep stereo,” in IEEE ICCV, 2017,**

使用SGM这些传统算法生成原始视差图(认为视差是高可信度的)，将网络的输出视差与其比较，偏离越大则在损失函数中惩罚越大



（2）**“Zoom and learn: Generalizing deep stereo matching to novel domains in IEEE CVPR, 2018**

用合成数据预训练模型，基于一个现象：上采样后的图通过网络能得到更清楚的细节，所以将目标域图按不同尺度上采样，并将输出作为输入fine-tune网络，还施加graph Laplacian regularization保持所需边缘等



（3）**“Unsupervised domain adaptation for depth prediction from images,” IEEE TPAMI, 2019**

Unsupervised adaptation for deep stereo,” in IEEE ICCV, 2017的延伸，但使用的损失函数更复杂，而且还对单目的情况也实验了。



（4）**”AdaStereo: A Simple and Efficient Approach for Adaptive Stereo Matching“    CVPR 2020**

提出一种新的domain adaptation pipeline，尤其去网络的输入做了多级对齐处理



（5）**Domain-Invariant Stereo Matching Networks  ECCV 2020**

加入“domain normalization”，还利用了graph  structure 



（3）是对（1）的拓展，增加更多损失项等





## 2. online ,real-time,continual adaptation

（1）**Open-world stereo video matching with deep rnn.  ECCV 2018**

使用stereo video 作为输入，用LSTM模块记忆学习到的内容



（2）**“Real-time self-adaptive deep stereo,” in IEEE CVPR, 2019**

提出MADNet



（3）“Learning to Adapt for Stereo,” in IEEE CVPR, 2019

使用meta-learning的想法



（4）**Faster Self-adaptive Deep Stereo ACCV 2020**

在MADNet的基础上加入了 Knowledge Reverse Distillatio和Adapt-or-Hold (AoH) mechanism机制



（5）**Continual Adaptation for Deep Stere  2020CVPR**
是MADNet的延伸



online domain adoptation好几篇都是在MADNet上做改进，后面有的文章引入了meta-learning的想法