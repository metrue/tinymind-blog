---
title: 认识和使用Flux
date: 2016-01-09T09:20:02.322Z
---

Flux 不是一种框架也不是一个库，它是实现 React
单向数据流的一种新架构。在 Facebook 内部和 React 配合使用。Facebook
提供了一个 Dispatcher 库项目，dispatcher 是一种全局的 pub/sub
handler，而 handler 则会将 payloads 广播到注册的回调里。

一种典型的 Flux 框架会使用 NodoJS 的 EventEmitter
模块来构建事件系统来管理应用的状态.

通过了解各个单独的模块，我们可以更加了解 Flux 架构.

* Actions: 一些 Helper methods 来协助数据向 Dispatcher 的传递.
* Dispatcher: 接收 actions, 然后广播 payloads 到注册的回调。
* Stores: 包含那些关于应用状态和逻辑，注册到 dispatcher 的容器.
* Controller Views: React 组件，从 stores 中获取 state，然后通过
    props 传递到子组件.

![Flux模块示意图](https://raw.githubusercontent.com/metrue/minghe.me/master/images/flux/flux-modules.png)

那么我们的 API 和上图是什么关系呢，当我们需要和外部 (比如后端的数据
API)
相互作用的时候，我们该怎么办呢，[这篇文章](https://scotch.io/tutorials/getting-to-know-flux-the-react-js-architecture)
说在 Actions 中引入数据到整个 Flux FLow 是一种最佳方式.

## Dispatcher

Dispatcher 是整个应用的中心枢纽, 管理着整个过程。 它接收 actions
然后 dispatch actions 和数据到注册的回调中。

Dispatcher 将 payload
广播到所有注册的回调中，而且可以以特定的顺序去调用那些回调，而且可以在执行之前进行等待。你的应用中只有一个
dispatcher， 它扮演着应用的中心枢纽。

```ruby

var Dispathcer = require('flux').Dispatcher;
var AppDispatcher = new Dispatcher();

AppDispatcher.handleViewAction = function(action) {
  this.dispatcher({
    source: 'VIEW_ACTION',
    action: action
  });
}

module.exports = AppDispatcher;

```

在上面的例子中，我们就创建了一个dispatcher,
然后创建了一个handleViewAction 方法。
当你想区分是view触发的actios还是 server API
触发的actions的时候，这种抽象是很有用的。

我们的方法调用 dispatch 方法，它将action payload
广播到它注册的回调里。 这个方法可以在 Stores 中发生，并且在一个
state 更新中产生结果。

整体图如下：
![Dispatcher dispatch](https://raw.githubusercontent.com/metrue/minghe.me/master/images/flux/dispatcher.png)

Dispatcher 最cool的地方是它可以在我们的 Stores 里面定义依赖
(Dependencies).
所以如果我们的应用里有一部分依赖于其他部分先更新，为了正确的render，Dispatcher
的 waitFor 方法会特别有用。

为了利用这个特性，我们需要保存 Dispatcher 的注册方法
(registration)的返回值，所以我们可以在我们的 Store 中这么做。

```ruby
  ShoeStore.dispatcherIndex = AppDispatcher.register(payload) {

  };
```

在我们的 Store 中，当我们处理分发过来的 action, 我们可以使用
Dispatcher 的 waitFor 方法去保证  ShoeStore 被更新。

```ruby
case 'BUY_SHOES':
  AppDispatcher.waitFor([ShoeStore.dispatcherIndex], function() {
    CheckoutStore.purchaseStoes(ShoeStore.getSeletedShoes());
  });
```

## Stores

在 Flux 中， Stores 管理着我们应用的状态。也就是 Store
负责管理数据，数据的获取，以及回调函数的 disapather.

```ruby

var AppDispatcher = require('../dispatcher/AppDispatcher');
var ShoeConstants = reuquire('../constants/ShoeConstants');
var EventEmitter = require('events').EventEmitter;
var merge = require('react/lib/merge');

var _shoes = {};

function loadShoes(data) {
  _shoes = data.shoes;
}

var ShoeStore = merge(EventEmitter.prototype, {
  getShoes: function() {
    return _shoes;
  },

  emitChange: function() {
    this.emit('change');
  },

  addChangeListener: function(callback) {
    this.on('change', callback);
  },

  removeChangeListener: function() {
    this.removeListener('change', callback);
  }
});

AppDispatcher.register(function(payload) {
  var action = payload.action;
  var text;

  switch(action.actionType) {
    case ShoeConstants.LOAD_SHOES:
      loadShoes(action.data);
      break;
    default:
      return true;
  }

  ShoeStore.emitChange();

  return true;
});

module.exports = ShoeStore;
```

上面的代码我们使用了 NodeJS 的  EventEmitter,
它让我们可以监听和广播我们的事件. 然后我们的 Views/Components
可以基于这些事件去更新。 因为我们的View Controller
监听这些事件，所以发射这些改变的事件可以让我们的 View Controller
知道我们的应用 state
已经改变，可以去获取这些状态然后保持应用的更新。

与此同时，我们也屌用了 AppDispatcher 的 register
我们的回调，着意味着我们的 Store 现在监听了 AppDispatcher
的广播。而 switch
语句则判定所到来的广播是否需要响应的action去承担。如果有相应的action响应，一个事件就会被发射，那些监听这个事件的view就会更新它们的states.

![Store, Dispatch, Action, Controller View](https://raw.githubusercontent.com/metrue/minghe.me/master/images/flux/store.png)

而public方法 getShoes 则是获取所有的shoes放到 _shoes
对象中，然后在我们的组件中使用这些数据.
这只是一个简单的例子，复杂的逻辑我们可以放到这里，而不是
views，可以让我们的代码更佳整洁.

## Action Creator & Actions

Action Creators 是发送 actions 到 Dispatcher 的一组方法的集合,
Actions 是通过 dispatcher 传送的真正的 payloads;

 Facebook 这样使用 Action 的， 一个action包含action type, 和 action
 data, action type 是常量(constants), 定义了什么action即将发生，而action data
 则是action的数据。对于注册了的回调函数来说，action type
 决定了哪一个回调函数去处理，而action data
 则成为了回调函数的输入参数。

 ```ruby
 var keyMirror = require('react/lib/keyMirror');

 module.exports = keyMirror({
    LOAD_SHOES: null
 });
 ```

 我们使用 keyMirror 来定义我们的我们的Constants,
 看下面的例子你就知道 keyMirror 的基本功能了。

```ruby
var keyMirror = require('keymirror');
key = keyMirror({NAME: null});

key -> {NAME: 'NAME'}

```

我们来看看相应的 Action Creator 怎么定义吧:

```ruby
var AppDispatcher = require('../dispatcher/AppDispatcher');
var ShoeStoreConstants = require('../constants/ShoeStoreConstants');

var ShoeStoreActions = {
  loadShoes: function(data) {
    AppDispatcher.handleAction({
      actionType: ShoeStoreConstants.LOAD_SHOES,
      data: data
    });
  };
};

module.exports = ShoeStoreActions;
```

在上面的代码中，我们创建了一个loadStores的action，通过 dispatcher
广播这个 action， 这样 ShoeStore
里面就会听到这个事件，然后调用相应的方法去装载一些shoes。

## Controller Views

Controller Views 就是一些 React
组件，这些组件监听事件的改变，然后从Stores获取应用的状态，然后通过props传递数据到子组件中。

![Store Controller View](https://raw.githubusercontent.com/metrue/minghe.me/master/images/flux/controller-view.png)

```ruby
var React = require('react');
var ShoesStore = require('../stores/ShoeStore');

function getAppState() {
  return {
    shoes: ShoeStore.getShoes();
  };
}

var ShoeStoreApp = React.creteClass({
  getInitialState: function() {
    return getAppState();
  },

  componentDidMount: function() {
    ShoeStore.addChangeListener(this._onChange);
  },

  componentWillUnmount: function() {
    ShoesStore.removeChangeListener(this._onChange);
  },

  render: function() {
    <ShoeStore shoes={this.state.shoes}/>
  },

  _onChange: function() {
    this.setState(getAppState());
  }
});

module.exports = ShoeStoreApp;
```

在上面的例子中，我们使用 addChangeListener
去监听事件的改变，然后更新应用的状态当接收到事件的时候。

我们的 Stores 持有我们应用的 state, 我们使用 Stores 中的 Public
方法获取数据，然后set 应用的 state.

## 放在一起

我们已经对 Flux 每一个单独的模块进行了分析了解，我们现在对于 Flux
架构如何工作有了一个更好的认识了，通过下面这幅图，也许我们可以更加清晰了。

![Flux 架构工作流](https://raw.githubusercontent.com/metrue/minghe.me/master/images/flux/flux-work-flow.png)

至此，我相信你应该可以在你下一个 React 项目中使用 Flux 架构了。

## 参考文献
[Getting To Know Flux, the React.js Architecture](https://scotch.io/tutorials/getting-to-know-flux-the-react-js-architecture)
