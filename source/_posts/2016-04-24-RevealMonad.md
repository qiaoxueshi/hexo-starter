---
title: 揭开 Monad 的神秘面纱
date: 2016-04-24 03:29:09
tags: [iOS, Monad]
---

我们知道 Swift 语言支持函数式编程范式，所以函数式编程的一些概念近来比较火。有一些相对于OOP来说不太一样的概念，比如 `Applicative`, `Functor` 以及今天的主题 `Monad`.  如果单纯的从字面上来看，很神秘，完全不知道其含义。中文翻译叫做`单子`，但是翻译过来之后对于这个词的理解并没有起到任何帮助。

我的理解很简单，`Functor`是实现了`map`函数的容器，`Monad` 就是实现了 `flatMap` 方法的容器，比如在Swift里，`Optional`, `CollectionType` 等等都可以称为 Monad。

既然有了 map, flatMap 又有什么作用呢？两者有什么联系和区别呢？

###  map vs flatMap
map 和 flatMap 的共同点都是接受一个 transform 函数，把一个容器转换为另外一个容器。

下面主要从`维度`这一块来解释两者的区别，我们先来简单的定义一下`维度`：
> 对于类型T，如果类型S是一个容器，且元素的类型包含T，那我们就说：  
S(维度) = T(维度) ＋ 1

举个🌰， [Int] (维度) = Int (维度) +1, Int? (维度) = Int(维度) + 1.

map 和 flatMap的区别是，对于map，容器里的一个元素经过transform后只产生一个元素，是 one-to-one的关系，也就是说经过转换后，纬度是不变的。比如:

```Swift
var intArray: [Int] = [1, 2, 3, 4, 5]
var stringArray: [Int] = intArray.map { (value: Int) -> String in
    return "\(value)"
}
//stringArray: ["1", "2", "3", "4", "5"]
```
这个 transform 函数的是 `Int -> Int` 的，两边的维度是一致的。

对于flatmap，容器里的一个元素经过transform可能转换成 0个，1个 或者多个元素，也就是 one-to-any 的关系，既然是 any 的关系，就需要一个容器来存放any个元素，所以经过transform的返回值通常是一个容器，所以 transform 函数执行之后，相当于维度＋1。

```Swift
var oddIntArray: [Int] = intArray.flatMap { (value: Int) -> Int? in
    return value % 2 == 1 ? value : nil
}
//oddIntArray: [1, 3, 5]
```
这里的 transform 是 `Int -> Int?` 的，我们知道 Int? 是 Int 的包装类型，所以说 transform 相当于对每个元素都包了一层，提升了一个维度.

但是我们看一下上面例子里stringArray和oddIntArray的类型，都是[Int]，也就是说flatMap函数对 transform函数的返回值做了降维处理。那么flat的意思在这里也就知道了，就是把transform 返回的容器`降维攻击`(拍扁)，拿出里面的元素。

flatMap 函数为什么要这么做呢？在函数式编程中，通常会对一个值/操作进行链式操作，为了保证后面还可以继续方便的进行链式操作，一般需要保持维度不变。其实可以看作一个约定，大家都遵循一定的规则，才都有得玩。

### 如何确定使用 map or flatMap 的时机？
从上面可以看到 map 对 transform 的返回值没有做特殊的处理，flatMap对于transform的返回值会做降维处理，比如 unwrap optional 值等。

其实可以反推，如果给定的transform函数会对调用者容器里的每个元素做升维，那我们需要用 flatMap 对它的结果进行降维，来保证调用flatMap前后维度保持一致。如果说transform调用前后维度没有变化，使用map方法就行了。

### Swift 中的 map 和 flatMap 方法
首先看看 Optional<Wrapped> 的 map 和 flatMap 方法：

```Swift
    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
    @warn_unused_result
    public func map<U>(@noescape f: (Wrapped) throws -> U) rethrows -> U?
    /// Returns `nil` if `self` is `nil`, `f(self!)` otherwise.
    @warn_unused_result
    public func flatMap<U>(@noescape f: (Wrapped) throws -> U?) rethrows -> U?
```

map 的 transform 是 `Wrapped -> U` 维度不变, flatMap 的 transform 方法是 `Wrapped -> U?`，维度+1。因为 Optional 的特殊性，flatMap 提供了 one-to-zero/one 的关系。

继续看看 CollectionType：

```
    public func map<T>(@noescape transform: (Self.Generator.Element) throws -> T) rethrows -> [T]

    public func flatMap<T>(@noescape transform: (Self.Generator.Element) throws -> T?) rethrows -> [T]

    public func flatMap<S : SequenceType>(transform: (Self.Generator.Element) throws -> S) rethrows -> [S.Generator.Element]
```
有一个map函数和两个flatMap, map 的 transform 函数是 `Element -> T` 维度不变，两个 flatMap 的 transform 函数分别是 `Element -> T? `(one-to-zero/one) 和 `Element -> S: SequenceType`, SequenceType 是个集合，相当于 one-to-any，这两个transform维度都升了一级。

> 特别感谢我的同事 [王轲](http://weibo.com/u/2010406980), 本文的很多思路都得益于和他的讨论。


