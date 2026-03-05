## **技能系统组件** (`UAbilitySystemComponent`)
这是一个Actorcomponent，处理GAS中的所有交互
## Gameplay属性和属性集
**游戏技能系统（Gameplay Ability System）** 使用 **游戏属性（Gameplay Attributes）**（`FGameplayAttribute`）来保存、计算和修改与游戏相关的浮点值。这些值可以描述其拥有者的任何特征，比如角色的剩余生命值，车辆的最高速度，或者物品在损坏前可以使用的次数。游戏技能系统中的角色将他们的游戏属性存储在一个 **属性集（Attribute Set）** 中，该属性集有助于管理游戏属性与系统其他部分之间的交互，并将自己注册到角色的技能系统组件中。这些交互包括对数值范围的限定、临时数值变更、对永久改变基础值的事件做出反应等。

拥有属性集的角色会zi