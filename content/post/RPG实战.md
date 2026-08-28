---
title: "RPG实战"
categories: [游戏客户端开发]
tags: [Unreal Engine 5, RPG, 客户端开发]
weight: 40
draft: false
---

# RPG实战

此博客为AI工作流自动管理，由博主的个人笔迹与GPT对话习惯自动整理，为博主复习整理查阅，必有错误，请注意鉴别是否AI幻觉。

这篇文章集中记录 RPG 项目开发过程中的系统设计、实现思路和待补知识。内容按照实际功能拆分为道具、基础功能、战斗、UI、AI 与常用蓝图节点六部分，保留开发时的思考和问题，后续会随着项目推进继续补充。

## 道具系统

### 结构选择
- 全平展：车花

### 合成、分解与堆叠
- 需要掌握TileView和ListView的与ListEntry的数据交互 用数据列表来处理一些功能就会比较简单

## 基础功能开发

### UI 设计原则
简洁性：尽管提供必要的信息很重要，但过度复杂的UI设计会分散玩家的注意力，甚至造成困扰。良好的UI设计应该力求简洁，避免不必要的修饰和元素。
- 清晰性：UI的首要任务是传达信息，因此它必须清晰易懂。所有的元素都应该一目了然，玩家能够轻松识别各种图标，指示器和文字。
- 所见即所得
- 一致性和标准化：用户不应该不断重新学习新的操作方式或界面布局。保持UI元素在不同界面和环境中的一致性，可以帮助玩家建立对游戏操作的直觉理解。颜色字体，图标样式等的一致性，能够让玩家更快适应游戏环境。
- 反馈及时性： 用户的任何操作都应该获得及时的反馈
- 视觉层次和导航清晰
- 美观性：除了功能性以外 UI的视觉设计应该与主题风格保持一致
- 尺寸使用8的倍数：绝大多数的设备屏幕都可以被8整除

### Unreal 编码约定
- 类型名前缀需要使用额外的大写字母，用于区分其和变量名。例如FSkin为类型名，而Skin则是FSkin的实例。
- 模板类的前缀为T
- 继承自UObject的前缀为U
- 继承自AActor的前缀为A
- 继承自SWidget的类前缀为S
- 抽象界面类的前缀为I
- Epic提供的概念类型的类（用作TModels的第一个参数），其前缀为C
- 枚举的前缀为E
- 布尔变量必须以B

![RPG 实战笔记配图](/images/rpg-practice/lifetime-component.png)

### 生命周期组件
- API
- TMap容器：虚幻引擎的键值对映射容器，类似c++的std：：map ，提供高效的查找和插入操作
- Transient属性：标记属性不参与序列化 适用于运行临时数据（如状态实例）
- TObjectPtr智能指针：自动管理对象生命周期 当指向的对象被销毁时自动置为nullptr
- BeginPlay：组件初始化完成后调用，用于资源释放。
- 组件工作流程：
- 初始化阶段：1.构造函数中注册默认状态 2.BeginPlay中设置初始状态
- 状态切换阶段：1.调用SetLifetimeState触发状态转换 2.旧状态执行Exit逻辑，新状态执行Enter逻辑 3.广播状态变更事件
- 更新阶段：TickComponent中调用当前状态的Tick逻辑（如果需要）
- 这个组件实现了完整的角色生命周期状态管理系统，核心包括：
- 状态枚举定义与管理
- 状态转换框架与事件通知
- 状态实例的创建与缓存
- 蓝图集成与编辑器支持
- 理解此代码需要掌握虚幻引擎的反射系统、组件架构和状态设计模式，这些是开发复杂游戏系统的基础。通过此组件，角色可以在不同状态（如生成、活动、死亡）下表现出不同行为，同时保持代码的可维护性和可扩展性。
- Transient 标记属性为瞬态 不参与序列化（不会保存到存档或网络传输）。适用于运行时临时数据（如临时引用，中间计算结果），避免无效的数据存储。

### 生命周期管理
- 具体的代码更改情况
- 把生命周期的组件和技能的组件放到RPGCharacterLifetimeBase基类里 因为我们未来的NPC和玩家都要用到这个组件 首先定义生命周期状态的枚举 养成习惯 在前面打上RPG的这个标记 方便查找
- 我们应该如何实现生命周期组件 首先先声明三个参数的委托
- 简单的生命周期状态所需要的功能 组件的实现 最主要的是首先要开启一个Tick 我们需要知道的是在一个状态机里 我们需要哪些具体的类型 对应哪些的处理的类我们再把他做成配置化灵活的去使用 这样就可以把不同的需求组成不同实际的状态 注意有限的状态机的实现
- setLifetimeState这个函数里面 其实就是一个有限状态机的切换的一个过程Unreal的UObject结构都是可以直接new出来的 13分钟
> 注意：自己写的时候 遇到不理解的地方 告诉AI我不理解为什么要这样用 而不是那样用 我是需要补哪方面的基础知识吗 或许是我哪个地方有误解呢 并告诉AI我的引擎版本是多少（这很重要）
- state的实现 基础知识
- 定义的接口的话 就是看他是否有更新状态 owner就是直接通过Lifetimecomponent的owner去做的 本身是UActor的component挂到我们的Play上（注意看 这个地方很重要）记得深度挖掘 18.05 保障我们的状态机的工作流程是完备的
- gamemode的路径是写死的 要注意一下 31分钟 实现更多的State 35.12这个计算逻辑要好好的搞明白 已完成
- 暂时的慢是为了以后的快 把基础打牢 打扎实

### 生命周期管理配置化
- 前言
- 我们希望把生命周期管理的扩展变成是可以重复利用的配置基于命令模式的一种形式来进行配置 那么实际上我们只需要在扩展这里的时候 增加一个基于action配置的，把它的enter action和exit action也就是进出的action这么去实现 我们就可以通过再引入一个action的继承结构 这也就是命令模式它只有一个excute的接口，像之前的隐藏角色或者说是播放动画还有就是停止播放动画这些 做成不同的c++实现或者说是相应的蓝图实现 这样我们就可以在actionbased这个状态类里面去配置
- 命令模式：怎么做和什么时候做解耦 各自完成自己的事情
- 我们需要把进出的action做一个配置化 state的代码比较少 把之前的owner传进去就行了 ActorAction
- 给玩家打Tag是什么东西 可以深入一下Tag GamePlayTag.cpp 打Tag主要是为了不要让他在初始化的时候移动 实现了三个不同版本C++的action
- possess 占有 contains 拥有
- UObject继承下来 本身不具备Tick功能 需要手动把Tick转调出去

### 状态机实现
- 先判断是否改变 缓存旧状态 （因为接下来的旧状态要保留用于日志记录和状态退出逻辑） 先处理状态的退出 Exit 再根据新状态创建具体的状态对象
- 攀爬动画C++扩展实现 动画蓝图更新时大部分是走他的Tick 然后不停的去走他的Player 或者其他地方去拿她的这些状态数据 再复制一份到蓝图实例的本身自己的变量去 这样的效率是非常低下的

### 冲刺与短跑
- 增加冲刺动作按键释放检测
- 在按下时设定一个变量来标记，如blsDash
- 释放时 清除blsDash
- 在判定转换为Sprint的时候同时判定blsDash为true的情况

### 阶段回顾
- 对象生命周期 ：通过生命周期状态流程控制角色 可用FSM图形化实现 不同状态下控制相应的输入响应，动画
- 再是把有限状态机做成配置化

### 分层状态机
- 第一层解决不同类型游戏状态下的操控需求 URPGCharacterMovementPolicy 策略模式
- 第二层解决操控逻辑状态并控制物理状态 IRPGMovementState 接口控制
- 第三层就是单个状态下的细节处理
- 通过三层的拆解 代码变得特别简单
- 多研究设计模式 反复的思考 设计与分析。
- Unreal的游戏框架 Game GameMode GameState PlayerController input HUD PlayerCameraManger
- DS开发是带网络同步的开发
- Unreal操控重要的属性与方法需要重点去学习
- 动画蓝图通过继承UAnimalInstance
- 还有一些调试技巧 这个也是非常重要的需要去掌握
- 利用HUD调试复杂的状态变化
- 绘制线框等进行几何信息的可视化
- 在Tick中响应参数调整实现可视化的调试
- 调试代码开关功能。

### 滑行实现
- 参考原神分析进出滑行状态如何实现（同样是Jump按扭如何进出滑行状态）
- 滑行的动画混合空间应该在JumpLoop节点切换
- CharacterMovementComponent的GravityScale属性可以控制滑行落地速度。

### 指针选择
- 管理对象生命周期 -\> 用智能指针(TSharedPtr, TUniquePtr)
- UE对象系统内引用 -\> 用UObject指针 + UPROPERTY
- 纯C++对象/临时引用 - \>用原始指针

## 战斗系统 GAS

### 技能系统分析与设计

### 技能术语
- 技能 GameplayAbility GA
- 效果 GameplayEffect GE Buff 增益 HOT HealOverlayTime Debuff减益 DOT Damage OverTime
- 效果提示 GamePlayerCue GC ：特效 音效

### 技能系统循环
- Effect（Buff）是一个动态规则 可以通过修改属性来改变技能（规则）
- Unreal官方商城 ABLE ABILTY SYSTEM插件 做一些简单的技能系统可以用这个来实现

### 技能系统概览
- 依托于UAbilitySystemComponent
- UAttributSet ：属性集
- UGameplayEffect：效果，属性修改器（BUFF）
- UAbilitySystemComponent
- 技能系统Actor整合接口
- 负责网络数据同步策略
- GameplayTags：标记，状态
- GameplayCue：视觉表现系统
- GameplayEvents:事件系统

### 属性集
- UAttributeSet需要根据实际情况扩展再使用，一个游戏内可定义若干套属性集用于不同的技能计算。比如除了战斗相关的属性集，还可以定义任务系统需要的变量，天赋等

### Gameplay Tags
- Gameplay Tags用于代替枚举（enum）更灵活的标记各种状态
- 武器类型：空手 单手 双手 双持
- 道具类型：装备 材料 武器 道具 礼包
- 元素类型： 金木水火土
- 角色标记：可移动 可释放某类技能，免疫某些伤害
- 基于FName实现保障了一定的性能开销

### 技能任务
- 播放动画
- 创建Actor
- 重复处理
- 可视化技能目标
- 监听事件，Tag，属性变化，技能状态，输入处理
- 异步处理（等待操控后的结果）
- GE不带蒙太奇动画 技能都是放到Effect上面

### 动画通知
- 在动画上创建事件通知
- 播放音效，特效
- 通知技能Tag，Gameplay事件
- 窗口功能：控制功能开关，如碰撞检测开启，布料模拟开关，是否可以交互等
- 自定义的通知功能
- 可以通过不同的分段 创立不同的衔接

### 目标选择
- 新的方案 是插件 Targeting System 用来辅助瞄准 目标选择系统

### Gameplay Effect
- 主要功能有：作用周期管理 溢出处理 失效处理 免疫标签判定 技能消耗 堆叠形式：按源和目标 调用伤害计算公式（UE4）
- GE的激活：FActiveGameplayEffectsContainer：：ExecuteActiveEffectsFrom
- 计算流程 Modifier会先计算 然后再由计算公式计算
### Components 与 Modifiers
- Components用于定义GE整体的结构和行为
- 它们通常包括以下内容：
- Duration Policy（持续时间策略）：定义GE是立即生效、持续一段时间还是永久生效4。
- Period（周期）：用于设置GE的周期性触发，例如每秒触发一次效果4。
- Stacking（堆叠）：控制GE是否可以叠加，以及叠加时的行为规则4。
- 这些 Components 共同决定 GE 的基本行为和作用方式，为 Modifiers 的执行提供框架。
- Modifiers 是 Gameplay Effect 的具体执行部分，用于定义如何修改目标 Actor 的属性或状态。常见的 Modifiers 包括：
- Attribute Based Modifiers（基于属性的修改器）：根据目标 Actor 的属性值计算修改效果，例如根据攻击力计算伤害值。
- Set By Caller（由调用者设置）：允许在运行时动态设置修改值，例如技能释放时指定具体伤害数值4。
- Scalable Float（可缩放浮点数）：通过曲线表或其他动态数据源调整修改值4。
- Modifiers 是 GE 的核心逻辑，直接决定属性变化的具体数值和方式。
- 三个方面：
- 1.功能定位：
- Components定义GE的整体结构和行为，例如持续时间和周期性
- Modifiers 定义具体的属性修改逻辑，例如如何计算伤害或治疗值。
- 2.作用范围：
- Components影响GE的整个生命周期，例如设置GE的持续时间或者是否堆叠
- Modifiers 专注于属性修改，例如改变目标 Actor 的生命值或移动速度。
- 3.依赖关系：
- Components 为 Modifiers 的执行提供环境，例如通过 Duration Policy 决定 Modifiers 的作用时间。
- Modifiers 依赖 Components 的设置，例如周期性 Modifiers 需要 Period 的支持。
- 总结：Gameplay Effect 的 Components 和 Modifiers 分工明确。Components 负责定义 GE 的整体行为框架，Modifiers 负责具体的属性修改逻辑；两者结合后，GAS 才能实现灵活且复杂的属性变化和状态管理。

### 技能效果组件
- 将溢出，堆叠等功能抽到组件中，利于GE生命周期实现
- 在最新的版本中把生命周期的一些东西移到了GameplayeffectComponent这个类上去实现 生命周期在技能的每个阶段它去回调到我们的技能效果组件上 再由技能效果组件来实现这一功能
- 这样来看 性能更好 扩展也会更方便 这样就可以根据我们的需求去扩展我们的技能组件
- Gameplay的效果组件 说明
- （随机性效果）UChanceToApplyGameplayEffectComponent 应用Gameplay效果的概率 针对的是当前技能效果是否可以生效
- （状态控制）UBlockAblityTagGameplayEffectComponent 根据所有者的Gameplay效果目标Actor的Gameplay标签，进行Gameplay技能激活阻止处理 观察类型对象有没有相同的Tag 有的话就阻止技能激活
- UAssetTagsGameplayEffectComponent Gameplay效果资产拥有的标签，这些标签不会转移到Actor 模板
- UAdditionalEffectsGameplayEffectComponent 添加尝试在特定条件下激活（或任何条件下都不激活）的其他Gameplay效果 允许一个GE在应用时触发其他GE，形成链式效果 比如一个治疗GE同时附带一个短暂的护盾GE 注意：可以设定条件（如标签要求）来控制是否触发附加GE
- UTargetTagsGameplayEffectComponent 将标签授予Gameplay效果的目标（有时指所有者） 就是直接增加Tag 用于系统内部逻辑判断，比如“所有带 Poison 标签的 GE 都会被某个效果清除”。
- UTargetTagRequirementsGameplayEffrctComponent 指定如果此GE须应用或继续执行，目标（Gameplay效果的拥有者）必须具备的标签要求
- URemoveOthrGameplayEffectComponent 基于某些条件移除其他的Gameplay效果
- UCustomCanApplyGameplayEffectComponent 处理CustomApplicationRequirement函数的配置，以查看是否应该应用此Gameplay效果
- UImmunityGameplayEffectComponent 免疫会阻止其他GameplayEffectSpecs的应用 表示其他的效果不会使用
- 把这些功能都抽给技能效果组件来实现了 这是一个比较好的开始
- 我们的技能也好 技能效果也好 技能效果组件也好 我们都是围绕Tag标签来做判定
- UI更多还是用表格来配置 伤害和治疗就是计算公式的不同

### 伤害计算
- UGameplayEffectExecutionCalculation 主要是以属性复杂的计算过程为主，俗称计算公式
- 伤害计算 治疗计算
- 扩展类 UGameplayModMagnitudeCalculation 属性修改器自定义数值计算
- 伤害计算首先要定义一些属性 比如说防御能量 攻击能量 伤害值主要是这三个参数进行计算 固定写法 DECLARE_ATTRIBUTE_CAPTUREDEF(DefensePower);
- 然后再DEFINE一下，同时准备一个Statics函数 给到这个计算公式的构造函数里面去 把他的捕捉塞进去 这三个属性的捕捉关系 然后再开始实行这个计算
- 最终计算伤害的时候总归是通过防御值 伤害值 来计算 Damage Done = Damage \* AttackPower / DefensePower
- ARPG代码RPGDamageExtcution.cpp文件 目标和圆的技能组件 然后再把相应的目标和圆的 这个对象也找到 然后再把它相应的组合的Tag然后执行一个计算的参数 再调用计算参数 根据属性去计算 做一个保底if 如果防御变成0 那就给他赋值1.0f （是为了防止除以0的错误） attack也做一个属性的执行 最后计算汇总 如果伤害大于0 那么就给他增加一个Modifier的output进去
- GetAvatarActor_Direct()用于获取技能的发起者和承受者对应的Actor FGameplayEffectSpec是技能/效果的规格说明，包含发起者，承受者的标签（Tag），属性等信息
- 这两行代码的作用是获取技能系统组件（AbilitySystemComponent）对应的角色 Actor，逻辑解析如下：
- 三目运算符：第一行 AActor\* SourceActor = SourceAbilitySystemComponent ? SourceAbilitySystemComponent-\>GetAvatarActor_Direct() : nullptr;
- 判断 SourceAbilitySystemComponent 是否有效（非空）
- 如果有效，调用其 GetAvatarActor_Direct() 方法获取它所附着的角色 Actor（通常是技能的发起者）
- 如果无效，SourceActor 赋值为 nullptr
- 第二行 AActor\* TargetActor = TargetAbilitySystemComponent ? TargetAbilitySystemComponent-\>GetAvatarActor_Direct() : nullptr;
- 逻辑与第一行类似，用于获取目标技能系统组件所附着的角色 Actor（通常是技能的接收者）
- 这是 UE 中技能系统（GAS）的常见写法，通过技能系统组件反向获取对应的角色实体，以便在技能逻辑中操作角色（如施加伤害、播放动画等）。使用三目运算符简化了 "空指针判断 + 取值" 的逻辑。

### FGameplayTagContainer
- 1. 核心定位：为什么需要它？
- 单个 FGameplayTag 是 “用于标记游戏对象状态、行为或属性的字符串标识符”（如 Gameplay/Character/IsPlayer、Item/Weapon/Sword），而 FGameplayTagContainer 解决了 “多个标签的批量管理” 问题 —— 比如一个角色可能同时拥有 “玩家”“满血”“持有武器” 多个标签，用容器可统一维护这些标签，避免单独处理每个标签的冗余逻辑。
- 2. 核心功能（常用接口）
- 标签的添加 / 移除：通过 AddTag()/AddTags() 批量添加标签，RemoveTag()/ClearTags() 移除单个或所有标签，支持避免重复添加。 标签判断：核心接口 HasTag()（判断是否包含某个标签）、HasAnyTags()（判断是否包含多个标签中的任意一个）、HasAllTags()（判断是否包含多个标签的全部），是 gameplay 逻辑判断的常用方式（如 “角色是否同时有‘可攻击’和‘未眩晕’标签”）。 标签对比 / 合并：支持与其他 FGameplayTagContainer 对比（Matches()）、合并（AppendTags()），或获取标签差集（RemoveTags()），适合多对象标签交互（如 “技能效果标签与角色免疫标签对比，过滤无效效果”）。 序列化与编辑：支持蓝图可视化编辑（在细节面板中直接添加 / 编辑标签），也支持序列化（保存到配置文件或网络同步），方便设计师与程序协作。
- 3. 典型使用场景
- 角色状态管理：存储角色当前的所有状态标签（如 State/Alive、State/Stunned、Buff/Invincible），技能逻辑中通过容器判断是否满足释放条件。 物品 / 技能分类：武器对象的容器存储 “武器类型”“伤害属性” 标签（如 Weapon/Ranged、Damage/Fire），战斗系统通过标签匹配伤害计算规则。 事件触发条件：关卡蓝图中，判断 “玩家容器是否包含 Quest/Accepted 和 Location/BossRoom 标签”，满足则触发 BOSS 战。
- 4. 关键特性
- 基于层级的标签支持：若标签存在层级关系（如 Parent.Child），容器可通过 HasTagMatchingTagHierarchy() 判断是否包含某层级下的所有子标签（如判断是否有 Item.Weapon 下的任意武器标签）。 轻量高效：内部通过哈希表存储标签，查询、添加操作效率高，适合高频调用（如每帧的角色状态判断）。 跨模块兼容：在蓝图、C++、Gameplay Ability System（GAS）中均完全支持，是虚幻中跨系统传递 “状态 / 属性信息” 的通用载体。
- AttemptCalculateCaptruedAttributeMagnitude 是 GAS 的属性读取接口，用于从AbilitySystemComponent中获取 “防御力（DefensePower）”“攻击力（AttackPower）”“基础伤害（Damage）” 的当前值。
- 在虚幻引擎（UE）的代码语境中，DamageStatics() 通常是一个损伤相关的静态工具类或函数，用于封装与伤害计算、处理相关的通用逻辑。
- 它的典型功能可能包括：
- 计算伤害值（如根据攻击者属性、防御者抗性等调整最终伤害）
- 生成伤害事件（如触发OnTakeDamage回调）
- 处理伤害类型转换（如物理伤害、魔法伤害的差异化逻辑）
- 验证伤害有效性（如是否命中、是否暴击等）
- 这类 “Statics” 命名的工具类（如UGameplayStatics）通常提供静态方法，无需实例化即可调用，方便在项目各处复用伤害相关的核心逻辑，避免代码冗余。
- 计算最终伤害并输出
- 扩展：
- 要让上述代码生效，必须先注册属性（如攻击力，防御力） 步骤如下：

![RPG 实战笔记配图](/images/rpg-practice/gas-attributes.png)

### Lyra 示例
- Lyra是一个可以带服务器的一个示例工程 主要是一些物理材质 距离的衰减 具体的参数要和策划商量

### 技能消耗
- Lyra是自定义实现的类
- 这些功能都是很常见的 我们可以观察平时游戏的一些机制 功能 可以用GA与GE的组合来拆解他们
- “GE Ranged Base” 是 UE 中 GAS（Gameplay Ability System）框架下的 “远程技能基础 Gameplay Effect” 的缩写（GE = Gameplay Effect，Ranged = 远程，Base = 基础 / 父类）。它通常作为所有远程技能（如弓箭、火球术、枪械攻击等）的 “基础模板”，定义远程技能的通用逻辑和属性，避免重复配置。

### 技能基础框架
- 同步执行 是指程序按顺序执行代码，当前操作完成后才继续执行下一步。整个流程是阻塞的，执行过程是连续且线性的。
- 异步执行 是指程序可以启动某项操作后，不必等待该操作完成就继续执行后续代码，操作在后台或其他线程运行，完成后通过回调、事件或任务通知结果。
- 区别：
- 阻塞性：同步执行会阻塞流程等待结果，异步执行不会阻塞主流程。
- 执行时间：同步通常较快完成但可能引起卡顿，异步允许同时执行多个任务，提升效率和响应性。
- 复杂度：异步需要额外逻辑管理状态和结果回调，同步逻辑简单直观。
- 应用场景：同步适合快速、连续的小任务；异步适合耗时或需要并行处理的任务，如网络请求、文件I/O、复杂计算。
- 在UE5中，任务系统（Tasks System）提供异步执行能力，通过 启动异步任务，主线程可继续运行而不阻塞，等待任务完成可使用LaunchWait()

### 技能完整流程
- Gameplay Ability 基础的技能框架开始实现
- Gameplay Effect
- Gameplay Tags 武器装备
- Weapon Actor
- Anim Montage相应的动画模块机怎么去做
- Anim Notify 相应的伤害怎么去触发
- Equipment Items 最终要把这个武器作为一个装备 怎么去使用上
- Equipment Drop Equipment Action 做一些掉落的东西
- Socket同时牵扯到许多配置
- Input Action最后通过鼠标去触发他
- Gameplay Cue相应的动画特效表现
- DataTable 配置表

![RPG 实战笔记配图](/images/rpg-practice/gas-flow.jpeg)
- GameplayTag可以直接拿官方的 文档 实际上官方归类已经归类的很好了
- 打开Iris需要增加这句 不然会有链接错误：SetupIrisSupport（Target）；
- 添加新的东西 技能要增加相应的组件 对警告报错 保持零容忍的态度
- 我们最多用到的是直接一个Tag来激活
- 先把流程图捋明白 再把老师讲的记录下来 再去抠代码 不懂的立马扩展
- 流程图： 我们是通过数据表去配置相应的装备道具数据 以及他相应的一些特性 这样来去让这些武器 Weapon Actor这个东西可以生效 这边是关联引用了一些相应的Action 他怎么去装备一个武器到我的手上 这个时候会通过我们的背包 他会有一个拾取物掉在地上 之后拾取到背包 装备到手上后会通过这个技能 武器装备管理器去把技能授予给我们的技能系统组件 WeaponActor会附加到我手上的这个插槽 一个技能对象去Play一个蒙太奇 蒙太奇上有相应的动画通知 动画通知就去把我们的武器的这个actor的碰撞打开 同时他在整个动画播放的过程中 他就会碰到我们的一些actor 那就是我们的敌人 也可能是我们自己 我们需要去做一些判断 然后再发送一个gameplay事件 把这个游戏事件发出去 技能蒙太奇在播放后 会等待事件的触发 GE去触发Cue然后再通知VFX和SoundFX特效和音效
- 技能系统组件可以看看基本的函数 不要过深的深入 再继续看基础的技能类 实际开发中也是把很多的功能实现起来再需要的时候去看具体的函数
- 句柄 怎么去索引 所以创建一个Handles 是一个非常有用的技术 用来管理
- URPGAbilitySet 技能集合
- RPGAbilitySourceInterface 圆的一个技能接口 做不同的技能打不同的效果 衰减效果
- 本地化的字符变量尽量用FText
- 属性集直接拿官方的代码 黑盒用 宏定义 委托通知
- 属性集的建立
- 大流程需要在脑袋里非常的清晰 我们需要实现怎么样的一个流程 增加了武器的通知 数据方面 近战的基础一个GA 一个GE
- 使用平砍动画 标签的话增加了近战的这个伤害事件

### 工程实现记录
- RPGWeaponActor.h东西并不多 就定义了一个装备类型和一个伤害类型 使用了GameplayTag 实现的话很基础 没什么东西
- 5:30 RPGWeaponActorMelee.h
- 伤害的技能效果直接用属性去修改 这节只需要一个动画 就是一个平砍 标签的话DefaultGameplayTags.ini 激活技能需要使用到input Tag
- 装备部分增加了 RPGEquipmentActorInterface.h里面有接口可以获取动画装备类型，用于识别。
- if（OtherActor == GetOwner（））\{return；\}
- BP_OnOverlapBegin（OverLappedComp，OtherActor，OtherComp，OtherBodyIndex，bFromSweep，SweepResult）；做一个蓝图的事件调用，更方便的处理负责的逻辑 用模板函数的方式吧函数抛出去给蓝图去扩展，
- 模板函数扩展：
- 用模板函数的方式吧函数抛出去给蓝图调用，模板函数可以写通用代码，减少重复。把他抛给蓝图调用，可以直接使用这个函数。在C++里，不用在为每种类型店都写一个单独的函数。
- 普通函数只能针对固定的数据类型，要处理不同类型就得写好多遍，代码量大。像计算两个数相加，整数相加写一个，浮点又得写一个，很麻烦。
- 模板函数就不一样了，它可以处理各种类型，代码更简洁通用。刚才我们写的交换变量的模板函数，不管是整数，浮点数和自定义类型，只要是类型支持相应操作就可以直接用，大大减少了重复代码

### 待办与练习
- 7.24 用简单的方法修正光环将拾取物碰撞的问题
- 增加回血和反击的Gameplay Cue的表现 这个去做一下配置就好了
- 7.25
- 美化技能槽UI,增加边框和输入反馈
- 实现一个需要检测持续按键处理的技能并完成动作插槽的功能 可以参考lary
- 将IRPGSlotAction修改为蓝图可扩展，参考URPGInventoryItemAction的做法。
- 7.26
- 将Action Slot的配置保存下来，重新启动游戏时可加载并恢复。 可以看一下Unreal的存档功能 把相关的属性存成一个structrue
- 完善ActionSlotHUD的快捷键显示TObjectPtr\<URPGActionWight\>InputActionIcon
- 7.27
- 用Niagara的粒子效果表现投掷预览 可以用粒子效果控制一下
- 增加反弹爆炸音效（参考初级篇子弹相关内容）
- 重构ARPGProjectileBase使得ARPGProjectileThrowable可复用相应的功能代码。
- 7.28
- 实现一个光线攻击引导技能，根据鼠标指向进行攻击。
- 如果要实现物品掉落或者捡起背包 需要数据表实现
- 实现一个基础战斗技能GA_MeleeBase 父类RPGGameplayAbilityformEquipment 通过装备获得的技能
- 属性的话有一个动画蒙太奇 有一个handle 还有一个要给技能施加的一个GE 默认值选择的是Base
- 默认是使用技能的时候把他激活提交一次，不提交的话可能，没办法正确执行
- PlayMontageAndWait 需要把蒙太奇传进来 Start Section蒙太奇多个段的一个数据参数 从第几段开始播放
- AsynsTask任务节点

### Handle（句柄）
- 本质：Handle 是 “资源的代理”，你拿到 Handle 不代表拿到资源本身，而是拿到了 “操作资源的权限和入口”。
- 核心价值：
- 隔离资源细节：无需知道资源在内存中的具体地址，避免直接操作指针导致的野指针、内存泄漏问题；
- 统一管理资源：引擎 / 系统可通过 Handle 跟踪资源的生命周期（如资源被销毁时，Handle 会自动失效）；
- 安全校验：使用前可先判断 Handle 是否 “有效”（如 IsValid()），避免访问已销毁的资源。

### AbilitySystemComponent
- 核心定位与作用 ASC本质上是一个组件（Component），需挂载到Actor上才能生效，其核心职责可概括为三点：
- 1.能力的容器与管理者：
- 负责授予，激活，取消，移除实体的能力，并跟踪所有已授予能力的状态
- 2.属性与效果的载体：
- 管理实体的“GameplayAttribute”（游戏性属性）（如生命值，攻击力，移动速度），以及“GameplayEffect（游戏性效果）”（如持续伤害护盾，属性加成）--例如，ASC会计算“火焰灼烧”效果对“生命值”的实时扣减，或力量药水对攻击力的临时加强。

## UI 系统

### UI 元素层级
- 页面（Page） ： 具有导航性质的UI组成部分，常用于描述多页设置菜单中的单独一页
- 对话框（Dialogue）：弹出窗口，用于显示信息，确认信息或者收集玩家输入
- 面板（Panel）：展示相关信息和选项的区域，如物品栏，角色信息面板等
- 小部件（Widget）：构成用户界面的基本元素，如按钮，滑动条，文本框等，用于执行操作或者输入信息
- 抬头显示（HUD）：游戏中一直显示的信息，提供重要的即时数据，如生命值，地图等
- 覆盖层（Overlay）：在主游戏画面上的额外UI层，用于显示信息或提供附加交互
- 工具提示（Tooltip）：鼠标悬停时显示的小文本框，提供关于UI元素的额外信息
- 模态窗口（Modal）：需要玩家交互的特殊对话框或面板，玩家需要完成或关闭后才能与游戏其他部分交互

### UI 设计原则
- 最小惊讶原则
- 简洁性：尽管提供必要的信息很重要，但过度复杂的UI设计会分散玩家的注意力，甚至造成困扰。良好的UI设计应该力求简洁，避免不必要的装饰和元素。
- 清晰性：UI的首要任务是传达信息，因此它必须清晰易懂。所有的元素应该一目了然，玩家可以轻松识别各种图标，指示器和文字。
- 所见即所得
- 一致性和标准化：用户不应该不断重新学习新的操作方式和界面布局。保持UI元素在不同界面和环境中的一致性，可以帮助玩家建立对游戏操作的直觉理解。颜色，字体，图标样式的一致性，能够让玩家更快的适应游戏环境。
- 反馈及时性：用户的任何操作都应该获得及时反馈，无论是点击按钮，提交表单，还是任何的交互动作。及时反馈可以让用户知道系统已经接受到了他们的操作，增加了交互的透明度和信任感
- 尺寸基本上是8的倍数
- UI 增加了UI相关的代码 利用ActorAction来显示我们的菜单 把get world重载一下

### 工程变化
- Source/RPG/UI RPGUITags.h
- RPGUITags.cpp
- RPGUIPlayerController

### GameInstance
- 核心特点： GameInstance在游戏启动时创建，直到游戏完全关闭（进程退出）时才会销毁，不会随关卡切换，场景加载而重载。这使其成为存储关卡全局的理想选择
- 全局唯一性：一个游戏进程中只会存在一个GameInstance实例，所有的关卡，系统都可以通过他访问全局共享的数据或者功能
- 与其他类的区别：1.和GameMode不同：GameMode负责单局游戏规则（如胜利条件，玩家重生），会在关卡切换时重新初始化
- 和PlayerController不同：PlayerController绑定特定玩家，可能随玩家退出而销毁
- GameInstance则专注于“贯穿整个游戏生命周期”的全局逻辑
- 常见用途：
- 存储跨关卡的玩家数据（如进度，装备，设置）
- 管理游戏的保存/加载系统
- 处理网络连接（如多人游戏中的服务器/客户端初始化）
- 维护全局管理器（如音频管理器，UI管理器的引用）
- 监听游戏的生命周期事件（如启动，退出，暂停继续）

### GameUIManagerSubsystem
- 是UE中的模块化的功能组织方式，具有明确的生命周期（与所属的宿主绑定，如World Engine GameInstance等）
- 核心功能：
- UI资源管理：负责加载 缓存常用UI蓝图（如主菜单，HUD，弹窗，面板等），避免重复加载导致的性能损耗，同时统一管理UI资源的释放
- UI实例控制：提供接口用于创建指定UI （如ShowMainMenu（），OpenPopup()）,隐藏/销毁UI（（如HideHUD（），ClosePanel（）），并维护当前活跃UI的列表
- UI层级与优先级：管理UI的显示层级（如弹窗始终在主界面之上），输入焦点（如弹窗打开时屏蔽底层UI交互）避免多个UI同时显示时的冲突
- 状态同步：作为UI与游戏逻辑的中间层，接收游戏状态变化（如角色受伤，任务完成）并通知对应UI更新（如血条刷新，显示任务提示），解耦UI与业务逻辑。
- 跨UI通信：协调不同的UI之间的交互（如背包面板与装备面板的数据同步），通过Subsystem作为中介传递消息，避免UI之间直接耦合

- GameUIManagerSubsystem 是 UI 系统的总管。通过模块化设计，可以让 UI 逻辑更清晰，也更方便维护和扩展。

### 初始化流程
- UGameUIManagerSubsystem::Initialize
- UComnmonGameInstance::Init
- CommonGameInstance::AddLocalPlayer_3
- 重叠显示的内容使用 Overlay 管理下层组件。

## AI 系统

### UE AI 框架总览
- AIController 服务器端“大脑”，控制Pawn并调用行为树等系统。
- Pawn 受控实体，负责物理移动与动画表现
- Blackboard+Behavior Tree数据驱动决策模型，Blackboard存状态，Behavior Tree读写并切换分支
- 组件式设计 导航，感知，EQS等

### 导航与寻路
- PathfindingAPI MoveTo Task 自动调用A\*在NavMesh上自动寻路并躲避障碍。

### 调试与可视化

- 按 `P` 可以显示 NavMesh；Gameplay Debugger 的 NavMesh 页面可用于查看细分多边形和寻路成本。

### AI Perception 感知
- Sight/Hearing/Damage/Touch/Team/Prediction
- 统一事件回调OnTargetPerceptionUpdated
- StimuliSource
- Actor带UAIPerceptionStimuliSourceComponent即可发出刺激
- 可视化

## Unreal 蓝图库

### PlayMontageAndWait
- 动画蒙太奇（Animation Montage）是 UE 中用于组合、分段、混合动画片段的工具（比如将 “攻击起手→攻击动作→收招” 三段动画组合成一个连贯的攻击流程）。
- PlayMontageAndWait 的作用是：
- 触发指定的动画蒙太奇播放；
- 蓝图节点会阻塞后续逻辑，直到蒙太奇完全播放完毕后，才继续执行后续节点。

![RPG 实战笔记配图](/images/rpg-practice/play-montage-and-wait.png)

### 扩展知识与注意事项
- 1. 与PlayMontage的区别
- PlayMontage：异步播放，调用后立即执行后续蓝图逻辑，不等待蒙太奇结束。适合 “动画播放与逻辑并行” 的场景（如角色移动时播放走路动画）。 PlayMontageAndWait：同步等待，必须等动画结束才继续执行。适合 “动画时序严格绑定逻辑” 的场景（如攻击、技能）。
- PlayMontageAndWait：同步等待，必须等动画结束才继续执行。适合 “动画时序严格绑定逻辑” 的场景（如攻击、技能）。
- 2. 动画分段（Section）的管理
- 动画蒙太奇可通过 “分段（Section）” 拆分不同子动画（如同一攻击蒙太奇包含 “轻击”“重击”“蓄力攻击” 三段）。在Starting Section中指定分段名称，可实现 “一次蒙太奇资源，多种动画流程” 的复用。
- 3. 中断与异常处理
- 若需在蒙太奇播放中强制中断（如角色被击中需播放受击动画），可结合StopMontage节点
- 4.动画通知（Notify）的配合
- 在动画蒙太奇的关键帧添加动画通知（Notify），可在特定时间点触发逻辑（如攻击动画的“击中帧”添加DamageNotify，播放到该帧时立即检查伤害）。结合PlayMontageAndWait，能实现“动画帧级别的逻辑触发”。
- 5.总结

PlayMontageAndWait 适合动画时序与逻辑需要严格绑定的场景。实际使用时，需要根据是否等待动画结束进行选择，并配合 Section、Notify 和中断逻辑完成角色动作流程。

### EndAbility
- 在 Unreal Engine 的Gameplay Ability System（GAS）中，EndAbility 是控制技能生命周期的核心函数之一，用于主动或被动结束一个技能的执行流程。
- 一、核心功能：“终结技能的执行周期”
- 技能（UGameplayAbility）的生命周期通常为 “激活（Activate）→ 执行（Commit / 持续逻辑）→ 结束（End）”。EndAbility 的作用是：
- 清理技能激活时创建的资源（如生成的特效 Actor、临时属性修改）；
- 触发技能结束时的逻辑（如播放收招动画、恢复角色状态）；
- 向系统反馈技能结束的原因（如正常结束、被打断、主动取消）。

### WaitGameplayEvent
- 在 Unreal Engine 的Gameplay Ability System（GAS） 中，WaitGameplayEvent 是一个异步等待指定 Gameplay Event（游戏事件）触发的核心工具（通常以蓝图节点或 C++ 任务形式存在），用于实现 “事件驱动” 的技能逻辑。
- 一、核心功能：“异步监听并响应特定事件”
- Gameplay Event 是 GAS 中用于跨系统传递信息的轻量机制（通过FGameplayTag标记事件类型，如 “Ability.Hit” 表示命中事件、“Player.Jump” 表示跳跃事件）。
- WaitGameplayEvent 的作用是：
- 异步监听指定标签的 Gameplay Event（如 “Damage.Taken” 受击事件）；
- 当事件被触发时，立即执行后续逻辑（如播放受击动画、触发反击技能）；
- 整个过程不阻塞当前蓝图 / 代码流程（异步特性），适合 “等待某个条件触发后再行动” 的场景。

![RPG 实战笔记配图](/images/rpg-practice/wait-gameplay-event.png)

### Handle（句柄）
- UE 中大量使用 Handle 管理核心资源，尤其是 GAS 框架，你之前接触的技能、特效相关逻辑都离不开它：
- 1. FGameplayAbilitySpecHandle（技能句柄）
- 用途：标识一个 “技能实例”（FGameplayAbilitySpec），每个角色的技能槽位都对应一个该 Handle。 场景：激活技能、取消技能时，需要通过这个 Handle 告诉 GAS “要操作哪个技能”。
