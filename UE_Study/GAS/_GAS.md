[虚幻引擎的Gameplay技能系统 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-ability-system-for-unreal-engine)




## 关于GAS的初始化配置

1. 创建[[技能系统组件(Ability System Component)]] 
	   需要在Actor类或PlayerState等Actor可获取到的类中挂载ASC，才能和AbilitySystem交互
2. 配置[[Gameplay Attribute]]
	- AttributeSet也要挂载在Actor类或PlayerState上
		- 在创建的GameplayAttribute类（继承自GameplayAttribut）中需要通过`FGameplayAttributeData`声明游戏所需的属性（血量、体力等）
		- 在 AttributeSet 头文件顶部声明访问器宏
3. 创建[[Gameplay Effect]]
	- Gameplay Effect可以不通过C++创建，需要在其细节面板的Gameplay Effect中的修饰符中选择在Attribute声明的属性，通过游戏效果配置属性