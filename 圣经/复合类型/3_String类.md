使用指针来处理字符串
在很多方面，使用string对象的方式与使用字符数组相同

可以使用C-风格字符串来初始化string对象。
可以使用cin来将键盘输入存储到string 对象中。
可以使用cout来显示string对象。
可以使用数组表示法来访问存储在string对象中的字符。


## 1.赋值、拼接和附加

使用字符数组时不能将一个数组赋值给另一个数组，但是可以将一个string对象赋值给另一个string对象
```
char charr1 [20]; // create an empty array
char charr2 [20] = "jaguar"; // create an initialized array
string strl; // create an empty string object
string str2 = "panther"; // create an initialized string
charr1 = charr2;// INVALID, no array assignment
strl = str2; VALID, object assignment ok
```

string类简化了字符串合并操作。可以使用运算符+将两个string对象合并起来，还可以使用运算符+=将字符串附加到string对象的末尾。
```
string str3;
str3 = strl + str2;// assign str3 the joined strings
str1 += str2;// add str2 to the end of str1
```



## 2.string类 I/O
可以使用cin和运算符 >> 来将输入存储到string对象中，使用cout和运算符 << 来显示string 对象，其句法与处理C-风格字符串相同。但每次读取一行而不是一个单词时，使用的句法不同
C-风格字符串读取一行使用
`cin.getline(charname,size)`
但是string不知道字符串长度，所以使用
`getline(cin,strname)`

