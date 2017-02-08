---
layout: post
title:  "Bootstrap alert.js分析"
date:   2017-02-07 19:00:05 +0800
categories: bootstrap
---

查看[bootstrap-3.3.7 alert.js源码][alert-js]

alert.js的代码组织如下:

#### 自调用函数 ####
```javascript
+function ($) {
}(jQuery);
```
* 通过一个自调用函数，创建一个函数域空间，在该私有的域空间内定义Alert
* 传入jQuery参数，是为了得到一个函数局部的$符, 来标记jQuery. 因为全局的$符标记的不一定是jQuery, 有可能其他的js库也定义了$符.
* function前面的+号，是为了让js解析器将+号后面的部分识别为expression, 这样才可以进行函数调用。可以用其他符号，比如 - , ! 等等. 参考[自调用函数定义][so-1]

#### 定义Alert ####
```javascript
// ALERT CLASS DEFINITION
// ======================
var dismiss = '[data-dismiss="alert"]'
var Alert   = function (el) {
  $(el).on('click', dismiss, this.close)
}
Alert.VERSION = '3.3.7'
Alert.TRANSITION_DURATION = 150
Alert.prototype.close = function (e) {
  // close具体实现
}
```

#### 定义Alert Plugin & 暴露Alert和Alert Plugin ####
```javascript
// ALERT PLUGIN DEFINITION
// =======================
function Plugin(option) {
  return this.each(function () {
      var $this = $(this)
      var data  = $this.data('bs.alert')

      if (!data) $this.data('bs.alert', (data = new Alert(this)))
      if (typeof option == 'string') data[option].call($this)
  })
}
// EXPOSE ALERT & ALERT PLUGIN
// =================
$.fn.alert             = Plugin
$.fn.alert.Constructor = Alert
```
* 可以通过`$(el).alert()`来使el具有Alert的特性。 `$(el).data(bs.alert)`来获取Alert对应的实例。

#### Alert Conflict处理 ####
```javascript
var old = $.fn.alert
// ALERT NO CONFLICT
// =================
$.fn.alert.noConflict = function () {
  $.fn.alert = old
  return this
}
```
* 可以通过调用`$.fn.alert.noConflict()`来恢复之前的$.fn.alert定义

#### 绑定Alert相关事件 ####
```javascript
// ALERT DATA-API
// ==============
$(document).on('click.bs.alert.data-api', dismiss, Alert.prototype.close)
```
* document上绑定click事件的hook函数: Alert.prototyp.close. 
* click.bs.alert.data-api事件, 作用就是我们可以在取消某些事件处理注册的时候，不会影响其它已经注册的事件处理程序。参考[jquery event namespace][jquery-1]

[alert-js]: https://raw.githubusercontent.com/twbs/bootstrap/v3-dev/js/alert.js
[so-1]: http://stackoverflow.com/questions/13341698/javascript-plus-sign-in-front-of-function-name
[jquery-1]: https://css-tricks.com/namespaced-events-jquery/