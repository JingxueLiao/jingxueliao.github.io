---
layout: post
title: C++中的new和delete详解
---

允许对内存的手动管理，是C++的一个重要特性。通过研究程序使用内存的特点，并据此提供合适的内存分配和回收例程，我们可以得到最好的性能。

但是在此之前，我们需要完全理解C++中的内存管理行为。

## new operator

```c++
string *s = new string("Memory Management");
```

在上面这个程序片段中，我们所使用的new是new operator，new operator是一个操作符，它是C++语言内置的，我们无法改变它的含义。
 
 new operator所完成的功能可以分为两部分：
 
1. 调用operator new函数分配足够存放请求对象的原始内存。
2. 在分配的内存上调用构造函数来初始化对象。

上面这个程序片段生成的代码大致如下：

```c++
// allocate raw memory for a string object
void *memory = operator new(sizeof(string));

// initialize the object in the memory
call string::string("Memory Management") on memory;

// make ps point to the new object
string *s = static_cast<string *>(memory);
```

注意上面第二步中在指定内存上调用构造函数并不能由程序员完成，你只能调用new operator然后由编译器完成。

## operator new

new operator调用operator new函数来分配内存，我们可以重写或重载该函数来改变分配内存的行为。

通常operator new的声明和调用方法如下：

```c++
void *operator new(size_t size);

void *raw_memory = operator new(sizeof(Type));
```

注意：

1. operator new函数的返回类型是void *，因为该函数返回的是未初始化的原始内存。
2. size指定分配多少内存。
3. 可以通过增加其他参数重载operator new，但第一个参数必须是size_t。
4. operator new只负责分配内存，由new operator来负责在分配的内存上构造对象。

## placement new

广义地说，placement new是指除了size_t还接受其他参数的operator new。一般情况下，我们所说的placement new是指除了size_t参数还接受void*参数的operator new。

placement new的常用方式如下：

```c++
void *operator new(size_t, void *location) {
	return location;
}

void *memory = operator new(sizeof(string));
string *s = new (memory) string("Memory Management");
```

## new小结

1. 如果你想在堆上创建一个对象，调用new operator。
2. 如果你只想要分配内存，调用operator new函数。
3. 如果你在堆上创建对象但定制内存分配行为，你可以重写operator new函数并调用new operator。
4. 如果你想要在指定内存构建对象，使用placement new。

## delete operator

delete operator与new operator类似，它的功能也分为两部分：

1. 析构对象。
2. 调用operator delete释放内存。

```c++
string *s;
delete s;
```

生成的代码大致如下：

```c++
s->~string();
operator delete(s);
```

注意:

1 和构造函数不同，你可以直接调用对象的析构函数，所以你无需使用placement delete并使用delete operator来调用堆对象的析构函数。placement delete是用在当new operator使用placement new时，若构造函数抛出异常可让编译器自动调用以避免内存泄漏。

2 如果你只使用operator new来分配内存而没有构造对象，那你也只能使用operator delete释放内存，而不能使用delete operator，否则在未初始化的内存上调用析构函数会导致未定义行为。

```c++
void *buffer = operator new(128 * sizeof(char));

...

operator delete(buffer);
```

3 如果你使用placement new来分配内存，那你也不应该使用delete operator，因为delete operator调用operator delete函数释放内存，但在placement new中，该内存不一定是operator new函数分配的。你应该显示调用析构函数，然后调用适当函数释放内存。

```c++
void *p = malloc(sizeof(string));
string *s = new (p) string("Memory Management");
...
s->~string();
free(p);
```

## 数组形式的new和delete

我们可以使用new []和delete []来创建和删除数组。new []调用operator new[]函数来分配内存，然后对数组的每个元素调用构造函数。delete []对每个数组元素调用析构函数，然后调用operator delete[]释放内存。

## 使用相同形式的new和delete

new需要和delete搭配使用，new []需要和delete[]搭配使用，混合使用会导致未定义行为。

造成这一问题的原因在于，给单一对象和对象数组分配的的内存布局不同。new []在分配内存时，除了数组需要占用的内存，还需要多分配一些内存来记录数组信息（可以想象成在数组前面有一个int来记录数组大小），然后new []会对每一个数组元素调用构造函数。delete []首先查询数组大小，然后对每一个数组元素调用析构函数，最后释放数组和数组信息所占用的内存。

注意如果用typedef为数组定义了别名，对该别名的new需要配合delete []使用。

```c++
typedef string StringArray[2];
string *s = new StringArray;
delete [] s;
```

## new handler

当operator new无法分配请求的内存时，它会抛出`bad_alloc`异常。但是如果安装了错误处理函数`new_handler`，operator new会调用它处理。可以通过调用`set_new_handler`函数来设置`new_handler`。

`new_handler`和`set_new_handler`的声明如下：

```c++
namespace std {
    typedef void (*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```

`set_new_handler`返回之前设置的`new_handler`。

如果operator new不能分配请求的内存，它会不断地调用`new_handler`，直到它能够找到足够的内存。所以一个良好设计的`new_handler`应该完成下面所列举的事情之一。

1. 使更多的内存可用。
2. 安装另一个`new_handler`或改变自身的行为。
3. 卸载`new_handler`。
4. 抛出`bad_alloc`或派生自`bad_alloc`的异常，operator new函数不会catch该类异常，其被传递到请求内存处。
5. 不返回，调用`abort`或`exit`等函数退出。

C++并不支持为每一个类指定专属的`new_handler`，但我们可以轻松地模拟出该行为。

```c++
class NewHandlerHolder {
public:
    explicit NewHandlerHolder(new_handler p) : handler(p) {}

    ~NewHandlerHolder() {
        set_new_handler(handler);
    }

private:
    new_handler handler;

    NewHandlerHolder(const NewHandlerHolder &);
    NewHandlerHolder &operator=(const NewHandlerHolder &);
};

class Widget {
public:
    static new_handler set_new_handler(new_handler p) throw();
    static void *operator new(size_t size) throw(bad_alloc);
    static void *operator new[](size_t size) throw(bad_alloc);

private:
    static new_handler handler;
};

new_handler Widget::handler = nullptr;

new_handler Widget::set_new_handler(new_handler p) throw() {
    new_handler old = handler;
    handler = p;
    return old;
}

void *Widget::operator new(size_t size) throw(bad_alloc) {
    NewHandlerHolder holder(std::set_new_handler(handler));
    return ::operator new(size);
}

void *Widget::operator new[](size_t size) throw(bad_alloc) {
    NewHandlerHolder holder(std::set_new_handler(handler));
    return ::operator new(size);
}
```

为了复用这一实现，我们可以把其作为基类，则其派生类可以继承它指定专属`new_handler`的能力，但是基类的属性会被派生类共享，为了让每一个派生类都有自己的`new_handler`，我们可以把该基类转成一个template。

```c++
class NewHandlerHolder {
public:
    explicit NewHandlerHolder(new_handler p) : handler(p) {}

    ~NewHandlerHolder() {
        set_new_handler(handler);
    }

private:
    new_handler handler;

    NewHandlerHolder(const NewHandlerHolder &);
    NewHandlerHolder &operator=(const NewHandlerHolder &);
};

template <typename T>
class NewHandlerSupport {
public:
    static new_handler set_new_handler(new_handler p) throw();
    static void *operator new(size_t size) throw(bad_alloc);
    static void *operator new[](size_t size) throw(bad_alloc);

private:
    static new_handler handler;
};

template <typename T>
new_handler NewHandlerSupport<T>::handler = nullptr;

template <typename T>
new_handler NewHandlerSupport<T>::set_new_handler(new_handler p) throw() {
    new_handler old = handler;
    handler = p;
    return old;
}

template <typename T>
void *NewHandlerSupport<T>::operator new(size_t size) throw(bad_alloc) {
    NewHandlerHolder holder(std::set_new_handler(handler));
    return ::operator new(size);
}

template <typename T>
void *NewHandlerSupport<T>::operator new[](size_t size) throw(bad_alloc) {
    NewHandlerHolder holder(std::set_new_handler(handler));
    return ::operator new(size);
}
```

有了该template，为一个类增加指定专属`new_handler`的能力可以通过简单的继承实现。

```c++
class Widget : public NewHandlerSupport<Widget> {
};
```

在1993年以前，如果operator new函数不能分配请求的内存，它会返回空指针。要想兼容以前的代码，可以使用nothrow形式的new。

```c++
Widget *widget = new (nothrow) Widget();
```

注意，nothrow形式的new并不能保证不抛出异常，因为虽然nothrow形式的operator new函数不会抛出异常，但接着调用的构造函数却可能抛出异常。

