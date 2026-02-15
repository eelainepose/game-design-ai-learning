# Stable Diffusion 概念图生成指南

> 使用 AUTOMATIC1111/stable-diffusion-webui 生成游戏概念图

---

## 环境配置

### 本地部署
```bash
# 克隆仓库
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui

# 运行安装脚本
./webui.sh  # Linux/Mac
webui.bat   # Windows
```

### 推荐模型
- **通用概念图**：SDXL Base + Refiner
- **二次元风格**：Anything V5 / Counterfeit V3
- **写实风格**：Realistic Vision V5
- **游戏专用**：Game Icon Institute 系列

---

## 游戏概念图 Prompt 技巧

### 角色设计
```
Prompt:
masterpiece, best quality, concept art, character design,
(female warrior:1.2), armor, sword, fantasy style,
detailed face, intricate details, dramatic lighting,
white background, full body

Negative:
low quality, blurry, bad anatomy, watermark, signature
```

### 场景设计
```
Prompt:
masterpiece, best quality, concept art, environment design,
(fantasy forest:1.3), ancient ruins, magical atmosphere,
dense vegetation, rays of light, volumetric lighting,
8k, highly detailed

Negative:
low quality, blurry, modern buildings, people
```

### 道具/图标
```
Prompt:
masterpiece, game icon, (magic sword:1.4), glowing effect,
metallic texture, gem decorations, isolated on black background,
symmetrical, centered

Negative:
low quality, blurry, complex background, multiple objects
```

---

## 参数设置建议

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| Sampling Steps | 20-30 | 步数越多细节越多但速度越慢 |
| CFG Scale | 7-9 | 提示词遵循度，越高越严格 |
| Resolution | 512x512 / 1024x1024 | 根据需求调整 |
| Sampler | DPM++ 2M Karras | 质量与速度平衡 |

---

## 游戏设计工作流

### 1. 头脑风暴阶段
- 生成大量草图（低 step，快速迭代）
- 筛选方向，确定风格

### 2. 细化设计阶段
- 选定方向后生成高质量版本
- 使用 ControlNet 控制构图

### 3. 文档配图
- 生成 GDD 配图
- 统一风格保持文档一致性

---

## ControlNet 进阶技巧

### 线稿上色
1. 手绘线稿或提取线稿
2. 启用 ControlNet - Lineart
3. 配合适当提示词生成上色版本

### 姿态控制
1. 使用 OpenPose 提取姿态
2. ControlNet - OpenPose
3. 生成相同姿态的不同角色

### 深度图控制
1. 生成或绘制深度图
2. ControlNet - Depth
3. 保持空间结构一致

---

## InvokeAI 备选方案

如果 WebUI 太复杂，可以试试 **InvokeAI**：
- 更简洁的界面
- 更好的画布模式
- 适合概念设计工作流

```bash
git clone https://github.com/invoke-ai/InvokeAI.git
```

---

## 常见问题

**Q: 生成的图质量不高？**
A: 检查提示词权重，增加 `masterpiece, best quality`，调整 CFG Scale

**Q: 角色手指畸形？**
A: 添加 `perfect hands, detailed fingers` 到提示词，或使用负提示 `bad hands`

**Q: 如何保持角色一致性？**
A: 使用 LoRA 训练角色，或使用 Reference ControlNet

---

*待补充实际生成案例...*
