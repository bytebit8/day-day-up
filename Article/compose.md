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

整个过程，就如同从内一层一层的剥洋葱皮。

**闭包**
简单来说，当一个函数作为返回值被另一个函数返回时，并且作为返回值的函数有访问其父级函数作用域的标识符（变量），那么这个返回值函数也就是“闭包函数 (closure)”。

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

假设作为参数的多个函数如下：

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

根据 `compose`函数的核心定义，即实现函数的组合串联，我们可以通过一个最简单的例子来直观了解

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

虽然只是将两个固定的参数函数进行串联调用，但是也足以让我们对 `compose`函数的整体实现有一个直观的了解。

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

这种方式的思路则是通过“递归”的特性来替代循环，但是编码的思路依然是想通的。

```js
function compose(...arr) {
    return function (...arr2) {
        (function aa(n) {
            if (n < arr.length - 1) {
                return arr[n](<aa(++n) >)
            } else {
                return arr[n](...arr2);
            }
        })(0)
    }
}
```

```js
var compose = function(...fns) {
  var len = fns.length; // 记录我们传入所有函数的个数
  var index = len - 1; // 游标记录函数执行情况, 也作为我们运行 fns 中的中函数的索引
  var reslut; // 结果, 每次函数执行完成后, 向下传递
  return function f1(...arg1) {
    reslut = fns[index].apply(this, arg1);
    if (index <= 0) {
      index = len - 1;
      return reslut;
    }
    --index;
    return f1.call(null, reslut);
  };
};
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

稍微改变一下：

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

## pipe

前面我们提到过，“组合函数”在执行时，其内部的所有函数会按照组合的顺序以队列的形式有序的执行，因此很明显 `compose` 函数返回的组合函数，其顺序是从右至左，而 `pipe` 函数则与其完全相反，它返回的组合函数其执行顺序是从左至右，更符合人们的习惯。

实现 `pipe`函数非常简单，只需要对 `compose` 函数的包裹顺序进行调整一下即可。

## 参考

> https://segmentfault.com/a/1190000008394749
