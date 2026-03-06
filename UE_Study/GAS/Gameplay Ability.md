[使用虚幻引擎中的Gameplay Abilities | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-gameplay-abilities-in-unreal-engine)

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