---
layout:     post
title:      "js：原型链"
subtitle:   "原型和原型链"
date:       2017-06-29 20:47:00
author:     "Zi Ning"
header-img: "img/js.jpeg"
tags:
    - JavaScript
    - js
    - prototype 
    - the prototype chain
---

# js：原型链

在js中流传着一句话“一切皆对象”，仔细说起来其实是不严谨的。不过这恰恰说明了对象的思想在JavaScript中占据着重要的地位。对象不仅在实际编程中常常用到，而且认识它对我们了解这门语言大有裨益。原型和原型链就是掌握它的基础和关键。在复习了一遍《JavaScript高级程序设计》后整理了一下这篇笔记。

## 原型

我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个**指针**，指向一个对象，这个对象包含着可以由特定类型的所有实例**共享**。这样不必多写许多重复的属性和方法，在实例中统一从原型中继承！

例如：

```javascript
function Person() {}

Person.prototype.name = "zining";
Person.prototype.age = "22";
Person.prototype.dreamJob = "software Engineer";
Person.prototype.sayName = function() {
  alert(this.name);
}

var person1 = new Person();
Person1.sayName();  // "zining"

var person2 = new Person();
Person2.sayName();  // "zining"
```

可以看到person1和person2都从原型中继承了相同属性和sayName方法。

要理解原型模式的工作原理，必须先理解ECMAScript中原型对象的性质。

### 原型对象

无论什么时候，只要创建一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。同样，所有的原型对象也都会自动获得一个constructor属性，这个属性指向函数的原型对象。

创建自定义的构造函数之后，其原型对象默认只会取得constructor属性，其他的方法都是从Object继承而来的。

当构造函数创建一个新实例后，该实例内部将包含一个指针（内部属性）【[[Prototype]]，可用\__proto__访问】，指向构造函数的原型对象。

大致如图：

![原型对象](/img/in-post/post-js-the-prototype-chain/prototype-relationship.png)

可以看出，`Person.prototype`指向原型对象和`Person.prototype.constructor`又指回了`Person`，原型对象中除了`constructor`属性外还包含后来添加的属性。而实例中，`person1`，`person2`都包含一个内部属性（[[prototype]]，该属性仅仅指向了`Person.prototype`。

### 确定原型对象之间的关系

可以使用`isPrototypeOf()`来确定对象之间是否存在这种关系。

```javascript
alert(Person.prototype.isPrototype(person1))  // true
alert(Person.prototype.isPrototype(person1))  // false
```

还可以使用ES5的新方法`Object.getPrototypeOf()`来返回[[prototype]]的值

```javascript
alert(Object.getPrototypeOf(person1) == Person.prototype)  // true
```

#### 属于实例的属性

使用`hasOwnPrototype()`方法可以检测一个属性是存在实例中还是存在原型之中。只在给定属性存在于对象实例中时，才返回true。

```javascript
alert(person1.hasOwnPrototy("name"))
```

来自ES5的Object.keys()方法接收一个对象作为参数，返回一个包含所有可枚举实例属性的字符串数组。

```javascript
Object.keys(person1)
```

#### 判断属性

in操作符会在对象能够访问给定属性时返回true，无论该属性存在实例中还是原型中。

```javascript
alert("name" in person1)
```

for...in..循环返回的是所以能够通过对象访问的、可枚举的属性，既包含存在实例中的属性，也包含存在原型中的属性。可配合`hasOwnPrototy`使用来进一步确认实例属性。

### 原型对象的搜索机制

每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。首先搜索从对象实例本身开始。如果在实例中找到了具有给定名字的属性，就返回属性的值；如果没找到，则继续搜索指针指向的原型对象，在原型对象中查找给定名字的属性。如果在原型对象中找到这个属性，就返回该属性的值。

例如：

```javascript
function Person() {}

Person.prototype.name = "zining";
Person.prototype.age = "22";
Person.prototype.dreamJob = "software Engineer";
Person.prototype.sayName = function() {
  alert(this.name);
}

var person1 = new Person();
var person2 = new Person();

person1.name = "ningersan";
alert(person1.name);  // "ningersan"
alert(person2.name);  // "zining"
```

这个例子里，`person1`的`name`被一个新值给屏蔽了。因为开始在实例上搜索一个名为`name`的属性，这个属性确实存在，于是就不必在搜索原型了。同样的方式访问person2的`name`，实例没有找到变向上搜索原型，结果在那里找到了`name`属性。

当为对象实例添加一个属性时，该属性会屏蔽原型对象保存中的同名属性。不过，我们可以使用delete操作符删除实例属性。

### 原型对象的简单写法

常见的做法是用一个包含所有属性和方法的对象字面量来重写整个原型对象。但也会带来一定的副作用。

```javascript
function Person() {}
Person.prototype = {
  // constructor: Person,
  name: "zining",
  age: "22",
  dreamJob: "Software Engineer",
  sayName: function () {
    alert(this.name);
  }
};
```

在这段代码中将`Person.prototype`设置为对象字面量形式创建的新对象，从结果上看是一样的。但由于重写了`prototype`，`constructor`属性不再指向Person了，需要手动让其指向到`Person`。

看起来似乎解决了问题，实际上这种方式重设`constructor`属性会导致它的[[Enumerable]]设置为true。默认情况下，原生的`constructor`属性是不可枚举的。这样再使用for...in来遍历属性就会把`constructor`一并遍历出来。

### 原型的动态性

由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够从实例上反应出来。**即使是先创建了实例后再修改原型也照样如此。**

```javascript
var friend = new Person();

Person.prototype.sayHi = function () {
  alert("hi");
}

friend.sayHi();  // "hi"
```

注意：**尽管可以随时为原型添加属性和方法，并且修改能立刻在所有对象实例中反映出来，但如果重写整个原型对象就等于切断了构造函数与最初原型之间的联系，原型不再具有动态性。**

了解了原型，再来看看原型链。

## 原型链

原型链作为实现继承的主要方法，其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

假如我们让原型对象等于另一个类型的实例，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针，假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。

基本模式如下：

```javascript
function SuperType() {
  this.property = true;
}

SuperType.prototype.getSuperValue = function () {
  return this.property;
}

function SubType() {
  this.subproperty = false;
}

// 继承SuperType
SubType.prototype = new SuperType();

SubType.property.getSubValue = function () {
  return this.subproperty;
};

var instance - new SubType();
alert(instance.getSuperValue);  // true
```

以上代码定义了两个类型：`SubperType`和`SubType`。没一个类型分别有一个属性和方法。`SubType`继承了`SuperType`，而继承是通过创建`SuperType`的实例，并给该实例赋给`SubType.prototype`实现的。实现的本质是重写原型对象，代之以一个新类型的实例。原本存在于`SuperType`的实例中所有的属性和方法，现在也存在于`SubType.prototyp`e中。之后我们又给`SubType.prototype`添加了一个新方法。

实例与构造原型和原型之间的关系如图：

![js原型链](/img/in-post/post-js-the-prototype-chain/the-prototype-chain.png)

我们没有使用`SubType`默认提供的原型，而是给它换了一个新原型：`SuperType`的实例。新原型不仅具有作为`SuperType`的实例所拥有的全部属性和方法，而且其内部还有一个指针指向`SuperType`的原型。于是，instance指向`SubType`的原型，`SubType`的原型又指向`SuperType`的原型。注意`instance.construct`现在指向的是	`SuperType`，这是因为原来`SubType.prototype`中的`constructor`被重写了的缘故。

通过原型链，本质上扩写了原型搜索机制。在例子中，调用instance.getSuperValue()会经历一下搜索步骤：

1. 搜索实例


2. 搜索SubType.prototype
3. 搜索SuperType.prototype，找到该方法。
4. 在找不到属性或方法的情况先下，搜索过程要一环一环地前进到原型链末端才会停下来。

对了，**别忘了默认的原型**，所有引用类型默认都继承了`Object`，这个继承也是通过原型链来实现的。要记住，所有函数的默认原型都是`Object`实例，因此默认原型会包含一个内部指针，指向`object.prototype`。这也正是所有自定义类型都继承`toString()`，`valueOf()`等默认方法的根本原因。

### 确定原型和实例的关系

#### 使用instanceOf操作符

只要用这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回true。

#### 使用isPrototypeOf()方法

只要是该原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，结果返回true。

### 注意

* 子类型有时候需要覆盖超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但不管怎样，**给原型添加方法的代码一定要放在替换原型的语句之后。**

* 再通过原型链实现继承时，不能用对象字面量创建原型方法，这样会重写原型链。

### 问题

原型链虽然很强大，可以用它来实现继承，但也存在着一些问题。

最主要的问题来自于包含引用类型值的原型，包含引用类型值的原型属性会被所有实例共享。所以当实例修改引用类型值的时候，该原型对象也会被改写，结果就是所有的实例都会共享该改写的值。

第二个问题是，在创建子类型实例时，不能向超类型的构造函数中传递参数。





