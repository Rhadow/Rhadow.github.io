---
layout     : post
title      : "量子電腦(2) - 量子閘？邏輯閘？"
subtitle   : "介紹量子運算常用到的量子閘"
date       : 2020-07-19
author     : "Rhadow"
tags       : QuantumComputing
comments   : true
signature  : true
---

量子運算就如同當今的電腦是由許多邏輯閘所建立出來的，想要理解量子演算法，認識這些量子閘 (Quantum gates) 是必要的第一步。本篇將帶領讀者一步一步認識常用的量子閘。

## 量子閘 (Quantum Gate) 是什麼

在我們開始之前，我們先快速了解一下量子閘是什麼。量子閘是可以改變量子比特狀態的閘門，透過組合這些閘門，我們可以做出各式各樣的運算。你可能會說，這不就是電腦裡的邏輯閘嗎？的確很類似，但量子閘有一個很重要的條件，就是它必須是可逆的。可逆代表我們可以透過閘門的輸出就可以推測出輸入是什麼。舉個例子，邏輯閘中的[「非」門](https://zh.wikipedia.org/wiki/%E5%8F%8D%E7%9B%B8%E5%99%A8) 就是一個可逆的閘門。當你丟 0 進去時他會輸出 1，丟 1 進去時他會輸出 0。所以當你看到 0 在輸出時，可以很自然的推測出 1 是輸入。而邏輯閘中的[「與」門](https://zh.wikipedia.org/wiki/%E4%B8%8E%E9%97%A8)就是不可逆的。當「與」門輸出 0 時，你無法確定輸入是 00, 01 還是 10。

我們在[前一篇](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/)提到過量子比特其實就是一個向量。量子閘則可以用矩陣來表達。當量子閘作用於量子比特時，輸出量子比特的狀態就是矩陣乘上向量。而這個矩陣必須滿足數學上的一個條件，它必須是[么正矩陣](https://zh.wikipedia.org/wiki/%E9%85%89%E7%9F%A9%E9%98%B5)。背後的原理當然要深入量子力學才能理解，這邊想要強調的重點是只要滿足上述條件的都是量子閘。

## X閘 (X Gate)

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

小提示：
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