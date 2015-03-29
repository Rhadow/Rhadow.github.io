---
layout     : post
title      : "學習 Flexbox 版面配置"
subtitle   : "分享 CSS Flexbox 語法介紹與應用"
date       : 2015-02-26 00:05:42
author     : "Rhadow"
tags       : CSS Flexbox
comments   : true
signature  : true
---

Flexbox 是 CSS3 裡一個比較特殊的版面設計模式。最初被設計出來的主要原因是為了提供更有效率的方式來完成傳統 CSS 無法輕易達成的效果 (例如垂直置中)。 Flexbox 適用於小區塊的版面使用，因此，整體版面結構還是建議用傳統的 Grid 模式做調整。本文將簡介目前最新標準的 Flexbox 語法。參考與圖片來源為:

* [Treehouse- A Complete Guide to Flexbox](http://css-tricks.com/snippets/css/a-guide-to-flexbox/)
* [Dive into Flexbox](http://bocoup.com/weblog/dive-into-flexbox/)

## 為什麼要學 Flexbox
由於 Flexbox 的規範從 2009 年發布到 2012 年間經歷了許多變動，導致許多瀏覽器的支援性不佳，也讓開發者遲遲不敢使用這個技術。不過幸運的是，目前的規範已趨於穩定，而 Flexbox 也能有效地解決許多切版問題，各家的瀏覽器也開始比較有規範的支援。更重要的是，前一陣子才宣布的 [React Native](https://code.facebook.com/videos/786462671439502/react-js-conf-2015-keynote-introducing-react-native-/) 就是使用 Flexbox 作為切版基底。也就是說，未來想透過 React Native 來寫 mobile app 的開發者或多或少都需要了解一點 Flexbox。

## 基礎概念
Flexbox 不同於其他單一的 CSS 屬性，它有專屬它自己的一些屬性和抽象概念。主要可分為兩大類:

* Flex container (容器)
* Flex item (子項目)

想要將一個 HTML 元素設定為 Flex container 相當容易，你只需要在元素中加入 `display: flex` 或 `display: inline-flex` 。兩者的差別類似 `display: block` 和 `display: inline-block` 的差別。所有在 Flex container 內的子元素就是 Flex item。如下圖:

![Flexbox concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2011/08/flexbox.png)

在圖中可觀察到的另一個重要觀念就是 Flexbox 是以兩個主軸為基礎，main axis 與 cross axis。這邊先大致說明一下圖中的各個關鍵字:

* **main axis:** 這是 Flexbox 內的主軸之一，所有的 Flex item 都會沿著 main axis 排列。要特別注意的是，main axis 的方向不一定是水平的，它會依照 `flex-direction` 的值做改變
* **main start | main end:** main axis 的起點邊界與終點邊界
* **main size:** Flexbox 的寬或高，方向依照 `flex-direction` 的值決定
* **cross axis:** Flexbox 內的另一個主軸，垂直於 main axis。方向依照`flex-direction` 的值做改變
* **cross start | cross end:** cross axis 的起點邊界與終點邊界
* **cross size:** Flexbox 的寬或高，方向依照 `flex-direction` 的值決定

在 Flexbox 裡，所有元素的位置都是相對於上面所述的兩個主軸。隨著方向改變，所有的 flex item 位置也會有所變化。

## Flex Container 屬性介紹
### flex-direction

![Flex Direction concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-direction1.svg)

此屬性決定了 main axis 的方向，也代表著所有 flex item 的內容會如何排列。

```css
.container {
    flex-direction: row | row-reverse | column | column-reverse;
}
```

* **row:** (預設值) 當 `writing-mode` 為 `ltr` 時，main axis 方向為從左到右。當 `writing-mode` 為 `rtl` 時，main axis 方向為從右到左
* **row-reverse:** 當 `writing-mode` 為 `ltr` 時，main axis 方向為從右到左。當 `writing-mode` 為 `rtl` 時，main axis 方向為從左到右
* **column:** 同 row ，方向改為從上到下
* **column-reverse:** 同 row-reverse ，方向改為從下到上

### flex-wrap

![Flex Wrap concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-wrap.svg)

Flexbox 預設會將所有 Flex item 都排列在同一排，此屬性可讓 flex container 在同一排塞不下所有 item 時產生換行效果。

```css
.container{
    flex-wrap: nowrap | wrap | wrap-reverse;
}
```

* **no-wrap:** (預設值) 不換行，當 `writing-mode` 為 `ltr` 時，排列方向為從左到右。當 `writing-mode` 為 `rtl` 時，排列方向為從右到左
* **wrap:** 換行，方向同上
* **wrap-reverse:** 換行，方向與 wrap 相反

### flex-flow

`flex-flow` 屬性是 `flex-direction` 和 `flex-wrap` 的縮寫，可快速定義 flex container 的主軸。預設為 `row nowrap` 。寫法如下：

```css
.container{
    flex-flow: <flex-direction> || <flex-wrap>
}
```

### justify-content

![Justify-content concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2013/04/justify-content.svg)

`justify-content` 是用來定義 flex item 在 main axis 上的排列位置。它會依照賦予的值分配多餘的空白空間。

```css
.container {
    justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

* **flex-start:** (預設值) Flex item 會靠往 `main-start` 對齊
* **flex-end:** Flex item 會靠往 `main-end` 對齊
* **center:** 於 main axis 上置中 Flex item 
* **space-between:** 將空白平均分配到所有 Flex item 之間，第一個 item 靠在 `main-start`，最後一個 item 靠在 `main-end`
* **space-around:** 平均分配空白到所有的 Flex item 周圍，包括第一個與最後一個 item。你可能會注意到最左和最右邊的間距會比中間小，原因是在於中間的間隔是左邊 item 的右間隔加上右邊 item 的左間隔

### align-items

![align-items concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2014/05/align-items.svg)

此屬性是用來定義 flex item 在 cross axis 上的排列位置。你可以這樣想：cross axis 與 `align-items` 的關係就像 main axis 與 `justify-content`之間的關係。

```css
.container {
    align-items: flex-start | flex-end | center | baseline | stretch;
}
```

* **flex-start:** Flex item 會靠往 `cross-start` 對齊
* **flex-end:** Flex item 會靠往 `cross-end` 對齊
* **center:** 於 cross axis 上置中 Flex item
* **baseline:** Flex item 會依照各個 item 的基準線對齊
* **stretch:** (預設值) 自動伸縮將多餘的空間補滿

### align-content

![align-content concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2013/04/align-content.svg)

`align-content` 主要是用在當 main-axis 有換行時。此屬性相當於 `justify-content` 與 `align-items` 的綜合版，不過它排列的是整個內容，而不是單一行。排列對應的軸是 cross axis 而非 main-axis。注意：當 main axis 沒換行時，此屬性不會有作用。

```css
.container {
    align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

* **flex-start:** 所有內容會靠往 `cross-start` 對齊
* **flex-end:** 所有內容會靠往 `cross-end` 對齊
* **center:** 於 cross axis 上置中所有內容
* **space-between:** 將空白平均分配到每行內容之間，第一行靠在 `cross-start`，最後一行靠在 `cross-end`
* **space-around:** 平均分配空白到每行內容周圍，包括第一行與最後一行
* **stretch:** (預設值) 自動伸縮將多餘的空間補滿

## Flex Item 屬性介紹
所有包在 Flex Container 內的直接子元素都是 Flex item，包含文字。`float`，`clear` 和 `vertical-align`對 Flex item 沒有作用。Flex item 內的元素與內容不會被 Flexbox 影響。舉個例子：在 Flex item 元素上加入 `float: left` 不會有任何反應，但在 Flex item 的子元素裡加入 `float: left` 則會正常往左浮動。

### order

![order concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2013/04/order-2.svg)

`order` 是一個非常好用的屬性，它賦予了我們不用修改 HTML 就能更動元素位置的能力。一般來說，每個 item 都是依照在 HTML 中的排列順序出現，但此屬性可控制元素出現的先後順序。值越小的排越前面。

```css
.item {
    order: <integer>;
}
```

### flex-grow

![flex-grow concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-grow.svg)

此屬性賦予每個 item 在需要時自動佔滿剩餘寬度的能力。接受的值為一個無單位的數字，此數字決定了 item 在剩餘的空間內該佔滿多少比例的寬度。如果所有的 item 都將 `flex-grow` 設為 1，那麼每個 item 都會有相同寬度並佔滿所有空間。如果將其中一個的 `flex-grow` 改為 2 的話，被修改的元素將會佔滿比其他元素兩倍大的空間。

```css
.item {
    flex-grow: <number>; /* default 0 */
}
```

### flex-shrink

此屬性為 `flex-grow` 的相反。

```css
.item {
    flex-shrink: <number>; /* default 1 */
}
```

### flex-basis

此屬性會依照 `flex-direction` 定義 item 最小的寬度或高度，有點類似 `min-width`。這個屬性可以翻譯成：當需要伸縮時，依照 `flex-grow` 或 `Flex-shrink` 的值來伸縮，但是寬度或高度不能小於 X px 或 em。

```css
.item {
    flex-basis: <length> | auto; /* default auto */
}
```

### flex

`flex` 屬性是 `flex-grow`，`flex-shrink` 和 `flex-basis` 的縮寫，第二和第三個參數 (`flex-shrink` 和 `flex-basis`) 可不給。預設值為 `0 1 auto`。

```css
.item {
    flex-basis: <length> | auto; /* default auto */
}
```

建議使用此屬性而非獨立使用上述三個屬性來設定 item 寬/高度。

### align-self

![align-self concept from TreeHouse](http://cdn.css-tricks.com/wp-content/uploads/2014/05/align-self.svg)

此屬性可以將個別 item 的 `align-items` 屬性覆蓋過去進而提供更為彈性的設計模式。

```css
.item {
    align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

對於此屬性的值的解說，請參考 `align-items` 部分。

## 總結

Flexbox 是一個非常強大的排版模式，它輕易的解決了以往許多需要使用各種怪異的方法來解決的問題。但，它的整體架構與常見的 Grid 系統又不太一樣，開發者需要換一個角度來思考版面的配置。希望這篇文章有讓大家準備好在未來的專案裡使用 Flexbox。如果文章內有任何問題，也歡迎各位討論糾正。