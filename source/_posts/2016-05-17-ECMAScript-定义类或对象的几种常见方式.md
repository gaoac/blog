---
title: ECMAScript 定义类或对象的几种常见方式
date: 2016-05-17 19:17:12
tags: [ECMAScript,JavaScript]
categories: ECMAScript
---
### 创建对象
当我们需要创建一个对象时，通常会采取Object构造函数 var Dog = new Object()，或者对象字面量 var Dog = {}。然而当使用同一个接口创建很多对象时，就会产生很多重复的代码。
<!-- more -->
例如：   
```javascript
var oDog1 = new Object();
oDog1.color = "white";
oDog1.weight = "8kg";
oDog1.showColor = function() {
    alert(this.color);
};

var oDog2 = new Object();
oDog2.color = "black";
oDog2.weight = "6kg";
oDog2.showColor = function() {
    alert(this.color);
};
```
### 解决方案
#### 工厂模式
考虑到ECMAScript中无法创建类（ECMAScript6中可以创建Class），开发人员就发明了一种函数，用函数来封装创建对象的细节，如下图例子所示。
```javascript
function createDog() {
    var oTempDog = new Object();
    oTempDog.color = "black";
    oTempDog.weight = "6kg";
    oTempDog.showColor = function() {
      alert(this.color);
    };
    return oTempDog;
}

var oDog1 = createDog();
var oDog2 = createDog();
```
我们还可以修改 createDog() 函数，给它传递各个属性的默认值，而不是简单地赋予属性默认值：
```javascript
function createDog(sColor,sWeight) {
    var oTempDog = new Object();
    oTempDog.color = sColor;
    oTempDog.weight = sWeight;
    oTempDog.showColor = function() {
      alert(this.color);
    };
    return oTempDog;
}

var oDog1 = createDog("red","12kg");
var oDog2 = createDog("black","10kg");
```
#### 在工厂函数外定义对象的方法
虽然 ECMAScript 越来越正式化，但创建对象的方法却被置之不理，且其规范化至今还遭人反对。一部分是语义上的原因（它看起来不像使用带有构造函数 new 运算符那么正规），一部分是功能上的原因。功能原因在于用这种方式必须创建对象的方法。前面的例子中，每次调用函数 createDog()，都要创建新函数 showColor()，意味着每个对象都有自己的 showColor() 版本。而事实上，每个对象都共享同一个函数。
有些开发者在工厂函数外定义对象的方法，然后通过属性指向该方法，从而避免这个问题：
 ```javascript
function showColor() {
    alert(this.color);
}

function createDog(sColor,sWeight) {
    var oTempDog = new Object();
    oTempDog.color = sColor;
    oTempDog.weight = sWeight;
    oTempDog.showColor = showColor;
    return oTempDog;
}

var oDog1 = createDog("red","12kg");
var oDog2 = createDog("black","10kg");

Dog1.showColor(); //输出 "red"
Dog2.showColor(); //输出 "black"
```
在上面这段重写的代码中，在函数 createDog() 之前定义了函数 showColor()。在 createDog() 内部，赋予对象一个指向已经存在的 showColor() 函数的指针。从功能上讲，这样解决了重复创建函数对象的问题；但是从语义上讲，该函数不太像是对象的方法。
所有这些问题都引发了开发者定义的构造函数的出现。
#### 构造函数方式
创建构造函数就像创建工厂函数一样容易。第一步选择类名，即构造函数的名字。根据惯例，这个名字的首字母大写，以使它与首字母通常是小写的变量名分开。除了这点不同，构造函数看起来很像工厂函数。请考虑下面的例子：
```javascript
function Dog(sColor,sWeight) {
  this.color = sColor;
  this.weight = sWeight;
  this.showColor = function() {
    alert(this.color);
  };
}

var oDog1 = new Dog("red","13kg");
var oDog2 = new Dog("blue","11kg");
```
下面为您解释上面的代码与工厂方式的差别。首先在构造函数内没有创建对象，而是使用 this 关键字。使用 new 运算符构造函数时，在执行第一行代码前先创建一个对象，只有用 this 才能访问该对象。然后可以直接赋予 this 属性，默认情况下是构造函数的返回值（不必明确使用 return 运算符）。
现在，用 new 运算符和类名 Dog 创建对象，就更像 ECMAScript 中一般对象的创建方式了。
你也许会问，这种方式在管理函数方面是否存在于前一种方式相同的问题呢？是的。
就像工厂函数，构造函数会重复生成函数，为每个对象都创建独立的函数版本。不过，与工厂函数相似，也可以用外部函数重写构造函数，同样地，这么做语义上无任何意义。这正是下面要讲的原型方式的优势所在。
#### 原型方式
该方式利用了对象的 prototype 属性，可以把它看成创建新对象所依赖的原型。
这里，首先用空构造函数来设置类名。然后所有的属性和方法都被直接赋予 prototype 属性。我们重写了前面的例子，代码如下：
```javascript
function Dog() {
}

Dog.prototype.color = "blue";
Dog.prototype.weight = "8kg";
Dog.prototype.showColor = function() {
  alert(this.color);
};

var oDog1 = new Dog();
var oDog2 = new Dog();
```
在这段代码中，首先定义构造函数（Dog），其中无任何代码。接下来的几行代码，通过给 Dog 的 prototype 属性添加属性去定义 Dog 对象的属性。调用 new Dog() 时，原型的所有属性都被立即赋予要创建的对象，意味着所有 Dog 实例存放的都是指向 showColor() 函数的指针。从语义上讲，所有属性看起来都属于一个对象，因此解决了前面两种方式存在的问题。
此外，使用这种方式，还能用 instanceof 运算符检查给定变量指向的对象的类型。因此，下面的代码将输出 TRUE
```javascript
alert(oDog1 instanceof Dog);	//输出 "true"
```
#### 原型方式的问题
原型方式看起来是个不错的解决方案。遗憾的是，它并不尽如人意。
首先，这个构造函数没有参数。使用原型方式，不能通过给构造函数传递参数来初始化属性的值，因为 oDog1 和 oDog2 的 color 属性都等于 "blue"，weight 属性都等于 "8kg"。这意味着必须在对象创建后才能改变属性的默认值，这点很令人讨厌，但还没完。真正的问题出现在属性指向的是对象，而不是函数时。函数共享不会造成问题，但对象却很少被多个实例共享。请思考下面的例子：
```javascript
function Dog() {
}

Dog.prototype.color = "blue";
Dog.prototype.weight = "8kg";
Dog.prototype.nickname = new Array("Tom","Bean");
Dog.prototype.showColor = function() {
  alert(this.color);
};

var oDog1 = new Dog();
var oDog2 = new Dog();

oDog1.nickname.push("Kil");

alert(oDog1.nickname); //输出 "Tom,Bean,Kil"
alert(oDog2.nickname); //输出 "Tom,Bean,Kil"
```
上面的代码中，属性 nickname 是指向 Array 对象的指针，该数组中包含两个名字 "Tom" 和 "Bean"。由于 nickname 是引用值，Dog 的两个实例都指向同一个数组。这意味着给 oDog1.nickname 添加值 "Kil"，在 oDog2.nickname 中也能看到。输出这两个指针中的任何一个，结果都是显示字符串 "Tom,Bean,Kil"。
由于创建对象时有这么多问题，你一定会想，是否有种合理的创建对象的方法呢？答案是有，需要联合使用构造函数和原型方式
#### 混合的构造函数/原型方式
联合使用构造函数和原型方式，就可像用其他程序设计语言一样创建对象。这种概念非常简单，即用构造函数定义对象的所有非函数属性，用原型方式定义对象的函数属性（方法）。结果是，所有函数都只创建一次，而每个对象都具有自己的对象属性实例。
我们重写了前面的例子，代码如下：
```javascript
function Dog(sColor,sWeight) {
  this.color = sColor;
  this.weight = sWeight;
  this.nickname = new Array("Tom","Bean");
}

Dog.prototype.showColor = function() {
  alert(this.color);
};

var oDog1 = new Dog("black","8kg");
var oDog2 = new Dog("blue","20kg");

oDog1.nickname.push("Kil");

alert(oDog1.nickname); //输出 "Tom,Bean,Kil"
alert(oDog2.nickname); //输出 "Tom,Bean"
```
现在就更像创建一般对象了。所有的非函数属性都在构造函数中创建，意味着又能够用构造函数的参数赋予属性默认值了。因为只创建 showColor() 函数的一个实例，所以没有内存浪费。此外，给 oDog1 的 nickname 数组添加 "Kil" 值，不会影响到 oDog2 的数组，所以输出这些数组的值时，oDog1.nickname 显示的是 "Tom,Bean,Kil"，而 oDog2.nickname 显示的是 "Tom,Bean"。因为使用了原型方式，所以仍然能利用 instanceof 运算符来判断对象的类型。
这种方式是 ECMAScript 采用的主要方式，它具有其他方式的特性，却没有他们的副作用。不过，有些开发者仍觉得这种方法不够完美。
#### 动态原型方法
动态原型方法的基本想法与混合的构造函数/原型方式相同，即在构造函数内定义非函数属性，而函数属性则利用原型属性定义。唯一的区别是赋予对象方法的位置。下面是用动态原型方法重写的 Dog 类：
```javascript
class Dog {
  public String color = "blue";
  public String weight = "8kg";

  public Dog(String color, int weight) {
    this.color = color;
    this.weight = weight;
  }
  
  public void showColor() {
    System.out.println(color);
  }
}
```
Java 很好地打包了 Dog 类的所有属性和方法，因此看见这段代码就知道它要实现什么功能，它定义了一个对象的信息。批评混合的构造函数/原型方式的人认为，在构造函数内部找属性，在其外部找方法的做法不合逻辑。因此，他们设计了动态原型方法，以提供更友好的编码风格。
动态原型方法的基本想法与混合的构造函数/原型方式相同，即在构造函数内定义非函数属性，而函数属性则利用原型属性定义。唯一的区别是赋予对象方法的位置。下面是用动态原型方法重写的 Dog 类：
```javascript
function Dog(sColor,sWeight) {
  this.color = sColor;
  this.weight = sWeight;
  this.nickname = new Array("Tom","Bean");
  
  if (typeof Dog._initialized == "undefined") {
    Dog.prototype.showColor = function() {
      alert(this.color);
    };
	
    Dog._initialized = true;
  }
}
```
直到检查 typeof Dog._initialized 是否等于 "undefined" 之前，这个构造函数都未发生变化。这行代码是动态原型方法中最重要的部分。如果这个值未定义，构造函数将用原型方式继续定义对象的方法，然后把 Car._initialized 设置为 true。如果这个值定义了（它的值为 true 时，typeof 的值为 Boolean），那么就不再创建该方法。简而言之，该方法使用标志（_initialized）来判断是否已给原型赋予了任何方法。该方法只创建并赋值一次，传统的 OOP 开发者会高兴地发现，这段代码看起来更像其他语言中的类定义了。
#### 采用哪种方式
如前所述，目前使用最广泛的是混合的构造函数/原型方式。此外，动态原始方法也很流行，在功能上与构造函数/原型方式等价。可以采用这两种方式中的任何一种。不过不要单独使用经典的构造函数或原型方式，因为这样会给代码引入问题。
### 原文地址
[ECMAScript 定义类或对象](http://www.w3school.com.cn/js/pro_js_object_defining.asp)