[Gameplay Effects for the Gameplay Ability System in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community](https://dev.epicgames.com/documentation/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine)



## 基础知识点
Gameplay Effect通常不创建C++类，它用来决定GameplayAttribute怎么变
通常要在Gameplay效果中选择我们在GameplayAttribute中定义的属性进行赋值

### 如何在ASC中应用GE？
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
在ASC类中声明一个`TSubclassOf<UGameplayEffect>`的数组，用来承载GameplayEffect，这是数据驱动的思想；
创建一个函数，遍历数组，并对数组中的每一个元素（GameplayEffect）`ApplyGameplayEffectSpecToSelf(*SpecHandle.Data.Get());`




## 1. 核心分类：持续时间策略 (Duration Policy)

GE 的存在形态由它的持续时间决定，这直接影响它是修改 `BaseValue` 还是 `CurrentValue`。

- **Instant（瞬时）**
    
    - **机制**：立即应用并销毁，**永久改变 BaseValue**。
        
    - **典型场景**：子弹命中造成的真实扣血、拾取急救包恢复生命值。
        
- **Has Duration（有持续时间）**
    
    - **机制**：作为 `ActiveGameplayEffect` 驻留在目标的 ASC 中，计时结束后自动移除。**只改变 CurrentValue**。
        
    - **典型场景**：被闪光弹致盲 3 秒、持续 5 秒的冲刺移速加成。
        
- **Infinite（无限期）**
    
    - **机制**：永久驻留，直到被外部逻辑显式移除（调用 `RemoveActiveGameplayEffect`）。**只改变 CurrentValue**。
        
    - **典型场景**：装备防弹衣提升护甲、给枪械安装扩容弹匣配件（卸下配件时移除 GE）。
        

---

## 2. 核心功能：修改器 (Modifiers)

Modifiers 决定了 GE 如何改变属性。一个 GE 可以包含多个 Modifier（例如一个药剂同时加血和加移速）。

### A. 修改操作 (Modifier Op)

- **Add**：加法（填负数就是减法，最常用）。
    
- **Multiply**：乘法（常用于百分比 Buff，如移速增加 20% 填 1.2）。
    
- **Divide**：除法。
    
- **Override**：覆盖（强制将属性设为某个特定值，慎用，容易破坏叠加规则）。
    

### B. 数值来源 (Magnitude Calculation Type)

在 FPS 项目中，伤害数值往往不是固定不变的，GE 提供了多种动态取值方式：

- **Scalable Float**：基于曲线（Curve Table）读取。配合 GE 的 `Level` 参数，可以实现“1级技能造成10点伤害，2级造成20点”。
    
- **Attribute Based**：基于自身或目标的某个属性。例如“造成施法者最大生命值 10% 的伤害”。
    
- **Custom Calculation Class (MMC)**：**极度重要**。指向一个用 C++ 或蓝图编写的 `UGameplayModMagnitudeCalculation` 类，允许你写极其复杂的计算逻辑来返回一个 Float 作为最终的 Modifier 增量。
    

---

## 3. 标签交互 (Gameplay Tags)

GE 是运载和处理 Tag 的完美容器，通过 Tag 可以实现复杂的逻辑互斥和状态赋予。

- **Granted Tags (赋予标签)**：当这个 GE 处于激活状态时（Duration/Infinite），目标会获得这些 Tag。比如应用一个“冲刺”GE，同时赋予目标 `State.Movement.Sprinting` 标签。
    
- **Application Tag Requirements (应用前置条件)**：目标必须拥有（或必须没有）某些 Tag，这个 GE 才能被成功应用。
    
- **Immunity (免疫)**：如果目标拥有这里配置的 Tag，就会免疫（拒绝接收）这个 GE。比如目标有 `State.Invincible`（无敌）标签，伤害 GE 就会被直接拦截。
    

---

## 4. 进阶核心：执行计算 (Execution Calculation / ExecCalc)

对于复杂的战斗逻辑，简单的 Modifiers 往往不够用。比如一个射击游戏的伤害公式需要考虑：**子弹基础伤害 * 距离衰减 * 部位倍率（爆头） - 目标护甲**。

这时候必须使用 **ExecCalc (`UGameplayEffectExecutionCalculation`)**：

- **完全由 C++ 编写**：性能极高，适合高频触发的射击伤害结算。
    
- **多属性捕获**：可以在一个类中同时抓取施法者（Source）的攻击力、破甲值，以及目标（Target）的护甲、抗性等多个属性。
    
- **Meta Attribute 处理**：在这里计算出最终伤害后，通常不直接扣血，而是将被修改的值赋给一个叫 `Damage` 的元属性，然后去 `AttributeSet::PostGameplayEffectExecute` 里统一结算扣血逻辑。
    

---

## 5. 数据载体与生命周期

在 C++ 底层，我们需要区分三个概念：

1. **`UGameplayEffect` (CDO/类数据)**：它只是一个配置模板（只读的数据资产）。**不要**在运行时尝试修改它。
    
2. **`FGameplayEffectSpec` (实例数据)**：通过 `ASC->MakeOutgoingSpec` 生成。它是 GE 的动态副本，包含了当前的 Level、是谁施放的、动态传入的数值（SetByCaller）等。
    
3. **`FGameplayEffectContext` (上下文)**：包含在此次 GE 应用中的物理环境信息，比如 `HitResult`（打中了哪个部位的碰撞体）、谁是 Instigator（枪手）、谁是 EffectCauser（子弹本身）。