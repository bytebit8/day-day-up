# Compose

## 简介&特点

`compose` 函数接收多个独立的函数作为参数，然后将这些函数进行组合串联，最终返回一个“组合函数”。
“组合函数”执行时，其内部所有函数都会按照组合时的顺序，以队列的形式有序的执行，上一个执行完函数的返回值会作为下一个将要执行的函数的参数。

```js
tasks = [task1,task2,task3,task4,...];
```

**compose 函数的特点:**

- 参数是多个函数，返回值是一个组合函数
- 组合函数内的所有的函数从右至左一个一个执行。
- 除了第一个执行函数的参数是多元的，其它函数的参数都是接收上一个函数的返回值。

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

## 知识点

实现一个 `compose` 函数，最好事先具备以下知识点：

**函数嵌套调用**
嵌套函数调用顺序是由内到外一层一层执行。

a(b(c(d())))
-> d() -> c() -> b() -> a()

**闭包**
**递归**

## 实现

function compose(funs) {
    var combin = null;
    for (var i = 0; i < funs.length; i++) {
        combin = (function (i, combin) {
            return combin ? function () { combin(funs[i]()) } : function () {
                funs[i]()
            }
        }(i, combin))
    }
    return combin;
}

var flow = function(fns) {
    var len = fns.length
    var index = len
    return function(...args) {
        var index = 0
        var reslut = len ? fns[index].apply(this, args) : args[0]
        while (++index < len) {
            reslut = fns[index].call(this, reslut)
        }
        return reslut
    }
}


function compose(...arr) {
    return function(...arr2) {
        (function aa(n) {
            if (n < arr.length - 1) {
                return arr[n](aa(++n))
            } else {
                return arr[n](...arr2);
            }
        })(0)
    }
}


var compose = function(...fns) {
    var len = fns.length // 记录我们传入所有函数的个数
    var index = len - 1 // 游标记录函数执行情况, 也作为我们运行fns中的中函数的索引
    var reslut // 结果, 每次函数执行完成后, 向下传递
    return function f1(...arg1) {
        reslut = fns[index].apply(this, arg1)
        if (index <= 0) {
            index = len - 1 
            return reslut
        }
        --index
        return f1.call(null, reslut)
    }
}

function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }

    return funcs.reduce((a, b) => (...args) => a(b(...args)))
}

function compose(...funs) {
    if (funs.length === 0) {
        return arg => arg
    }
    if (funs.length === 1) {
        return funs[0]
    }

    return funs.reverse().reduce((a, b) => (...arg) => b(a(arg)));
}

## 参考

> https://segmentfault.com/a/1190000008394749
