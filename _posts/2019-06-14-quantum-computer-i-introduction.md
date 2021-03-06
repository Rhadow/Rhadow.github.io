---
layout     : post
title      : "淺談量子電腦"
subtitle   : "什麼是量子電腦？它與現在使用的電腦有什麼差別？"
date       : 2019-06-14
author     : "Rhadow"
tags       : QuantumComputing
comments   : true
signature  : true
---

## 什麼是量子電腦？

在現代生活中，人類建立了各種演算法讓電腦去執行並解決複雜的問題，但還是有許多難題是目前的電腦無法解決或是需要執行很長的時間才能得到答案。舉個簡單的例子，如果有人問你 15 是哪兩個質數乘出來的，你可以很快的回答 3 和 5。但如果把 15 換成一個三百位數的數字呢？有人估計大約需要 [15 萬年](https://www.quora.com/Is-it-true-that-factoring-a-300-digit-number-takes-150-000-years-for-current-computers-China-is-going-to-launch-a-quantum-satellite-National-TV-CCTV-created-a-video-to-illustrate-quantum-computers-and-said-that-they-could-factor-it-in-a-second) 才能得出解答。正是這類型的難題促使了量子電腦的誕生。

量子電腦與我們使用的傳統電腦有著很大的不同。在傳統電腦中，我們使用 bit 也就是常聽到的 1 和 0 來做計算單位。在量子電腦裡，最小的計算單位是 Qubit (量子比特)，它有著一些超乎常理的特性例如疊加態，糾纏態，使量子電腦可以在某些問題的計算上比傳統電腦還要快。

量子電腦的出現並不會取代傳統電腦。它們之間的關係比較像是車子與船的差別，各自有自己擅長解決的領域。因此可以期待在未來看到兩種電腦並行一起解決各種難題。

## 概率與硬幣

在開始討論神奇的 qubit 之前，讓我們先來玩個猜硬幣正反面的遊戲吧。把一枚硬幣放到盒子裡搖晃後反蓋在桌上讓你猜那一面向上，你猜對的機率是多少？相信大家都知道會有一半 (50%) 的機會猜對。用數學來表示這枚硬幣的機率的話可以寫成以下向量：

$$
\begin{pmatrix}
    0.5 \\
    0.5 \\
\end{pmatrix}
$$

上面的 0.5 代表出現正面的機率，下面的 0.5 則是出現反面的機率。如果硬幣被動了手腳，出現正面的機率變成 90% 的話，那機率的表示就會變成：

$$
\begin{pmatrix}
    0.9 \\
    0.1 \\
\end{pmatrix}
$$

我們也可以把它寫成下面這種形式：

$$
\begin{pmatrix}
    0.9 \\
    0.1 \\
\end{pmatrix} = 0.9 \begin{pmatrix}
    1 \\
    0 \\
\end{pmatrix}  + 0.1 \begin{pmatrix}
    0 \\
    1 \\
\end{pmatrix}
$$

之所以要寫成這樣是因為我們把
$$
\begin{pmatrix}
    1 \\
    0 \\
\end{pmatrix}
$$
$$
\begin{pmatrix}
    0 \\
    1 \\
\end{pmatrix}
$$
給抽了出來，它們分別代表了正面與反面也是硬幣的兩個基本狀態。這樣一來就可以把硬幣的概率公式化：

$$
\begin{pmatrix}
    p_{0} \\
    p_{1} \\
\end{pmatrix} = p_{0} \begin{pmatrix}
    1 \\
    0 \\
\end{pmatrix}  + p_{1} \begin{pmatrix}
    0 \\
    1 \\
\end{pmatrix}
$$

\\(p_{0}\\) 是出現正面的機率，\\(p_{1}\\) 是出現反面的機率。這兩個變數還需要滿足兩個條件：

1. \\(p_{0}, p_{1} \geq 0\\)，因為機率不會是負的
2. \\(p_{0} + p_{1} = 1\\)，一個事件的所有可能性機率相加是一

當有兩枚硬幣時，公式會變化成如下：

$$
\begin{pmatrix}
    p_{00} \\
    p_{01} \\
    p_{10} \\
    p_{11} \\
\end{pmatrix} = p_{00} \begin{pmatrix}
    1 \\
    0 \\
    0 \\
    0\\
\end{pmatrix}  + p_{01} \begin{pmatrix}
    0 \\
    1 \\
    0 \\
    0 \\
\end{pmatrix} + p_{10} \begin{pmatrix}
    0 \\
    0 \\
    1 \\
    0 \\
\end{pmatrix} + p_{11} \begin{pmatrix}
    0 \\
    0 \\
    0 \\
    1\\
\end{pmatrix}
$$

\\(p_{00}\\) 是兩枚正面的機率，\\(p_{01}\\) 是出現正反的機率，\\(p_{10}\\) 是出現反正的機率，\\(p_{11}\\) 是出現都是反面的機率。這四個機率一樣要滿足上面提到的兩個條件。

以上就是我們所熟悉的機率。接著再來看看機率跟量子電腦有什麼關係。

## 量子比特 (Qubit) 和疊加態 (Superposition)

在微觀世界中，我們認知的物理常識並不適用(有興趣的人可以查查看`雙縫實驗`)。大部分量子的狀態在實際觀測前都像開箱前的硬幣一樣是用機率來表示的，而這種不確定最後結果的狀態，科學家們給了一個名字叫做疊加態 (superposition)。

舉個簡單的例子：我們準備一顆向下自旋的電子，讓它通過一個有一半機率使它往上轉的裝置。這時，這顆電子就處在疊加態。在實際觀測這顆電子的自旋前，我們必須用類似上面的機率公式來表示這顆電子自旋的狀態。直到觀測之後疊加態就會崩塌成其中一種基本狀態。

另一種不那麼準確的說法是這顆電子正同時向上與向下轉，其實最好用來詮釋量子特性的語言是數學。接下來，我們來看看在數學中是如何表示一個量子或是 qubit 的狀態。這邊開始就不再用正面與反面或向下轉與向上轉而用 0 與 1 來代表兩個不同的基本狀態。首先來定義基本狀態：

$$
\left| 0 \right> = \begin{pmatrix}
    1 \\
    0 \\
\end{pmatrix}
$$

$$
\left| 1 \right> = \begin{pmatrix}
    0 \\
    1 \\
\end{pmatrix}
$$

上面那個奇怪的括弧叫做 bra-ket notation，是物理學家用來表示向量的一種方法。量子狀態的表示公式：

$$
\left| \psi \right> = \psi_{0} \left| 0 \right>  + \psi_{1} \left| 1 \right>
$$

是不是和之前看到用來表示硬幣機率的公式非常類似？他們之間唯一的差別就在於滿足條件的差別，在這裡只有一個條件需要滿足：

$$
\psi_{0}^2 + \psi_{1}^2 = 1
$$

\\(\psi_{0}^2\\) 代表觀測到 0 的機率，\\(\psi_{1}^2\\) 代表觀測到 1 的機率。

## 糾纏態 (Entanglement)

接著來看看當有兩個 qubit 時公式會有什麼樣的變化，基本狀態變成四種可能：

$$
\left| 00 \right> = \begin{pmatrix}
    1 \\
    0 \\
    0 \\
    0 \\
\end{pmatrix}
$$

$$
\left| 01 \right> = \begin{pmatrix}
    0 \\
    1 \\
    0 \\
    0 \\
\end{pmatrix}
$$

$$
\left| 10 \right> = \begin{pmatrix}
    0 \\
    0 \\
    1 \\
    0 \\
\end{pmatrix}
$$

$$
\left| 11 \right> = \begin{pmatrix}
    0 \\
    0 \\
    0 \\
    1 \\
\end{pmatrix}
$$

公式變成：

$$
\left| \psi \right> = \psi_{00} \left| 00 \right>  + \psi_{01} \left| 01 \right> + \psi_{10} \left| 10 \right> + \psi_{11} \left| 11 \right>
$$

看起來與雙硬幣的概率公式也差不多，但讓我們來看看兩個例子：

第一個例子假設有兩個 qubit 的狀態如下：

$$
\left| \psi \right> = \frac{1}{2} \left| 00 \right>  + \frac{1}{2} \left| 01 \right> + \frac{1}{2} \left| 10 \right> + \frac{1}{2} \left| 11 \right>
$$

如同一個 qubit 時，觀測機率是該組合前方係數的平方，例如 \\(\left| 00 \right>\\) 的機率是 \\((\frac{1}{2})^2\\)。
我們可以很輕易地預測每種組合出現的機率都是 \\(\frac{1}{4}\\)，而且全部加起來也是一，符合條件。來試著做分解：

$$
\left| \psi \right> = \frac{1}{2} \left| 00 \right>  + \frac{1}{2} \left| 01 \right> + \frac{1}{2} \left| 10 \right> + \frac{1}{2} \left| 11 \right> = (\frac{1}{\sqrt{2}} \left| 0 \right>  + \frac{1}{\sqrt{2}} \left| 1 \right>)(\frac{1}{\sqrt{2}} \left| 0 \right>  + \frac{1}{\sqrt{2}} \left| 1 \right>)
$$

可以發現分解後小括弧內其實是一個一半機率出現 0 一半機率出現 1 的 qubit，因此我們可以把這兩個 qubit 看成是兩個獨立的 qubit 組合而成的。用猜硬幣的例子來說，一次丟兩枚硬幣，和一次丟一枚丟完兩枚後再來猜正反面並不影響各種組合的機率。

再來看看第二個例子：

$$
\left| \psi \right> = \frac{1}{\sqrt{2}} \left| 00 \right>  + \frac{1}{\sqrt{2}} \left| 11 \right>
$$

恩...有 \\(\frac{1}{2}\\) 機率出現 00 和 \\(\frac{1}{2}\\) 機率出現 11，兩個機率加起來是一，符合公式的限制條件。接著來試著分解它，首先從兩個單獨的 qubit 來組組看：

$$
(a \left| 0 \right>  + b \left| 1 \right>)(c \left| 0 \right>  + d \left| 1 \right>) = 
ac \left| 00 \right>  + ad \left| 01 \right> + bc \left| 10 \right> + bd \left| 11 \right>
$$

如果要組成目標的話 `ad` 和 `bc` 一定是 0，假設 `a` 是 0 的話，那
$$
\left| 00 \right>
$$
那項就也會消失，說明 `d` 是 0 囉？但如果 `d` 是 0 的話，
$$
\left| 11 \right>
$$
也會消失。也就是說，例子二的狀態時無法被分解的。我們的確有兩個 qubit 但這兩個 qubit 沒辦法獨立，這種狀態科學家稱它為糾纏態 (Entanglement)。

## 應用

看到這裡你可能會想說這到底能做什麼？其實聰明的科學家們已經想出了一些酷炫玩法。
例如:

1. 超密編碼 (Super dense coding): 兩台電腦之間只需要傳送一個 qubit 就能夠帶著兩個 bit 的資訊。
2. 量子傳送 (Quantum Teleportation): 一種利用量子糾纏態特性來傳送量子至任意距離的技術。

還有一些利用疊加態的演算法，讓量子在計算過程中都保持在疊加態，直到最後才實際測量並拿出最後的結果。因為疊加態的特性，n 個 qubit 可以同時代表 \\(2^n\\) 個數字並同時進行運算。有興趣的人可以搜尋 `deutsch jozsa algorithm` 和 `shor algorithm`。

但這也不代表量子電腦就沒缺點。由於得出答案需要測量，每次測量的結果並不一定是正確答案，因此要多跑幾次找出最常出現的值才認定它是解答。最後，畢竟我不是物理專家，許多資訊也是從書籍中與網路得來。如有錯誤，還請各位大大指正。
