---
layout     : post
title      : "Redux 入門"
subtitle   : "簡單介紹如何使用 Redux 與 Redux 資料流"
date       : 2015-07-30 00:43:52
author     : "Rhadow"
tags       : Redux React
comments   : true
signature  : true
---

與 React 搭配的單向資料流架構 Flux 已經推出一段時間，想必有在使用 React 的開發者們都已相當熟悉。社群也陸陸續續開發出許多 Flux 相關的 library 像是：Reflux, Fluxxor, Alt 等等...。而最近有在關注 React 社群的人一定都有聽過 [Redux](https://github.com/gaearon/redux) 這個迅速竄紅的 library。但由於 Redux 在設計上與原生的 Flux 較為不同，因此希望能藉由這篇文章幫助各位了解 Redux 的基本運作方式。

## Redux 核心概念

* **只使用一個 store 將整個應用程式的狀態 (state) 用物件樹 (object tree) 的方式儲存起來。** 原生的 Flux 會有許多分散的 store 儲存各個不同的狀態，但在 redux 中，只會有一個 store 將所有的資料用物件的方式包起來。請看以下比喻：

```js
//原生 Flux 的 store
var firstStore = {
    first: 1
}
var secondStore = {
    second: 2
}

// Redux 的單一 store
var state = {
    firstState: {
        first: 1
    },
    secondState: {
        second: 2
    }
}
```

* **唯一可以改變這個 state 的方法就是發送 action，這個 action 其實就只是一個物件告訴 state 該怎麼改變而已。** 這部分保留了原生 flux 的特性，確保 state 不會被其他的動作給更動。

* **實際因應 action 裡的內容對 state 做變化的函式叫做 reducer。** Reducer 有兩個參數，一個是現有的舊 state，另一個就是 action，而 reducer 回傳的就是經過更新後的新 state。

## Actions

Action 在原生 Flux 和 redux 裡，都是一個告知 state 需要改變的物件。通常會長得像這樣：

```js
{
  type: ADD_TODO,
  payload: {
    text: 'Build my first Redux app'
  }
}
```

這部分並沒有太大差異。

## Action creators

以往在原生 Flux 的設計中，action creator 需要做 dispatch 的動作。
而在 redux 的 action creator 中，我們只需要簡單地將 action 物件回傳就好。

原生 Flux action creator:

```js
function addTodoWithDispatch(text) {
    dispatch({
        type: ADD_TODO,
        payload: {
            text
        }
    });
}
// 實際發送 action
addTodoWithDispatch(text)
```

Redux action creator:

```js
function addTodo(text) {
    return {
        type: ADD_TODO,
        payload: {
            text
        }
    };
}

// 實際發送 action
dispatch(addTodo(text));
dispatch(removeTodo(id));
```

在 redux 中，`dispatch()` 這個方法來自於 `store.dispatch()`，但就大部份的狀況來說，我們可以使用作者在 [react-redux](https://github.com/gaearon/react-redux) 內提供的 `Connector` 元件。此元件會將 `dispatch()` 方法抽出來提供我們使用，因此並不需要特別從 store 中提取。

而 redux 提供的另一個方法 `bindActionCreators()` 就是將我們提供的 actionCreator 外面再包上一層 `dispatch()`。一般來說，在實作上通常都會想偷懶不想寫像 `dispatch(removeTodo(id))` 的程式碼，
因此 `bindActionCreators()` 算是一個常用到的方法之一。

## Reducers

Reducer 類似於原生 Flux 的 Store。但由於 Redux 只有一個 Store，這些 reducers 的功能就是針對這個唯一的 Store 內的 State 的部分內容進行更新。Reducer 接收舊 state 與 action 並回傳一個新的 state:

```js
(previousState, action) => newState
```

我們來看一下實際範例吧：

```js
const initialState = { todos: [], idCounter: 0 };

function todos(state = initialState, action) {
    switch (action.type) {
        case ADD_TODO:
            return {
                ...state,
                todos: [
                    ...state.todos,
                    { text: action.payload, id: state.idCounter + 1 }
                ],
                idCounter: state.idCounter + 1
            };
        case REMOVE_TODO:
            return {
                ...state,
                todos: state.todos.filter(todo => todo.id !== action.payload)
            };
        default:
            return state;
    }
}
```

Reducer 會先查看 action 的 type 是什麼並做出相對應的動作。例如 `ADD_TODO`，會在 `todos` 最後加入一個新的 todo 物件並把 `action.payload` 的資料放到 `text` 中，同時更新 `idCounter`。特別注意的是，這邊回傳的是**完全新的物件而不是修改原本物件的內容**。

```js
return {
    ...state,
    todos: [
        ...state.todos,
        { text: action.payload, id: state.idCounter + 1 }
    ],
    idCounter: state.idCounter + 1
};
```

`...state` 是 ES7 的寫法，可以像 ES6 打散 array 一樣地把物件內容抽取出來。如果你是使用 babel 的話，請記得去調整 babel 的設定。

## Store

在 redux 內只有一個 store，這個 store 是基於我們所建立的許多 reducers 上。Redux 有提供 `createStore()` 的方法。

單一 reducer 時建立 store 的方式:

```js
import { createStore } from 'redux';
import todos from '../reducers/todos';
const store = createStore(todos);
```

但一般來說，我們的 app 都會包含多個 reducers，這時我們可以使用 `combineReducers()` 這個方法將多個 reducers 合併為一個 reducer 再丟給 `createStore()` 來做出 store。

```js
export function todos(state, action) {
  /* ... */
}

export function counter(state, action) {
  /* ... */
}
```

多個 reducers 時建立 store 的方式:

```js
import { createStore, combineReducers } from 'redux';
import * as reducers from '../reducers';
const reducer = combineReducers(reducers);
const store = createStore(reducer);
```

建立出來的 store 會是一個物件，內容屬性如下：

```js
{
    dispatch,
    subscribe,
    getState,
    getReducer,
    replaceReducer
};
```

其中的 `getState()` 會回傳整個應用的單一 state。如果以上述的兩個 reducers 來說，做出來的 state 會長的像是這樣：

```js
const state = store.getState();
console.log(state);

// 結果：
{
    todos: todoState,
    counter: counterState
}
```

## 與元件連結

目前官方的範例都是使用 container pattern。簡單來說，就是有一個只管接收 props 並 render 的笨元件，與包覆在笨元件外圍負責管理資料並將需要的資料傳給笨元件的 container 元件。而負責與 redux 交流的正是這個 container 元件。

[react-redux](https://github.com/gaearon/react-redux) 提供了 `Provider` 元件與 `connect` 方法。

`Provider` 是使用在應用程式的根元件內，負責將唯一的 store 傳下去給其他子元件。

```js
const reducer = combineReducers(reducers);
const store = createStore(reducer);

class App extends Component {
    render() {
        return (
            <Provider store={store}>
                {() => <App />}
            </Provider>
        );
    }
}
```

而 `connect` 會接收一個函示當參數並回傳一個 Component class。
`connect` 的功用是將 dispatch 方法透過 props 的方式加到元件中。
我們可以透過這個函式來選取這個 container 需要 state 的哪一部分。

例如在 counter container 中，我們可以只選取 state 裡的 counter 部分：

```js
function select(state) {
  return { counter: state.counter };
}

class CounterApp {
    render() {
        const { counter, dispatch } = this.props;
        return (
            <Counter counter={counter} />
        );
    }
}

export default connect(select)(CounterApp)
```

當然你也可以不用 `select`，那麼在 `Connector` 裡接收到的就是整個應用程式的完整 state 了。

## 總結
Redux 是一個保有 Flux 特性但在實作上卻又沒有那麼像原生 Flux 的一個 library。它減少了許多 boilerplater 並簡化了 Store 內部的程式碼。
如果有人對 redux 處理非同步有興趣的話可以參考 [redux-example](https://github.com/Rhadow/redux-example/tree/feature/redux%401.0)，裡面有用到一點在本文中未提到的 middleware。
最後，各位如果對文章內容有任何建議或指正，歡迎留言指教。






