---
layout     : post
title      : "如何使用 Webpack 模組整合工具"
subtitle   : "簡介 Webpack 功能與如何將它應用在 ReactJS 的專案中"
date       : 2015-03-23 09:52:04
author     : "Rhadow"
tags       : Webpack React
comments   : true
signature  : true
---

前端的世界變化的很快，最近如果有持續在關注 ReactJS 的人應該或多或少都會聽過 [Webpack](https://github.com/webpack/webpack) 這個名詞。這段 [Pete Hunt 的演講](https://www.youtube.com/watch?v=VkTCL6Nqm6Y) 清楚的指出了 Webpack 與其他模組整合工具的差別，也給了正在使用 gulp 和 browserify 的開發者另一個選擇來完成模組整合的工作。Pete 也在 Github 上寫了 [webpack-howto](https://github.com/petehunt/webpack-howto) 來更加詳細地解釋 Webpack 的概念。本篇將參考 Pete 提供的範例，英文比較好的讀者建議直接閱讀原出處。

## Webpack
Webpack 是德國開發者 Tobias Koppers 開發的模組整合工具。它的核心功能如下:

* 可同時整合 [CommonJS](http://www.commonjs.org/specs/modules/1.0/) 和 [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) 模組
* 轉換 JSX, Coffee Script, TypeScript 等
* 分散封裝專案使用的程式碼，使載入頁面時只需載入當頁所需的程式碼以加速載入速度
* 整合樣式表 (css, sass, less 等)
* 處理圖片與字型
* 建置 production-ready 的程式碼 (壓縮)

## 給使用 Browserify 的開發者

執行下列兩行指令會得到相同的結果:

```js
browserify main.js > bundle.js
```

```js
webpack main.js bundle.js
```

不過，Webpack 提供比 browserify 更多的功能，建議創建一個名為 `webpack.config.js` 的檔案來做集中管理。如果您有在用 Gulp，這隻檔案的功能就類似於 `gulpfile.js`。

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  }
};
```

## 如何啟動 Webpack

切換到有 `webpack.config.js` 檔案的目錄下並執行:

  * `webpack` : 會在開發模式下開始一次性的建置
  * `webpack -p` : 會建置 production-ready 的程式碼 (壓縮)
  * `webpack --watch` : 會在開發模式下因應程式碼的變換持續更新建置 (快速!)
  * `webpack -d` : 加入 source maps 檔案


## 將其他轉譯語言轉換回 JavaScript

Webpack 的 **loader** 相當於 browserify 內的 transforms。以下程式碼顯示了如何在 Webpack 中轉換 CoffeeScript 和 Facebook 的 JSX+ES6。(必須先 `npm install jsx-loader coffee-loader`):

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      { test: /\.js$/, loader: 'jsx-loader?harmony' } 
      // loaders 可以像 querystring 一樣接收參數
    ]
  }
};
```

如果希望在 `require()` 時不需要加入副檔名，可以像下面範例加入一個 `resolve.extensions` 屬性並告訴 webpack 哪些副檔名是可以省略的:

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      { test: /\.js$/, loader: 'jsx-loader?harmony' } 
      // loaders 可以像 querystring 一樣接收參數
    ]
  },
  resolve: {
    // 設定後只需要寫 require('file') 而不用寫成 require('file.coffee')
    extensions: ['', '.js', '.json', '.coffee'] 
  }
};
```


## 樣式表與圖片

Webpack 允許使用 `require()` 的方式將樣式表與圖片加入:

```js
require('./bootstrap.css');
require('./myapp.less');

var img = document.createElement('img');
img.src = require('./glyph.png');
```

當引用 CSS (less, sass等) 時，webpack 會將 CSS 內容包在 `<style>` 標籤中並加入到頁面內。
引用圖片時，webpack 則會返回圖片的路徑。
以下範例為如何設定對應的 `loader` :

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    path: './build', // 儲存圖片與JS檔案的目錄
    publicPath: 'http://mycdn.com/', 
    // webpack 使用 require() 時參考的路徑，例如圖片的路徑
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      { test: /\.less$/, loader: 'style-loader!css-loader!less-loader' }, 
      // 使用 ! 來串接 loaders
      { test: /\.css$/, loader: 'style-loader!css-loader' },
      { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} 
      // 當圖片大小小於 8k 時使用 base64 URL, 其餘使用直接連接到圖片的 URL
    ]
  }
};
```

不了解 base64 URL 的讀者可參考[維基百科](http://en.wikipedia.org/wiki/Data_URI_scheme)。

## 特定功能標籤

當有功能在某些條件下才會開啟時，例如顯示偵錯訊息，可以使用這個功能來隱藏部分程式碼。在你的程式碼內加入一些全域變數:

```js
if (__DEV__) {
  console.warn('Extra logging');
}
// ...
if (__PRERELEASE__) {
  showSecretFeature();
}
```

接著告訴 webpack 該如何處理這些全域變數:

```js
// webpack.config.js

// definePlugin 接收字串，因此你也可以直接寫入字串而不使用 JSON.stringify()
var definePlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.BUILD_DEV || 'true')),
  __PRERELEASE__: JSON.stringify(JSON.parse(process.env.BUILD_PRERELEASE || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  plugins: [definePlugin]
};
```

設定完成後，可以使用 `BUILD_DEV=1 BUILD_PRERELEASE=1 webpack` 來完成建置。
調整指令的參數即可模擬各狀況下的建置。特別注意的是，`webpack -p`在壓縮時會將沒用到的程式碼刪除，
因此不用擔心會有不該出現的程式碼被加入到最終產出的程式碼內。

## 分批整合

假設現在專案中有 Profile 頁面和 Feed 頁面。通常我們不希望使用者在連接到 Profile 頁面時載入 Feed 頁的程式碼。因此，我們需要為每個頁面分批整合:

```js
// webpack.config.js
module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' 
    // [name] 會依據上面 entry 的屬性名稱變動
  }
};
```

在 Profile 頁面，使用 `<script src="build/Profile.js"></script>` 引入。同理可應用到 Feed 頁面。


## 共用程式碼優化

假設 Feed 和 Profile 頁面擁有許多相同處 (像 React 的元件)，webpack 可以分析出共通處並額外整合成一組在頁面間可快取的共用包:

```js
// webpack.config.js

var webpack = require('webpack');

var commonsPlugin =
  new webpack.optimize.CommonsChunkPlugin('common.js');

module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js'
    // [name] 會依據上面 entry 的屬性名稱變動
  },
  plugins: [commonsPlugin]
};
```

別忘了要在引入 Page 和 Feed 頁面的 `script` 標籤前加入 `<script src="build/common.js"></script>`

## 非同步載入

Webpack 提供非同步載入功能。以下程式碼透過路徑名稱與非同步載入功能決定哪些模組需要被載入以避免載入一些非必要的程式碼。

```js
if (window.location.pathname === '/feed') {
  showLoadingState();
  require.ensure([], function() {
    hideLoadingState();
    require('./feed').show(); 
    // 此時 callback 內容所呼叫的模組 (此為 './feed') 已可以被同步呼叫
  });
} else if (window.location.pathname === '/profile') {
  showLoadingState();
  require.ensure([], function() {
    hideLoadingState();
    require('./profile').show();
  });
}
```

Webpack 預設要被引入的檔案存在於根目錄，你可以在 `output.publicPath` 的屬性下更改設定。

```js
// webpack.config.js
output: {
    path: "/home/proj/public/assets", //webpack 建置專案的路徑
    publicPath: "/assets/" //webpack 使用 require() 時參考的路徑
}
```

## 總結

Webpack 提供給了開發者另一種模組整合的選擇，它與 ReactJS 的配合度也相當高，目前的缺點在於缺乏詳細的文件解說。希望這篇文章有讓各位多認識 Webpack 一點，如果有興趣想深入研究請到 [Webpack 官方網站](http://webpack.github.io/docs/?utm_source=github&utm_medium=readme&utm_campaign=trdr) 閱讀更詳細的說明。