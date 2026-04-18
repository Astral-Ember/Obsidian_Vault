委托是在游戏中发生某些事情时触发的事件  
例如：当一个角色与另一个角色重叠时，我们可以将函数连接到这些委托，这样当这些委托被触发时，这些函数就会被调用

![Exported image](Exported%20image%2020251125140808-0.png)  
![Exported image](Exported%20image%2020251125140812-1.png)   
委托的原理是将我们定义的函数于引擎中发生的事件相绑定，但是定义的函数要与事件所要求的参数相同

[仅使用C++的示例 | 虚幻引擎 5.6 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/cpp-only-example)  
[Delegates and Lambda Functions in Unreal Engine | 虚幻引擎 5.6 文档 | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/delegates-and-lambda-functions-in-unreal-engine)

![Exported image](Exported%20image%2020251125140815-2.png)

/** 当某对象进入球体组件时调用 */  
UFUNCTION()  
void OnOverlapBegin(class UPrimitiveComponent* OverlappedComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
 
/** 当某对象离开球体组件时调用 */  
UFUNCTION()  
void OnOverlapEnd(class UPrimitiveComponent* OverlappedComp, class AActor* OtherActor, class UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);