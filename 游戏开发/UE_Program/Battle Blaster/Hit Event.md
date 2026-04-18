[使用OnHit事件 | 虚幻引擎 5.7 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-the-onhit-event)  
在projectile中实现  
创建函数和委托

![Exported image](Exported%20image%2020251125141022-0.png)  

这个委托需要传入五个不同的参数

|   |
|---|
|DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_FiveParams( FComponentHitSignature, UPrimitiveComponent, OnComponentHit, UPrimitiveComponent*, HitComponent, AActor*, OtherActor, UPrimitiveComponent*, OtherComp, FVector, NormalImpulse, const FHitResult&, Hit );|

![Exported image](Exported%20image%2020251125141024-1.png)

因此所调用的函数也要对应五个