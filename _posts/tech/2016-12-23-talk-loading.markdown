---
layout: post
title: 写一个网页进度loading
date: 2016-12-23 19:51 +0800
categories: posts tech
nav: Tech
---
loading随处可见，比如一个app经常会有下拉刷新，上拉加载的功能，在刷新和加载的过程中为了让用户感知到 **load** 的过程，我们会使用一些过渡动画来表达。最常见的比如“转圈圈”，“省略号”等等。  
<!--more-->
网页loading有很多用处，比如页面的加载进度，数据的加载过程等等，数据的加载loading很好做，只需要在加载数据之前（before ajax）显示loading效果，在数据返回之后（ajax completed）结束loading效果，就可以了。  

但是页面的加载进度，需要一点技巧。  

页面加载进度一直以来都是一个常见而又晦涩的需求，常见是因为它在某些“重”网页（特别是网页游戏）的应用特别重要；晦涩是因为web的特性，各种零散资源决定它很难是“真实”的进度，只能是一种“假”的进度，至少在逻辑代码加载完成之前，我们都不能统计到进度，而逻辑代码自身的进度也无法统计。另外，我们不可能监控到所有资源的加载情况。  

所以页面的加载进度都是“假”的，它存在的目的是为了提高用户体验，使用户不至于在打开页面之后长时间面对一片空白，导致用户流失。  

既然是“假”的，我们就要做到“仿真”才有用。仿真是有意义的，事实上用户并不在乎某一刻你是不是真的加载到了百分之几，他只关心你还要load多久。所以接下来我们就来实现一个页面加载进度loading。  

首先准备一段loading的html：  

```html
<!DOCTYPE html>
<html>
<head>
    <title>写一个网页进度loading</title>
</head>
<body>
  <div class="loading" id="loading">
    <div class="progress" id="progress">0%</div>
  </div>
</body>
</html>
```

来点样式装扮一下：  

```css
.loading {
  display: table;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #fff;
  z-index: 5;
}

.loading .progress {
  display: table-cell;
  vertical-align: middle;
  text-align: center;
}
```

我们先假设这个loading只需要在页面加载完成之后隐藏，中间不需要显示进度。那么很简单，我们第一时间想到的就是window.onload：  

（以下内容为了方便演示，默认使用jQuery，语法有es6的箭头函数）  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')

window.onload = () => {
  $loading.hide()
}
```

ok，这样基本的loading流程就有了，我们增加一个进度的效果，每隔100ms就自增1，一直到100%为止，而另一方面window loaded的时候，我们把loading给隐藏。  

我们来补充一下进度：  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0  // 初始化进度

var timer = window.setInterval(() => {  // 设置定时器
  if (prg >= 100) {  // 到达终点，关闭定时器
    window.clearInterval(timer)
    prg = 100
  } else {  // 未到终点，进度自增
    prg++
  }
  
  $progress.html(prg + '%')
  console.log(prg)
}, 100)

window.onload = () => {
  $loading.hide()
}
```


效果不错，但是有个问题，万一window loaded太慢了，导致进度显示load到100%了，loading还没有隐藏，那就打脸了。所以，我们需要让loading在window loaded的时候才到达终点，在这之前，loading可以保持一个等待的状态，比如在80%的时候，先停一停，然后在loaded的时候快速将进度推至100%。这个做法是目前绝大部份进度条的做法。  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0

var timer = window.setInterval(() => {
  if (prg >= 80) {  // 到达第一阶段80%，关闭定时器，保持等待
    window.clearInterval(timer)
    prg = 100
  } else {
    prg++
  }
  
  $progress.html(prg + '%')
  console.log(prg)
}, 100)

window.onload = () => {
  window.clearInterval(timer)
  window.setInterval(() => {
    if (prg >= 100) {  // 到达终点，关闭定时器
      window.clearInterval(timer)
      prg = 100
      $loading.hide()
    } else {
      prg++
    }
    
    $progress.html(prg + '%')
    console.log(prg)
  }, 10)  // 时间间隔缩短
}
```

ok，这差不多就是我们想要的功能了，我们来提炼一下代码，把重复的代码给封装一下：  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0

var timer = 0

progress(80, 100)

window.onload = () => {
  progress(100, 10, () => {
    $loading.hide()
  })
}

function progress (dist, delay, callback) {
  window.clearInterval(timer)
  timer = window.setInterval(() => {
    if (prg >= dist) {
      window.clearInterval(timer)
      prg = dist
      callback && callback()
    } else {
      prg++
    }
    
    $progress.html(prg + '%')
    console.log(prg)
  }, delay)
}
```

我们得到了一个progress函数，这个函数就是我们主要的功能模块，通过传入一个目标值、一个时间间隔，就可以模拟进度的演化过程。  

目前来看，这个进度还是有些问题的： 

1. 进度太平均，相同的时间间隔，相同的增量，不符合网络环境的特点；  
2. window.onload太快，我们还来不及看清100%，loading就已经不见了；  
3. 每次第一阶段都是在80%就暂停了，露馅儿了；  

第一个点，我们要让时间间隔随机，增量也随机；第二个点很简单，我们延迟一下就好了；第三点也需要我们随机产生一个初始值。  

增量随机很好办，如何让时间间隔随机？setInterval是无法动态设置delay的，那么我们就要把它改造一下，使用setTimeout来实现。（setInterval跟setTimeout的用法和区别就不细说了吧？）  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0

var timer = 0

progress([80, 90], [1, 3], 100)  // 使用数组来表示随机数的区间

window.onload = () => {
  progress(100, [1, 5], 10, () => {
    window.setTimeout(() => {  // 延迟了一秒再隐藏loading
      $loading.hide()
    }, 1000)
  })
}

function progress (dist, speed, delay, callback) {
  var _dist = random(dist)
  var _delay = random(delay)
  var _speed = random(speed)
  window.clearTimeout(timer)
  timer = window.setTimeout(() => {
    if (prg + _speed >= _dist) {
      window.clearTimeout(timer)
      prg = _dist
      callback && callback()
    } else {
      prg += _speed
      progress (_dist, speed, delay, callback)
    }
    
    $progress.html(parseInt(prg) + '%')  // 留意，由于已经不是自增1，所以这里要取整
    console.log(prg)
  }, _delay)
}

function random (n) {
  if (typeof n === 'object') {
    var times = n[1] - n[0]
    var offset = n[0]
    return Math.random() * times + offset
  } else {
    return n
  }
}
```

至此，我们差不多完成了需求。  

but，还有一个比较隐蔽的问题，我们现在使用window.onload，发现从进入页面，到window.onload这中间相隔时间十分短，我们基本是感受不到第一阶段进度（80%）的，这是没有问题的——我们在意的是，如果页面的加载资源数量很多，体积很大的时候，从进入页面，到window.onload就不是这么快速了，这中间可能会很漫长（5～20秒不等），但事实上，我们只需要为 **首屏资源** 的加载争取时间就可以了，不需要等待所有资源就绪，而且更快地呈现页面也是提高用户体验的关键。  

我们应该考虑页面loading停留过久的情况，我们需要为loading设置一个超时时间，超过这个时间，假设window.onload还没有完成，我们也要把进度推到100%，把loading结束掉。  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0

var timer = 0

progress([80, 90], [1, 3], 100)  // 使用数组来表示随机数的区间

window.onload = () => {
  progress(100, [1, 5], 10, () => {
    window.setTimeout(() => {  // 延迟了一秒再隐藏loading
      $loading.hide()
    }, 1000)
  })
}

window.setTimeout(() => {  // 设置5秒的超时时间
  progress(100, [1, 5], 10, () => {
    window.setTimeout(() => {  // 延迟了一秒再隐藏loading
      $loading.hide()
    }, 1000)
  })
}, 5000)

function progress (dist, speed, delay, callback) {
  var _dist = random(dist)
  var _delay = random(delay)
  var _speed = random(speed)
  window.clearTimeout(timer)
  timer = window.setTimeout(() => {
    if (prg + _speed >= _dist) {
      window.clearTimeout(timer)
      prg = _dist
      callback && callback()
    } else {
      prg += _speed
      progress (_dist, speed, delay, callback)
    }
    
    $progress.html(parseInt(prg) + '%')  // 留意，由于已经不是自增1，所以这里要取整
    console.log(prg)
  }, _delay)
}

function random (n) {
  if (typeof n === 'object') {
    var times = n[1] - n[0]
    var offset = n[0]
    return Math.random() * times + offset
  } else {
    return n
  }
}
```

我们直接设置了一个定时器，5s的时间来作为超时时间。这样做是可以的。  

but，还是有问题，这个定时器是在js加载完毕之后才开始生效的，也就是说，我们忽略了js加载完毕之前的时间，这误差可大可小，我们设置的5s，实际用户可能等待了8s，这是有问题的。我们做用户体验，需要从实际情况去考虑，所以这个开始时间还需要再提前一些，我们在head里来记录这个开始时间，然后在js当中去做对比，如果时间差大于超时时间，那我们就可以直接执行最后的完成步骤，如果小于超时时间，则等待 **剩余的时间** 过后，再完成进度。  

先在head里埋点，记录用户进入页面的时间`loadingStartTime`：  

```html
<!DOCTYPE html>
<html>
<head>
  <title>写一个网页进度loading</title>
  <script>
    window.loadingStartTime = new Date()
  </script>
  <script src="index.js"></script>
</head>
<body>
  <div class="loading" id="loading">
    <div class="progress" id="progress">0%</div>
  </div>
</body>
</html>
```

然后，我们对比 **当前的时间** ，看是否超时：（为了方便复用代码，我把完成的部分封装成函数complete）  

```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0

var timer = 0
var now = new Date()  // 记录当前时间
var timeout = 5000  // 超时时间

progress([80, 90], [1, 3], 100)

window.onload = () => {
  complete()
}

if (now - loadingStartTime > timeout) {  // 超时
  complete()
} else {
  window.setTimeout(() => {  // 未超时，则等待剩余时间
    complete()
  }, timeout - (now - loadingStartTime))
}

function complete () {  // 封装完成进度功能
  progress(100, [1, 5], 10, () => {
    window.setTimeout(() => {
      $loading.hide()
    }, 1000)
  })
}

function progress (dist, speed, delay, callback) {
  var _dist = random(dist)
  var _delay = random(delay)
  var _speed = random(speed)
  window.clearTimeout(timer)
  timer = window.setTimeout(() => {
    if (prg + _speed >= _dist) {
      window.clearTimeout(timer)
      prg = _dist
      callback && callback()
    } else {
      prg += _speed
      progress (_dist, speed, delay, callback)
    }
    
    $progress.html(parseInt(prg) + '%')
    console.log(prg)
  }, _delay)
}

function random (n) {
  if (typeof n === 'object') {
    var times = n[1] - n[0]
    var offset = n[0]
    return Math.random() * times + offset
  } else {
    return n
  }
}
```

至此，我们算是完整地实现了这一功能。  

然而，事情还没有结束，少年你太天真。  

如果目的是为了写一个纯粹障眼法的伪loading，那跟其他loading的实现就没什么区别了，我们做事讲究脚踏实地，能实现的实现，不能实现的，为了团队和谐，我们不得已坑蒙拐骗。那么我们还能更贴近实际情况一点吗？其实是可以的。  

我们来分析一个场景，假设我们想让我们的loading更加真实一些，那么我们可以选择性地对页面上几个比较大的资源的加载进行跟踪，然后拆分整个进度条，比如我们页面有三张大图a、b、c，那么我们将进度条拆成五段，每加载完一张图我们就推进一个进度：  

随机初始化[10, 20] ->  
图a推进20%的进度 ->  
图b推进25%的进度 ->  
图c推进30%的进度 ->  
完成100%  

这三张图要占`20% + 25% + 30% = 75%`的进度。  

问题是，如果图片加载完成是按照顺序来的，那我们可以很简单地：10（假设初始进度是10%） -> 30 -> 55 -> 85 -> 100，但事实是，图片不会按照顺序来，谁早到谁晚到是说不准的，所以我们需要更合理的方式去管理这些进度增量，使它们不会互相覆盖。

1. 我们需要一个能够替我们累计增量的变量`next`；  
2. 由于我们的`progress`都是传目的进度的，我们需要另外一个函数`add`，来传增量进度。  


```javascript
var $loading = $('#loading')
var $progress = $('#progress')
var prg = 0

var timer = 0
var now = new Date()
var timeout = 5000

var next = prg

add([30, 50], [1, 3], 100)  // 第一阶段

window.setTimeout(() => {  // 模拟图a加载完
  add(20, [1, 3], 200)
}, 1000)

window.setTimeout(() => {  // 模拟图c加载完
  add(30, [1, 3], 200)
}, 2000)

window.setTimeout(() => {  // 模拟图b加载完
  add(25, [1, 3], 200)
}, 2500)

window.onload = () => {
  complete()
}

if (now - loadingStartTime > timeout) {
  complete()
} else {
  window.setTimeout(() => {
    complete()
  }, timeout - (now - loadingStartTime))
}

function complete () {
  add(100, [1, 5], 10, () => {
    window.setTimeout(() => {
      $loading.hide()
    }, 1000)
  })
}

function add (dist, speed, delay, callback) {
  var _dist = random(dist)
  if (next + _dist > 100) {  // 对超出部分裁剪对齐
    next = 100
  } else {
    next += _dist
  }

  progress(next, speed, delay, callback)
}

function progress (dist, speed, delay, callback) {
  var _delay = random(delay)
  var _speed = random(speed)
  window.clearTimeout(timer)
  timer = window.setTimeout(() => {
    if (prg + _speed >= dist) {
      window.clearTimeout(timer)
      prg = dist
      callback && callback()
    } else {
      prg += _speed
      progress (dist, speed, delay, callback)
    }
    
    $progress.html(parseInt(prg) + '%')
    console.log(prg)
  }, _delay)
}

function random (n) {
  if (typeof n === 'object') {
    var times = n[1] - n[0]
    var offset = n[0]
    return Math.random() * times + offset
  } else {
    return n
  }
}
```


我们这里为了方便，用setTimeout来模拟图片的加载，真实应用应该是使用`image.onload`。  

以上，就是我们一步步实现一个进度loading的过程了，演示代码可以戳我的codePen [写一个网页进度loading](http://codepen.io/Jack-Lo/pen/woZyRB)。  

看似很简单的一个功能，其实仔细推敲，还是有很多细节要考虑的。  

到这里，其实真的已经完成了，代码有点多有点乱是不是？你可以整理一下，封装成为插件的。  

然而，好吧，其实我已经把这个进度封装成插件了。。。  

没错，其实我就是来帮自己打广告的。。。  

好吧，github仓库在此 [ez-progress](https://github.com/jack-Lo/ez-progress)。  

**ez-progress** 是一个web（伪）进度插件，使用 **ez-progress** 实现这个功能非常简单：  

```javascript
var Progress = require('ez-progress')
var prg = new Progress()

var $loading = $('#loading')
var $progress = $('#progress')

prg.on('progress', function (res) {
  var progress = parseInt(res.progress)  // 注意进度取整，不然有可能会出现小数
  $progress.html(progress + '%')
})

prg.go([60, 70], function (res) {
  prg.complete(null, [0, 5], [0, 50])  // 飞一般地冲向终点
}, [0, 3], [0, 200])

window.onload = function () {
  prg.complete(null, [0, 5], [0, 50])  // 飞一般地冲向终点
}
```


木油错，94这么简单！  

这可能是我目前写过最短的博文了，由此看出以前是有多么的啰嗦，哈哈哈哈！