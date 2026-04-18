C++创建输入映射上下文

![Exported image](Exported%20image%2020251125141149-0.png)

“项目名”.Build.cs是虚幻构建所需要的文件要在这一行添加EnhancedInput 增强输入系统  
然后在Tank类中编写代码，而不是BasePawn，因为只有Tank是由玩家操控的，需要输入系统  
声明这样的头文件，才能使用想要的类

![Exported image](Exported%20image%2020251125141150-1.png)  
![Exported image](Exported%20image%2020251125141154-2.png)

可以自由定义指针名  
在BeginPlay()中实现￼这就是一个模板  
[在虚幻引擎中使用C++配置角色移动 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/configure-character-movement-with-cplusplus-in-unreal-engine)

![Exported image](Exported%20image%2020251125141157-3.png)

红框部分可以自定义为刚刚在.h中定义的输入映射上下文指针的指针名

![Exported image](Exported%20image%2020251125141159-4.png)

角色蓝图中选择输入映射上下文即可