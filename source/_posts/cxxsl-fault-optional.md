---
title: C++ 标准库缺陷：optional
date: 2022-09-21 00:24:22
tags:
  - C++
---

## History

C++17 引入了 `std::optional`(看起来是来自 `boost::hana::optional`) 用来表示可能持有值的类型，相当于 `Maybe`(haskell) 或者是 `Optional`(java)。但是当时这个类并不非常好用，因为 api 设计的种种问题，很多时候不得不写这样的代码：

```c++
if (a.has_value()) {
  b = do_something(a.value())
} else {
  b = nullopt;
}
```

这自然是十分讨厌的，于是 C++23 委员会加入了如下的 monadic 特性：

```c++
template <class Self, class Func>
constexpr auto std::optional<T>::and_then(this Self&& self, Func&& func);

template <class Self, class Func>
constexpr auto std::optional<T>::transform(this Self&& self, Func&& func);

template <class Self, class Func>
constexpr auto std::optional<T>::or_else(this Self&& self, Func&& func);
```

## 说明

但是 `and_then` 和 `transform` 存在一些设计上的问题，我们知道，函数有如下签名：

```c++
SomeType f();
```

一般来说，`SomeType` 绝不会是不完整类型，除了一种例外，有 cv 限定的 `void`，对于这种情况，大部分函数有关的类型都会谨慎处理。

很可惜，在 `std::optional` 上面并没有，我们不可能有 `std::optional<void>`，因为类型参数有引用的约束不满足，这显然是标准库的过失。

## fix

- 特化 `std::optional<void>`，`has_value` 恒定为假。
- 添加函数 `std::optional<T>::for_each`，就像算法库那样。

