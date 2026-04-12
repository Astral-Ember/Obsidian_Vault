[Gameplay Attributes and Attribute Sets for the Gameplay Ability System in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community](https://dev.epicgames.com/documentation/unreal-engine/gameplay-attributes-and-attribute-sets-for-the-gameplay-ability-system-in-unreal-engine)

AttributeSet需要挂载到Actor或PlayerState上


对于属性，可以添加为`FGameplayAttributeData`
```
UCLASS()  
class UMAttributeSet : public UAttributeSet  
{  
    GENERATED_BODY()  
public:  
          
private:  
    UPROPERTY()  
    FGameplayAttributeData Health;  
};
```
声明了一个名为Health 的属性集

对于Gameplay Attribute来说，有四个重要的宏


| 宏（带参数）｜生成函数的签名｜行为/用途｜

|                                                              |                                         |                                         |
| ------------------------------------------------------------ | --------------------------------------- | --------------------------------------- |
| `GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UMyAttributeSet, Health)` | `static FGameplayAttribute GetHealth()` | 静态函数从虚幻引擎的反射系统中返回`FGameplayAttribute`结构 |
| `GAMEPLAYATTRIBUTE_VALUE_GETTER(Health)`                     | `float GetHealth() const`               | 返回"生命值"游戏玩法属性的当前值                       |
| `GAMEPLAYATTRIBUTE_VALUE_SETTER(Health)`                     | `void SetHealth(float NewVal)`          | 将"生命值"游戏玩法属性的值设置为`NewVal`               |
| `GAMEPLAYATTRIBUTE_VALUE_INITTER(Health)`                    | `void InitHealth(float NewVal)`         | 将"生命值"游戏玩法属性的值初始化为`NewVal`              |


## 1. 核心概念

`GameplayAttribute`（游戏玩法属性）是 GAS 中用于表示角色数值（如生命值、护甲、弹药量、移动速度等）的核心数据结构。

- **数据结构**：底层由 `FGameplayAttributeData` 结构体表示。
    
- **双值机制**：
    
    - **BaseValue（基础值）**：角色的永久属性。只有永久性修改（如升级加点、永久消耗）才会改变它。
        
    - **CurrentValue（当前值）**：在 BaseValue 的基础上，叠加了所有当前激活的、有持续时间的 `GameplayEffect`（Buff/Debuff）后的最终结算值。
        

---

## 2. 声明与定义 (C++ 规范)

属性必须在继承自 `UAttributeSet` 的类中声明，并配合官方宏来自动生成 Getter 和 Setter。

### 2.1 必备宏定义

在 AttributeSet 头文件顶部声明访问器宏：


```
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

### 2.2 属性声明

对于需要网络同步的属性（如 FPS 游戏中的血量或弹药），需要标准模板：


```
UPROPERTY(BlueprintReadOnly, Category = "Attributes", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UMyAttributeSet, Health)

UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

---

## 3. 读取与修改

在业务逻辑中，绝不应该直接操作 `FGameplayAttributeData` 的内部成员。

- **读取**：
    
    - 获取当前值：`GetHealth()`
        
    - 获取基础值：`Health.GetBaseValue()`
        
- **初始化（仅限出生/重置时）**：
    
    - `InitHealth(100.f)`
        
- **强行修改（不推荐作为常规手段）**：
    
    - `SetHealth(NewValue)` （注意：这会改变 BaseValue）
        
- **标准修改流**：
    
    - 通过构建并应用 **GameplayEffect (GE)** 来修改属性。这是触发 GAS 内部事件、预测和网络同步的基础。
        

---

## 4. 核心生命周期与回调函数

`UAttributeSet` 提供了几个关键的虚函数，用于在属性变化的前后插入自定义逻辑（如数值钳制、死亡判定）。

### 4.1 `PreAttributeChange`

- **触发时机**：在 `CurrentValue` 即将发生变化之前。
    
- **作用**：用于**钳制（Clamp）当前值**。例如，确保移动速度受 Debuff 影响时不会变成负数。
    
- **注意**：不要在这里写 Gameplay 逻辑（如死亡判定），因为它经常被预测系统和模拟触发。
    


```
virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;
```

### 4.2 `PreAttributeBaseChange`

- **触发时机**：在 `BaseValue` 即将发生变化之前。
    
- **作用**：类似于 `PreAttributeChange`，但专门针对 BaseValue 的钳制。
    

### 4.3 `PostGameplayEffectExecute` (最常用的逻辑驻地)

- **触发时机**：在一个 `Instant`（瞬时）或 `Periodic`（周期性）的 `GameplayEffect` 执行完毕，且 `BaseValue` 已经被修改之后。
    
- **作用**：处理**游戏核心逻辑**。例如：
    
    - 将受到的“伤害值”（Meta Attribute）从“生命值”中扣除。
        
    - 血量归零时触发死亡事件。
        
    - 钳制修改后的最终 BaseValue（确保血量不超过最大血量）。
        


```
virtual void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data) override;
```

---

## 5. 网络同步 (Replication)

对于多人游戏，`AttributeSet` 的同步必须严谨。

### 5.1 注册同步属性

在 `.cpp` 中重写 `GetLifetimeReplicatedProps`，并使用条件同步：


```
void UMyAttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    // REPNOTIFY_Always 确保即使服务端数值覆盖为原值，客户端也强制刷新
	DOREPLIFETIME_CONDITION_NOTIFY(UMyAttributeSet, Health, COND_None, REPNOTIFY_Always);
}
```

### 5.2 实现 OnRep 函数

在回调函数中使用 GAS 宏通知 ASC 同步已发生，这对系统内部的状态机和客户端预测至关重要：


```
void UMyAttributeSet::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UMyAttributeSet, Health, OldHealth);
}
```

---

## 6. Meta Attributes (元属性)

- **定义**：一种特殊的 `GameplayAttribute`，它通常不代表实体的持久状态，而是作为计算的**临时载体**。
    
- **经典用例**：**伤害 (Damage)**。
    
    - 在 GE 中，不直接扣减 Health，而是增加 Damage。
        
    - 在 `PostGameplayEffectExecute` 中拦截 Damage 属性。
        
    - 根据护甲、抗性等公式结算实际伤害。
        
    - 最终从 Health 的 BaseValue 中扣除结算后的伤害值。
        
    - 清空 Damage 属性（将其归零），等待下一次伤害计算。