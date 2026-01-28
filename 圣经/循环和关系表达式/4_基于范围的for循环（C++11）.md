C++11新增了一种循环：基于范围(range-based)的for循环。这简化了一种常见的循环任务：对数组(或容器类，如vector和array)的每个元素执行相同的操作，如下例所示:
```
double prices [5] = {4.99, 10.99, 6.87, 7.99, 8.49};
for (double x : prices)
	cout << x << std :: endl;
```
