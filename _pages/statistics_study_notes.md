---
title: Statistics study notes
author: DNV825
created: 2025-03-22
date: 2025-03-22
order: 3
category: Jekyll
layout: post
extra_css:
  - assets/tikzjax/fonts.css
  - assets/tikzjax/tikzjax-adjuster.css
  - assets/mathjax/align-left.css
extra_header_js: assets/tikzjax/tikzjax.js
---

## 統計学 基礎の基礎

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
