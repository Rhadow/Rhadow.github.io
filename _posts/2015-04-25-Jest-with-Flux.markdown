---
layout     : post
title      : "Let's Flux with Jest"
subtitle   : "分享如何使用 Jest 解決 Flux 的單元測試"
date       : 2015-04-25 12:29:28
author     : "Rhadow"
tags       : Jest React
comments   : true
signature  : true
---

繼[上一篇](http://rhadow.github.io/2015/04/24/Jest-with-React/)分享了如何使用 [Jest](https://facebook.github.io/jest/) 對 React 元件做單元測試，本篇將把焦點專注在如何對 [Flux](https://facebook.github.io/flux/) 做單元測試。對於測試來說，我們必須將被測的模組獨立於其他模組，因此，Jest 的自動模擬化將測試整個 Flux 設計模式的流程變得更為容易。

## TL;DR

如果對你來說程式碼比文字更容易讀的話，本篇內所有的程式碼都放在[這裡](https://github.com/Rhadow/Flux-Reflux-Practice)，請直接參考。

## Flux 簡介

Flux 主要是由 Action, Dispatcher, Store 與 View 所組成。使用者透過與 View 的互動，發送不同的 Action，再由 Dispatcher 將這些 Action 分配給各個 Store 來變更內部資料，當 Store 資料改變後，再將 View 更新做一次循環。如下圖所示：

![Image of Flux](http://blog.krawaller.se/img/flux-diagram.png)

以單元測試的角度來看，我們已經在[上一篇](http://rhadow.github.io/2015/04/24/Jest-with-React/)解決了對 View (React 元件) 與 Action (是否正常發送) 的測試，Dispatcher 則是由官方提供。因此，我們真正需要寫單元測試的部分其實只有 Store 而已。

## 測試 Store

首先讓我們大致看一下即將被測試的 Store：

```js

'use strict';

var appDispatcher = require('../dispatcher/dispatcher.js');
var eventEmitter = require('events').EventEmitter;
var constants = require('../constants/constants.js');
var _ = require('underscore');

var _userList = [{
    name: 'Howard',
    age: 27
}, {
    name: 'Shaun',
    age: 22
}, {
    name: 'Amy',
    age: 26
}];

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

module.exports = appStore;

```

這是一個簡單的 Store 搭配『加入新使用者』與『刪除使用者』這兩個 Action，同時預設有三個使用者已經存在。

引用於 Facebook 官方的說法：

>By design, stores can't be modified from the outside. They have no setters. The only way new data can enter a store 
>is through the callback it registers with the dispatcher.

大致上的翻譯是：Store 的內部資料無法從外部直接修改，唯一將新資料加進 Store 的方法就是透過 dispatcher 註冊的回呼函式裡面的各個 Action 將資料帶入。以我們的例子來說就是以下這部分：

```js
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
```

由於需要模擬不同的 Action ，我們勢必需要在測試中將從以上回呼函式取出來，這樣才能丟進不同的 `payload` 來模擬不同的 Action。幸運的是，經過 Jest 模擬化 (mock) 過後的 `register()` 函式會有一個叫做 `mock` 的屬性。它會記錄著所有有關 `register()` 被呼叫的紀錄，包括次數，參數等等。於是，我們就可以使用以下程式碼將上述的回呼函式取出了。

```js
var mockRegister = appDispatcher.register;
//模擬化過後的 regeister()

var mockRegisterInfo = mockRegister.mock;
//找出記錄所有資訊的 mock 屬性

var callsToRegister = mockRegisterInfo.calls;
//找出呼叫紀錄

var firstCall = callsToRegister[0];
//取出第一次呼叫的紀錄

var firstArgument = firstCall[0];
//取出第一次呼叫的參數

var callback = firstArgument;
//需要的結果
```

如果上面的程式碼對你來說很陌生的話，建議可以閱讀 [Jest 官方對 Mock Function 的說明](https://facebook.github.io/jest/docs/mock-functions.html#content)。在我們繼續之前，讓我們先簡化一下程式碼：

```js
callback = appDispatcher.register.mock.calls[0][0];
```

有了這個回呼函式之後，我們就可以隨心所欲的模擬各種 Action 丟入 Store 來做測試了！

## 整合

接下來讓我們來看看完整的測試程式碼長得如何吧：

```js

'use strict';

jest.dontMock('../src/app/stores/appStore.js');
jest.dontMock('underscore');

describe('appStore', function() {
    var appStore,
        appDispatcher,
        callback,
        constants = require('../src/app/constants/constants.js'),
        addUserAction = {
            action: {
                actionType: constants.ADD_USER,
                data: {
                    name: 'Tester',
                    age: 27
                }
            }
        },
        deleteUserAction = {
            action: {
                actionType: constants.DELETE_USER,
                data: 'replace me in test'
            }
        };

    beforeEach(function() {
        appStore = require('../src/app/stores/appStore.js');
        appDispatcher = require('../src/app/dispatcher/dispatcher.js');
        callback = appDispatcher.register.mock.calls[0][0];
    });

    it('should register a callback to store', function() {
        expect(appDispatcher.register.mock.calls.length).toBe(1);
    });

    it('should initialize with a preset data', function() {
        expect(appStore.getUserList()).toEqual([{
            name: 'Howard',
            age: 27
        }, {
            name: 'Shaun',
            age: 22
        }, {
            name: 'Amy',
            age: 26
        }]);
    });

    it('should add a new user after addUserAction is fired', function() {
        callback(addUserAction);
        var allUsers = appStore.getUserList();
        expect(allUsers.length).toBe(4);
        expect(allUsers[allUsers.length - 1]).toEqual({
            name: 'Tester',
            age: 27
        });
    });

    it('should delete user according to index after deleteUserAction is fired', function() {
        deleteUserAction.action.data = 1;
        callback(deleteUserAction);
        var allUsers = appStore.getUserList();
        expect(allUsers.length).toBe(2);
        expect(allUsers).toEqual([{
            name: 'Howard',
            age: 27
        }, {
            name: 'Amy',
            age: 26
        }]);
    });
});

```

整體的測試程式碼看起來其實非常直接明白，這也是單元測試的一個要點，讀起來必須要像說明文件一樣。

這部分比較需要注意的地方有兩點：

第一是我們在最開始特別告訴 Jest 除了不要模擬 (`dontMock()`) 這次測試的 Store 外，同時也不要模擬 underscore。
原因是我們的 Store 是由 `_.extend` 方法所組成。透過 Jest 自動模擬過後的模組是不帶有任何邏輯的，因此，如果你是使用別的方法組 Store 的話，請記得要特別提醒 Jest。

第二則是我們在每次的小測試之前 (`it()`) 都會重新載入一遍 Store 以免被前一次小測試的結果影響到。

## 總結

其實使用 Jest 測試 Flux Store 會比測試 React 元件單純許多，我們不需要使用到 [React Test Utilities](https://facebook.github.io/react/docs/test-utils.html)，也不需要特別為 jsx 建立一個 `preprocessor.js`。但是，當應用程式的規模越變越大，Store 之間互相有依賴性時，可能就會需要使用到一些進階技巧，例如：手動建立模擬函式 (Manual mocks)。這些都有在 Jest 官網上做介紹。未來有機會也會發相關文章與大家討論。

