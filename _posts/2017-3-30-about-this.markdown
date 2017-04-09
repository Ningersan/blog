---
layout:     post
title:      "js：关于this"
subtitle:   "多变的this与不变的规则"
date:       2017-03-30 20:47:00
author:     "Zi Ning"
header-img: "img/this.jpeg"
tags:
    - JavaScript
    - js this
---

# js: 关于 this

 除了闭包，this可以说是JavaScript中比较难以理解和多变的机制之一了。即使是语法的细微变化，都可能意外改变this的值，而且在程序中还经常能够碰到形形色色的this，理解它就更是必要了。（详细请参考《你不知道的JavaScript》）

其实，理解this的关键是知道this对象是运行时基于函数的**执行**环境绑定的。与闭包不同，这里是执行环境，而不是我们熟悉的词法作用域。关于词法作用域，可以看上一篇[闭包](http://zining.me/2017/03/14/closure/)中的概念。

## 对于this的误解

在不了解this的情况下，盲目的猜测很容易出现两个误解。一是，this指向函数自身，二是，this指向函数作用域。这两点其实都不对。实际上，this的指向非常的灵活多变，但本质上还是基于函数调用时的执行环境。理解this，这就要抛开以往错误的假设和理解。

## this的实质

this是在**运行**时绑定的，而不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this的绑定和函数声明的位置没有任何关系，只是取决于函数的**调用**关系。

当一个函数被调用时，会创建一个活动记录（执行上下文）。这个记录会包含函数在哪里被调用，以及函数的调用方式、传入的参数等等信息。this就是这个记录的一个属性，会在函数的执行过程中用到。

## 调用位置

既然说了，this的绑定离不开调用关系，执行环境。那就要对调用位置有必要的理解。

调用位置：函数在代码中被调用的位置（注意不是声明的位置！）。

调用栈：为了到达执行位置所调用的所有函数，你可以把它想象成一条函数调用链。

看一个简单的例子：

```javascript
function baz() {
    // 当前的调用栈是：baz
    // 当前的调用位置是：全局作用域
    
    console.log("baz");
    bar();  // bar的调用位置
}

function bar() {
    // 当前调用栈： baz -> bar 
    // 当前调用位置：bar
    
    console.log("bar");
}

baz(); // baz的调用位置
```

tips：在复杂的函数调用中，往往很难清楚的找到函数的调用链。我们可以借助开发者工具来帮助我们找到它。通常的做法是在函数的第一行设置断点或debugger，程序会在那个位置暂停，同时展现当前位置的函数列表，据此来判断函数的调用位置。

## 绑定规则

在上面我们了解找到了调用位置。根据调用位置，我们总结出了四种this绑定的规则。根据每种情况，来判断this的真正指向。

### 默认绑定

通过名字也可以看出，这是一种最常见的函数调用类型：**独立函数调用**。换句话说，就是我们最常用的普通函数调用。默认绑定中，this指向全局对象。

可以把这条规则看作是无法应用其他规则时的默认规则。

```javascript
var a = 2;
function foo() {
    console.log(this.a);
}
foo();  // 2
```

根据规则，可以看到调用foo()时，this.a被解析成了全局对象。

这里有一点例外，需要注意。即在**严格模式**下调用foo()，this的指向**不会**指向全局对象，而是会出现undefined。

### 隐式绑定

隐式绑定要考虑调用位置是否有上下文对象，或者说是否被某个对象拥有或包含。通俗来说，就是作为函数的方法被调用。在这里this会绑定到上下文对象上。什么是上下文对象呢，看例子就知道了。

```javascript
function foo() {
    console.log(this.a);
}

var obj = {
    a: 2,
    foo: foo
};

obj.foo();  // 2
```

在这里，`foo()`先在对象外声明，再被添加为`obj`对象的引用属性。调用时，`foo()`的上下文实际就是`obj`对象。因此，this就被绑定到了`obj`上。

复杂一点，假如引用链上有多个多对象的话，如`obj1.obj2.foo()`，也只是会在最后一层的调用位置中起作用而已。`foo()`的上下文是`obj2`，this也就指向它。

这里也会发生一个例外，隐式绑定会出现**丢失绑定对象**的场景，也就是说它会应用默认绑定，从而把this绑定到全局对象或者undefined（严格模式）上。

```javascript
var a = "global";

function foo() {
    console.log(this.a);
}

var obj = {
    a: 2,
    foo: foo
};

var bar = obj.foo;  // 声明变量，引用函数指针
bar();  // "global"
```

在这里，使用了`obj.foo()`，看似是一个隐式绑定，但实际上结果恰恰相反，this应用了默认绑定。

分析一下，我们使用了一个变量`bar`来记录函数指针（`obj.foo`），最终调用`bar()`，此时bar()其实是一个不带任何修饰函数调用，因此应用了默认绑定。

在使用回调函数的时候，丢失this绑定的现象也非常常见。例如：`setTimeout(obj.foo, 1000);`this的值也会被修改，指向全局对象。

所以，在使用隐式绑定的时候要格外注意this丢失的意外！！！如果违背我们的预期，要及时的修正this的指向。常见的方法是使用`self/that`之类的变量记录我们需要的this（未被修改过），然后后面使用它们来完成剩余操作。还有一种做法就是接下来要说的显示绑定，通过apply、call、bind显示绑定this值。

### 显式绑定

回想上面的场景，如果我们想在某个对象上强制调用函数，该怎么做？

ECAMScript3给Function原型定义了两个方法，分别是call和apply，js提供的绝大多数函数和我们自己创建的所有函数都可以使用它们。我们先来了解两个JavaScript函数通用方法：

#### apply()

apply函数接受两个参数，第一个参数指定了函数体内this对象的指向，第二个参数为一个带下标的集合，这个集合可以是数组，也可以类数组。apply方法把这个集合中的元素作为参数传递给被调用的函数。

#### call()

call函数传入的参数不固定，与apply相同第一个制定函数体内this对象的指向，从第二个参数开始以后，每个参数被依次传入函数。

可见，这两种方法作用是一模一样的，**区别**仅仅在于传入参数的形式不同。

知道了用法，我们就可以解决开始的问题了。

在函数调用时，我们可以使用apply、call方法来将函数内部的this绑定到我们制定的对象上去。我们称之为显示绑定。

```javascript
function foo() {
    console.log(this.a);
}

var obj = {
    a: 2
};

foo.call(obj);  // 2
```

通过call(...)，我们在调用`foo()`时强制把它的this绑定到了obj上。

然而，即使这样，依然无法解决我们之前提出的丢失绑定问题。显示绑定的另一个变**硬绑定**(bind)可以解决这个问题。

```javascript
function foo() {
    console.log(this.a);
}

var obj = {
    a: 2
}

var bar = function() {
    foo.call(obj);
};

bar();  // 2
setTimeout(bar, 1000);  // 2

//硬绑定的bar不能在修改它的this
bar.call(window);  // 2
```

我们创建了函数`bar()`，并在它的内部手动调用`foo.call(obj)`，因此强制把`foo`的this绑定到了`obj`上。无论之后如何调用函数bar，this值也不会丢失，因为它总会手动的在`obj`上调用`foo`。这就是硬绑定，经过硬绑定的函数就不能再修改它的this了。

再看一个例子：

```JavaScript
function foo(something) {
    console.log(this.a, something);
    return this.a + something;
}

var obj = {
    a: 2
}

var bar = function() {
    return foo.apply(obj, argument);
}

var b = bar(3);  // 2 3
console.log(b)  // 5
```

可以看出来，硬绑定就是在函数中使用显示绑定来强制绑定this的指向。

好在ES5中提供了Function.prototype.bind通用方法。bind() 方法会自动替我们创建一个新的函数，当这个新函数被调用时候，bind()的第一个参数将作为它运行时的this，之后的一序列参数将会在传递的实参前传入作为它的参数。

```javascript
function foo(something) {
    console.log(this.a, something);
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = foo.bind(obj);

var b = bar(3);  // 2 3
console.log(b);  // 5
```

即使没有bind方法，我们也可以手动实现一个简易版的bind！

```javascript
Function.prototype.bind = function(context) {
    var self = this;  // this指向调用时func.bind()的func
    return function() {
        // 执行新函数时，会把之前传入的context当做新函数体内的this
        return self.apply(context, arguments);  
    }
}
func.bind(obj);
```

在Function.prototype.bind内部实现中，我们把函数的引用保存起来，然后返回一个新的函数。当我们在将来执行`func`函数时，实际上执行的事这个刚刚返回的新函数。在新函数内部，就是显示绑定的内容了！`self.apply(context, arguments)`这句代码才是执行原来的`func`函数，并且制定content对象为`func`函数体内的this。

## new绑定

这是第四条也是最后一条this的绑定规则。

首先要澄清一个误解，在JavaScript中，构造函数只是一些使用new操作符时被调用的函数。他们并不属于某个类，也不会实例化一个类，甚至都不能说是一种特殊的函数类型，它们只是被new操作符调用的普通函数而已。

我们首先要了解一下使用new来调用函数，或者说发生构造函数调用时，发生了些什么？

1. 创建一个新的对象
2. 这个新的对象会执行[[Prototype]]链接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中函数调用会自动返回这个新对象。

```javascript
function foo(a) {
    this.a = a;
}

var bar = new foo(2);
console.log(bar.a);  // 2
```

使用new来调用`foo()`时，我们会构造一个新对象并把它绑定到`foo(...)`调用中的this上。

## 优先级

了解了上面四条规则后，难免会有一个疑问。如果某个调用位置上可以应用多个规则，那哪条规则会最终生效呢？这时候优先级的重要性就体现出来了！

优先级按照重要程度顺序排列如下：

1. 函数是否在new中调用（new绑定）？如果是的话，this绑定的是新创建的对象。
2. 函数是否通过call、apply（显示绑定）或者是硬绑定调用？如果是，this绑定的是指定的对象。
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是，this绑定的是那个上下文对象。
4. 如果都不是的话，使用默认绑定。在严格模式下，就绑定到undefined，否则绑定到全局对象。

来看一些例子，判断一下吧。

隐式绑定 vs 显式绑定

```javascript
function foo() {
    console.log(this.a);
}

var obj1 = {
    a: 2,
    foo: foo
}

var obj2 = {
    a: 3,
    foo: foo
} 

obj1.foo()  // 2
obj2.foo()  // 3

obj1.foo.call(obj2)  // 3
obj2.foo.call(obj1)  // 2
```

显式绑定win！！！

new绑定 vs 显示绑定

```javascript
function foo(something) {
    this.a = something;
}

var obj1 = {
    foo: foo
};

var obj2 = {}

obj1.foo(2);
console.log(obj1.a);  // 2

obj1.foo.call(obj2, 3);
console.log(obj2.a);  // 3

var bar = new obj1.foo(4);
console.log(obj1.a);  // 2
console.log(bar.a);  // 4
```

new绑定win！！！

硬绑定 vs 显式绑定

```
function foo(something) {
    this.a = something;
}

var obj1 = {}

var bar = foo.bind(boj1);
bar(2);
console.log(obj1.a);  // 2

var baz = new bar(3);
console.log(obj1.a);  // 2
console.log(baz.a);  // 3
```

new绑定win！！！

但是这里有一点需要注意：bar被硬绑定到了obj1上，但是new bar(3)并没有像我们想象的那样把`obj1.a`修改为3。相反，new修改了硬绑定（到obj1的）调用bar(...)中的this。因为使用了new绑定，我们得到了一个名字为`baz`的新对象，并且`baz.a`的值是3。

### 间接引用

间接引用最容易在赋值时发生，它会把 ：

就像前面看到的隐式绑定丢失一样：

```javascript
var a = 2;
var o = {
    a: 3,
    foo: foo
};
var p = {
    a: 4
};

function foo() {
    console.log(this.a);
}

o.foo();  // 3  隐式绑定
(p.foo = o.foo)();  // 2 复制后间接引用，应用默认绑定
```

赋值表达式p.foo = o.foo的返回值是目标函数的引用，因此调用位置是foo()而不是p.foo()或者o.foo()。其实只要把它当做指针就可以理解了。

## 扩展：this词法

由于混乱的this带来的种种困惑，ES6中介绍了一种特殊函数类型：箭头函数。

箭头函数并非使用function关键字定义，而是使用被称为“胖箭头”的操作符=>定义的。箭头函数根据外层（函数或者全局）**作用域**来决定this。

```javascript
function foo() {
    // 返回一个箭头函数
    return (a) => {
        //this继承自foo()
        console.log(this.a);
    };
}

var obj1 = {
    a: 2
}

var obj2 = {
    a: 3
}

var bar = foo.call(obj1);
bar.call(obj2);  // 2, 不是3！
```

foo()内部创建的箭头函数会捕获调用时foo()的this。由于foo()的this绑定到obj1，bar（引用箭头函数）的this也会绑定到obj1，**箭头函数的绑定无法被修改**。（new也不行）

箭头函数最常用于回调函数中，例如事件处理器或者定时器（它们是最容易丢失this的）：

```javascript
function foo() {
    setTimeout(() => {
        // 这里的this在词法上继承自foo()
        console.log(this.a);
    } , 100);
}

var obj = {
    a: 2
}

foo.call(obj);  // 2
```

可见箭头函数可以向bind(...)一样确保函数的this被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域来取代了传统的this机制。

如果你经常编写this风格的代码，但是绝大部分时候都会使用self = this或者箭头函数来否定this机制，那么你或许应当：

1.只使用词法作用域并完全抛弃this风格的代码；

2.完全采取this方格，在必要时使用bind(...)，尽量避免使用self = this和箭头函数。

当然，包含两种代码风格的程序可以正常运行，但是在同一个函数或者同一个程序中混合使用这两种风格通常会使代码更难以维护。











