---
layout:     post
title:      "创建一个长度为100的数组，且每个元素的值等于它的下标"
subtitle:   "优雅的创建密集数组"
date:       2017-07-17 20:22:00
author:     "Zi Ning"
header-img: "img/js.jpeg"
tags:
    - JavaScript
    - Array
    - ES2015
    - ES6
---
在写代码的时候碰到了一个类似的问题，要自动生成一个按顺序排列的数组。用map的方式做了下，发现竟然返回的是空数组！上网搜了一下，原来是稀疏数组的问题。。。而且在知乎的一个[问题](https://www.zhihu.com/question/41493194)里，也讨论了这题的许多种解法，特地来总结了一下。

## 循环

最简单的办法肯定就是循环了（打表不算，逃。

```javascript
var arr = new Array(100)
for (var i = 0, len = arr.length; i < len; i++) {
  arr[i] = i
}
console.log(arr)  // (100)[0, 1, 2, ..., 99]
```

委婉一点我们可以用`map`方法：

```javascript
var arr = new Array(100)
var newArr = arr.map(function(value, index) {
  return index
})
console.log(newArr)  // (100)[undefined * 100]
```

好了，问题来了。看起来代码似乎没什么问题啊，然而并没有如同第一段代码那样返回一个我们需要的数组，反而返回的数组中每一项都是`undefined`值？

看到高票回答上说是因为这样生成的是稀疏数组。吓得我赶紧查了一下什么是稀疏数组。

### 稀疏数组（ Sparse arrays）

> A sparse array is one in which the elements do not have contiguous indexes starting at 0.
>
> 稀疏数组是包含不规律索引的数组。

在C/C++/java中，数组是一片连续的存储空间，有固定的长度，中间没有空隙。而在JavaScript中，数组通常是稀疏的，中间会又空隙。当数组中有未被赋值的项或有被删除的项时，就可以认定该数组为稀疏数组了。

如何产生稀疏数组：

```javascript
var arr = new Array()
arr[0] = 0
arr[99] = 99
console.log(arr[1], arr[50]) // undefined, undefined
```

```javascript
var arr = new Array(100)
console.log(arr[0])  // undefined
```

```javascript
var arr = [,,,]
console.log(arr[0])  // undefined
```

#### 注意：

当使用数组方法遍历这些数组时，往往会跳过这些空隙:)。这也是为什么上面使用map遍历会出错的原因。

### 密集数组（Dense arrays）

作为解决方法，我们只要产生密集数组就好啦。

```javascript
var denseArr = Array.apply(null, Array(100))    // 兼容旧浏览器
var denseArr = Array.apply(null, {length: 100}) // ES5
var denseArr = Array.from({length: 100})        // ES6
var denseArr = Array(100).fill('naive')         // ES6
var denseArr = [...Array(100)]                  // ES6
```

第一张方法实际上等于

```javascript
Array(undefined, undefined, ..., undefined)    
```

当我们使用这个方法来遍历

```javascript
var arr = Array.apply(null, Array(100))
var newArr = arr.map(function(value, index) {
  return index
})
console.log(newArr)  // (100)[0, 1, 2, ..., 99]
```

你会发现，结果变成了我们所期待的。而且，只要把第一行换成上面产生密集数组的任意一种你喜欢的方法，结果都不会改变！只是要注意，ES6的方法要考虑浏览器的兼容，因为并不是所有的浏览器都支持ES6的语法，保险起见可以用`babel`来编译下。

ES6数组的新方法`Array.from()`，提供了第二个函数参数，可以用来在每个数组的元素上使用并返回新的数组。

这样就有了一个更简洁的方法了，至于要一行！

```javascript
var arr = Array.from({length: 100}, (v, i) => i)   // (100)[0, 1, 2, ..., 99]
```

### 注意：

在实际使用时，最好将其封装成函数，这样可以提高可读性。例如`range()`...

## 如果不使用循环方法呢

我们可以配合`Object.keys`和`Array.keys()`方法,，这样连map都可以省去。

ES5:

```javascript
Object.keys(Array.apply(null, {length: 100}))
```

ES6:

```javascript
Array.from(Array(100).keys())
[...Array(100).keys()]
```

### 注意

`Array.keys()`是继承了`Object.keys()`的方法，因为在JavaScript中，数组实际上也是对象，也是由键值所对应，同样也会处于对象的原型链中的一部分。所以，`Array.keys()`返回的是数组索引所构成的新数组。

同样我们也可以使用递归和迭代（尾递归）的方式。

## 递归

用递归的方式就好玩许多了。

```javascript
function range(i) {
  if (i < 0) {
    return []
  }
  return range(i-1).concat(i)
}
range(99)  // (100)[0, 1, 2, ..., 99]
```

## 迭代（尾递归）

更推荐使用尾递归的方式，因为当递归太深的时后，往往会出现堆栈溢出的错误，而且影响性能和内存。

```javascript
function range(prev, cur) {
  if (cur >= 100) {
    return prev
  }
  return range(prev.concat(cur), cur + 1)
  
  // 或者
  prev.push(cur)
  return range(prev, cur + 1)
}
range([], 0) 
```

## Generator

Generator是ES6的新语法。它就像一个惰性求值器，可以通过`yield`来暂停函数的执行，但只有通过显式的`next`方法，才会执行下一步。我觉得这个比较像python里的`iterator`（据说是借鉴了它）

```javascript
function* range(i) {
  yield i
  if (i < 99) {
    yeild* range(i + 1)
  }
}
Array.from(range(0))
```

注意到，这里和其他函数的不同是，在`function`紧跟着一个`*`号，用来区别这是个Generator。

当然还有些奇技淫巧，像`Y combinator`、`setInterval`之类的，有些我也没怎么看懂╥﹏╥，就不写下来了，需要的去那个回答看好了。

## 参考资料

* [如何不使用loop循环，创建一个长度为100的数组，并且每个元素的值等于它的下标？](https://www.zhihu.com/question/41493194)
* [JavaScript中的稀疏数组与密集数组](http://www.cnblogs.com/ziyunfei/archive/2012/09/16/2687165.html)

