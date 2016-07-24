---
layout: post
title: 如何创建没有默认构造函数的类的数组
---

因为我们无法在创建数组时给类的构造函数传递参数，所以创建没有默认构造函数的类的数组会有一些问题，但我们有三种方法来绕开这个限制。

```c++
class Person {
public:
	Person(int id) : id_(id) {}
	
private:
	int id_;
};

Person persons[2]; // error! No way to call constructor
Person *persons = new Person[2]; // error! No way to call constructor
```

## 1. 利用数组列表初始化和类的拷贝构造函数

```c++
Person persons[10] = {
    Person(0),
    Person(1),
    Person(2),
    Person(3),
    Person(4),
    Person(5),
    Person(6),
    Person(7),
    Person(8),
    Person(9),
};
```

缺点：

1. 只能用于非堆数组上。
2. 要手动输入每一个数组对象的构造函数。 

## 2. 利用指针数组来替代对象数组

```c++
Person *persons[10];
Person **persons = new Person *[10];

for (int i = 0; i < 10; ++i)
    persons[i] = new Person(i);
```

缺点：

1. 必须记得delete每一个数组元素，否则会发生内存泄漏。
2. 需要耗费额外的内存来存放指针。

## 3. 先给数组分配原始内存再利用placement new调用每一个数组元素的构造函数

```c++
Person *persons = static_cast<Person *>(operator new [](10 * sizeof(Person)));
for (int i = 0; i < 10; ++i)
    new (persons + i) Person(i);
```

缺点：

1. 必须手动调用每一个数组元素的析构函数，然后调用operator delete []来释放内存。
2. 如果使用普通的delete []会导致未定义行为。

```c++
for (int i = 0; i < 10; ++i)
    persons[i].~Person();
operator delete [](persons);
```
