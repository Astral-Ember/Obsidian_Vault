代码过冲  
防止平台长时间 移动 后偏离原来轨道

![Exported image](Exported%20image%2020251125140356-0.png)

由于是DistanceMoved\>=MoveDistance  
所以难免会出现一些偏差  
而原来的代码新的初始位置被设置成 当前位置 在长时间运行后可能导致位置偏差过大
 
而修正代码过冲问题  
解决方法：  
主要是当DistanceMoved\>=MoveDistance后，把角色位置每次都会重置在“真正初始位置+移动方向*移动距离”的位置，同时重置StartLocation以 计算新的DistaceMoved