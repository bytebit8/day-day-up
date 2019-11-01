## 说明

使用 `void 0` 来替代 `undefined` 的优势：

- 节省字节数
- 规避 `undefined` 会被重写的缺陷。

由于 `undefined` 不是一个“保留字”也不是一个“关键字”，它只是全局对象 `window` 的一个属性， 在 IE 低版本中会被重写

```js
// for IE8-
var undefined = 1;
alert(undefined); //1
undefined = 2;
alert(undefined); //2
```

在 ECMAScript 5 中，虽然 `undefined` 属于全局对象的 **（只读）** 属性，不能被重写，但在局部作用域中仍然能被重写。

```js
function test() {
  var undefined = 1;
  console.log(undefined); //1
}
test();
```

## void 运算符

根据 MDN 的说明，`void` 运算符对任何表达式进行求值，返回的都是 `undefined`。

```js
void 0; //undefined;
void 1; //undefined;
```

所以使用 `void` 可以完美替代全局对象的 `undefined` 属性。
另外 `void` 运算符还可以用于立即执行函数：

```js
void (function() {
  console.log("IIFE");
})();
```

原理就在于 `void` 运算符会对表达式求值，所以这里的函数声明语句就会变成一个“函数表达式”，只有函数表达式才能与之紧临的一对执行圆括符结合，实现立即执行的特点。

> 更多关于函数立即执行的说明，[访问这里](./immediately-invoked-function-expression.md)。

## 代码

```js
//bad
if (k === undefined) {
}

//good
if (k === void 0) {
}
```

## 参考

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void
