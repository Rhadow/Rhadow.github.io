---
layout     : post
title      : "Jekyll x Github x Blog (Part 1)"
subtitle   : "探討 Jekyll 部落格架構與如何建立專屬的 Github 頁面"
date       : 2015-02-18 15:37:04
author     : "Rhadow"
tags       : Jekyll Github
comments   : true
signature  : true
---

由於身邊大部份的同事們都擁有屬於自己的部落格，用來分享技術或記錄自己的學習過程等，讓不想落後別人的我有了很大的動力來製作屬於自己的部落格。本篇主要是記錄 Jekyll 部落格的架構，Part 2 會分享一些功能的實作，更詳細的教學可參考 [Jekyll 官網](http://jekyllrb.com/)。

## Jekyll 簡介
Jekyll 是一個部落格平台，主要的功能是能夠讓人們可以在 Github 上使用靜態的 HTML 頁面建置部落格。使用者使用 markdown 語法撰寫部落格文章，Jekyll 則會透過動態模板 (Template) 將文章轉為靜態 HTML 網頁。同時 Github Pages 也提供方便的 Host 功能，只需要將你的專案存在 Github 的儲存庫 (Repository) 中，就能夠讓 Github 來協助我們作 Host 的動作，不用再費其他心思。

![alt text](http://jekyllrb.com/img/octojekyll.png "Jekyll’s Octocat mascot.")

## 靜態頁面的好處
當初會選擇使用 Jekyll 來當製作部落格的工具主要是因為以下的原因：

* 單純
  * 不需要 Database，所有的文章都被轉換成一份份的 HTML 檔
  * 快速，由於檔案都是靜態的 HTML 檔，載入起來速度相對會比較快
  * 輕巧，不會有預設的進階功能 (標籤，評論等)，換句話說，需要什麼功能都必須自己實作
* Host 方便
  * Github Pages 會自動將你的 Jekyll 部落格建置並 Host 在專屬於你的網址。這項功能完全免費，更棒的是還可以同時享受 Github 本來就有提供的版本控制功能
* 控制自由度高
  * 由於 Jekyll 只提供簡單的基本瀏覽功能，使用者可以客製化自己的部落格，也能從製作不同功能的過程中學習

## 快速建置 Jekyll 部落格
建置一個 Jekyll 部落格主要有兩種方法：

* 使用 Command Line 安裝 Jekyll，新增一個 Jekyll 專案並使用 `jekyll new` 來初始化。完成之後使用 `jekyll build` 和 `jekyll serve` 來建置一個位於本地端的部落格。詳細的流程可以參考 [Jekyll Quick-start 頁面](http://jekyllrb.com/docs/quickstart/)。
* 如果你像我一樣懶得把 Jekyll 的文件都讀一遍，另一個選擇就是使用其他高手以 Jekyll 為基底預先寫好的 Generator，並增加自己想要的新功能與樣式。
本部落格使用的 Generator 是 [Poole/Hyde](https://github.com/poole/hyde)，當然也有其他不錯的選擇，像是 [JekyllBootstrap](http://jekyllbootstrap.com/)，[MadeMistakes](https://mademistakes.com/work/) 等。

## 建立基礎
開始的第一步是將 [Poole/Hyde](https://github.com/poole/hyde) 的內容搬到我們自己的部落格儲存庫內。這邊要使用 Fork 或是 Clone 都可以，看各位喜好。

特別注意的是，在命名儲存庫時有兩種選擇：

* 自訂專案名稱
* 命名為 `你的帳號.github.io`

這兩種命名方法會影響部落格的 base URL 與版本分支結構：

使用自訂專案名稱的話，此儲存庫的 Github Page 會是一個專案網站。你的部落格 base URL 會是 `你的帳號.github.io/你的專案名稱`，同時必須將專案內容放到 `gh-pages` 的分支下，才能夠正常建置。每一個 Github 帳號能夠擁有多個專案網站。

另一方面，專案使用特定名稱 `你的帳號.github.io` 的話，此儲存庫的 Github Page 會成為使用者網站。部落格的 base URL 則是 `你的帳號.github.io/`。專案內容放在預設的 `master` 分支底下就能夠正常建置頁面。要注意的是每個 Github 帳號只能夠有一個使用者網站。

確定加入所有內容並 Push 後就算完成第一步，如果對應儲存庫的命名與存放內容的分支都正確的話，你已經可以在此儲存庫專屬的網址內看到 Poole/Hyde 的內容囉。

## 部落格結構
接著讓我們來看看 Poole 的基本架構，詳細的結構可以在 [Poole/Hyde](https://github.com/poole/hyde) 的官方 Github 頁面查詢：

```
404.html
CNAME
LICENSE.md
README.md
_config.yml
_includes
_layouts
_posts
about.md
atom.xml
index.html
public
```

當在本地端使用 `jekyll build` 和 `jekyll serve` 建置後，會有一個 `_site` 的資料夾產生，此資料夾是建置後的結果。在建置的過程中，所有的專案內容，除了有底線開頭的檔案都會被複製到這個 `_site` 資料夾中，markdown 檔案則會透過 [Liquid](http://liquidmarkup.org/) 動態模板轉換成靜態 HTML 頁面。

_posts 資料夾是用來存放所有 markdown 格式的部落格文章，Poole 預設的範例文章如下：

```
2013-12-31-whats-jekyll.md
2014-01-01-example-content.md
2014-01-02-introducing-poole.md
```

[index.html](https://github.com/poole/poole/blob/master/index.html) 應該不用解釋太多，簡單來說就是初始呈現出來的網頁結構，[about.md](https://github.com/poole/poole/blob/master/about.md) 則是自我介紹頁面，雖然說是頁面，但它其實只是個 markdown 格式的文章偽裝成頁面罷了。新增文章的方法很簡單，只需要另存一份 markdown 檔案到 _post 資料夾內，Jekyll 就會自動生成新頁面了。這邊比較需要注意的是檔案的命名規則，必須以 “年-月-日-文章標題” 的規則來命名。

[_config.yml](https://github.com/poole/poole/blob/master/_config.yml) 是很重要的檔案，部落格內的所有設定都必須到 `config.yml` 內去做，可設定的內容包含網址路徑，說明，markdown 的寫法等，詳細的內容可到 [Jekyll Configuration](http://jekyllrb.com/docs/configuration/) 查看。

最後，[_layout](https://github.com/poole/poole/tree/master/_layouts) 和 [_include](https://github.com/poole/poole/tree/master/_includes) 資料夾裡面裝著所有組成部落格的 boilerplate HTML 檔案：

```
default.html
page.html
post.html
$ ls -1 _includes/
head.html
```

## 樣式微調
[Poole/Hyde](https://github.com/poole/hyde) 有內建的一些樣式提供簡單的客製化功能，如下圖：
![Hyde in red](https://f.cloud.github.com/assets/98681/1831229/42b0b354-7384-11e3-8462-31b8df193fe5.png)

以下是 Poole/Hyde 提供的八個內建顏色：
![Hyde theme classes](https://f.cloud.github.com/assets/98681/1817044/e5b0ec06-6f68-11e3-83d7-acd1942797a1.png)
要改變顏色很容易，只需要在 `default.html` 加入對應的 class 到  `<body>` 就完成了。範例如下：

```html
<body class="theme-base-08">
  ...
</body>
```

Poole/Hyde 也提供將導覽 bar 調到右邊的樣式：
![Hyde with reverse layout](https://f.cloud.github.com/assets/98681/1831230/42b0d3ac-7384-11e3-8d54-2065afd03f9e.png)
要達成此效果也很簡單，一樣在 `default.html` 中的  `<body>` 內加入 `layout-reverse` 的 class 就可完成。範例如下：

```html
<body class="layout-reverse">
  ...
</body>
```

如果你了解一些 CSS 的技巧，也可以加入自己的 CSS 檔案，剩下的就是自由發揮了。

## 總結
使用 Jekyll 建置一個簡易的部落格不是一件難事，它擁有單純，方便與自由度等優點。但是，當你需要某些特殊功能時又該怎麼做呢？沒錯，就是自己動手刻。在接下來 Part 2 文章內，我會分享如何實作改善文章顯示方法，評論，標籤與加入 Google Analytics 等功能。如果文章內有任何錯誤或問題，歡迎各位討論糾正。
