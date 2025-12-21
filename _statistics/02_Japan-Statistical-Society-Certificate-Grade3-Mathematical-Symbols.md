---
title: 2. 統計検定3級 数学記号
author: DNV825
created: 2025-12-21
date: 2025-12-21
order: 2
category: Jekyll
layout: post
extra_css:
  - assets/tikzjax/fonts.css
  - assets/tikzjax/tikzjax-adjuster.css
  - assets/mathjax/align-left.css
extra_header_js: assets/tikzjax/tikzjax.js
---

## 2.1. 統計検定3級 代表的な数学記号

統計検定3級に登場する数学記号たち。

| 代表的な記号 | $LaTeX$ 記法 | 意味（英語） | 意味（日本語） |
| --- | --- | --- | --- |
| $M$ | `M` | Median | 中央値 |
| $Q_1, Q_2, Q_3$ | `Q_1, Q_2, Q_3` | First Quartile (= 25%), Second Quartile(= 50% (median), Third Quartile (= 75%) | 第1四分位数、第2四分位数、第3四分位数（四分位点、四分位） |
| $IOR$ | `IOR` | Inner Quartile Range ($Q_3 - Q_1$) | 四分位範囲
| $r$ | `r` | Correlation Coefficient | 相関係数 |
| $r_{xy}$ | `r_{xy}` | - | $x$ と $y$ の相関関係、 他に $r(x, y)$ など |
| $R^2$ | `R^2` | Coefficient of Determiration | 決定係数 |
| $s^2$ | `s^2` | Variance | 分散 |
| $s$ | `s` | Standard Diviation | 標準偏差（分散の正の平方根） |
| $s^2_x, s_{xx}$ | `s^2_x, s_{xx}` | - | 観測値 $x_1, x_2, ..., x_n$ の分散 $\cfrac{\Sigma(x_{i} - \bar{x})^2}{n}$ |
| $s_{xy}$ |`s_{xy}` | Covariance | $x$ と $y$ の共分散 $\cfrac{\Sigma(x_i - \bar{x})(y_i - \bar{y})}{n}$
| $\bar{x}$ | `\bar{x}` | Mean, Average | 観測値 $x_1, x_2, ..., x_n$ の平均値、算術平均、読みは「エックスバー」 |
| $z$ | `z` | - | $z$ 値、$z$ スコア、観測値を標準化（基準化）した値 |
| $\alpha, \beta$ | `\alpha, \beta` | Regression Coefficient | 回帰直線の回帰係数 |
| $\hat{\alpha}, \hat{\beta}$ | `\hat{\alpha}, \hat{\beta}` | Estimate Value | 回帰係数の推定値、読みは「アルファヘット」 |
| $A^{c}, \overline{A}$ | `A^{c}, \overline{A}` | Complement | 事象 $A$ の余事象、他に $A^C$ など |
| $A \cup B$ | `A \cup B` | Union of Events | 事象 $A$ と $B$ の和事象 |
| $A \cap B$ | `A \cap B` | Intersection of Events | 事象 $A$ と $B$ の積事象、他に $AB$ など |
| $B(n, p)$ | `B(n, p)` | Binomial Distribution | 試行回数 $n$、成功確率 $p$ の二項分布 |
| $E(X), \mu$ | `E(X), \mu` | Expectation Value | 確率変数 $X$ の期待値、平均 |
| $H_0, H_1$ | `H_0, H_1` | Null Hypothesis, Alternative Hypothesis | 帰無仮説、対立仮設 |
| $N(\mu, \sigma^2)$ | `N(\mu, \sigma^2)` | Nomal Distribution, Ganss Distribution | 平均 $\mu$、分散 $\sigma^2$ の正規分布 |
| $N(0, 1)$ | `N(0, 1)` | Standard Nomal Distribution | 平均 $0$ 、分散 $1$ の標準正規分布 |
| $P(A), Pr(A)$ | `P(A), Pr(A)` | Probability | 事象 $A$ の確率 |
| $P(A\|B)$ | `P(A\|B)` | Conditional Proberbility | 事象 $B$ を与えた下での事象 $A$ の条件付確率 |
| $\hat{p}$ | `\hat{p}` | Sample Propotion, percentage, ratio | 標本比率 |
| $U$ | `U` | Universal Set | 全事象、全集合、他に $\Omega, S$ など |
| $V(X), \sigma^2$ | `V(X), \sigma^2` | Variance | 確率変数 $X$ の分散、他に $var(X)$ など |
| $\overline{X}$ | `\overline{X}` | Sample Mean, Sample Average | 確率変数 $X_1, X_2, ..., X_n$ の標本平均 |
| $Z$ | `Z` | Random Variable | 標準正規分布 $N(0, 1)$ に従う確率変数 |
| $\alpha$ | `\alpha` | Level of Significance, Significance Level | 有意水準 |
| $\alpha(X), \sigma$ | `\alpha(X), \sigma` | - | 確率変数 $X$ の標準偏差 |
| $\phi, \emptyset, \varnothing$ | `\phi, \emptyset, \varnothing` | Impossible Event, Empty Event, Null Event, Void Event | 空事象 |
