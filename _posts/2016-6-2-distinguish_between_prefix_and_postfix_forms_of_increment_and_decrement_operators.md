---
layout: post
title: ++/--的前缀和后缀形式
---

```c++
class Integer {
public:
    // prefix ++
    Integer &operator++() {
        ++num;
        return *this;
    }

    // postfix ++
    const Integer operator++(int) {
        Integer old = *this;
        ++(*this);
        return old;
    }

    // prefix --
    Integer &operator--() {
        --num;
        return *this;
    }

    // postfix --
    const Integer operator--(int) {
        Integer old = *this;
        --(*this);
        return old;
    }

private:
    int num;
};
```

注意：

1. 后缀形式多一个int类型参数，调用时会给其传0。
2. 前缀形式返回该类型引用，后缀形式返回const对象。
3. 因为后缀形式并不使用传进来的int类型参数，所以可以省略该参数名字从而消除编译器警告。
4. 把后缀形式返回的对象声明为const是为了避免错误地对该临时变量进行修改，从而导致不希望的结果。
5. 前缀形式效率更高，因为后缀形式需要创建一个临时对象。
6. 后缀形式应该调用前缀形式来实现，这样只需要维护前缀形式的版本，而后缀形式自动表现出一致性。

