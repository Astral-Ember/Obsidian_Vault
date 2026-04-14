[虚幻引擎的Gameplay技能系统 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-ability-system-for-unreal-engine)

[[GAS的核心]]
[[技能系统组件(Ability System Component)]]
[[Gameplay Attribute]]
[[Gameplay Effect]]
[[Gameplay Ability]]
[[Gameplay Tags]]
[[Gameplay Tasks]]
[[Gameplay Events]]





## 关于GAS的初始化配置

1. 创建[[技能系统组件(Ability System Component)]] 
	   需要在Actor类或PlayerState等Actor可获取到的类中挂载ASC，才能和AbilitySystem交互
2. 配置[[Gameplay Attribute]]
	- AttributeSet也要挂载在Actor类或PlayerState上
		- 在创建的GameplayAttribute类（继承自GameplayAttribut）中需要通过`FGameplayAttributeData`声明游戏所需的属性（血量、体力等）
		- 在 AttributeSet 头文件顶部声明访问器宏
3. 创建[[Gameplay Effect]]
	- Gameplay Effect可以不通过C++创建，需要在其细节面板的Gameplay Effect中的修饰符中选择在Attribute声明的属性，通过GE配置属性
4. 在ASC中应用GE
	在整个GAS体系中`AbilitySystemComponent` (ASC) 是 `GameplayEffect` (GE) 的容器、执行者和分发中心
	在引用GE前，需要判断是否有权限，即使用`HasAuthority()`，判断是否在服务器上运行
	第一步：创建 Effect Context
		`FGameplayEffectContextHandle` 
		包含了谁施放了效果、从哪里施放、打中了哪个物理点等上下文信息。
		`FGameplayEffectContextHandle ContextHandle = MakeEffectContext();`
	第二步：创建Effect Spec
		`FGameplayEffectSpecHandle` 是 GE 的具体“规格说明书”，它结合了模板类（GE Class）和上下文数据。
		`FGameplayEffectSpecHandle SpecHandle = MakeOutgoingSpec(GEClass, Level, ContextHandle);`
	第三步：应用到目标
		`ApplyGameplayEffectSpecToSelf(*SpecHandle.Data.Get());`
		
	因此采取以下流程：
	- 在ASC类中声明一个`TSubclassOf<UGameplayEffect>`的数组，用来承载GameplayEffect，这是数据驱动的思想；
	- 创建一个函数，遍历数组，并对数组中的每一个元素（GameplayEffect）`ApplyGameplayEffectSpecToSelf(*SpecHandle.Data.Get());`


## 在服务器和客户端的初始化
必要条件：服务器权威，所以要在[[技能系统组件(Ability System Component)#^562c57|服务器端]]应用GE
创建函数，在服务器端和客户端都要调用 ==`InitAbilityActorInfo`==

在这个流程中存在一些需要==注意==的点：

==关于初始化有两种方式==：
一、在Actor（玩家类）中直接实现Possessby（服务端）和OnRep_PlayerState（客户端）函数在其中调用ASC类下的`InitAbilityActorInfo`函数，传入两个参数，第一个是持有ASC的对象（**Owner Actor** 通常是 `PlayerState` 或 `Character`），第二个是世界中代表该实体的物理对象（**Avatar Actor** 通常是 `Character`）

二、在PlayerController中实现（因为PlayerController是玩家的“灵魂”不会随玩家“死亡”而销毁）
这个方法适用于更严格的网格同步环境，在PlayerController中调用OnPossess（服务端）和AcknowledgePossession（客户端）

## 同步服务器与客户端的变量
这里包括GAS变量和普通C++变量的实现方法：

| **特性**    | **Gameplay Attribute (GAS)**               | **普通 C++ 变量 (Native)**                     |
| --------- | ------------------------------------------ | ------------------------------------------ |
| **底层类型**  | `FGameplayAttributeData` (结构体)             | 基本类型 (`float`, `int32`, `bool`, `FVector`) |
| **变量声明**  | `UPROPERTY(ReplicatedUsing = OnRep_HP)`    | `UPROPERTY(Replicated)`                    |
| **同步内容**  | **双值同步**：同时包含 `BaseValue` 和 `CurrentValue` | **单值同步**：仅同步当前变量存储的值                       |
| **必须注册**  | 是，在 `GetLifetimeReplicatedProps` 中         | 是，在 `GetLifetimeReplicatedProps` 中         |
| **推荐宏**   | `DOREPLIFETIME_CONDITION_NOTIFY`           | `DOREPLIFETIME`                            |
| **回调机制**  | 必须配合 `GAMEPLAYATTRIBUTE_REPNOTIFY` 宏       | 手动编写常规业务代码                                 |
| **修改权限**  | **服务器权威**（通过 `GameplayEffect` 修改）          | **服务器权威**（直接赋值或通过 RPC 修改）                  |
| **预测与回滚** | **原生支持**：客户端预测后可由服务器自动纠偏                   | **不支持**：需要开发者手动实现预测逻辑                      |
| **适用场景**  | 生命、魔法、攻击力、防御力等战斗数值                         | 等级、金钱、阵营、玩家名称、准备状态                         |
### 详细实现流程

#### A. GAS 变量同步 (在 `AttributeSet` 中)

GAS 属性同步的重点在于通知 `AbilitySystemComponent` (ASC) 属性已更新，以便处理潜在的预测回滚。

##### 第一步：头文件 (.h)

```
// 定义属性
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;

// 声明 OnRep 函数
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

##### 第二步：源文件 (.cpp)

```
#include "Net/UnrealNetwork.h"

// 1. 注册同步
void UMyAttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    // REPNOTIFY_Always 确保即使服务器发来的值与本地一致，也触发 OnRep，这在 GAS 中对齐状态至关重要
    DOREPLIFETIME_CONDITION_NOTIFY(UMyAttributeSet, Health, COND_None, REPNOTIFY_Always);
}

// 2. 实现回调
void UMyAttributeSet::OnRep_Health(const FGameplayAttributeData& OldHealth) {
    // 关键宏：同步 BaseValue 和 CurrentValue，并允许 ASC 处理预测冲突
    GAMEPLAYATTRIBUTE_REPNOTIFY(UMyAttributeSet, Health, OldHealth);
}
```

---

#### B. 普通 C++ 变量同步 (在 `PlayerState` 或 `Actor` 中)

普通变量同步更简单、直接，性能消耗相对更低，适合非战斗数值。

##### 第一步：头文件 (.h)

```
// 只需要声明 Replicated 即可，如果需要回调则用 ReplicatedUsing
UPROPERTY(EditAnywhere, BlueprintReadOnly, ReplicatedUsing = OnRep_PlayerGold)
int32 PlayerGold;

UFUNCTION()
void OnRep_PlayerGold();
```

##### 第二步：源文件 (.cpp)

```
#include "Net/UnrealNetwork.h"

// 1. 注册同步
void AMyPlayerState::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    // 最简单的同步注册
    DOREPLIFETIME(AMyPlayerState, PlayerGold);
    
    // 或者只同步给拥有该 PC 的客户端（优化带宽）
    // DOREPLIFETIME_CONDITION(AMyPlayerState, PlayerGold, COND_OwnerOnly);
}

// 2. 实现回调
void AMyPlayerState::OnRep_PlayerGold() {
    // 处理本地逻辑，例如刷新商店界面 UI
    if (MyGoldWidget) {
        MyGoldWidget->UpdateGold(PlayerGold);
    }
}
```

---

###  开发建议与避坑

1. **不要在客户端改值**： 无论哪种变量，在客户端直接 `Health = 50` 或 `PlayerGold = 999` 仅会在本地修改，**不会同步给服务器或其他玩家**。所有的修改逻辑必须在服务器执行（`HasAuthority() == true`）。
    
2. **GAS 必须用 GE 修改**： 对于 GAS 属性，即便是服务器，也推荐通过应用 `GameplayEffect` 来修改值，而不是直接调用 `SetHealth()`。这样能确保触发 `PostGameplayEffectExecute` 等生命周期函数。
    
3. **UI 绑定的最佳位置**：
    
    - **GAS 属性**：通常在 `Character` 的 `BeginPlay` 中通过 `GetAbilitySystemComponent()->GetGameplayAttributeValueChangeDelegate` 绑定监听。
        
    - **普通变量**：直接在 `OnRep` 回调中调用 UI 更新函数。
        
4. **带宽优化**： 如果一个变量只有玩家自己需要知道（如任务进度、详细背包数据），务必使用 `COND_OwnerOnly` 条件同步，避免浪费其他玩家的下行带宽。