---
layout     : post
title      : "Elm 簡介"
subtitle   : "介紹 Elm 的一些基本語法與概念"
date       : 2016-04-23
author     : "Rhadow"
tags       : Elm
comments   : true
signature  : true
---

## 前言

函數式編程 (Functional Programming) 一直都是大家在討論卻又很少在專案中看到的一種編程模式。雖然少見，但還是有不少的函數式編程語言像是： Haskell, Scala, Closure 等等被開發出來在各式各樣的專案中使用。但是，這些語言很少被拿來當成開發前端的工具。今天要跟各位介紹的就是為前端人所設計的函數式編程語言： Elm。

## 介紹

Elm 是一個強型別的函數式編程語言。我們在最終會將它編譯成 JavaScript 以便於在瀏覽器上使用。在它的[官網](http://elm-lang.org/)你可以看到它所主打的一些特色，我個人認為以下幾點是非常吸引人的:

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

```elm
add x y = x + y

add = (\x y -> x + y)

add 1 2 -- 3
```

相當於 JavaScript 的

```js
function add(x, y) {
    return x + y;
}

// ES6
var add = (x, y) => x + y;

add(1,2) // 3
```

上面的例子使用了兩種不同的方法建立 `add` 函數 (正規與 inline)。最行一行則是使用函式的方法。而出現在最後的 `--` 在 Elm 中就是註解。

如果需要有類似局部變數 (local variable) 的時機時可以使用 `let...in` 的寫法：

```elm
cube x =
    let
        squaredX = x * x
    in squaredX * x

cube 5 --125
```

在 Elm 世界的函數裡，回傳的值會是最後執行到的那行的結果。

### 常用資料結構

Elm 提供了各式各樣的資料結構供我們使用，本篇會針對常用的三個 (List, Tuple 及 Record) 做介紹。較不常用的 Array, Set 和 Dict 歡迎大家參考這篇 [Data Structures in Elm](http://tech.noredink.com/post/140646140878/data-structures-in-elm)。

#### List

List 類似於 JavaScript 中的 Array，但又不全然相同。在原生的 Elm 中，我們無法像在 JavaScript 裡一樣靠 index 來取值，如果需要這功能的話需要額外載入[elm-list-extra](http://package.elm-lang.org/packages/circuithub/elm-list-extra/3.10.0/List-Extra) 或使用 Elm 提供的 Array。需要特別注意的是在 List 中的每個元素必須是**相同**型別的。

```elm
xs = [1,2,3,4]
xs = 1 :: [2,3,4]
xs = [1,2] ++ [3,4]
```

以上三行都代表著一個叫 `xs` 的 List 包含了 1,2,3,4 的值在裡面。第一種表達法相當直觀，這邊就不多作解釋。第二種的 `::` 在 Elm 中稱作 `cons`，相當於 JavaScript 中的 `unshift`。最後一種做法則是類似於 JS 中的 `concat`，會將前後兩個 List 合併為一個。有興趣瞭解其他關於 List 方法的讀者可到[官方文件](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/List)查詢。

#### Tuple

Tuple 在許多程式語言中都有，它有點像是簡易版的 List。差別在於內部元素的數量是在建立時就決定的，無法像 List 一樣使用 `cons` 或其他方法修改，同時內部的元素也可以是**不同**的型別。

```elm
t1 = ("Foo", 1)
t2 = ("bar", 2, [True, False])

fst t1 -- "Foo"
snd t2 -- 2
```

我們可以使用 `fst` (first 的縮寫) 來取得 Tuple 中第一個元素，`snd` (second 的縮寫) 來取得第二個元素。 更多和 Tuple 相關的文件請到[官網](http://elm-lang.org/)查詢。

#### Record

Record 類似於 JavaScript 中的 Object。

```elm
person = { name = "Rhadow", age = 18 }
person.name -- "Rhadow"
.age person -- 18

updatedPerson = { person | name = "Amy" }
updatedPerson.name -- "Amy"
updatedPerson.age -- 18

person.name -- "Rhadow"
```

相當於 JavaScript (ES7) 中的

```js
let person = { name: "Rhadow", age: 18 };
person.name // "Rhadow"
person.age // 18

let updatedPerson = { ...person, name: "Amy" }
updatedPerson.name // "Amy"
updatedPerson.age // 18

person.name // "Rhadow"
```

上例比較特別的地方在於 Elm 把 `.xxx` (xxx 為 key 值) 也做成了一個函數，所以我們不只可以用 `person.name` 也可以用 `.name person` 來取得同樣的值。

另一個特色是在 Elm 裡，所有的資料都是 Immutable，因此當我們想要更新 person 內部的值時需要指派一個新的變數 (`updatedPerson`) 給它，同時原本的變數 (`person`) 會保持不變。

### 模式匹配 (Pattern Matching)

模式匹配是一個非常強大的功能，它有點類似於 ES6 中的 destructuring，但又附帶了一些額外的功能。

```elm
head' list =
    case list of
        [] -> Nothing
        [x] -> Just x
        x::xs -> Just x

head' [] -- Nothing
head' [2] -- Just 2
head' [1,2,3] -- Just 1
```

上面的例子是一個叫做 `head'` 的函數，用來幫我們取出 list 中第一個元素。是的，你沒看錯，在 Elm 中 `'` 可被拿來與字串一起使用當成函數的名稱。 `case ... of ...` 類似於我們熟知的 `switch`。緊接著 `case` 下面的三行列出了 `list` 這個變數的所有可能以及應對的方法。

當 list 為空時，我們會回傳 `Nothing`。`Nothing` 有點類似於 JS 中的 `null` 或 `undefined`，`Nothing` 與 `Just xxx` 的細節會在接下來介紹型別的段落討論。第二個可能是當 list 只有一個元素時，我們直接將該元素回傳。最後一個可能則是含有多個元素時的應對，這裡可以看到模式匹配強大的地方。假設 list 為 `[1,2,3]` 時，我們知道 `1 :: [2, 3] == [1, 2, 3]`，模式匹配會自動將我們指派的變數 `x` 設為 `1` 和 `xs` 設為 `[2, 3]` 以利作為後續利用。

### 型別

我們知道 Elm 是一個強型別的語言，除了提供一些常見的型別像是 `Int`, `Float`, `String`, `Bool` 等等...也同時提供給開發者自創型別的方法。得到新型別的手法有兩種，一種是建立型別的別名，另一種則是直接建立新的型別。在講如何建立型別之前，讓我們先看看如何將這些型別定義到我們已知的知識上吧。

如果我們拿剛提到的 `add` 例子來看，加入型別後的程式會像是以下：

```elm
a: Int
a = 1

b: Int
b = 2

add: Int -> Int -> Int
add x y = x + y

c: Int
c = add a b -- c == 3
```

`:` 單一冒號代表定義變數的型別，Elm 提倡在寫程式時最好將型別定義一同寫入。這邊比較有趣的是 `add: Int -> Int -> Int`，函數的型別定義規則是最後一個箭頭指向的型別就是被回傳的值的型別，而其他的則是輸入參數的型別。因此，`add` 的定義是會接受兩個 `Int` 並回傳一個 `Int`。

#### 型別別名 (type alias)

有時候，單純的基本型別可能無法完全詮釋我們想要表達的意思，也有可能是資料結構太複雜，我們想用一個簡單的名稱稱呼它，這時型別別名就可派上用場。

```elm
type alias Name = String
type alias Age = Int
type alias Person =
    { name: String
    , age: Int
    }

createPerson: Name -> Age -> Person
createPerson name age =
    { name = name
    , age = age
    }
```

`type alias Name = String` 的意思可以翻譯為 `將 Name 設定為 String 的另一個別名`。如此一來，我們就可以像這樣定義:

`createPerson: Name -> Age -> Person`

而不是

`createPerson: String -> Int -> Person`

雖然兩者對程式來說相同，但前者對人類來說更清楚明瞭。

#### 建立自創型別與 Union Type

自創型別的方法如下：

```elm
-- Example 1.
type Directions = Up | Down | Left | Right

-- Example 2.
type Maybe a = Nothing | Just a
```

我們可以創造兩種不同款式的型別，第一種像是上例的 `Directions` 比較直觀簡單，在這型別下只會有四種可能的值，就是我們定義的 `Up`，`Down`，`Left`，`Right`。在使用上的話我們可以這麼做：

```elm
convertDirectionsToInt: Directions -> Int
convertDirectionsToInt dir =
    case dir of
        Up -> 1
        Down -> 2
        Left -> 3
        Right -> 4

convertDirectionsToInt Up -- 1
```

另一種則被稱為 Union Type，指的是我們可以將其它已存在的型別組合進我們自創的型別中。範例中我們建立了一個叫 `Maybe` 的型別，後面跟著的 `a` 是所謂的 `型別變數`，再更後面則是型別的值。我們可以自由在型別變數中帶入其他的型別如下：

```elm
-- 把 String 帶入 a
maybeString: Maybe String
maybeString = Just "I am a string"

-- 把 Int 帶入 a
maybeInt: Maybe Int
maybeInt = Just 1

anotherMaybeInt: Maybe Int
maybeInt = Nothing
```

以上三個變數都是屬於 `Maybe a` 型別，但隨著帶入的 `a` 不同，型別裡的值也會跟著改變。當然你也可以建立更複雜的 Union Type，例如:

```elm
type Directions a b = Up a | Down b | Left a b | Right
```

### 模組與條件判斷

呼～把困難的都說的差不多了，最後來點簡單的吧。

Elm 載入模組的方法非常簡單，假設我們有一個叫 `Html` 的模組並且我們想使用 `div` 這個函數時有以下幾種方法：

```elm
-- Example 1.
import Html

-- Example 2.
import Html as MyHtml

--Example 3.
import Html exposing (div)

--Example 4.
import Html exposing (..)
```

在範例一中想使用 `div` 時要這樣呼叫: `Html.div`

在範例二中想使用 `div` 時要這樣呼叫: `MyHtml.div`

在範例三中想使用 `div` 可以直接呼叫: `div`，但想使用其他函數時還是得加上命名空間: `Html.span`

在範例四中可直接呼叫任意在 Html 中的函數: `div`, `span`, `text`...


輸出模組時只需要在檔案最開始加入:

```elm
module MyModule where

--接下來寫的內容都可被輸出
```

Elm 的條件判斷相當簡潔，只有 `if..then..else..`

```elm
if sleepingHours < 8 then
    "Get some sleep!!"
else
    "Write more code!!"
```

你所熟知跟迴圈有關的 `for` 與 `while` 都不存在，在 Functional 的世界裡，你可以用 `map`， `filter`， `fold`(在 JS 叫 `reduce`) 或其他的方法來解決需要迴圈的問題。


## 結語

大家都知道前端的變動非常的快，但在了解 Elm 的一些基本概念後，你會發現前端的改動和 Elm 所前進的方向是相同的。例如：Facebook 推出的 ImmutableJS，單一資料流的 Redux，越來越多人關注的 Reactive Programming 等等都已經是 Elm 的基本配備 (是的，Elm 有內建類似 Observables 的東西，叫做 [Signal](http://elm-lang.org/guide/reactivity))。 當然，Elm 也是有它的弱勢，像是目前不支援 Server Side Rendering 以及社群太小還沒太多有用的套件。

最後個人的一點小心得：函數式編程語言有別於我們所習慣的寫程式方法，但有時候如果打開心胸去學習這種非主流的思考方式時，也會幫助你在你的程式生涯裡更上一層樓喔。
