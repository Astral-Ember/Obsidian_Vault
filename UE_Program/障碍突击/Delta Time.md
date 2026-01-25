DeltaTime变量可以使游戏帧率独立于其他帧率  
由于不同电脑运行游戏的速度不一样，如果单纯的在tick中直接加减乘除，  
就会出现一些问题：如果我的帧率较高，则tick执行会变多（快？）  
而DeltaTime这一特殊变量：这是CPU计算游戏最后一帧所需时间即，执行一次Loop循环所需时间  
所以在tick中进行类似操作时，我们可以 将添加的值与deltatime相乘，以使我们代码与帧率无关，这意为着我们的代码将以相同的帧率运行

![Exported image](Exported%20image%2020251125140447-0.png)  

这样写更好一点

![Exported image](Exported%20image%2020251125140449-1.png)  
![Exported image](Exported%20image%2020251125140453-2.png)