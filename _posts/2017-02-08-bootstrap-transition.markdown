---
layout: post
title:  "Bootstrap transition.js分析"
date:   2017-02-08 19:00:05 +0800
categories: bootstrap
---

查看[bootstrap-3.3.7 transition.js源码][alert-js]

transition.js的代码组织如下:

#### 自调用函数 ####
```javascript
+function ($) {
}(jQuery);
```
* 通过一个自调用函数，创建一个函数域空间，在该私有的域空间内定义Alert
* 传入jQuery参数，是为了得到一个函数局部的$符，来标记jQuery。因为全局的$符标记的不一定是jQuery, 有可能其他的js库也定义了$符。
* function前面的+号，是为了让js解析器将+号后面的部分识别为expression, 这样才可以进行函数调用。可以用其他符号，比如 - , ! 等等. 参考[自调用函数定义][so-1]

[alert-js]: https://raw.githubusercontent.com/twbs/bootstrap/v3-dev/js/alert.js
