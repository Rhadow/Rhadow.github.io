---
layout     : post
title      : "Elm 簡介"
subtitle   : "介紹 Elm 的一些基本語法與架構"
date       : 2016-04-23
author     : "Rhadow"
tags       : Elm
comments   : true
signature  : true
---

## 前言

函數式編程 (Functional Programming) 一直都是大家在討論卻又很少在專案中看到的一種編程模式。雖然少見，但還是有不少的函數式編程語言像是： Haskell, Scala, Closure 等等被開發出來在各式各樣的專案中使用。但是，這些語言很少被拿來當成開發前端的工具。今天要跟各位介紹的就是專為前端人所設計的函數式編程語言： Elm。

## 介紹

Elm 是一個強型別的函數式編程語言。我們在最終會將它編譯成 JavaScript 以便於在瀏覽器上使用。在它的[官網](http://elm-lang.org/)你可以看到它所主打的一些特色，我個人認為以下幾項是非常吸引人的:

1.  不會有執行階段錯誤 (Runtime Error)
2.  Render 的速度非常快
3.  語法簡潔易懂，易於測試
4.  超級友善的錯誤訊息
5.  良好的設計架構 ([Elm-Architecture](https://github.com/evancz/elm-architecture-tutorial))

不會有執行階段錯誤代表著我們可以避免使用者直接看到程式在他面前爆炸，Elm 在編譯階段就會抓出許多潛在的錯誤，大大降低出包率。

Render 的速度在現在這個講求速度的年代已經是基本配備了，特別提出來的原因是在於它與目前最熱門的 React 一樣使用了 Virtual DOM 的技術來提高效能並號稱比 React 還快。以下是官方提供的比較圖給各位參考：

![Elm 效能比較圖](http://elm-lang.org/diagrams/sampleResults.png)

由於在 Elm 的世界裡只有 Immutable Data 和無副作用的函數 (Pure Function)，在開發時會變得很容易測試。當不小心寫錯程式時，貼心的錯誤提示也能夠幫助你快速修正。

最後提到的設計架構叫 [Elm-Architecture](https://github.com/evancz/elm-architecture-tutorial)。是作者提出的一種單向資料流設計模式。就連 React 御用的 Redux 當初也是從 Elm 這裡取得靈感才被開發出來的喔。

## 語法

說了這麼多介紹，也是要趕快來看看程式碼該怎麼寫吧。由於 Elm 是用 Haskell 開發出來的，它們兩個的語法可說是非常接近，熟悉 Haskell 的讀者應該會覺得非常親切。以下的語法是以目前(2016-04)最新的版本 `v0.16.0` 作為範例。

### Function

```
add x y = x + y

add = (\x y -> x + y)

add 1 2 -- 3
```

相當於 JavaScript 的

```
function add(x, y) {
    return x + y;
}

// ES6
var add = (x, y) => x + y;

add(1,2) // 3
```

上面的例子使用了兩種不同的方法建立 `add` 函數 (正規與 inline)。最行一行則是使用函式的方法。而出現在最後的 `--` 在 Elm 中就是註解。

如果需要有類似局部變數 (local variable) 的時機時可以使用 `let...in` 的寫法：

```
cube x =
    let
        squaredX = x * x
    in squaredX * x

cube 5 --125
```

在 Elm 世界的函數裡，回傳的值會是最後執行到的那行的結果。

### 模式匹配 (Pattern Matching)

模式匹配是一個非常強大的功能，它有點類似於 JavaScript 中的 switch 與 ES6 的 destructuring 的結合體，但又附帶了一些額外的功能。

```
head' list =
    case list of
        [] -> Nothing
        [x] -> Just x
        x::xs -> Just x
```
