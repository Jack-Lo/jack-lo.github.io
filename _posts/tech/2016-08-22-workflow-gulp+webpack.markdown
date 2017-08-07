---
layout: post
title: 强化：构建易用易扩展的工作流
date: 2016-08-22 17:50:00 +0800
categories: posts tech
nav: Tech
---
#### 一、回顾与思考

在上一节的[【进阶：构建具备版本管理能力的项目】]({% post_url tech/2016-08-14-workflow-webpack %})中我们讲解了如何用webpack去搭建一个工作流。
<!--more-->
我们说webpack有很多的loader用来编译打包静态资源，而gulp也有很多的以`gulp-*`格式命名的工作模块用来处理各种资源文件，那webpack和gulp是什么样的联系？有什么样的区别？

这也是很多初学者没有搞明白的，webpack和gulp是不是同类型的工具？

webpack专注于处理各种资源，而gulp专注于任务管理，两者的职能是不同的。打个比方：

gulp好比是大boss，平时要管理包括运营、产品、设计、开发、财务、后勤等等各个部门的工作，但是其实大boss只想知道产品做成什么样、财务剩下多少钱，其他的部门他能管理，但是太琐碎了。现在来了一个叫做webpack的小伙子，告诉大boss说，我能帮你管理运营、设计、开发、后勤这几个部门的工作，你给我个副总当当，然后你就只需要管理产品、财务，还有我这三个对象就好了。两人一拍即合，从此过上了幸福的生活。

没错，我不骗你，就是这么狗血。

虽然单靠webpack也可以搭建一套像模像样的工作流出来，gulp没有webpack一样也活得很好。但是我们拨开表象看本质，gulp的任务管理能力很强，webpack处理资源很方便，为何不结合起来使用呢？

嗯，就这么干！这一节我们就来尝试使用gulp+webpack构建一个又好用又容易扩展功能模块的工作流。我们以gulp为大框架，整合webpack的方式来开展。

#### 二、编译打包

我们先把第一节的[gulpfile.js](https://github.com/jack-Lo/demos/blob/master/gulp_base/gulpfile.js)文件搬出来，老规矩，我们先来实现打包的工作，所以我们先把dev相关的内容剃掉：

```javascript
var gulp = require('gulp')
var sass = require('gulp-sass')
var swig = require('gulp-swig')

gulp.task('sass', function () {
  return gulp.src('src/sass/*.scss')
  .pipe(sass({
    outputStyle: 'compressed'  // 此配置使文件编译并输出压缩过的文件
  }))
  .pipe(gulp.dest('dist/static'))
})

gulp.task('js', function () {
  return gulp.src('src/js/*.js')
  .pipe(gulp.dest('dist/static'))
})

gulp.task('tpl', function () {
  return gulp.src('src/tpl/*.swig')
  .pipe(swig({
    defaults: {
      cache: false  // 此配置强制编译文件不缓存
    }
  }))
  .pipe(gulp.dest('dist'))
})

gulp.task('build', ['sass', 'js', 'tpl'])
```

我们说好相关资源的处理工作是要交给webpack的，所以我们还需要把`sass`、`js`、`tpl`三个gulp任务给去掉，取而代之的是webpack的loader：

```javascript
var gulp = require('gulp')
var webpack = require('webpack')

gulp.task('webpack', function () {
  // do something...
})

gulp.task('build', ['webpack'])
```

嗯，我们已经基本能够预料到接下来的步骤了，填充一下这个webpack任务，而我们在上一节已经有过直接使用webpack的api来工作的尝试，我们直接使用webpack_base里的[webpack.config.js](https://github.com/jack-Lo/demos/blob/master/webpack_base/webpack.config.js)文件：

```javascript
var gulp = require('gulp')
var webpack = require('webpack')
var config = require('./webpack.config.js')

gulp.task('webpack', function () {
  return webpack(config)
})

gulp.task('build', ['webpack'])
```

好像好简单的样子，一切都好顺利肿么办，心情好到飞起，于是迫不及待地在命令行输入：

```
gulp webpack
```

愉快地回车！

好，居然没有报错，完美地兼容了！

回头看一眼根目录！啊。。。。。。说好的dist目录呢？为啥什么都没有生成？！！！

由于webpack处理资源的时候是一系列的异步操作，而gulp并不知道你什么时候处理完了资源，所以对于gulp来说，你webpack的任务从开始到结束我默认当你是同步的，这个任务开始之后，不等你处理完我就已经结束掉了这个工作。所以webpack需要一个操作来告诉gulp，我webpack什么时候处理完了这些资源。

我们先来看最终的代码：

```javascript
var gulp = require('gulp')
var webpack = require('webpack')
var config = require('./webpack.config.js')

gulp.task('webpack', function (cb) {
  webpack(config, function () {
    cb()
  })
})

gulp.task('build', ['webpack'])
```

这里的cb（callback简写）就是一个回调操作，我们通过在webpack完成编译之后的回调里，调用gulp的回调函数，来达到通知gulp任务完成的目的。这里可能一时不好理解，大家花点心思琢磨一下。

之后我们在命令行里执行编译操作，就能看到根目录下生成的dist文件夹了，里面也存放了被打包了的文件。

这里有个问题，我们在命令行里只能看到：

```
[23:35:49] Using gulpfile ~/gulp-webpack_base/gulpfile.js
[23:35:49] Starting 'webpack'...
[23:35:49] Finished 'webpack' after 20 ms
```

然后就没有其他信息了。我们期待什么呢？我们期待能够看到webpack的编译信息，上面我提到过，webpack没有任何的报错信息，事实上就算是真的有错误，也完全不会有任何提示信息出现，在gulp中如果需要输出模块自己的信息，我们需要借助于 **gulp-util** ，这个工具我们在创建ftp任务的时候已经见过面了，具体修改如下：

```javascript
var gulp = require('gulp')
var webpack = require('webpack')
var config = require('./webpack.config.js')
var gutil = require('gulp-util')

gulp.task('webpack', function (cb) {
  webpack(config, function (err, stats) {
    if (err) {
      throw new gutil.PluginError('webpack', err)
    }

    gutil.log('[webpack]', stats.toString({
      colors: true,
      chunks: false
    }))

    cb()
  })
})

gulp.task('build', ['webpack'])
```

这样一来，打包的工作就完成了。

#### 开发环境

不知道大家有没有听过[express](http://www.expressjs.com.cn)？

> Express 是一个基于 Node.js 平台的极简、灵活的 web 应用开发框架，它提供一系列强大的特性，帮助你创建各种 Web 和移动设备应用。  —— express中文网

简单点说，express就是一个基于nodejs的web框架，我们可以用它来很方便地构建一个web项目。而我们这一次使用它的目的是用它来构建一个本地开发服务器。事实上我们上次演示用到的 **webpack-dev-server** 底层就是用express+webpack-dev-middleware实现的，并且我们还要结合webpack的[webpack-dev-middleware](http://webpack.github.io/docs/webpack-dev-middleware.html)和[webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)两个中间件，来了却之前我们没有完成的心愿——热更新。

如果没了解过express，可以在看完本节之后，再去官网补充一些知识点，本节只是用到了一些简单用法。

二话不说，我们先在gulpfile.js中把需要的几个依赖包引入，并且为了方便，我们也直接使用上一节中webpack的开发配置文件[webpack.dev.config.js](https://github.com/jack-Lo/demos/blob/master/webpack_base/webpack.dev.config.js)：

```javascript
...
var webpackHotMiddleware =  require('webpack-hot-middleware')
var webpackDevMiddleware =  require('webpack-dev-middleware')
var devConfig = require('./webpack.dev.config.js')
var express = require('express')
var app = express()
...
```

这里的app便是我们要使用的本地服务器，相对应上一节中的webpackDevServer，然后我们来书写一下server任务：

```javascript
gulp.task('server', function () {
  app.listen(8080, function (err) {
    if (err) {
      console.log(err)
      return
    }
    console.log('listening at http://localhost:8080')
  })
})
```

其实这时候我们已经基本完成了一个服务器搭建，我们在命令行里输入：

```
gulp server
```

然后回车，然后在浏览器里输入`http://localhost:8080`，我们可以看到其实这个服务已经跑起来了，而我们只能在页面上看到`Cannot GET /`，是因为我们还没有定义路由规则，接下来，我们的工作基本就集中在定于路由规则上。

我们先来试试随便指定根路径，返回一个`hello, world!`试试：

```javascript
gulp.task('server', function () {
  app.use('/', function (req, res) {
    res.send('hello, world!')
  })

  app.listen(8080, function (err) {
    if (err) {
      console.log(err)
      return
    }
    console.log('listening at http://localhost:8080')
  })
})
```

现在我们重复上面的操作，命令行输入`gulp server`然后回车一下，刷新浏览器，是不是就看到了`hello, world!`？

聪明的人一下子就想到了express的另外一个用途：为项目写接口，返回一些假数据。这样做当然可以，大家可以自己去亲手尝试一遍。

回到正题，我们希望代码更加清晰，功能更加专一，我们把定义路由的工作，从任务`server`迁移出来，放到另外一个任务`server:init`去做，这样可以使得任务的功能更加单薄，更容易开发调试：

```javascript
gulp.task('server:init', function () {
  app.use('/', function (req, res) {
    res.send('hello, world!')
  })
})

gulp.task('server', ['server:init'], function () {
  app.listen(8080, function (err) {
    if (err) {
      console.log(err)
      return
    }
    console.log('listening at http://localhost:8080')
  })
})
```

好了，我们来插播一段知识点，关于middleware（中间件）：

> 中间件是一种独立的系统软件或服务程序，分布式应用软件借助这种软件在不同的技术之间共享资源。  —— 百度百科

说的有点抽象，简单说来，middleware就是一个提供某种服务的组件，它遵循某种公共的协议或约定，可以整合到各种框架当中。

webpack-dev-middleware和webpack-hot-middleware就是两个中间件，前者提供webpack编译打包的服务，后者提供热更新的服务，两者组合在一起，就是我们之前接触过的webpack-dev-server。

我们单独拿出来，是因为我们需要去订制一套自己的『webpack-dev-server』。上一节我们讲到webpack-dev-server的热更新的时候，遇到一个问题，那就是html模板无法自动刷新的问题，所以我们只能放弃hot-replacement的功能，使用reload的形式，虽然效果也不会差到哪里去，但是既然我们用着webpack，那就没理由舍弃这个特性，我们的目标是：~~装逼装到底~~ 送佛送到西。

我们已经借助于express搭建了一个本地服务器，我们把测试的路由给去掉，接下来我们来试着整合webpack-dev-middleware：

```javascript
gulp.task('server:init', function () {
  var compiler = webpack(devConfig)
  var devMiddleware = webpackDevMiddleware(compiler, {
    stats: {
      colors: true,
      chunks: false
    }
  })

  app.use(devMiddleware)
})
```

是不是跟webDevServer的配置十分相似？

我们命令行里跑一下，刷新浏览器，效果出来啦啦啦啦啦！完美。

我们接着整合webpack-hot-middleware，文档是这么要求的：

1. 增加以下plugin：`new webpack.optimize.OccurenceOrderPlugin()`、`new webpack.HotModuleReplacementPlugin()`、`new webpack.NoErrorsPlugin()`；
2. 每个入口增加`webpack-hot-middleware/client?reload=true`。

操作下来，得到：

```javascript
gulp.task('server:init', function () {
  for (var key in devConfig.entry) {
    var entry = devConfig.entry[key]
    entry.unshift('webpack-hot-middleware/client?reload=true')
  }

  devConfig.plugins.unshift(
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
  )

  var compiler = webpack(devConfig)
  var devMiddleware = webpackDevMiddleware(compiler, {
    hot: true,
    stats: {
      colors: true,
      chunks: false
    }
  })

  var hotMiddleware = webpackHotMiddleware(compiler)

  app.use(devMiddleware)
  app.use(hotMiddleware)
})
```

我们命令行跑一下，然后修改一下index.scss，保存，我们可以在浏览器里直接看到修改了！

但是我们如果修改index.swig，保存后，还是看不到浏览器刷新，这跟我们上次使用webpackDevServer遇到的情况一样。

解决这个问题的方式比较曲折，我们需要解决两个问题：

1. 如何知道用户修改并保存了html；
2. 如何手动通知浏览器刷新页面。

**问题一**  
第一个问题比较好解决，那就是为`html-webpack-plugin`在修改文件并保存之后，注册一个回调，用来告诉我们文件被修改了：

```javascript
compiler.plugin('compilation', function(compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function(data, callback) {
    // 需要在这里通知浏览器刷新页面
    callback()
  })
})
```

详细用法可以[html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin)的参考官方文档，内容太多，这里不详细介绍。

**问题二**  
第二个问题，我们首先需要了解`webpack-hot-middleware/client?reload=true`到底是什么？

为了方便我们就简称它为`client`。

事实上，client就是我们注入到浏览器中的脚本，我们在编辑器里进行的一系列修改，浏览器自动更新，这中间的通讯过程就是由它来完成的，简单来说，我项目的文件修改了，服务器（hotMiddleware）便发送指令给client，client在接收到指令之后，根据指令的内容，相对应地完成工作，如刷新页面，更新资源等等。

我们假设它有一个指令叫做 *reload* ，那我们可以这样操作：

```javascript
compiler.plugin('compilation', function(compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function(data, callback) {
    hotMiddleware.publish({ action: 'reload' })
    callback()
  })
})
```

我们通过使用hotMiddleware来发布（publish）一个action为reload的指令。嗯，这样是可行的，接下来我们需要来实现这个 *reload* 。

因为client本没有这个指令的相关内容，所以我们需要来对它进行扩展，我们在根目录下新建一个 **client.js** 文件，内容如下：

```javascript
var client = require('webpack-hot-middleware/client?reload=true')

client.subscribe(function (obj) {
  if (obj.action === 'reload') {
    window.location.reload()
  }
})
```

我们引入了之前在入口的时候配置的client，然后对它扩展了一个action为reload的类型，并且定义了刷新的脚本。这样我们就完成了对client的功能扩展，以及在修改html的时候，对client发布一个reload的指令这样一个过程。

最后一步，我们把之前我们引入的client（也就是webpack-hot-middleware/client?reload=true）替换成我们自己的client，得到最终的`server:init`：

```javascript
gulp.task('server:init', function () {
  for (var key in devConfig.entry) {
    var entry = devConfig.entry[key]
    entry.unshift('./client.js')
  }

  devConfig.plugins.unshift(
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
  )

  var compiler = webpack(devConfig)
  var devMiddleware = webpackDevMiddleware(compiler, {
    hot: true,
    stats: {
      colors: true,
      chunks: false
    }
  })

  var hotMiddleware = webpackHotMiddleware(compiler)

  compiler.plugin('compilation', function(compilation) {
    compilation.plugin('html-webpack-plugin-after-emit', function(data, callback) {
      hotMiddleware.publish({ action: 'reload' })
      callback()
    })
  })

  app.use(devMiddleware)
  app.use(hotMiddleware)
})
```

ok，我们命令行里输入`gulp server`，然后回车一下！

修改文件，保存，浏览器自动刷新了！css是热更新的，swig文件也可以自动刷新页面了！

到这里为止，我们的开发环境也已经是构建完成了，可以应付开发与打包的工作。

但是由于我的字数还没达到要求不能交卷，所以我需要继续扯下去。

既然我们标题已经说好了是要构建一个 **易用**、 **易扩展** 的工作流，那怎么的也得扩展点东西来看看吧？

嗯，好吧，自己装的逼，怎么的也得自圆其说才行。

#### mock&proxy

我们来谈谈，项目开发中，如何mock数据。

我们现在讨论的是 **基于通过api获取数据的前后端分离模式** 。假设我们当前有以下两种情况：

1. 后端还没写好接口，我们需要自己来生成一些假数据；
2. 后端已经写好接口，我们本地开发调用的时候需要解决跨域问题。

第一个问题很好解决，我们在之前整合express的时候已经稍微提到了一下，我们可以自己写路由来满足调用，我们首先拦截路由`/api/:method`，然后写一个mock-middleware来专门处理它的请求，任务变成了这样（注：这里我们为了专注于mock，把前面middleware的内容给省略掉）：

```javascript
var mockMiddleware = require('./mock-middleware.js')

gulp.task('server:init', function () {
  app.use('/api/:method', mockMiddleware)
})
```

我们来实现这个`mock-middleware`，其实很简单：

```javascript
var map = {
  hello: {
    data: [1, 2, 3],
    msg: null,
    status: 0
  }
}

module.exports = function (req, res, next) {
  var apiKey = req.params.method

  if (apiKey in map) {
    res.json(map[apiKey])
  } else {
    res.status(404).send('api no found!')
  }
}
```

代码也不多，而且都是字面上的意思，咱们简单一点介绍过去：我们首先获取url中的method保存为apiKey，然后我们预先定义好一个map，这个map包含了所有的mock数据，我们定义了一个hello的接口，最后拿apiKey匹配map，如果匹配则返回预设的数据，如果不匹配则返回一个404页面。

第一个问题我们就这么解决了，我们看第二个问题，重点在于：解决跨域问题。

如何解决跨域问题？用服务器代理（proxy）接口。

> 代理（英语：Proxy），也称网络代理，是一种特殊的网络服务，允许一个网络终端（一般为客户端）通过这个服务与另一个网络终端（一般为服务器）进行非直接的连接。  —— 百度百科

百度这解释太晦涩难懂了，还是我来说吧。比如你需要访问服务器A的数据，但是某些原因导致你无法直接访问到或者访问很困难，那么这时候有一台服务器B，你访问B没有障碍，而B访问A也没有障碍，那么我就让B帮我去访问A，我只要访问B，B接收我的这次访问内容，然后去A上面相对应地取数据，取回数据之后返回给我。这个B就是传说中的 ~~黄牛党~~ 代理服务器。

现在我本地页面去访问另外一台服务器上的后端接口，遇到了跨域问题，那么我可以通过服务器代理接口的方式，把接口代理到我本地，我访问本地的接口，就相当于访问了后端服务器的接口，并且没有跨域问题。

这里随便找了一个[proxy-middleware](https://www.npmjs.com/package/proxy-middleware)，这类包相当多，大家可以自行选择，实现如下：

```javascript
var url = require('url')
var proxy = require('proxy-middleware')

app.use(proxy(url.parse('http://tx2.biz.lizhi.fm')))
```

跑起来之后，试着在浏览器访问`localhost:8080/audio/hot?page=1`，可以看到结果：

```
{
  data: {
    content: [
      {
        coverBig: "http://cdn103.img.lizhi.fm/audio_cover/2016/07/29/30278047895714567.jpg",
        coverThumb: "http://cdn103.img.lizhi.fm/audio_cover/2016/07/29/30278047895714567_80x80.jpg",
        createTime: "2016-07-29 11:48:39",
        duration: 49,
        file: "http://cdn5.lizhi.fm/audio/2016/07/29/2548065522170814982_hd.mp3",
        id: 9,
        mediaId: "2548065522170814982",
        name: "#天下骄傲#伏羲",
        status: 0,
        type: "app",
        uid: "2543827817207562796",
        vote: 42559
      },
      ...
    ],
    pageIndex: 1,
    pageSize: 10,
    queryAll: false,
    totalCount: 200,
    totalPage: 20
  },
  msg: null,
  status: 0
}
```

搞定！（这接口别玩得太过啊，万一我项目服务器挂了我怨你们，建议拿百度的练手~）

附上最终`server:init`代码：

```javascript
gulp.task('server:init', function () {
  for (var key in devConfig.entry) {
    var entry = devConfig.entry[key]
    entry.unshift('./client.js')
  }

  devConfig.plugins.unshift(
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
  )

  var compiler = webpack(devConfig)
  var devMiddleware = webpackDevMiddleware(compiler, {
    hot: true,
    stats: {
      colors: true,
      chunks: false
    }
  })

  var hotMiddleware = webpackHotMiddleware(compiler)

  compiler.plugin('compilation', function(compilation) {
    compilation.plugin('html-webpack-plugin-after-emit', function(data, callback) {
      hotMiddleware.publish({ action: 'reload' })
      callback()
    })
  })

  app.use('/api/:method', mockMiddleware)
  app.use(proxy(url.parse('http://tx2.biz.lizhi.fm')))

  app.use(devMiddleware)
  app.use(hotMiddleware)
})
```

好，到这里为止，我们已经完成了大部分的使用场景，基本上是啥需求都能满足了，最后再加上个我们在介绍gulp的时候就已经讲到过的ftp任务，那就算功德圆满了。

#### 总结

抛开我们中间的一些扩展的知识点不讲，我们只看大框架gulp+webpack，这样一个组合是不是具有很多的优点？我们可以看到，单单对于gulp项目来说，项目对资源的处理能力提升了，而对于webpack的项目来说，项目的功能更加齐备了而且扩展也相当方便。

事到如今，你还会觉得gulp和webpack是一回事吗？你还会感到困惑吗？

会的话我也没法怎么着了，你自己看着办吧。

最后的最后，再给大家布置一些任务，由于篇幅关系，我们很多细节其实没有做好：

1. 随着功能越来越多，根目录下的配置文件也越来越多，像gulpfile.js、webpack.config.js、webpack.dev.config.js、server.js等等，有些散乱，而且这些都是项目无关的文件，属于工具，我们可以建个 **build** 文件夹来收纳一下，这里相对应的就有很多路径需要修改，这操劳的事就大家自觉去做了；
2. 除了第一节的gulp讲解之外，我为了省事都没有提醒大家把诸如`gulp build`、`gulp server`等这些命令封装在package.json中作为预设脚本，一定程度上影响了雅观，事实上我自己是做了的，也希望大家要自觉去做；
3. 每次运行编译打包（build）相关命令之前，都要加上`rimraf dist`，清除过期的内容；
4. 有一些类型的资源我没有讲到，比如image、font，甚至是react，等等，其实这是留给大家自己去尝试的，大家要能举一反三，何况这又是很简单的事。

那这一系列就到此完结了，希望大家看完最终都能有所收获。

本次演示项目的git地址：[gulp-webpack_base](https://github.com/jack-Lo/demos/tree/master/gulp-webpack_base)

[【上一篇：进阶：构建具备版本管理能力的项目】]({% post_url tech/2016-08-14-workflow-webpack %})

（文章有任何谬误之处，欢迎留言指出）