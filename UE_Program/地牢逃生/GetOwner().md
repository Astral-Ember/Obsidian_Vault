![Exported image](Exported%20image%2020251125140930-0.png)

它将获取拥有该组件的Actor并返回指向该角色的指针

|   |   |
|---|---|
|AActor 是 Unreal Engine 中的一个核心类，以下是关于它的要点：  <br>• 基类性质：AActor 是 Unreal Engine 中所有可放置到游戏世界中的对象的基类  <br>• 功能作用：  <br>• 代表游戏世界中的一个实体对象  <br>• 可以被放置在关卡中  <br>• 支持场景渲染、碰撞检测、网络复制等功能  <br>• 可以包含和管理各种 UActorComponent 组件  <br>• 继承关系：许多重要的游戏对象类都继承自 AActor，如：  <br>• APawn（游戏角色）  <br>• ACharacter（角色）  <br>• AController（控制器）  <br>• ACameraActor（摄像机）  <br>• 主要特性：  <br>• 拥有位置、旋转、缩放等变换属性  <br>• 支持生命周期管理（生成、销毁）  <br>• 可以响应游戏世界的各种事件  <br>简单来说，AActor 就是构成游戏世界的基本对象类型。|GetOwner() 返回的是 AActor 类型的指针，而 GetActorLocation() 是 AActor 类的基本成员函数  <br>• 多态性：无论 GetOwner() 返回的具体是什么类型的 Actor（如 APawn、ACharacter 等），它们都继承自 AActor 基类，因此都可以调用 AActor 中定义的公共方法  <br>• 虚函数机制：GetActorLocation() 等方法在 AActor 中被声明为虚函数，允许派生类重写这些方法的具体实现，但保持了统一的接口  <br>• 组件与Actor的关系：UActorComponent（组件）通过 GetOwner() 方法获取其所属的 AActor 对象，这样组件就可以访问和操作其拥有者 Actor 的各种属性和方法|

举个例子

![Exported image](Exported%20image%2020251125140933-1.png)  
![Exported image](Exported%20image%2020251125140940-2.png)