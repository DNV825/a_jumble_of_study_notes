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
