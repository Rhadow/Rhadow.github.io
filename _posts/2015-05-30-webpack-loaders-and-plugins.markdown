---
layout     : post
title      : "深入了解 Webpack Plugins"
subtitle   : "分析常用的 Plugins 的功能與使用時機"
date       : 2015-05-30 11:39:34
author     : "Rhadow"
tags       : Webpack React
comments   : true
signature  : true
---

[Webpack](https://github.com/webpack/webpack) 是一個非常好用的打包工具，如果只是想單純地使用基本功能的話，它非常的好理解。
相信有看過教學的人都對以下這段基本設定相當熟悉：

```js
// webpack.config.js

module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

但現實總是與理想不同，一旦你開始想使用一些進階功能時，就會開始遇到各式各樣的 Plugins。由於在網路上除了官網的介紹之外，很少找到有把 Plugins 統整起來並介紹的文章，因此，本篇將針對常用到的 Plugins 做簡易的介紹。

## Hot Module Replacement Plugin

一般來說，Hot Module Replacement Plugin 通常會搭配 [Webpack dev server](http://webpack.github.io/docs/webpack-dev-server.html) 一起使用。 它提供類似於 Live Reload 的功能，一旦你的程式碼有做更動，此 Plugin 會直接更新畫面並做出相對應的變化。

通常會使用到 Hot Module Replacement Plugin 的時機是當 webpack dev server 不是透過 inline mode 做啟動時。

webpack-dev-server 有兩種啟動模式：

第一種是教學中常見到的：

```sh
$ webpack-dev-server --devtool eval --progress --colors --hot --content-base build
```

這就是所謂的 inline mode，指令中的 `--hot` 會自動將 Hot Module Replacement Plugin 的功能加入，因此，我們不用特地到 `webpack.config.js` 內做設定。

另一種啟動 webpack dev server 的方式則是透過程式：

```js
var webpack = require('webpack'),
    WebpackDevServer = require('webpack-dev-server'),
    webpackConfig = require('../webpack.config.js');

module.exports = function() {
    var compiler = webpack(webpackConfig);

    var bundler = new WebpackDevServer(compiler, {
        publicPath: '/build/',
        hot: true,
        quiet: false,
        noInfo: true,
        stats: {
            colors: true
        }
    });

    bundler.listen(8080, 'localhost', function() {
        console.log('Webpack-dev-server is live on port 8080');
    });
};
```

當使用這種方式啟動 webpack dev server 時，我們就必須記得到 `webpack.config.js` 內加入 Hot Module Replacement Plugin 的設定，如下：

```js
//webpack.config.js

//...other webpack settings

plugins: [
    new Webpack.HotModuleReplacementPlugin()
]
```

## Commons Chunk Plugin

Commons Chunk Plugin 的用法複雜且多樣，本文只針對基本用法做介紹。

此 Plugin 其中之一的用途是在於分離第三方套件與專案內部程式碼。由於專案內部程式碼會不斷做更新，而第三方套件一般來說修改的頻率較低，如果沒有與第三方套件分開打包的話，使用者在每一次更新後都必須下載第三方套件加上專案內部程式碼的一整包檔案，使專案運行起來的速度變慢，並降低使用者體驗。

此 Plugin 使用方式如下：

```js
var path = require('path');
var webpack = require('webpack');
var node_modules_dir = path.resolve(__dirname, 'node_modules');

var config = {
  entry: {
    app: path.resolve(__dirname, 'app/main.js'),
    vendors: ['react', 'jquery', 'bootstrap']
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'app.js'
  },
  module: {
    loaders: [{
      test: /\.js$/,
      exclude: [node_modules_dir],
      loader: 'babel'
    }]
  },
  plugins: [
    // 第一個參數對應到 entry 內的屬性名
    // 第二個參數是輸出檔案的名稱
    new webpack.optimize.CommonsChunkPlugin('vendors', 'vendors.js')
  ]
};

module.exports = config;
```

以上設定會打包成 `app.js` 與 `vendors.js` 兩個檔案。

## Provide Plugin

你有遇到將 jquery 和 bootstrap 打包到 vendors 卻出現 `Uncaught ReferenceError: jQuery is not defined` 的問題嗎？ Provide Plugin 會是解決這個問題的一種辦法。

我們先來看看解法再來探討它的用處：

```js
//webpack.config.js

//...other webpack settings

plugins: [
    new Webpack.ProvidePlugin({
        $: 'jquery',
        jQuery: 'jquery',
        'window.jQuery': 'jquery',
        'root.jQuery': 'jquery'
    }),
]
```

Provide Plugin 的主要功能是當在程式中遇到特定字元且沒被定義時會自動載入特定模組。上例的設定是在碰到 `$`，`jQuery`，`window.jQuery` 或 `root.jQuery` 時都載入 jquery 套件。所以，當在某檔案中不定義 `$` 直接使用 `$("#item")` 時，程式不會報錯。

而本段落初所碰到的問題正是在 bootstrap 中有使用 jquery 卻沒定義的情況，因此只要做了以上設定，bootstrap 就能正常引用 jquery 的功能了。

## No Errors Plugin

此 Plugin 主要功能是在程式碼出錯時，不會將有錯誤的程式碼打包以確保打包完後的程式碼語法上的正確性。比較容易會碰到的問題是，當有使用 [eslint-loader](https://github.com/MoOx/eslint-loader) 來為 ES6 做語法檢查時，不管是 warning 或是 error 都會使 No Error Plugin 停止打包。

同時使用此 Plugin 與 eslint-loader 的設定有好有壞，好處是你的程式碼必須乾淨到連 warning 都沒有才能正常打包。壞處則是檢查太嚴格，有些開發者可能不習慣這樣的模式。當你想要保有檢查 es6 語法功能卻又不想被這麼嚴苛的對待時，就把 No Errors Plugin 拿掉吧。

## Extract Text Plugin

大家都知道在 webpack 中 CSS 是可以被 `require()` 的，webpack 會自動生成一個 `<style>` 標籤並加入到 html 的 `<head>` 中。但是在發佈時，我們可能只希望有一個被打包過後的 css 檔。這時 Extract Text Plugin 就能幫助我們完成這項任務。

基本使用的方式很簡單，只需要給 Plugin 一個最後輸出完的檔名即可：

```js
//webpack.config.js

//...other webpack settings

plugins: [
    new ExtractTextPlugin('app.bundle.css')
]
```

以上設定會輸出一個 `app.bundle.css` 的檔案。

## UglifyJsPlugin

UglifyJsPlugin 的功能正如其名，它會壓縮 javascript 檔。在 `webpack -p` 也就是 production mode 時會自動執行。這邊特別提出來的原因在於有時會在壓縮時有一些多餘的警告 `(Side effects in initialization of unused variable define)` 跑出來，如果你也像我一樣龜毛不喜歡看到警告的話，你可以在 `webpack.config.js` 內做以下設定：

```js
//webpack.config.js

//...other webpack settings

plugins: [
    new Webpack.optimize.UglifyJsPlugin({
        compress: {
            warnings: false
        }
    })
]
```

上述程式碼只是把壓縮時的警告關上而已。

## HtmlWebpackPlugin

最後與各位分享的是一個非官方的 Plugin。[html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) 能夠動態產生 `index.html` 並支援 Extract Text Plugin 自動將打包完後的 js 與 css 檔加入。

基本使用方法如下：

```js
//webpack.config.js

//...other webpack settings

plugins: [
    new HtmlWebpackPlugin()
]
```

以上設定會自動生成一個基本的 `index.html` 並將打包過後的 js 和 css 檔透過 `<script>` 和 `<link>` 加入。
它也提供開發者自訂 template 方法，使用的引擎是 [blueimp](https://github.com/blueimp/JavaScript-Templates)，詳細設定請參考它的官方 Github。

那麼在什麼時候我們會需要動態組 `index.html` 呢？當你想要在 js 或 css 檔後面加上版本號時 (例如: app.bundle.0.0.1.js)，自動組成的 HTML 檔就會派上用場了，你不必在每次更新都要去改動 `index.html` 內的檔名。

## 總結

呼～寫了這麼長一篇的 Webpack Plugins 介紹，希望花時間看完的你有一些收穫。但這些也都只是 Webpack 冰山一角而已，除了 plugins 之外，還有各式各樣的 loaders 等著我們去使用。前端工程師真的是無時無刻都要學習啊。如果對以上介紹有問題或指正的地方，歡迎留言指教。需要看完整程式碼的讀者可以到 [React-Webpack-Boilerplate](https://github.com/Rhadow/react-webpack-boilerplate) 參觀。
