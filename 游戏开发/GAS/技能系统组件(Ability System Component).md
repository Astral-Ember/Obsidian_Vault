[虚幻引擎GAS组件和Gameplay属性 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-ability-system-component-and-gameplay-attributes-in-unreal-engine)

可以创建一个Player State类让我们控制的角色来使用（**将AbilitySystemComponet定义在PlayerState上运行**）：把能力系统组件添加到PlayerState中，然后实现`AbilitySystemInterface`并重写`GetAbilitySystemComponent`，最后让它返回AbilitySystemComponent

一些步骤：
- 在插件中要启用Gameplay Ability
- 在Build.cs文件中添加`"GameplayAbilities", "GameplayTags","GameplayTasks"` 
- 声明必要头文件`#include "AbilitySystemInterface.h"`,`#include "AbilitySystemComponent.h"`
- 继承`AbilitySystemInterface`
- 声明`TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;`
- 重写`virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;`并`return AbilitySystemComponent;`（因为在`AbilitySystemInterface`中`GetAbilitySystemComponent()`为纯虚函数）

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

对于服务端声明`virtual void PossessedBy(AController* NewController) override;`并定义：
```
void AGAS_PlayerCharacter::PossessedBy(AController* NewController)  
{  
    Super::PossessedBy(NewController);  
    if (!GetAbilitySystemComponent()) return;  
    GetAbilitySystemComponent()->InitAbilityActorInfo(GetPlayerState(), this);  
}
```

对于客户端声明`virtual void OnRep_PlayerState() override;`并定义：
```
void AGAS_PlayerCharacter::OnRep_PlayerState()  
{  
    Super::OnRep_PlayerState();  
    if (!GetAbilitySystemComponent()) return;  
    GetAbilitySystemComponent()->InitAbilityActorInfo(GetPlayerState(), this);  
}
```

### `InitAbilityActorInfo` 具体干了什么？

它在告诉 `AbilitySystemComponent` (ASC) 两件非常关键的事：

1. **Owner 是谁？**（数据归谁管）：在这里是 `GetPlayerState()`。ASC 会去这里找属性（血量、蓝量）和技能列表。
    
2. **Avatar 是谁？**（技能表现给谁）：在这里是 `this`（即 Character）。ASC 以后播动画（Montage）、播放特效（GameplayCue）都会在这个 Character 上执行。


### 在 GAS 项目中，代码实现顺序通常是这样的：

1. **第一步（定义）：** 在 `PlayerState` 中创建 `AbilitySystemComponent` 对象。
    
2. **第二步（寻找）：** 在 `Character` 中实现 `GetAbilitySystemComponent()`，让 Character 能通过 PS 访问到 ASC。
    
3. **第三步（握手）：** 在 `PossessedBy` (服务器) 和 `OnRep_PlayerState` (客户端) 调用 `InitAbilityActorInfo`