# v1.3.0 - Vehicle Hard Tile Mining+Higher SHAI Overlap 消融实验

## 一、版本定位 / Version Position

`v1.3.0` 是在 `v1.2.0` 基础上进行的 **vehicle 类专项优化消融实验**。

在 `v1.2.0` 中，SAHI 切图显著提升了整体检测效果，但 `vehicle` 类仍然是主要短板。该类别在遥感图像中具有以下特点：

- 目标尺寸极小；
- 数量密集；
- 容易出现在 patch 边缘；
- 容易漏检；
- 对后续伪标签质量影响较大。

因此，`v1.3.0` 只引入两个轻量级数据层改动：

1. 提高 SAHI overlap；
2. 引入 vehicle hard tile mining。

本版本目标不是全面提升 overall mAP，而是验证：

> 增加 vehicle 困难样本曝光，是否能够提升 vehicle 类检测能力，同时保持整体 Teacher 性能不下降。

---

## 二、实验配置 / Experimental Setup

| 项目 | 设置 |
|---|---|
| 模型 | YOLOv8x |
| 数据集 | DOTA 五类子集 |
| 类别 | plane, ship, vehicle, harbor, storage_tank |
| 标签格式 | DOTA OBB → YOLO HBB |
| 数据策略 | SAHI 切图 + vehicle hard tile mining |
| 输入尺寸 | 1024 |
| batch | 4 |
| epochs | 100 |
| EarlyStopping | patience = 30 |
| 最优轮次 | epoch 37 |
| 训练时间 | 约 6.416 h |
| 训练图片数 | 10,466 |
| 验证图片数 | 1,094 |
| 验证目标数 | 98,473 |

---

## 三、整体结果 / Overall Results

| 指标 | 数值 |
|---|---:|
| Precision | 0.912 |
| Recall | 0.791 |
| mAP50 | 0.825 |
| mAP50-95 | 0.559 |

整体来看，`v1.3.0` 的性能与 `v1.2.0` 基本持平，说明：

> vehicle hard tile mining 和更高 overlap 没有破坏模型整体检测能力。

---

## 四、分类别结果 / Per-Class Results

| 类别 | Images | Instances | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|---:|---:|
| plane | 300 | 4,484 | 0.953 | 0.959 | 0.974 | 0.761 |
| ship | 295 | 18,375 | 0.956 | 0.835 | 0.865 | 0.581 |
| vehicle | 660 | 69,411 | 0.806 | 0.496 | 0.486 | 0.266 |
| harbor | 218 | 3,253 | 0.925 | 0.850 | 0.892 | 0.568 |
| storage_tank | 90 | 2,950 | 0.921 | 0.817 | 0.906 | 0.620 |

可以看到，`plane`、`ship`、`harbor`、`storage_tank` 均保持了较高性能，但 `vehicle` 仍然明显偏弱。

---

## 五、与 v1.2.0 对比 / Comparison with v1.2.0

| 版本 | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| v1.2.0 SAHI Teacher | 0.902 | 0.795 | 0.827 | 0.562 |
| v1.3.0 Vehicle Hard Mining | 0.912 | 0.791 | 0.825 | 0.559 |

变化情况：

| 指标 | 变化 |
|---|---:|
| Precision | +0.010 |
| Recall | -0.004 |
| mAP50 | -0.002 |
| mAP50-95 | -0.003 |

可以看出：

> v1.3.0 的整体指标与 v1.2.0 几乎一致，没有明显下降，也没有明显超越。

---

## 六、vehicle 类分析 / Vehicle Analysis

`v1.3.0` 的主要目标是改善 vehicle 类。

| 版本 | vehicle Precision | vehicle Recall | vehicle mAP50 | vehicle mAP50-95 |
|---|---:|---:|---:|---:|
| v1.2.0 | ~0.788 | ~0.506 | ~0.487 | ~0.270 |
| v1.3.0 | 0.806 | 0.496 | 0.486 | 0.266 |

结果显示：

- vehicle Precision 略有提升；
- vehicle Recall 略有下降；
- vehicle mAP50 基本不变；
- vehicle mAP50-95 略有下降。

因此，本次实验说明：

> 单纯提高 vehicle hard tile 的曝光率，并不能显著解决 vehicle 类检测问题。

vehicle 的瓶颈不只是“困难样本数量不足”，更可能来自以下因素：

- vehicle 本身尺度极小；
- 密集分布导致漏检；
- HBB 水平框对小目标定位精度有限；
- 密集目标可能受到 NMS 抑制；
- 模型对极小目标的特征提取仍然不足。

---

## 七、实验结论 / Conclusion

`v1.3.0` 证明了：

1. 提高 SAHI overlap 和引入 vehicle hard tile mining 不会破坏整体检测性能；
2. 模型在更偏向 vehicle 困难样本的数据分布下仍能保持稳定；
3. 但该策略没有显著提升 vehicle 类 AP；
4. vehicle 仍是后续伪标签生成和自适应学习中的关键瓶颈类别。

因此，本项目最终保留：

```text
v1.2.0 → 主 Teacher 模型
v1.3.0 → vehicle hard mining 消融实验
