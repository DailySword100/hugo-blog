---
title: "Unity Shader 入门精要学习笔记"
date: 2026-04-06
categories: [学习笔记]
tags: [Unity, Shader, 图形学]
draft: false
---

# Unity Shader 入门精要学习笔记

## Unity Shader 中 SubShader 的工作机制和选择逻辑

- SubShader 的基本规则：一个 Shader 文件可以包含一个或多个 SubShader，但最少要有一个
- SubShader 的基本规则：平台适配性（根据硬件内容）
- Unity 是如何选择 SubShader 的：在加载 Shader 时会遍历所有 SubShader，然后选择第一个可以在目标平台运行的，如果都不支持的话，会使用 FallBack 指定的 Shader
- 为什么需要多个 SubShader？因为不同显卡的能力不同，高端 PC 支持复杂计算、多 Pass、高级特效，低端手机只可以跑基础版本
- 每个 Pass 等于一次完整的渲染流程（顶点处理 → 片元处理 → 输出），所以多 Pass 会影响帧率性能

### 第一层：架构全景

![第一层架构全景](/images/image-1.png)

### 第二层：Pass 内部

![第二层Pass内部](/images/image-2.png)

### 第三层：常用语义/宏

![第三层常用语义宏](/images/image-3.png)

---

## SubShader 的 Tags 怎么选，用于控制渲染行为为引擎特性

- 高频
- Queue（渲染队列）：所用 控制渲染顺序，解决透明物体排序问题

![Queue渲染队列](/images/image-4.png)

- RenderType（渲染类型）：着色器分类，用于 Shader Replacement（着色器替换）着色器替换

![RenderType渲染类型](/images/image-5.png)

- 中频

- DisableBatching（禁用批量处理）：强制关闭动态批处理，强制移动动态批处理/静态批处理问题

![DisableBatching](/images/image-6.png)

- ForceNoShadowCasting（强制无阴影）：该物体不投射阴影

![ForceNoShadowCasting](/images/image-7.png)

- IgnoreProjector（忽略投影器）：不受 Projector（投影器）影响

![IgnoreProjector](/images/image-8.png)

![Tags总览表格](/images/image-9.png)
