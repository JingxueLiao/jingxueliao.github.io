---
layout: post
title: C++中的隐式类型转换
---

## 有两种方法可以给类增加隐式类型转换的能力

1. 单一参数的构造函数
2. 隐式类型转换操作符

```c++
class Rational {
public:
    // single-argument constructor
    Rational(int numerator = 0, int denominator = 1);

    // implicit type conversion operator
    operator double() const;
};
```

注意：

1. 单一参数的构造函数是指可以通过一个参数来调用的构造函数，它可以声明多个参数但都有缺省值。
2. 不能给隐式类型转换操作符指定返回类型，因为该函数的名字中已经指定了返回类型。

## 禁止类的隐式类型转换的方法

隐式类型转换的根本问题在于这些类型转换有可能在你不需要的时候执行，从而导致错误的程序行为并且难以判断错误原因。所以有时候我们会想要禁止类的隐式类型转换的能力。

### 隐式类型转换操作符

考虑下面的代码：

```c++
Rational r(1, 2);
cout << r << endl;
```

虽然我们没有重载接受Rational的operator <<，但是并没有编译错误，因为r被隐式转换成了double类型。

我们可以使用普通的成员函数代替隐式类型转换操作符完成类型转换，从而禁止通过隐式类型转换操作符进行隐式类型转换，例如std::string::c_str()。

### 单一参数的构造函数

1. explicit关键字
2. 利用隐式类型转换不能包含超过一个用户自定义类型转换规则，通过代理模式禁止隐式类型转换

```c++
class Rational {
public:
    class Integer {
    public:
        Integer(int i);
    };
    
    Rational(Integer numerator = 0, Integer denominator = 1);
};
```
