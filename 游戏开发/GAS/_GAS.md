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

关于初始化有两种方式：
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
