---
title: Statistics study notes
author: DNV825
date: 2025-03-22
category: Jekyll
layout: post
extra_css:
  - assets/tikzjax/fonts.css
  - assets/tikzjax/tikzjax-adjuster.css
  - assets/mathjax/align-left.css
extra_header_js: assets/tikzjax/tikzjax.js
---

## TikZJax について

JavaScript で tikz を利用可能にするもの。すごい。参考：<https://tikzjax.com/>

以下のようにインクルードし、

```html
<link rel="stylesheet" type="text/css" href="https://tikzjax.com/v1/fonts.css">
<script src="https://tikzjax.com/v1/tikzjax.js"></script>
```

`<script type="text/tikz"></script>` タグの中に tikz を記述する。`html {cmd=true hide=true}` を指定しても、Markdown Preview Enhanced では実行してくれないようだ。

通常の TikzJax は % から始まるコメントが使えないが、[bill-ion さんの作成した fork](https://github.com/bill-ion/tikzjax-live/) ならコメントが使える。通常の TikzJax は 2 バイト文字の表示は非対応。テキストは英語を使うのが無難。

Jekyll で TikzJax を利用する場合、以下のように記述すればよい。それをさらに &#123;% raw %&#125; ... &#123;% endraw %&#125; で囲むこと。

```html
<script type="text/tikz">
\begin{document}
\begin{tikzpicture}
\draw (0,0) circle (1in);
\end{tikzpicture}
\end{document}
</script>
```

{% raw %}
<script type="text/tikz">
\begin{document}
\begin{tikzpicture}
\draw (0,0) circle (1in);
\end{tikzpicture}
\end{document}
</script>
{% endraw %}

## 指数と対数

対数はほぼ使わないけど。でも、たまに出てくるような…。

### 指数法則

指数に対して以下の法則が成り立つ。

$$
%\newcommand{\arraystretch}{2.0}
\begin{align*}
& a \neq 0, b \neq 0 とし、m, n を整数とする \\
& (1) \space a^m a^n=a^{m+n} && (2) \space \cfrac{a^m}{a^n} = a^{m-n} \\
& (3) \space (a^m)^n = a^{m n} && (4) \space (ab)^n = a^n b^n \\
& (5) \space a^0 = 1 && (6) \space a^{-1} = \cfrac{1}{a} \\
& (7) \space a^{-n} = \cfrac{1}{a^n}
\end{align*}
$$

### 対数の定義

対数の定義は以下の通り。

$$
\begin{align*}
& a > 0, a \neq 1, M > 0 のとき \\
\\
& a^p = M \Leftrightarrow \log_a{M} = p \\
\\
& (1) \space \log_a{M} を a を底とする M の対数という。\\
& (2) \space M を \log_a{M} の真数という。
\end{align*}
$$

### 対数法則

対数に対して以下の法則が成り立つ。

$$
%\newcommand{\arraystretch}{2.0}
\begin{align*}
& (1) \space \log_a{M} + \log_a{N} = \log_a{MN} \\
& (2) \space \log_a{M^p} = p\log_a{M} \\
& (3) \space \log_a{\cfrac{1}{M}} = - \log_a{M} \\
& (4) \space \log_a{M} - \log_a{N} = \log_a{\cfrac{M}{N}} \\
& (5) \space \log_a{1} = 0 \\
& (6) \space \log_a{b} = \cfrac{\log_c{b}}{\log_c{a}}
\end{align*}
$$

#### 対数法則の証明

$(1) \log_a{M} + \log_a{N} = \log_a{MN}$
: $$
\begin{align*}
& \log_a{M} = x, \space \log_a{N} = y \space \text{と置くと、対数の定義より} \\
& a^x = M, a^y = N \space \text{よって、指数法則により} \\
& a^{x+y} = MN \space \text{となる。これは対数の定義より} \\
& \log_a{MN} = x + y \space \text{であることを表す。}
\end{align*}
$$

## 計算方法

### 平方完成

統計検定 3 級に登場した。出題されても 1 問程度なので捨ててよいかも。平方完成は $3x^2 -12x +6$ といった式の $x$ を $(x - n)^2$ の形にまとめる手法のこと。平方完成を使うと、$(x - n)^2 = X^2$ のように値を置き換えて考えることができるようになる。

$$
%\newcommand{\arraystretch}{2.0}
\begin{align*}
&3x^2  - 12x + 6 \\
& = 3 (x^2 - 4x) + 6 & \dots & \text{ $x^2$ の係数を外に出す } \\
& = 3 (x^2 -2 \cdot 2x + 2^2 - 2^2) + 6 & \dots & \text{ $-4x$ の係数を $\cfrac{1}{2}$ にし、かつ $\cfrac{1}{2}$ にした値の2乗を足し引きする}\\
& = 3 \{(x - 2)^2 - 4\} + 6 & \dots & \text{ $x^2 -2 \cdot 2x + 2^2$ を因数分解する }\\
& = 3 (x - 2)^2 - 12 + 6 & \dots & \text{ $3$ を展開する } \\
& = 3 (x - 2)^2 - 6 & \dots & \text{ 数値を計算しておしまい }
\end{align*}
$$

## 基礎の基礎

### 平均値

$$
%\newcommand{\arraystretch}{2.0}
\begin{align*}
\bar{x} = \cfrac{\text{観測値の合計}}{\text{観測値の個数}} = \cfrac{x_1 + x_2 + \cdots + x_n}{n} = \cfrac{1}{n}\sum_{i=1}^{n}{x_i}
\end{align*}
$$

### 中央値

n 個の観測値 $x_1, x_2, \cdots , x_n$ を昇順（小さい順）に並べたものを

$$
%\newcommand{\arraystretch}{2.0}
\begin{align*}
&{x_{(1)} \le x_{(2)} \le \cdots \le x_{(n)}} \\
\end{align*}
$$

とするとき、

$$
%$\newcommand{\arraystretch}{2.0}
\begin{align*}
&\text{n が奇数の場合}: x_{\frac{(n+1)}{2}}\\
&\text{n が偶数の場合}: \cfrac{x_{\frac{n}{2}} + x_{\frac{n}{2}+1}}{2}\\
\end{align*}
$$

### 最頻値

最も頻繁に出現する値のこと。

## ベイズの定理

{% raw %}
<script type="text/tikz">
\begin{document}
\begin{tikzpicture}[scale=0.1]

% 大枠の四角
\draw (0,0) rectangle (160,100);

% 保菌者（0.01%）の領域
\fill[orange!30] (0,3) rectangle (3,100);

% 陽性（真）のラベル
\draw[thick] (0,100) -- (-10,100);
\draw[thick, <->] (-5,4) -- (-5,99)  node [midway, left,  align=center] {$99\%$};
\draw[thick] (0,3) -- (-10,3);
\draw[->] (10,50) -- (5,50) node[right] at (10,50) {Positive (true)};

% 非保菌者（99.99%）のうち陽性（誤り）の領域
\fill[blue!20] (0,100) rectangle (160,97);

% 陽性（誤り）のラベル
\draw[<->] (10,97) -- (10,100) node[midway,right]{$1\%$};
\draw[thick, ->] (75,93) -- (75,97) node[below] at (75,92) {Positive (false)};

% 上部のラベル
\draw[thick, <->] (0,103) -- (3, 103) node[midway, above,align=center] at(0, 105) {P(B) = 0.01\%\\(carrier) };
\draw[thick, <->] (3,103) -- (160,103) node [midway, above, align=center] {P($\overline{B}$) = 99.99\%\\(not carrier)};

% 数式
\draw (200,90) -- (180,90) node[left, align=right] {P(A $\vert$ B) =};
\fill[orange!30] (187,96) rectangle (193,92);
\fill[orange!30] (180,88) rectangle (186,84);
\draw (190,86) node {$+$};
\fill[blue!20] (194,88) rectangle (200,84);

\end{tikzpicture}
\end{document}
</script>
{% endraw %}
