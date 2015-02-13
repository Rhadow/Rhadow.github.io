---
layout     : post
title      : "To Flux or Reflux"
subtitle   : "分享 Flux 與 Reflux 架構上的差別"
date       : 2015-02-02 11:01:00
author     : "Rhadow"
tags       : React Flux
comments   : true
signature  : true
---

這篇文章主要分享 Flux 與 Reflux 架構上的差別。如果對 React 和 Flux 有基本了解的話，會較能理解本篇內容。主要參考來源為[這篇](http://blog.krawaller.se/posts/react-js-architecture-flux-vs-reflux/)國外分享以及自己實驗的一些心得。

## Flux 概念難以理解
最近由於在專案的開發上會用到 React，在學習的過程中必然會接觸到 Flux，這個與 React 息息相關的設計模式。但在初學 Flux 的過程中，相信有不少人在看完教學後對整個設計架構還是很迷茫。

React 對許多人來說最大的優點就是學習曲線相對來得低，只要搞懂 props 與 state 的差別就已經差不多可以開始拿來用了，算是相當好上手。回過頭來看看其他框架，例如 Angular，想完全駕馭它之前你必須先了解[這些](https://docs.angularjs.org/guide/concepts)。

Flux 很完美的把上述的 React 優點給毀了，想要搞懂 Flux 就必須像 Angular 一樣，投入一定程度的時間來理解與消化整個概念。

Flux 概念圖：
![Image of Flux](http://blog.krawaller.se/img/flux-diagram.png)

## Reflux 的出現
當有人覺得某種概念很複雜難以理解時，相對的就會有人以另一種方式來詮釋或簡化這個概念。Mikael Brassman 開發出的 [Reflux](https://github.com/spoike/refluxjs) 簡化了原本 Flux 繁瑣的程式碼，使整個資料傳輸的流程更加容易理解。 

為了理解 Reflux 是如何簡化整個資料流的傳輸，以下會分別使用上述兩種架構來製作同一個簡易人員名單（可新增，刪除人員）的應用程式來做比較。

## 依賴性比較
首先，使用 Reflux 意味著必須使用 Reflux 函式庫，需要透過 npm 安裝。
Flux 雖然是一種設計模式，但在實做上，還是會需要使用到 Facebook 提供的 [Dispatcher](https://github.com/facebook/flux/blob/master/src/Dispatcher.js)。 
實際使用 Reflux 時，只是把 `require('flux')` 換成 `require('reflux')`。

## Action 比較
在元件裡呼叫特定 Action 的程式碼在兩種架構裡是完全一樣的：

```javascript
handleDelete: function(e){
    e.preventDefault();
    appActions.deleteUser(this.props.id);
}
//其他省略
```
儘管兩種架構不完全相同，appActions 都是用來呼叫特定的 Action 來執行。以上程式碼就是用來呼叫刪除使用者這個 Action。

## 元件從 Store 取資料的比較
在取資料部分，兩種架構上僅有些微差異，我們先來看看 Flux：

```javascript
componentWillMount: function(){
    appStore.addChangeListener(this._onChange);
},
//其他省略
```

Flux 在 `componentDidMount` 時加入了偵查 Store 變化的 listener，接著再由回呼函式來將元件內部做更新的動作。
我們再來看看 Reflux：

```javascript
ExampleApp = React.createClass({
    mixins: [reflux.ListenerMixin],
    componentDidMount: function(){
        this.listenTo(appStore, this._onChange);
    },
    //其他省略
});
```

Reflux 透過 mixin 幫元件加入了 `listenTo()` 的方法，用來偵測任何 Store 的改變，同樣由一個回呼函式 (上述例子是 `this._onChange` )來處理元件內部的資料更新。

## Store 比較
有趣的部分來了，首先來看看 Flux 架構下的 Store：

```javascript
var appStore = _.extend({}, eventEmitter.prototype, {
    getUserList: function() {
        return _userList;
    },
    addUser: function(user) {
        _userList.push(user);
    },
    deleteUser: function(index) {
        _userList.splice(index, 1);
    },
    emitChange: function() {
        this.emit('change');
    },
    addChangeListener: function(callback) {
        this.on('change', callback);
    },
    removeChangeListener: function(callback) {
        this.removeListener('change', callback);
    },
});

appDispatcher.register(function(payload) {
    var action = payload.action;
    switch (action.actionType) {
        case constants.ADD_USER:
            appStore.addUser(action.data);
            break;
        case constants.DELETE_USER:
            appStore.deleteUser(action.data);
            break;
        default:
            return true;
    }
    appStore.emitChange();
    return true;
});
//其他省略
```

Flux 架構下的 Store 需要透過 Dispatcher 傳過來的 Action 類型字串來決定接下來該怎麼做，同時得提供外部元件用來偵測這個 Store 變化的方法 （ `addChangeListener()`，`emitChange()`， `removeChangeListener()`）。
接著看看 Reflux：

```javascript
var appStore = reflux.createStore({
	init: function(){
        this.listenTo(appActions.addUser, this.onAddUser);
        this.listenTo(appActions.deleteUser, this.onDeleteUser);
    },
	onAddUser: function(user){
		_userList.push(user);
		this.trigger(_userList);
	},
	onDeleteUser: function(index){
		_userList.splice(index, 1);
		this.trigger(_userList);
	},
	getUserList: function(){
		return _userList;
	}
});
//其他省略
```

Reflux 透過 `createStore()` 來設定 Store。在 Reflux 中，Store 跳過 Dispatcher 這層並直接對各個 Action 做偵測的動作。在 Store 內需要做的就是設定各種 Action 的回呼函式並使用 `trigger` 讓元件知道 Store 有變化了。聰明的各位可能會注意到，當我們的應用程式變複雜，Action 越變越多時，`init()` 的內容也會隨之擴增。Reflux 已經幫各位想到這點，我們可以改用以下的方式來縮短程式碼：

```javascript
var appStore = reflux.createStore({
	listenables: appActions,
	onAddUser: function(user){
		_userList.push(user);
		this.trigger(_userList);
	},
	onDeleteUser: function(index){
		_userList.splice(index, 1);
		this.trigger(_userList);
	},
	getUserList: function(){
		return _userList;
	}
});
//其他省略
```

透過 `listenables`，Reflux 會幫 appActions 內的所有 Action 自動註冊一個 ”on + 事件名稱“ 的回呼函式，我們需要做的就只是定義這些回呼函式而已。在 Store 部分，我們已經可以漸漸看到 Reflux 為我們縮減程式碼的好處囉。

## Action 比較
以下為 Flux 架構下的 Action 設定：

```javascript
var appActions = {
	addUser: function(user){
		appDispatcher.handleViewAction({
			actionType: constants.ADD_USER,
			data: user
		});
	},
	deleteUser: function(index){
		appDispatcher.handleViewAction({
			actionType: constants.DELETE_USER,
			data: index
		});
	}
};
//其他省略
```

Flux 的 Action 需要針對每個 Action 設定一個類型，以便 Store 在接收時做比對。在 Action 越來越多的狀況下，appActions 的冗長度可想而知。
接著我們來看看 Reflux 的 Action：

```javascript
var appActions = reflux.createActions([
	'addUser',
	'deleteUser'
]);
//其他省略
```

在 Reflux 底下，只需要透過 `createAction()` 這個方法並傳入一個字串的陣列就可輕鬆設定好 Action 囉。

## Dispatcher 比較
最後讓我們來看看 Dispatcher。
Flux 版的 Dispatcher：

```javascript
var Dispatcher = require('flux').Dispatcher;
var appDispatcher = new Dispatcher();

appDispatcher.handleViewAction = function(action) {
    this.dispatch({
    	actionType: 'VIEW_ACTION',
    	action: action
    });
};
//其他省略
```

這裏的 `handleViewAction` 是選擇性的。主要是為了區分 Action 的來源（有可能會有來自 Server 的 Action ）。
Reflux 版的 Dispatcher：

```javascript
//完全不需要
```

是的，你沒看錯。在 Reflux 架構中 Store 是直接偵測 Action，而不是透過 Dispatcher 將 Action 發佈給 Store。因此，Reflux 完全不需要 Dispatcher。 

## 總結
看完了上面的比較，相信大家應該可以了解到 Reflux 是如何縮短程式碼並將整個資料傳輸流程縮短。在 Reflux 中：元件負責偵測 Store 的變化並觸發各種 Action，Store 偵測到不同的 Action 並做出相對應的更新。
![Image of Reflux](http://blog.krawaller.se/img/reflux-flow.jpg)
雖然整體的程式碼減少了，但並不代表我們不需要去了解 Flux。Flux 與 Reflux 各有優缺，而程式設計師則需要根據需求而決定使用的設計模式。
以上程式碼可在[這裏](https://github.com/Rhadow/Flux-Reflux-Practice)查看。如果文章中有任何問題，也歡迎各位討論糾正。
