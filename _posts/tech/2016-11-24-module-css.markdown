---
layout: post
title: 从css谈模块化
date: 2016-11-24 00:05:00 +0800
categories: posts tech
nav: Tech
---
模块化是现今我们随处都可以听到的一个名词，什么是模块化？为什么我们需要模块化？这是本系列文章我们要弄明白的一个问题。我们也借这部分内容，顺带回顾一下前端的发展历程。  
<!--more-->
说实话，模块化这个主题有点大，我一时也不知道从哪里讲起比较合适，通常来说，前端的工作内容主要涉及三个方面：html、css、js（javascript），其他的像as（actionscript，flash的脚本语言）、jsp、smarty等等模版类的语法标记我们在此就先略去了，因为不是特别重要。那我们所说的模块化也可以分别当成这三条线去看，如html的模块化、css的模块化，以及js的模块化，这三者我们称为（web）前端模块化，搞清楚这几者的关系，我们接下来要了解的事情已经很清晰了。  

我们单独来讲，我想先从css，因为这是我认为最容易入手且非常鲜明的一块内容。  

### 背景

起初的.css是长什么样的？

```css
/* index.css */

body {
  margin: 0;
  padding: 0;
  font-size: 18px;
}

.box {
  background: #333;
  color: #fff;
}

.box .list {
  margin-left: 10px;
}

.box .list .item {
  border-bottom: 1px solid #ccc;
}

.box .list .item:last-child {
  border-bottom: 0;
}

.box .list .item a {
  text-decoration: none;
  color: #fff;
}

.box .list .item span {
  color: red;
}

.box .list .item a ... {
  ...
}
```


是不是很熟悉？一个简单的手打的列表模块。  

这里存在的问题是：

* 按照这个顺序写下去，选择器会越写越长，造成书写累赘；
* 越来越长的选择器容易使我们混淆dom的空间顺序，想象如果有好几个平级的选择器（如.box .list .item a与.box .list .item span），我们可能一时看不出这两者的关系，是父子还是兄弟元素？
* 维护困难，假设我们需要重构这个box，在.box和.list之间加入一层.wrap，在.item与a和span之间加入一层.block，那简直就是个灾难，我们要谨慎地找到确切的位置，然后再找到所有匹配的、长长的选择器，在合适的位置全部做修改；
* 我们很难从做到复用，假设我们在另外一个页面也需要这个box，那我们就需要把所有跟box相关的部分复制粘贴一份，而当这个box需要修改的时候，我们可能要重新找出所有用到这个box的地方，然后又是复制粘贴一份——当然，有人说这个问题是可以通过一定的方法解决的，我们后面再来讨论这个问题；

……  

（欢迎补充css的开发痛点）   

事实上我们手打css遇到的问题可以大致归纳为以下几点：

* 选择器繁琐冗长；
* 命名冲突；
* 层级结构不清晰；
* 代码难以复用；


问题很多，怎么去解决？由于css的发展很缓慢，在工具上也一直没有进展，导致这些问题，比如上述的第一点，几乎无解。所以问题往往要依赖“规范”来解决。我们先来看代码复用。  

### 复用

要实现代码复用很简单，我们只需要提供一个公共css库，来存放我们的公共样式以及公共模块即可：

```css
/* common.css */

body {
  background: #fff;
  color: #333;
  font-size: 16px;
}

.box ... {
  background: #333;
  color: #fff;
  ...
}

.another-box ... {
  ...
}

```

然后我们在其他的css文件中引用这个common.css，这样就实现了代码的复用，只要是想全局共享的样式和模块，只要在这里添加进去就可以了。  

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
  <link rel="stylesheet" type="text/css" href="./style/common.css">
  <link rel="stylesheet" type="text/css" href="./style/index.css">
</head>
<body>
  <div class="box">
    ...
  </div>
  <div class="another-box">
    ...
  </div>
</body>
</html>
```


很完美，   

——至少目前为止是这样没错。  

这里我们来探究几种情况：

1. 假设我们这个项目非常大，大概有20个页面这么多，那么我们每做一个页面就会往common里面补充3～4个公共样式/模块，那么在这个项目开发完成以后，common的体积可能要比其他css的体积都大；
2. 假设有几个这样的页面，他们本身内容非常少，比如404页面，可能只需要用到少量的公共样式，但是由于考虑到维护问题，我们还是要引入common（单独写样式会使得该页面在common更新的时候无法同步得到更新），这就使得一个页面变得很“重”；
3. 由于common越写越大，它所占用的命名就越多，那么我们在引入common的时候，即使我们页面还什么都没有，但已经默认被占用了很多的命名，使得我们在某个页面的可用命名变少，而且是越来越少；
4. 我们在common中书写公共模块，在具体页面的私有css里书写私有模块，假设现在我们需要全局添加一个公共模块.nice-box，我们发现，这个模块名已经在index.css中被占用了，于是我们试着把名字改成.handsome-box，却又发现这个名字在about.css中被占用了，哦买噶的！

……  

瞬间整个人都不好了，内心充满了绝望，心想还是转行吧，垃圾语言毁我一生。  

我们分析一下，上面这些情况其实重点只有两个：一、冗余；二、污染。冗余是难免的，为了维护牺牲一部分灵活性也是可以接受的，不过我们需要用一些方式来减少这样的冗余，避免让它成为负担；而污染，却是亟待解决的问题，这颗定时炸弹随着项目的增大最终会变成一场无法挽回的灾难！这时候就体现出了命名规范的重要性了。  

这就要求我们用一套合理的规范来约束和组织我们的代码。

### 规范

> 编程规范使得我们的项目在一定程度上是可维护的。比如针对类名污染制定了命名规范，针对选择器指定了书写选择器所要遵循的规范，等等。这些规范都在一定程度上约束了css的书写，使得项目不至于混乱。网易的[NEC](http://nec.netease.com)是其中一种比较完整的解决方案。有兴趣的童鞋可以搜索了解，笔者本身受nec影响较大，所以以下内容有一定程度的雷同。  

现在我们来试着制定一套css编程规范，来解决以上提到的问题。  

我们规定页面由且只由几种基本结构体构成：框架、模块，以及元件。其他零散的元素，除了是作为模块的辅助类，否则不能独立于这三者存在。  

#### 框架
框架是指构成页面的基础结构，它是一个页面的筋骨。我们假设有个页面index.html，它的整体最外围表现为一个class为.g-index的div，然后它由页头（.g-hd）、主体（.g-bd）、页脚（.g-ft）三个部分组成：

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
</head>
<body>
  <div class="g-index">
    <div class="g-hd"></div>
    <div class="g-bd"></div>
    <div class="g-ft"></div>
  </div>
</body>
</html>
```

这样我们就大概能描绘出一个页面的基本轮廓了。紧接着我们来给它补充一些模块。

#### 模块
模块是页面上数量最多，同时也是最重要的部分，它是代码复用的主体部分，是一个个按照功能划分的区域，如导航栏、轮播图、登录窗口、信息列表等等，模块之间相互独立，分布在页面上，嵌在框架的各个位置上，组成一个丰富多彩的页面。  

还是以index.html为例，我们假设页头有个导航栏模块（.m-nav），主体有个新闻列表模块（.m-news），页脚有个版权声明模块（.m-copy_right）：

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
</head>
<body>
  <div class="g-index">
    <div class="g-hd">
      <div class="m-nav">
        nav
      </div>
    </div>
    <div class="g-bd">
      <div class="m-news">
        news
      </div>
    </div>
    <div class="g-ft">
      <div class="m-copy_right">
        copy_right
      </div>
    </div>
  </div>
</body>
</html>
```

#### 元件
元件是独立的、可重复使用的，并且在某些情况下可以作为模块的组成部分的一种细颗粒。比如一个按钮，一个logo等等。某种意义上说，它其实可以等同于模块，因为它们两者的区别只是规模不同而已。模块更强调一个功能完整的整体，而元件则更强调独立性。

我们假设这个页面还需要在页头放个logo（.u-logo），在导航栏中放置一个登录按钮（.u-login_btn）：

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
</head>
<body>
  <div class="g-index">
    <div class="g-hd">
      <img class="u-logo" alt="logo">
      <div class="m-nav">
        nav
        <a href="/logoin" class="u-login_btn">登录</a>
      </div>
    </div>
    <div class="g-bd">
      <div class="m-news">
        news
      </div>
    </div>
    <div class="g-ft">
      <div class="m-copy_right">
        copy_right
      </div>
    </div>
  </div>
</body>
</html>
```

三种基本结构体介绍完，我们回来总结一下。你一定发现了一个现象，在搭建框架的时候，我给框架元素命名用`g-`开头，给模块命名使用`m-`，给元件命名使用`u-`。这是命名规范的一部分，我们使用这三个前缀给相应结构体命名，就是为了更好地标志一个结构体，更好地展示它的功用，这也是我们常说的 **语义化** ，同时也能实现隔离作用，起到类似命名空间的效果。  

1. 框架的命名以`g-`开头，一般与页面同名，比如index.html，那框架就是最外层就是.g-index，about.html就是.g-about，以此类推，其他常用的内部结构有.g-hd（header）、.g-bd（body）、.g-ft（footer）、.g-sd（side）、.g-mn（main）等等；
2. 模块命名以`m-`开头，一般以相对应的用途来命名，比如导航栏`m-nav`、新闻`m-news`、版权`m-copy_right`等等，一般来说模块名是唯一的，而且模块本身应该是可移植、可复用的；
3. 元件命名以`u-`开头，一般以自身含义来命名，比如`u-logo`表示一个logo，`u-btn`表示一个按钮。

那么除却框架、模块、元件的相关命名内容之外，命名规范还有以下几点内容：

1. 命名尽量以缩写的方式，言简意赅地表达，比如用`bd`表达`body`，用`nav`表达`navigator`等，使用长长的单词显得多余又臃肿；
2. 前缀与名称之间用`-`连接，而名称之间的若干单词以`_`连接，组合单词除外，如`side-menu`；
3. `z-`开头表示状态，如`z-active`、`z-succ`、`z-disabled`等等；
4. 可以根据需要定制其他开头，但是请尽量将分类控制在少数，因为太多的分类反而造成困惑和不必要的分类开销，其实gmuz就已经可以满足日常开发了。

### 重构common

有了命名规范，我们可以对common进行一次改写：

```css
/* common.css */

body {
  background: #fff;
  color: #333;
  font-size: 16px;
}

.m-nav { ... }

.m-news { ... }

.m-copy_right { ... }

```

ok，现在我们一定程度上缓解了“污染”的问题，至少按照命名规范，我们的common构成由原来笼统的一类，变成了现在gmuz四类，变得更加可管理且“没那么容易冲突”了，但是这还远没有解决“污染”。  

以下为了方便表述，我们把common.css称为“common”，把对应页面的css，比如index.html -> index.css、about.html -> about.css，称为“页面css”。

> 这里有个问题需要细致思考一下：**模块的属性**。理论上讲，一个模块应该是公有或者私有的，假设一个模块它基本只可能在某个页面用，或者我们不打算在其他页面用到它，我们可以说这个模块是这个页面的私有模块，比如文章页里的文章列表模块（m-article_list），以及组成这个模块的列表单元元件（u-article\_item），我们基本可以确定这两者不会在其他页面被复用到了，那么它们其实是已经默认私有的属性，没必要放在common里，直接放在article.css就可以了。这样也可以人为地减少common的体积。那么问题来了，如果模块既可以存放在common，又可以存放在页面css，那么我们后续在common中添加公共模块的时候，如何避免模块名已经在页面css中被占用的情况？（即上文对common的设计提问的第4点）

我曾经跟一位同事针对“后续添加公共模块可能与其他页面的私有模块命名冲突”的问题进行探讨，最后我们得出两种解决方案：

1. 默认由common管理所有模块，所有模块默认为公共模块，不允许私有模块；
2. 为公共模块单独使用一种前缀`cm-`来做区分，所有`m-`前缀的模块都是私有模块。

第一种方案会使得common体积非常大，而且会一直增大，不可取；第二种方案显式地声明模块属性，以此来避免冲突，可取。

于是乎又变成了：

```css
/* common.css */

body {
  background: #fff;
  color: #333;
  font-size: 16px;
}

.cm-nav { ... }

.cm-news { ... }

.cm-copy_right { ... }

```

而我们的私有模块是这样的：

```css
/* index.css */

.g-index {
  background: #fff;
  color: #333;
  font-size: 16px;
}

.m-nav { ... }

.m-news { ... }

.m-copy_right { ... }

```

这样子处理之后，我们的公共模块和私有模块之间的命名冲突就解决了，而且也不会出现“一个还什么都没有页面引用了common之后，许多的类名就被占用了”的情况，因为common绝大部分内容都是`cm`模块，而页面自己的css里只能拥有私有的`m`模块。  

显然这种方案是可行的，但是我们会多一种前缀，而且还略丑。但就当时的手打css技术来说，我们没有其他更好的解决方案，这个问题就到这里暂算了结。我们解决了问题，但是方法还不够好，后续等我们提及css预处理语言的时候，我们会提出一些更好的解决方案。  

好，接下来我们思考这样一个问题：假设我们已经写完了index页面，接着写about页面，这时候我们发现，原本在index中的一个模块`m-news`，我们将它归为私有模块，而现在在about中居然也需要用到这一个模块，于是乎，我们重新回到index页面，把`m-news`模块从index.css转移到了common.css当中，并改名为`cm-news`，然后回到index页面，把与`m-news`相关的内容（html、js）都修改成`cm-news`。这还是在我们能够意识到的情况下做的，如果页面多了起来，我们根本没有印象哪个页面是不是也有这样一个模块，要不要把它提升为公共模块。一个月之后，这个项目一个星期前已经搞定了，现在需要进行后续的开发，加多一个contact页面，然后我们又发现，这页面里用到了一个原本我们在about页面里把它划为私有模块的`m-loc`，于是乎，我们又走了一遍提升公共模块的流程。。。  

为什么会出现这样的问题？根本原因在于，我们无法事先规划好所有的模块，无法在一开始就对一个模块的属性清晰地划分。这个问题也基本算是无解。矛盾在于，我们对模块进行了私有和公有的属性划分，却无法事先掌握所有的模块属性，只能走一步算一步，错了就回来再改改。  

解决这问题的办法是，取消对模块的属性划分，所有模块都默认为公共模块，可以随时取用。但是这样就倒退回了我们之前的那种情况，所有的模块都是`m-*`，且都扎堆在common里，导致common的体积过大，所以这个问题只能到这里为止了。  

### 模块

如何界定一个模块？或者说，怎么样才能把一部分代码划分为一个模块？划分的依据是什么？这是我们接下去要探讨的问题。  

#### 设计原则

我们说模块是一个功能相对独立且完整的结构体，其实这应该是 **组件** 的概念，我们这里只从css的范围内来探讨模块化，那么模块的定义就可以缩窄到：一个（组）样式相对独立且完整的类。比如：

```css
/* copy_right */
.m-copy_right {
  color: #ccc;
  background: #666;
  font-size: 14px;
  text-align: center;
  padding: 20px 0;
  line-height: 1.8;
}

/* nav */
.m-nav {
  color: #ccc;
  background: #666;
  font-size: 14px;
}

.m-nav .u-logo { ... }

.m-nav .list { ... }

.m-nav .list .item { ... }

```

原则上来讲，一个css模块应该遵循以下几点要求：

1. 只对外暴露一个类名；

```css
/**
 * 正确示范，所有模块相关的代码都挂在模块的选择器名下
 */
.m-nav { ... }
.m-nav .list { ... }
.m-nav .list .item { ... }

/**
 * 错误示范，暴露了.m-nav和.list两个类名，污染了空间
 */
.m-nav { ... }
.list { ... }
.list .item { ... }
```

2. 不影响周围布局：一般情况下，尽量不要使用一个脱离文档流的布局（既使用了float:left/right，position:absolute/fixed的布局），尽量不要使用外边距（margin）。这是为了使得模块更加稳定、具备更高的可塑性；

```css
/**
 * 正确示范，在common中定义一个模块，在页面css中对模块进行定位和偏移
 */

/* common */
.u-logo {
  width: 100px;
  height: 100px;
}

.cm-news {
  width: 200px;
  height: 100px;
}

/* index */
.u-logo {
  position: absolute;
  left: 20px;
  top: 20px;
}

.cm-news {
  margin-top: 50px;
}
```

```css
/**
 * 错误示范，在common中定义一个模块并固定它的位置
 */

/* common */
.u-logo {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 20px;
  top: 20px;
}

.cm-news {
  width: 200px;
  height: 100px;
  margin-top: 50px;
}
```

3. 模块尽量设计为方便复用的量级，避免大而全，求精巧；

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
</head>
<body>
  <div class="g-index">
    <div class="g-bd">
      <!-- 正确的示范 -->
      <!-- 创建一个大的内容块article_box，而不是一个大模块 -->
      <div class="article_box">
        <div class="hd">
          最新文章
        </div>
        <div class="bd">
          <div class="list">
            <!-- 这里我们把每一个项作为可复用的私有模块 -->
            <div class="m-list_item">
              <img class="cover" />
              <div class="info">
                <div class="title">
                  <a href="#">文章标题</a>
                </div>
                <div class="desc">文章简介</div>
              </div>
            </div>
          </div>
        </div>
        <div class="ft">
          <!-- 这里我们直接引入了一个公共分页模块 -->
          <div class="cm-page">
            <a href="#" class="pg">1</a>
            <a href="#" class="pg">2</a>
            <a href="#" class="pg">3</a>
            <a href="#" class="pg">4</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</body>
</html>
```

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
</head>
<body>
  <div class="g-index">
    <div class="g-bd">
      <!-- 错误的示范 -->
      <!-- 创建一个庞大且不可复用的私有模块m-article_box -->
      <div class="m-article_box">
        <div class="hd">
          最新文章
        </div>
        <div class="bd">
          <div class="list">
            <div class="item">
              <img class="cover" />
              <div class="info">
                <div class="title">
                  <a href="#">文章标题</a>
                </div>
                <div class="desc">文章简介</div>
              </div>
            </div>
          </div>
        </div>
        <div class="ft">
          <div class="page">
            <a href="#" class="pg">1</a>
            <a href="#" class="pg">2</a>
            <a href="#" class="pg">3</a>
            <a href="#" class="pg">4</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</body>
</html>
```

> 值得注意的是，这里的原则第三点，并不只是出于css模块化的考虑，事实上这更适用于 **组件化** 的设计思路，这在后面讲组件化的时候我们会提到。

#### 继承

css的继承也是很简单的，一般来说是有这么几种方式：

1. 在css中并写两个类，如`.cm-nav, .m-nav`，我们知道，这相当于让两个（组）类共享一套样式，然后我们再单独对.m-nav进行补充，实现继承和定制；
2. 在class属性里并写两个类，如`<img class="u-logo logo">`，这样我们只需要在页面css中单独对.logo类进行补充，就可以实现定制；
3. 在页面css中直接对类进行引用
，然后补充样式，实现定制，如`.cm-nav { margin-bottom: 20px; }`；

……

（还有许多黑魔法，欢迎补充）

第一种在我们这套模式里是不可取的，因为我们的公共模块都是放在common里，不可能每继承一次就上去补一个类；  
第二种可取，但是需要多一个近似的类名，不提倡；  
第三种又简单又靠谱。  

```css
/* common.css */

body {
  background: #fff;
  color: #333;
  font-size: 16px;
}

.cm-nav {
  width: 100%;
  height: 50px;
  color: #fff;
  background: #333;
}

```

我们在页面css可以这样用：

```css
/* index.css */

.g-index {
  background: #fff;
  color: #333;
  font-size: 16px;
}

.cm-nav {
  width: 1000px;  /* 样式覆盖 */
  margin: auto;  /* 样式增加 */
}

```


#### 状态

我们在上面讲前缀的时候，提到过一个前缀`z-`，我们说它可以用来表示状态。一个模块是可以有 **状态** 的，当然，这里说的不是状态好状态差的意思（模块还成精了～），这里指的是有多种表现形式，我们举例来说，一个弹窗模块`m-dialog`，它应该至少具备两种状态：显示和隐藏（关闭）。我们用关键字 **active** 来表示这两种状态，添加`z-active`类表示显示，不加表示隐藏。如下：

```css
/* index.css */

.m-dialog {
    display: none;
}

.m-dialog.z-active {
    display: block;
}
```

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
</head>
<body>
  <div class="g-index">
    <div class="g-bd">
      <div class="m-dialog">
        这是一个未激活的弹窗，你看不到！
      </div>
      <div class="m-dialog z-active">
        这是一个已激活的弹窗，你看得到！
      </div>
    </div>
  </div>
</body>
</html>
```

弹窗一个比较有代表性的例子，另一个典型的例子是按钮，用过bootstrap的人都知道，按钮`btn`只需要相对应添加几个状态类，就可以有不同的配色方案，应付不同的场景需要，这其实就是我们的`z-`的含义。`z-`是很常用的，我们应该把我们的模块设计得尽量满足多种可预见的需求，而不是每次都在页面去定制和覆盖基本样式。  

### 总结

到目前为止，我们已经定制了一套基于规范的css模块化生产方式，虽然并不完美，但这却也是一套简单的、零工具、零成本的解决方案，适用于任何项目。  

可以看出，我们这里所谓的模块化，其实是规范化的子集，通过制定了一套规范，才产生了模块。所以css的模块化过程其实是css规范化的过程。  

事实上，由于css本身并不是一门语言，不具备语言的特性，我们只有借助其他方式和工具，才能达到模块化的目的。而目前为止，我们还只是停留在规范的约束方式上，内容看起来比较low :)，没关系，下一节我们会开始介绍【[css预处理语言的模块化实践](http://www.jianshu.com/p/3461c1cefe5c)】。要知道，对于早已习惯sass编程的我，也是闷着一大口气好不容易才写到了这里的。。。   

想必你也留意到了文中多处提到“手打css”这一说法，其实这只是对传统css编程方式的一种戏称，说实话有哪种编程不是手打的，难不成用脚么？哈哈。但是说实话，有了css预处理，模块化才能算得上真正意义的模块化，模块化的意义才凸显出来，因为我们所有的思考与努力，机关算尽，最终的目的都只有一个——提高工作效率。  

### 预告

css预处理语言的出现是一个十分重要的阶段，它直接推进了css的发展，改变了长久以来css的编程方式。  

我们来瞥一眼sass：

```scss
// index.scss

@import './common/normalize';
@import './common/common';
@import './common/mixin';

.g-index {
  @include m-nav;
  @include m-news;
  @include m-copy_right;

  .g-hd {
    .m-nav {
      width: 100px;
      margin: auto;
    }
  }

  .g-bd {
    .m-news {
      margin-top: 20px;
    }
  }

  .g-ft {
    .m-copy_right {
      width: 100px;
      margin: auto;
    }
  }
}
```

sass的选择器嵌套写法是不是亮瞎双眼？文件直接导入是不是很方便？还有变量？还能写函数？？？  

下一节，【[css预处理语言的模块化实践]({% post_url tech/2016-12-10-module-precss %})】我们来看css预处理语言如何使模块化更加美好。