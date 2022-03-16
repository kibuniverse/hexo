---
title: rust trait 生命周期记录
date: 2022-03-03
tag: rust trait
---

> rust 的生命周期一直是 rust 中比较有意思的一个东西，本片文章主要讨论我对于 rust 生命周期的理解。



## 所有权

在讨论生命周期之前我想先从所有权说起，rust 的独特之处就在于其对 gc 的处理，rust 通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查，从而决定是否将引用的堆内存进行 gc，所有权有一些基本的规则。

1. rust 中的每一个值都有一个被称为其 **所有者**（*owner*）的变量。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃。



### 引用

由于所有权的机制，当我们需要在一个函数中对一个变量进行操作但是并不需要该变量的所有权的时候，我们需要使用引用(&)。

```rust
fn get_longest_str(str1: &str, str2: &str) -> str {
  if str1.len() > str2.len() {
        str1
  } else {
        str2
  }
}
fn main() {
  let str1 = String::from("Hello World!");
  let str2 = "ddd";
  let result = get_longest_str(str1.as_str(), str2);
}
```



这个时候就会出现一个问题，我们的 result 是对于 str1 or str2 的一个引用，但是 rust 编译器在回收值的时候并不会判断当前值的引用，所有就会出现可能 str1 or str2 已经被回收了，但是我们的 result 还需要使用，这个时候就会出现错误（这种情况一般被称为悬垂指针），为了避免上述问题的发生，所以 rust 编译器在编译的时候会帮我们检查这些问题，下面看一个更清晰的例子

> 来自 rust the book 官网

```rust
{
    let r;
    {
        let x = 5;
        r = &x;
    }
    println!("r: {}", r);
}
```

这个例子就很清晰的说明了上面可能会发生的情况。

```rust
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
} 
```

我们的变量 r 的生命周期为 `'a`，x 的生命周期为 `'b`，很明显 r 在 x 生命周期结束后还在尝试 println，所以上面的代码是无法通过编译的，换句话说就是在 rust 中 **数据需要比引用有着更长的生命周期**。



我们继续回到上面的例子

```rust
fn get_longest_str(str1: &str, str2: &str) -> &str {
  if str1.len() > str2.len() {
        str1
  } else {
        str2
  }
}
```

针对于这个 get_longest_str 函数来说，我们返回的是一个引用，rust 如何保证我们返回的引用不会出现 *悬垂指针 *的情况呢，答案就是传入生命周期参数。



## 生命周期

悬垂指针的问题

