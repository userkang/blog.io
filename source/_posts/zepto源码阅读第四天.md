---
title: zepto源码阅读第四天
date: 2018-06-22 00:53:12
tags: zepto
---

今天把 $.fn 的其余函数一起列出来哈。

```js
get: function(idx){
   // 可选，不传时，将Zetpo转换成数组
   return idx === undefined ? slice.call(this) : this[idx >= 0 ? idx : idx + this.length]
 },
 toArray: function(){ return this.get() },
 size: function(){
   return this.length
 },
 remove: function(){
   return this.each(function(){
     if (this.parentNode != null)
       this.parentNode.removeChild(this)
   })
 },
 // 遍历集合，将集合中的每一项放入callback中进行处理，如果callback返回false，那么就会停止循环了
 each: function(callback){
   emptyArray.every.call(this, function(el, idx){
     return callback.call(el, idx, el) !== false
   })
   return this
 },
 filter: function(selector){
   // this.not(selector)取到需要排除的集合，第二次再取反(这个时候this.not的参数就是一个集合了)，得到想要的集合
   if (isFunction(selector)) return this.not(this.not(selector))

   // 原生filter如果返回true就收集
   return $(filter.call(this, function(element){
     // 如果匹配就返回true
     return zepto.matches(element, selector)
   }))
 },
 add: function(selector,context){
   // 合并后并保证唯一
   return $(uniq(this.concat($(selector,context))))
 },
 is: function(selector){
   return typeof selector == 'string' ? this.length > 0 && zepto.matches(this[0], selector) : 
       selector && this.selector == selector.selector
 },
 not: function(selector){
   var nodes=[]
   // 当selector为函数时，safari下的typeof nodeList也是function，所以这里需要再加一个判断selector.call !== undefined
   if (isFunction(selector) && selector.call !== undefined)
     this.each(function(idx){
       // 注意这里收集的是selector.call(this,idx)返回结果为false的时候记录
       if (!selector.call(this,idx)) nodes.push(this)
     })
   else {
     // 当selector为nodeList时执行slice.call(selector),注意这里的isFunction(selector.item)是为了排除selector为数组的情况
     // 当selector为css选择器，执行$(selector)
     var excludes = typeof selector == 'string' ? this.filter(selector) :
       (likeArray(selector) && isFunction(selector.item)) ? slice.call(selector) : $(selector)
     this.forEach(function(el){
       if (excludes.indexOf(el) < 0) nodes.push(el)
     })
   }
   // 由于上面得到的结果是数组，这里需要转成zepto对象，以便继承其它方法，实现链写
   return $(nodes)
 },
 /*
   接收node和string作为参数，给当前集合筛选出包含selector的集合
   isObject(selector)是判断参数是否是node，因为typeof node == 'object'
   当参数为node时，只需要判读当前记当里是否包含node节点即可
   当参数为string时，则在当前记录里查询selector，如果长度为0，则为false，filter函数就会过滤掉这条记录，否则保存该记录
   */
 has: function(selector){
   return this.filter(function(){
     return isObject(selector) ?
       $.contains(this, selector) :
       $(this).find(selector).size()
   })
 },
 eq: function(idx){
   // 取Zepto中的指定索引的元素，再包装成Zepto返回
   return idx === -1 ? this.slice(idx) : this.slice(idx, + idx + 1)
 },
 first: function(){
   var el = this[0]
   // 这里不知道什么时候isObject(el)为false，el还可以是什么类型?
   return el && !isObject(el) ? el : $(el)
 },
 last: function(){
   var el = this[this.length - 1]
   return el && !isObject(el) ? el : $(el)
 },
 find: function(selector){
   var result, $this = this
   if (!selector) result = $()
   // 如果selector为对象，遍历selector，筛选出父级为集合中记录的selector
   else if (typeof selector == 'object')
     result = $(selector).filter(function(){
       var node = this
       // 如果$.contains(parent, node)返回true，则emptyArray.some也会返回true,外层的filter则会收录该条记录
       return emptyArray.some.call($this, function(parent){
         return $.contains(parent, node)
       })
     })
   // 如果selector是css选择器
   // 如果当前集合长度为1时，调用zepto.qsa，将结果转成zepto对象
   else if (this.length == 1) result = $(zepto.qsa(this[0], selector))
   // 如果长度大于1，则调用map遍历
   else result = this.map(function(){ return zepto.qsa(this, selector) })
   return result
 },
 closest: function(selector, context){
   var nodes = [], collection = typeof selector == 'object' && $(selector)
   this.each(function(_, node){
     while (node && !(collection ? collection.indexOf(node) >= 0 : zepto.matches(node, selector)))
       node = node !== context && !isDocument(node) && node.parentNode
     if (node && nodes.indexOf(node) < 0) nodes.push(node)
   })
   return $(nodes)
 },
 parents: function(selector){
   var ancestors = [], nodes = this
   while (nodes.length > 0)
     nodes = $.map(nodes, function(node){
       if ((node = node.parentNode) && !isDocument(node) && ancestors.indexOf(node) < 0) {
         ancestors.push(node)
         return node
       }
     })
   return filtered(ancestors, selector)
 },
 parent: function(selector){
   return filtered(uniq(this.pluck('parentNode')), selector)
 },
 children: function(selector){
   return filtered(this.map(function(){ return children(this) }), selector)
 },
 contents: function() {
   return this.map(function() { return this.contentDocument || slice.call(this.childNodes) })
 },
 siblings: function(selector){
   return filtered(this.map(function(i, el){
     // 到其父元素取得所有子节点，再排除本身
     return filter.call(children(el.parentNode), function(child){ return child!==el })
   }), selector)
 },
 empty: function(){
   return this.each(function(){ this.innerHTML = '' })
 },
 // `pluck` is borrowed from Prototype.js
 pluck: function(property){
   return $.map(this, function(el){ return el[property] })
 },
 show: function(){
   return this.each(function(){
     // 清除内联样式display="none"
     this.style.display == "none" && (this.style.display = '')
     // 计算样式display为none时，重赋显示值
     if (getComputedStyle(this, '').getPropertyValue("display") == "none")
       // defaultDisplay是获取元素默认display的方法
       this.style.display = defaultDisplay(this.nodeName)
   })
 },
 replaceWith: function(newContent){
   // 将要替换内容插到被替换内容前面，然后删除被替换内容
   return this.before(newContent).remove()
 },
 wrap: function(structure){
   var func = isFunction(structure)
   if (this[0] && !func)
     var dom   = $(structure).get(0),
         clone = dom.parentNode || this.length > 1

   return this.each(function(index){
     $(this).wrapAll(
       func ? structure.call(this, index) :
         clone ? dom.cloneNode(true) : dom
     )
   })
 },
 wrapAll: function(structure){
   if (this[0]) {
     $(this[0]).before(structure = $(structure))

     var children
     // drill down to the inmost element
     while ((children = structure.children()).length) structure = children.first()
     $(structure).append(this)
   }
   return this
 },
 wrapInner: function(structure){
   var func = isFunction(structure)
   return this.each(function(index){
     var self = $(this), contents = self.contents(),
         dom  = func ? structure.call(this, index) : structure
     contents.length ? contents.wrapAll(dom) : self.append(dom)
   })
 },
 unwrap: function(){
   this.parent().each(function(){
     $(this).replaceWith($(this).children())
   })
   return this
 },
 clone: function(){
   return this.map(function(){ return this.cloneNode(true) })
 },
 hide: function(){
   return this.css("display", "none")
 },
 toggle: function(setting){
   return this.each(function(){
     var el = $(this)
     ;(setting === undefined ? el.css("display") == "none" : setting) ? el.show() : el.hide()
   })
 },
 prev: function(selector){ return $(this.pluck('previousElementSibling')).filter(selector || '*') },
 next: function(selector){ return $(this.pluck('nextElementSibling')).filter(selector || '*') },
 html: function(html){
   return 0 in arguments ?
     this.each(function(idx){
       var originHtml = this.innerHTML
       $(this).empty().append( funcArg(this, html, idx, originHtml) )
     }) :
     (0 in this ? this[0].innerHTML : null)
 },
 text: function(text){
   return 0 in arguments ?
     this.each(function(idx){
       var newText = funcArg(this, text, idx, this.textContent)
       this.textContent = newText == null ? '' : ''+newText
     }) :
     (0 in this ? this.pluck('textContent').join("") : null)
 },
 attr: function(name, value){
   var result
   return (typeof name == 'string' && !(1 in arguments)) ?
     // 仅有name，且为字符串时，表示读，直接用getAttribute(name)读
     (0 in this && this[0].nodeType == 1 && (result = this[0].getAttribute(name)) != null ? result : undefined) :
     this.each(function(idx){
       if (this.nodeType !== 1) return
       // 如果name为对象，批量设置属性
       if (isObject(name)) for (key in name) setAttribute(this, key, name[key])
       // 处理value为函数 null/undefined 的情况
       else setAttribute(this, name, funcArg(this, value, idx, this.getAttribute(name)))
     })
 },
 removeAttr: function(name){
   return this.each(function(){ this.nodeType === 1 && name.split(' ').forEach(function(attribute){
     setAttribute(this, attribute)
   }, this)})
 },
 prop: function(name, value){
   name = propMap[name] || name
   return (typeof name == 'string' && !(1 in arguments)) ?
     (this[0] && this[0][name]) :
     this.each(function(idx){
       if (isObject(name)) for (key in name) this[propMap[key] || key] = name[key]
       else this[name] = funcArg(this, value, idx, this[name])
     })
 },
 removeProp: function(name){
   name = propMap[name] || name
   return this.each(function(){ delete this[name] })
 },
 data: function(name, value){
   var attrName = 'data-' + name.replace(capitalRE, '-$1').toLowerCase()

   var data = (1 in arguments) ?
     this.attr(attrName, value) :
     this.attr(attrName)

   return data !== null ? deserializeValue(data) : undefined
 },
 val: function(value){
   if (0 in arguments) {
     if (value == null) value = ""
     return this.each(function(idx){
       this.value = funcArg(this, value, idx, this.value)
     })
   } else {
     return this[0] && (this[0].multiple ?
        $(this[0]).find('option').filter(function(){ return this.selected }).pluck('value') :
        this[0].value)
   }
 },
 offset: function(coordinates){
   // 写入坐标
   if (coordinates) return this.each(function(index){
     var $this = $(this),
         coords = funcArg(this, coordinates, index, $this.offset()),
         parentOffset = $this.offsetParent().offset(),
         props = {
           top:  coords.top  - parentOffset.top,
           left: coords.left - parentOffset.left
         }

     if ($this.css('position') == 'static') props['position'] = 'relative'
     $this.css(props)
   })
   // 读取坐标 取第一个元素的坐标
   if (!this.length) return null
   // 如果父元素是document
   if (document.documentElement !== this[0] && !$.contains(document.documentElement, this[0]))
     return {top: 0, left: 0}
   // 读取到元素相对于页面视窗的位置
   var obj = this[0].getBoundingClientRect()
   return {
     left: obj.left + window.pageXOffset,
     top: obj.top + window.pageYOffset,
     width: Math.round(obj.width),
     height: Math.round(obj.height)
   }
 },
 css: function(property, value){
   // 只有一个参数的时候，只读
   if (arguments.length < 2) {
     var element = this[0]
     if (typeof property == 'string') {
       if (!element) return
       // getComputedStyle是一个可以获取当前元素所有最终使用的CSS属性值。返回的是一个CSS样式声明对象([object CSSStyleDeclaration])，只读
       return element.style[camelize(property)] || getComputedStyle(element, '').getPropertyValue(property)
     } else if (isArray(property)) {
       if (!element) return
       var props = {}
       var computedStyle = getComputedStyle(element, '')
       $.each(property, function(_, prop){
         props[prop] = (element.style[camelize(prop)] || computedStyle.getPropertyValue(prop))
       })
       return props
     }
   }

   var css = ''
   if (type(property) == 'string') {
     if (!value && value !== 0)
       // dasherize 是将字符串转换成css属性(background-color格式）
       this.each(function(){ this.style.removeProperty(dasherize(property)) })
     else
       css = dasherize(property) + ":" + maybeAddPx(property, value)
   } else {
     // property 是一个对象
     for (key in property)
       // 当property[key]的值为 null/undefined，删除属性
       if (!property[key] && property[key] !== 0)
         this.each(function(){ this.style.removeProperty(dasherize(key)) })
       else
         css += dasherize(key) + ':' + maybeAddPx(key, property[key]) + ';'
   }

   return this.each(function(){ this.style.cssText += ';' + css })
 },
 index: function(element){
   // 这里的$(element)[0]是为了将字符串转成node,因为this是个包含node的数组
   // 当不指定element时，取集合中第一条记录在其父节点的位置
   // this.parent().children().indexOf(this[0])这句很巧妙，和取第一记录的parent().children().indexOf(this)相同
   return element ? this.indexOf($(element)[0]) : this.parent().children().indexOf(this[0])
 },
 hasClass: function(name){
   if (!name) return false
   // some ES5的新方法 有一个匹配，即返回true
   return emptyArray.some.call(this, function(el){
     // this是classRE(name)生成的正则
     return this.test(className(el))
   }, classRE(name))
 }
```

```js
addClass: function(name){
   if (!name) return this
   return this.each(function(idx){
     // 已存在，返回
     if (!('className' in this)) return
     classList = []
     var cls = className(this), newName = funcArg(this, name, idx, cls)
     // 多个类，空格分隔为数组
     newName.split(/\s+/g).forEach(function(klass){
       if (!$(this).hasClass(klass)) classList.push(klass)
     }, this)
     // 设值
     classList.length && className(this, cls + (cls ? " " : "") + classList.join(" "))
   })
 },
 removeClass: function(name){
   return this.each(function(idx){
     if (!('className' in this)) return
     if (name === undefined) return className(this, '')
     classList = className(this)
     funcArg(this, name, idx, classList).split(/\s+/g).forEach(function(klass){
       classList = classList.replace(classRE(klass), " ")
     })
     className(this, classList.trim())
   })
 },
 toggleClass: function(name, when){
   if (!name) return this
   return this.each(function(idx){
     var $this = $(this), names = funcArg(this, name, idx, className(this))
     names.split(/\s+/g).forEach(function(klass){
       // when 通过true/false控制
       (when === undefined ? !$this.hasClass(klass) : when) ?
         $this.addClass(klass) : $this.removeClass(klass)
     })
   })
 },
 scrollTop: function(value){
   if (!this.length) return
   var hasScrollTop = 'scrollTop' in this[0]
   if (value === undefined) return hasScrollTop ? this[0].scrollTop : this[0].pageYOffset
   return this.each(hasScrollTop ?
     // 支持scrollTop，直接赋值
     function(){ this.scrollTop = value } :
     // 不支持scrollTop，滚到指定坐标  pageYOffset 属性是 scrollY 属性的别名 pageYOffset兼容更好
     function(){ this.scrollTo(this.scrollX, value) })
 },
 scrollLeft: function(value){
   if (!this.length) return
   var hasScrollLeft = 'scrollLeft' in this[0]
   if (value === undefined) return hasScrollLeft ? this[0].scrollLeft : this[0].pageXOffset
   return this.each(hasScrollLeft ?
     function(){ this.scrollLeft = value } :
     function(){ this.scrollTo(value, this.scrollY) })
 },
 position: function() {
   if (!this.length) return

   var elem = this[0],
     // Get *real* offsetParent
     // 读到父元素
     offsetParent = this.offsetParent(),
     // Get correct offsets
     // 读到坐标
     offset       = this.offset(),
     // 读到父元素的坐标
     parentOffset = rootNodeRE.test(offsetParent[0].nodeName) ? { top: 0, left: 0 } : offsetParent.offset()

   // Subtract element margins
   // note: when an element has margin: auto the offsetLeft and marginLeft
   // are the same in Safari causing offset.left to incorrectly be 0
   // 坐标减去外边框
   offset.top  -= parseFloat( $(elem).css('margin-top') ) || 0
   offset.left -= parseFloat( $(elem).css('margin-left') ) || 0

   // Add offsetParent borders
   // 加上父元素的border
   parentOffset.top  += parseFloat( $(offsetParent[0]).css('border-top-width') ) || 0
   parentOffset.left += parseFloat( $(offsetParent[0]).css('border-left-width') ) || 0

   // Subtract the two offsets
   return {
     top:  offset.top  - parentOffset.top,
     left: offset.left - parentOffset.left
   }
 },
 offsetParent: function() {
 /**
  * 返回第一个匹配元素用于定位的祖先元素
  * 原理：读取父元素中第一个其position设为relative或absolute的可见元素
  */
   return this.map(function(){
     // 读取定位父元素，没有，则body
     var parent = this.offsetParent || document.body
     // 如果找到的定位元素  position='static'继续往上找，直到body/Html
     while (parent && !rootNodeRE.test(parent.nodeName) && $(parent).css("position") == "static")
       parent = parent.offsetParent
     return parent
   })
 }
```