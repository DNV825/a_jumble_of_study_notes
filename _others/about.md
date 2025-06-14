---
title: About others
author: DNV825
date: 2025-06-14
category: Jekyll
layout: post
---

This is an about page for "others" in the collections.

## 数式が表示されなくなったら

MathJax の数式を右クリックすると表示モードを変更できるのだが、この時に原文表示にすると数式表示に戻せなくなる。

その場合、開発者ツールで以下のコマンドを実行し、設定をリセットすれば数式を表示できるようになる。

```javascript
localStorage.removeItem('MathJax-Menu-Settings');
location.reload();
```
