代码中如果有实现相同功能的代码 就可以将这些代码重构为一个新的函数
 ![Exported image](Exported%20image%2020251125140745-0.png)  
![Exported image](Exported%20image%2020251125140748-1.png) ![Exported image](Exported%20image%2020251125140750-2.png)

Trigger是我重构的函数，用来重写原代码中重复的部分，并添加了一个功能：如果存在相同标签的Actor，即使有Actor退出，Trigger的状态不会被改变