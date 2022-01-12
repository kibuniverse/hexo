# TypeScript 逆变和协变

> 文章中所有的代码均在这里 [TS Playground - An online editor for exploring TypeScript and JavaScript](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgIImAWzgG2QbwChkTkQ5MIAuZAZzClAHNCBfQw0SWRFAEQD2TZBAAekEABNaaDNjxFSyAPTLkgcRNAfDqAX3WKkA7nCZNmAFTjAcACgCUNAG4Dgkthy7R4SZAGEBUEyLiEFIygsKKpAhwIADqRiYgTDTwOLQQroQ4EGDI0Vi4NOj5eAC8BHok5JQ0AOR58jUANGwA3JnZyJJCNGHIZRGVFNTINWACmE0VyIbGZhbWNgTICAIgtAJZAHQ4QlY1MwlM5pYAhDWLrM2sbYT1uH2dQhxgAJ4ADigAYiAAjA9WcH8PSEixKAD5kGFnu8viAAEz-QFJWTFUEQ3z+YAcLI5GC-GjfH5tHHIPFwgnwm6EPF-MpkoA)

## 一些概念

### Subtyping 子类型

在编程语言理论中，子类型是一种类型多态的形式，这种类型可以替换其超类型(supertype)。如果 S 是 T 的子类型，一般表示为 `S <: T`，意思是在任何类型为 T 的地方都可以安全的使用 S。

```
type T = number
type S = T | string

type Fn = (arg: T) => void

let fn: Fn = (arg: S) => {}
```

上面的代码我们定义了 T 和 S 类型，然后定义了一个 参数为 T 返回值为 any 的函数类型 Fn， 但是在具体的赋值的时候我们将 args 变为 S，程序是可以正常运行的。

### 协变与逆变

> **协变与逆变**（Covariance and contravariance）是在计算机科学中，描述具有父/子型别关系的多个型别通过型别构造器、构造出的多个复杂型别之间是否有父/子型别关系的用语。

> 在一门程序设计语言的[类型系统](https://zh.wikipedia.org/wiki/型別系統)中，一个类型规则或者类型构造器是：

-   > **协变**（covariant），如果它保持了[子类型序关系≦](https://zh.wikipedia.org/wiki/子型別)。该序关系是：子类型≦基类型。
    

-   > **逆变**（contravariant），如果它逆转了子类型序关系。
    

-   > **不变**（invariant），如果上述两种均不适用。
    

维基百科中的描述有点抽象，我们提取一下关键词

-   父/子型别
-   构造器构造
-   是否具有父/子型别关系

我们先用简单的数学模拟一下，假设我们现在有两个值 `x = -1` ， `y = -5`，x 和 y 的关系为 x > y。

假设现在有函数

$$f(t) = t + 10$$

将我们的 x 和 y 代入 f(t) 后得到 f(x) = 9 和 f(y) = 5 ，可以发现 f(x) 和 f(y) 的关系仍然为 f(x) > f(y)，这就是典型的协变。

假设如下函数

$$g(t) = t^2$$

g(x) = 1 < g(y) = 5 这就是逆变。

上面只是举了一个很简单并且不够严谨的例子，主要是为了方便理解，现在我们看一下在 ts 中的实现。

## TypeScript实现

考虑如下类型

```
dinterface Animal {
    name: string
}

interface Dog extends Animal {
    // 摇尾巴
    waggingTail(): void
}

interface Corgi extends Dog {
    canWagging: false
}
```

代码中我们定义了三个类型，他们之间的子集关系为 `Corgi <: Dog <: Animal`。

```
// 协变
type AnimalList = Array<Animal>
type DogList = Array<Dog>

let animalArr: AnimalList = [];
let dogArr: DogList = [];
// ✅ DogList 为 AnimalList 子集
animalArr = dogArr;
```

我们再考虑一种特殊情况

```
type Fn1 = (arg: Dog) => Dog
```

Fn1 类型的子类型是什么呢？

是 `(arg: Corgi) => Corgi`吗？

考虑如下场景

![](https://sh1twn2kpn.feishu.cn/space/api/box/stream/download/asynccode/?code=NDgyM2FmNjNiOWYxOGY1Y2Y0YWIxOTRlNTdlYmE5YmNfVXE5Z2ppNWt2dGw2aW1tQ0IyVnBKVE9BVGxRTkpPWVRfVG9rZW46Ym94Y25qMU5Xb1Jib3BKYUVDdG1wcURoZkdoXzE2NDE5NTk4NjE6MTY0MTk2MzQ2MV9WNA)

显然 Fn3 并不是 Fn1 的子类型。

我们分析一下这样为什么不被允许，由于 Fn3 类型的限制，这里 fn3 函数的参数只能传入 arg 为 Corgi 类型的函数，但是本来的 Fn1 是允许任何 Dog 类型的参数，所以 ts 禁止了这样的操作。

所以 Fn1 的子类型应该为参数可以接受任何类型的 Dog，返回值具有 Dog 所有值的类型。

所以正确答案是 `(arg: Animal) => Corgi` 。

![](https://sh1twn2kpn.feishu.cn/space/api/box/stream/download/asynccode/?code=OGRmNjI5MzhjZDliMDc0ZWJiYjFkM2M1ZTkyYWEwMzlfVEpNV1JUaTluWTFNcWhUMlEwWUd1dnI4S3NtV21kU1lfVG9rZW46Ym94Y25LZWlXeDZjcXZ1S0VlQW5WbHhaZlViXzE2NDE5NTk4NjE6MTY0MTk2MzQ2MV9WNA)

参考

-   [维基百科-子类型](https://zh.wikipedia.org/wiki/%E5%AD%90%E7%B1%BB%E5%9E%8B)
-   [维基百科-协变与逆变](https://zh.wikipedia.org/wiki/%E5%8D%8F%E5%8F%98%E4%B8%8E%E9%80%86%E5%8F%98#%E2%80%9C%E5%8D%8F%E5%8F%98%E2%80%9D%E4%B8%80%E8%AF%8D%E7%9A%84%E6%9D%A5%E6%BA%90)
-   [What are covariance and contravariance? | Stephan Boyer](https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)