在虚幻中可以自定义timeline曲线来获得不同的效果

###  Step1:在头文件中声明Timeline所需内容
```
	//声明一个时间轴对象
	FTimeline MyTimeline;

	//声明一个曲线对象
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Timeline")
    UCurveFloat* MyCurve;

	//声明时间轴float曲线更新委托
    FOnTimelineFloat OnMyTimelineTickCallBack;
	//声明时间轴播放完毕委托
	FOnTimelineEvent OnMyTimelineFinishedCallBack;

	//声明相关回调函数，如每帧执行、结束时执行（一定要用UFUNCTION）
	UFUNCTION()
    void MyTimelineTickCallBack(float Value);
    UFUNCTION()
    void MyTimelineFinishedCallBack();
```


### Step2:在.cpp文件中初始化Timeline（绑定事件和函数）

```
void AMyCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	if (MyCurve)//如果曲线存在
	{
		//注册float曲线更新事件，float曲线更新时执行名为MyTimelineTickCallBack的函数
        OnMyTimelineTickCallBack.BindUFunction(this, TEXT("MyTimelineTickCallBack"));
        MyTimeline.AddInterpFloat(MyCurve, OnMyTimelineTickCallBack);

		//注册时间轴播放结束事件，在播放完毕时执行名为MyTimelineFinishedCallBack的函数
        OnMyTimelineFinishedCallBack.BindUFunction(this, TEXT("MyTimelineFinishedCallBack"));
        MyTimeline.SetTimelineFinishedFunc(OnMyTimelineFinishedCallBack);
	}
}
```

### Step3:在.cpp文件中定义函数
```
void AMyCharacter::MyTimelineTickCallBack(float Value)
{
	// 此函数在时间轴更新时（每帧）调用
}

void AFPSCharacter::MyTimelineFinishedCallBack()
{
    // 此函数在时间轴完成时调用
}
```


### Step4:在.cpp文件中设置时间轴的Tick
这一步不会过渡消耗性能，只有在事件轴开始运行时才会消耗Tick
```
void AFPSCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	// 只要 Timeline 正在播放就更新它（包括正向和反向播放）
	if (MyTimeline.IsPlaying())
	{
		MyTimeline.TickTimeline(DeltaTime);
	}
}
```


### Step5:如何使用？
MyTimeline.操作名()
“播放”：MyTimeline.Play();
”逆向播放“：MyTimeline.Reverse();
“从头播放”：MyTimeline.PlayFromStart();
“从末尾逆向播放”：MyTimeline.ReverseFromEnd();
“停止播放”：MyTimeline.Stop();