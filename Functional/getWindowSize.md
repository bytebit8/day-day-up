# getWindowSize (获取浏览器窗口尺寸)

## 说明

获取浏览器窗口 `window` 的尺寸。
由于 PC 端物理像素与逻辑像素比值通常为 1，所以 window 的尺寸便是“视图窗口(viewport)”的尺寸。

## 思路

对于低版本浏览器，可以通过 `html` 或 `body` 元素的客户区大小来间接获取视图窗口的尺寸。

## 实现

```js
function getWindowSize() {
  var docElement =
    document.scrollingElement || document.documentElement || document.body;
  return {
    width:
      window.innerWidth ||
      docElement.clientWidth ||
      docElement.clientWidth ||
      docElement.clientWidth,
    height:
      window.innerHeight ||
      docElement.clientHeight ||
      docElement.clientHeight ||
      docElement.clientHeight
  };
}
```

## 知识点

- scrollingElement : 用于解决 `documentElement` 与 `body` 属性的兼容性问题。

## 拓展

- `innerWidth/innerHeight` 还对应着一对 `outerWidth/outerHeight`属性，在我个人理解中，它们是用于获取浏览器本身的尺寸，但 chrome 与 Firefox 和 IE 存在差异，并且也比较少能使用到。
- `window.resizeTo()` 以及 `window.resizeBy()` 这两个方法可以控制弹出窗口的尺寸(opener)。
