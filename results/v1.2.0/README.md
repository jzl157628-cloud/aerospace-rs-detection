# v1.2.0 - SAHI 切图增强 Teacher

## 一、版本定位

`v1.2.0` 是本项目的第二阶段 Teacher 模型版本。

该版本在 `v1.1.0` 原图监督基线的基础上，引入 **SAHI-style slicing / patch training**，将原始遥感大图切分为多个局部 patch 后再进行 YOLOv8x 训练。

本版本的核心目标是解决遥感目标检测中的小目标问题：

- 原图直接 resize 到 1024 后，小目标特征被压缩；
- vehicle、ship、storage_tank 等目标在整图中占比很小；
- 密集目标区域容易出现漏检；
- 直接原图训练时 mAP50 和 mAP50-95 明显受限。

因此，`v1.2.0` 通过切图方式重构训练数据表达，使模型能够在局部 patch 中看到更清晰的小目标特征。

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
| Best Epoch | 37 |
| Training Time | ~5.487 h |

---

## 三、切图数据统计

### Train split

| 项目 | 数值 |
|---|---:|
| Source Images | 1129 |
| Patch Images | 9005 |
| Object Patches | 8616 |
| Empty Patches | 389 |

类别实例统计：

| Class | Instances |
|---|---:|
| plane | 14484 |
| ship | 59011 |
| vehicle | 239916 |
| harbor | 11114 |
| storage_tank | 8840 |

### Val split

| 项目 | 数值 |
|---|---:|
| Source Images | 282 |
| Patch Images | 985 |
| Object Patches | 800 |
| Empty Patches | 185 |

类别实例统计：

| Class | Instances |
|---|---:|
| plane | 3207 |
| ship | 14728 |
| vehicle | 56653 |
| harbor | 2617 |
| storage_tank | 2173 |

标签检查结果：

```text
class_ids = [0, 1, 2, 3, 4]
bad labels = 0
