---
layout: post
title: 不要尝试对数组使用多态
---

使用多态时，我们可以通过父类的指针或引用来操作子类的对象。由于对数组进行操作时，往往转换成指针进行操作，这会给我们一个错觉，以为能够使用多态数组，例如把存放子类对象的数组当成存放父类对象的数组来操作，但是结果往往出人意料。下面给出一个具体的例子：

```c++
#include <iostream>
#include <vector>

using namespace std;

class Father {
public:
    virtual void print() const {
        cout << "Father" << endl;
    }
};

class Son : public Father {
public:
    virtual void print() const {
        cout << "Son" << endl;
    }

private:
    string father;
};

void print(Father fathers[], int size) {
    for (int i = 0; i < size; ++i)
        fathers[i].print();
}

int main() {
    Father fathers[10];
    Son sons[10];
    print(fathers, 10);
    print(sons, 10);
    return 0;
}
```

上面这个程序可以编译通过，但运行后会异常终止，问题在于取数组元素`fathers[i]`会转换成指针操作`*(fathers + i * sizeof(Father))`，由于Son和Father大小不同，导致指针偏移量计算错误，所以指针的解引用会产生未定义行为。

delete一个数组时也会产生类似错误，因为对数组进行`delete []`操作时，会依次对数组的每个元素调用析构函数。

```c++
delete [] array;

// generate code
for (int i = 0; i < array.size(); ++i)
	array[i].Type::~Type();
```

所以通过父类对象的指针来删除存放子类对象的数组会导致未定义行为。
