---
title: 使用 ts 类型系统实现斐波那契
tag: TypeScript 
---


> ts 的类型系统本身非常强大和完善，并且也是图灵完备的，所以理论上 ts 的类型编程可以实现任何可计算的 stuff， 本篇文章主要的使用 ts 类型系统来实现斐波那契数列。

## 斐波那契数列

斐波那契数列定义并不复杂，它是一个通项为
$$an = 
\begin{cases}
n <= 2, & 1 \\
n > 2, & a_n = a_{n-1} + a_{n-2} \end{cases}$$
的数列，我们简单列出前 6 项。
$$\{1, 1, 2, 3, 5, 8 \}$$
## 正常实现
### JavaScript
```js
const fibonacci = n => {
	if (n <= 2) {
		return 1
	}
	return fibonacci(n - 1) + fibonacci(n - 2)
}
```
### Rust
```rust
fn fibonacci(n: u32) -> u32 {
	match n {
		1 => 1,
		2 => 1,
		_ => fibonacci(n - 1) + fibonacci(n - 2)
	}
}
```

## ts 类型系统实现
### 构造自然数
由于 ts 类型系统没有自然数，无法直接进行加减乘除的操作，即下面这样的操作是不被允许的。
```ts
type one = 1
type two = 2

type third = one + two ❌
```

所以首先我们要做的就是构造自然数，以满足加减乘除的操作，在这里我们使用数组类型属性 `length`，由于 `length` 可以获取数组类型的长度，所以我们可以使用对不同长度的数组进行操作从而实现模拟自然数，下面那我们简单模拟几个。
```ts
type one = [unknown]["length"]
type two = [unknown, unknown]["length"]

```
为了好看起见，我们简单把获取数组长度的操作封装一下。
```ts
type Length<Array extends unknown[] = []> = Array["length"]
```
现在上面的代码就可以写成这样：
```ts
type one = Length<[unknown]>
type two = Length<[unknown, unknown]>
```

### 通过数字生成数组
因为我们的运算都是通过数组的长度组合来实现的，所以我们还需要根据数字来生成数组从而进行实际的运算。
```ts
type StructureArr<N extends number = 0, ANS extends unknown[] = []> = 
	Length<ANS> extends N 
	? ANS
	: StructureArr<N, [...ANS, any]>
```

到这里代码可能就有一点复杂了，这里我们定义了一个 `StructureArr` 的类型，这个类型接受两个泛型参数，返回传入的第一个泛型参数长度的 `unknown` 类型数组。
这里解释一下，首先是 `extends` 关键词，这个关键词在 ts 中除了接口继承外还可以用来代替其他语言中的 `==` 操作符，再搭配三目表达式就实现了基本的控制流逻辑，我们把上面的代码用 `JavaScript` 翻译一下就是：
```js
function StructureArr(n, ans) {
	if (ans.length == n) {
		return ans
	}
	return StructureArr(n, [...ans, 'unknown'])
}
```
同时我们也可以使用多个三目表达式嵌套来实现更为复杂的逻辑，这里我们用不上，就不写了～，有兴趣的同学可以看看这个 [TypeScript 类型系统实现中国象棋](https://zhuanlan.zhihu.com/p/426966480)，实在是棒！这里就不过多讨论，后面有时间的话考虑把其中一些难点或者设计独到之处拿出来剖析一下😂


到这里我们就是就可以构造自己的自然数了。
```ts
type one = Length<StructureArr<1>>
type two = Length<StructureArr<2>>
```
我们可以再简单封装一下。
```ts
type TypeNumber<N extends number = 0> = Length<StructureArr<N>>
type one = TypeNumber<1>
type two = TypeNumber<2>
```

### 实现所需运算操作
由于 ts 类型系统的运算操作相对比较复杂，所以我们本篇文章只实现所需的运算操作，通过上文的代码我们可以发现在实现 fibonacci 数列需要的操作有
 - 等于
 - 加法
 - 减法

#### 加法
因为我们的数字是由数组长度来进行表示和计算的，所以在实现加法的时候我们需要将数组的长度拼接实现。
```ts
type ConCatArr<Arr1 extends unknown[], Arr2 extends unknown[]> = [...Arr1, ...Arr2]
```

所以我们实现加法的方法：
1. 构造两个加数长度的数组
2. 将两个数组进行拼接
3. 计算拼接后数组的长度即位答案

```ts
type Add<A extends number = 0, B extends number = 0> = Length<ConcatArr<StructureArr<A>, StructureArr<B>>>
```

#### 减法
减法相对于加法来说稍微复杂一点，关键点在于数组可以直接进行拼接但无法进行减法，所以我们需要用到一个新的关键词 `infer` ，这个关键词最早出现在[此pr中](https://github.com/Microsoft/TypeScript/pull/21496)，它可以用来进行**推断赋值**，像下面这样
```ts
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number
```
它不但适用对象的类型推断，并且可以推断数组，所以我们数组的减法可以写成这样 
```ts
type MinusArr<A extends unknown[] = [], B extends unknown[] = []> = A extends [B, ...infer U] ? U : unknown[]
```
同时，数字的减法也实现了
```ts
type MinusNumber<A extends number = 0, B extends number = 0> = Length<MinusArr<StructureArr<A>, StructureArr<B>>>
```

### 实现 fibonacci 数列
我们先用简单的 JavaScript 代码实现一下
```js
function fibonacci(n) {
	if (n == 1) {
		return 1
	}
	if (n == 2) {
		return 2
	}
	return fibonacci(n - 1) + fibonacci(n - 2)
}
```
我们用 `extends` 模拟 `==` ，加法和减法我们已经实现，所以最终的代码是这样的
```ts
type Fibonacci<N extends number = 0> = N extends 1
	? 1
	: N extends 2
	? 1 : AddNumber<Fibonacci<MinusNumber<N, one>>, Fibonacci<MinusNumber<N, two>>>
```
因为是使用递归实现的，这里 ts 会有提示，2589，可能递归层数过于深。

到这里我们就算是成功了，本文是一个 ts 类型系统的简单的入门题，将简单的关键词组合出复杂的能力是很有意思的事情，下面给出几片好文推荐，感兴趣的可以看看～，👋

这是[代码的地址](https://github.com/kibuniverse/ts-type-exer/blob/master/fibonacci.ts)，要是有更好的思路或者更优雅的代码欢迎提 pr 呦～

## TypeScript 类型系统好文推荐
- https://zhuanlan.zhihu.com/p/64446259
- https://zhuanlan.zhihu.com/p/426966480