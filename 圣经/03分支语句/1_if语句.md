当C++程序必须决定是否执行某个操作时，通常使用if语句来实现选择。if有两种格式:if和if else

if 语句的语法与while 相似:
```
if (test-condition)
	statement
```
如果test-condition(测试条件)为true，则程序将执行statement(语句)，反之则跳过statement

## 1. if else 语句
if语句让程序决定是否执行特定的语句或语句块，而if else语句则让程序决定执行两条语句或语句块中的哪一条，这种语句对于选择其中一种操作很有用
if else 语句的通用格式如下
```
if (test-condition)
	statement1
else
	statement2
```
如果测试条件为true或非零，则程序将执行statement1，跳过statement2；如果测试条件为false或0，则程序将跳过statement1，执行statement2

## 2. if  else if  else结构
当处理两个以上的选择时，可以选择这种结构。else之后可以是一条语句，也可以是语句块。由于if else语句本就是一条语句，可以放在else 后面