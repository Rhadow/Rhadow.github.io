---
layout     : post
title      : "Elm Architecture 裡的 Address 由來"
subtitle   : "探討 Elm Architecture 與 Address 和 Signal 之間的關係"
date       : 2016-04-25
author     : "Rhadow"
tags       : Elm
comments   : true
signature  : true
---

## 前言

多數人在學習 Elm 時都會看到 [Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) 這個教學文件，也都應該會遇到那個神秘的 `address`。本篇會使用最簡單的 Counter 例子讓大家瞭解 `address` 與這個架構的關聯性。本篇文章使用的 Elm 是目前 (2016-04) 最新的版本 `v0.16.0`。

**2016-11-14 更新： 最新版本的 Elm 已經把 `Signal` 拿掉了，若有興趣了解過往的 Elm 是如何處理資料流的話再往下讀吧。這邊是[官方說明文件](http://elm-lang.org/blog/farewell-to-frp)。**

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

type Action
    = Increase
    | Decrease
    | NoOp

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

`update` 的型別定義為 `Action -> Model -> Model` ，代表著接收一個 `Action` 與原本的 `Model`。在經過我們設定的邏輯後，進而產生一個最新的 `Model` 給我們。很直覺吧！

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

目前的 `view` 的型別定義也很直觀，接受資料並根據內容產生出畫面，在這個例子中就是計數器目前的算到的值。

## Main

為了要讓 Elm 執行我們剛剛做好的畫面，我們需要透過 `main` 把以上函數給串接起來。

```elm

main: Html
main = view init

```

到目前為止的程式碼如下，有興趣的讀者可將程式碼複製貼上到官方提供的[線上編譯器](http://elm-lang.org/try)測試:

<script src="https://gist.github.com/Rhadow/22085e7a3cfb1cb9505307f0a855c7d8.js"></script>

## Signal, Mailbox 與 Address

目前程式跑起來了，但是問題在於我們與畫面完全無法互動。我們都知道 Elm 裡的變數都是 Immutable，無法隨意變動。因此，Elm 將反應式編程 (Reactive Programming) 的概念帶入並提供了 [Signal](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal) 供開發者使用。

### Signal

Signal 其實就是一個會隨著時間改變的變數，你也可以把它想像成是一個隨著時間發生一連串事件後產生的結果。一個簡單的例子就是如果我們有個 Signal 用來儲存目前滑鼠的位置，隨著我們移動滑鼠，Signal 裡的內容也會跟著變動。你可能會說那直接允許我們改變變數，不要把什麼都規定要 Immutable 不就簡單多了嗎？ Elm 的思考模式是: 沒錯，東西會改變，但我們需要用一些特別的方法在納入改變的同時也能確保狀態的穩定性。至於為什麼這麼執著於 Immutability，歡迎大家參考這篇[演講](https://www.youtube.com/watch?v=I7IdS-PbEgI)。


接下來，讓我們來看看 [Signal](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal#Signal) 的型別：

```elm

type Signal a

```

在上例中可以發現 Signal 的內容可以是任何型別。讓我們來思考一下，最終的畫面是個會變動的 `Html` 型別，也就是 `Signal Html`。記得我們的 `view` 函數嗎？畫面是以 `model` 為基底，因此，也需要有一個 `Signal Model` 型別的變數。我們又透過 `update` 函數根據不同的 `Action` 來改變 `model`，所以最後還需要一個 `Signal Action` 型別的變數。如果你還是不太清楚 Signal 的觀念的話，可以看看這個 [視覺化 Signal](http://yang-wei.github.io/elmflux/#/mouseSignal) 的 Demo


由於我們開發的應用不可能都像這個計數器這麼簡單，因此，Elm 提供了一個特殊的方法使開發者能夠在複雜的結構下仍然能夠有效率的改動和保留這些 Signal 的狀態。它就是 `Mailbox (信箱)` 囉。

### Mailbox 與 Address

Mailbox 隸屬於 `Signal` 這個模組下面而且概念就真的像一個信箱一樣，有專屬的`地址 (address)` 與信箱裡的內容 `(signal)`。讓我們來看看[官方](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal#Mailbox)對 `Mailbox` 的型別定義：

```elm

type alias Mailbox a =
    { address : Address a
    , signal : Signal a
    }

```

當我們想要修改某個 Signal 內部的值時，必須明確的告知 Elm 這個 Signal 所在的地址，這也就是 `address` 的由來。官方提供了 [mailbox](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal#mailbox) 這個函數來讓我們快速建立起一個信箱。需要特別注意的是 `mailbox` 是小寫的才是函數，大寫的 `Mailbox` 是型別喔。

## 將 Signal 套用

接著我們把剛剛了解的概念套用到計數器上，還記得我們需要哪些 Signal 嗎？先是需要一個 `Signal Action`，進而產生 `Signal Model`，最後才是我們的結果 `Signal Html`。Elm 的其中一個好處就是能夠透過型別來推斷出我們需要哪些函數，最後再一一實作它們。

```elm

actionMailbox: Signal.Mailbox Action
actionMailbox = Signal.mailbox NoOp

```

我們透過 `Signal.mailbox` 建立起了專屬 `Action` 型別的信箱，這裡使用了當初設定的 `NoOp` 作為初始值。這樣我們需要的 `Signal Action` 就可以透過 `actionMailbox.signal` 來取得，並透過專屬的 `actionMailbox.address` 來更新它。


接下來需要的是 `Signal Model`:

```elm

modelSignal: Signal Model
modelSignal = Signal.foldp update init actionMailbox.signal

```

這短短的一行程式碼是將整個程式串接起來的核心。[foldp](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Signal#foldp) (fold from past 的簡寫) 類似於 JavaScript 中的 `reduce`，會將我們提供的 Signal (本例中為 `actionMailbox.signal`) 中的值取出並透過同樣是我們自定的 `update` 函數更新 model 的狀態。而 `init` 則是 model 的初始值。


在取得 `modelSignal` 後，自然就是把畫面 render 出來囉。在那之前，我們先來更新一下之前所寫的 `view` 函數：

```elm

view: Signal.Address Action -> Model -> Html
view address model =
    div
        []
        [ button [onClick address Decrease] [ text "-" ]
        , span [] [text (toString model)]
        , button [onClick address Increase] [ text "+"]

```

我們多新增了一個型別為 `Signal.Address Action` 的變數。別被它的怪異長相嚇到，其實這只是從 `Signal` 模組裡取出 `Address` 這個型別而已。而這個 `Address` 型別又需要另一個型別變數來完成 (在本例就是 `Action`) 才變得這麼嚇人的。假設今天我們在載入模組時使用 `import Signal exposing (Address)` 的話，它的型別定義就會是 `Address Action -> Model -> Html`，是不是親民一點了呢？

至於為什麼要新增這個名為 `address` 的變數相信大家都很清楚，就是為了將新的 `Action` 送到它專屬的信箱中並透過一系列的更新，最終將畫面顯示為我們期望的樣子。

好啦，終於到了最後一步。我們需要將 `modelSignal` 中的 `model` 取出並帶入 `view` 中進而產生出一個 `Signal Html`。剛好，`Signal` 模組中有個符合我們需求的函數，就是大名鼎鼎的 `map` 啦。就像是 List 的 `map` 一樣， `Signal.map` 會幫我們把 `Signal` 內的值取出並帶入我們指定的函數內，接著產出一個新的 Signal。

這個指定的函數也是小有學問的。照道理來說，這個指定函數就是 `view`。但是 `view` 接收的參數不只一個，而是 `address`，再來才是 `map` 會丟給我們的 `model`。好在，在 Elm 的世界裡，所有函數都是預設可以 [Curry](http://www.sitepoint.com/currying-in-functional-javascript/) 的。意味著，我們只需要將 `address` 帶入後就可以取得一個型別為 `Model -> Html` 的函數。至於這個 `address` 要帶什麼值呢？當然就是 `actionMailbox.address` 囉！

```elm

main: Signal Html
main = Signal.map (view actionMailbox.address) modelSignal

```

最終完成的程式碼如下，一樣歡迎大家複製貼上到 [elm-lang.org/try](http://elm-lang.org/try) 實驗看看：

<script src="https://gist.github.com/Rhadow/bd80cb378b3bcdcc4a00eb6678f6535f.js"></script>

## 結語

在看完本篇後，希望大家都能夠明白 [Start-App](https://github.com/evancz/start-app) 在背後為我們做了些什麼，也希望大家能夠瞭解 address 的由來。未來在寫起 Elm 應用程式時也更清楚整個來龍去脈，在需要變化的時候，也不會受限於 [Start-App](https://github.com/evancz/start-app) 而能夠更靈活的發揮！
