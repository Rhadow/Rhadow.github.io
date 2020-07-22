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

從七龍珠的悟空到各種遊戲裡的閃現，瞬間移動這種夢幻般的技能常常在 ACG 裡出現。但你知道在現實生活中，我們已經可以利用量子力學來做到類似的效果而且有成功的實驗了嗎？這篇文章的目的就是帶大家了解瞬間移動背後的理論與計算。你可能需要對線性代數有一些了解，如果不是很熟的話，推薦你先去看看 [3 blue 1 brown 的播放清單](https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) 前面幾集，也歡迎讀一下 [淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/) 增加對量子比特 (Qubit) 的一些認識。

## 先備知識

在我們開始之前，我們需要定義本篇文章指的瞬間移動是什麼。正確的來說是叫量子傳送 (Quantum Teleportation)。由於傳送的速度接近光速，在一定的距離裡，對人類來說就出現類似瞬間移動的效果。我們要傳送的是一個處在任意疊加態 (Superposition) 的 [Qubit](https://zh.wikipedia.org/wiki/%E9%87%8F%E5%AD%90%E4%BD%8D%E5%85%83)，用數學來表示就是：

$$
\left| \psi \right> = \alpha \left| 0 \right>  + \beta \left| 1 \right>
$$

我們並不清楚 \\(\alpha\\) 和 \\(\beta\\) 的數值，但這其實不重要，我們只要確保傳送前後這兩個數值沒有改變就算成功傳送了。一旦 \\(\alpha\\) 和 \\(\beta\\) 改變，代表這個 Qubit 的狀態與傳送前不同，也意味著傳送失敗。

接下來，我們快速介紹一些傳送會需要用到的量子閘 (Quantum Gate)。量子閘與電腦裡的邏輯閘很類似，它們是用來改變 Qubit 狀態的閘門。在 [淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/) 裡有提到 Qubit 在數學的世界裡就是一個向量。

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

而量子閘我們則是用矩陣來表達。當量子閘作用於 Qubit 時，輸出的 Qubit 狀態就是矩陣乘上向量。

### X閘 (X Gate)

X gate 相當於邏輯閘的「非」門。下圖是 X gate 的真值表，每一行表示當這個閘門的輸入是左列的數值的話，輸出就是右列的值。例如把
$$
\left| 0 \right>
$$
輸入給 X gate，它會將輸入 Qubit 轉換成
$$
\left| 1 \right>
$$
並輸出。

![x_gate_truth_table](https://rhadow.github.io/public/quantum/x_gate_truth_table.png)

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

Hadamard gate 又稱 H gate。H gate 在量子演算法裡被大量地被運用，原因是它可以把一個在非疊加態的 Qubit 例如
$$
\left| 0 \right>
$$
轉換成疊加態。 H gate 同時也是產生糾纏態 (Entanglement) 線路的必要零件。以下是它的真值表：

![h_gate_truth_table](https://rhadow.github.io/public/quantum/h_gate_truth_table.png)

線路圖表示：

![H gate](https://rhadow.github.io/public/quantum/h_gate.png)

矩陣形式為:

$$
\begin{bmatrix}
    \frac{1}{\sqrt{2}} && \frac{1}{\sqrt{2}}\\
    \frac{1}{\sqrt{2}} && \frac{-1}{\sqrt{2}}\\
\end{bmatrix}
$$
