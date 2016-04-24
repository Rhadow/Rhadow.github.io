---
layout     : post
title      : "Elm Architecture 裡的 Address 由來"
subtitle   : "探討 Elm Architecture 與 Address 和 Signal 之間的關係"
date       : 2016-04-27
author     : "Rhadow"
tags       : Elm
comments   : true
signature  : true
---

## 前言

多數人在學習 Elm 時都會看到 [Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) 這個教學文件，也都應該會遇到那個神秘的 `address`。本篇會使用最簡單的 Counter 例子讓大家瞭解 `address` 與這個架構的關聯性。本篇文章使用的 Elm 是目前 (2016-04) 最新的版本 `v0.16.0`。

## 需求

首先來瞭解一下我們的目標是做出一個簡單的[計數器](http://evancz.github.io/elm-architecture-tutorial/examples/1.html)，這也是[Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) 教學中的第一個例子。 接下來我們會在不使用 [Start-App](https://github.com/evancz/start-app) 這個 library 的幫助下完成這個程式。

## Model

首先決定 Model 的型別，由於計數器只需要一個數字，我們走最簡潔路線直接將它設定成 `Int` 並把初始值設為 `0`。

```elm
type alias Model = Int

init: Model
init = 0
```

## Action

Action 決定了我們能夠如何與程式互動。我們的目標只有兩個動作，一個增加，一個減少。

```elm
type Action = Increase | Decrease | NoOp
```

這裡多了一個 `NoOp` (No Operation 縮寫) 作為預設行為，至於為什麼要有這個 Action 會在接下來的內容裡提到。另外，如果你曾經使用過 Redux 架構的話，你會發現這與 Redux 中的 `actionTypes` 很類似。

## Update

`update` 包含了這個計數器裡的所有邏輯，對應到 Redux 就是所謂的 `reducer`。

```elm
update: Action -> Model -> Model
update action model =
    case action of
        NoOp -> model
        Increase -> model + 1
        Decrease -> model - 1
```

`update` 的型別定義為 `Action -> Model -> Model` 意味著接收一個 Action 與原先的 Model，經過我們設定的邏輯後，進而產生一個最新的 Model 給我們。很直覺吧！

## View

`view` 當然就是用來產生使用者介面的函數囉。

```elm
view: Model -> Html
view model =
    div
        []
        [ button [] [ text "-" ]
        , span [] [text (toString model)]
        , button [] [ text "+"]
        ]
```

目前的 `view` 的型別定義也很直觀，接受資料並根據內容產生出畫面，在這個例子就是計數器目前的數量。

## Main

為了要讓 Elm 執行我們剛剛做好的畫面，我們需要透過 `main` 把以上函數給串接起來。

```elm
main: Html
main = view init
```

到目前為止的程式碼如下，有興趣的讀者可將程式碼複製貼上到官方提供的[線上編譯器](http://elm-lang.org/try)測試:

{% gist 22085e7a3cfb1cb9505307f0a855c7d8 %}

## Signal, Mailbox 與 Address

目前程式跑起來了，但是問題在於我們與畫面完全無法互動。我們都知道 Elm 裡的變數都是 Immutable，無法隨意變動。因此，Elm 將反應式編程 (Reactive Programming) 的概念帶入並提供了 [Signal](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal) 供開發者使用。

### Signal

Signal 其實就是一個會隨著時間改變的變數，你也可以把它想像成是一個隨著時間發生一連串事件後產生的結果。一個簡單的例子就是如果我們有個 Signal 用來儲存目前滑鼠的位置，隨著我們移動滑鼠，Signal 裡的內容也會跟著變動。你可能會說那直接允許我們改變變數，不要把什麼都規定要 Immutable 不就簡單多了嗎？ Elm 的思考模式是: 沒錯，東西會改變，但我們需要用一些特別的方法在納入改變的同時也能確保狀態的穩定性。至於為什麼這麼執著於 Immutability，歡迎大家參考這篇[演講](https://www.youtube.com/watch?v=I7IdS-PbEgI)。


接下來，讓我們來看看 Signal 的型別：

```elm
type Signal a
```

在上例中可以發現 Signal 的內容可以是任何型別，我們也會利用這點來為計數器加入互動。

### Mailbox 與 Address
