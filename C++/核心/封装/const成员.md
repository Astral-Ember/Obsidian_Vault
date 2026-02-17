const成员

1. 知识回顾 [常量](C++/入门/常量.md)
2. const成员变量
3. const成员函数
4. mutable关键字

常量是指在程序执行过程中其值不能被改变的变量，大部分的常量在声明时都必须初始化  
关键点：  
必须初始化  
不能被修改  
指向常量的指针： const在 * 前面添加即可，值不能改，指向可以改  
指针常量： const在 * 后加，值可以改，指向不能改  
指向常量的指针常量： const在 * 前后都加，值和指向都不能改
 
规则及作用和常量知识点一样  
唯一需要注意的是  
如果想要从外部传值初始化const成员变量 需要在构造函数的初始化列表中初始化

|   |
|---|
|test::test():PI(3.14)  <br>{  <br>}|

规则  
在声明完成员函数后 在后方加上const关键字

|   |
|---|
|void test::show() const  <br>{  <br>cout \<\< "PI=" \<\< PI \<\< endl;  <br>}|

作用  
函数中只能读取 成员变量，不能修改成员变量（但是由于static变量存储位置与其他变量不一样，所以static可以被修改）  
const对象只能调用const方法 不能调用非const方法

![Exported image](Exported%20image%2020251125140134-0.png)  
![Exported image](Exported%20image%2020251125140136-1.png)

注意  
不管成员是常量还是普通成员 都不能在const函数中进行修改  
作用：避免在某一个函数中去修改类对象的成员变量  
如果我们想在const方法中修改普通成员变量  
我们可以在声明该变量时，在前面加上mutable关键字  
.h

|   |
|---|
|#pragma once  <br>class test  <br>{  <br>public:  <br>const float PI;  <br>mutable int a = 10;  <br>test();  <br>void show()const;  <br>void change(float)const;  <br>};|

.cpp

|   |
|---|
|#include "test.h"  <br>#include\<iostream\>  <br>using namespace std;  <br>test::test():PI(3.14)  <br>{  <br>}<br><br>  <br><br>void test::show() const  <br>{  <br>cout \<\< "PI=" \<\< PI \<\< endl;  <br>}<br><br>  <br><br>void test::change(float a) const  <br>{  <br>this-\>a = a;  <br>}<br><br>  <br><br>int main()  <br>{  <br>test t;  <br>t.show();  <br>const test t1;  <br>t1.show();  <br>}|