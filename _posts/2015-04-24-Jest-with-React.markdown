---
layout     : post
title      : "Let's Jest some React Components"
subtitle   : "分享如何使用 Facebook 的單元測試框架 Jest 來測試 React 元件"
date       : 2015-04-24 12:30:37
author     : "Rhadow"
tags       : Jest React
comments   : true
signature  : true
---

單元測試在軟體開發過程中是一個很重要但常被忽略的環節。單元測試有著許多優點，例如，我們可以大幅度更動原本的程式碼並且不用擔心有哪邊被改壞了，因為只要單元測試都通過了，就代表著新更動過的程式是沒問題的。因此，如何寫出好的單元測試也是一門值得研究的課題。本篇將用一個簡單的例子分享如何使用 Facebook 的單元測試框架 [Jest](https://facebook.github.io/jest/) 來測試 React 元件。

## Jest 是什麼

Jest 是 Facebook 開發用來在 React 專案上做單元測試的框架。雖然說是 Facebook 開發的，但其實 Jest 是基於原本就很有名的 [Jasmine](http://jasmine.github.io/) 測試框架所衍生出來的。因此，在 Jest 中其實可以看到許多 Jasmine 內的方法 (像是 `toBe()` `toEqual()` 等等)。你可能會問: 有這麼多的測試框架可以選擇，為什麼要選 Jest 呢? Jest 最大的特點在於它會在測試時自動模擬 (mock) 透過 `require()` 進來的模組，讓你專注在目前被測試的模組中。

模擬化是指只做出該模組的空殼，裡面的實際邏輯都不存在。這在測試時特別有用，因為我們不希望其他的模組影響到我們目前測試中的模組。詳細請參考 [CommonJS Testing](https://facebook.github.io/jest/docs/common-js-testing.html#content) 和 [Mocks Aren't Stubs](http://martinfowler.com/articles/mocksArentStubs.html)。

## 準備

在測試開始前需要先做一些安裝與準備。第一步當然是透過 `npm install jest-cli` 來安裝 Jest。如果是使用 Windows 的開發者，你可能會碰到安裝上的問題，建議參考[國外提供的解法](http://ryanlanciaux.github.io/blog/2014/08/02/using-jest-for-testing-react-components-on-windows/)，這裡就不多做說明。

接下來請在專案的根目錄建立一個 `__tests__` 的資料夾用來放我們寫的單元測試檔案，Jest 在執行時會自動搜尋這個資料夾裡的檔案並一一執行測試。

接著，一樣在根目錄新創一個叫 `preprocessor.js` 的檔案，內容為:

```js
// preprocessor.js
var ReactTools = require('react-tools');
module.exports = {
    process: function(src) {
        return ReactTools.transform(src);
    }
};
```

這個檔案的主要功能是在測試時用來把 jsx 轉換成一般的 javascript。

最後一步則是到 `package.json` 檔內做設定:

```js
// package.json
{
    ...

    "scripts": {
        "test": "jest"
    }
    "dependencies": {
        "react": "*",
        "react-tools": "*"
    },
    "jest": {
        "scriptPreprocessor": "<rootDir>/preprocessor.js",
        "unmockedModulePathPatterns": ["react"]
    }

    ...
}
```

React 在最初設計時就被設計為不需要被模擬也能執行測試，因此我們使用 `unmockedModulePathPatterns` 來防止 React 在測試時被模擬化。
以上設定完成後我們就可以使用 `npm test` 的指令開始測試。

## 被測試的元件

以下是一個簡單的 React 元件，內容有兩個輸入框，一個是用來填名字，另一個用來填年齡。它還有一個確定按鈕，透過 Action 將使用者輸入的內容發送出去。在發送 Action 之前，會先檢查輸入框狀態，如果任何一個有留空的話則不會發送 Action。

```js
'use strict';

var React = require('react');
var appActions = require('../actions/appActions.js');

var Input = React.createClass({

    handleSubmit: function(e){
        e.preventDefault();
        var newUser = {
            name: this.refs.name.getDOMNode().value,
            age: +this.refs.age.getDOMNode().value
        };
        if(newUser.name && newUser.age){
            this.refs.name.getDOMNode().value = '';
            this.refs.age.getDOMNode().value = '';
            appActions.addUser(newUser);
        }
    },

    render: function() {
        return (
            /* jshint ignore:start */
            <div>
                <input type="text" placeholder="Please enter your name" ref="name" />
                <input type="text" placeholder="Please enter your age" ref="age" />
                <input type="button" value="Add" onClick={this.handleSubmit} />
            </div>
            /* jshint ignore:end */
        );
    }
});

module.exports = Input;
```

## 單元測試檔案

先來看看程式碼:


```js
//__tests__/Input-test.js

'use strict';

jest.dontMock('../src/app/components/Input.js');

describe('Input Component', function() {
    var Input = require('../src/app/components/Input.js'),
        React = require('react/addons'),
        appActions = require('../src/app/actions/appActions.js'),
        TestUtils = React.addons.TestUtils,
        tree,
        inputElements,
        nameElement,
        ageElement,
        submitElement;

    tree = TestUtils.renderIntoDocument(
        /* jshint ignore:start */
        <Input />
        /* jshint ignore:end */
    );

    inputElements = TestUtils.scryRenderedDOMComponentsWithTag(tree, 'input');
    nameElement = inputElements[0];
    ageElement = inputElements[1];
    submitElement = inputElements[2];

    beforeEach(function() {
        nameElement.getDOMNode().value = '';
        ageElement.getDOMNode().value = '';
        appActions.addUser.mockClear();
    });

    it('should keep user\'s input if the input format is wrong', function() {
        nameElement.getDOMNode().value = 'Tester';
        TestUtils.Simulate.click(submitElement);
        expect(nameElement.getDOMNode().value).toEqual('Tester');
        expect(ageElement.getDOMNode().value).toBeFalsy();
    });

    it('should clear user\'s input if the input formats are correct', function() {
        nameElement.getDOMNode().value = 'Tester';
        ageElement.getDOMNode().value = 27;
        TestUtils.Simulate.click(submitElement);
        expect(nameElement.getDOMNode().value).toEqual('');
        expect(ageElement.getDOMNode().value).toEqual('');
    });

    it('should send add user action with correct data when the inputs format are correct', function() {
        nameElement.getDOMNode().value = 'Tester';
        ageElement.getDOMNode().value = 27;
        TestUtils.Simulate.click(submitElement);
        expect(appActions.addUser).toBeCalledWith({
            name: 'Tester',
            age: 27
        });
    });
    
    it('should not send add user action when the inputs format are wrong', function() {
        nameElement.getDOMNode().value = 'Tester';
        ageElement.getDOMNode().value = 'wrong input';
        TestUtils.Simulate.click(submitElement);
        expect(appActions.addUser).not.toBeCalled();
    });
});
```

第一行非常重要: `jest.dontMock('../src/app/components/Input.js');` 

Jest 預設在 `require()` 時會自動模擬 (mock) 化進來的模組，因此這行我們告訴 Jest 當 `require('../src/app/components/Input.js')` 時，不要模擬 (don't mock) 化此模組，我們需要正常版的 Input 元件來做測試。

程式碼內出現的 `describe()`，`it()`，`expect()` 是 jasmine 內原來就有的方法。主要是讓我們的測試程式讀起來就像是一般英文的句子。

```js
describe('sum', function() {
    it('adds 1 + 2 to equal 3', function() {
       var sum = require('../sum');
       expect(sum(1, 2)).toBe(3);
    });
});
```

我們可以把以上程式碼翻成這樣的中文:

>這段程式碼將描述 (describe) "sum" 這個方法，它 (it) 將 "一和二相加並輸出三"。
>預計(expect) 執行 sum(1,2) 後得到的結果會是 3。

`beforeEach()` 也是一個 Jasmine 原有的方法。
使用方式是丟一個函式進去，作用是之後每次做小測試 (`it(...)`) 之前都會執行一次我們丟進去的函式。
在範例中，我們拿來清空輸入框與清空模擬化過的 action 紀錄。

React 有提供一系列的[測試工具](https://facebook.github.io/react/docs/test-utils.html)，我們會用它們來模擬一些事件，像是 click, change, keydown 之類的。

由於在測試時，元件並沒有實際 render 到 DOM 上，我們必須透過 `renderIntoDocument` 來將元件 render 到一個虛擬的 DOM 上。

`scryRenderedDOMComponentsWithTag` 則是讓我們能夠到上述的虛擬 DOM 中找出特定 HTML 元素並存在一個陣列中。在這個例子，我們找出了 Input 元件中所有的 input 元素。

在每個小測試 (`it(...)`) 中，我們也使用了 `Simulate` 這個方法來對送出按鈕模擬點擊事件。

建議各位可以邊寫邊看 Jasmine 與 React TestUtils 的各種方法，這樣在寫測試時能夠寫出更靈活更有彈性的測試。

## 總結

雖然使用 Jest 對 React 做測試非常方便，但它也沒有像官網所說的"無痛做測試"那樣的無痛。
除了在 windows 上安裝困難重重外，Jest 目前只支援到 NodeJS 0.10 版。最新的 0.12 版與 io.js 都是會報錯的。除了一些缺點之外，Jest 的確為我們省去了不少寫測試時的麻煩。同時，Facebook 也是 Jest 與 React 的背後推手，相信 Jest 對 React 的相容性應該不會輸給其他的測試框架。