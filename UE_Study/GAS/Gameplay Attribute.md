[Gameplay Attributes and Attribute Sets for the Gameplay Ability System in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community](https://dev.epicgames.com/documentation/unreal-engine/gameplay-attributes-and-attribute-sets-for-the-gameplay-ability-system-in-unreal-engine)

AttributeSet需要挂载到Actor或PlayerState上


对于属性，可以添加为`FGameplayAttributeData`
```
UCLASS()  
class UMAttributeSet : public UAttributeSet  
{  
    GENERATED_BODY()  
public:  
          
private:  
    UPROPERTY()  
    FGameplayAttributeData Health;  
};
```
声明了一个名为Health 的属性集

对于Gameplay Attribute来说，有四个重要的宏


| 宏（带参数）｜生成函数的签名｜行为/用途｜

|                                                              |                                         |                                         |
| ------------------------------------------------------------ | --------------------------------------- | --------------------------------------- |
| `GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UMyAttributeSet, Health)` | `static FGameplayAttribute GetHealth()` | 静态函数从虚幻引擎的反射系统中返回`FGameplayAttribute`结构 |
| `GAMEPLAYATTRIBUTE_VALUE_GETTER(Health)`                     | `float GetHealth() const`               | 返回"生命值"游戏玩法属性的当前值                       |
| `GAMEPLAYATTRIBUTE_VALUE_SETTER(Health)`                     | `void SetHealth(float NewVal)`          | 将"生命值"游戏玩法属性的值设置为`NewVal`               |
| `GAMEPLAYATTRIBUTE_VALUE_INITTER(Health)`                    | `void InitHealth(float NewVal)`         | 将"生命值"游戏玩法属性的值初始化为`NewVal`              |
