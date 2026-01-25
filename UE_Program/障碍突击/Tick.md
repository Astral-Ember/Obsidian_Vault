![Exported image](Exported%20image%2020251125140503-0.png)

Game Loop  
所有的游戏和游戏引擎都使用所谓的游戏循环来运行  
这是一个持续运行并更新游戏的循环：

1. Get input from player 从玩家的鼠标或键盘获取输入
2. Update game state 在输入得到处理后更新游戏状态
3. Render graphics to the screen 在屏幕上显示图形渲染
 
Tick  
1s内的帧数（frame）有一个专门的术语，叫做FPS（frames per second）
 
Tick是一个特殊函数，在游戏的每一帧都会被调用
   

移动的实现

![Exported image](Exported%20image%2020251125140504-1.png)