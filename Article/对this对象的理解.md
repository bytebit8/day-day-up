- [0.1. 什么是 this？](#01-什么是-this)
- [0.2. this 保存在哪里？](#02-this-保存在哪里)
- [0.3. this 的一般规则](#03-this-的一般规则)
- [0.4. this 的注意事项](#04-this-的注意事项)
  - [0.4.1. use strict](#041-use-strict)
  - [0.4.2. new 运算符](#042-new-运算符)
  - [0.4.3. 函数和方法](#043-函数和方法)
  - [0.4.4. 箭头函数](#044-箭头函数)
  - [0.4.5. apply、call、bind](#045-applycallbind)
- [0.5. this 指向的判断步骤](#05-this-指向的判断步骤)
- [0.6. 其它相关](#06-其它相关)
  - [0.6.1. 定时器中的 this](#061-定时器中的-this)

---

## 0.1. 什么是 this？

`this` 代指了程序当前执行的上下文对象。
`this` 是 ECMAScript 中的关键字，但也是一个引用类型的变量，其值就是上下文对象所在“堆”中的引用地址。
`this` 只会与函数/方法有关，因为它们是可执行的代码。

而上下文对象可以简单粗暴的理解为当前代码执行所处的对象。例如下面代码的执行就是发生在 `window` 对象中，所以通过 `this`关键字，就可以拿到 `window` 对象的属性与方法。

```js
var _name = "world";
function sayHello(name) {
  return `Hello ${this._name}`;
}

sayHello();
```

而下面代码的执行则是放生在对象 `obj` 中的。

```js
var _name = "world";
var obj = {
  _name: "obj",
  getObjName() {
    return this.name;
  },
  getTopName() {
    return this._name;
  },
};

obj.getObjName(); //obj
obj.getTopName(); // undefined
```

我们约定：

- 全局对象所调用的函数就称之为“函数”，例如 `sayHello()` => `window.sayHello()`。
- 对象所调用的函数则称之为“方法”。

相信聪明的你一定发现了，`this` 对象与作用域肯定存在一定的联系，但是为什么 `obj.getTopName()`的值是 `undefined` 呢？而不是遵循局部作用域在解析标识符时会继续沿着作用域链(scope chain)查找呢？

## 0.2. this 保存在哪里？

`this` 会被保存在作用域的“活动对象(AO)”中，在全局作用域中它被称之为”变量对象(VO)”。
由于函数不属于某个对象的成员属性，所以其活动对象中的 `this` 默认指向的是 `window` 对象（当然这在严格模式下会有不同），因而就会造成函数每次搜索 `this` 的时候，只会搜索到当前活动对象为止，即永远也不会访问外部函数或方法作用域中的 `this`。这也是为什么 `obj.getTopName()` 的值是 `undefined` 了。

## 0.3. this 的一般规则

- 对象方法中的 `this` 引用的是这个对象。
- 函数中的 `this` 引用的是 `window` 对象，因为它属于全局调用。
- 箭头函数的 `this` 是其所处最近一层上下文对象的引用。

## 0.4. this 的注意事项

### 0.4.1. use strict

严格模式下，函数的 `this` 指向的是 `undefined`。

```js
"use strict";
function useStrictThis() {
  console.log(this);
}

useStrictThis(); //undefined
```

### 0.4.2. new 运算符

使用 `new` 运算符调用一个构造函数时，构造函数中的 `this` 指向的是其创建的实例对象。

```js
function Person(name) {
  this.name = name;
  this.age = 27;
}

Person.prototype.sayName = function () {
  return this.name;
};
Person.prototype.sayAge = function () {
  return this.age;
};

//----------------------------
var p1 = new Person("jack");
p1.sayName(); //jack
p1.sayAge(); //27
```

至于为什么构造函数的实例对象还能获取到其构造函数原型对象上的方法或属性，这就是基于原型继承的特点造成的了。
`new` 运算符实际上是一个语法糖，它主要封装了创建对象、关联对象原型、调用构造函数实例化对象以及最终返回该对象的功能。

```js
function New(Constrouct) {
  var obj = Object.create(null);
  obj.__proto__ = Constrouct.prototype;
  Constrouct.call(obj);
  return obj;
}
```

### 0.4.3. 函数和方法

我们已经掌握了函数与方法的定义区分，并且也知道了在非严格模式下函数中的 `this` 默认指向的是 `window` 对象，而严格模式下则保持值为`undefined`。

```js
var _name = "world";
function greet() {
  var _name = "kim";

  function say() {
    console.log(this._name); //world
  }

  say();
}
```

至于对象方法中的 `this` 永远指向的是最后一个点运算符 `.` 所属的对象。

```js
var family = {
  name: "We are a family",
  father: "jack",
  mather: "alis",
  son: {
    name: "kim",
    say() {
      return `my name is ${this.name}`;
    },
  },
  say() {
    return this.name;
  },
};

family.say(); //We are a family
family.son.say(); //my name is kim
```

### 0.4.4. 箭头函数

箭头函数没有 `this`，或者说其 `this` 引用的是当前箭头函数所处作用域中的 `this`，也就是外部环境的上下文。

```js
var obj = {
  greet() {
    var say = () => {
      console.log(this);
    };
    say();
  },
  say: () => {
    console.log(this);
  },
};

obj.greet(); //obj
obj.say(); //window
```

> 需要注意的是 js 是基于词法作用域的。

```js
function Animal(name){
    this.name = name;
}
Animal.prototype.eat = ()=>{
    console.log(this); //window
};
Animal.prototype.say=function(){
    console.log(this); Animal{}
};

var sheep = new Animal('sheep');
sheep.eat();
sheep.say();
```

永远记着:

1. 箭头函数没有 `this`。
2. 箭头函数的 `this` 引用的是外部作用域环境的 `this`。
3. `call`、`apply`、`bind` 等方法无法对箭头函数产生作用。

### 0.4.5. apply、call、bind

`apply,call,bind` 等方法可以修改函数/方法执行时 `this` 的引用。从而人为的指定函数/方法执行时的上下文。

```js
var _name = "world";
var conf = {
  _name: "config",
};
function greet() {
  return `Hello,${this._name}`;
}

greet(); //Hello world
greet.call(conf); //Hello config
```

## 0.5. this 指向的判断步骤

1. 判断函数的调用方式，是函数调用，还是方法调用。
2. 如果是方法调用，那么点 `.` 运算符左边的对象就是 `this` 的引用，否则没有，继续第三步。
3. 函数或方法是否通过 `call`、`apply`、`bind` 等方法调用的，如果是，它们会强制修改 `this` 的引用，如果不是继续进行下一步。
4. 该函数或方法是否充当了构造函数通过 `new` 运算符调用？如果是`this`的引用就是该构造函数的实例对象，如果不是继续下一步。
5. 如果是严格模式下，则 `this` 就是 `undefined`，否则默认指向 `window` 对象。

## 0.6. 其它相关

### 0.6.1. 定时器中的 this

> 就算在严格模式 `use strict` 定时器 `setTimeout` 与 `setInterval` 中的 `this` 都是指向 window 对象。

```js
"use strict";
setTimeout(() => {
  console.log(this); //window
});
```
