[虚幻引擎GAS组件和Gameplay属性 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-ability-system-component-and-gameplay-attributes-in-unreal-engine)

可以创建一个Player State类让我们控制的角色来使用，然后把能力系统组件添加到Player State中，然后实现`AbilitySystemInterface`并重写`GetAbilitySystemComponent`，然后让它返回Ability System Component

一些步骤：
- 在插件中要启用Gameplay Ability
- 在Build.cs文件中添加`"GameplayAbilities", "GameplayTags","GameplayTasks"` 
- 声明必要头文件`#include "AbilitySystemInterface.h"`,`#include "AbilitySystemComponent.h"`
- 声明`TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;`并定义
- 重写`GetAbilitySystemComponent()`并`return AbilitySystemComponent;`
