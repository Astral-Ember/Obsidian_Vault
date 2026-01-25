|   |
|---|
|1.项目中所有的C++文件|
|2.将所有这些文件输入Unreal Header Tool(UHT；虚幻头文件工具）的工具  <br>这是虚幻引擎中的一个程序，它将查看我们所有源代码文件中虚幻特定语法并生成更多的C++代码；比如UCLASS、UPROPERTY什么的|
|3.Standard C++ Compilation 进行标准C++编译￼Perprocessor预处理器-\>Compiler编译器-\>Linker链接器￼预处理器与UHT有些类似，都会查看源代码文件和预处理器指令（#include之类的）￼把预处理器处理过后的文件交给编译器，编译器把这些文件编译成二进制可执行代码，但是这些文件仍然是多个独立的文件￼最后一步是将这些可执行 的二进制代码文件输入链接器，链接器会创建所有必要的连接，并输出最后的单个可执行文件 (.exe)|
 
e.g.  
有三个文件

|   |   |   |   |   |
|---|---|---|---|---|
|预处理器|||编译器|链接器|
|game.cpp  <br>#include "math.h"  <br>void MyGame()  <br>{  <br>Add(1, 2);  <br>}|引入头文件调用函数|game.i  <br>int Add(int a, int b);  <br>void MyGame()  <br>{  <br>Add(1, 2);  <br>}|game.obj  <br>转换为二进制机器码|game.exe|
|math.h  <br>int Add(int a, int b);|声明函数||||
|math.cpp  <br>int Add(int a, int b);  <br>{  <br>return a + b;  <br>}|定义函数|math.i  <br>int Add(int a, int b);  <br>{  <br>return a + b;  <br>}|math.obj  <br>转换为二进制机器码||