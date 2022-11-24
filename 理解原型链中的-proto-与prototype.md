---
title: 理解原型链中的__proto__与prototype
tags:
  - js
categories:
  - 原创
date: 2018-12-28 08:55:56
---

## 前言

前面发了一篇关于深入理解js的继承机制的文章，入口如下：

[【转】彻底理解js中关于prototype的继承机制](https://blog.chace0120.cn/2018/12/25/%E3%80%90%E8%BD%AC%E3%80%91%E5%BD%BB%E5%BA%95%E7%90%86%E8%A7%A3js%E4%B8%AD%E5%85%B3%E4%BA%8Eprototype%E7%9A%84%E7%BB%A7%E6%89%BF%E6%9C%BA%E5%88%B6/)

在js的继承机制中，prototype对象有着至关重要的作用。笔者在此从原型链的角度，聊聊__proto__和prototype。

## __proto__与prototype的区别

原型链是一个耳熟能详的词儿，它代表着js对象的“族谱”。一个对象的原型链展示了一条完整的继承路径。

在原型链中，__proto__与prototype起着承上启下的作用。虽然两者长得很像，但还是有本质区别的。

- js中每个对象都具有内置属性__proto__，该属性的值就是该对象构造函数的prototype对象。对象字面量的__proto__的值就是Object.prototype。
- js中并不是所有的对象都有内置属性prototype。因为js中函数可以充当构造函数，所有只有函数才有prototype属性。当然函数也是对象，所以函数也有内置的__proto__属性。只不过只有拿函数当作构造函数来使用时，才需要我们去关注__proto__和prototype属性。
- 构造函数构造出实例对象后，实例对象默认是继承prototype对象中的属性和方法的。具体内部是怎样实现继承的呢？实例通过设置自己的__proto__指向承构造函数的prototype来实现这种继承。
- 一般情况下是不推荐直接操作对象的内置属性__proto__的。而prototype却没有这种限制，可以进行修改调整，实现继承的扩展。

<!-- more -->

## 原型链图解

所谓一图胜千言，先来看下原型链的一张图解，概括的很详尽。

{% asset_img img1.jpg %}

从图中可以看出，在原型链中，对象的__proto__属性可以理解为“穿针引线”，而链中的每个节点则是构造函数的prototype对象，prototype对象中定义了需要被继承的属性和方法。

### instanceof运算符

instanceof运算符返回一个布尔值，表示对象是否为某个构造函数的实例。

``` javascript
f1 instanceof Foo; // true
f1 instanceof Object; // true
```

看上面原型链图中的实例对象f1，结合上面代码的运行结果，可以看出instanceof运算符，实际是沿着指定实例对象的原型链在查找指定构造函数的prototype对象，如果存在该prototype对象则返回true，否则返回false。

### 鸡和蛋的问题

众所周知，js是单继承的。从原型链图中可以看出，任何一个对象的原型链顶端一定是Object.prototype，而Object.prototype之上就是null啦。

``` javascript
Function instanceof Object; // true
```

js中，万物皆对象。Function作为构造函数时，Function.prototype通过__proto__指向了顶端的Object.prototype，所以上面的运行结果表明Function继承于Object。

``` javascript
Object instanceof Function; // true
```

看到这里很多人会迷惑了，到底是Object继承了Function，还是Function继承了Object呢？这个就好比先有的鸡还是先有的蛋呢？

笔者认为，通过上图中的原型链说明，毋庸置疑的是Object.prototype位于链的顶端。只不过函数在js中太特别了，Object可以作为构造函数存在，而所有函数（包括构造函数）均是通过Function构造函数产生的实例对象，所以Object构造函数的__proto__指向的是Function.prototype，也就产生了上面的运行结果。

从原型链图中也可以发现，Function作为构造函数，Function.__proto__指向的是Function.prototype，而Function的prototype对象也是Function.prototype，这点是非常特殊的存在。

### Object.create方法

Object.create方法允许不通过构造函数产生一个实例对象，产生的实例对象继承于指定的另一个实例对象，看下面的代码原理。

``` javascript
if (typeof Object.create !== 'function') {
  Object.create = function (obj) {
    function F() {}
    F.prototype = obj;
    return new F();
  };
}
```

可以看出，Object.create方法产生的实例对象，其__proto__指向指定的obj，obj中的属性和方法可以供产生的实例对象继承。

## 参考文章

[从__proto__和prototype来深入理解JS对象和原型链](https://github.com/creeperyang/blog/issues/9)
