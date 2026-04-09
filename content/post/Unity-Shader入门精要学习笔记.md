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

---

*2026.4.9 更新*

## Unity Shader 的形式

### Unity 的宠儿：表面着色器

适用于快速实现光照效果。核心特点是 Unity 自动生成多 Pass 代码，只需定义表面属性（颜色、光滑度、法线），光照计算由 Unity 内置处理。无需关心多光源，适合做：

- PBR 材质（金属、光滑度）
- 简单的卡通渲染
- 快速原型验证

### 最聪明的孩子：顶点/片元着色器

核心特点是完全手动控制渲染流程，可以直接编写顶点和片元着色器，但需要显式定义 Pass。

### 应该选择哪种 Unity Shader 的形式？

除非有非常明确的需求必须使用固定函数着色器（如需要在非常旧的设备上运行，这类设备非常少见），否则请使用可编程管线的着色器，即表面着色器或顶点/片元着色器。

- 如果你想和各种光源打交道，你可能更喜欢使用**表面着色器**，但需要小心它在移动平台的性能表现
- 如果你需要使用的光照数目非常少，如只有一个平行光，那么使用**顶点/片元着色器**是一个更好的选择
- 最重要的是，如果你有很多自定义的渲染效果，那么请选择**顶点/片元着色器**

### 此书学习使用的 Unity Shader 形式

学习此书的目的不仅在于教给读者如何使用 Unity Shader，更重要的是想让读者掌握渲染背后的原理。仅仅了解高层抽象虽然可能会暂时使工作简化，但从长久来看"知其然不知其所以然"所带来的影响会更加深远。

---

## Unity Shader 不一定是真正的 Shader

Unity Shader 并不等于第 2 章所讲的 Shader，尽管 Unity Shader 翻译过来就是 Unity 着色器。在 Unity 里，Unity Shader 实际上指的就是一个 ShaderLab 文件——硬盘上以 `.shader` 作为文件后缀的一种文件。

与传统 Shader 相比，Unity Shader 的优势：

| 对比项 | 传统 Shader | Unity Shader |
|---|---|---|
| 着色器类型 | 只能编写特定类型（顶点/片元等） | 同一文件可同时包含顶点和片元着色器 |
| 渲染设置 | 需要在另外的代码中手动设置（混合、深度测试等） | 一行指令即可完成 |
| 输入输出 | 需要编写冗长代码，手动处理位置对应关系 | 在特定语句块声明属性，依靠材质方便修改 |

> 高度封装性的代价：对于一些类型的 Shader，例如曲面细分着色器等，Unity 的支持就差一些。

### Unity Shader 和 CG/HLSL 之间的关系

Unity Shader 是用 ShaderLab 编写的，但对于表面着色器和顶点/片元着色器，我们可以在 ShaderLab 内部嵌套 Cg/HLSL 代码。Cg/HLSL 代码嵌套在 `CGPROGRAM` 和 `ENDCG` 之间。

> 从本质上来说，Unity 中只存在顶点/片元着色器。

---

## 第四章：学习 Shader 所需的数学基础

### 笛卡尔坐标系

分为左手坐标系和右手坐标系，**Unity 是左手坐标系**。

### 点和矢量

| | 点 | 矢量 |
|---|---|---|
| 含义 | 位置 | 方向 + 大小 |
| 运算 | 无加减 | 可加减、数乘、点乘、叉乘 |
| 齐次坐标 | (x, y, z, **1**) | (x, y, z, **0**) |

- **矢量的模**：矢量的长度
- **矩阵**：对空间进行线性操作的工具，向量 × 矩阵 = 变换后的新向量

### 什么是变换？

**线性变换**：保留矢量加法和标量乘法的变换，特点是直线还是直线，原点不动。

**仿射变换**：线性变换 + 平移，最常用，旋转、缩放、平移的组合。

**齐次坐标**：在三维空间中，平移不是线性的，所以要升维到 4×4 矩阵。

> "在顶点着色器流水线阶段，做的工作就是把模型的顶点坐标从模型空间转换到齐次裁剪坐标空间。"

### MVP 变换的完整链条

$$\text{模型空间} \rightarrow \text{世界空间} \rightarrow \text{观察空间} \rightarrow \text{裁剪空间} \rightarrow \text{NDC 空间} \rightarrow \text{屏幕空间}$$

### 为什么需要这么多坐标空间？

- 让每个空间只处理特定的问题
- 某些概念只在特定空间有意义（如法线在模型空间定义）
- 硬件适配需要，最终都要映射到屏幕像素

---

> 📌 因为自己比较容易焦虑、急，所以看看大佬的知乎放在这里，回看时鼓励一下自己。
