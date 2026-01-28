while 循环是没有初始化和更新部分的for循环，它只有测试条件和循环体:
```
while (test-condition)
	body
```
首先，程序计算圆括号内的测试条件(test-condition)表达式。如果该表达式为true，则执行循环体中的语句
执行完循环体后，程序返回测试条件，并对它进行重新评估

## 1.for和while
在C++中，for和while循环本质上是相同的。例如，下面的for循环:
```
for (init-expression; test-expression; update-expression)
{
	statement (s)
}
```
可以改写成这样：
```
init-expression;
while (test-expression)
{
	statement (s)
	update-expression;
}
```
同样，下面的while循环：
```
while (test-expression)
	body
```
可以改成这样：
```
for ( ;test-expression; )
	body
```

通常，使用for循环来为循环计数，因为for循环格式允许将所有相关的信息——初始值、终止值和更新计数器的方法——放在同一个地方。在无法预先知道循环将执行的次数时，就可以使用while循环

在设计循环时有下面几条指导原则
- 指定循环终止的条件
- 在首次测试之前初始化条件
- 在条件被再次测试前更新条件