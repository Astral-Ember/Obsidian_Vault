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
	- Gameplay Effect可以不通过C++创建，需要在其细节面板的Gameplay Effect中的修饰符中选择在Attribute声明的属性，通过游戏效果配置属性
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

有必要条件：服务器权威，所以要在[[技能系统组件(Ability System Component)#^562c57|服务器端]]应用GE
创建函数，在服务器端和客户端都要调用 ==`InitAbilityActorInfo`==

在这个流程中存在一些需要==注意==的点：
在服务器端
