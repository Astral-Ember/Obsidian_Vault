[使用虚幻引擎中的Gameplay Abilities | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-gameplay-abilities-in-unreal-engine)



## Gameplay Ability的一般流程：
1. 创建Gameplay标签（可新建文件后使用命名空间创建）
2. 在玩家控制器中创建回调函数（总之就是控制输入的地方）
3. 创建GameplayAbility蓝图并添加到GameplayAbility数组中



添加GameplayAbility蓝图（有事件激活能力和事件OnEndAbility节点）

在角色类中

通过TArray存储Abilities：
```
UPROPERTY(EditDefaultsOnly, Category="Abilities")  
TArray<TSubclassOf<UGameplayAbility>> StartupAbilities;
```

定义一个函数，用于赋予技能
```
void AGAS_BaseCharacter::GiveStartupAbility()  
{  
    //遍历数组  
    for (const auto& Ability : StartupAbilities)  
    {
	    FGameplayAbilitySpec AbilitySpec(Ability);  
	    GetAbilitySystemComponent()->GiveAbility(AbilitySpec);  
    }
}
```

由于只有服务器才能赋予或撤销技能因此要在`PossessedBy`中执行该函数

在角色蓝图的Ability数组中添加GamePlayAbility蓝图


## 通过Gameplay Tag激活Ability

在输入的回调函数处实现
主要是要获取AbilitySystemComponent，由于输入操作的实现是在PlayerController类中实现，通过静态函数库`UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(GetPawn());`可以获取到角色的AbilitySystemComponent，不需要使用任何转换