# 002_坐姿检测技术调研

## 1. 概述

robot-keep项目需要具备坐姿检测能力，用于监测儿童在学习时的姿势，发现不良坐姿时进行语音提示。本文档调研现有开源方案。

## 2. 核心需求

- 人体姿态估计：检测人体关键点（头、肩、背、臀等）
- 坐姿分类：判断当前姿势是否正确
- 实时性：需要实时检测并及时反馈
- 成本可控：方案要适合家庭场景

## 3. 姿态估计开源方案对比

### 3.1 MediaPipe Pose

**技术栈：**
- Google出品
- BlazePose模型
- 支持TFLite/EdgeTPU

**能力：**
- 33个关键点（3D）
- 实时性好（手机30fps+）
- 支持多人

**优点：**
- 移动端性能优异
- 易于部署
- 预训练模型丰富

**缺点：**
- 精度略低于OpenPose
- 需要角度计算实现坐姿判断

**GitHub：** https://github.com/google/mediapipe

---

### 3.2 OpenPose

**技术栈：**
- CMU出品
- C++/Python
- CUDA加速

**能力：**
- 25个身体关键点（2D/3D）
- 手指关键点（21x2）
- 面部关键点（70）

**优点：**
- 精度高
- 开源成熟度高
- 应用广泛

**缺点：**
- 计算量大
- 需要GPU
- 实时性较差

**GitHub：** https://github.com/CMU-Perceptual-Computing-Lab/openpose

---

### 3.3 MoveNet

**技术栈：**
- TensorFlow Lite
- Google出品
- 专为移动端优化

**能力：**
- Lightning：17个关键点，快速
- Thunder：17个关键点，高精度

**优点：**
- 移动端速度快
- 可在树莓派运行
- 易与TensorFlow Lite集成

**缺点：**
- 关键点数量较少
- 需要额外逻辑判断坐姿

**模型下载：** TensorFlow Hub

---

### 3.4 YOLOv8 Pose

**技术栈：**
- Ultralytics YOLOv8
- 目标检测+关键点

**能力：**
- 人体检测 + 姿态估计
- 统一框架

**优点：**
- 速度快
- 易于部署
- 支持移动端（ONNX）

**缺点：**
- 需要自定义坐姿判断逻辑

---

### 3.5 现有坐姿检测开源项目

#### pose-monitor ("让爷康康")
- 安卓App
- MoveNet + 全连接网络分类
- 姿态分类：标准坐姿、翘二郎腿、脖子前倾驼背
- 纯离线运行
- GitHub: https://gitcode.com/gh_mirrors/po/pose-monitor

#### fix-posture
- Web摄像头方案
- TensorFlow.js + PoseNet/ml5.js
- 不良坐姿时模糊屏幕
- GitHub: https://github.com/monolesan/fix-posture

#### Standard Sitting (基于OpenPose)
- 卡内基梅隆大学OpenPose项目
- 坐姿分类：头部不正、身体不直、腰背弯曲
- 语音提示

---

## 4. 推荐方案

### 一期方案（验证阶段）

**技术选型：** MediaPipe Pose + 自定义角度判断逻辑

**理由：**
1. 移动端性能好，可在嵌入式设备运行
2. 关键点数量足够（33点）
3. 安装配置相对简单
4. 支持TFLite边缘部署

**坐姿判断逻辑：**
1. 头部前倾角度检测
2. 背部倾斜角度检测
3. 肩膀是否水平
4. 综合评分输出判断

### 二期考虑

1. 结合眨眼频率、眼睛闭合检测（疲劳判断）
2. 历史数据统计与分析
3. 多年龄段模型适配

## 5. 关键点角度计算

```
头部前倾 = atan2(耳朵.y - 肩膀.y, 耳朵.x - 肩膀.x)
背部倾斜 = atan2(肩膀.y - 臀部.y, 肩膀.x - 臀部.x)
```

## 6. 成本估算

| 方案 | 硬件要求 | 成本 |
|------|---------|------|
| MediaPipe | 手机/树莓派4 | 低 |
| OpenPose | GPU加速 | 高 |
| MoveNet | ARM/EdgeTPU | 低 |
