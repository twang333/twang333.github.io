---
layout: post
title:  "Bootstrap alert.js分析"
date:   2017-02-07 19:00:05 +0800
categories: bootstrap
---
alert.js的代码组织如下:

```javascript
+function ($) {
  // ALERT CLASS DEFINITION
  // ======================
  var dismiss = '[data-dismiss="alert"]'
  var Alert   = function (el) {
    $(el).on('click', dismiss, this.close)
  }
  Alert.VERSION = '3.3.7'
  Alert.TRANSITION_DURATION = 150
  Alert.prototype.close = function (e) {
  }

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

  var old = $.fn.alert

  // EXPOSE ALERT & ALERT PLUGIN
  // =================
  $.fn.alert             = Plugin
  $.fn.alert.Constructor = Alert

  // ALERT NO CONFLICT
  // =================
  $.fn.alert.noConflict = function () {
    $.fn.alert = old
    return this
  }

  // ALERT DATA-API
  // ==============
  $(document).on('click.bs.alert.data-api', dismiss, Alert.prototype.close)
}(jQuery);
```

这份代码包括下面5个部分:
1. 定义Alert
2. 定义Alert Plugin
3. 暴露Alert和Alert Plugin
4. Alert NO CONFLICT处理
5. 绑定Alert相关事件
查看[bootstrap-3.3.7 alert.js源码][alert-js]

[alert-js]: https://raw.githubusercontent.com/twbs/bootstrap/v3-dev/js/alert.js
