# v1.2.0 - SAHI 切图增强 Teacher

## 一、版本定位

`v1.2.0` 是本项目的第二阶段 Teacher 模型版本。

该版本在 `v1.1.0` 原图监督基线的基础上，引入 **SAHI-style slicing / patch training**。核心思路是：不再直接将遥感大图整体缩放到 1024，而是先将原图切分为多个局部 patch，再使用 YOLOv8x 进行训练。

这样做的主要目的是解决航空遥感小目标检测中的尺度问题，尤其是 vehicle、ship、storage_tank 等目标在原图中占比过小，直接 resize 后容易造成特征损失。

---

## 二、实验配置

| 项目 | 配置 |
|---|---|
| Model | YOLOv8x |
| Dataset | DOTA five-class subset |
| Classes | plane, ship, vehicle, harbor, storage_tank |
| Label Format | DOTA OBB → YOLO HBB |
| Data Strategy | SAHI-style patch slicing |
| imgsz | 1024 |
| batch | 4 |
| epochs | 100 |
| EarlyStopping | patience = 30 |
| Stopped Epoch | 67 |
| Best Epoch | 37 |
| Training Time | 5.487 h |
| GPU | RTX 5090 |

---

## 三、整体结果

最终验证结果如下：

| Metric | Value |
|---|---:|
| Precision | 0.902 |
| Recall | 0.795 |
| mAP50 | 0.827 |
| mAP50-95 | 0.562 |

与 `v1.1.0` 原图监督基线相比，`v1.2.0` 获得了显著提升：

| Version | mAP50 | mAP50-95 |
|---|---:|---:|
| v1.1.0 原图监督基线 | 0.475 | 0.258 |
| v1.2.0 SAHI切图增强 Teacher | 0.827 | 0.562 |
| Improvement | +0.352 | +0.304 |

---

## 四、分类别结果

| Class | Images | Instances | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|---:|
| all | 985 | 79378 | 0.902 | 0.795 | 0.827 | 0.562 |
| plane | 233 | 3207 | 0.956 | 0.955 | 0.973 | 0.751 |
| ship | 250 | 14728 | 0.942 | 0.833 | 0.862 | 0.569 |
| vehicle | 584 | 56653 | 0.788 | 0.506 | 0.487 | 0.270 |
| harbor | 189 | 2617 | 0.904 | 0.868 | 0.895 | 0.571 |
| storage_tank | 68 | 2173 | 0.921 | 0.814 | 0.916 | 0.646 |

---

## 五、实验分析

`v1.2.0` 的结果说明，SAHI 切图对遥感小目标检测具有显著作用。

在 `v1.1.0` 中，原始大图被直接缩放到 1024，导致大量小目标在特征图中占比过低，尤其是 vehicle 类。经过切图后，同一目标在局部 patch 中的相对尺寸变大，模型能够学习到更清晰的局部纹理和几何特征。

因此，本版本的提升主要来自数据表达方式的改变：

```text
原始大图训练 → 小目标特征被压缩
SAHI切图训练 → 小目标在局部 patch 中变得更清晰
