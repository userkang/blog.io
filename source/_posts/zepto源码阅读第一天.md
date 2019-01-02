---
title: zepto源码阅读第一天
date: 2018-06-04 02:00:01
tags: zepto
---

[zepto](zepto) github地址：[git@github.com:madrobby/zepto.git](git@github.com:madrobby/zepto.git)

看到目录结构后，找到 src/zepto.js

1、代码结构：

```js
var Zepto = (function() {...})()

window.Zepto = Zepto
window.$ === undefined && (window.$ = Zepto)
```
这里使用 IDE 折叠代码可以清楚的看到整个代码结构。如果项目中 $ 符被占用，可以使用Zepto来调用。

2、代码入口：

```js
$ = function(selector, context){
   return zepto.init(selector, context)
}
```
平时项目中都是通过 $(xx) 的方式调用的。看来这里是调用了 init 函数，那再看看init函数里是什么吧？

3、zepto.init 函数：

```js
  zepto.init = function(selector, context) {
    var dom
    // 如果selector为空，就返回一个空的Zepto对象
    if (!selector) return zepto.Z()
    // 判断selector是否是字符串
    else if (typeof selector == 'string') {
	  // 过滤空白字符
      selector = selector.trim()
      // 如果是一个html片段，就创建一个Dom节点
      // 注意: 在谷歌21和火狐15版本的浏览器中，如果html片断不是以'<'开始会报异常
      if (selector[0] == '<' && fragmentRE.test(selector))
        dom = zepto.fragment(selector, RegExp.$1, context), selector = null
      // 如果传了上下文的参数，会先创建一个上下文集合，然后在这个集合内去寻找selector节点
      else if (context !== undefined) return $(context).find(selector)
      // 如果是CSS选择器，直接调用原生querySelectorAll去寻找节点(当然qsa方法里面做了很多东西，先这样理解)
      else dom = zepto.qsa(document, selector)
    }
    // 如果传入的是一个方法，则调用Dom ready的方法
    else if (isFunction(selector)) return $(document).ready(selector)
    // 如果本身传入的就是一个Zepto对象，就直接返回这个对象
    else if (zepto.isZ(selector)) return selector
    else {
      // 如果传入的selector是一个节点数组，就格式化下数组，清除里面的空对象
      if (isArray(selector)) dom = compact(selector)
      // 如果传入是一个对象，就把它包成一个数组
      else if (isObject(selector))
        dom = [selector], selector = null
      // 下面的操作和上面的类似，就不多说了
      else if (fragmentRE.test(selector))
        dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null
      else if (context !== undefined) return $(context).find(selector)
      else dom = zepto.qsa(document, selector)
    }
    // 最后将生成的Dom和选择器传入Z函数
    return zepto.Z(dom, selector)
  }
```
接下来就要看看 zepto.Z 函数里面发生了什么。期待。

在 zepto.init 函数中用很多工具方法，都很有意思。比如 fragment、qsa、compact 等。以后再细细展开，今天就先到这啦。



