---
title: "Unity Shader 入门精要学习笔记"
date: 2026-04-06
categories: [学习笔记]
tags: [Unity, Shader, 图形学]
draft: false
---

# Unity Shader 入门精要学习笔记

![思维导图](/images/image-1.png)

## SubShader 的工作机制和选择逻辑

### SubShader 的基本规则
一个 Shader 文件可以包含一个或者多个 SubShader，但最少要有一个。

![SubShader基本规则](/images/image-2.png)

### Unity 是如何选择 SubShader 的
在加载 Shader 时会遍历所有 SubShader，然后选择第一个可以在目标平台运行的。如果都不支持的话，会使用 FallBack 指定的 Shader。

![SubShader选择逻辑](/images/image-3.png)

### 为什么需要多个 SubShader？
因为不同显卡的能力不同：
- **高端 PC**：支持复杂计算、多 Pass、高级特效
- **低端手机**：只可以跑基础版本

![多SubShader需求](/images/image-4.png)

### Pass 的作用
每个 Pass 等于一次完整的渲染流程（顶点处理 → 片元处理 → 输出），所以多 Pass 会影响帧率性能。

![Pass渲染流程](/images/image-5.png)

---

## 层次结构

![层次结构](/images/image-6.png)

### 第一层：架构全景
![第一层架构全景](/images/image-7.png)

### 第二层：Pass 内部
![第二层Pass内部](/images/image-8.png)

### 第三层：常用语义/宏
![第三层常用语义](/images/image-9.png)

---

## SubShader 的 Tags（标签）系统

Tags 用于控制渲染行为和引擎特性。

### 高频 Tags

#### Queue（渲染队列）
**作用**：控制渲染顺序，解决透明物体排序问题

#### RenderType（渲染类型）
**作用**：着色器分类，用于 Shader Replacement（着色器替换）

### 中频 Tags

#### DisableBatching（禁用批量处理）
**作用**：强制关闭动态批处理/静态批处理

#### ForceNoShadowCasting（强制无阴影）
**作用**：该物体不投射阴影

#### IgnoreProjector（忽略投影器）
**作用**：不受 Projector（投影器）影响

---

> **当前文档**：18 条主题，共 326 字
