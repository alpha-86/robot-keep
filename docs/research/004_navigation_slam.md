# 004_导航与SLAM技术调研

## 1. 概述

robot-keep项目长期需要室内导航能力。本文档调研现有开源方案。（注：一期不考虑车的问题，此部分为后续参考）

## 2. 核心需求

- 室内地图构建（SLAM）
- 定位与路径规划
- 障碍物检测与避障
- 成本可控

## 3. 开源方案对比

### 3.1 视觉SLAM

#### ORB-SLAM2/3
- 特征点法SLAM
- 单目/双目/RGB-D
- 成熟开源方案

**优点：**
- 精度高
- 开源成熟
- 应用广泛

**缺点：**
- 计算量大
- 依赖特征点质量

**GitHub：** https://github.com/UZ-SLAMLab/ORB_SLAM2

#### stella_vslam
- ROS集成版本
- 支持多种相机
- 实时性好

**GitHub：** https://github.com/stella-cv/stella_vslam

#### LIO-SAM
- 激光雷达+IMU融合
- 精度高
- 适合复杂环境

**GitHub：** https://github.com/TixiaoShan/LIO-SAM

---

### 3.2 低成本方案

#### RGB-D相机 + 视觉里程计
- Intel RealSense
- Azure Kinect
- Orbbec Astra

**组合：**
- RealSense D435i（深度+IMU）
- RTAB-Map（实时建图）

**GitHub：** https://github.com/introlab/rtabmap

#### 低成本移动机器人方案
- RGB-D相机 + 低端嵌入式
- 无需LiDAR
- 适合室内场景

**参考论文：** An Open-Source Low-Cost Mobile Robot System with an RGB-D Camera and Efficient Real-Time Navigation Algorithm

---

### 3.3 机器人导航框架

#### ROS/ROS2 Navigation Stack
- 标准机器人导航框架
- move_base / nav2
- 路径规划、定位、避障

#### home-robot (Facebook)
- 家庭环境机器人导航
- 视觉导航为主
- 移动抓取

**GitHub：** https://github.com/facebookresearch/home-robot

---

### 3.4 仿真环境

#### Gazebo
- 机器人仿真
- 支持多种机器人模型

#### Isaac Sim
- NVIDIA高保真仿真

---

## 4. 推荐方案（供后续参考）

### 一期：暂不涉及

### 二期/三期考虑

**方案A（低成本）：**
- RealSense D435i（深度相机）
- Jetson Nano/Orin Nano（计算）
- RTAB-Map（建图）
- ROS2 Nav2（导航）

**方案B（视觉为主）：**
- Orbbec Astra Pro
- Raspberry Pi 5 / Jetson
- ORB-SLAM3
- 纯视觉导航

### 长期演进

1. 集成毫米波雷达（低成本方案）
2. 考虑激光雷达（精度方案）

## 5. 成本估算

| 方案 | 硬件 | 成本 |
|------|------|------|
| 纯视觉SLAM | RealSense D435i + Jetson | ~$400 |
| 激光雷达方案 | 单线LiDAR + IMU + Jetson | ~$500-800 |
| 毫米波方案 | 毫米波雷达 + 相机 | 待研究 |

---

## 6. 与一期关系

**一期不涉及此部分**，一期聚焦于：
1. 人体识别
2. 坐姿检测
3. 语音交互

导航能力在后续迭代中加入。
