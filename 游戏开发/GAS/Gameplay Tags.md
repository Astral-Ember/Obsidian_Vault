[Using Gameplay Tags in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-gameplay-tags-in-unreal-engine)
GameplayTag并不局限于GAS，也可以在GAS项目中使用，但是Tag在GAS中占据了核心地位，标签可以用于识别各种能力，标签可以分配给角色的AbilityComponent用来标示任何特定类型的状态


创建文件以自定义标签
在头文件中定义命名空间，避免名称冲突
```
#include "CoreMinimal.h"  
#include "NativeGameplayTags.h"  
  
namespace GASTags  
{  
    namespace GASAbilities  
    {  
       UE_DECLARE_GAMEPLAY_TAG_EXTERN(Primary);  
    }
}
```

在.cpp文件中定义GameplayTags
```
#include "GameplayTags/GASTags.h"  
  
namespace GASTags  
{  
    namespace GASAbilities  
    {  
       //第一个是标签名，第二个是层级，第三个是描述  
       UE_DEFINE_GAMEPLAY_TAG_COMMENT(Primary, "GASTags.GASAbilities.Primary", "Tag for the primary Ability");  
    }
}
```

在C++中使用Tag的方式是作用域解析符::
`GASTags::GASAbilities::Primary`