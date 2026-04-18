在第三人称模板的输入系统中，输入所调用的函数是这样的
![[Pasted image 20251126192823.png]]![[Pasted image 20251126195849.png]]
YawRotation是绕Z轴旋转，即偏航
计算前进方向矢量和右转方向矢量并保存
GetUnitAxis的作用是：获取单位轴
AddMovementInput是一个让玩家移动的函数，在第一个参数中提供移动方向，第二个参数介于-1到1之间

同时在输入映射上下文中
![[Pasted image 20251126195959.png]]
在输入操作中
![[Pasted image 20251126200032.png]]

A和D设置X的值，W和S设置Y的值
在UE_LOG中表现出来
![[Pasted image 20251126213246.png]]



==至于向量归一化的问题：Unreal Engine的CharacterMovementComponent会在内部自动处理向量归一化（还是太轮椅了==

还有另一种方法：
为了使对角线移动与单方向移动速度一致，可以在DoMove中添加归一化处理
```
void AShooterCharacter::DoMove(float Right, float Forward)
{
    if (GetController() != nullptr)
    {
        // 归一化输入向量，防止对角线速度过快
        FVector InputVector(Right, Forward, 0.0f);
        if (InputVector.SizeSquared() > 1.0f)
        {
            InputVector = InputVector.GetSafeNormal();
        }

        // find out which way is forward
        const FRotator Rotation = GetController()->GetControlRotation();
        const FRotator YawRotation(0, Rotation.Yaw, 0);

        // get forward vector
        const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);

        // get right vector 
        const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

        // add movement 
        AddMovementInput(ForwardDirection, InputVector.Y);
        AddMovementInput(RightDirection, InputVector.X);
    }
}

```