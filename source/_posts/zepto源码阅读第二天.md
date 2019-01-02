---
title: zepto源码阅读第二天
date: 2018-06-10 22:11:21
tags: zepto
---

上次说到了zepto.Z函数，下面继续哈。

一、zepto.Z 函数

```js
function Z(dom, selector) {
  var i, len = dom ? dom.length : 0
  for (i = 0; i < len; i++) this[i] = dom[i]
  this.length = len   // length属性在这里赋值
  this.selector = selector || ''
}

// `zepto.Z` 用 `$.fn` 替换了自己的 prototype 属性
// 这样实例化的dom对象就拥有了挂载在$.fn的方法
zepto.Z = function(dom, selector) {
  return new Z(dom, selector)
}
```
这里的 `Z` 函数是一个内部方法，是不希望暴露出来被修改的。

二、$.fn 函数

上面说到 `zepto.Z` 用 `$.fn` 替换了自己的 prototype 属性，代码在这里：

```js
zepto.Z.prototype = Z.prototype = $.fn
```
如果要实现让实例对象拥有 `$.fn` 上的方法，其实这里用 `Z.prototype = $.fn` 就可以实现了,这里的 zepto.Z.prototype 有什么意义呢？在 `v1.1.6` 版之前因为还没有 `Z` 函数，当时的版本里是通过将实例的 `__proto__`  直接指向 `$.fn` 来实现继承的。
所以当时的 `isZ` 函数内部用 `zepto.Z` 来判断一个实例是否是 `zepto` 对象，这样是很好理解的：

```js
zepto.isZ = function(object) {
  return object instanceof zepto.Z
}
```
那么现在有了 `Z` 函数，这里可以改成 `object instanceof Z` 了吧。

至于为什么还保留 `zepto.Z.prototype` 我有几个猜想哈

1、历史原因 

或许之前有很多人这样用？

```js
$().__proto__ === $.zepto.Z.prototype
```
在 `Z` 函数没有暴露的情况下，不能使用 `$().__proto__ === $.Z.prototype`  这样的操作。

二、本来就是这样设计的
```js
$.fn = {
  constructor: zepto.Z,
  length: 0,
}

```
这里可以看到 `$.fn` 将 constructor 重新指向 zepto.Z 构造函数，而不是 Z。也可以看出设计本身就希望 Z 仅仅是一个工厂方法，引入它是希望优化掉 `__proto__` 这种不优雅的写法，不想改动原有的设计。
   
或许 `zepto.Z.prototype` 的存在还有其他原因吧，我上面说的就是记录下来，等待未来的某一天打脸用的。哈哈。

有些跑偏了，下篇还是继续来看 `$.fn` 函数吧。

