---
title: 【转】彻底理解js中关于prototype的继承机制
tags:
  - js
categories:
  - 转发好文
date: 2018-12-25 15:31:31
---

> 特别声明：下面文中的很多要点、代码示例均摘录自阮一峰文章中的内容。虽然加入了一些笔者自己的思考和总结，但笔者认为该文章应该属于转发。
>
> 分享文章作者：阮一峰
>
> 分享文章：[Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)
>
> 分享文章：[Javascript 面向对象编程（一）：封装](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html)
>
> 分享文章：[Javascript面向对象编程（二）：构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html)
>
> 分享文章：[Javascript面向对象编程（三）：非构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance_continued.html)

## 前言 

笔者是学java出身，所以关于继承，脑海中反射最多的是类、实例对象等概念。笔者在学习和研究js这门语言时，最初曾尝试套用java的理论来对照学习。

java是一门面向对象编程的语言，它与C++、C#等很相似。js中虽然也有对象的概念，但是js中没有类的概念，如果强行将java的继承机制套用到js中来理解js的构造函数、prototype，那无疑是一个痛苦的过程。

所以笔者得出的结论是，最好不要混为一谈，最好不要对照学习。学习js，就要完全按照它的“游戏规则”来做，试着去接受它的机制和理论。

后来笔者阅读了《你不知道的JavaScript（上卷）》，这本书写的很好，很多js语法原理的内容都讲述的很细。其中有章节就讲述了原型（prototype），原型在继承中的作用，甚至包括面向委托的设计、与类设计的对比等等。读完后笔者仿佛有些感悟，仿佛领略到js中原型设计胜过java中类设计的要点。但是，可能书中的文字描述太过于理论吧，过了一段时间，笔者几乎全忘记了。。。

直到笔者阅读过阮一峰大神的4篇文章，笔者认为自己对js的继承机制与应用才有了深入的认知。阮大神的文章一向通俗易懂，举例清晰明了，颇有良师风范呀！

下面笔者会记录学习到的知识点，并加入自己的思考、理解。

<!-- more -->

## js中的封装

所谓封装，笔者认为，就是将一些松散结构的数据，变为一个有组织结构的数据。java中是以类作为封装手段，而js则是以对象作为封装手段。

### 原始方式

下面是一个原型对象，它说明了关于Cat的信息结构。

``` javascript
　var Cat = {
　　　　name : '',

　　　　color : ''

　　}

```

那么基于Cat的规格（Schema），可以产生很多实例对象，如下。

``` javascript
var cat1 = {}; // 创建一个空对象

　　　　cat1.name = "大毛"; // 按照原型对象的属性赋值

　　　　cat1.color = "黄色";

　　var cat2 = {};

　　　　cat2.name = "二毛";

　　　　cat2.color = "黑色";
```

> 但是这种方式有两个缺点：一是如果多生成几个实例，写起来就非常麻烦；二是实例与原型之间，没有任何办法，可以看出有什么联系。

**知识点**：这里提到原型对象和实例对象，是为了说明继承关系。这里提到的原型对象跟prototype还没有任何关系！

### 原始方式的改进

原型对象改进后如下。

``` javascript
　function Cat(name,color) {

　　　　return {

　　　　　　name:name,

　　　　　　color:color

　　　　}

　　}
```

实例对象生成如下，其实就是在调用函数。

```
var cat1 = Cat("大毛","黄色");

var cat2 = Cat("二毛","黑色");
```

> 问题依然存在，cat1和cat2之间没有内在的联系，不能反映出它们是同一个原型对象的实例。

### js中的构造函数

为了解决从原型对象生成实例的问题，构造函数（Constructor）模式应运而生。

> 所谓"构造函数"，其实就是一个普通函数，但是内部使用了this变量。对构造函数使用new运算符，就能生成实例，并且this变量会绑定在实例对象上。

原型对象转换为了构造函数后，如下。

``` javascript
function Cat(name,color){

　　this.name=name;

　　this.color=color;

}
```

**知识点**：构造函数本质上还是函数，js中函数也是对象。函数名应该以大写字母开头。

基于构造函数生成实例对象如下。

``` javascript
var cat1 = new Cat("大毛","黄色");
var cat2 = new Cat("二毛","黑色");
alert(cat1.name); // 大毛
alert(cat1.color); // 黄色
```

js对每个实例对象自动包含一个constructor属性，该属性指向实例对象的构造函数。

``` javascript
alert(cat1.constructor == Cat); //true
alert(cat2.constructor == Cat); //true
```

js还提供了一个instanceof运算符，验证原型对象与实例对象之间的关系。

``` javascript
alert(cat1 instanceof Cat); //true
alert(cat2 instanceof Cat); //true
```

一切看起来很美好，但是单纯的构造函数，依然存在问题。

现在为原型对象Cat添加一个不变的属性type，一个方法eat，如下。

``` javascript
function Cat(name,color){
    this.name = name;
    this.color = color;
    this.type = "猫科动物";
    this.eat = function(){alert("吃老鼠");};
}
```

生成实例对象后，表面上看着一切正常。

``` javascript
var cat1 = new Cat("大毛","黄色");
var cat2 = new Cat ("二毛","黑色");
alert(cat1.type); // 猫科动物
cat1.eat(); // 吃老鼠
```

> 但实际上这样做存在一个很大的弊端，那就是对于每一个实例对象，type属性和eat()方法都是一模一样的内容，每一次生成一个实例，都必须为重复的内容，多占用一些内存。这样既不环保，也缺乏效率。

``` javascript
alert(cat1.eat == cat2.eat); //false
```

能不能让type属性和eat()方法在内存中只生成一次，然后所有实例都指向那个内存地址呢？回答是可以的，prototype就是来解决这个问题。

### prototype模式

> js规定，每个构造函数都有一个prototype属性，该属性指向一个对象，该对象的所有属性和方法，都会被构造函数的实例对象继承。

**思考点**：prototype属性指向的对象，就是经常提到的prototype对象。那么这个prototype属性是否只是针对构造函数才有呢？实例对象是否有自带的prototype属性呢？

**知识点**：自带的prototype属性只是针对构造函数，实例对象是不自带prototype属性的。

``` javascript
function Cat(name,color){
    this.name = name;
    this.color = color;
    this.type = "猫科动物";
    this.eat = function(){alert("吃老鼠");};
}
alert(Cate.prototype); // Object

var cat1 = new Cat('大毛', '黄色');
alert(cat1.prototype); // undefined
```

> 有了prototype，我们可以将那些不变的属性和方法，定义在prototype对象上。

``` javascript
function Cat(name,color){

　　　　this.name = name;

　　　　this.color = color;
}

Cat.prototype.type = "猫科动物";

Cat.prototype.eat = function(){alert("吃老鼠")};
```

看下生成实例对象，继承了prototype中的属性和方法。

``` javascript
var cat1 = new Cat("大毛","黄色");
var cat2 = new Cat("二毛","黑色");
alert(cat1.type); // 猫科动物
cat1.eat(); // 吃老鼠
```

> 这时所有实例的type属性和eat()方法，其实都是同一个内存地址，指向prototype对象，因此就提高了运行效率。

``` javascript
alert(cat1.eat == cat2.eat); //true
```

**思考点**：笔者在这里很好奇，将不变的属性和方法定义在prototype对象上，实例对象对这些属性和方法做改变会影响其他实例和prototype对象吗？

**知识点**：不会，实例对象改变从prototype继承来的属性或方法，并不会导致prototype对象和其他实例对象中对应的属性和方法跟着变化。同时笔者认为这里用“不变”这个词不合适，容易产生误解。prototype对象上的属性或方法是可以变的，变的目的是影响到所有继承该prototype对象的实例对象，所以笔者认为这里应该将“不变的属性和方法”改为“需要共享给所有实例对象的属性和方法”比较合适。下面介绍js的继承机制时也会提到这点。如果是prototype改变了属性或方法，继承该prototype的实例对象中对应的属性或方法会跟着变化，但是如果实例对象已经在prototype变化前将对应的属性或方法做了修改，那么其不受prototype的变化影响，笔者觉着应该是对实例对象继承而来的属性或方法做修改时，产生了深拷贝。

``` javascript
function Cat(name,color){

　　　　this.name = name;

　　　　this.color = color;
}

Cat.prototype.type = "猫科动物";
Cat.prototype.eat = function(){alert("吃老鼠")};

var cat1 = new Cat("大毛","黄色");
var cat2 = new Cat("二毛","黑色");
cat1.type = '两栖动物';
alert(Cat.prototype.type); // 猫科动物
alert(cat1.type); // 两栖动物
alert(cat2.type); // 猫科动物
Cat.prototype.type = "狗科动物";
alert(cat1.type); // 两栖动物
alert(cat2.type); // 狗科动物
```

js内部提供了关于prototype的一些验证方法。

**isPrototypeOf()**

用来判断某个proptotype对象和某个实例之间的关系。

``` javascript
alert(Cat.prototype.isPrototypeOf(cat1)); //true
alert(Cat.prototype.isPrototypeOf(cat2)); //true
```

**hasOwnProperty()**

每个实例对象都有一个hasOwnProperty()方法，用来判断某一个属性到底是本地属性，还是继承自prototype对象的属性。

``` javascript
alert(cat1.hasOwnProperty("name")); // true
alert(cat1.hasOwnProperty("type")); // false
```

**in运算符**

in运算符可以用来判断，某个实例是否含有某个属性，不管是不是本地属性。

``` javascript
alert("name" in cat1); // true
alert("type" in cat1); // true
```

in运算符还可以用来遍历某个对象的所有属性。

``` javascript
for(var prop in cat1) { alert("cat1["+prop+"]="+cat1[prop]); }
```

## js的继承机制设计

当笔者了解了js设计的来龙去脉后，关于对js中prototype的理解，可以说更清晰了，也更轻松了。你会发现，一直以来觉着很难懂的js继承机制，原来竟是js设计者为了让js更容易学习与接收才这样设计的。

### js的来源

> 1994年，网景公司急需一种网页脚本语言，使得浏览器可以与网页互动。工程师Brendan Eich负责开发这种新语言。他觉得，没必要设计得很复杂，这种语言只要能够完成一些简单操作就够了，比如判断用户有没有填写表单。

> 1994年正是面向对象编程（object-oriented programming）最兴盛的时期，C++是当时最流行的语言，而Java语言的1.0版即将于第二年推出，Sun公司正在大肆造势。

> Brendan Eich无疑受到了影响，Javascript里面所有的数据类型都是对象（object），到底要不要设计"继承"机制呢？

> 真的是一种简易的脚本语言，其实不需要有"继承"机制。但是，Javascript里面都是对象，必须有一种机制，将所有对象联系起来。所以，Brendan Eich最后还是设计了"继承"。

### prototype的产生

js的设计者选择了继承，但不想引入类这个概念，原因很简单，他觉着引入类概念太正式了，他希望js更降低初学者的入门难度。于是他虽然借鉴了当时java和c++中的new来生成实例对象，但new后面跟的并不是类，而是构造函数。

但是new + 构造函数的方式并不完美，存在一个问题。也就是上面提到的关于单纯的构造函数方式生成实例对象时存在的问题，无法共享属性和方法，每个实例对象各有有一套自己的属性和方法副本，这是对内存资源的浪费。同时一个实例对象修改了某个具有共享意义的属性后，其他实例是不受影响的。

``` javascript
function DOG(name){
  this.name = name;
  this.species = '犬科';
}

var dogA = new DOG('大毛');
var dogB = new DOG('二毛');
dogA.species = '猫科';
alert(dogB.species); // 显示"犬科"，不受dogA的影响
```

因此，为了解决上述的问题，js的设计者为构造函数设置了一个prototype属性。

> 这个属性包含一个对象，所有实例对象需要共享的属性和方法，都放在这个对象里面；那些不需要共享的属性和方法，就放在构造函数里面。

实例对象一旦创建，将自动引用prototype对象的属性和方法。也就是说，实例对象的属性和方法，分成两种，一种是本地的，另一种是引用prototype的。

``` javascript
function DOG(name){
  this.name = name;
}
DOG.prototype.species = '犬科';

var dogA = new DOG('大毛');
var dogB = new DOG('二毛');
alert(dogA.species); // 犬科
alert(dogB.species); // 犬科

DOG.prototype.species = '猫科';
alert(dogA.species); // 猫科
alert(dogB.species); // 猫科
```

现在，species属性放在prototype对象里，是两个实例对象共享的。只要修改了prototype对象，就会同时影响到两个实例对象。

**知识点**：了解js继承机制的来龙去脉后，笔者有了全新的认知。以前看很多文字描述中，提到的原型对象其实更多指的是prototype对象，笔者认为这只是表面现象下得出的一种映射关系。笔者认为在js继承机制中，是构造函数 + prototype对象共同起到了原型对象的作用，prototype是对构造函数的补充，原型对象产生了实例对象，而实例对象则好像是继承于原型对象，只不过prototype对象在继承机制中起到了至关重要的作用，解决了需要共享属性和方法的问题。

## 构造函数的继承

既然已经了解了js的继承机制，下面来学习一下一些关于js继承的应用场景。先来看下关于**构造函数之间**的继承（笔者认为这应该也是原型对象之间的继承）。

先来设定一个Animal的构造函数，一个Cat的构造函数，Cat构造函数需要继承Animal构造函数。这是一个很正常的一个继承场景示例。那么js中有很多方式可以实现。

``` javascript
function Animal(){
  this.species = "动物";
}

function Cat(name,color){
  this.name = name;
  this.color = color;
}
```

### 方式1：构造函数绑定

``` javascript
function Cat(name,color){
  Animal.apply(this, arguments);
  this.name = name;
  this.color = color;
}

var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

这种方式是在Cat构造函数调用时，同时调用了Animal构造函数，在Animal构造函数中在产生的Cat实例对象上设置了species属性。这是在实例对象产生之际做的一次性的属性设置。

**思考点**：从这里来看，笔者认为prototype在继承中并不是必须要用的呀。不过这种方式并不是常用手段。

### 方式2：prototype+构造函数

使用prototype对象来实现继承，是常见的方式。

``` javascript
Cat.prototype = new Animal();
Cat.prototype.constructor = Cat;
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

这种方式是将Cat构造函数的prototype对象指向了一个**Animal实例**，那么所有的Cat实例就相当于继承了这个**Animal实例**中的属性。

> 这种方式相当于完全删除了prototype 对象原先的值，然后赋予一个新值。

那么下面这行代码就显得至关重要了。构造函数默认的prototype对象中，默认提供一个constructor属性，这个属性指向它的构造函数。更重要的是，实例对象中默认也有一个constructor属性，它默认调用prototype对象的constructor属性。

``` javascript
Cat.prototype.constructor = Cat;
```

> 因此如果没有上面这行代码，就会导致继承链的紊乱（cat1明明是用构造函数Cat生成的）。

```
alert(Cat.prototype.constructor == Animal); //true
alert(cat1.constructor == Animal); //true
```

**知识点**：切记，实现继承时，一旦人为主观地修改了prototype对象的默认指向，下一步必然是为新的prototype对象加上constructor属性，并将这个属性指回原来的构造函数。

### 方式3：直接继承prototype

延续方式2的思路进行改进，对于Animal中需要共享的属性或方法，可以放到其prototype对象中，那么让Cat直接继承Animal的prototype对象，优点是效率高些（不用建立Animal实例），节省内存。缺点是Cat.prototype和Animal.prototype现在指向了同一个对象，任何对Cat.prototype的修改，都会反映到Animal.prototype。

``` javascript
function Animal(){ }
Animal.prototype.species = "动物";

Cat.prototype = Animal.prototype;
Cat.prototype.constructor = Cat;
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物

alert(Animal.prototype.constructor); // Cat
```

因此这种方式是有缺陷的，上面的代码中可以看到Animal.prototype.constructor被变为了Cat。

### 方式4：利用空对象作为中介

为了解决方式3中的缺陷，可以利用空对象作为中介，从而产生了方式4。

``` javascript
var F = function(){};
F.prototype = Animal.prototype;
Cat.prototype = new F();
Cat.prototype.constructor = Cat;
alert(Animal.prototype.constructor); // Animal
```

> F是空对象，所以几乎不占内存。这时，修改Cat的prototype对象，就不会影响到Animal的prototype对象。

利用这种方式，可以封装一个函数，如下。

``` javascript 
function extend(Child, Parent) {
    var F = function(){};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.prototype.constructor = Child;
    Child.uber = Parent.prototype;
}

extend(Cat,Animal);
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

其中有一行需要说明一下。

``` javascript
Child.uber = Parent.prototype;
```

> 意思是为子对象设一个uber属性，这个属性直接指向父对象的prototype属性。（uber是一个德语词，意思是"向上"、"上一层"。）这等于在子对象上打开一条通道，可以直接调用父对象的方法。这一行放在这里，只是为了实现继承的完备性，纯属备用性质。

### 方式5：拷贝继承 

转换一种思路，将要继承的父对象属性和方法拷贝到子对象中，也能实现继承的目的，如下。

``` javascript
function Animal(){}
Animal.prototype.species = "动物";

function extend2(Child, Parent) {
    var p = Parent.prototype;
    var c = Child.prototype;
    for (var i in p) {
        c[i] = p[i];
    }
    c.uber = p;
}

extend2(Cat, Animal);
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

**思考点**：虽然这也是一种方式，但是笔者认为并不常用。而且这里的代码示例中用的是浅拷贝，使用这种方式的话需要注意浅拷贝的影响。当然你如果很中意这种方式的话，完善上面的代码，改造为深拷贝的方式，更稳妥些。

## 非构造函数的继承

这里学习不通过构造函数，来实现两个实例对象之间的继承。

``` javascript
var Chinese = {
  nation:'中国'
}; // 普通对象，代表中国人

var Doctor = {
  career:'医生'
}; // 普通对象，代表医生
```

那么如何让Doctor对象继承Chinese对象呢？

### 方式1：object()方法

json格式的发明人Douglas Crockford，提出了一个object()函数，可以做到这一点。

``` javascript
function object(o) {
    function F() {};
    F.prototype = o;
    return new F();
}
```

这种方式本质上是借助了一个空对象（构造函数），然后将构造函数的prototype对象指向到了父对象上，然后new出子对象。剩下的就是将子对象需要的属性和方法绑定上去即可。

``` javascript
var Doctor = object(Chinese);
Doctor.career = '医生';
alert(Doctor.nation); //中国
```

**思考点**：上面object方法中的F.prototype对象中的constructor属性值是什么呢？

**知识点**：F.prototype.constructor是Object，也就是说Doctor.constructor == Object。原因是传入的Chinese是一个实例对象，即new Object()出来的。因此Doctor.constructor -> F.prototype.constructor -> Chinese.constructor -> Object.prototype.constructor -> Object。

熟悉Object.create()方法的人，看到这里会发现这种方式跟Object.create方法的内部实现是一样的嘛。

``` javascript
if (typeof Object.create !== 'function') {
  Object.create = function (obj) {
    function F() {}
    F.prototype = obj;
    return new F();
  };
}
```

没错，Object.create就是在没有构造函数的场景下，允许在指定一个实例对象为prototype对象的基础上，创建出另一个实例对象，该实例对象包含了指定实例对象的方法和属性。

看到这里，笔者认为这里所谓的非构造函数的继承，其实是**对象关联**。《你不知道的JavaScript（上卷）》中大力推荐面向委托的设计，规避js中长久以来基于构造函数，强行模仿类继承设计而带来的诸多复杂情况，令人费解。而书中提到的面向委托的设计，则是基于对象关联展开的，Object.create方法恰恰是实现对象关联的方式。

### 方式2：浅拷贝

利用拷贝的方式，将父对象中的属性和方法拷贝到子对象中去，从而实现继承。

``` javascript
function extendCopy(p) {
    var c = {};
    for (var i in p) { 
        c[i] = p[i];
    }
    c.uber = p;
    return c;
}

var Doctor = extendCopy(Chinese);
Doctor.career = '医生';
alert(Doctor.nation); // 中国
```

但是这是浅拷贝，存在一个问题，如果父对象中的属性是一个数组或对象（当然方法也是对象），那么子对象中拷贝到的只是一个内存地址，子对象对拷贝过来的属性修改后会影响到父对象中的属性值，这是很危险的。

``` javascript
Chinese.birthPlaces = ['北京','上海','香港'];
var Doctor = extendCopy(Chinese);
Doctor.birthPlaces.push('厦门');

alert(Doctor.birthPlaces); //北京, 上海, 香港, 厦门
alert(Chinese.birthPlaces); //北京, 上海, 香港, 厦门
```

> extendCopy()只是拷贝基本类型的数据，我们把这种拷贝叫做"浅拷贝"。这是早期jQuery实现继承的方式。

### 方式3：深拷贝

依然是拷贝的方式，为了解决浅拷贝的问题，改用深拷贝。

> 所谓"深拷贝"，就是能够实现真正意义上的数组和对象的拷贝。它的实现并不难，只要递归调用"浅拷贝"就行了。

``` javascript
function deepCopy(p, c) {
	var c = c || {};
	for (var i in p) {
		if (typeof p[i] === 'object') {
			c[i] = (p[i].constructor === Array) ? [] : {};
			deepCopy(p[i], c[i]);
		} else {
			c[i] = p[i];
		}
	}

	return c;
}
```

子对象上修改继承来的属性时，则不会再影响到父对象了。

``` javascript
var Doctor = deepCopy(Chinese);
Chinese.birthPlaces = ['北京','上海','香港'];
Doctor.birthPlaces.push('厦门');

alert(Doctor.birthPlaces); //北京, 上海, 香港, 厦门
alert(Chinese.birthPlaces); //北京, 上海, 香港
```

## 总结

综上所述，笔者认为js中的继承，prototype对象是最常用的手段，当然还有其他方式，比如拷贝，但并不常见。看到【构造函数的继承】中的【方式2:prototype+构造函数】，提到将prototype对象指向实例对象是有些许缺陷的，但再看【构造函数的继承】中的【方式4:利用空对象作为中介】和【非构造函数的继承】中的【方式1:object()方法】，不难发现，无论是构造函数还是非构造函数的继承，实现方式最终的本质依然是prototype对象指向了一个实例对象，只不过这些实现方式中解决了那些缺陷。



