---
layout: post
title: css预处理语言的模块化实践
date: 2016-12-10 00:57:00 +0800
categories: posts tech
nav: Tech
---
编写css是前端工作中，一项普通而又频繁的劳动，由于css并不是一门语言，所以在程序设计上显得有些简陋。对于小型项目来说，css的量还不至于庞大，问题没有凸显，而如果要开发和持续维护一个较为大型的项目，那就需要对css进行管理和规范了，否则会发生不可挽回的后果（吓唬谁呢？？）。  
<!--more-->
### 背景

上一节【[从css谈模块化]({% post_url tech/2016-11-24-module-css %})】我们通过规范的约束，将css的编写方式进行了优化和改进，形成一种可持续发展的路线。但还是遗留了一些问题：冗余。虽然我们通过定义公共模块和私有模块，来委婉地分担common的体积，但common的体积还是太大了，而且从设计上考虑，我们应该尽量多地提炼公共模块，以便更好地实现复用。最理想的情况是，所有的模块都寄存在一个公共的库里，哪里需要用到就从库中直接调过来。这个美好的愿望不是不可实现的，借助预处理语言，我们可以很轻易地完成这事情。  

> 预处理语言是一种类css的语言，我们知道css本身不是语言，而预处理语言的诞生，就是为填补这一部分语言功能。它实现了变量、函数、混合的定义，以及文件的引用、合并、压缩功能，使得css也能面向对象，应付复杂庞大的业务。

目前流行的预处理语言主要有两种：less和sass。作为学习，两者都可以入门一下，而作为工作，尽量熟悉一种。我比较常用sass，所以以下内容都是以sass为基本语言做介绍，两者在特性上有很多相似的地方，所以大家不必担心实现上有什么千差万别。  

### sass

基本语法可以到[官网](http://sass-lang.com)（英语）或者[w3cplus sass guide](http://www.w3cplus.com/sassguide)（中文）查看学习，我们这里只简单地过一遍，讲一些我们需要用到的内容，不会面面俱到。  

> sass有两种后缀名文件：一种后缀名为sass，不使用大括号和分号；另一种就是我们这里使用的scss文件，这种和我们平时写的css文件格式差不多，使用大括号和分号。而本教程中所说的所有sass文件都指后缀名为scss的文件。在此也建议使用后缀名为scss的文件，以避免sass后缀名的严格格式要求报错。  ——摘自w3cplus sass guide

#### 1、嵌套（非常重要的特性）

> sass的嵌套包括两种：一种是选择器的嵌套；另一种是属性的嵌套。我们一般说起或用到的都是选择器的嵌套。  ——摘自w3cplus sass guide

> **选择器嵌套** 所谓选择器嵌套指的是在一个选择器中嵌套另一个选择器来实现继承，从而增强了sass文件的结构性和可读性。在选择器嵌套中，可以使用&表示父元素选择器。  ——摘自w3cplus sass guide

```scss
// index.scss

.g-index {
  ...

  .g-hd {
    ...

    .m-nav { ... }
  }

  .g-bd {
    ...

    .m-news { ... }
  }

  .g-ft {
    ...

    .m-copy_right { ... }
  }

  .m-dialog {
    display: none;

    &.z-active {  // 留意此处&的用法
      display: block;
    }
  }
}
```

编译后：

```css
/* index.css */

.g-index { ... }
.g-index .g-hd { ... }
.g-index .g-hd .m-nav { ... }

.g-index .g-bd { ... }
.g-index .g-bd .m-news { ... }

.g-index .g-ft { ... }
.g-index .g-ft .m-copy_right { ... }

.g-index .m-dialog {
  display: none;
}

.g-index .m-dialog.z-active {  // 留意此处&的编译结果
  display: block;
}

```

是不是爽歪歪？再也不用一遍一遍地去复制和修改一大堆的选择器，也不需要去整理它们之间的关系，只需要嵌套一下，所有的关系就如同直接看dom一样简单明了了！解放双手，解放双眼，同时还提高效率。值得留意的是，我们书写sass的时候，应该尽量保持sass的嵌套顺序与dom一致，注意，是嵌套顺序一致，而不是层次一致，因为并不是dom里的所有元素都需要写样式。  

我们再来提一个场景，说明sass的嵌套写法利于维护，假设`g-bd`下原本有个模块`m-article_box`，现在我们要把`m-article_box`从`g-bd`迁移到`g-hd`中（当然这个需求有些不合理～），我们来看原始代码：

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

```css
.g-bd { ... }
.g-bd .m-article_box { ... }
.g-bd .m-article_box .hd { ... }

.g-bd .m-article_box .bd { ... }
.g-bd .m-article_box .bd .list { ... }
.g-bd .m-article_box .bd .list .item { ... }
.g-bd .m-article_box .bd .list .item .cover { ... }
.g-bd .m-article_box .bd .list .item .info { ... }
.g-bd .m-article_box .bd .list .item .info .title { ... }
.g-bd .m-article_box .bd .list .item .info .desc { ... }

.g-bd .m-article_box .ft { ... }
.g-bd .m-article_box .ft .page { ... }
.g-bd .m-article_box .ft .page .pg { ... }
```

按照css的方式，我们就要把所有跟`m-article_box`有关的部分，从`g-bd`全部复制到`g-hd`里去。这还是在模块的书写符合规范的情况下，如果这个模块书写不符合规范，没有把全部结构都挂在`m-article_box`类下，那真的就是灾难了～而现在使用sass的话，我们只需要把`m-article_box`的区块整个从`g-bd`剪切到`g-hd`就完事了（这里为了突出修改的工作量大，我特地把整个模块结构都写全了——才不是为了凑字数。。。）：  

```scss
// 修改前
.g-hd { ... }

.g-bd {
  .m-article_box {
    .hd { ... }
    .bd {
      .list {
        .item {
          .cover {
            ...
          }

          .info {
            .title {
              ...
            }

            .desc {
              ...
            }
          }
        }
      }
    }

    .ft {
      .page {
        .pg { ... }
      }
    }
  }
}

// 修改后
.g-hd {
  .m-article_box {
    .hd { ... }
    .bd {
      .list {
        .item {
          .cover {
            ...
          }

          .info {
            .title {
              ...
            }

            .desc {
              ...
            }
          }
        }
      }
    }

    .ft {
      .page {
        .pg { ... }
      }
    }
  }
}

.g-bd { ... }
```

非常方便，而且不容易出错。  

#### 2、变量（variable）  

咱们直接上代码：  

```scss
// index.scss

$fontSize: 16px;
$grey: #ccc;

.m-nav {
  font-size: $fontSize;
  color: $grey;
}
```

编译结果：

```css
/* index.css */

.m-nav {
  font-size: 16px;
  color: #ccc;
}
```

写过代码的人都熟悉 **参数** 的用法吧，太简单太直白了不想说太多，自己意会吧。


#### 3、函数（function）  

```scss
// pixels to rems

@function rem($px) {
    @return $px / 640 * 16rem;
}
```

太简单了直白了不想说太多，自己意会吧。  

#### 4、混合（mixin）  

混合，顾名思义，就是混合的意思。。。也就是我们可以事先定义一段代码块，在需要使用到的地方，直接引用（include），而在引用之前，这段代码都不会出现在编译文件中，也就是不会生成任何内容。  

这也是非常重要的一个特性！我们知道common的体积非常大，而体积大的根本原因是它存放了许许多多的模块。我们设想一下，如果将每一个模块都打包成mixin，那common不就减肥成功了？！多年的顽疾终于看到希望，没有比这更让人惊喜的了！我们这就上车：

```css
/* common.css */

.m-nav { ... }
.m-news { ... }
.m-copy_right { ... }
```

改造后

```scss
// common.scss

@mixin m-nav {
  .m-nav { ... }
}

@mixin m-news {
  .m-news { ... }
}

@mixin m-copy_right {
  .m-copy_right { ... }
}


// index.scss

.g-index {
  @include m-nav;
  @include m-news;
  @include m-copy_right;
}
```

#### 5、import  

这个属性很眼熟？没错，事实上，css本身就有这个属性实现，我们可以在css文件中直接使用`import`来引入其他文件。那么css的`import`和sass的`import`有什么区别？从含义和用法上来说，没有区别，区别在于工作原理。css的import是阻塞的，而sass的import在编译后，其实是合并文件，最后只产出一个css文件，而css则没有合并，该多少个文件就还是多少个文件。  

注意：  
1. 只有import一个.sass/.scss文件的时候，才可以省去后缀名，如果是直接import一个.css文件，要补全文件名；
2. import之后的分号`;`不要漏写，会报错；
3. sass如果import的是一个.css文件的话，那它的作用就跟css原生的import作用一样，只有import一个sass文件的时候，才是合并文件。

如下：  

```scss
// index.scss
@import 'common';
@import 'a.css';
```

编译结果：  

```css
/* index.scss */

.m-nav { ... }
.m-news { ... }
.m-copy_right { ... }

@import url('a.css');
```

css的import之所以没有被普遍使用是有原因的。我们可以大概猜到它的工作原理：a.css import b.css以后，当浏览器加载到页面中的a.css的时候，已经准备按照a.css的内容来渲染页面了，刚解析到第一行，发现a.css居然还import了一个b.css，于是它不得不先放下a.css（既阻塞a.css），去加载b.css，直到b.css加载完，并且优先解析它，然后才开始回来解析a.css——鬼知道b.css会不会又import了c.css……这直接导致了渲染工作滞后，引发性能问题。  

说实话我还不如直接用两个link标签去同步加载a.css和b.css，效率会高一些。  

所以css的import基本是被抛弃了的属性。  

sass的import主要的好处就是把文件合并了，减少了请求。原本需要link好几个css文件的页面，现在只需要一个。  

### 模块化

终于要开始干点正事了，首先我们来回顾一下，上一节我们以规范为基础构建的模块化项目，遗留了一些什么问题。  

1. **冗余** 体积庞大的common；
2. 使用`cm-`模块区别`m-`模块，使得后期开发过程中，`m-`模块向`cm-`模块转变过程比较繁琐；

……  

好像，问题也不是特别多，我们一个一个解决。  

为了方便，在这里我们把每个页面所对应的scss文件叫做 **页面scss**；把变量、函数、混合等（没有被引用或者执行的情况下）编译后不产生实际内容的代码叫做 **定义类代码** ，那么相对应的其他内容就是 **实际内容代码**。  

#### 1、mixin.scss

我们知道，一方面，在common中过多地添加模块最终会导致common的体积过大，使得资源冗余，另一方面，为了方便维护，我们又希望尽量多地把模块公有化。  

这是一对矛盾，仅靠css本身是无法解决的，但sass可以！如果我们使用mixin来代替直接书写模块，由于mixin并不直接生成代码，而是通过主动引用，才能生成对应内容，那么理论上，common就可以无限多地存放模块而不必占用一点空间！  

（注意，这里说的是理论上，实际应用中，文件太过庞大的话，免不了还是要受到命名冲突的限制的，不过这问题不大。）  

说干就干，我们把common中的模块全部打包成mixin：  

```css
/* common.css */

.m-nav { ... }
.m-news { ... }
.m-copy_right { ... }
```

改造后

```scss
// common.scss

@mixin m-nav {
  .m-nav { ... }
}

@mixin m-news {
  .m-news { ... }
}

@mixin m-copy_right {
  .m-copy_right { ... }
}
```
调用方式如下：

```scss
// index.scss

@import 'common'; // 记得先引入common

.g-index {
  @include m-nav;
  @include m-news;
  @include m-copy_right;
}
```

原本我们会在每个需要用到公共模块的页面中，先引用common，然后再引用页面css，而现在，我们只需要在页面scss中直接`@import common;`就可以了。  

使用common：

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
  ...
</body>
</html>
```

改造后：

```scss
// index.scss
@import 'common';
```

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
  <link rel="stylesheet" type="text/css" href="./style/index.css">
</head>
<body>
  ...
</body>
</html>
```

很完美，  
——至少目前为止是这样。  

我们思考一个问题，common除了存放模块之外，还有没有其他内容？答案是肯定的，大家一定知道有个东西叫做`css reset`（或者normalize.css），这肯定是全局的；另外，如果是做后台管理系统的话，可能还会有`bootstrap`。当然，还有一些自定义的全局的样式，比如常见的`.clearfix`，等等。  

这些东西目前我们也堆积在common当中，而且合情合理，因为它们都是全局的样式。但是对比起mixin来说，这些实际内容代码显得很少量，有种被淹没的感觉，使得整个common看上去就像只有mixin。但是这些实际内容代码的作用却又非常重要。为了使common的构成更加直观，我们把mixin全部都抽离出来，单独存放一个叫做mixin.scss的文件中，然后在common引用它，这样，mixin的管理更加的规范，而且common的结构也更加清晰了。  

抽离mixin还有另外一个重要原因，后面会讲到的，我们希望mixin作为一个纯粹定义类代码文件，随处可以引用而不会生成多余的代码。  

原本我们会在每个需要用到公共模块的页面中，先引用common，然后再引用页面css，而现在，我们只需要在页面scss中直接`@import mixin;`就可以了。  

使用mixin：  

```scss
// index.scss
@import 'common';  // 引入common，如果有需要，common里一样可以引入mixin
@import 'mixin';  // 引入mixin

.g-index {
  ...
}
```

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
<head>
  <title>index</title>
  <link rel="stylesheet" type="text/css" href="./style/index.css">
</head>
<body>
  ...
</body>
</html>
```

#### 2、common.scss

好，抽离了mixin之后，我们现在来重新看回common，common里应该是些什么样的内容。上面的内容我们稍稍提到了一点，我们来展开一下。  

##### 2.1、css reset(normalize)

我们知道浏览器千差万别，各浏览器的默认样式也是不尽相同，最常见的比如body的默认内边距，p标签的默认内边距，以及ul/ol等等。这些不统一的默认样式经常让我们感到头疼，所以就有人提出一开始写样式就先把它们消除的想法，于是就催生了后来非常流行的`reset.css`。  

起初的reset.css很简单，大概是这样的：  

```css
html, body, h1, h2, h3, h4, h5, h6, div, dl, dt, dd, ul, ol, li, p {
  margin: 0;
  padding: 0;
}
```

没错，就是把几乎所有会用到的标签都给去了内边距和外边距，简单粗暴，这样所有的标签就都统一了，而且在不同的浏览器下也是统一的。  

其他的部分每个人有各自的补充，比如有人会把`h1～h6`的所有字号给定义一遍，以保证在不同浏览器下他们有统一的大小；有人会给`a`标签设置统一的字体颜色和hover效果，诸如此类等等。  

很好，没毛病。我们把这些统称为`css reset`，然后再统一封装到一个叫做`reset.css`的文件中，然后每个页面都引用。  

这种方式一直以来都挺实用，而且大家也都这么用，没出过什么问题。只是后来有人提出，这种方式太过粗暴（居然还心疼浏览器了）。。。而且会降低页面渲染的性能，最重要的是，这使得我们原本设计出来的表达各种含义的标签儿们，变得毫无特点了。。。  

说的好有道理，如果你家里所有人名字不一样但是都长一个样，还有啥意思。  

于是，就出现了`normalize.css`，normalize的目的同样是为了统一各个浏览器下各不相同的默认样式，不过它并不是简单粗暴地全部抹平，而是根据规范，来人为地把那些不符合规范的默认样式“扶正”，从而达到统一各个浏览器默认样式，同时保留各个标签原有特点的目的。  

我们不能说reset与normalize这两种思想孰好孰坏，只能说各有各的特点和作用，它们的存在都是为了解决同样的问题。  

##### 2.2、插件

一般来说，一个ui插件都会至少包括一个css文件，像bootstrap、datepicker等等。假设我们项目中需要以bootstrap为基础框架，实现快速开发，那么这时候我们就需要在项目中全局引入bootstrap.min.css，当然，还有bootstrap.min.js。说到全局暴露，我们第一时间想到的是common，没错，我们可以在common中引入。  

有人问，插件的.css文件怎么import？额，改一下扩展名为.scss就可以了，scss是兼容原生css语法的～  

所以最终，我们的common大概是这样子的：

```scss
// common.scss

@import './reset';
@import './bootstrap.min';
@import './mixin';
```

事实上，如果我们不需要使用到 **mixin.scss** 中的变量和mixin的话，我们可以不引用它。  

那么我们的页面scss应该是这样的：  

```scss
// index.scss

@import './common';
@import './mixin';

.g-index {
  ...
}
```
干净，整洁。  

#### 3、mixin编写规范

每添加一个新角色，我们就要及时给它设置规范，好让它按照我们的期望工作别添乱。我们接下来来归纳一下mixin的书写规范。  

场景一：项目里有mixin.scss、a.scss（假设这是某个功能文件）、index.scss三个文件，mixin中定义了一个变量`$fontSize: 16px;`，a中定义了一个变量`$color: #ccc;`，我们在index中同时引用这两个文件，那么我们在index中是可以直接使用`$fontSize`和`$color`这两个变量的——我的意思是，尽管在index中我们并没有看到这两个变量的声明和定义，但它们就这么存在了。  

这是好事还是坏事呢？直觉告诉我，这可能有问题。没错，这是不是跟我们之前讨论过的 **污染** 很像？只不过我们之前是引用了common之后，index什么都还没写就已经被占用了很多模块名，而现在是因为引用了其他文件，而占用了index的很多变量名。另外，就维护的角度来看，这也是有问题的，如果我不事先告诉你，或者你不事先看过一遍mixin和a，你知道index中的`$color`是哪里来的吗？假设我们需要字体大小，你知道去哪个文件修改吗？另外，你怎么保证同时引用mixin与a的时候，他们之间有没有可能存在同名的变量？那谁覆盖谁呢？这些问题看起来很小，但是当你项目规模大的时候，这可能是无法挽回的灾难（吓唬谁呢？？？）。  

场景二：假设我们的项目有一个主题色，边框、tab背景、导航栏背景，以及字体颜色等等，都是这个主题色，为了方便使用，不想总是用取色器去取值，于是我们在mixin中定义了一个全局变量`$color: #ff9900`，然后就可以愉快地到处使用了！  

整个网站开发完了，一个月后，设计师突然过来跟你说：“老板说，这个主题色要改改，有点土，咱们换个大红。”，于是你一脸不情愿然而内心却窃喜地打开mixin，把$color的值改成了大红，然后得瑟地对设计师说：“幸好我早有准备，搞定了，你看看吧。”，保存，打开页面一看，设计师和你的脸都绿了，页面怎么这么丑，有些字原本是主题色，但背景是红色，而现在一改，整块都变成红的，内容都看不清了，有些边框原本就是红色的，但是字体是原本的主题色，然而现在一改，边框跟字体都变成红的了。  

设计师：“不不不，我只是想把背景颜色改一下。”  
你：“你不是说改主题色吗？那就是所有的地方啊。”  
设计师：“不用，改背景就好了。”  
你：“不行啊。。。”  
设计师：“为什么不行，不就是改个背景颜色吗？怎么设置的就怎么改回来呀。”  
你：“不是你想的那么简单。。。”  

……  

好吧我就是吓唬你的，你要是特能折腾那么这些都不叫事儿。  

所以我们需要对（全局）变量进行管理，就像我们当初管理mixin那样，不能想在哪里定义就在哪里定义，也不能动不动就修改一个全局变量：  

1. 全局变量只在mixin中定义，其他scss文件定义的变量（无论是暴露到全局还是局部）都只看作局部变量，不在当前文件以外的地方使用（即便是在能引用到的情况下，也避免使用）；
2. 需要使用全局变量的地方直接import mixin；
3. 一般来说，定义全局变量应该慎重，全局变量的数量应该尽量少；
4. 尽可能不改动，如果需求变动，除非是对用途十分确定的情况，否则请新增一个全局变量来逐步替换需要修改的地方；
5. 不要使用太过笼统的名词来作为全局变量，比如`color`，建议直接是用色值的描述，比如`$orange: #ff9900`，这使得我们在维护上更方便扩展，如果色值需要修改，但是又不是所有的地方都需要修改，那么我们可以新定义一个变量来扩展它，比如`$red： red`。  

这些点说起来都有点飘忽，事实上也确实很难说明白为什么要这么做，毕竟都是经验总结，所以大家不妨先熟悉使用sass一段时间之后，再来细细思考这些问题。  

注意，以上讲的这些都不是死规定，在某些时候，这个规范是需要根据实际项目而做调整的，就比如我们之后要讲到的SPA。十全十美的项目是不存在的，也不存在能适用所有项目的开发模式，因地制宜才能更好地解决问题。而且我们目前提到的问题都不是致命的，致命的问题在上一节我们制定规范的时候已经避开了。  

### 调用模块

问题，在哪里调用模块？  
答，页面scss。  

在页面scss中调用模块是一个好习惯，它使得我们在每个页面所用到的模块既是一致的又是互相隔离的，不像在common中直接引用模块那样，使得一个页面scss还没有内容的时候就已经被很多模块名污染了。  

再提个问题，在页面scss的哪里调用模块？  

例一，根类外：

```scss
// index.scss

@import './common';
@import './mixin';

@include m-nav;
@include m-news;
@include m-copy_right;

.g-index {
  ...
}
```

例二，根类内：

```scss
// index.scss

@import './common';
@import './mixin';

.g-index {
  @include m-nav;
  @include m-news;
  @include m-copy_right;

  ...
}
```

目前为止，这两种方式都是可以的，至于我为什么用“目前为止”这个词，那是因为我们后面将要讲到的SPA，如果用例一的方式是有问题的。所以我比较鼓励使用例二的方式。当然，我说了，目前为止例一也是没问题的。  


### 性能优化

目前为止，我们的模块化工作已经算是完成了，其实已经可以收工了。不过我们还是可以稍微做一下优化。  

#### 1、缓存

我们需要考虑一个问题：缓存。  

缓存是我们web开发中最常见的情况之一，很多时候我们都需要跟缓存打交道，特别是在做性能优化的时候。  

一般来说，静态资源在被加载到浏览器之后，浏览器会把它本地缓存下来，以便下次请求同个资源的时候可以快速响应，不需要再去远程服务器加载。  

我们就css来说，假设我们按照原来的方式，使用多个link去加载reset、bootstrap、common、index这几个文件的话，这几个文件都会被缓存下来，以使得下次再访问这个页面，这个页面的加载速度会快很多。  

如果是从index页面跳转到about页面呢？你会发现也很快，因为about页面的全局css（reset、bootstrap、common）和index页面是一样的，而它们在你访问index的时候，已经加载过了，得益于缓存的作用，之后的页面打开都快。  

我们现在的方式是，一个页面所用到的所有css文件都被合并成一个，也就不存在相同的文件可以利用缓存这样的优势了。  

那我们有办法改进吗？有的！我们只需要把common独立出来，那么common就可以做为被缓存的公共文件了。最终我们从一个页面只引用一个文件变成了一个页面引用两个文件，即common和页面css：

```scss
// common.scss

@import './reset';
@import './bootstrap.min';
@import './mixin';
```

```scss
// index.scss

@import './mixin';

.g-index {
  ...
}
```

注意，不同于之前，我们这里的index.scss不再引入common.scss，所以我们最终是得到了两个css文件，而common.css是在所有页面中通过link标签引入的。  

如此一来，我们就实现了既能够合并文件，减少请求数，又可以利用缓存，提高加载速度。  

#### 2、压缩

代码压缩是优化工作中最基本的一步，css的压缩空间是很大的，尤其是我们这种 **垂直的书写方式** ，压缩起来是相当高效的。  

在sass中这很简单，sass在编译的时候提供了几种模式，其中的`compressed`模式是最高效的压缩模式，记得在编译打包的时候选择compressed模式就行了。  

### 总结

总的来说，预处理语言在使我们编程更加美好的同时，也使得规范更加的完善。在css本身无法实现的情况下，我们通过工具来完成了模块化开发。  

我不会讲如何去安装和配置sass环境，因为这些[w3cplus sass guide](http://www.w3cplus.com/sassguide)有详细的介绍了，建议使用nodejs的方式，不会捣鼓nodejs/npm的前端不是好前端。  

最后，我们回到一开始提到的问题——为什么要模块化？现在我们可以先从css的工作来回答，从某种意义上讲，模块化提高了我们编程能力和解决问题的能力，使得构建一个庞大而可扩展可维护的项目成为可能，使得我们能够以架构的思维和眼光去搭建整个项目。  

下一节本想就着这一篇的内容直接说SPA的，但是发现SPA已经是组件化的内容了，也就是说它不单单是css的内容，需要更多的知识点来搭台，所以还是先放着，下一节，我们来了解【[html与模板引擎的模块化实践](#)】（近期推出，敬请留意）