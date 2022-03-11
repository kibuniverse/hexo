-----

title: trait
tag: rust
date: 2022-03-03

------

# trait

## 一、什么是 trait

trait 是对于未知类型(Self)抽象的一系列方法的集合。

在 trait 定义后我们可以为任意类型实现 trait，包含基础类型。

```rust
trait MyTrait {
  fn say_hello(&self) -> String {
    String::from("Hello World!")
  }
}
impl MyTrait for i32 { 
}


fn main() {
    let a: i32 = 10;
    println!("{}", a.say_hello()); // Hello World
}

```

