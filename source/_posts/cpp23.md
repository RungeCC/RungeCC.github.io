---
title: C++ 23
date: 2022-09-12 19:42:39
tags:
  - C++
---

## assume

assume 是一个 expression-liked 属性，语法形如：

```c++
[[assume(expr)]]
```

其中 `expr` 具有 `bool` 类型，这段代码用来告诉优化器存在形如 `expr` 的约束不变式以资参考，例如：

```c++
void do_something(int var) {
  [[assume(var > 0)]];
  if (var <= 0 ) do_1();
  else do_2();
}
```

这种情况下，优化器将优化掉不可能进入的分支，同时，如果 `assume` 被违背将导致未定义行为。

`assume` 表达式（用来指代 `expr`）具有下面的特点：

- 不求值；
- 要求 ODR 使用。

## static `operator()`

这个提案允许成员运算符 `operator()` 被声明为静态的，避免毫无意义的 `this` 指针传递的开销：

```c++
template <class T>
class Foo {
public:
  static constexpr bool operator()(T const& val) {
    // some expr...
    return true;
  }
};
```

这样做是有意义的，因为 C++ 存在大量用只有定义了 `operator()` 的类传递函数对象，例如：`std::less` 类模板。

同样，现在允许用 `static` 修饰 lambda，这种情况下不允许捕捉。

```c++
auto lmd = []<class T>(T const& val) static -> bool { 
  return true; 
};
```

## Deducing `this`

这个提案允许显示的指定成员函数的第一个参数（`this` 指针）的类型，按照如下语法：

```c++
class Foo {
public:
  template <class SomeType, class...OtherTypes>
  void bar(this SomeType self, OtherTypes...args);
};
```

这有一个明显的优点，过去，如果要准备完美转发的成员函数调用需要写全部的四个重载：

```c++
class Foo {
public:
  void do_something() &;
  void do_something() &&;
  void do_something() const&;
  void do_something() const&&;
};
```

这样繁琐的事情现在只需要通过简单的模板 `this` 参数来实现：

```c++
class Foo {
public:
  template <class Self>
  void do_something(this Self&& self);
};
```

还允许获取 lambda 函数自身的引用，这就意味着可以写递归 lambda：

```c++
auto lmd = [](this auto& self, int n) {
  return n == 0 ? 1 : n * self(n - 1);
};
```

这点不难理解，使用[C++ Insights](https://cppinsights.io/)可以知道 lambda 会被当作一个重载了 `operator()` 的匿名类的对象，他的调用相当于 `operator()` 的调用。

## std module

对标准库提供最低限度的模块支持。

C++ 20 最为重要的四个特性（Coroutine, Concept, Range, Module）之一就是模块，他彻底改变了 C++ 的编译-依赖样式（以至于 cmake 暂时不支持 module，[issue#18355](https://gitlab.kitware.com/cmake/cmake/-/issues/18355)），但是标准库都缺乏 module，以至于这个特性很难入其他三大特性一样得到很好的利用。

BS 提议加入 `import std` 和 `import std.compact`，前者引入所有 `::std` 的公开符号，后者是兼容性的，他引入的 C 函数在全局名字空间中（因为，显然，很多代码使用的是 `fopen` 而并非 `std::fopen`）。

注意，std module 不会导出任何宏，这意味着：

- 如果需要特性测试宏，需要 `import <versions>`；
- 如果需要 C 风格的宏函数，则要引入对应的头文件；