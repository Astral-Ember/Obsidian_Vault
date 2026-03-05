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

