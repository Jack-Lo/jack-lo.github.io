---
layout: post
title: redux在react中的应用（基础篇）
date: 2017-02-25 01:27:00 +0800
categories: posts tech
nav: Tech
---
上一篇[【react的SPA实践】]({% post_url tech/2017-02-22-react-spa %})里，我们留下了一些问题，比如深层嵌套的组件之间的通讯问题。  
<!--more-->
虽然我们通过尽量减少深层次嵌套的方式，可以规避这个问题，但是这毕竟没有解决问题。  

这一篇我们主要讲react如何搭配 **redux** 使用，从而构建一个更完（niu）善（bi）的react应用。  

官方文档对redux的介绍：  

> Redux是javascript状态容器，提供可预测化的状态管理。

注意，redux的集成是非必选的。通过之前的内容介绍我们了解到，单枪匹马的react构建的页面也可以运作得很好。  

你可能会对redux的编程方式感到无所适从。  

然而，不管是出于构建大型应用考虑，还是作为进一步的技能提升，redux绝对是一个值得你花时间去学习的框架。  

我到底是想说什么。。。  

好吧，啰嗦就到此为止，下面介绍一下本篇的内容大纲：  

* store
* action
* reducer
* dispatch
* middleware


## 引子  

我们知道在react中数据是单向流动的。   

啥？我之前没有说过？？？   

好吧，漏了。  

所谓的数据单向流动，顾名思义，就是数据是从一个方向传播下去的意思。  

我们假设数据中心是一个负责发行身份证的地方政府，每个人都可以到这里来领取自己的身份证，但是如果你对自己的身份证头像不满意，你不能自己去村口拍个十块钱的大头贴贴上去，也不能自己修改上面的个人信息，这样的身份证是不被承认的（弄不好可能还得坐牢），正确的操作方式应该是，带着你的修改意见，去到地方政府，找相关工作人员帮你重新办理，办理完之后再发放给你。  

这个例子当中，数据中心我们视为父组件，身份证就是子组件通过父组件接收到的`props`，你不能直接修改自己的身份证，也就是你不能直接`this.props.name = '葬爱'`，只能通知父组件去修改这个props，即调用父组件的提供的changeName方法来操作：`this.props.changeName('葬爱')`，这个过程事实上是修改了父组件的state，从而使得子元素接收到的数据也发生了改变。  

所以数据由始至终都是从父元素流向子元素，我们称为数据单向流动。  

我们回想一下我们之前构建过的所有react应用，数据都是由最顶层父组件（页面组件）一层层向下传递的。  

这也是深层次的组件之间通讯困难的原因：数据的传递是单向的，子组件的数据只能就近获取，但是真正的数据源却离得太远，没有捷径可以直接通知数据源更新状态。  

redux的出现改变了react的这种窘迫处境，它提供了整个应用的唯一数据源`store`，这个数据源是随处可以访问的，不需要靠父子相传，并且还提供了（间接）更新这个数据源的方法，并且是随处可使用的！  

天呐，这也太好了吧。  

子组件A：老子再也不用千里迢迢等你传个props了！  
子组件B：老子再也不用盼天盼地等你传个handler了！  

然而，事情并没有那么简单。  

前方高能！  


## store  

**store** 是redux提供的唯一数据源，它存储了整个应用的state，并且提供了获取state的方法，即`store.getState()`。  

就没了？？？修改数据的方法还没说呢？？？  

store是只读的。  

redux没有提供直接修改数据的方法，改变state的唯一方法就是触发（**dispatch**） **action** 。  


## action  

**action** 是一个用于描述已发生事件的普通对象。  

简单来说，就是“你干了一件什么事情”。  

但是单单讲了你干的事情，我们并不知道你干的这件事产生了什么牛逼效果，于是有了一个专门负责描述某个行动对应产生某种效果的机构，叫做 **reducer** 。  

## reducer  

**reducer** 只是一个接收state和action，并返回新的state的函数。  

它就像是一部法典，根据你所做的事情，提供对应的后果，这个后果直接对数据源起作用。  

什么鬼比喻。。。  

并且，这部法典应该尽可能覆盖所有的犯罪事件类型！  

你还来。。。  

ok，现在我们知道怎么修改数据源了，首先必须先定义好我们即将做的事情，也就是定义一个action，跟着，我们需要相对应地补充我们做的这件事要怎么影响数据源，于是我们根据这个action补充了一个reducer，最后我们触发这个action：`store.dispatch(action)`，数据源就会根据reducer定义好的规则来更新自己了。  

就是这样！  

是不是很简单？  

是呀是呀！  

但好像还是没什么概念，连个action长什么样都不知道。  

没事，我们接着讲。  


## 场景代入  

我们假设整个应用只有两个组件，一个身份证组件，一个弹窗组件，那么应用的`state`树应该是这样子的：  

```javascript
// store.getState()

{
  card: {
    name: 'Jack',
    picture: 'a.jpg'
  },
  dialog: {
    status: false
  }
}
```

action本质上是一个普通的js对象，因为它只是一个用来描述事件的对象，先来两个现成的action看看：  

```javascript
{
  type: 'CHANGE_NAME',
  name: '葬爱'
}

{
  type: 'CHANGE_PICTURE',
  picture: 'b.jpg'
}
```

发现了吗？action都会带一个type属性，这个属性是必选的，而其他的内容，比如name、picture等等，都是可选的，它们是由action携带，最后传递给reducer的内容，就好比我说我要改名字，这是事件，但是我没有说我要改成什么名字，这个操作就不完整，所以我还需要补充说，我要改名叫“葬爱”，所以我还需要提供一个name给你，这才能完整实现一个动作。这些附属的参数，我们称为 **payload**（载荷）。  

我们说过payload是可选的，也就是说，有些动作的触发是不需要其他信息的，比如“激活弹窗”、“关闭弹窗”等等，这类动作只需要一个type就可以传达意思了：  

```javascript
{
  type: 'SHOW_DIALOG'
}

{
  type: 'CLOSE_DIALOG'
}
```

于是，一个完整的触发动作是这样的：  

```javascript
// 修改名字
dispatch({
  type: 'CHANGE_NAME',
  name: '葬爱'
})
// 激活弹窗
dispatch({
  type: 'SHOW_DIALOG'
})
```

我们已经知道如何触发一个动作，现在我们来了解如何接收并处理这个动作，也就是补充一个reducer。  

可以认为，reducer就是根据传入的各种action不同，相对应对state进行处理，最后返回一个新state的函数。  

那么可以推断出这个函数需要的参数至少是：当前的state，以及一个action。  

事实也正是如此，下面是一个真实的reducer：  

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'CHANGE_NAME':
    return {
      card: {
        name: action.name, // 使用action携带的新name
        picture: state.card.picture  // 不需要修改，使用旧state的值
      },
      dialog: state.dialog  // 不需要修改，使用旧state的值
    }

    case 'SHOW_DIALOG':
    return {
      card: state.card,  // 不需要修改，使用旧state的值
      dialog: {
        status: true
      }
    }

    case 'CLOSE_DIALOG':
    return {
      card: state.card,  // 不需要修改，使用旧state的值
      dialog: {
        status: false
      }
    }

    default:
    return state  // 没有匹配的action type，返回原来的state
  }
}
```

如上，reducer接收一个修改前的state和一个action，然后通过判断actionType的方式来进行不同操作（没有匹配的actionType则默认返回原state），而这个操作的最终目的就是 **拼装** 一个新的state，并最终return，这样就达到更新state的目的了！  

看到这里你可能会有疑问，为什么不直接拿state，然后修改它`state.card.name = action.name`，最后`return state`不就好了吗？  

这是一个不好的实践，因为state是一个对象，直接修改state是会对其他引用了state的地方产生影响的，这种影响我们称为 **副作用** ，而redux规定reducer必须是 **纯函数** ，纯函数是没有副作用的。  

> reducer 一定要保持纯净，只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。  ——redux官方文档

关于纯函数的概念，可以查找相关介绍，这里我们不细讲，只需要记住，纯函数的特征是，只要两次输入的值是一样的，那这两次输出值就一定是一样，不可能出现差异。  

我们回到上面的示例代码来，仔细观察这个state，我们发现它由两部分构成，分别是`card`和`dialog`，但是这两者之间并没有关联，那么我们可不可以把它们拆分出来，分别管理呢？  

我们可以试试看：  

```javascript
function cardReducer(state, action) {
  switch (action.type) {
    case 'CHANGE_NAME':
    return {
      name: action.name, // 使用action携带的新name
      picture: state.card.picture  // 不需要修改，使用旧state的值
    }

    default:
    return state  // 没有匹配的action type，返回原来的state
  }
}

function dialogReducer(state, action) {
  switch (action.type) {
    case 'SHOW_DIALOG':
    return {
      status: true
    }

    case 'CLOSE_DIALOG':
    return {
      status: false
    }

    default:
    return state  // 没有匹配的action type，返回原来的state
  }
}

function reducer(state, action) {
  return {
    card: cardReducer(state.card, action),
    dialog: dialogReducer(state.dialog, action)
  }
}

export default reducer
```

这个由两个reducer组成的大reducer所实现的功能，跟之前只有一个reducer实现的功能是没有差别的！我们成功地将原本比较笼统的reducer拆分成了多个专注于不同功能的reducer，如果你愿意的话，还可以继续拆下去，但目前来看没有这个必要。  

这是个很重要的思想！强调一遍，拆分reducer是一个很重要的思想，这是我们之后编写reducer的最基本方式。  

我们可以看得出来，reducer的拆分规则是跟state的结构紧密相关的，所以很多时候拆分reducer并不是什么困难的事情。  

而且过程还很爽有木有。。。  

好吧，可能只是我的个人感受。  


### combineReducers  

值得一提的是，redux对reducer的合并提供了一些便捷的方法，我们可以这么写：  

```javascript
function card(state, action) {
  switch (action.type) {
    case 'CHANGE_NAME':
    return {
      name: action.name, // 使用action携带的新name
      picture: state.card.picture  // 不需要修改，使用旧state的值
    }

    default:
    return state  // 没有匹配的action type，返回原来的state
  }
}

function dialog(state, action) {
  switch (action.type) {
    case 'SHOW_DIALOG':
    return {
      status: true
    }

    case 'CLOSE_DIALOG':
    return {
      status: false
    }

    default:
    return state  // 没有匹配的action type，返回原来的state
  }
}

export default combineReducers({
  card,
  dialog
})
```

需要特别注意，使用combineReducers来合并reducer，需要子reducer的名字跟对应要接收的state的key一致，所以你看到上面我们把原来的子reducer名字分别从cardReducer和dialogReducer，改为了card和dialog。  

其实，好像也没多方便啦。。。  

不过可能逼格高一些就是了。  

使用combineReducers的目的是为了减少模板代码，但是在这里其实并没有显得多必要，反而，这种方式可能会限制子reducer的命名，显得不灵活，所以不建议使用。  

以上，就是redux的全部思想了，真的，除了获取state和改变state之外，我们没有更多的需求了。  

接下来我们就介绍如何在react项目中集成redux。  


## 集成  

假设我们原有项目（react+react-router）的入口文件`App.jsx`如下：  

```javascript
import React, { Component, PropTypes } from 'react'
import ReactDom from 'react-dom'
import { Router, Route, IndexRoute, hashHistory } from 'react-router'

import Index from './containers/index'
import About from './containers/about'

ReactDom.render(
  <Router history={hashHistory}>
    <Route path="/" component={Index}/>
  </Router>,
  document.getElementById('root')
)
```

在react中使用redux需要装两个包：  

```shell
npm install redux react-redux --save
```

然后我们在现有项目中引入：  

```javascript
import { createStore, applyMiddleware } from 'redux'
import { Provider, connect } from 'react-redux'
```

`createStore`是用来创建store的，`applyMiddleware`是用来整合接入`middleware`的，这个我们后面再介绍，`Provider`是用来实现`store`的全局访问的，而`connect`则是用来针对某个展示组件，创建包裹这个组件的容器组件的，有点绕，这个我们也留在后面再介绍。  

```javascript
import React, { Component, PropTypes } from 'react'
import ReactDom from 'react-dom'
import { Router, Route, IndexRoute, hashHistory } from 'react-router'

import Index from './containers/index'

// 引入redux
import { createStore, applyMiddleware } from 'redux'
import { Provider, connect } from 'react-redux'

// 引入reducer
import reducer from './reducers'

// 创建一个初始化的state
var initState = {
  card: {
    name: 'Jack',
    picture: 'a.jpg'
  },
  dialog: {
    status: false
  }
}

// 创建store
const store = createStore(reducer, initState)

ReactDom.render(
  <Provider store={store}>
    <Router history={hashHistory}>
      <Route path="/" component={Index}/>
    </Router>
  </Provider>,
  document.getElementById('root')
)
```

这样，我们就集成了redux！  

请仔细思考以上带注释的部分，结合之前介绍的内容，重新整理一遍思路，加深理解。  

整个结构搭建好了之后，就可以来实际操作了！  


## connect  

我们把视线转移到页面上，这个例子当中有一个页面`Index`，我们来看：  

```javascript
import React from 'react'

import Card from '../../components/Card'
import Dialog from '../../components/Dialog'

const Index = React.createClass({
  render () {
    return <div className="g-index">
      <Card />
      <Dialog />
    </div>
  }
})
```

我们需要将这个页面需要的state以及修改state的handler注入给它；这个state现在由store管理了，而这个handler也应该对应变成dispatch(action)！  

那么，我们怎么才能关联到store呢？这就要依靠上面提到的 **connect**了。  

connect需要知道你这个组件需要获取哪些state，以及你需要dispatch哪些action，我们通过下面的方式来表达：  

```javascript
import React from 'react'

import Card from '../../components/Card'
import Dialog from '../../components/Dialog'

const Index = React.createClass({
  render () {
    return <div className="g-index">
      <Card />
      <Dialog />
    </div>
  }
})

function mapStateToProps(state) {
  return state
}

function mapDispatchToProps(dispatch) {
  return {
    changeName () {
      dispatch({
        type: 'CHANGE_NAME',
        name: '葬爱'
      })
    },
    showDialog () {
      dispatch({
        type: 'SHOW_DIALOG'
      })
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Index)
```

mapStateToProps中传入的state就是整个应用的state，mapDispatchToProps中传入的dispatch就是store的dispatch！  

通过这样的定义和映射，我们的Index最终获得了所需要的数据以及方法，这些都是在Index中直接通过props可以访问到的！  

由于Index是页面，所以我们在mapStateToProps中直接return了整个state，代表整个state都可以通过props访问到：  

```javascript
const Index = React.createClass({
  render () {
    return <div className="g-index">
      <Card />
      <Dialog />
      <button onClick={this.props.changeName}>change name</button>
      <button onClick={this.props.showDialog}>show dialog</button>
    </div>
  }
})
```

我们再来看看两个子组件：  

```javascript
// Card.jsx
import React from 'react'
import { connect } from 'react-redux'

const Card = (props) => {
  return <div className="m-card">
    <div>
      姓名：{props.name}
    </div>
    <div>
      照片：{props.picture}
    </div>
  </div>
}

function mapStateToProps(state) {
  var info = state.card

  return {
    name: info.name,
    picture: info.picture
  }
}

export default connect(mapStateToProps)(Card)
```

```javascript
// Dialog.jsx
import React from 'react'
import { connect } from 'react-redux'

const Dialog = (props) => {
  if (!props.status) {
    return null
  }

  return <div className="m-dialog">
    <div>
      dialog
    </div>
    <button onClick={props.hideDialog}>close</button>
  </div>
}

function mapStateToProps(state) {
  var info = state.dialog

  return {
    status: info.status
  }
}

function mapDispatchToProps(dispatch) {
  return {
    hideDialog () {
      dispatch({
        type: 'CLOSE_DIALOG'
      })
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Dialog)
```

同样的，需要哪些state，就在mapStateToProps里return，需要哪些handler就在mapDispatchToProps里return，是不是很简单？  

这样我们就实现：在首页点击“change name”按钮，Card组件就会修改名字，点击“show dialog”按钮，弹窗（Dialog）就会出现；在弹窗里点击“close”按钮，弹窗就会关闭。  

这里可以发现，现在我们已经不在乎组件嵌套有多深了，因为不管在哪里，都能实现全局的通讯！  

之前我们遗留的问题终于得到解决了！  

开心得就地打滚起来。  

但是回头一想，你可能会问，虽然这样是实现了，但是感觉很麻烦，为什么不让每个组件直接接收store，然后在组件内直接通过store.getState()来获取需要的state，再直接通过store.dispatch(action)的方式来改变state？  

明明之前就是这么介绍的啊，难道你前面讲的都是在骗我？？？  

我想你可能是这样想的：  

```javascript
import React from 'react'

import Card from '../../components/Card'
import Dialog from '../../components/Dialog'

const Index = React.createClass({
  render () {
    var store = this.props.store
    var state = store.getState()

    return <div className="g-index">
      <div>
        {state.card.name}
      </div>
      <button onClick={this.changeName}>change name</button>
    </div>
  },
  changeName () {
    this.props.store.dispatch({type: 'CHANGE_NAME', name: '葬爱'})
  }
})

export default Index
```

这样好像简单了很多！  

事实上，上面这段代码是会报错的，因为store并不在props上；但是，我们确实可以直接获取到store，正确的获取方式是这样的：  

```javascript
import React from 'react'

import Card from '../../components/Card'
import Dialog from '../../components/Dialog'

const Index = React.createClass({
  render () {
    var store = this.context.store
    var state = store.getState()

    return <div className="g-index">
      <div>
        {state.card.name}
      </div>
      <button onClick={this.changeName}>change name</button>
    </div>
  },
  changeName () {
    this.context.store.dispatch({type: 'CHANGE_NAME', name: '葬爱'})
  }
})

Index.contextTypes = {
  store: React.PropTypes.object
}

export default Index
```

我们说过，`Provider`是用来实现`store`的全局访问的，它的原理就是使用react的[context](https://facebook.github.io/react/docs/context.html)，context是可以实现跨组件之间传递的，而且不需要像props一样显式地表达，是完全透明的存在！  

现在你可以看到上面的例子是可以正常运行的，没有报错。  

但是，当你点击“change name”按钮的时候，页面上的`state.card.name`并没有相对应改变！  

纳尼？？？  

我们知道，在react中，state和props的改变会促使react重新渲染（rerender）当前组件；但是，使用context传递的属性却不会！也就是说，即便你的store发生了变化，当前组件也不会知道，所以不会重新渲染页面！  

要实现及时的渲染，我们需要订阅store的变化！store提供了一个底层api：`subscribe`。  

但是，使用subscribe的方式，会改变我们整个程序的工作模式，从原本的数据主动流动，变成了订阅/发布！我们需要去主动地监听store的变化，订阅每一个在组件里使用到的state的变化，然后再更新组件，这让我们的工作变得复杂，并且容易出问题！  

一个最简单的例子，假设我在subscribe的listener里执行了dispatch，猜猜会是什么情况？  

没错，死循环。  

因此，为了避免这样的事情发生，简化我们的工作，redux（准确来说是react-redux）提供了connect的方式，来避免变成被动的订阅/发布模式，维持react原本的自动更新方式。  

所以还是老老实实用connect吧，人家官方文档也说了：  

> 这个方法做了性能优化来避免很多不必要的重复渲染。（这样你就不必为了性能而手动实现 React 性能优化建议 中的 shouldComponentUpdate 方法。）

不管你信不信，反正我不信，实质是浅比较，对于比较复杂的对象，依然无能为力。  

好吧，这些都是题外话，我们接着讲。  


## middleware  

引用官方文档的话：  

> middleware提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。

也就是说，一个action刚发起，到生效之前，这一个过程中，你可以相对应做一些内容扩展，比如你希望在某一次action前后打印一些内容，那么，你可以这么做：  

```javascript
function mapDispatchToProps(dispatch) {
  return {
    changeName () {
      console.log('即将改名……')
      dispatch({
        type: 'CHANGE_NAME',
        name: '葬爱'
      })
      console.log('完成改名！')
    }
  }
}
```

这确实可以实现，但是很low，而且，如果你想对所有action都生效，那你绝对不会这么去做，因为一个个都这么写你会写到吐血。使用 **middleware** 可以轻松搞定！  

官方推荐了一个用于实现相同功能的middleware——[redux-logger](#)，我们可以直接使用：  

```javascript
// App.jsx
import createLogger from 'redux-logger'
const store = createStore(reducer, initState, applyMiddleware(createLogger()))
```

（以上省略若干代码。。。）  

就是这么简单！  

其实middleware是一个完全可选的部分，因为一般情况下，我们并不需要对一个action的过程做什么处理，我们的工作主要是编写action以及相对于的reducer，这就够了。  

那为什么还要特地介绍它？因为接下来我们将会涉及到异步action的相关实现。就目前来看，我们所编写的action都是同步的，但是web开发中，我们常常会需要用到异步操作，比如从服务端拉取数据，那这时候的action应该怎么写？  

你可能已经稍微有点思路了，根据现有的方式，其实实现起来也不难。但是过程会比较麻烦，你可以试试看；使用middleware的话可以大大简化这部分工作，不过这都是后话了，我们下回再介绍。  

因为不知不觉又超字数了。。。  


## 总结  

好啦，基本的概念先讲到这里，通过上面的内容，我们已经可以在react中集成redux，并且尝试着编写一些简单的逻辑，使得整个应用运作起来！  

而且我们还解决了之前颇为诟病的嵌套组件之间的通讯问题！  

那么，到了这一步之后，我们就可以真正投入业务当中使用了吗？  

答案当然是——no！  

尼玛。。。  

学了这么多你跟我说还不能用，我要掀桌子了！  

事实上，我们还有一些东西没有规范好，比如action的type，如果应用做大起来，type的数量会非常庞大，而每个type又都必须是唯一的，如何解决这个命名冲突问题？当state变得庞大复杂的时候，如何高效地更新需要变动的部分？如何编写异步action？  

这些问题，我们会在之后的内容当中一一讲解。  


参考文献：  
react官方文档：[https://facebook.github.io/react/](https://facebook.github.io/react/)  
redux中文官方文档：[http://cn.redux.js.org/](http://cn.redux.js.org/)