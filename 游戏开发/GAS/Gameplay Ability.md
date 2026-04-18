[使用虚幻引擎中的Gameplay Abilities | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-gameplay-abilities-in-unreal-engine)
源自`UGameplayAbility`重要函数均在该基类中定义



可以在C++中定义一个BaseAbilityClass（继承自UGameplayAbility）

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


## 总的来说
对于GA，==只有服务器才能赋予技能==，技能必须实例化，并且必须能够在服务器上运行，因此应用技能相关函数（自定义）必须要在服务器同步相关函数内调用，例如：`PossessedBy(AController* NewController)`或`OnPossess(APawn* InPawn)`
应用GA可以在角色类中，也可以在ASC中，在哪里不重要，重要的是以下步骤：

### 授予技能
#### 1、获取GA类
通常是`TSubclassOf<UGameplayAbility>`可以通过数组，或者是DataAsset获取，总是，初始化GA，这是刚需

#### 2、GiveAbility
这是ASC中的一个函数，如果在角色类中定义需要通过哦GetAbilitySystemComponent，在这个类中虽然可以直接以GA类作为参数，但是还是最好显示定义`FGameplayAbilitySpec`再GiveAbility



### 发动/调用技能
在GameplayAbility类中（一般是继承自UGameplayAbility）覆写`ActivateAbility()`函数，该函数会在调用ASC的