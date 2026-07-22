---
title: "Niagara与Material"
categories: [学习笔记]
tags: [Unreal, Niagara, Material, 特效]
draft: false
---

# Niagara与Material

## Sprite Rendering 的 Alignment

Sprite Rendering 的 Alignment（对齐）是非等比拉伸的黄金搭档。简单来说，Size 决定了粒子被拉得有多长，而 Alignment 决定了这个长条朝哪个方向转。特效优化和动态模糊，经常会用到这里的底层逻辑。

### 1. Unaligned（未对齐 / 默认模式）

粒子在原地“躺平”，不会根据自己飞行的方向旋转。它的旋转角度是固定的（默认朝上），或者通过 Sprite Rotation 节点指定角度。

适用场景：圆形、没有方向性的东西。比如漫天飘落的圆形雪花、圆滚滚的烟雾球、魔法阵的底图。因为它们是圆的，不怎么飞，看起来都一样，不需要跟着速度转。

### 2. Velocity Aligned（速度对齐 / 绝杀模式）

粒子的 X 轴和 Y 轴会死死盯住它当前运动的速度方向（Velocity）。它往哪飞，它的头就往哪边指。

适用场景：有明显方向感的运动物体。比如刀光挥舞甩出的火花（Sparks）、疾风骤雨中的雨滴、弓箭射出时的拖影。

高阶组合技：**Velocity Aligned + Non-Uniform Size = 完美的假运动模糊。** 当粒子速度越快时，把它拉得越长，并且让它顺着速度方向对齐，就能用极低的性能开销做出极其逼真的“高速残影”效果。

### 3. Custom Alignment（自定义对齐 / 硬核模式）

引擎把方向盘彻底交给你，需要自己提供一个 Vector3（三维向量）来告诉粒子该朝哪边指。它不管速度，也不管默认朝向，只听你算出来的那个向量。

适用场景：被外力强行扭曲的特效。比如粒子明明在往上飞，但受到一个神秘黑洞引力的影响，要求所有粒子必须用针尖指向黑洞中心。这时候就要在解算器里算出一个引力向量，然后喂给自定义对齐。

## 火花 Emitter 模板

- Noise Force：强度和频率参数。
- Engine Owner Position：粒子系统在世界中的位置。Simulation Position 是模拟位置，根据 Local Space 判断。
- Size by Speed：根据粒子速度缩放粒子尺寸。
- Initial Location to Velocity：创建基于初始位置的速度节点，以后经常会用到。
- Velocity、Facing：基于速度的朝向。
- Curve + Random：分配数值、得到权重数据的方法。

## 火花 Emitter 模板的材质

### Texture Sample（纹理采样）

提供火花的“基础形状”。RGB 往往不代表颜色，而是代表三个不同的黑白遮罩（Mask）。通过分别拖出 R 和 G 的线，提取出两种不同的火花形态，准备对它们进行混合。

### Dynamic Parameter（动态参数）

这是材质留给粒子系统（Niagara 或 Cascade）的“自定义遥控器”。材质本身是静态的，要想让每个火花粒子看起来不一样（有的模糊、有的清晰），就需要粒子系统实时传数据进来。

这个节点接收外部传来的数值（命名为 Blur），并将数值传递给 Lerp 的 Alpha，以此实时遥控火花的模糊程度。

### Lerp（Linear Interpolate，线性插值）

根据 Alpha 的指示，在 A（形状 1）和 B（形状 2）之间平滑切换。数学公式是：

```text
Output = A × (1 - Alpha) + B × Alpha
```

这是 Shader 中用得最多的节点。只要需要“从状态 1 过渡到状态 2”，第一时间就要想到 Lerp。

### Particle Color（粒子颜色）

获取粒子系统赋予该粒子的颜色和整体透明度。

- RGB 端口：直接连接最终材质的 Emissive Color（自发光颜色），决定火花是红色、黄色还是蓝色。
- Alpha 端口：代表粒子本身的生命周期透明度，比如火花诞生时不透明，快消失时逐渐变为全透明。

### Multiply（乘法）

将 Lerp 选出来的形状和粒子本身的透明度结合起来。黑色在计算机里的数值是 0，白色是 1，任何数乘以 0 都是 0（不可见）。

所以把 Particle Color 的透明度变化（A 端口）乘以计算好的火花形状（B 端口），就能得到最终的透明度遮罩，最后连入材质的 Opacity（不透明度）。

- 需要过渡和混合，找 Lerp。
- 需要控制强度，找 Multiply。
- 需要做时间动态，找 Time 配合 Sine。

## 数据覆盖陷阱

假设这个盒子里原本已经装好了数据：

```text
(X=1, Y=0.5, Z=2, W=0)
```

现在只想更新 W 格子里的数据，比如把 W 设置为粒子的当前速度。如果不加思考，直接用一个 Map Set 节点，把计算好的速度值硬塞给 `DynamicMaterialParameter3`，Niagara 会认为：“你要给我一个新的盒子。你只告诉了我新盒子的 W 是多少，没告诉我 X、Y、Z 是多少，那我就默认它们全是 0。”

结果就是原来的盒子被整个替换，变成：

```text
(X=0, Y=0, Z=0, W=新速度)
```

前三个格子里的数据瞬间消失，材质效果直接“爆炸”或变黑。

## Niagara 和材质的沟通

依照速度控制贴图读取，伪造更好看的动态模糊。

Niagara 和材质的沟通其实就是 Binding。使用 Dynamic Parameter，最多支持四组、每组四个通道。注意在节点内调用时不要覆盖其他值。

Particle Color 也是通过类似的方法传递信息，只是名字不同而已。如果可能，甚至可以强行覆盖一些属性来传递自定义值，例如将 Color 当作自定义 Density 使用。

## Curve Atlas

用颜色控制的方法：Curve Atlas。它是对 Niagara 进行更精细控制的一种方法。

![Curve Atlas 与 Niagara 的参数控制](/images/niagara-curve-atlas.png)

## Flare 主材质制作

### Mask Pick 节点

### DepthFade 节点

DepthFade 可以根据深度对透明物体做淡化。注意这依赖引擎的深度通道。根据 Alpha 的 DepthFade Function，你值得拥有。

![DepthFade 节点](/images/niagara-depth-fade.png)

要从物理的角度去理解所有特效，比如光斑效果是衰减的反比例函数。

![光斑的衰减与反比例函数](/images/niagara-flare-attenuation.png)

![Flare 材质参考](/images/niagara-flare-material.png)

![Flare 效果参考](/images/niagara-flare-preview.png)

## Lerp（Linear Interpolate，线性插值）

这是整个图形学、Shader 编写和特效制作中，最伟大、最常用、最核心的节点，没有之一。

数学逻辑是：

```text
Result = A × (1 - Alpha) + B × Alpha
```

也可以写成：

```text
Result = A + (B - A) × Alpha
```

Lerp 不只可以混合两种颜色，万物皆可 Lerp。

![Lerp 的使用例子](/images/niagara-lerp.png)

## Alpha 和 RGB 的真实关系

在图形学底层，Alpha 根本不是透明度，它只是一个权重系数（Weight / Factor），就是一个没有任何物理意义的浮点数（Float）。RGB 也不过是三个浮点数的集合 `(R, G, B)`。

### 当 Alpha 超出 0 到 1 的范围时

对于美术同学，可以这样理解公式 `A + (B - A) × Alpha`：`B - A` 是从 A 到 B 的距离，Alpha 可以理解为进度条。

![Alpha 超出 0 到 1 时的 Lerp](/images/niagara-alpha-rgb.png)
