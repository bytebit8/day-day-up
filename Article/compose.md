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

## 参考

> https://segmentfault.com/a/1190000008394749
