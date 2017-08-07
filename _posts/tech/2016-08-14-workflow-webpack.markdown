---
layout: post
title: 进阶：构建具备版本管理能力的项目
date: 2016-08-14 13:09:00 +0800
categories: posts tech
nav: Tech
---
[webpack](http://webpack.github.io/)是时下十分流行的编译和打包工具，它提供一种可扩展的loader的方式，简单的配置，便可以编译打包各类型的文件，包括js、css、image、font、html，以及各种预编译语言都不在话下。
<!--more-->
#### 一、回顾与思考

在上一节的[【入门：十分钟自动化构建】]({% post_url tech/2016-08-03-workflow-gulp %})中我们讲解了如何用gulp去搭建一个工作流。我们认识到gulp是一个流程管理工具，以单个任务为基础单元，组合成为一套完整的工作流，而且gulp还有很多的以`gulp-*`格式命名的工作模块，用来处理各种资源文件，如果没有看过上一节内容的同学，建议先回去看过，再往下阅读，因为本节内容是跟上一节的知识点紧密联系的。

这一节我们会讲解**如何构建具备版本管理能力的项目**，什么是**版本管理能力**？不是什么svn或者branch，我们看这样一个场景来帮助理解：

> 你用上次搭建的工作流开发了一个网站，然后上线。  
第二天打开发现有bug，心想尼玛赶紧趁着老板没发现修复一下。  
改完代码，打包，发布，一气呵成，完美。  
然而十分钟以后老板让你去一趟办公室，打开页面跟你说有个bug。  
心里一抽，一看！我勒个去，这坑爹的缓存啊。。。

怎么去解决这个缓存？，或者说，怎么保证我上线一个新版本，可以完完全全地替代旧版本？这就是版本管理。

这问题有很多解决方法，包括手动打个戳啊什么的，像`src="a.jpg?201608062315"`，这确实可以解决，但是如果一次更新的东西很多，你压根改不过来。

gulp能帮我们做这事吗？可以，麻烦，有兴趣的同学可以自行搜索资料。有没有简单点的套路？有的，webpack天然支持这一功能。接下来我们就介绍如何用webpack来搭建这么一套工作流。

#### 二、webpack

webpack的用法，我们简单介绍一下。跟gulp一样，webpack也是写好配置文件才能开始工作。

全局安装webpack
```
npm install webpack -g
```

记得养成好习惯，也本地安装一下哦

```
npm install webpack --save-dev
```

顺带我们把接下来要用到的几个loader一起安装了：
```
npm install style-loader css-loader sass-loader swig-loader --save-dev
```

这里的`*-loader`作用跟`gulp-*`差不多，就是一些编译用的模块。

紧接着我们在根目录下，新建一个webpack.config.js配置文件，我们直接来看代码：
```javascript
var path = require('path')

module.exports = {
  entry: {
    Index: ['./src/js/index.js']
  },
  output: {
    path: path.resolve(__dirname, './dist/static'),
    publicPath: 'static/',
    filename: '[name].js'
  },
  resolve: {
    extensions: ['', '.js', '.scss', '.swig']
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: 'style!css'
      },
      {
        test: /\.scss$/,
        loader: 'style!css!sass'
      },
      {
        test: /\.swig$/,
        loader: 'swig'
      }
    ]
  }
}
```
这里大致分为四部分的内容：  
**entry**  
入口文件，也就是一切工作的起点，你可以将整个web应用都最终打包成一个js文件，那你只需要定义一个入口，而如果你希望对多个页面独立开来，你需要定义多个入口，最终在不同的页面引用不同的js。一个entry对应生成一个bundle。

**output**  
定义打包输出的配置：
- path是打包文件的存放路径，按上面的配置，意味着我们待会打包的文件是要放在`dist/static`下的；
- publicPath是定义文件被打包后的url前缀的，效果是`<script src="index.js"></script>` => `<script src="static/index.js"></script>`；
- filename顾名思义，就是定义打包后的名字，你可以按照上面的方式定义，最终文件名与原来的保持一致，也可以在这个基础上自己再拼接一些名词，比如`[name].min.js`，最终打包出来文件名就是`index.min.js`。

**resolve**  
这个配置可有可无，它是定义一些常用的文件拓展名，被定义了的文件格式，在引用的时候可以不加扩展名，比如我配置了`.js`的拓展名之后，在开发中我可以直接`require('./common')`，而不需要`require('./common.js')`。

**module**  
最重要的一个部分，我们在这里定义针对各种类型文件的loader（加载器/编译器），可以看到我们分别定义了css、scss、swig三种文件对应的loader，多个loader之间用`!`隔开，'-loader'可以省略不写。注意，编译的优先次序是从右到左的，比如scss文件是先被sass-loader处理，然后再被css-loader处理，最后再被style-loader处理。

除了这几点，webpack还有个比较重要的配置项`plugins`，这个我们暂时不介绍，后面会在具体的应用场景里引入。除此之外，webpack的其他配置项大家可以去官网了解。

#### 三、源码

话不多说，我们还是取原来的gulp_base项目源码来做介绍。
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
webpack的模块化使我们可以很方便地使用commonjs的规范来组织代码。有关commonjs的内容，大家可以自行查阅相关文章，这里不作深入展开；或者关注我之后的文章，我将会针对模块化做一些讲解。不过这都是后话了。

在这里我们只需要知道，我们通过将页面划分成很多的小模块，每个模块都是单独的js文件，我们可以通过require的方式，去引入一个模块，进而组织成一个大的web应用。而webpack更甚，在webpack里，"一切资源皆模块"，不管是图片还是css，都可以在js文件中直接require，当然前提是你已经在webpack.config.js的**module**项里配置了这类文件的相关loader。

#### 四、编译打包

ok，准备就绪，我们来试着编译一下
```
webpack
```
理论上，我们如果把js文件放在一个叫做js的文件夹里，那几乎百分百确定这货就是.js结尾的，扩展名显得有点多余。当然如果你硬是要在js文件夹里放个.jpg的文件我也拿你没办法是吧。

咱们看看dist目录里面是不是有了个static文件夹？文件夹里是不是有个文件叫做index.js？是不是就成功了？没成功你找我。

好，我们试着验证另外一个配置，`output.filename`，我们改一下：
```javascript
module.exports = {
  output: {
    path: path.resolve(__dirname, './dist/static'),
    publicPath: 'static/',
    filename: '[name].[chunkhash].js'
  },
  ...
}
```
命令行里webpack一下：
```
Hash: 408544aa45e4f298cb49
Version: webpack 1.13.1
Time: 49ms
                        Asset     Size  Chunks             Chunk Names
Index.e167a63b2bcf28077701.js  1.53 kB       0  [emitted]  Index
   [0] multi Index 28 bytes {0} [built]
   [1] ./src/js/index.js 21 bytes {0} [built]
```
打开dist/static目录一看，生成了一个`Index.e167a63b2bcf28077701.js`文件，`chunkhash`是由webpack对index.js这个文件进行 **md5** 编码之后得到的一个戳，这个戳是唯一的，拼接在文件名上，可以表示这个文件的唯一性，只要index.js里面的内容被修改一丁点，这个生成的戳就会不一样。

这不正是我们开头想要的版本管理吗？太美好了，没想到这么容易就解决了这个问题。

有个问题，如果我每修改一点，这个文件名就不一样了，那我不就要在html中每个引用了这个js的地方再修改一下文件名？这根本就没有减负嘛！

怎么去解决这个问题？webpack当然提供了解决方案，如果我们用webpack帮我们打包html，并且自动地去注入这些资源，那问题就解决了。

一般来说webpack就只是处理入口文件，也就是从js去入手，这样的话怎么都轮不到html，那怎么样才能让webpack去处理一下html？答案就是我们之前提过的，**plugins**！

我们来修改一下配置：
```javascript
var path = require('path')
var HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: {
    Index: ['./src/js/index.js']
  },
  output: {
    path: path.resolve(__dirname, './dist/static'),
    publicPath: 'static/',
    filename: '[name].[chunkhash].js'
  },
  resolve: {
    extensions: ['', '.js', '.scss', '.swig']
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: 'style!css'
      },
      {
        test: /\.scss$/,
        loader: 'style!css!sass'
      },
      {
        test: /\.swig$/,
        loader: 'swig'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      chunks: ['Index'],
      filename: '../index.html',  // 留意这里，这里的路径是相对来path配置的
      template: './src/tpl/index',
      inject: true
    })
  ]
}
```

请留意两点修改的地方，一个是`var HtmlWebpackPlugin = require('html-webpack-plugin')`，一个是`plugins`的配置内容。

webpack允许以插件的方式去扩展功能。[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)是webpack提供的一个简化html处理的插件，其实本意就是为了解决这个引用bundle的问题，顺便提供一些像压缩啊注入、变量啊什么的功能。具体其他功能大家可以去官网看文档。

使用前记得先install啊！

我们再次编译一下，然后查看一下dist文件夹，是不是多了个index.html？而且index.js也被编译后自动注入了页面  
编译前：
```html
// index.swig
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>index</title>
  </head>
  <body>
    hello, world!
  </body>
</html>
```

编译后：
```html
// index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>index</title>
  </head>
  <body>
    hello, world!
  <script type="text/javascript" src="static/Index.e167a63b2bcf28077701.js"></script></body>
</html>
```
done！

看着这html，有没有感觉少了点什么？嗯，少了css，我们来看看css怎么注入。

其实css更简单，我们已经说过了：
> 在webpack里，一切资源皆模块。

我们只需要在index.js中直接require就可以了：
```javascript
// index.js
require('../sass/index')
console.log('index')
```

编译一下，打开index.html：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>index</title>
  </head>
  <body>
    hello, world!
  <script type="text/javascript" src="static/index.c04c40a250e85ea100ab.js"></script></body>
</html>
```

没看到有引入css啊？你个骗纸！

别着急，试着双击index.html文件，在浏览器打开试试？是不是有样式了！神奇，样式是哪里来的？原来css都被打包进js里面了！

虽然说目的是达到了，但是总觉得有点不好，我们还是习惯外联css啦，能不能实现？可以！甩你一个plugin！[extract-text-webpack-plugin](https://www.npmjs.com/package/extract-text-webpack-plugin)：
```javascript
var ExtractTextPlugin = require('extract-text-webpack-plugin')

module.exports = {
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', ['css'])
      },
      {
        test: /\.scss$/,
        loader: ExtractTextPlugin.extract('style', ['css', 'sass'])
      },
      {
        test: /\.swig$/,
        loader: 'swig'
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin('[name].[chunkhash].css'),
    ...
  ],
  ...
}
```
看着就能明白吧，这里不再细讲，反正也就是这么用而已。

编译一遍，看看index.html：
```html
// index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>index</title>
  <link href="static/index.d4085060ebf342f7a5a1.css" rel="stylesheet"></head>
  <body>
    hello, world!
  <script type="text/javascript" src="static/index.d4085060ebf342f7a5a1.js"></script></body>
</html>
```
完美！

到这里，我们的打包工作算是介绍完了，下面我们试着搭建开发环境。

#### 五、开发环境

我们在使用gulp的时候，搭建开发环境是借助[browser-sync](http://www.browsersync.cn/)，而对于webpack来说，我们有更好的选择：[webpack-dev-server](http://webpack.github.io/docs/webpack-dev-server.html)！

webpack-dev-server是webpack官网提供的一款本地开发服务器，可以为我们提供一个适用于在webpack下进行开发的环境。

webpack-dev-server的用法其实很简单，基于webpack的配置，稍稍补充一些内容就可以了。webpack-dev-server有两种使用方式，cli和api，我们分别做介绍。

在开始之前，我们需要针对开发环境单独创建一份配置文件，有印象的同学可以记得，我们上一节讲过，打包与开发的配置文件有一个不同点，打包是以产出并且优化为目的的，而开发则更侧重效率，所以我们需要把一些压缩和多余的优化手段给去掉，减轻开发过程中的编译负担。

我们新建一份配置文件webpack.dev.config.js：
```javascript
// webpack.dev.config.js
var path = require('path')
var HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: {
    index: ['./src/js/index.js']
  },
  output: {
    path: path.resolve(__dirname, './dist/static')
    filename: '[name].js'
  },
  resolve: {
    extensions: ['', '.js', '.scss', '.swig']
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: 'style!css'
      },
      {
        test: /\.scss$/,
        loader: 'style!css!sass'
      },
      {
        test: /\.swig$/,
        loader: 'swig'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      chunks: ['index'],
      filename: '../index.html',
      template: './src/tpl/index',
      inject: true
    })
  ]
}
```

我们把`extract-text-webpack-plugin`去掉，再把为了避免缓存而在文件名加戳的方式也去掉，这样就差不多了。

值得注意的是，我们把path和publicPath两项都去掉了，这里是为了配合下面的cli方案使用，我们接下去看就会明白。

**cli**  
cli的方式要求我们全局安装webpack-dev-server：
```
npm install webpack-dev-server -g
```

然后我们直接在命令行里输入：
```
webpack-dev-server --config ./webpack.dev.config.js --display-error-details --devtool eval --content-base ./ --inline --progress --colors --host 0.0.0.0
```

天了噜，这也太长了。这长长的一串命令，都是针对webpack-dev-server初始化的配置，当然，webpack-dev-server的配置也是可以在webpack.dev.config.js里定义的，只是我们这里主要展示cli的用法，所以就直接这样输入了，具体各个参数是什么功能，我们可以不必全部了解，因为这里并没有特别关键的配置项，大部分都是对终端里输出形式的一些设置，我们挑重点的来说：

1. **content-base** 设置开发服务器的根目录；
2. **host** 设置为0.0.0.0使我们在局域网中可以通过ip的方式访问；
3. **hot** 配置热更新。

热更新的作用是使得我们在修改并保存代码之后，在不刷新浏览器的情况下，自动更新浏览器对应部分的代码！注意，此时页面是不会刷新的，但变化立刻就可以反映出来！这一项我们并没有设置，为什么？因为这里有一点小问题，热更新的功能目前并不能对.html（或者静态模板）这样的文件起作用，也就是说，如果我们修改的是html，它将会不起任何作用，除非我们手动刷新，才能看到效果。这就有点捡芝麻丢西瓜了，所以我们选择了放弃它。

当然，如果是像react这样的主要由js来构建的项目，那我们可以毫无顾忌地使用热更新。

我们在配置文件中去掉了publicPath的设置，其实你可以试着设置一下，你会发现publicPath是会影响到全部的资源包括index.html，使得我们所有的文件都被放在服务器的`static`目录下，此时只能通过`/static`来访问到index.html，所以我们选择去掉这两项设置，讲文件统一编译打包到根目录下。

其他配置项有兴趣的同学可以到[官网的文档](http://webpack.github.io/docs/webpack-dev-server.html#webpack-dev-server-cli)里了解。

**api**
cli的方式虽然很方便很简单，但是如果要使用更多的定制化，api会更灵活一些。

> 注意，这一部分内容十分重要，因为我们之后乃至下一节的文章里，都是基于api来配置工作流的，请务必掌握这一节的内容，如果你还有再深入研究下去的想法的话，不要满足于cli的便捷方式而对这一部分内容粗略带过。

使用api的方式，意味着我们需要自己来手动定义一个开发服务器，并且补充相关配置项，为了使配置与实现逻辑分开，我们还是使用原来那份配置文件webpack.dev.config.js，但是新建一份js文件server.js来编写这个服务器逻辑。
```javascript
var WebpackDevServer = require('webpack-dev-server')
var webpack = require('webpack')
var config = require('./webpack.dev.config.js')
var path = require('path')

var compiler = webpack(config)

var server = new WebpackDevServer(compiler, {
  stats: {
    colors: true,
    chunks: false
  }
})

server.listen(8080, 'localhost', function() {})
```
ok，我们先来试着跑一下：
```
node ./server.js
```
回车，成功啦！而且运行效果貌似跟cli的一样。

这里的内容实在没什么可讲的，我们首先使用webpack传入配置内容config，得到一个编译的实例，之后我们再创建一个webpackDevServer的实例（本地服务器），来跑这份编译实例，服务器监听8080端口，所以我们在浏览器中输入[http://localhost:8080](http://localhost:8080)就可以访问了，搞定！

到这里就ok了吗？其实还没有，我们试着修改index.js，然后保存，看看浏览器。

浏览器没有反应，也就是说，这里还没有实现自动刷新浏览器的功能，我们配置少了一些东西。

好，那我们接着来。

[webpackDevServer](http://webpack.github.io/docs/webpack-dev-server.html#hot-module-replacement-with-node-js-api)的官方文档里其实已经说得很清楚了：
> Similar to the inline mode the user must make changes to the webpack configuration.
Three changes are needed:
- add an entry point to the webpack configuration: `webpack/hot/dev-server`.
- add the `new webpack.HotModuleReplacementPlugin()` to the webpack configuration.
- add `hot: true` to the webpack-dev-server configuration to enable HMR on the server.

这里就算看不懂意思也能猜得到吧~

- 每个入口都添加`webpack/hot/dev-server`
- plugins里增加`new webpack.HotModuleReplacementPlugin()`
- 配置 `hot: true`

二话不说就开搞，我们按照指示修改得到最终的server.js长这样：
```javascript
var WebpackDevServer = require('webpack-dev-server')
var webpack = require('webpack')
var config = require('./webpack.dev.config.js')
var path = require('path')

for (var key in config.entry) {
  var entry = config.entry[key]
  entry.unshift('webpack-dev-server/client?http://localhost:8080', 'webpack/hot/dev-server')
}

config.plugins.push(new webpack.HotModuleReplacementPlugin())

var compiler = webpack(config)

var server = new WebpackDevServer(compiler, {
  hot: true,
  stats: {
    colors: true,
    chunks: false
  }
})

server.listen(8080, 'localhost', function() {})
```
都是挺简单的东西没啥看不懂的，咱就不细讲了，直接跑起来！

修改一下index.js，保存！呀，效果出来了，挺好；  
修改一下index.scss，呀，效果出来了，倍儿棒；  
修改一下index.swig，保存！呀，你咋就没反应了啊。。。

还记得我们用cli来搭建的时候遇到的类似情况吗？WebpackDevServer的**Hot Module Replacement**是不支持html模板的，所以这里也一样，怎么解决？

还是像原来，我们把`hot: true`的配置项去掉就好了，ok，运行一遍，修改，保存，done！

#### 六、总结

gulpfile.js文件我们一看就明白，逻辑清晰层次分明，webpack的配置文件则需要细细咀嚼。

相比gulp，webpack构建工作流是不是会复杂些？当然，我们只是对比来说，事实上如果只是从操作来看，也并没有多复杂。我描述一个感受，看大家是不是有共鸣：  
gulp是一套流程管理工具，专门管理一个个的任务，同时结合一些编译模块来处理各种资源文件；  
webpack则是一个大而全的编译器，主要用于各种资源文件的编译和打包工作，由此为中心衍生出了一系列周边工具，比如开发工具，插件等等。

事实上，webpackDevServer编译的文件是存放在内存中的，操作起来速度非常的快，整个的项目编译完成几乎就是那么一个刷新的瞬间，而gulp则还是硬盘上的操作，速度要慢几个量级，我们没有感受到明显的差异，那是因为项目还不够大。

有没有留意到，我们这一节并没有像上一节那样介绍到ftp工具？之所以没有作介绍，是因为webpack并没有提供这样的插件或者模块去完成这样的工作，事实上也不应该有，因为webpack本身是一个资源处理器，与项目的上传部署没有太多关系。我们需要借助其他的npm包，用一种毫无联系的方式去组合这两个功能，我们可能需要编写一个ftp.js，不知大家是否还记得我们上一节当中的`npm run build && npm run upload`命令？差不多就是这样。这个ftp.js就跟webpack的关系相当微弱了，不像我们写gulp那样，扩展一个功能的时候，只需要写多一个task。

这里概念非常抽象，我也不能完完整整地传达，不知道大家理解多少。老方法，我还是举个栗子来说明：
webpack就好比一套很好用的螺丝刀，基本所有的机器的维修工作都能应付。但是如果需要完整一点的工程，我们可能还需要大锤啊电钻啊什么乱七八糟的。但是东西一过来可能没法整理到一起，只能拿个塑料袋粗略地装起来；  
gulp好比一个工具箱，里面也有几件比较简单的螺丝刀，如果有其他的工具，来了就往里面放，妥妥的。

嗯，又是一个不怎么贴切的栗子，凑合着用吧憋唧唧歪歪的。

我们先理解到这里，进一步的对比，我们会在教程最后的章节里呈现一次比较细致的说明。

后话：我们可以把webpack这把好用的螺丝刀放gulp这个工具箱里吗？答案当然是yes！帅（美）的人已经动手了，丑的还在卖萌。下一节[【强化：构建易用易扩展的工作流】](http://www.jianshu.com/p/435ca031a5b7)我们将为大家讲解如何使用gulp结合webpack，各取所长地搭建一套简单易用、高可扩展性的工作流。


本次演示项目的git地址：[webpack_base](https://github.com/jack-Lo/demos/tree/master/webpack_base)

[【上一篇：入门：十分钟自动化构建】]({% post_url tech/2016-08-03-workflow-gulp %})   
[【下一篇：强化：构建易用易扩展的工作流】]({% post_url tech/2016-08-22-workflow-gulp+webpack %})

（文章有任何谬误之处，欢迎留言指出）