---
layout: post
title: 不要重载&&、||和,
---

我们知道`&&`、`||`和`,`这三个操作符是具有短路特性的，那么如果我们重载了这几个操作符后它们是否还有短路特性呢。例如：

```c++
if (expression1 && expression2)
```

如果我们重载了`&&`操作符，那么其等同于下面的代码：

```c++
// when operator&& is a member function
if (expression1.operator&&(expression2))

// when operator&& is a global function
if (operator&&(expression1, expression2))
```

因为函数调用的参数计算顺序是未定义的，所以重载后的`&&`操作符失去了短路特性。

因为你无法保持`&&`、`||`和`,`操作符的短路特性，所以不要去重载它们。
