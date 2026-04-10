动画蓝图的基类，可以从中获取到一些值供状态机使用
有三个重要函数：
```
virtual void NativeUpdateAnimation(float DeltaSeconds) override;

virtual void NativeInitializeAnimation() override;

virtual void NativeThreadSafeUpdateAnimation(float DeltaSeconds) override;
```

## 1.NativeInitializeAnimation()
**用途：初始化** 
这个函数相当于蓝图的 `BeginPlay`。在动画蓝图中就是“事件蓝图更新动画”，它在动画实例被创建时调用一次
**主要功能**：获取并缓存指向 Pawn 或 Character 的引用，避免在每帧更新时重复进行昂贵的类型转换（Cast）

## 2.NativeUpdateAnimation(float DeltaSeconds)
**用途：每帧逻辑更新（主线程）**
这个函数相当于蓝图的`Tick` 在动画蓝图中是“事件蓝图更新动画”
**主要功能**：计算每帧需要的动画参数（如速度、方向、是否在空中等）。
**运行环境**：它运行在**游戏主线程**（Game Thread）上。这意味着它可以安全地访问大多数 Actor 属性。

## 3.NativeThreadSafeUpdateAnimation(float DeltaSeconds)
**用途：线程安全的逻辑更新（工作线程）**
**主要功能**：与 `NativeUpdateAnimation` 类似，但它运行在**动画工作线程**（Worker Thread）上

**为什么要用它？**：
**性能优化**：它不会阻塞主线程，利用多核 CPU 提高并行效率。
**数据隔离**：在这个函数里，你**不能**直接访问 `Character` 或其他 Actor 的属性，因为它们可能正在被主线程修改。
**典型用法**：配合 **Property Access**（属性访问）机制，或者处理纯数学计算。如果你在做复杂的手动混合逻辑，这里是最高效的地方。

|**函数名**|**调用频率**|**运行线程**|**核心目的**|
|---|---|---|---|
|**NativeInitialize**|仅一次|主线程|缓存引用（获取 Owner）|
|**NativeUpdate**|每帧|主线程|通用逻辑、与外部 Actor 交互|
|**NativeThreadSafeUpdate**|每帧|动画线程|高性能并行更新、纯数据计算|
关于实现：
在 `NativeInitializeAnimation` 中存好指针。
    
在 `NativeUpdateAnimation` 中读取角色数据（如 `Velocity`, `IsFalling`）。
    
如果动画逻辑非常复杂（涉及大量数学计算），尽量尝试将计算移至 `NativeThreadSafeUpdateAnimation` 以优化性能。