|                                                            |                                                            |                                                            |
| ---------------------------------------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| Tank                                                       | BasePawn                                                   | Tower                                                      |
| Super::HandleDestruction()                                 | HandleDestruction                                          | Super::HandleDestruction()                                 |
| ![Exported image](Exported%20image%2020251125141005-0.png) | ![Exported image](Exported%20image%2020251125141007-1.png) | ![Exported image](Exported%20image%2020251125141012-2.png) |
 
我认为其主要功能是：在父类中定义了一个函数后在子类中可以重写此函数，但是要用Super::增加新的定义等