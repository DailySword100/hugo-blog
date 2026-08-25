---
title: "雪景环境Demo开发文档"
categories: [个人作品开发文档]
tags: [Unreal Engine 5, 环境制作, PCG, Material, Render Target, VHM, RVT, 交互雪]
draft: false
---

# 雪景环境Demo开发文档

此博客为AI工作流自动管理，由博主的个人笔迹与GPT对话习惯自动整理，为博主复习整理查阅，必有错误，请注意鉴别是否AI幻觉。

## 1. 项目说明

这个 Demo 不只是交互雪效果，而是一套完整的雪山环境：先在 Houdini KTT 中程序化制作山体并输出遮罩，再把结果导入 UE，制作雪山地表材质，使用 UE5 PCG 布置树木、灌木、草和岩石，最后把交互雪接入可以行走的区域。

交互雪是这个雪山 Demo 的一个子系统。它参考公开技术文章完成复现，不是从零原创方案；我真正做的工作是把这套方案接入自己的场景，并处理材质、Render Target、VHM、RVT 与现有环境之间的关系。

从项目现有资源反查，场景的组成可以分为四层：

```text
Houdini KTT 程序化山体 → UE Landscape
    ↓
雪地 / 裸岩混合材质
    ↓
PCG 树木、灌木、草和岩石
    ↓
角色附近的交互雪区域
```

## 2. 山体与资源导入

### 使用 Houdini KTT 程序化制作山体

山体由 Houdini KTT 插件程序化制作，因此它本身也属于广义的 PCG（Procedural Content Generation）。这里需要和 UE5 中名为 PCG Framework 的工具区分开：Houdini KTT 负责山体地形的程序化生成与遮罩输出；项目中的四张 `PCG_*` Graph 则读取导入 UE 后的 Landscape，在山体表面布置树木、灌木、草和岩石。

项目里的四张 PCG Graph 都先使用 `Get Landscape Data` 获取关卡中的 `LandscapeProxy`，再在 Landscape 表面采样点。它们使用的是 Houdini 阶段已经确定的山体，而不是重新生成山形。

目前项目中保留了 `Snowy_Mountain_Map_Polish01`、`Snowy_Mountain_Map_SnowTest` 和 `完整雪山` 等不同阶段的关卡。正式整理时以 `Snowy_Mountain_Map_Polish01` 和 `完整雪山` 中仍在使用的资源关系为准，测试关卡只用于验证材质和交互雪。

### 从 Houdini 到 UE 的导入阶段

当前山体流程是：

```text
在 Houdini KTT 中程序化生成山体
    ↓
根据山体结构输出遮罩图
    ↓
将山体结果与遮罩导入 UE
    ↓
在 UE 中形成可供材质和 PCG 读取的 Landscape
    ↓
使用遮罩控制雪地、裸岩等地表区域
    ↓
使用 UE5 PCG Graph 布置植被和岩石
```

Houdini 阶段不仅决定山体轮廓，也负责把后续材质需要的结构信息拆成遮罩。UE 项目中保留了 `mountain_mask` 和 `T_SnowMountain_TerrainStructure`，用于承接山体区域信息，让雪岩分布能够跟随山体结构，而不是在 UE 中重新手绘一遍。

### 导入后实际保留下来的内容

冬季环境资源集中在 `NaturePack_WinterEnvironment` 目录，里面包含树木、灌木、草、岩石、地表贴图和相关材质。导入后没有直接在原资源目录里反复修改，而是在自己的 `RPG/Environment` 目录下建立 PCG Graph、雪山材质和交互雪资产，再引用资源包中的网格。

这一步的整理方式对后续维护很重要：

- 原始资源包负责提供可复用的 Mesh 与 Texture；
- `RPG/Environment/PCG` 保存我自己的散布规则；
- `RPG/Environment/Snow` 保存雪山材质与贴图；
- `RPG/Environment/DeformableSnow` 保存交互雪系统；
- 不同阶段的 Map 用于隔离场景搭建、材质测试和交互雪调试。

当前文档已经确认山体来自 Houdini KTT，并确认 UE 中存在对应的山体结构遮罩；但项目目录里没有找到可直接核对的 Houdini 工程文件和导出参数。因此暂时不填写遮罩分辨率、输出格式、Landscape 尺寸与 Z Scale，等源工程重新归档后再补，避免把记忆中的参数写成最终标准。

## 3. 雪山材质

场景实际引用的是材质实例 `MI_SnowMountain_TerrainRock`。它的父材质不是同目录下两个未被关卡引用的测试材质，而是交互雪目录中的 `M_SnowGround_Deformable`。因此雪山基础材质与交互雪不是两套完全分离的表面：山体使用同一套可变形雪材质，再通过实例参数补充裸岩表现。

这个材质实例当前使用 Slate 岩石贴图，并开放了几组直接影响雪岩分布的参数：

| 参数 | 当前值 | 作用 |
| --- | ---: | --- |
| `Rock_Overall_Amount` | 1.00 | 控制整体裸岩量 |
| `Rock_Ridge_Strength` | 1.18 | 加强山脊位置的岩石 |
| `Rock_Foot_Strength` | 0.82 | 控制山脚区域的岩石 |
| `Snow_Concavity_Protection` | 0.48 | 让凹陷区域更容易保留积雪 |
| `Rock_World_Texture_Size` | 1800 | 控制岩石纹理的世界空间尺度 |

这里的重点不是把雪和石头简单二选一，而是用山体结构组织分布：山脊、山脚和凹处使用不同强度，减少整座山均匀铺雪或均匀露岩的人工感。材质使用世界尺度控制岩石纹理，也能避免不同大小的山体因为 UV 比例不同而出现密度跳变。

`RPG/Environment/Snow` 中还保留了雪的 Base Color、Normal、Height、Roughness、AO，以及 `T_SnowMountain_TerrainStructure` 结构遮罩。这些资源属于雪山表面制作资料；但同目录的 `M_SnowMountain_TerrainRock` 与 `M_SnowMountain_TerrainRock1` 当前没有关卡引用，所以在文档中把它们记录为实验资产，不把它们写成最终场景方案。

## 4. 使用 PCG 布置雪山环境

树、灌木、草和岩石分别使用四张 PCG Graph，而不是全部塞进一张图。四张图的主体流程相同：

```text
获取重叠范围内的 Landscape
    ↓
Surface Sampler 在山体表面生成候选点
    ↓
读取带指定 Tag 的排除区域并做 Difference
    ↓
Attribute Noise 写入随机 Density
    ↓
Density Filter 保留需要的密度区间
    ↓
Transform Points 随机旋转和缩放
    ↓
Self Pruning 清理相互穿插的点
    ↓
Static Mesh Spawner 按权重选择网格
```

### 四类物体的实际参数

| PCG Graph | 使用资源 | 表面采样密度 | 点范围 | Density 区间 | 随机缩放 |
| --- | --- | ---: | --- | --- | --- |
| `PCG_SnowForest_Test` | 7 种雪松网格 | 0.01 | 250 × 250 × 100 | 0.60–1.00 | 0.80–2.00 |
| `PCG_SnowBushes` | 7 种灌木网格 | 0.02 | 100 × 100 × 100 | 0.45–0.80 | 0.65–1.15 |
| `PCG_SnowGrass` | 4 种雪地草网格 | 0.03 | 80 × 80 × 30 | 0.40–1.00 | X 0.90–1.40，YZ 0.75–1.40 |
| `PCG_SnowRocks` | 3 种岩石网格 | 0.01 | 250 × 250 × 100 | 0.28–0.48 | 0.80–1.60 |

所有类型都允许 `0–360°` 的随机 Yaw，并使用 `Large to Small` 的 Self Pruning，半径相似系数为 `0.25`。这样先保留较大的占位，再移除相互挤压的小点，能减少树木、灌木和岩石明显穿插。

不同物体没有只靠 Mesh 大小区分密度。草的采样最密、点范围最小；树木和岩石的点范围更大，避免大型物体靠得太近；灌木位于两者之间。Density 区间进一步把同一份随机噪声切成不同分布，防止四类物体完全重叠。

### 排除区域

树木、灌木和草会读取带 `PCG_Exclude` Tag 的 Actor，并通过 Difference 从 Landscape 候选区域中减去；岩石单独读取 `PCG_RockExclude`。这让我可以在道路、角色活动区、交互雪展示区或构图留白处摆放简单的排除体积，而不必回到 Graph 中手工删除生成点。

四张 Graph 都被 `Snowy_Mountain_Map_Polish01`、`Snowy_Mountain_Map_SnowTest` 和 `完整雪山` 等关卡引用。环境布置因此不是一次性烘死在某一张地图里，而是同一套规则在不同测试阶段复用。

## 5. 交互雪

我做的不是播放一张固定脚印贴图，而是把角色与雪地的接触信息实时写入高度数据。角色移动以后，雪地材质继续读取之前留下的轨迹，并把它表现为连续的凹陷、法线变化和受压区域的材质变化。

当前已经打通的基础闭环是：

```text
角色与雪地接触
    ↓
整理接触位置、范围、深度和硬度
    ↓
世界坐标转换为雪地区域 UV
    ↓
本帧接触写入 RT_Current
    ↓
RT_Current 与移动修正后的 RT_History 合并到 RT_Trails
    ↓
RT_Trails 复制回 RT_History，供下一帧使用
    ↓
雪地材质采样轨迹，计算高度、法线和表面变化
    ↓
VHM 显示连续的雪地形变
```

### 为什么需要移动式 Render Target

如果给整个大型雪景准备一张高分辨率 Render Target，尺寸和性能都不太现实。所以我用角色附近的一块正方形区域记录交互，把有限的纹理精度集中在玩家周围。

这种做法的问题是：记录区域会跟着角色移动。历史轨迹如果仍然按照旧 UV 直接采样，看起来就会跟着 RT 中心一起滑走。

![旧轨迹错误地跟随 RT 中心移动](/images/snow-environment-demo/history-offset-error.png)

因此每次中心变化后，都要根据新旧中心的世界坐标差重新计算历史 UV 偏移。这里真正需要保持不动的是世界空间里的脚印，而不是纹理里的像素位置。

### 获取接触信息

系统首先检测角色或其他物体与雪地的接触。每次有效接触整理为一组材质可以使用的数据：

- 世界空间接触位置；
- 接触范围；
- 压入雪地的深度；
- 接触方向；
- 轨迹边缘的软硬程度。

这些数据决定轨迹写入 RT 时的位置、大小、形状和强度。相比直接播放脚印贴图，这种方式能记录连续移动，也更容易适配不同大小的交互物体。

### 世界坐标转换为 RT UV

Render Target 保存的是二维 UV，角色接触点来自三维世界坐标，因此需要做一次映射。

以角色附近的雪地区域为记录范围，把区域中心视为 UV 中心 `(0.5, 0.5)`：

```text
ContactUV = (ContactWorldPosition - SnowAreaCenter) / SnowAreaSize + 0.5
```

只有落在有效 UV 范围内的接触才写入 RT。这样不需要为整个场景准备一张超大纹理。

### 三张 Render Target 各做什么

#### RT_Current：只记录这一帧

每帧开始先清空 `RT_Current`，再用 `M_TrailDrawer` 根据接触点的 UV、范围、深度和硬度绘制本帧轨迹。

这一张图只回答：**这一帧有哪些地方刚刚受到挤压？**

如果只有它，轨迹下一帧就会消失。

#### RT_History：保存上一帧结果

`RT_History` 是系统的记忆，保存上一帧已经累计完成的轨迹。读取它时，需要加上记录中心移动产生的 UV 偏移，否则旧轨迹会错误平移。

#### RT_Trails：给雪地材质使用的最终结果

合并材质把当前帧与经过移动修正的历史数据组合：

```text
RT_Trails = Merge(RT_Current, Offset(RT_History))
```

- `RT_Current` 提供本帧新增形变；
- `RT_History` 提供之前留下的轨迹；
- 历史 UV 偏移修正记录中心移动；
- 合并规则决定同一位置反复受压时如何累计或覆盖。

合并以后，再把 `RT_Trails` 复制回 `RT_History`：

```text
RT_Current → RT_Trails → RT_History → 下一帧 RT_Trails
```

### 雪地材质如何使用轨迹

雪地材质通过 `MF_SnowTrailsSample` 读取最终轨迹，把灰度解释为雪地受压程度。

#### 高度形变

轨迹值被转换为向下的高度偏移。没有被踩踏的位置保持原有雪层高度，轨迹区域根据强度向下凹陷。

#### 法线变化

根据轨迹高度差重建局部法线，让凹陷边缘能够正确响应光照。法线构建接入后，我遇到过 RT 内外表面表现不一致的问题，这说明高度闭环打通以后，法线采样空间仍然需要单独核对。

![构建法线后 RT 内外表现不一致](/images/snow-environment-demo/normal-mismatch.png)

#### 表面材质变化

轨迹遮罩还可以调整受压区域的颜色、粗糙度和细节强度，让踩实的雪和周围松软雪面产生区别。

### 为什么最后选择 VHM

最开始轨迹虽然能写入，但地形的几何密度不够，印记不明显，形状也过于规整。这里真正缺少的不是更强的颜色对比，而是承载雪变形的顶点密度。

当时考虑过两个方案：

1. Virtual Heightfield Mesh；
2. Nanite 网格代理配合材质 World Position Offset。

当前实现选择 VHM。WPO 告诉顶点应该移动多少，VHM 则提供足够密、适合高度场的几何网格。

![接入 VHM 后的初始雪地形变](/images/snow-environment-demo/initial-deformation.png)

VHM 主要承担：

- 承载雪面的高度变化；
- 显示 RT 生成的连续轨迹；
- 配合法线、颜色和粗糙度增强立体感；
- 在较大雪地区域维持相对连续的表面细节。

### 每帧运行顺序

每一帧按下面的顺序更新：

1. 检测角色或物体是否与雪地发生有效接触；
2. 获取接触位置、范围、方向、深度和硬度；
3. 将世界坐标转换为当前记录区域的 UV；
4. 清空 `RT_Current`；
5. 使用 `M_TrailDrawer` 把本帧接触写入 `RT_Current`；
6. 根据记录中心移动计算历史 UV 偏移；
7. 使用合并材质，把 `RT_Current` 与修正后的 `RT_History` 写入 `RT_Trails`；
8. 把合并结果复制到 `RT_History`；
9. 基于稳定的轨迹结果计算凹陷、两侧堆雪和表面细节；
10. 雪地材质采样最终结果，由 VHM 显示形变。

这里最容易出错的是读写顺序。不能同时读取和写入同一张 `RT_Trails`，也不能把模糊、堆雪等处理结果不断累积进未清空的历史纹理，否则 `SpiralBlur` 会逐帧叠加，轨迹会越来越软，甚至像液体一样流动。

### 轨迹从规则 Stamp 到两侧堆雪

基础闭环完成后，效果仍然很像一个个圆形 Stamp：中间只是规则地下陷，两边没有被挤开的雪，两脚之间也缺少未接触的隆起。

![规则 Stamp 阶段的轨迹](/images/snow-environment-demo/regular-stamp.png)

当前的堆雪思路是：

```text
SpiralBlur(TrailsBuffer) - TrailsBuffer
    ↓
得到轨迹外侧隆起区域
    ↓
对隆起区域整形
    ↓
加入基于世界坐标采样的雪噪声
    ↓
与原始凹陷统一记录为高度
```

高度约定为：原始雪面等于 `1`，两侧堆雪大于 `1`，凹陷区域小于 `1`。

噪声必须根据当前位置重建世界坐标采样，不能直接使用跟随角色移动的 TexCoord，否则噪声也会随着 RT 滑动。

目前已经能得到连续形变，但真实踩雪应该有脚步间隔，不能始终连成一条沟。后续还需要把连续接触改成与实际步态对应的离散接触。

![当前阶段的连续脚印效果](/images/snow-environment-demo/footprint-result.png)

### 移动 RT 边缘的轨迹截断

移动式 RT 还有一个很明显的问题：旧脚印离开记录范围后，数据会突然消失，雪地瞬间恢复，画面上会出现生硬的截断。

当前先使用边缘渐变遮罩，让轨迹靠近 RT 边界时逐渐恢复，而不是直接消失。

![移动 RT 的边缘恢复遮罩](/images/snow-environment-demo/rt-edge-fade.png)

这只是当前方案。另一种思路是把离开 RT 的旧痕迹继续保存，再经过一段时间缓慢恢复。这个方案需要继续设计历史数据的保存范围，目前还没有完成，因此不把它写成已经解决的功能。

## 6. 开发过程中遇到的问题

### VHM 随镜头移动被错误剔除

现象是一块地形会在镜头移动时突然消失。最初怀疑是 LOD，调整后没有改善，随后转向检查 VHM Bounds。清空 MinMax Texture 后问题消失，因此问题与 VHM 高度范围或剔除数据有关。

![排查 VHM 的 MinMax Texture](/images/snow-environment-demo/vhm-bounds-minmax.png)

### 阴影区域出现黑色噪点

黑点定位到 Lumen Short Range AO：高频法线和微小形变被放大成颗粒遮蔽。调试时可以使用下面的命令关闭它，确认噪点是否来自这里：

```text
r.Lumen.ScreenProbeGather.ShortRangeAO 0
```

![Lumen Short Range AO 产生的黑色噪点](/images/snow-environment-demo/lumen-ao-noise.png)

这条命令更适合定位问题，并不等于最终项目一定要永久关闭 Short Range AO。最终仍要在画面质量和雪面法线细节之间继续平衡。

### 场景突然出现大面积黑块

排查时先把 VHM 切换成默认材质，异常随材质变化消失；继续隔离 Base Color 后，黑块也消失，范围缩小到 `M_VHM_Snow` 读取的 RVT Base Color。

![RVT 数据异常造成的黑块](/images/snow-environment-demo/rvt-black-block.png)

![替换默认材质继续隔离问题](/images/snow-environment-demo/material-isolation.png)

最后发现 `RVT_SnowSurface` 被误删，重新加回后恢复正常。这个问题本身不复杂，但排查顺序值得保留：先替换默认材质区分几何与材质，再隔离材质通道，最后检查对应 RVT 资产是否存在。

## 7. 关键资产职责

| 资产 | 作用 |
| --- | --- |
| `MI_SnowMountain_TerrainRock` | 雪山当前使用的雪地 / 裸岩材质实例 |
| `PCG_SnowForest_Test` | 在 Landscape 上生成雪松 |
| `PCG_SnowBushes` | 在 Landscape 上生成灌木 |
| `PCG_SnowGrass` | 在 Landscape 上生成草簇 |
| `PCG_SnowRocks` | 在 Landscape 上生成岩石，并使用独立排除 Tag |
| `RT_Current` | 保存当前帧新产生的接触数据 |
| `RT_Trails` | 保存当前帧与历史数据合并后的最终轨迹 |
| `RT_History` | 保存上一帧轨迹，为下一帧累计提供数据 |
| `M_TrailDrawer` | 根据接触位置、范围、深度和硬度绘制本帧轨迹 |
| `M_HistoryMerge` / `M_TrailsOperation` | 合并当前轨迹和经过位置修正的历史轨迹 |
| `M_HistoryCopy` | 把本帧最终结果复制为下一帧历史数据 |
| `MF_SnowTrailsSample` | 在雪地材质中采样并解析轨迹纹理 |
| `M_SnowGround_Deformable` | 把轨迹应用到雪地表面表现 |
| `M_VHM_Snow` | VHM 使用的雪地显示材质 |
| `RVT_SnowHeight` | 为 VHM 提供雪地高度数据 |
| `RVT_SnowHeight_MinMax` | 描述或辅助解析高度范围 |
| `RVT_SnowSurface` | 向 VHM 传递雪地表面信息 |

## 8. 当前完成度与后续方向

目前已经完成雪山关卡基础、雪岩材质、四类 PCG 环境布置，以及从接触信息、二维轨迹、历史累积到 VHM 雪地形变的基础闭环。交互雪已经开始处理两侧堆雪、法线和表面变化，但山体导入源文件与可重复导入参数还需要重新归档。

接下来主要优化：

1. 修正凹陷截面，让雪面缓慢下降，而不是突然掉下去；
2. 加宽两侧堆雪并降低峰值；
3. 让相邻脚印自然融合，但仍保留真实步态间隔；
4. 增加颗粒、碎边和蓬松雪；
5. 设计离开移动 RT 范围后的轨迹保存与恢复；
6. 补充性能数据，评估 RT 分辨率、更新频率、VHM 密度和模糊采样次数。

现在这个效果仍属于学习和复现阶段。复杂物体接触、边缘堆雪、轨迹恢复和性能控制都还没有完成，不能把当前版本描述成完整的通用交互雪方案。

## 作品集一句话描述

使用 Houdini KTT 程序化生成雪山并输出地形遮罩，在 UE5 中完成雪地 / 裸岩材质与四套 PCG 环境布置，并把基于 Render Target、RVT 与 VHM 的交互雪系统接入可行走区域。
