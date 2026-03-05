
## 核心
在 GAS (Gameplay Ability System) 中，理解 **Owner (所有者)** 和 **Avatar (化身)** 的关系是掌握该系统的核心。简单来说，这是一个**大脑与肉体**的分离设计。

### 1. 核心定义：谁是谁？

在你的代码场景中（`PlayerState` 模式），它们的角色分配如下：

- **Owner Actor (所有者):** 逻辑上的拥有者。
    
    - 通常是 **`PlayerState`**。
        
    - **特点：** 在整个关卡中持续存在，玩家死亡重生时它不会销毁。它负责保存数据（如等级、属性、已获得的技能）。
        
- **Avatar Actor (化身):** 物理上的表现者。
    
    - 通常是 **`Character`** 或 **`Pawn`**。
        
    - **特点：** 玩家在场景中的三维实体。它负责执行动作（播动画、放特效、处理碰撞）。
        

---

### 2. 它们之间的关系

它们通过 `InitAbilityActorInfo(Owner, Avatar)` 函数“绑定”在一起。这种设计带来了三大优势：

#### A. 死亡重生的平滑处理

如果你的 `ASC` 挂在 `Character` 上，一旦玩家掉进岩浆死亡，`Character` 销毁，你的技能和属性也就丢了。

- **分离后：** `Character` 死了销毁掉，但 `PlayerState`（Owner）还在。当新的 `Character` 生成后，再次调用 `InitAbilityActorInfo`，把新的 `Character` 设为 `Avatar`。
    
- **结果：** 玩家复活后，技能和血量上限依然保持不变。
    

#### B. “灵魂附体” (Possession)

想象一下你的玩家可以控制一辆坦克，或者控制一只小鸟：

- **Owner** 始终是你的 `PlayerState`。
    
- **Avatar** 可以从 `HumanCharacter` 切换到 `TankPawn`。
    
- **结果：** 你的玩家数据（Owner）没变，但施展技能的表现形式（Avatar）变了。
    

#### C. 网络同步的层级

- **Owner** 负责处理网络权限和状态同步的核心逻辑。
    
- **Avatar** 负责处理客户端预测（Client Prediction）的表现。
    

---

### 3. 代码中的体现

当你调用 `InitAbilityActorInfo` 时，你实际上是在告诉 GAS：

一旦绑定完成：

- 调用 `GetOwnershipActor()` 会得到 `PlayerState`。
    
- 调用 `GetAvatarActor()` 会得到 `Character`。




## **技能系统组件** (`UAbilitySystemComponent`)

**技能系统组件** (`UAbilitySystemComponent`) 是Actor和 **游戏玩法技能组件（Gameplay Ability System）** 之间的桥梁。Actor需要拥有自己的技能系统组件，或访问PlayerState或Pawn上的技能系统组件，才能与游戏玩法技能系统进行互动，这是一个Actorcomponent，处理GAS中的所有交互


## Gameplay属性和属性集
**游戏技能系统（Gameplay Ability System）** 使用 **游戏属性（Gameplay Attributes）**（`FGameplayAttribute`）来保存、计算和修改与游戏相关的浮点值。这些值可以描述其拥有者的任何特征，比如角色的剩余生命值，车辆的最高速度，或者物品在损坏前可以使用的次数。游戏技能系统中的角色将他们的游戏属性存储在一个 **属性集（Attribute Set）** 中，该属性集有助于管理游戏属性与系统其他部分之间的交互，并将自己注册到角色的技能系统组件中。这些交互包括对数值范围的限定、临时数值变更、对永久改变基础值的事件做出反应等。

拥有属性集的角色会自动将属性集注册到角色的能力系统组件中


## Gameplay Ability
**Gameplay Ability** 源自 `UGameplayAbility` 类，定义了游戏中技能的效果、使用技能付出的代价（如有），以及何时或在何情况下可以使用等。玩法技能可以作为异步运行的实例化对象存在，因此你可以运行专门的多阶段任务，包括角色动画、粒子和声效，乃至根据执行时的用户输入或角色交互设计分支。玩法技能可以在整个网络中自我复制，运行在客户端或服务器计算机上（包括客户端预测支持），甚至还能同步变量和执行远程过程调用（RPC）。玩法技能还使引擎可在游戏会话期间灵活实现游戏技能，例如提供的扩展功能可用于实现冷却和使用消耗、玩家输入、使用动画蒙太奇的动画，以及对给予Actor的技能本身做出反应。

## Gameplay效果
Gameplay技能系统会利用 **Gameplay效果** 更改Gameplay技能所针对Actor的属性。Gameplay效果包含你可以应用到Actor属性的函数库。这些效果可以是即时效果，比如施加伤害，也可以是持续效果，比如毒杀，在一定的时间内对角色造成伤害。

- 你可以使用Gameplay效果进行增益和减益。
    
    - 根据你的游戏设计，让你的角色变得更强或更弱。
- Gameplay效果属于资产，因此在运行时不可变。
    
    - 也有例外情况，例如，当在运行时创建Gameplay效果，但未在创建和配置时修改数据的情况。
- **Gameplay效果规格** 是Gameplay效果的运行时版本。
    
    - 它们是Gameplay效果的实例数据封装器（这是一种资产）。通常，在运行时使用Gameplay效果时（如创建蓝图图标），你需要处理Gameplay效果规格，而非Gameplay效果。例如，技能系统蓝图库（Ability System Blueprint Library）广泛使用了Gameplay效果规格。
    
    蓝图功能本身关注的是Gameplay效果规格，而不是Gameplay效果，这反映在技能系统蓝图库中。

## 技能任务(Ability Task)
技能任务（C++类"UAbilityTask"）是更常规的技能任务类的特殊形式，旨在使用游戏性技能。使用游戏性技能系统的游戏通常包括各种自定义技能任务，这些任务实施其独特的游戏功能。它们在游戏性技能执行过程中执行异步工作，并且能够通过调用[委托（Delegate）](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/delegates-and-lambda-functions-in-unreal-engine) （在本地C++代码中）或移经一个或多个输出执行引脚（在蓝图中）来影响执行流。这使技能能够跨多个帧执行，并可在同一帧内执行多个不同的函数。大部分技能任务都有一个即时触发的执行引脚，使蓝图能够在任务开始后继续执行。此外，通常还有一些特定于任务的引脚，它们会在延迟后或在可能发生或不发生的某个事件之后触发。

技能任务可以通过调用"EndTask"函数自行终止，或者等待运行它的游戏性技能结束，此时它会自动终止。这可以防止幻影技能任务运行，有效地泄漏CPU周期和内存。例如，某个技能任务可能播放一个施法动画，而另一个任务则在玩家的瞄准点处放置一个靶向标线。如果玩家点击确认输入来施放该法术，或者等待动画结束而未确认该法术，游戏性技能就会结束。虽然它们可以在任何时候自动终止，但是技能任务保证最晚在主要技能结束时终止。
