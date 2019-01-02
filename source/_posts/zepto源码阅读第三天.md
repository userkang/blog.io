---
title: zepto源码阅读第三天
date: 2018-06-11 03:56:36
tags: zepto
---

今天准备把 `$.fn` 上的函数对照 API 文档过一下。由于函数过多，就不一一记录了，就把觉得重要的记录下来吧。

zepto API  在线文档地址：

英文版：[http://zeptojs.com/](http://zeptojs.com/)
中文版：[http://www.css88.com/doc/zeptojs_api/](http://www.css88.com/doc/zeptojs_api/)

1、直接继承原声js的几个方法

```
forEach: emptyArray.forEach,
reduce: emptyArray.reduce,
push: emptyArray.push,
sort: emptyArray.sort,
splice: emptyArray.splice,
indexOf: emptyArray.indexOf,
```

2、concat

```js
concat: function(){
   var i, value, args = []
   for (i = 0; i < arguments.length; i++) {
     value = arguments[i]
     args[i] = zepto.isZ(value) ? value.toArray() : value
   }
   return concat.apply(zepto.isZ(this) ? this.toArray() : this, args)
}
```
如果发现 value 是 zepto 对象，就把它转成数组。`toArray` 函数内部大概用 `[].slice.call()` 的方式把类数组转为数组，ES6中已经有 `Array.from` 方法可以实现同样的功能了哈。

3、map

```js
map: function(fn){
	return $($.map(this, function(el, i){ return fn.call(el, i, el) }))
}

$.map = function(elements, callback){
   console.log(callback)
   var value, values = [], i, key
   if (likeArray(elements))
     for (i = 0; i < elements.length; i++) {
       value = callback(elements[i], i)
       if (value != null) values.push(value)
     }
   else
     for (key in elements) {
       value = callback(elements[key], key)
       if (value != null) values.push(value)
     }
   return flatten(values)
 }
```
这里 `$.map(this, function(el, i){ return fn.call(el, i, el) })` 得注意下

情况一： `$.map([], function(value, key))` ，`value` 在前，`key` 在后。
情况二： `$('xx').map(function(key, value))` 这种的，`key` 在前，`value` 在后。
`fn.call(el, i, el)` 看这就可以知道，将参数做了调换。第一个 `el` 是上下文。后面两个参数就是 `key 和 value` 的值了。

4、ready

```js
ready: function(callback){
     // don't use "interactive" on IE <= 10 (it can fired premature)
     if (document.readyState === "complete" ||
         (document.readyState !== "loading" && !document.documentElement.doScroll))
       setTimeout(function(){ callback($) }, 0)
     else {
       var handler = function() {
         document.removeEventListener("DOMContentLoaded", handler, false)
         window.removeEventListener("load", handler, false)
         callback($)
       }
       document.addEventListener("DOMContentLoaded", handler, false)
       window.addEventListener("load", handler, false)
     }
     return this
}
```
文档加载的几个状态：

>“uninitialized” – 原始状态
>“loading” – 下载数据中..
>“loaded” – 下载完成
>“interactive” – 还未执行完毕.
>“complete” – 脚本执行完毕

大概的执行顺序：

- document.readyState 为 loading 
- 触发DOMContentLoaded事件，监听DOMContentLoaded要执行的函数
- document.documentElement.doScroll （IE）
- document.readyState 为compelete 
- onload函数（资源加载完毕）

这里为了兼容，做了很多处理。目的是为了在DOM树构建完成的第一时间就能执行代码。
如果支持 DOMContentLoaded 事件，那最好不过。
如果不支持（IE），就只能通过检测 doScroll 或者 监听文档加载的状态来判断页面DOM树是否已经构建完成。

简单写个例子验证了下：
```js
document.addEventListener('readystatechange', function() {
	if (document.readyState === 'loading') {
		console.log('loading');
	}

	if (document.readyState === 'complete') {
	    console.log('complete');
	}
});

window.addEventListener('DOMContentLoaded', function() {
    console.log('DOMContentLoaded')
})

window.addEventListener('load', function () {
    console.log('load');
})
```
今天就到这了，下篇继续。
