# htmlEscape (HTML 标记转义)

## 简介

用于简单的防 XSS 攻击场景。

## 思路

对 HTML 的关键字符进行转义。

| 关键字符 | 实体名称  | 十六进制编码 |
| :------: | :-------: | :----------: |
|    &     |  \&amp;   |      /       |
|    <     |   \&lt;   |      /       |
|    >     |   \&gt;   |      /       |
|    '     | 　\&apos; |    \&#39;    |
|    "     |  \&quot;  |      /       |

> IE 低版本对 ``` 的实体名称存在兼容性问题，因此改用对应的十六进制编码。

## 实现

```js
function htmlEscape(str) {
  if (typeof str != "string") return str;
  if (str.length === 0) return str;

  str = str.replace(/&/gm, "&amp;");
  str = str.replace(/</gm, "&lt;");
  str = str.replace(/>/gm, "&gt;");
  str = str.replace(/"/gm, "&quot;");
  str = str.replace(/'/gm, "&#39;");

  return str;
}
```

## 注意

这个 demo 主要用于练习和验证使用，对于安全性要求比较高的项目，还是建议使用专业的防 XSS 过滤库。

推荐的防XSS攻击库有：

* [JS-XSS](https://github.com/leizongmin/js-xss)