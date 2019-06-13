---
title: learning-haskell
tags: [haskell]
date: 2019-4-30日 17:23
---

## zip

学习了 Haskell 的 zip 实现，按照 Javascript 的语法重新写了一版

Haskell:

```Haskell
zip' :: [a] -> [b] -> [(a, b)]
zip' _        []       = []
zip' []       _        = []
zip' (x : xs) (z : ys) = (x, z) : zip' xs ys
```

Javascript:

```javascript
const fn1 = arr => {
  if (!arr) {
    return []
  }

  if (arr.length === 0) {
    return []
  }

  return [arr[0], arr.slice(1, arr.length)]
}

const zip = (xs, ys) => {
  const [x1, xs2] = fn1(xs)
  const [y1, ys2] = fn1(ys)

  if (!x1) {
    return []
  }

  if (!y1) {
    return []
  }

  return [[x1, y1]].concat(zip(xs2, ys2))
}

const result = zip([1, 2, 3], [2, 3, 4])

result // [ [ 3, 4 ], [ 2, 3 ], [ 1, 2 ] ]
```

来点不一样的函数：

```haskell
applyTwice :: (a -> a) -> a -> a

applyTwice f x = f (f x)
```

传入一个 `f` 和 `x` 并将 `x` 作为 `f` 的参数，然后将 `f` 函数执行两次

applyTwice 的签名可以这样理解，接受 `f` 参数，接受 `x` 参数，最后返回一个 a，那么总共可以拆成三部分

- 接受 (a -> a) 的第一个参数
- 接受 a 的第二个参数
- 返回 a

```
(a -> a) -> a -> a
--------    -    -
    f       x    a
```

## zipWith

和 `zip` 函数类似，区别在于将两个 List 交叉到一起的重任交给了一个函数处理

```haskell
zipWith' :: (a -> b -> c) -> [a] -> [b] -> [c]
```

- 第一个签名 `(a -> b -> c)` 实际为执行 `f a b` 得到 c
- 继续接受 `[a]` 和 `[b]` 两个 List
- 最后返回 [c]

```haskell
zipWith' _ []       _        = []
zipWith' _ _        []       = []
zipWith' f (x : xs) (y : ys) = f x y : zipWith' f xs ys
```

让我们用 Javascript 实现一次

```javascript
const zipWith = (f, a, b) => {
  const [x, xs] = fn1(a)
  const [y, ys] = fn1(b)

  if (!x) {
    return []
  }

  if (!y) {
    return []
  }

  return [f(x, y)].concat(zipWith(f, xs, ys))
}

const add = (a, b) => a + b

const result = zipWith(add, [1, 2, 3], [2, 3, 4])

result // [3, 5, 7]
```

递归真奇妙！

```haskell
zipWith' (zipWith' (*)) [[1,2,3], [4,5,6]] [[1,2,3], [4,5,6]]
```

看到 Haskell 上这样用了一下第一眼是懵的，但观察一下其实非常简单，首先我们需要对 Curried functions 有了解，让我们用 Javascript 来进行同样的调用

```javascript
const multiply = (a, b) => a * b

const zipWith2 = f => (a, b) => zipWith(f, a, b)

const result = zipWith(zipWith2(multiply), [[1, 2, 3], [1, 2]], [[1, 2, 3], [1, 2]])

result // [ [ 1, 4, 9 ], [ 1, 4 ] ]
```

之前定义的 zipWith 方法并不是柯里化的无法部分调用，因为在这里我们需要先将 `f` 传入稍后再传入 `[a]` `[b]`，所以再改造一下声明 `zipWith2` 方法。

看完 Javascript 版本的 Haskell 是不是瞬间明白了。

Note:

一个类型为 `a -> b -> c` 的函数可以传入 `a -> a -> c` 类型给他， 但反过来就不行。

## flip

`flip` 函数在前端工作中出现频率也挺高的，下面是经常会用到的一种方式。

```javascript
const ids = [20, 30, 40]
const idExist = R.flip(R.includes)(ids)

idExist(10) // false
idExist(20) // true
```

```haskell
flip' :: (a -> b -> c) -> b -> a -> c
flip' f a b = f b a

flip'' :: (a -> b -> c) -> (b -> a -> c)
flip'' f = g
  where g x y = f y x
```

第一种是自己悟出来的 😄 但第二种更适合去思考，让我们来看下这个简单函数的签名

`(a -> b -> c)` 和 `(b -> a -> c)` 对应的是我们接受一个函数，返回一个函数，只是单纯的 `a` `b` 位置互换

`flip f = g`， `f` 就是 `(a -> b -> c)` g 则是 `(b -> a -> c)`，接着将 `g` 函数定义， `g x y = f y x` 一切似乎理所当然了

回来看第一种，首先将 `(b -> a -> c)` 签名部分去掉括号，因为 Haskell 是默认柯里化的，应该可以这样理解吧，传与不传都岁月静好。

```haskell
addTen = (flip' (+)) 10
-- addTen 20 // 30
```

## Collatz 串行

Haskell 书中介绍了 Collatz 串行 的实现方式，简单直接。

```haskell
chain' :: (Integral a) => a -> [a]
chain' 1 = [1]
chain' x | even x = x : chain' (x `div` 2)
         | odd x  = x : chain' (x * 3 + 1)
```

得到 1-100 之前所有数的 chain 长度大于 15 的

```haskell
numLongChains :: Int
numLongChains = length (filter isLong (map chain' [1 .. 100]))
  where isLong x = length x > 15
```

让我们使用 Javascript 实现同样的功能

```javascript
const chain = n => {
  if (n === 1) {
    return [1]
  }

  if (isEven(n)) {
    return [n].concat(chain(n * 3 + 1))
  }

  if (isOdd(n)) {
    return [n].concat(chain(n / 2))
  }
}

const numLongChains = () => {
  const isLong = x => x.length > 15
  const arr = new Array(100).fill(undefined).map((_, index) => index + 1)

  return arr.map(chain).filter(isLong).length
}
```

[^collatz 串行]: 取一个自然数，若为偶数就除以 2。 若为奇数就乘以 3 再加 1。再用相同的方式处理所得的结果，得到一组数字构成的链 ˘。

## Fold

Haskell 中 `foldl` 函数的签名 `foldl :: Foldable t => (b -> a -> b) -> b -> t a -> b`
Javascript 中 `reduce` 函数的签名 `reduce :: ((a, b) → a) → a → [b] → a` 是不是一个东西呢？看上去似乎是一个东西。

一个简单的 sum 函数

```javascript
;[1, 2, 3].reduce((acc, x) => acc + x)
```

真是一样的呀！

```haskell
foldl (\acc x -> acc + x) 0 [1,2,3]
--    (b    a -> b)       b t a
```

Google 了一下他俩的区别大概是 reduce 初始值是可选的， fold 需要提供一个初始值。

# TODO
