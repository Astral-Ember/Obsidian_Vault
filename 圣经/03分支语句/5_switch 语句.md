如果出现了5个情况，使用if  else if  else语句可以处理这5种情况，但是C++的switch 语句可以更简单的处理
```
switch (integer-expression)
{
	case labell : statement (s)
	case label2 : statement (s)
	...
	default: statement (s)
}
```
C++的switch语句就像指路牌，告诉计算机接下来应执行哪行代码。执行到switch语句时，程序将跳到使用integer-expression的值标记的那一行。例如，如果integer-expression的值为4，则程序将执行标签为case 4:那一行。顾名思义，integer-expression必须是一个结果为整数值的表达式另外,每个标签都必须是整数常量表达式。最常见的标签是int或char常量(如1或’q')，也可以是枚举量。如果integer-expression不与任何标签匹配，则程序将跳到标签为default的那一行。Default 标签是可选的，如果被省略，而又没有匹配的标签，则程序将跳到switch后面的语句处执行

==注意：==：程序跳到switch中特定代码行后，将依次执行之后的所有语句，除非有明确的其他指示。程序不会在执行到下一个case处自动停止，要让程序执行完一组特定语句后停止，必须使用break语句。这将导致程序跳到switch后面的语句处执行

## 1.将枚举量用作标签
switch语句将int值和枚举量标签进行比较时，将枚举量提升为int。另外，在while循环测试条件中，也会将枚举量提升为int类型。