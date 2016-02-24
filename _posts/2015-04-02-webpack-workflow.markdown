---
layout     : post
title      : "使用 Webpack 建立 React 專案開發環境"
subtitle   : "介紹如何使用 webpack-dev-server 與 react-hot-loader"
date       : 2015-04-02 09:28:48
author     : "Rhadow"
tags       : Webpack React
comments   : true
signature  : true
---

Webpack 包含了許多好用的 loader 可以協助開發者在開發過程中省去不少麻煩。而 LiveReload 更是節省時間不可或缺的功能。本篇將透過一個簡單的範例介紹如何使用 webpack-dev-server 與 react-hot-loader 來讓開發 React 專案更為快速方便。如果你是剛開始使用 Webpack 的開發者，建議先讀完 [Webpack howto](https://github.com/petehunt/webpack-howto) 或 [如何使用 Webpack 模組整合工具](http://rhadow.github.io/2015/03/23/webpackIntro/) 熟悉 Webpack 後再來使用各式各樣的 loader。

**2016-02-24 更新： 由於版本問題造成大家無法成功使用文章中的程式碼感到抱歉，以下已將版本號補上。最新的開發環境建議參考 [react-transform-boilerplate](https://github.com/gaearon/react-transform-boilerplate)。**

## webpack-dev-server 與 react-hot-loader 是什麼

[webpack-dev-server](https://github.com/webpack/webpack-dev-server) 是個小型的 node.js express server，主要用來跑專案內的檔案，同時提供 LiveReload 的功能。[react-hot-loader](https://github.com/gaearon/react-hot-loader) 則是可以在不改變 React 元件的 state 下，將更改過程式碼的元件直接更新到畫面上。

## 準備

在專案目錄內先使用 `npm init` 產生 `package.json` 再透過 `npm install webpack@1.12.14 webpack-dev-server@1.14.1 react-hot-loader@1.3.0 babel-loader@5.4.0 react@0.13.3 --save-dev` 指令安裝 webpack，webpack-dev-server，react-hot-loader，react 和 babel-loader，並在專案目錄下加入檔案與資料夾並整理如以下結構:

  * /app
    * main.js
    * TestOne.js
    * TestTwo.js
  * /build
    * bundle.js (會透過 webpack 自動生成)
    * index.html
  * package.json
  * webpack.config.js

順帶一提，babel-loader 是用來解譯 jsx 與 ES6 語法的 loader。

接著，將以下程式碼貼到對應的檔案內容內:

main.js

```js
/* main.js */

'use strict';

var React = require('react');
var TestOne = require('./TestOne.js');
var TestTwo = require('./TestTwo.js');

var Main = React.createClass({
    getInitialState: function() {
        return {
          switch: true
        };
    },
    _toggle() {
        this.setState({
            switch: !this.state.switch
        });
    },
    render() {
        return (
            <div>
                <input type="button" onClick={this._toggle} value="Press Me!"/>
                {this.state.switch ? <TestOne /> : <TestTwo />}
            </div>      
        );
    }
});

React.render(<Main />, document.body);
```

TestOne.js

```js
/* TestOne.js */

'use strict';

var React = require('react');

var TestOne = React.createClass({
    render() {
        return (
            /* jshint ignore: start*/
            <div>Hello I am TestOne Component</div>
            /* jshint ignore: end*/
        );
    }
});

module.exports = TestOne;
```

TestTwo.js

```js
/* TestTwo.js */

'use strict';

var React = require('react');

var TestTwo = React.createClass({
    render() {
        return (
            /* jshint ignore: start*/
            <h1>Hello I am TestTwo Component</h1>
            /* jshint ignore: end*/
        );
    }
});

module.exports = TestTwo;
```

index.html

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
</head>

<body>
    <script src="bundle.js"></script>
</body>

</html>
```

webpack.config.js

```js
var path = require('path');

var config = {
    entry: [path.resolve(__dirname, 'app/main.js')],
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js'
    },
    module: {
        loaders: [{
            test: /\.js$/,
            loaders: ['babel']
        }]
    }
};

module.exports = config;

```

最後在 `package.json` 內做修改:

```js
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack" //新加入這行
},
```
當執行 `npm run build` 後，系統會執行 `webpack` 的指令並打包，開啟 index.html 後就能看到程式正確執行。


## 使用 webpack-dev-server 實現 LiveReload

現階段每當我們更改程式碼並想要看到結果時，就必須要輸入 `npm run build` 一次，還要到瀏覽器點重新整理才能看到變動後的成果。還好，webpack-dev-server 為我們省去了這個麻煩。

首先，先到 `package.json` 內加入一個新 `dev` 的指令:

```js
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "dev": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
}
```
`dev`內的指令解釋如下:

  * `webpack-dev-server` 會在 localhost:8080 建立起專案的 server
  * `--devtool eval` 會顯示出發生錯誤的行數與檔案名稱
  * `--progress` 會顯示出打包的過程
  * `--colors` 會幫 webpack 顯示的訊息加入顏色
  * `--content-based build` 指向專案最終輸出的資料夾

接著更新 `index.html` :

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
</head>

<body>
    <script src="http://localhost:8080/webpack-dev-server.js"></script>
    <script src="bundle.js"></script>
</body>

</html>
```

再來到 `webpack.config.js` 的 `entry` 屬性內加入 `'webpack/hot/dev-server'`如下:

```js
var path = require('path');

var config = {
    entry: ['webpack/hot/dev-server', path.resolve(__dirname, 'app/main.js')],
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js'
    },
    module: {
        loaders: [{
            test: /\.js$/,
            loaders: ['babel']
        }]
    }
};

module.exports = config;
```

這樣就完成 LiveReload 的功能囉，各位可以在執行 `npm run dev` 後到 [http://localhost:8080/](http://localhost:8080/) 檢查是否有正確跑起來，也可以試著在程式碼內做些改變，並觀察畫面是否有跟著變動。

## 加入 react-hot-loader

如果你有跟著以上每一步做的話，程式跑起來後應該會有一個按鈕和 "Hello, I'm TestOne Component" 的字出現在畫面上。
重複點擊按鈕，"Hello, I'm TestOne Component" 和 "Hello, I'm TestTwo Component" 會輪流出現在畫面上。

現在請將畫面切到顯示 "Hello, I'm TestTwo Component" ，並到 `TestTwo.js` 將 "Hello, I'm TestTwo Component" 改為 "Hello" 後儲存。完成後，會發現瀏覽器畫面會回到最初的 "Hello, I'm TestOne Component"，這是因為 LiveReload 會重新整理畫面並將 React 元件的 state 重設回到初始值。

react-hot-loader 的功用就是在不清除 state 的狀態下更新畫面，讓我們趕快來看看該如何達成這個效果吧。

首先，修改 `webpack.config.js` 內的 `entry` 屬性如下:

```js
entry: [
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    path.resolve(__dirname, 'app/main.js')
]
```

再來修改 `module` 屬性如下:

```js
module: {
    loaders: [{
        test: /\.js$/,
        loaders: ['react-hot', 'babel'],
        include: path.join(__dirname, 'app')
    }]
},
```

這部分要特別注意的地方是一定要加入 `include: path.join(__dirname, 'app')`，不然 webpack 會把 `node_modules` 內的 js 檔都透過 react-hot-loader 跑一遍，會因此導致 Cannot read property 'NODE_ENV' of undefined 錯誤造成程式無法正常運作。

最後一步，在 `webpack.config.js` 內加入以下屬性，別忘了要先宣告 `var webpack = require('webpack');`:

```js
plugins: [
    //new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
]
```

這邊的 `webpack.NoErrorsPlugin()` 是選擇性的，主要的功能是當更改完的程式碼有語法錯誤時不要重新整理。
當錯誤修好後，畫面會自動重新整理。

`new webpack.HotModuleReplacementPlugin()` 則是當不使用 webpack-dev-server 的 inline-mode 時才需要加入。本例是使用 inline-mode，你可以試著把註解掉的部分加回程式裡，當元件在做更新時，會出現 `Uncaught RangeError: Maximum call stack size exceeded` 的錯誤訊息。inline-mode 詳情請參考 [webpack-dev-server 說明](http://webpack.github.io/docs/webpack-dev-server.html)。

現在執行 `npm run dev` 後再一次做本段落最開始的實驗，就會發現更改程式碼後，畫面不會再回到首頁了。

## 總結

最近使用 webpack 的開發者越來越多，如果想深入了解的話不妨多搜尋一下網路上的教學，會發現它能夠完成的事情其實很多。如果在實作以上範例有碰到問題或有更好的建議的話，歡迎各位留言討論。
