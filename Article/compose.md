# Compose

## 简介&特点

`compose` 函数可以接收多个独立的函数作为参数，然后将这些函数进行组合串联，最终返回一个“组合函数”。

“组合函数”执行时，其内部的所有函数都会按照组合时的顺序并以队列的形式有序的执行，前一个函数的返回值会作为下一个函数的参数被接收，因此“组合函数”中的第一个执行的函数可以接收多个参数，而之后的函数只能接收一个参数（上一个函数的返回值）。像这样一个或多个指定的参数会从“组合函数”的入口函数（第一个执行的函数）中被传入，而之后则会在多个组合串联的函数管道中进行加工、传输和输出。

组合函数的执行格式以及参数在管道中传输的格式如下：

```js
f(h(j(1, 2)));
```

**compose 函数的特点:**

- 参数是多个函数，返回值是一个“组合函数”。
- 组合函数内的所有的函数从右至左一个一个执行（主要符合数学从右到左的操作概念）。
- 组合函数内除了第一个执行函数的参数是多元的，其它函数的参数都是接收上一个函数的返回值。

**使用形式：**

```js
let sayHello = (...str) => `Hello , ${str.join(" And ")}`;
let toUpper = str => str.toUpperCase();
let combin = compose(
  sayHello,
  toUpper
);

combin("jack", "bob"); // HELLO , JACK AND BOB
```

相比与普通的调用方式，复合函数的方式是不是更简洁？直观？

## 知识点

`compose` 函数的核心就在于如何实现多个函数的组合串联，通过对多种实现方式的分析，在开始实现`compose`函数之前，我们最好事先掌握以下知识点：

**函数嵌套调用**

当多个函数存在嵌套调用是，其执行的顺序是先内后外，一层一层的执行。

```js
a(b(c(d())));
// d() -> c() -> b() -> a()
```

> 整个过程，就如同从内向外一层层的剥洋葱皮。

**闭包**

简单来说，当一个函数作为返回值被另一个函数返回时，并且该函数有访问其父级函数作用域的标识符（变量），那么这个返回值函数就是“闭包函数 (closure)”。

```js
function example() {
  var fnName = arguments.callee.name;
  return function closure() {
    console.log(fnName);
  };
}
```

**递归**

“递归(recursion)”，其实就是函数自己调用自己。常用于多层嵌套但数据结构较为固定的场景，以快速对数据进行处理。

```js
var ids = [];
var data = {
  id: "001",
  child: {
    id: "0002",
    child: {
      id: "0003"
    }
  }
};

function recursion(data) {
  if ("child" in data) {
    ids.push(data.id);
    arguments.callee(data.child);
  }
}

recursion(data);
```

## 实现 & 思路

下面是要被作为参数进行组合的多个函数，之后的示例中都会使用它们进行演示：

```js
function a(a) {
  console.log(a);
  return a + "a";
}
function b(b) {
  console.log(b);
  return b + "b";
}
function c(c) {
  console.log(c);
  return c + "c";
}
var funs = [a, b, c];
```

### 简单的栗子 🌰

`compose` 函数的核心就是实现函数的组合串联，而下面这个例子最能诠释这一过程：

```js
var compose = function(x, y) {
  return function() {
    x(y.apply(this, arguments));
  };
};

compose(
  a,
  b
)(1); //'1ba'
```

虽然这里只是固定的将两个参数(函数)进行串联调用，但是也足以让我们对 `compose`函数的实现有一个整体的了解，我们完全可以基于这种思路来实现多参数组合的`compose`函数。

### 闭包 + 循环的方式

我们将介绍两种通过闭包结合循环来实现 `compose` 函数的方式。
第一种是在循环中通过立即执行函数表达式（IIFE）以闭包的方式实现多个函数的嵌套组合串联。

```js
function compose(funs) {
  var combin = null;
  for (var i = 0; i < funs.length; i++) {
    combin = (function(i, combin) {
      return combin
        ? function(args) {
            return combin(funs[i](args));
          }
        : function(args) {
            return funs[i](args);
          };
    })(i, combin);
  }
  return combin;
}
```

另一种方式，则是与之完全相反，返回的闭包函数中通过循环来顺序执行传入的多个函数，并让上一个函数的返回值作为下一个函数参数。

```js
function compose(funs) {
  var len = funs.length;
  var index = len - 1;
  return function() {
    var result = len ? funs[index].apply(this, arguments) : arguments[0];
    while (--index >= 0) {
      result = funs[index].call(this, result);
    }
    return result;
  };
}
```

### 闭包 + 递归方式

“闭包”结合“递归”的方式，实际上就是利用“递归”的特性来代替循环遍历，它们的原理都是想通的。

下面是我收集到的“闭包”结合“递归”的例子之一，它在闭包中定义一个“立即执行函数表达式”，然后在这个立即执行函数中通过索引值 `n` 来控制“递归”的调用次数，每次递归都会创建一个独立的函数作用域，并且当前的递归都是放在上一个递归函数的调用中，因此多次递归就会生成多重嵌套的函数，但又受限于嵌套函数的执行顺序，所以只有最后一次递归的函数才会是组合函数中第一个被执行的入口函数。

```js
function compose(...arr) {
  return function(...arr2) {
    (function aa(n) {
      if (n < arr.length - 1) {
        return arr[n](aa(++n));
      } else {
        return arr[n](...arr2);
      }
    })(0);
  };
}
```

按照递归的次数，这个组合函数的嵌套形式如下：

```text
step1 : arr[0](aa(++n));
step2 : arr[0](arr[1](aa(++n)))
step3 : arr[0](arr[1](arr[2](...arr2)));
```

这种实现方式实际上还缺少了参数在函数管道中被传输、处理后的最终返回功能，下面是以 ES5 方式进行的改进实现：

```js
function compose(funs) {
  return function() {
    var args = arguments; //引用闭包函数的 arguments
    //返回立即执行函数的结果
    return (function IIFE(n) {
      if (n < funs.length - 1) {
        return funs[n](IIFE(++n));
      } else {
        return funs[n](args[0]);
      }
    })(0);
  };
}
```

下面这种方式跟上面方式的最大区别在于，它并没有通过递归来实现多个函数嵌套然后执行，而是通过递归来一个一个执行，并将上一次执行的结果作为参数传给下一个将要递归执行的函数，一直到递归结束，再返回最终处理好的参数结果。

```js
function compose(funs) {
  var len = funs.length;
  var index = len - 1; //从右向左执行。
  var result; //保存执行或递归执行的结果。
  return function recursion() {
    result = funs[index].apply(this, arguments);
    if (index <= 0) {
      index = len - 1; //条件不满足时，重置递归索引
      return result; //返回最终的结果
    }
    index--;
    return recursion.call(this, result);
  };
}
```

这种实现方式相比上种方式更符合人们的思维习惯，但是也有一个地方需要我们深思，比如闭包函数 `recursion` 为什么需要两次 `return` 呢？

- 第一个 `return` 用于再最后一次递归执行后将最终结果返回并阻止之后的递归执行。
- 第二个 `return` 实际上在每次递归执行的函数中都存在，用于一层层返回参数，直到将结果返回到闭包函数之外。

该示例的执行流程如下：

```text
-----------------------------
////////// step1 : //////////
-----------------------------

//正常执行结束，开始进入递归，这里的return用于将结果返回出闭包外。
//进入递归后，最外层将会暂停，等待内层的递归执行完毕后才会返回结果。
    return recursion.call(this, '1c');

-----------------------------
////////// step2 : //////////
-----------------------------

//递归执行，开始进入第二层递归，这里的return用于将结果返回到上一层
return recursion.call(this, '1cb');

-----------------------------
////////// step3: //////////
-----------------------------
//最后一次递归执行。
if (index <= 0) {
    //重置递归的索引控制
    index = len - 1;
    //终止之后的递归执行，并将结果返回到上一层递归函数中。
    return result;
}
```

### 更简洁的 ES6 方式。

```js
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg;
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
```

ES6 方式的重点就是利用了数组的 `reduce` 合并功能， 每次遍历合并都会将上一次组合后的函数返回回来再与当前的函数参数进行组合。依次不断的累积组合，最终返回这个组合函数。

```text
-----------------------------
////////// step1 : //////////
-----------------------------
a = f(a) ; b= f(b);
返回值: (...args) => a(b(...args))

-----------------------------
////////// step2 : //////////
-----------------------------
a = (...args) => a(b(...args)); b = f(c);
返回值: (...args) => ((...args) => a(b(...args))(c(...args)))
```

从上面的执行步骤中，我们可以发现，最右边的函数会在组合的最外层，这样便能保证从右向左执行的顺序。

下面则是对 ES6 方式的稍微改变：

```js
function compose(...funs) {
  if (funs.length === 0) {
    return arg => arg;
  }
  if (funs.length === 1) {
    return funs[0];
  }

  return funs.reverse().reduce((a, b) => (...arg) => b(a(arg)));
}
```

这个示例的变动主要有两点：一个是将“函数数组(funs)”进行了颠倒(reverse)从而最右边的函数被最先被遍历，第二个则是颠倒了函数的组合顺序，每次遍历都是用当前的函数来包裹组合后的函数，因此它看上去是一层层的通过包裹嵌套`a(b(c(...args)))`来实现函数的组合，也只有这样，才能保证最先遍历的函数在组合函数的最里层，从而按照嵌套函数的执行顺序，能够被最先执行(保证从右向左的执行顺序)。

```text
-----------------------------
////////// step1 : //////////
-----------------------------
a = f(c) ; b= f(b);
返回值: (...args) => b(c(...args))

-----------------------------
////////// step2 : //////////
-----------------------------
a = (...args) => b(c(...args)); b = f(a);
返回值: (...arg) => (a((...args) => b(c(...args))(...args)))
```

## pipe 与 compose

`pipe` 函数与 `compose`函数的共同点是都返回“组合函数”，区别则是执行的顺序不同，前者是从左向右执行，后者则是从右向左执行。
实现 `pipe`函数非常简单，只需要对 `compose` 函数的包裹顺序进行调整一下即可。

## 参考

> https://segmentfault.com/a/1190000008394749
