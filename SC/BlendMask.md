---
typora-copy-images-to: ./image
---

### BlendMask

KeyWord：Top-Down, Bottom-Up, FCIS, YOLACT

问题：

- two-stage: it is difficult for independent heads to share features with related tasks such as semantic segmentation which causes trouble for network architecture optimization. `Mask RCNN`
- one-stage: 优势：1) models consisting of only conventional operations are simpler and easier for
  cross-platform deployment; 2) a unified framework provides convenience and flexibility for multi-task network architecture optimization.
- 现有top-down问题：1) local-coherence between features and masks is lost; 2) the feature representation is redundant because a mask is repeatedly encoded at each foreground feature; 3) position information is degraded after downsampling with strided convolutions.
- 现有bottom-up问题: 1) heavy reliance on the dense prediction quality, leading to sub-par performance and fragmented/joint masks; 2) limited generalization ability to complex scenes with a large number of classes; 3) requirement for complex postprocessing techniques.


- 现有bottom-up与top-down结合问题: We recognize two important predecessors, FCIS [16] and YOLACT [2]. They predict instance-level information such as bounding box locations and combine it with per-pixel predictions using cropping (FCIS) and weighted summation (YOLACT), respectively. We argue that these overly simplified assembling designs may not provide a good balance for the representation power of top- and bottom-level features.

文章目标：

- 我们工作的重点之一是研究在完全卷积实例分割中更好地合并这两种方法。 更具体地说，我们通过丰富实例级信息并执行更细粒度的位置敏感掩码预测来概括基于建议的掩码组合的操作。 我们进行了广泛的消融研究，以发现最佳尺寸，分辨率，对准方法和特征位置。

贡献：

- blender，a flexible method for proposal-based instance mask generation called blender, which incorporate rich instance-level information with accurate dense pixel features.
- BlendMask’s the bottom module is able to output masks of much higher
  resolution, due to its flexibility and the bottom module not being strictly tied to the FPN. 
- BlendMask, which is closely tied to the state of the art one-stage object detector, FCOS, by adding moldiest computation overhead to the already simple framework.

![blendmask](/Users/ruoningsong/Nustore Files/NutStore/BUPT/论文笔记/image/blendmask.png)

