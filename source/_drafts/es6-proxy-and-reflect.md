title: es6-proxy-and-reflect
tags: javascript
---

为了更好地支持 meta-programming，ES6 新添加了 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 和 [Reflect](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)。

然而大部分情况下，你都不会用到他们。一方面是因为 meta-programming 本身并不是编程的必须要素，另一方面是因为 Proxy 和 Reflect 不过是**原型链继承**在特定场景下的语法糖而已。

有些你想要**扩展**一个对象的行为，即在原有对象的行为基础上增加一些花样。比如你有一个


