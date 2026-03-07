
## 1.定义GamePlayTags
### 创建.h
```
#pragma once  
  
#include "CoreMinimal.h"  
#include "NativeGameplayTags.h"  
  
namespace GameplayTags  
{  
    namespace InputTags  
    {  
       UE_DECLARE_GAMEPLAY_TAG_EXTERN(InputTag_Move);  
       UE_DECLARE_GAMEPLAY_TAG_EXTERN(InputTag_Look);  
    }}
```
### 创建.cpp
```
#include "WarriorGameplayTags.h"  
  
namespace GameplayTags  
{  
    namespace InputTags  
    {  
       UE_DEFINE_GAMEPLAY_TAG_COMMENT(InputTag_Move, "GameplayTags.InputTag.Move", "A tag that identifies the input action for moving.");  
       UE_DEFINE_GAMEPLAY_TAG_COMMENT(InputTag_Look, "GameplayTags.InputTag.Look", "A tag that identifies the input action for looking.");  
    }}
```



## 2.创建DataAsset，继承自UDataAsset
在该文件中一般需要创建一个结构体以关联Tag和InputAction；在类中要定义的内容一般是：`UInputMappingContext`的数组、一个刚刚定义的结构体数组、一个函数用来根据tag返回inputaction
### .h
```
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "GameplayTagContainer.h"  
#include "Engine/DataAsset.h"  
#include "DataAsset_InputConfig.generated.h"  
  
class UInputMappingContext;  
class UInputAction;  
/**  
 * FWarriorInputActionConfig ** 一个简单的结构体，用于将 GameplayTag 与特定的 InputAction 关联起来。  
 */USTRUCT(BlueprintType)  
struct FWarriorInputActionConfig  
{  
    GENERATED_BODY()  
  
    /** 关联的输入标签 */    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, meta = (Categories = "GameplayTags"))  
    FGameplayTag InputTag;  
  
    /** 关联的输入动作 */    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)  
    UInputAction* InputAction;  
};  
  
  
/**  
 * UDataAsset_InputConfig ** 存储所有输入映射上下文 (IMC) 和输入动作 (IA) 配置的数据资产。  
 */UCLASS()  
class WARRIOR_API UDataAsset_InputConfig : public UDataAsset  
{  
    GENERATED_BODY()  
  
public:  
    /** 默认的输入映射上下文列表 */    UPROPERTY(EditDefaultsOnly, Category = "Warrior|Input|Input Mappings")  
    TArray<UInputMappingContext*> DefaultMappingContexts;  
  
    /** 定义的输入动作列表 */    UPROPERTY(EditDefaultsOnly, Category = "Warrior|Input|Input Actions", meta = (TitleProperty = "InputTag"))  
    TArray<FWarriorInputActionConfig> InputActionLists;  
  
    /** 根据标签查找输入动作 */    UInputAction* FindInputActionForTag(const FGameplayTag& InputTag) const;  
};
```
### .cpp
```
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "DataAsset/Input/DataAsset_InputConfig.h"  
  
UInputAction* UDataAsset_InputConfig::FindInputActionForTag(const FGameplayTag& InputTag) const  
{  
    // 遍历输入动作列表查找匹配的标签  
    for (const FWarriorInputActionConfig& Action : InputActionLists)  
    {       if (Action.InputTag == InputTag && Action.InputAction)  
       {          return Action.InputAction;  
       }    }  
    // 如果没有找到匹配项，返回空  
    return nullptr;  
}
```

## 3.创建新的InputComponent继承自UEnhancedInputComponent
定义新的绑定函数模板，用来通过GameplayTag绑定动作。还需要在引擎的项目设置中替换默认的nhancedInputComponent
### .h
```
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "EnhancedInputComponent.h"  
#include "DataAsset/Input/DataAsset_InputConfig.h"  
#include "WarriorInputComponent.generated.h"  
  
/**  
 * UWarriorInputComponent ** 自定义输入组件类，继承自 UEnhancedInputComponent，  
 * 提供基于 GameplayTag 绑定输入动作的便捷模板函数。  
 */UCLASS()  
class WARRIOR_API UWarriorInputComponent : public UEnhancedInputComponent  
{  
    GENERATED_BODY()  
  
public:  
    /**  
     * 使用 GameplayTag 绑定输入动作  
     * @param InputConfig 输入配置数据资产  
     * @param InputTag 关联的 GameplayTag     * @param TriggerEvent 触发事件类型 (例如 Triggered)     * @param Object 绑定对象  
     * @param Func 绑定的回调函数  
     */    template <typename UserObject, typename CallbackFunc>  
    void BindInputAction(const UDataAsset_InputConfig* InputConfig, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent, UserObject* Object,  
                         CallbackFunc Func);  
};  
  
template <typename UserObject, typename CallbackFunc>  
void UWarriorInputComponent::BindInputAction(const UDataAsset_InputConfig* InputConfig, const FGameplayTag& InputTag, ETriggerEvent TriggerEvent,  
                                             UserObject* Object,  
                                             CallbackFunc Func)  
{  
    // 通过 Tag 获取对应的 InputAction 并执行绑定,由于继承自 UEnhancedInputComponent，所以可以直接使用BindAction  
    BindAction(InputConfig->FindInputActionForTag(InputTag), TriggerEvent, Object, Func);  
}
```

## 4.在Player Controller中绑定动作
需要获取增强输入子系统等一系列工作，以及定义回调函数

### .h
```
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "CoreMinimal.h"  
#include "GameFramework/PlayerController.h"  
#include "WarriorPlayerController.generated.h"  
  
  
class UDataAsset_InputConfig;  
struct FInputActionValue;  
/**  
 * AWarriorPlayerController ** 基础玩家控制器类，负责处理输入绑定和玩家移动逻辑。  
 */UCLASS()  
class WARRIOR_API AWarriorPlayerController : public APlayerController  
{  
    GENERATED_BODY()  
  
public:  
    /** 覆写输入组件设置，用于绑定增强输入 */    virtual void SetupInputComponent() override;  
  
private:  
    /** 输入配置数据资产，包含 Mapping Contexts 和 Input Actions */    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Warrior|Input", meta=(AllowPrivateAccess="true"))  
    TObjectPtr<UDataAsset_InputConfig> InputConfig;  
    /** 处理移动输入的函数 */    void Move(const FInputActionValue& Value);  
  
    /** 处理视角转动输入的函数 */    void Look(const FInputActionValue& Value);  
};
```

### .cpp
```
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "Player/WarriorPlayerController.h"  
#include "EnhancedInputSubsystems.h"  
#include "InputActionValue.h"  
#include "WarriorGameplayTags.h"  
#include "Component/WarriorInputComponent.h"  
#include "DataAsset/Input/DataAsset_InputConfig.h"  
  
  
void AWarriorPlayerController::SetupInputComponent()  
{  
    Super::SetupInputComponent();  
  
    // 仅为本地玩家控制器添加输入映射上下文 (IMC)    if (IsLocalPlayerController())  
    {       // 获取增强输入子系统  
       if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(GetLocalPlayer()))  
       {          // 从配置中加载默认的映射上下文  
          for (UInputMappingContext* CurrentContext : InputConfig->DefaultMappingContexts)  
          {             Subsystem->AddMappingContext(CurrentContext, 0);  
          }       }    }    // 转换输入组件并绑定特定的输入动作 (Input Actions)    if (UWarriorInputComponent* WarriorInputComponent = CastChecked<UWarriorInputComponent>(InputComponent))  
    {       // 绑定移动输入  
       WarriorInputComponent->BindInputAction(InputConfig, GameplayTags::InputTags::InputTag_Move, ETriggerEvent::Triggered, this,  
                                              &AWarriorPlayerController::Move);  
  
       // 绑定视角转动输入  
       WarriorInputComponent->BindInputAction(InputConfig, GameplayTags::InputTags::InputTag_Look, ETriggerEvent::Triggered, this,  
                                              &AWarriorPlayerController::Look);  
    }}  
  
void AWarriorPlayerController::Move(const FInputActionValue& Value)  
{  
    // 确保 Pawn 有效  
    if (!IsValid(GetPawn()))  
       return;  
  
    // 获取二维移动向量  
    FVector2D MoveVector = Value.Get<FVector2D>();  
  
    // 获取控制器的偏转角度 (Yaw)    const FRotator YawRotation(0.f, GetControlRotation().Yaw, 0.f);  
  
    // 基于相机朝向计算前方向和右方向  
    const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);  
    const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);  
  
    // 添加移动输入  
    GetPawn()->AddMovementInput(ForwardDirection, MoveVector.Y);  
    GetPawn()->AddMovementInput(RightDirection, MoveVector.X);  
}  
  
void AWarriorPlayerController::Look(const FInputActionValue& Value)  
{  
    // 获取二维转动向量  
    FVector2D LookVector = Value.Get<FVector2D>();  
  
    // 添加偏转 (Yaw) 和俯仰 (Pitch) 输入  
    AddYawInput(LookVector.X);  
    AddPitchInput(LookVector.Y);  
}
```


### 总结
这个方法相较于普通的方法：把动作绑定放在了Player Controller中，并使用GamePlayTag替换了先前冗杂的一系列inputaction声明，总之，好用