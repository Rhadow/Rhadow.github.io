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

從七龍珠的悟空到各種遊戲裡的閃現，瞬間移動這種夢幻般的技能常常在幻想世界裡出現。但你知道在現實生活中，我們已經可以利用量子力學來做到類似的效果而且有成功的實驗了嗎？這篇文章的目的就是帶大家了解瞬間移動背後的理論與計算。你可能需要對線性代數有一些了解，如果不是很熟的話，推薦你先去看看 [3 blue 1 brown 的播放清單](https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) 中的前面幾集，也歡迎讀一下 [淺談量子電腦](https://rhadow.github.io/2019/06/14/quantum-computer-i-introduction/) 增加對量子比特 (Qubit) 的一些認識。

## 先備知識

在我們開始之前，我們需要定義本篇文章指的瞬間移動是什麼。正確的來說是叫量子傳送 (Quantum Teleportation)，是一個把 Qubit 的狀態傳送到遠處 Qubit 上的技術。由於傳送前後的 Qubit 之間沒有任何連結，因此產生出資訊瞬間移動的效果。這邊要注意的是傳送的最快速度還是光速並不是一個超越光速能讓你瞬移到宇宙各處的技術。剛剛提到我們要傳送的是一個的 Qubit，而這個 Qubit 的狀態一般來說都是未知的，用數學來表示就是：

$$
\left| \psi \right> = \alpha \left| 0 \right>  + \beta \left| 1 \right>
$$

雖然我們並不清楚 \\(\alpha\\) 和 \\(\beta\\) 的數值，但這其實不重要，我們只要確保傳送前後這兩個數值沒有改變就算成功了。\\(\alpha\\) 和 \\(\beta\\) 就是我們要傳送的資訊。一旦它們改變，代表這個 Qubit 的狀態與傳送前不同，也意味著傳送失敗。

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

X gate 相當於邏輯閘的「非」門也是三個泡利閘 (Pauli Gates) 的其中之一。下圖是 X gate 的真值表，每一行表示當這個閘門的輸入是左列的數值的話，輸出就是右列的值。例如把
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

### Z閘 (Z Gate)

泡利閘 (Pauli Gates) 的其中之一，真值表：

![z_gate_truth_table](https://rhadow.github.io/public/quantum/z_gate_truth_table.png)

線路圖表示：

![Z gate](https://rhadow.github.io/public/quantum/z_gate.png)

矩陣形式為:

$$
\begin{bmatrix}
    1 && 0\\
    0 && -1\\
\end{bmatrix}
$$

前面提到的量子閘輸入都是一個 Qubit。接下來要介紹的是兩個輸入的量子閘。

### CNOT閘 (Controlled Not Gate 或 CNOT Gate)

這邊把線路圖放到前面，解釋起來比較清楚：

![cnot gate](https://rhadow.github.io/public/quantum/cnot_gate.png)

CNOT gate 有兩個輸入，一個是圖中有點的那一行，這個輸入相當於下面 X gate 的開關。當這個 Qubit 是
$$
\left| 1 \right>
$$
時，下面那行的 X gate 才會起作用。

真值表：

![cnot_gate_truth_table](https://rhadow.github.io/public/quantum/cnot_gate_truth_table.png)

可以看到當第一個 Qubit 是
$$
\left| 0 \right>
$$
時，第二個 Qubit 並沒有被反轉，相當於第二行的 X gate 不存在。但當第一個 Qubit 是
$$
\left| 1 \right>
$$
時，X gate 就發揮作用了。

矩陣形式為:

$$
\begin{bmatrix}
    1 && 0 && 0 && 0\\
    0 && 1 && 0 && 0\\
    0 && 0 && 0 && 1\\
    0 && 0 && 1 && 0\\
\end{bmatrix}
$$

### CZ閘 (Controlled-Z Gate 或 CZ Gate)

CZ 和 CNOT 類似，只是把 X gate 換成 Z gate。

![cz gate](https://rhadow.github.io/public/quantum/cz_gate.png)

真值表：

![cz_gate_truth_table](https://rhadow.github.io/public/quantum/cz_gate_truth_table.png)

矩陣形式為:

$$
\begin{bmatrix}
    1 && 0 && 0 && 0\\
    0 && 1 && 0 && 0\\
    0 && 0 && 1 && 0\\
    0 && 0 && 0 && -1\\
\end{bmatrix}
$$

### 測量 (Measurement)

測量在量子線路裡有著破壞性的後果，一旦 Qubit 被測量，它就會崩塌到
$$
\left| 0 \right>
$$
或
$$
\left| 1 \right>
$$
其中一個狀態。這樣講可能有點難懂，這時我們就要請出物理學四大神獸之一的[薛丁格的貓](https://zh.wikipedia.org/wiki/%E8%96%9B%E5%AE%9A%E8%B0%94%E7%8C%AB)。在這個思想實驗裡，我們在一個盒子中放入毒氣與貓，在我們打開盒子之前並不知道貓是生是死 (貓處於疊加態)。一但盒子被打開，貓的狀態 (死活) 就被確定了也就是前面提到的崩塌 (疊加態的崩塌)。在線路圖裡，測量的表示為：

![measure gate](https://rhadow.github.io/public/quantum/measure_gate.png)

這邊值得注意的小細節是過了測量後線從一條變成了兩條。這代表 Qubit 已經崩塌，作用已退化成一般電腦用的位元 (bit)。

### 不可克隆原理 (No-cloning theorem)

不可克隆原理指的是我們無法建構出一個量子線路來複製一個未知狀態的 Qubit。各位有興趣可以去看[數學證明](https://www.youtube.com/watch?v=owPC60Ue0BE)。量子傳送的初衷就是將一個未知狀態的 Qubit 從 A 點送到 B 點，而如果複製是可行的話，我們也就不用大費周章的傳送，只需要在目的地將目標複製出來就行了。也正是因為這個限制，才有了量子傳送的發明。

## 量子傳送

花了很多篇幅把先備知識交代完，總算可以進入正題了。我會先試著用文字的方式大概描述傳送的過程，接著再帶著各位配合量子線路一步步做計算，讓大家看到量子傳送是如何在數學上得到證明。