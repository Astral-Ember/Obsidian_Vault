连接至在线子系统

在角色C++类中进行配置：
.h
```
	/**
	 * @brief 在线会话接口的弱指针引用
	 *
	 * 该变量用于持有对在线会话接口的弱引用，避免循环引用导致的内存泄漏。
	 * 通过弱指针可以在需要时安全地访问在线会话功能，同时不影响对象的生命周期管理。
	 */
	TWeakPtr<IOnlineSession> OnlineSessionInterface;
```
.cpp的构造函数中
```
// 获取在线子系统并初始化会话接口
// 该代码块主要完成以下功能：
// 1. 获取当前世界的在线子系统实例
// 2. 从在线子系统中获取会话接口
// 3. 在屏幕上显示正在使用的在线子系统名称（用于调试）
	if (IOnlineSubsystem* OnlineSub = Online::GetSubsystem(GetWorld()))
	{
		// 获取会话接口，用于后续的在线会话操作
		OnlineSessionInterface = OnlineSub->GetSessionInterface();

		// 调试信息输出：显示当前使用的在线子系统名称
		if (GEngine)
		{
			GEngine->AddOnScreenDebugMessage(-1, 15.f, FColor::White,
				FString::Printf(
					TEXT("Using Online Subsystem: %s"),
					*OnlineSub->GetSubsystemName().ToString()));
		}
	}
```
如果配置成功，就会这样
![[Pasted image 20251205203907.png]]