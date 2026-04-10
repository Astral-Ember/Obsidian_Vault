动画蓝图的基类，可以从中获取到一些值供状态机使用
有三个重要函数：
```
virtual void NativeUpdateAnimation(float DeltaSeconds) override;

virtual void NativeInitializeAnimation() override;

virtual void NativeThreadSafeUpdateAnimation(float DeltaSeconds) override;
```

## 1.NativeInitializeAnimation()
事件蓝图初始化动画
**用途：初始化** 
这个函数相当于蓝图的 `BeginPlay`。它在动画实例被创建时调用一次

**主要功能**：获取并缓存指向 Pawn 或 Character 的引用，避免在每帧更新时重复进行昂贵的类型转换（Cast）

## 2.NativeUpdateAnimation(float DeltaSeconds)
事件蓝图更新动画
**用途：每帧逻辑更新（主线程）**
