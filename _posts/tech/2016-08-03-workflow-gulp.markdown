---
layout: post
title: 入门：十分钟自动化构建
date: 2016-08-03 00:30:00 +0800
categories: posts tech
nav: Tech
---
你是否还在手动压缩图片、js、css？  
是否还在手动编译sass、less、coffee？  
是否还在手动合并精灵图？  
是否还在  
……  
<!--more-->

> 简单点，干活的方式简单点。

在前端技术飞速发展的今天，如果你还在日复一日地重复着那无休止的体力劳动，那你要反思了，这年头不懂捣鼓个工作流出来都不好意思说自己学前端的。

毕竟javascript是最好的编程语言嘛。:)

玩笑开到这里，到底什么是工作流？

> 工作流（Workflow），指“业务过程的部分或整体在计算机应用环境下的自动化”。  ——*百度百科*

如题，此次的分享主题"自动化构建"，目的便是介绍如何根据需要搭建一套工作流，由浅入深。

#### 一、前提

往下看之前，请自备以下知识：
- nodejs的一些基础用法，比如：node index.js；
- npm的基本用法init、(un)install、run等；
- 了解[gulp](http://www.gulpjs.com.cn/)、[webpack](http://webpack.github.io/)（本章不会用到）的用途

以上知识点请务必掌握，不然没法接着往下讲，切勿好高骛远；   
如果你准备好了，那我们就开始了！

我们先来设定一个简单的需求：
- 一个本地开发环境，具备监控文件变化并实时更新的功能；
- 修改代码，保存之后浏览器自动刷新
- 实时编译各种预编译格式文件
- 压缩合并静态资源，打包输出
- 部署上传

一言不合就上图：
![自动化构建（入门）.png](http://upload-images.jianshu.io/upload_images/2636400-4dac1fe21724c316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好，有了这个规划之后，我们开始着手来实现它。

#### 二、源码

首先新建一个项目文件夹project，接着我们打开命令行工具，切换到这个目录下，开始初始化这个项目：

```
cd project
npm init
```

按照提示完成初始化，打开项目我们会得到一个package.json文件像这样：

```javascript
// package.json
{
  "name": "project",
  "version": "1.0.0",
  "description": "a test project",
  "main": "index.js",
  "scripts": {
    "test": "node ./build/test.js"
  },
  "author": "jack lo",
  "license": "ISC"
}
```

紧接着我们先来创造一份源码，结构如下：

```
- project
  |- src  // 源文件夹
  | |- tpl
  | | `- index.html
  | |- css
  | | `- index.css
  | |- js
  | | `- index.js
  |- dist  // 打包文件夹
  `- package.json
```

#### 三、编译打包

一切材料就绪，我们开始安装工具包，这一次我们主要使用gulp来构建，不需要用到webpack，同时，为了实现浏览器自动刷新的功能，我们还需要用到[browser-sync](http://www.browsersync.cn/)。

```
npm install gulp browser-sync --save-dev
```

这需要花点时间。

\---------- 虚度光阴中 ---------

事实上，这里有个小坑需要我们提前准备一下，因为我们接下来会用到gulp的cli，所以这里我们需要全局安装gulp，没错，我就是设套让你装两遍！

这是个好习惯，请务必以后也坚持这么做。把开发时候用到的所有npm包都记录在package.json文件中，可以方便日后自己以及他人的使用。

```
npm install gulp -g
```

mac用户可能还要sudo一下：

```
sudo npm install gulp -g
```

回车后按提示输入密码，继续回车，就可以正常安装了。

\---------- 再一次虚度光阴中 ---------

ok，安装完后，我们可以开始做点有意义的事了。

gulp的用法其实相当简单，api也就那么几个。简单来说，就是写好一个配置文件gulpfile.js，然后在命令行里执行它。

首先在根目录下新建一个gulpfile.js，然后随便建一个叫做test的任务：

```javascript
// gulpfile.js
var gulp = require('gulp')

// 创建一个名为test的任务，任务内容只是简单地在控制台输出一段文本
gulp.task('test', function () {
  return console.log('this is a test')
})
```

ok，保存。然后我们回到命令行，输入以下命令然后回车

```
gulp test
```

gulp后面跟的是你所要执行的任务名，于是我们得到了这样一个结果

```
[22:56:35] Using gulpfile ~/project/gulpfile.js
[22:56:35] Starting 'test'...
this is a test
[22:56:35] Finished 'test' after 149 μs
```

成功输出文本！恭喜，你已经掌握了gulp一半以上的用法。

现在我们尝试着来操作文件，我们将src/html里面的html文件给拷贝到dist文件夹（如果不存在则新建）里面去，怎么做？

```javascript
// 创建一个copy的任务
gulp.task('copy', function () {
  return gulp.src('src/tpl/*.html')
  .pipe(gulp.dest('dist'))
})
```

这里的pipe其实是一种管道，你可以一直pipe连着pipe下去，也就是这个工作流可以一直这样一层层执行下去，随便你定义多少个处理任务都行，这就是gulp的特点，简单明了的工作流程。很多人不明白gulp到底是做什么的，其实到这里，就可以有个大概的认识了：

> gulp是以定义并执行一个个任务的形式来工作的**流程管理工具**，它的作用在于提供一套简单易用的工作方式。

在gulp以前，处理一份sass文件，你可能需要先执行一次编译的任务，编译成css之后，再执行一遍压缩css的任务，压缩完之后，再手动拷贝到打包文件夹里。原本需要分三步来操作的一件事情，在gulp里面就是一个任务的sei而已：

```javascript
// 创建一个css的处理任务
gulp.task('sass', function () {
  return gulp.src('src/sass/*.scss')
  .pipe(sass())
  .pipe(minifycss())
  .pipe(gulp.dest('dist/static'))
})
```

甚至我们可以同时执行多个任务，我可以定义好coffee、sass、image、html的四个任务，再把他们合并到一个任务当中去，一次性执行完：

```javascript
// 创建一个build的处理任务
gulp.task('build', ['coffee', 'sass', 'image', 'html'])
```

我只需要：

```
gulp build
```

回车，这酸爽。

好了，介绍了这么多内容，最后我们还是回到最开始那份需求的实现上来。

我们来建几个任务，分别处理js、css、html文件，把js和css文件放到dist/static目录下，把html文件放到dist下：

```javascript
gulp.task('css', function () {
  return gulp.src('src/css/*.css')
  .pipe(gulp.dest('dist/static'))
})

gulp.task('js', function () {
  return gulp.src('src/js/*.js')
  .pipe(gulp.dest('dist/static'))
})

gulp.task('html', function () {
  return gulp.src('src/tpl/*.html')
  .pipe(gulp.dest('dist'))
})

gulp.task('build', ['css', 'js', 'html'])
```

命令行gulp build一下，回车！再看看dist文件夹，done！

咋一看，好像没什么问题，但好像又有哪里不太对劲。不对啊，除了复制到dist文件夹，好像没啥功能啊，说好的压缩合并呢？说好的处理预编译呢？嗯，有了这个框架，这些功能我们想加多少加多少。

这样简单的项目，我们没法玩出花样，我们来点预编译语言，css用sass代替，html用swig代替：

```
- project
  |- src  // 源文件夹
  | |- tpl
  | | `- index.swig
  | |- sass
  | | `- index.scss
  | |- js
  | | `- index.js
  |- dist  // 打包文件夹
  `- package.json
```

编译sass需要安装gulp-sass模块，编译swig需要gulp-swig。注意到了吗？基本上gulp的模块都以`gulp-*`的形式出现，所以如果以后你使用gulp的时候想用什么模块，可以试试在npm搜`gulp-模块名`。

```
npm install gulp-sass gulp-swig --save-dev
```

\---------- 虚度光阴中 ---------

安装完后，我们再来修改一下配置文件gulpfile.js

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

再接着gulp build一下，编译结束，可以看看dist下是不是有index.html，dist/static下是不是有编译完成且压缩过的index.css，没有你找我！这里就不分别展示源文件和打包后文件的内容了，因为并不重要，想看演示项目内容的朋友，可以在文章结尾处找到本次演示项目的git仓库链接。各种预编语言和前端模板大家可以根据自己的喜好选择，笔者只是选择自己熟悉的几种来做演示。

到这里我们基本上已经完成打包的工作了，我们来试着搭建开发环境。

#### 四、搭建开发环境

前文我们提到一个工具browser-sync，还记得吗？现在用得上了！我们先创建一个开发任务，像gulp build一样简单的任务，我们的目标是：~~没有蛀牙~~ 一句话搞定。

```javascript
gulp.task('dev', ['js:dev', 'sass:dev', 'tpl:dev'], function () {
  // do something here...
})
```

看着好像稍有不同。这一次我们创建一个叫做dev的任务，这个任务先执行`js:dev`、`sass:dev`和`tpl:dev`三个任务，然后再执行回调里的内容，我们的本地服务器就是要在回调里去定义并且启动。

在这之前，我们先来了解一下browser-sync：

> Browsersync能让浏览器实时、快速响应您的文件更改（html、js、css、sass、less等）并自动刷新页面。  ——Browsersync中文网

事实上Browsersync可以理解为一个本地服务器，类似于Apache，不同的是，Browsersync只提供很简单的http功能，它的主要功能，是通过搭建一个本地服务器，并且监听文件的更改，自动刷新浏览器，实时地呈现最新内容。

browser-sync的用法：

```javascript
var browserSync = require('browser-sync').create()

browserSync.init({
  server: {
    baseDir: "./"  // 设置服务器的根目录
  }
})
```

其实也很简单，没有太多内容，现在我们要将它整合进我们的gulp任务里，捣鼓几下，我们得到：

```javascript
gulp.task('dev', ['js:dev', 'sass:dev', 'tpl:dev'], function () {
  browserSync.init({
    server: {
      baseDir: "./dist"  // 设置服务器的根目录为dist目录
    },
    notify: false  // 开启静默模式
  })

  // 我们使用gulp的文件监听功能，来实时编译修改过后的文件
  gulp.watch('src/js/*.js', ['js:dev'])
  gulp.watch('src/sass/*.scss', ['sass:dev'])
  gulp.watch('src/tpl/*.swig', ['tpl:dev'])
})
```

这里的watch行为可以简单理解为：我（gulp）就这么盯着你（src/js/*.js文件），只要你被改动了，我就马上执行`js:dev`任务来处理你，产生最新的文件。

那么，我们现在还需要补充一下`js:dev`、`sass:dev`和`tpl:dev`这三个任务，与原来的`js`、`sass`和`tpl`三个任务大同小异，这里笼统地过一遍就好，我们直接看代码：

```javascript
var browserSync = require('browser-sync').create()
var reload = browserSync.reload

gulp.task('sass:dev', function () {
  return gulp.src('src/sass/*.scss')
  .pipe(sass())
  .pipe(gulp.dest('dist/static'))
  .pipe(reload({stream: true}))
})

gulp.task('js:dev', function () {
  return gulp.src('src/js/*.js')
  .pipe(gulp.dest('dist/static'))
  .pipe(reload({stream: true}))
})

gulp.task('tpl:dev', function () {
  return gulp.src('src/tpl/*.swig')
  .pipe(swig({
    defaults: {
      cache: false  // 此配置强制编译文件不缓存
    }
  }))
  .pipe(gulp.dest('dist'))
  .pipe(reload({stream: true}))
})
```

这里的`reload`方法，我们不需要过多了解，只要知道，通过执行它就可以刷新浏览器。这里的`stream`用法可以查阅官方文档，这里不重要所以不细讲。

> 值得留意的是：这里的`sass:dev`任务，我们并没有像`sass`任务一样配置编译模式为compressed，也就是不使用压缩功能，为什么呢？事实上，我们在开发的时候，并不需要压缩静态资源文件，可以说我们不在意它的体积是大一点还是小一点，我们在意的是样式是否写得符合期望，我们在乎的是功能是否实现，所以不需要启用压缩或者其他的什么优化功能，这样可以减轻编译的负担，加快编译速度。如果你对js或者图片也使用了压缩功能，建议在开发模式下去掉，只在打包模式下使用。

最终我们整理得到一份gulpfile.js文件：

```javascript
var gulp = require('gulp')
var sass = require('gulp-sass')
var swig = require('gulp-swig')
var browserSync = require('browser-sync').create()
var reload = browserSync.reload

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

gulp.task('sass:dev', function () {
  return gulp.src('src/sass/*.scss')
  .pipe(sass())
  .pipe(gulp.dest('dist/static'))
  .pipe(reload({stream: true}))
})

gulp.task('js:dev', function () {
  return gulp.src('src/js/*.js')
  .pipe(gulp.dest('dist/static'))
  .pipe(reload({stream: true}))
})

gulp.task('tpl:dev', function () {
  return gulp.src('src/tpl/*.swig')
  .pipe(swig({
    defaults: {
      cache: false  // 此配置强制编译文件不缓存
    }
  }))
  .pipe(gulp.dest('dist'))
  .pipe(reload({stream: true}))
})

gulp.task('dev', ['js:dev', 'sass:dev', 'tpl:dev'], function () {
  browserSync.init({
    server: {
      baseDir: "./dist"
    },
    notify: false
  })
  gulp.watch('src/js/*.js', ['js:dev'])
  gulp.watch('src/sass/*.scss', ['sass:dev'])
  gulp.watch('src/tpl/*.swig', ['tpl:dev'])
})

gulp.task('build', ['sass', 'js', 'tpl'])

```

嗯，完美，到这一步，我们这个自动化构建已经基本完成了，而且还算是完整。现在，我们开发的时候，就执行gulp dev，打包的时候就执行gulp build，是不是很方便？

> 注意：gulp dev任务启动以后是一直保持工作状态的，也就是它不像gulp build一样一次性执行完，它是keep alive的，所以我们如果要停止这个任务，需要手动按ctrl+c组合键，结束这个任务。

#### 五、补充

然而事情还没完，我们的目标是：~~装逼~~ 尽善尽美！

有三个地方其实我们还没做到位：
- 对命令进行包装，尽量简洁。gulp dev和gulp build其实已经很简洁了，但事实上这只是因为这个项目很简单，用到的命令很少，在开发复杂的项目时，我们通常要输入复杂的一长串的命令行。gulp dev其实也是gulp --gulpfile gulpfile.js dev的缺省写法而已，具体情况可以去gulp官网了解。所以我们需要有更简洁的方式去执行这些预先准备好的脚本，就好像windows系统下的快捷方式；
- 目前的情况是，我们每次执行，都会将文件拷贝到dist目录下，但是却没有删除的工作，也就是说文件数量只增不减，这样多次下来，随着文件的增删改动，必然会遗留很多没有用的文件。我们需要每次启动编译或者打包之前，先把整个dist文件夹删除，然后再重新生成dist；
- 上传的功能还没做呢？！

二话不说就开工！

第一点很好解决，我们可以把脚本作为一项配置存放在package.json文件中：

```javascript
{
  "name": "project",
  "version": "1.0.0",
  "description": "a test project",
  "main": "index.js",
  "scripts": {
    "dev": "gulp dev",  // 开发脚本
    "build": "gulp build",  // 打包脚本
    "test": "node ./build/test.js"
  },
  "author": "jack lo",
  "license": "ISC",
  "devDependencies": {
    "browser-sync": "^2.13.0",
    "gulp": "^3.9.1",
    "gulp-sass": "^2.3.2",
    "gulp-swig": "^0.8.0"
  }
}
```

注意到了吗？*scripts*项就是用来预定义脚本的地方，我们可以很方便地把脚本按照上面的形式封装好，然后执行的方式就变成了：

```
npm run dev  // 执行开发
npm run build  // 执行打包
```

搞定！

我们接着看第二点，删除dist文件夹，多简单的事啊！鼠标右键，删除，搞定！

……

哪有这么low的事，我们的目标是：~~懒癌晚期~~ 能不自己做的事情，绝不自己动手。

有一个叫做rimraf的包，可以帮我们做这事，我们需要用到它的cli，所以跟gulp一样，我们全局安装它：

```
npm install rimraf -g
```

安装完后，我们再重新修改一下package.json文件中的*scripts*内容：

```javascript
{
  "name": "project",
  "version": "1.0.0",
  "description": "a test project",
  "main": "index.js",
  "scripts": {
    "dev": "gulp dev",  // 开发脚本
    "build": "rimraf dist && gulp build",  // 打包脚本
    "test": "node ./build/test.js"
  },
  "author": "jack lo",
  "license": "ISC",
  "devDependencies": {
    "browser-sync": "^2.13.0",
    "gulp": "^3.9.1",
    "gulp-sass": "^2.3.2",
    "gulp-swig": "^0.8.0"
  }
}
```

ok，现在试试执行`npm run build`，dist文件夹是不是先被删除，然后再重新生成了？

完美。

第三点放到最后才补充，主要是考虑到它并不是必要的，因为有些项目并不需要ftp上传，一般是提交svn，然后再由后端或者运维去部署，笔者是需要将静态资源上传到cdn服务器进行加速的，所以需要这样一个任务，在此我们简单介绍一下。

举一反三一下，我们再创建一个upload任务：

```javascript
var ftp = require('gulp-ftp')
var gutil = require('gulp-util')

gulp.task('upload', function () {
  return gulp.src('dist/**')
  .pipe(ftp({
    host: '8.8.8.8',  // 远程主机ip
    port: 22,  // 端口
    user: 'username',  // 帐号
    pass: 'password',  // 密码
    remotePath: '/project'  // 上传路径，不存在则新建
  }))
  .pipe(gutil.noop())
})
```

自行安装一下gulp-ftp和gulp-util两个包，然后在package.json文件中的*scripts*补充一个脚本npm run upload来执行gulp upload。

笔者通常都是打包之后顺便上传，命令行直接输入npm run build && npm run upload，回车，然后就可以愉快地去跟旁边的妹纸聊天了。

#### 六、总结

到这里，我们已经完整搭完了这一套简易自动化工具，好像讲了很多东西，其实总结起来内容非常少：我们只不过分别用三个小任务（sass、js、tpl），组成了build和dev这两个大任务，仅此而已。熟练的情况下操作起来，整个过程也不过十分钟！

ps:网络太差怪我咯？呵呵。

由于时间和篇幅关系，我们只简单处理了css、js和html，事实上，你还可以在这个基础上继续完善下去，js可以由coffeejs编译得到，而且还可以继续压缩，甚至可以把全部js文件合并成一个！html也一样可以继续压缩。而且，你完全可以自己创建一个任务去处理其他诸如图片、字体等等。

好啦，简易版就讲到这里了。

后话：gulp与webpack都是目前比较流行的编译打包工具，那么它们之间有什么异同点？怎么去进行选择？我们下一节将介绍如何去用webpack搭建一套开发工具，从中我们可以感受这两者的差别，敬请留意。

本次演示项目的git地址：[gulp_base](https://github.com/jack-Lo/demos/tree/master/gulp_base)

[【下一篇：进阶：构建具备版本管理能力的项目】]({% post_url tech/2016-08-14-workflow-webpack %})

（文章有任何谬误之处，欢迎留言指出）