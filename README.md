# AEROBLADE 复现结果

复现 [AEROBLADE: Training-Free Detection of Latent Diffusion Images](https://github.com/umd-plum/aeroblade)（Ricker et al., CVPR 2024）：无训练潜扩散图像检测。本仓库包含本机全量复现的**检测性能**与**溯源分析**，以及对应的可视化页面（GitHub Pages）。

在线可视化页面：<https://lofiye.github.io/aeroblade-reproduction/>

## 复现结果（一句话）

假图由 SD-v1-4 本地生成、与 SD1 自编码器同分布时，**SD1-AE + lpips_vgg_2 检测最强**：

| 组合 | AUC | accuracy | AP | tpr@5%fpr |
|---|---:|---:|---:|---:|
| **SD1-v1-1 + lpips_vgg_2** | **0.988** | **96.0%** | **0.987** | **97%** |
| max（三模型池） | 0.970 | 95.0% | 0.957 | 83% |
| SD2-base + lpips_vgg_2 | 0.742 | 72.5% | 0.663 | 7% |
| Kandinsky + lpips_vgg_2 | 0.626 | 63.0% | 0.555 | 3% |

- accuracy 为 Youden 最优阈值下的分类准确率；tpr@5%fpr 为假阳率 5% 时的召回率。
- 溯源：**假图 100% 归到 SD1-AE（fraction=1.0）**，真图分散在 SD2（0.84）与 Kandinsky（0.17）。

## 与博客对齐口径的差异

博客（复现 AEROBLADE 论文）报告最优组合为 **kandinsky + lpips_vgg_2 → accuracy≈84%、tpr@5%fpr≈40%**。其假图由豆包在线生成、带文字水印且与三组 AE 均不同源；本复现的假图由 SD-v1-4 生成，与 SD1 同分布。因此：

1. 最优组合随假图来源变化——同源 AE 对同源生成器的检测最强（机制自洽，非复现失败）；
2. 数据更干净（无文字水印、完全同分布），指标整体高于博客口径（SD1 96%/97%，max 95%/83%）。

## 实验设置

| 项 | 值 |
|---|---|
| 真实图像 | Picsum 自然照片 100 张 |
| 假图像 | Stable Diffusion v1-4 本地生成 100 张 |
| AE 组件 | CompVis/stable-diffusion-v1-1、stabilityai/stable-diffusion-2-base、kandinsky-community/kandinsky-2-1（movq） |
| 距离度量 | lpips_vgg_2（负感知相似度） |
| 判定 | 单模型 + max 聚合，Youden 最优阈值 |
| 硬件 | RTX 5060 Laptop 8GB |
| 运行时长 | 600 次重建 + LPIPS ≈ 3 分 15 秒 |

## 复现方法（简述）

1. 环境：`uv` 管理虚拟环境（Python 3.13），`uv pip install` 安装官方仓库依赖（torch cu128 / diffusers / lpips 等）。
2. 下载三个 LDM 的 AE 组件（仅 VAE / movq，约 900MB，ModelScope 镜像加速）。
3. 官方 `aeroblade/image.py` 的 `create_pipeline` 增加**本地权重直连**分支：跳过 HF Hub 下载，直接从本地目录加载 `AutoencoderKL` / `VQModel`。
4. 运行官方 `experiments/01_detect.py` 全量检测，输出 `distances.parquet`、`detection_results.csv`、`attribution_results.csv`。
5. 由 `distances.parquet` 重新计算对齐指标（AUC / accuracy / AP / tpr@5%fpr），落盘 `alignment_metrics.csv`。

代码改造与本机实验脚本见 [kc_yunCompute 复现目录说明](#)（私有，未随仓库分发）。

## 文件结构

```
index.html                    可视化页面（GitHub Pages 入口）
results/
  alignment_metrics.csv       自算对齐指标（AUC/accuracy/AP/tpr@5%fpr）
  detection_results.csv       官方 01_detect 输出（AP/tpr5fpr）
  attribution_results.csv     官方溯源输出（fraction）
  distances.parquet           全量距离记录（真/假 × 3 AE）
```

## 参考

- 论文：Ricker, J., Lukovnikov, D., & Fischer, A. (2024). *AEROBLADE: Training-Free Detection of Latent Diffusion Images.* CVPR 2024. <https://arxiv.org/abs/2403.09243>
- 官方仓库：<https://github.com/umd-plum/aeroblade>
- 博客复现记录：<https://www.cnblogs.com/ljbguanli/p/19435639>

## License

MIT
