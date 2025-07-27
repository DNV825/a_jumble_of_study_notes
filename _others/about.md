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

## 数式が表示されなくなったら

MathJax の数式を右クリックすると表示モードを変更できるのだが、この時に原文表示にすると数式表示に戻せなくなる。

その場合、開発者ツールで以下のコマンドを実行し、設定をリセットすれば数式を表示できるようになる。

```javascript
localStorage.removeItem('MathJax-Menu-Settings');
location.reload();
```

## Tikz Online Editor

$\LaTeX$ をインストールせずに Tikz の表示内容を確認できる Web サイト。

- <https://bill-ion.github.io/tikzjax-live/>
- <https://pickedshares.com/en/tikz-online-editor/>
