1. 注册输入映射上下文
2. 把每个输入操作连接到C++函数
 
C++文件在这里

![Exported image](Exported%20image%2020251125140654-0.png)

PlayerController玩家控制器中

![Exported image](Exported%20image%2020251125140657-1.png)  
![Exported image](Exported%20image%2020251125140659-2.png)  

在这个文件中

![Exported image](Exported%20image%2020251125140701-3.png)

是用来连接输入操作的

|   |   |
|---|---|
|![Exported image](Exported%20image%2020251125140703-4.png)<br><br>对应|![Exported image](Exported%20image%2020251125140708-5.png)|
 ![Exported image](Exported%20image%2020251125140710-6.png)

声明函数 用于等下定义交互时被触发的功能

![Exported image](Exported%20image%2020251125140712-7.png)

这样就意为着，当输入操作进行交互时，输入操作就会被触发，Interact()函数就会被调用