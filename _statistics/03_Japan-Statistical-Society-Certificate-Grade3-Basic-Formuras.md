---
title: 3. 統計検定3級 基本的な公式
author: DNV825
created: 2025-12-21
date: 2025-12-21
order: 3
category: Jekyll
layout: post
extra_css:
  - assets/tikzjax/fonts.css
  - assets/tikzjax/tikzjax-adjuster.css
  - assets/mathjax/align-left.css
extra_header_js: assets/tikzjax/tikzjax.js
---

## 3.1. 指数

### 3.1.1. 指数法則

指数に対して以下の法則が成り立つ。

$$
\begin{align*}
&a \neq 0, b \neq 0 とし、&&m, n を整数とする\\
&(1) \space a^m a^n = a^{m+n}        &&(2) \space \cfrac{a^m}{a^n} = a^{m-n} &&(3) \space (a^m)^n = a^{m n} \\
&(4) \space (ab)^n  = a^n b^n        &&(5) \space a^0 = 1                    &&(6) \space a^{-1}  = \cfrac{1}{a} \\
&(7) \space a^{-n}  = \cfrac{1}{a^n}
\end{align*}
$$

## 3.2. 対数

### 3.2.1. 対数の定義

対数の定義は以下の通り。対数はほぼ使わないけど。

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

### 3.2.2. 対数法則

対数に対して以下の法則が成り立つ。

$$
\begin{align*}
&(1) \space \log_a{M} + \log_a{N} = \log_a{MN}           &&(2) \space \log_a{M^p} = p\log_a{M}  &&(3) \space \log_a{\cfrac{1}{M}} = - \log_a{M} \\
&(4) \space \log_a{M} - \log_a{N} = \log_a{\cfrac{M}{N}} &&(5) \space \log_a{1}   = 0           &&(6) \space \log_a{b}            = \cfrac{\log_c{b}}{\log_c{a}}
\end{align*}
$$

### 3.2.3. 対数法則の証明

#### $(1) \log_a{M} + \log_a{N} = \log_a{MN}$

$\log_a{M} = x, \space \log_a{N} = y$ と置くと、対数の定義より $a^x = M, a^y = N$ よって、  
  指数法則により $a^{x+y} = MN$ となる。  
  これは対数の定義より、 $\log_a{MN} = x + y$ であることを表す。

#### $(2) \space \log_a{M^p} = p\log_a{M}$

$\log_a{M} = x$ と置くと、対数の定義より $a^x = M$ よって、指数法則を使うと $a^{px} = M^p$ となる。  
  これは対数の定義より、 $\log_a{M^p} = px$ となる。

#### $(3) \space \log_a{\cfrac{1}{M}} = - \log_a{M}$

$(2)$ において、 $p = -1$ とすれば得られる。  
$\log_a{M^{-1}} = \log_a{\cfrac{1}{M}} = {-1}\log_a{M}$

#### $(4) \space \log_a{M} - \log_a{N} = \log_a{\cfrac{M}{N}}$

$(1)$ において、 $N$ ではなく $\cfrac{1}{N}$ を指定すると、 $\log_a{M} + \underline{\log_a{\cfrac{1}{N}}} = \log_a{\left(M \cfrac{1}{N} \right)} = \log_a{\cfrac{M}{N}}$ となる。  
下線部は $(3)$ より $-\log_a{N}$ になるため、 $\log_a{M} - \log_a{N} = \log_a{\cfrac{M}{N}}$ が成立する。

#### $(5) \space \log_a{1} = 0$

$a^0 = 1$ であることから、対数の定義より $\log_a{1} = 0$ を得る。

#### $(6) \space \log_a{b} = \cfrac{\log_c{b}}{\log_c{a}}$

対数の定義より、 $a^p = b \Leftrightarrow \log_a{b} = p$ である。  
このとき、左側の式の $p$ に $\log_a{b}$ を代入すると、 $a^{log_a{b}} = b$ が成立する。  
ここで、両辺の対数をとる。対数の底は $c$ とする。すると、  
$$
\begin{align*}
a^{\log_a{b}} &= b \\
\log_c{a^{\log_a{b}}} &= \log_c{b} \\
\log_a{b} \log_c{a} &= \log_c{b} \\
\log_a{b} &= \cfrac{\log_c{b}}{\log_c{a}}
\end{align*}
$$

## 3.3. 平方完成

統計検定 3 級に登場した。出題されても 1 問程度なので捨ててよいかも。平方完成は $3x^2 -12x +6$ といった式の $x$ を $(x - n)^2$ の形にまとめる手法のこと。平方完成を使うと、$(x - n)^2 = X^2$ のように値を置き換えて考えることができるようになる。

$$
%\newcommand{\arraystretch}{2.0}
\begin{align*}
&3x^2 - 12x + 6 &&= 3 (x^2 - 4x) + 6                    && \dots \text{ $x^2$ の係数を外に出す } \\
&               &&= 3 (x^2 -2 \cdot 2x + 2^2 - 2^2) + 6 && \dots \text{ $-4x$ の係数を $\frac{1}{2}$ にし、かつ $\frac{1}{2}$ にした値の2乗を足し引きする} \\
&               &&= 3 \{(x - 2)^2 - 4\} + 6             && \dots \text{ $x^2 -2 \cdot 2x + 2^2$ を因数分解する } \\
&               &&= 3 (x - 2)^2 - 12 + 6                && \dots \text{ $3$ を展開する } \\
&               &&= 3 (x - 2)^2 - 6                     && \dots \text{ 数値を計算しておしまい }
\end{align*}
$$
