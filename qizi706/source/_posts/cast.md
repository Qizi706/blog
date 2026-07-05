---
title: C++ 类型转换：从 C 风格强转到四种 cast

date: 2026-05-22 20:39:00
categories:
  - [C/C++]
tags:
  - [C++特性]
---

在 C++ 中，类型转换是一个很常见但也很容易写出隐患的知识点。对于初学者来说，最熟悉的可能是 C 风格强转：

```cpp
int x = (int)3.14;
```

但是在 C++ 项目中，更推荐使用 C++ 提供的四种显式类型转换：

```cpp
static_cast<T>(expr);
dynamic_cast<T>(expr);
const_cast<T>(expr);
reinterpret_cast<T>(expr);
```

它们分别表达不同的转换意图，比 C 风格强转更清晰，也更容易在代码审查和调试时发现问题。

本文会从 C 风格强转的问题开始，依次介绍 `static_cast`、`dynamic_cast`、`const_cast` 和 `reinterpret_cast` 的使用场景、底层含义、常见坑以及面试回答方式。

<!--more-->

## 1. 为什么不推荐 C 风格强转？

C 风格强转写法如下：

```cpp
double d = 3.14;
int x = (int)d;
```

这种写法简单，但问题是：**语义不明确**。

在 C++ 中，C 风格强转可能同时包含多种转换行为，例如：

```cpp
const int a = 10;
int *p = (int *)&a;
```

这段代码不仅把 `const int*` 转成了 `int*`，还去掉了 `const` 限定。也就是说，它可能同时做了类似 `const_cast` 和其他指针转换的事情。

C++ 更推荐把转换意图写清楚：

```cpp
static_cast<int>(d);
const_cast<int *>(&a);
reinterpret_cast<std::uintptr_t>(ptr);
```

这种代码可以立刻看出：你是在做普通类型转换、去掉 const，还是在做底层二进制解释。

因此不推荐 C 风格强转的原因就是：

> C 风格强转太粗暴，C++ cast 更明确。

---

## 2. static_cast：最常用的普通类型转换

`static_cast` 是 C++ 中最常见的类型转换。

它适合用于：

1. 基本类型转换；
2. 避免整数除法；
3. 继承体系中的向上转型；
4. 在程序员保证安全的情况下做向下转型；
5. `void*` 和具体类型指针之间的转换；
6. 枚举和整数之间的转换。

### 2.1 基本类型转换

```cpp
double d = 3.14;
int x = static_cast<int>(d);
```

结果是：

```cpp
x == 3
```

小数部分会被截断。

再比如：

```cpp
int a = 10;
double b = static_cast<double>(a);
```

这是非常普通的数值类型转换。

---

### 2.2 避免整数除法

```cpp
int a = 3;
int b = 2;

double c = a / b;
```

这里 `a / b` 会先进行整数除法，所以结果是：

```cpp
1
```

正确写法是：

```cpp
double c = static_cast<double>(a) / b;
```

结果是：

```cpp
1.5
```

这也是 `static_cast` 在项目代码和算法题中很常见的用法。

---

### 2.3 继承体系中的转换

假设有如下继承关系：

```cpp
class Base {};
class Derived : public Base {};
```

#### 子类转父类：安全

```cpp
Derived d;
Base *pb = static_cast<Base *>(&d);
```

这是安全的。因为 `Derived` 对象中一定包含一个 `Base` 子对象。

这种转换叫做向上转型，也叫 upcast。

实际上，这种情况下通常不需要显式写 `static_cast`：

```cpp
Base *pb = &d;
```

编译器会自动完成。

---

#### 父类转子类：可能危险

```cpp
Base *pb = new Derived;
Derived*pd = static_cast<Derived \*>(pb);

````

这段代码在逻辑上是正确的，因为 `pb` 实际指向的是 `Derived` 对象。

但如果是这样：

```cpp
Base *pb = new Base;
Derived *pd = static_cast<Derived *>(pb);
````

编译器可能允许，但这是危险的。因为 `pb` 实际指向的不是 `Derived` 对象，后续通过 `pd` 访问派生类成员会导致未定义行为。

因此要记住：

> `static_cast` 做向下转型时，不会进行运行时类型检查。

如果你不确定父类指针实际指向什么类型，应该使用 `dynamic_cast`。

---

### 2.4 void\* 转具体类型指针

```cpp
int x = 10;
void *p = &x;

int *ip = static_cast<int *>(p);
```

这是允许的。

但前提是：

> 你必须保证 `p` 原来确实指向一个 `int` 对象。

如果它原来指向的是 `double`，却被你转成 `int*`，那访问结果就是错误的。

---

### 2.5 static_cast 总结

`static_cast` 适合普通、明确、编译期可判断的转换。

{% note info %}
`static_cast` 主要用于普通类型转换，例如基本类型转换、子类到父类转换、`void*` 和具体指针之间的转换等。它是编译期转换，不做运行时类型检查，所以在父类向子类转换时需要程序员保证对象真实类型正确。
{% endnote %}

---
