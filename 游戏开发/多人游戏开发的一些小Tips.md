服务器本质上是在远程机器上运行的游戏的一个特殊版本
玩家在本地计算机上运行的叫客户端，客户端与服务器连接
对于这种服务器和客户端模型来说，有一个非常重要的规则：==服务器权威== 
意味着服务器是游戏状态的实际持有者，客户端显示的是服务端正在发生的事情

P2P模型：服务器向客户端分发（复制）游戏角色，但是不会复制Player Controller（因为玩家控制器是玩家的“灵魂”），在多人游戏中，==对PlayerController进行判断十分重要==
```
bool AMCharacter::IsLocallyControlledByPlayer() const  
{  
    return GetController() != nullptr && GetController()->IsLocalPlayerController();  
}
```


## UI添加相关
多人游戏添加UI，最好在PlayerController中添加
==注意==：添加时要特别检查`IsLocalController()`，否则服务器也会尝试在没有显卡的服务器进程里创建UI，导致报错

