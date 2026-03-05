[虚幻引擎GAS组件和Gameplay属性 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-ability-system-component-and-gameplay-attributes-in-unreal-engine)

可以创建一个Player State类让我们控制的角色来使用（**将AbilitySystemComponet定义在PlayerState上运行**）：把能力系统组件添加到PlayerState中，然后实现`AbilitySystemInterface`并重写`GetAbilitySystemComponent`，最后让它返回AbilitySystemComponent

一些步骤：
- 在插件中要启用Gameplay Ability
- 在Build.cs文件中添加`"GameplayAbilities", "GameplayTags","GameplayTasks"` 
- 声明必要头文件`#include "AbilitySystemInterface.h"`,`#include "AbilitySystemComponent.h"`
- 声明`TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;`并定义
- 重写`virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;`并`return AbilitySystemComponent;`

链接中官方实现方法是在actor类中重写GetAbilityComponent()但是如果在Player State类中实现，就需要在角色类中在此重写GetAbilityComponent()以返回AbilitySystemComponent
具体方法是：
在角色类的头文件中声明`virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;`
定义过程为：声明一个对应的Player State对象并通过Cast转换GetPlayerState()，然后返回Player State的GetAbilitySystemComponent()
```
UAbilitySystemComponent* AGAS_PlayerCharacter::GetAbilitySystemComponent() const  
{  
    AGAS_PlayerState* PlayerState = Cast<AGAS_PlayerState>(GetPlayerState());  
    if (!PlayerState) return nullptr;  
    return PlayerState->GetAbilitySystemComponent();  
}
```
