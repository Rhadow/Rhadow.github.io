---
layout     : post
title      : "量子力學真的能做到瞬間移動嗎？"
subtitle   : "帶你了解量子瞬間移動背後的原理"
date       : 2020-07-19
author     : "Rhadow"
tags       : QuantumComputing
comments   : true
signature  : true
---

從七龍珠的悟空到各種遊戲裡的閃現，瞬間移動這種夢幻般的技能常常在 ACG 裡出現。但你知道在現實生活中，我們已經可以利用量子力學來做到類似的效果而且有成功的實驗了嗎？這篇文章的目的就是帶大家了解瞬間移動背後的理論與計算。你可能需要對線性代數有一些了解，如果不是很熟的話，推薦你先去看看 [3 blue 1 brown 的播放清單](https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) 前面幾集，也歡迎讀一下 [淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/) 增加對量子比特的一些認識。

## 先備知識

在我們開始之前，我們需要定義本篇文章指的瞬間移動是什麼。正確的來說是叫量子傳送 (Quantum Teleportation)。由於傳送的速度可以接近光速，在一定的距離裡，對我們來說就出現類似瞬間移動的效果。我們要傳送的是一個處在任意疊加態的 [量子比特](https://zh.wikipedia.org/wiki/%E9%87%8F%E5%AD%90%E4%BD%8D%E5%85%83)，用數學來表示就是：

$$
\left| \psi \right> = \alpha \left| 0 \right>  + \beta \left| 1 \right>
$$

我們並不清楚 \\(\alpha\\) 和 \\(\beta\\) 的數值，但這其實不重要，我們只要確保傳送前後這兩個數值沒有改變就算成功傳送了。一旦 \\(\alpha\\) 和 \\(\beta\\) 改變了，這個量子比特的狀態就與傳送前不同了也意味著傳送失敗。

接下來，我們快速介紹一些傳送會需要用到的量子閘 (Quantum Gate)。量子閘與電腦裡的邏輯閘很類似，它們是用來改變量子比特狀態的閘門。在 [淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/) 裡有提到量子比特在數學的世界裡就是一個向量。

$$
\left| 0 \right> = \begin{bmatrix}
    1 \\
    0 \\
\end{bmatrix}
$$

$$
\left| 1 \right> = \begin{bmatrix}
    0 \\
    1 \\
\end{bmatrix}
$$

而量子閘我們則是用矩陣來表達。當量子閘作用於量子比特時，輸出的量子比特的狀態就是矩陣乘上向量。

### X閘 (X Gate)

X gate 相當於邏輯閘的「非」門，它可以將
$$
\left| 0 \right>
$$
轉換成
$$
\left| 1 \right>
$$
，
$$
\left| 1 \right>
$$
轉換成
$$
\left| 0 \right>
$$
。

線路圖表示：

![x_gate](https://rhadow.github.io/public/quantum/x_gate.png)

X gate 有兩種表示法，右邊的形式比較常出現在等等會提到的 CNOT gate。

矩陣形式為:

$$
\begin{bmatrix}
    0 && 1\\
    1 && 0\\
\end{bmatrix}
$$

### H閘 (Hadamard Gate)

Hadamard gate 又簡稱 H gate 相當於邏輯閘的「非」門，它可以將
$$
\left| 0 \right>
$$
轉換成
$$
\left| 1 \right>
$$
，
$$
\left| 1 \right>
$$
轉換成
$$
\left| 0 \right>
$$
。

線路圖表示：

![H gate](https://rhadow.github.io/public/quantum/h_gate.png)

矩陣形式為:

$$
\begin{bmatrix}
    1 && 0\\
    0 && -1\\
\end{bmatrix}
$$
