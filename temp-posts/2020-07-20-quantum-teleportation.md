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

從七龍珠的悟空到各種遊戲裡的閃現，瞬間移動這種夢幻般的技能常常在 ACG 裡出現。但你知道在現實生活中，我們已經可以利用量子力學來做到類似的效果而且有成功的實驗了嗎？這篇文章的目的就是帶大家了解瞬間移動背後的理論與計算。如果你對基本的線性代數不夠了解的話，推薦你先去看看 [3 blue 1 brown 的播放清單](https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)，也歡迎讀一下[淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/)增加對量子比特的一些認識。

## 先備知識

在我們開始之前，除了上面那些知識之外，我們還得再了解一些概念。首先，我們要瞬間移動的是任何一個處在疊加態的量子比特，用數學來表示就是：

$$
\left| \psi \right> = \alpha \left| 0 \right>  + \beta \left| 1 \right>
$$

我們並不清楚 \\(\alpha\\) 和 \\(\beta\\) 的數值，但這其實不重要，我們只要確保當它瞬移完 \\(\alpha\\) 和 \\(\beta\\) 沒有改變就可以。

接下來，我們快速介紹一些瞬移需要用到的量子閘 (Quantum Gate)。量子閘與電腦裡的邏輯閘很類似，它們是用來改變量子比特狀態的閘門。在[淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/)裡有提到量子比特在數學的世界裡就是一個向量。而量子閘我們則是用矩陣來表達。當量子閘作用於量子比特時，輸出量子比特的狀態就是矩陣乘上向量。

### X閘 (X Gate)

X閘相當於邏輯閘的「非」門，他可以將
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

矩陣形式為:

$$
\begin{pmatrix}
    0 && 1\\
    1 && 0\\
\end{pmatrix}
$$

來看看X閘作用於
$$
\left| 0 \right>
$$
的計算：

$$
\begin{pmatrix}
    0 && 1\\
    1 && 0\\
\end{pmatrix}
\begin{pmatrix}
    1 \\
    0 \\
\end{pmatrix}
=
\begin{pmatrix}
    0 \\
    1 \\
\end{pmatrix}
$$

別忘了：
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