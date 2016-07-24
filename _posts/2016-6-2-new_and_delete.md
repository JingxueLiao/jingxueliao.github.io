---
layout: post
title: C++中的new和delete详解
---

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

## new与数组

我们可以使用new []和delete []来创建和删除数组。new []调用operator new[]函数来分配内存，然后对数组的每个元素调用构造函数。delete []对每个数组元素调用析构函数，然后调用operator delete[]释放内存。
