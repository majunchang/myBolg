---
title: javascript的发布-订阅者模式
date: 2018-03-05 21:56:04
tags: javascript
categories: javascript
---

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
</html>
<script>
  // 代码的封装
  var event = {
    list: [],
    listen: function(key, fn) {
      if (!this.list[key]) {
        this.list[key] = []
      }
      this.list[key].push(fn)
    },
    trigger: function() {
      var key = [].shift.call(arguments)
      var fns = this.list[key]
      if (!fns || fns.length === 0) {
        return false
      }
      for (var i = 0, fn; fn = fns[i++];) {
        fn.apply(this, arguments)
      }
    },
    remove: function(key, fn) {
      var fns = this.list[key]
      if (!fns) {
        return false
      }
      if (!fn) {
        fns.length = 0
      } else {
        for (var i = fns.length - 1; i >= 0; i--) {
          var _fn = fns[i]
          if (_fn === fn) {
            fns.splice(i, 1)
          }
        }
      }
    },
  }
  //  定义一个inieEvent的函数 使得所有的对象 都有发布功能
  var initEvent = function(obj) {
    for (var i in event) {
      obj[i] = event[i]
    }
  }

  //  进行测试
  var shopObj = {}
  initEvent(shopObj)

  //  小红订阅以下消息
  shopObj.listen('red', fn1 = function(color, size) {
    console.log('小红你要得颜色是' + color)
    console.log('小红你要得尺码是' + size)
  })

  // 小花订阅以下消息
  shopObj.listen('flower',fn2 =  function(color, size) {
    console.log('小花你要得颜色是' + color)
    console.log('小花你要得尺码是' + size)
  })


  shopObj.remove('red',fn1);
  shopObj.trigger('red', 'red', 40)
  shopObj.trigger('flower', 'black', 43)


</script>
```
