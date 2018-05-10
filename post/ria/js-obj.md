---
layout: page
title:	js notes
category: blog
description:
---
# Preface
# Object

## keys
list forEach

	Object.keys({a:1, b:2})
		["a", "b"]
	Object.keys(obj).forEach(function(key){
		console.log(key, obj[key])
	})

### has key
1. hasOwnProperty: 不包括原型链
2. keys: 也不含proto
2. `in`: key 它可能是obj 继承的属性, 不一定是obj 本身的属性

    'toString' in xiaoming; // true, 不是xiaoming 本身，而是object 都有

	function hasValue(obj, key, value) {
		return obj.hasOwnProperty(key) && obj[key] === value;
	}

### delete key

	delete variable
	delete obj.name

## update, extend

	Object.prototype.extend = function(defaults) {
		for (var i in defaults) {
			if(i in this){
				this[i] = defaults[i];
			}
		}
	};

## Property 属性

    obj.constructor
    obj.hasOwnProperty('attr') # 不是继承proto的
    obj.propertyIsEnumerable('attr')
        判断给定的属性是否可以用 for...in 语句进行枚举。
    obj.__proto__

## value

	Object.prototype.hasOwnValue = function(val) {
		for(var prop in this) {
			if(this.hasOwnProperty(prop) && this[prop] === val) {
				return true;
			}
		}
		return false;
	};

### Object.defineProperty()

	obj[name] = value;
	//or
    Object.defineProperty(obj, name, { value: value, writable: false });

Example1，在ES5 中Prototype 可以用来将定义魔法属性，可以实现类似 PHP 类的`__get`, `__set`

	Number.prototype = Object.defineProperty(
	  Number.prototype, "double", {
		get: function (){return (this + this)}
	  }
	);
	3.double;//被当作小数点
	(3).double;//6
	3['double'].double;//12
	3..double;//6 第一个点被解析为小数点，第二个点被解释为操作符

对原始类型做对象操作时，js 会用原始类型的对象wrapper 把变量包装一下，然后在临时对象上操作。

	var a=3; //是数值，不是数值对象
	a.key=1; //js 数据类型的对象wrapper 会将a 包装为临时的数值对象. 相当于`(new Number(a)).key=1`
	a.key;//undefined 因为临时对象不存在了

# 定义与创建

## 对象实例的原型
1. prototype是原型独有的属性,也就是有constructor可以实例化对象的方法才有;
2. `__proto__` 是对象才有的属性, 指向原型属性，实现原型继承.

    Object.prototype//{}
    o=new Object()
    o.__proto__ === Object.prototype

    Object.__proto__//[function] 这个就别管它了

### new 与 Object.create
Object.create(func.prototype)相当于: `{__proto__:func.prototype}`
Object.create(obj)相当于: `{__proto__:obj}` 相当于对象继承了

    Object.create =  function (func.prototype) {
        var F = function () {};
        F.prototype = func.prototype;
        return new F();
    };

new func() 相当于:
        `{attrs:vals,__proto__:func.prototype}`

    var o1 = new Object();
    o1['__proto__'] = func.prototype;
    func.call(o1);

> arrow函数不是匿名函数，它没有`[[Construct]] internal method` ，不能进行`new`,  

### 对象继承对象

    a={age:10, name:'xiao'}
    b={age:12}
    b.__proto__ = a

### 类继承类
原型继承原型

    function ClassA(sColor){ }
    function ClassB(){ 
        self = Object.getPrototypeOf(this)
        //1. 原型链冒充 原类的静态成员(prototype).
        if(self.__init === undefined){
            self.__init === true
            ClassB.prototype = Object.create(ClassA.prototype) //prototype隔离
        }
        // 2. 再冒充ClassB对象.
        self.call(this, sColor);
    }

让我们封装一下:

    function inherits(that, Parent, ...args){
        if(that.__init === undefined){
            that.__init === true
            self = Object.getPrototypeOf(that)
            self.prototype = Object.create(Parent.prototype)
        }
        self.apply(that, args);
    }

## class 定义类
所有的方法都定义在prototype 上

    class Animal{
        constructor(name){
            this.name = name
        }
        method1(){}
    }
    class Cat extends Animal{
        constructor(name){
            super(name) //super 是编译时确定 必须在前
            this.xxx()
        }
        say(){
            return `Hello, ${this.name}!`
            return super.method1()
        }
    }
    (new Cat('ahui')).say()

class 定义的方法是不可枚举的

    class Point {
        constructor(x, y) {
            // ...
        }

        toString() {
            // ...
        }
        a(){}
    }

    Object.keys(Point.prototype)
    // []
    Object.getOwnPropertyNames(Point.prototype)
    // ["constructor","toString"]

你也可以这样es5 写法绑定就可以枚举

    class Point {
        constructor(){ }
    }

    Object.assign(Point.prototype, {
        toString(){},
        toValue(){}
    });

属性名可以用变量:

     [methodName]() { }

### constructor
constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象。

    class Foo {
        constructor() {
            return Object.create(null);
        }
    }
### set/get
    class MyClass {
        constructor() {
            // ...
        }
        get prop() {
            return 'getter';
        }
        set prop(value) {
            console.log('setter: '+value);
        }
    }
    var descriptor = Object.getOwnPropertyDescriptor( MyClass.prototype, "prop");

    "get" in descriptor  // true

### new.target === class
new.target会返回子类

    class Rectangle {
        constructor(length, width) {
            console.log(new.target === Rectangle);
            this.length = length;
            this.width = width;
        }
    }

    var obj = new Rectangle(3, 4); // 输出 true

### static
static 不可以被实例继承(不是prototype), static属于类自己(相当于proto)

#### static prop
    class Foo {
    }

    Foo.prop = 1;

提案

    class MyClass {
    static myStaticProp = 42;

#### static method
0. 只可以被子类继承
1. 如果静态方法包含this关键字，这个this指的是类，而不是实例。
2. super.staticMethod() ，super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类

    class Foo {
        static bar () {
            this.baz();
        }
        static baz () {
            console.log('hello');
        }
        baz () {
            console.log('world');
        }
    }

    Foo.bar() // hello

### Generator 
如果某个方法之前加上星号（*），就表示该方法是一个 Generator 函数。

    class Foo {
        constructor(...args) {
            this.args = args;
        }
        * [Symbol.iterator]() {
            for (let arg of this.args) {
            yield arg;
            }
        }
    }

    for (let x of new Foo('hello', 'world')) {
        console.log(x);
    }

上面代码中，Foo类的Symbol.iterator方法前有一个星号，表示该方法是一个 Generator 函数。Symbol.iterator方法返回一个Foo类的默认遍历器，for...of循环会自动调用这个遍历器。

### 直接定义对象：

    let parent = {
        method1() { .. },
    }

    let child = {
        __proto__: parent,
        method1() { return super.method1() },
    }

### 创建对象
1. obj = new Object();
2. person={firstname:"John"};
3. obj = new func(param1, param2);
    obj.constructor === func.prototype.constructor; // true

    let person = new class {
        constructor(name) {
            this.name = name;
        }

        sayName() {
            console.log(this.name);
        }
    }('张三');



## 对象分类
### 包装对象与原始primitive 对象
number、boolean和string都有包装对象. (包装对象一点用处也没有，所以还是用primitive 吧)

    typeof new Number(123); // 'object'
    typeof Number(123); // 'number'
    Number(123) === 123; // true

    var x1 = {};            // new object
    var x2 = "";            // new primitive string
    var x3 = 0;             // new primitive number
    var x4 = false;         // new primitive boolean
    var x5 = [];            // new array object
    var x6 = /str/igm       // new RegExp('str', 'igm') object
    var x7 = function(){};  // new function object

临时包装销毁：

    (123).toString(); // '123'; #加括号是为了防止解析为小数点
    123..toString(); // '123'; #加括号是为了防止解析为小数点

### 对象原型判断

    arr ----> Array.prototype ----> Object.prototype ----> null

注意null/Array/{}的类型是object, 不可以用typeof 判断

	obj instanceof funcname// 函数原型名
    null === null
    Array.isArray(arr)

全局/局部变量是否存在用:

    typeof window.myVar === 'undefined'
    typeof myVar === 'undefined'

## 定义对象的模式

### 工厂方式(deprecated)
工厂方式的点是:
1. 每次new factory 都会创建单独的函数
2. 太复杂

	function factory(v1, v2){
		obj = new Object();
		obj.param1 = v1;
		obj.param2 = v2;
		obj.func = function(){};
		return obj;
	}
	obj = factory(1,2)

### 构造方式(deprecated)
只避免了factory 的第二个缺点: 复杂性

1. 每次都会生成新的function.

	function construction(v1,v2){
		this.p1 = v1;
		this.p2 = v2;
		this.func = function(){}; //为避免重复的func, 可在外部定义
	}
	obj = new constructor(1,2)

### 原型方式(deprecated)
没有工厂方法的缺点，但产生了新的缺点：

- 不能传参数

	function Car() {
	}
	Car.prototype.color = "blue";
	Car.prototype.showColor = function() {
	  alert(this.color);
	};
	(new Car) instanceof Car;//true;
	(new Car) instanceof Object;//true;
	(new Car) instanceof Number;//false;

### 静态的构造原型混合
- 构造: 放私有的属性
- 原型: 放公共的属性

> 批评静态构造函数原型混合方式的人认为，在构造函数内部找属性，在其外部找方法的做法不合逻辑: 所以就有一种动态 构造/原型混合

#### 动态构造原型混合

	function Car(color){
		//private
		this.color=color;

		//public(prototype)
		//if (typeof Car._initialized === "undefined") {
		if (Car._initialized === undefined) {
			var self = Car;
			self._initialized = true;//非prototype 的_initialized 不会被继承，但是它相当于Car 内的静态变量
			self.prototype.showColor = function() {
			  console.log(this.color);
			};
		}
	  }
	}

## Extends 继承

### 对象冒充
#### 利用this 变化
ClassB 继承 ClassA

	function ClassA(color){
		this.color = color;
	}
	function ClassB(color, num){
		this.method = ClassA; //ClassB就冒充了ClassA中的this
		this.method(color)
		delete this.method;

		this.method = ClassA1; //ClassB就冒充了ClassA1中的this(可以多重继承的)
		this.method(num);
		delete this.method;

		this.color = value; //Notice; 会覆盖前面的属性. 请确保属性名不冲突
	}
	new ClassB('red', 5);

### Call()

	function sayColor(sPrefix,sSuffix) {
		alert(sPrefix + this.color + sSuffix);
	};

	var obj = new Object();
	obj.color = "blue";

	sayColor.call(obj, "The color is ", "a very nice color indeed.");// saycolor中的this会指向obj, obj对象就冒充了saycolor.

#### 用call 实现继承

	function ClassB(sColor, sName) {
		//this.newMethod = ClassA;
		//this.newMethod(color);
		//delete this.newMethod;
		ClassA.call(this, sColor);//执行时，this冒充了ClassA

		this.name = sName;
		this.sayName = function () {
			alert(this.name);
		};
	}

### apply 方法冒充继承
apply与call方法类似, 除了参数调用形式不一样.

	function sayColor(sPrefix,sSuffix) {
		alert(sPrefix + this.color + sSuffix);
	};

	var obj = new Object();
	obj.color = "blue";

	sayColor.apply(obj, new Array("The color is ", "a very nice color indeed."));

#### apply 实现继承.

	function ClassB(sColor, sName) {
		//this.newMethod = ClassA;
		//this.newMethod(color);
		//delete this.newMethod;
		ClassA.apply(this, arguments);//arguments === new Array(sColor);

		this.name = sName;
		this.sayName = function () {
			alert(this.name);
		};
	}

还有一个bind 方法，但函数使用bind 时，函数不会执行 `var get = request.bind(this, 'GET', arg2, ... );` request 就不会执行。

它主要用于定制一个新的 requset 的函数，并默认函数的this 及定制参数:
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind


### 原型链（prototype chaining）
用prototype 对象去继承.
缺点: 不能控制被继承类的传参(一般都不会传参数)

	function ClassA() {
	}

	ClassA.prototype.color = "blue";
	ClassA.prototype.sayColor = function () {
		alert(this.color);
	};

	function ClassB() {
	}

	ClassB.prototype = new ClassA(); //继承了啦.

### 混合方式
对象冒充: 不能冒充静态成员(prototype public)
原型链: 因为prototype 是公共的, 所以传argument(private)就不合适了.故产生了apply/call + 原型的混合方式.

	function ClassA(color) {
		this.color = color;
		if(ClassA.init === undefined){
			ClassA.init = true;
			ClassA.prototype.sayColor = function () {
				console.log(this.color);
			};
		}
	}

	function ClassB(sColor, sName) {
		var self = Object.getPrototypeOf(this)
		if( self.init === undefined){
			self.init = true;
            // 最好别传new ClassA(sColor),因为sColor应该是每个对象私有的.
			//self.prototype = new ClassA(); 
            
            //1. 原型链冒充 原类的静态成员(prototype). 
			self.prototype = Object.create(ClassA.prototype)
			self.prototype = ClassA.prototype
		}
		ClassA.call(this, sColor);// 2. 再冒充ClasA对象.

        //自身
		this.name = sName;
	}

用Call 继承属性方法初始化，用`Object.create` 继承公共属性与方法
再来一个例子：

	//Shape - superclass
	function Shape() {
	  this.x = 0;
	  this.y = 0;
	}

	Shape.prototype.move = function(x, y) {
		this.x += x;
		this.y += y;
		console.info("Shape moved.");
	};

	// Rectangle - subclass
	function Rectangle() {
	  Shape.call(this); //call super constructor.
	}

    //不要共享prototype
	Rectangle.prototype = Object.create(Shape.prototype);

	var rect = new Rectangle();

	rect instanceof Rectangle //true.
	rect instanceof Shape //true.

	rect.move(1, 1); //Outputs, "Shape moved."