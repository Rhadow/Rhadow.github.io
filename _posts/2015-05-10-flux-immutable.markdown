---
layout     : post
title      : "Immutable.js 簡介"
subtitle   : "簡介 ImmutableJS 與基本的內部資料結構"
date       : 2015-05-10 13:58:26
author     : "Rhadow"
tags       : ImmutableJS
comments   : true
signature  : true
---

最近有在使用 React 搭配 Flux 架構的開發者們應該都對
[Immutable.js](http://facebook.github.io/immutable-js/) 不會太陌生。這個函式庫能夠創建不可變的資料結構，使開發者能更清楚的理解目前資料的狀態，更能優化程式處理速度。同時，immutable.js 也與 React 內提供的 `pureRenderMixin` 相容，使本來就很快的 React 在速度上又提升到更高的層次。

## Immutable.js 內的資料結構

在 Immutable.js 的世界裡有一套屬於它自己的規則，如果我們想要擁有它所提供的便利性，就必須先把我們熟知的陣列與物件等轉換成它內部的資料結構來做更進一步的操作。由於 Immutable.js 有提供各式各樣不同的結構，本文將只介紹較基本且常用到的 List，Map 與 Sequence。對於其他結構有興趣的讀者，請閱讀[官方提供的說明頁](http://facebook.github.io/immutable-js/docs/#/)。

### fromJS()

在介紹這些內部結構之前，我們先來瞭解一下該如何將 JavaScript 轉換成 Immutable.js 可用的結構。轉換的方式有很多，`fromJS()`是比較直接的一種。這個函式的主要功能就是將你丟給他的 JavaScript 物件或陣列轉換成對應的 Immutable.js 結構。它的第二個參數則是一個 callback，讓開發者能夠自訂轉換出的最終結構，如不特別提供這個參數的話，預設會將陣列轉換成 List，物件轉換成 Map。

小提醒：`fromJS()`做的轉換是深轉換 (Deep Conversion)。

### List

在 Immutable 的世界裡，List 就相當於 JavaScript 的陣列。我們可以透過 `List.of()` 建立一個 List。

```js
var list1= Immutable.List.of('a','b','c');
var list2 = list1.push('d'); // 'a','b','c','d'
console.log(list1.size); // 3
console.log(list2.size); // 4
```

如同上例所示，list1 並沒有因為 `push()` 的關係改變了自已原本的結構，反而是生成了一個新的 List，而我們將它存為 list2。除此之外，許多 JavaScript 陣列原生方法例如 `shift()`，`unshift()`，`pop()` 等等，Immutable.js 都有實作對應的 API，讓開發者可以無縫接軌的使用。

理解如何生成 List 後，讓我們來看看如何取值與修改值吧。
如果 List 只有很單純的一層時，使用 `get()` 與 `set()` :

```js
var list1 = Immutable.List.of(1,2,3,4);
var list2 = list1.set(1,5);
// list2 相當於 Immutable.List.of(1,5,3,4)
console.log(list2.get(1)); // 5
```
使用的方式很簡單，就像陣列一樣透過 index 的方式取值改值。

當 List 內部又有包其他 List 時，則需要使用 `setIn()` 與 `getIn()` :

```js
var list1 = Immutable.fromJS([1,[2,3],4]);
var list2 = list1.setIn([1,1],100);
// list2 相當於 Immutable.fromJS([1,[2,100],4])
console.log(list2.getIn([1,0])); // 2
console.log(list2.getIn([1,1])); // 100
```

`setIn()` 和 `getIn()` 的用法也很簡單，陣列裡的數字其實就是每個層級的 Index。我們只要指到需要拿取或修改的 Index 就可以了。
上例在 `setIn()` 裡的 [1,1] 其實就是要取得 [1,[2,3],4] 裡的 [2,3] 裡的 3 並將它修改成 100。同理也可應用於取值。

### Map

Map 對應到的則是 JavaScript 中的物件。注意：物件並不等於是 Map，從 ES6 開始後，JavaScript 也有原生的 Map。建立 Map 與設值取值其實和 List 差不多:

```js
var map1 = Immutable.Map({js:'AngularJS',css:'Bootstrap'});
var map2 = map1.set('js','ReactJS');
console.log(map1.get('js')); // 'AngularJS'
console.log(map2.get('js')); // 'ReactJS'
```

深度取值與改值也是使用 `setIn()` 與 `getIn()` :

```js
var map1 = Immutable.fromJS({
    name: 'Howard',
    birthday: {
        year: 1988,
        month: 3,
        day: 28
    },
});
var map2 = map1.setIn(['birthday', 'year'], 2015);
console.log(map1.getIn(['birthday', 'year'])); // 1988
console.log(map2.getIn(['birthday', 'year'])); // 2015
```

### Sequence

Immutable.js 的設計靈感其實有一部分來自於 Clojure, Scala, Haskell 這些函數式編程語言。因此 Immutable.js 裡有個特殊的結構叫 Sequence。Map 和 List 都可以透過 `toSeq()` 這個方法來轉換成 Sequence。 Sequence 有兩個很重要的特性：

* Immutable (不可變)
* Lazy (延遲)

Immutable 我相信你應該很了解它的意思了，但延遲這特性是怎麼回事？我們直接來看看官方提供的例子好了：

```js
var oddSquares = Immutable.Seq.of(1,2,3,4,5,6,7,8)
  .filter(x => x % 2).map(x => x * x);
```

如果上面的例子是一般的 List 而不是 Sequence 的話，oddSquares 其實是等於 `[1,9,25,49]`。但因為 Sequence 擁有延遲的特性，在你要求它給你值之前他是不會把結果計算出來的。

```js
console.log(oddSquares.get(1)); // 9
```
當我們透過以上程式碼向它取值時，oddSquares 才會把 9 給我們。特別注意的是，它也不會繼續做後面 25 與 49 的運算。因為延遲的關係，Sequence 只會做到我們向它要求的地方。也因此，程式其實省去了很多不必要的運算，這也是為什麼 Immutable.js 會優化速度的原因之一。

為了理解，我們再來看一個官方提供的例子：

```js
Immutable.Range(1, Infinity)
  .skip(1000)
  .map(n => -n)
  .filter(n => n % 2 === 0)
  .take(2)
  .reduce((r, n) => r * n, 1);
// 1006008
```

Range 結構本身就是 Sequence，所以我們不需要特地使用 `toSeq()` 來做轉換。在這個範例中，我們可以把它想像成是一個類似陣列的結構。

首先，建立一個類似 `[1,2,3...Infinity]` 的 Range。因為 Immutable 的關係，每一步其實都是一個新的 Range，但由於 Lazy 的特性，這些為了取值而在過程中建立的 Range 都不會被儲存。

`skip(1000)` 的作用是跳過 Range 前 1000 個值。新的 Range 長這樣：`[1001, 1002...Infinity]`。

接下來的 `map()` 和 `filter()` 會把新的 Range 過濾成 `[-1002, -1004, -1006...Infinity]`。

`take(2)` 則是取 Range 內最前面兩筆資料。

最後再透過 `reduce()` 相乘得到 1006008。

以上這些運算如果不是透過 Sequence 延遲這個特性，在 `take(2)` 前的每一步都有大量的運算，電腦根本無法負荷。

## 總結

相信耐心讀完本文的讀者應該都對 Immutable.js 有了基本的了解。未來有機會的話也希望與各位分享如何將 Immutable.js 應用在 Flux 架構中。想要深入研究的讀者也可以閱讀[官方文件](http://facebook.github.io/immutable-js/docs/#/)或觀看 Immutable.js 的開發者 Lee Byron 在 [2015 F8 的演講](https://www.youtube.com/watch?v=I7IdS-PbEgI&list=PLb0IAmt7-GS1cbw4qonlQztYV1TAW0sCr&index=13)。
