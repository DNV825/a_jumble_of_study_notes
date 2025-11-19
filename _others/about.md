---
title: About others
author: DNV825
date: 2025-06-14
category: Jekyll
layout: post
---

## ToDo

- [ ] front matter の date でソートする仕組みになっているが、 \_rust や \_powershell ディレクトリのファイルは、例えば order でソートするようにしたい。

## Azure Static Web Apps の実験

<https://proud-bay-0f2cc7400.2.azurestaticapps.net/>

## MathJax 関係のメモ

### 数式が表示されなくなったら

MathJax の数式を右クリックすると表示モードを変更できるのだが、この時に原文表示にすると数式表示に戻せなくなる。

その場合、開発者ツールで以下のコマンドを実行し、設定をリセットすれば数式を表示できるようになる。

```javascript
localStorage.removeItem('MathJax-Menu-Settings');
location.reload();
```

### MathJax の display mode の中央揃え設定を打ち消す

以下の文を markdown に書き込めばよい。

```html
<style>
mjx-container[display="true"] {
    justify-content: flex-start !important;
    text-align: left !important;
}
</style>
```

## TikzJax 関係のメモ

### TikZJax について

JavaScript で tikz を利用可能にするもの。すごい。参考：<https://tikzjax.com/>

以下のようにインクルードし、

```html
<link rel="stylesheet" type="text/css" href="https://tikzjax.com/v1/fonts.css">
<script src="https://tikzjax.com/v1/tikzjax.js"></script>
```

`<script type="text/tikz"></script>` タグの中に tikz を記述する。`html {cmd=true hide=true}` を指定しても、Markdown Preview Enhanced では実行してくれないようだ。

通常の TikzJax は % から始まるコメントが使えないが、[bill-ion さんの作成した fork](https://github.com/bill-ion/tikzjax-live/) ならコメントが使える。通常の TikzJax は 2 バイト文字の表示は非対応。テキストは英語を使うのが無難。あと、もともとの TikzJax は一部の文字がおかしな表示になる（例えば "\|" が "♣" になる。）この問題は bill-on さんバージョンだと解決されている。

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

### TikzJax Live

bill-ion さん作成の TikzJax fork 。このソースコードをお借りして、 Tikz を描画できるか試している。

- <https://bill-ion.github.io/tikzjax-live/>
- <https://github.com/bill-ion/tikzjax-live/>

ローカルで動作確認する際は python のワンライナーサーバーを使うのが簡単。

```python
python -m http.server
```

### Tikz Online Editor

$\LaTeX$ をインストールせずに Tikz の表示内容を確認できる Web サイト。

- <https://pickedshares.com/en/tikz-online-editor/>
