## 说明

使用 `void 0` 来替代 `undefined` 的优势：

- 节省字节数
- 规避 `undefined` 会被重写的缺陷。

由于 `undefined` 不是一个保留字也不是一个关键字，它只是全局对象 `window` 的一个属性， 在 IE 低版本中会被重写

```js
// for IE8-
var undefined = 1;
alert(undefined); //1
undefined = 2;
alert(undefined); //2
```

在 ECMAScript 5 中，虽然 `undefined` 属于全局对象的**（只读）**属性，不能被重写，但在局部作用域中仍然能被重写。

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

所以使用 `void` 可以完美替代直接使用全局对象的 `undefined` 属性。
另外 `void`

## 参考

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void
