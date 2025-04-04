---
title: "JavaScript で Web ページに出力した KaTeX の数式がレンダリングされないときの対処法"
date: 2025-04-04T09:30:00+09:00
categories:
  - Web
tags:
  - KaTeX
  - JavaScript
katex: true
toc: false
---

ページ読み込み (onload) 後に JavaScript で Web ページに出力した KaTeX の数式がレンダリングされないときの対処法についてのメモ

## はじめに

数式表示のJavaScriptライブラリ $\KaTeX$ を用いると $\LaTeX$ の数式を Web ページで表示できるようになるが，一旦，ページを読み込んだ後に，JavaScript で HTML 要素の innerHTML を書き換えて数式を出力しただけでは，その数式はレンダリングされない．数式として表示させるためには，innerHTML を書き換えたタイミングで KaTeX の数式をレンダリングし直す必要がある．

## 対処法

Web ページにおいて KaTeX で数式を表示させるには，そのページの `<head></head>` 内に以下の設定（CDN）を記述する．

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.css" integrity="sha384-Xi8rHCmBmhbuyyhbI88391ZKP2dmfnOl4rT9ZfRI7mLTdk1wblIUnrIq35nqwEvC" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.js" integrity="sha384-X/XCfMm41VSsqRNQgDerQczD69XqmjOOOwYQvr/uuC+j4OPoNhVgjdGFwhvN02Ja" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/contrib/auto-render.min.js" integrity="sha384-+XBljXPPiv+OzfbB3cVmLHf4hdUFHlWNZN5spNQ7rmHTXpd7WvJum6fIACpNNfIR" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>
```

上記の3行目で KaTeX 拡張機能 Auto-render を読み込み，ページのすべてのリソースの読み込みが完了した時点で renderMathInElement(document.body) が実行され，ページ内の数式がレンダリングされる．

その後に，なんらかのタイミングで JavaScript によって，以下のように，ある HTML 要素の中身を書き換えても数式として表示させたい箇所 `$ax^2+bx+c=0$` はそのまま表示される．

```javascript
document.getElementById('要素のID').innerHTML = "2次方程式 $ax^2+bx+c=0$";
```

数式として表示させるためには，innerHTML を書き換えたタイミングで再度，renderMathInElement を実行して，数式をレンダリングし直せばよい．ただし，renderMathInElement(document.body) を実行すると，body 要素内のすべての数式をレンダリングするので，一部だけ書き換えたとしても body 要素内の数式が多いと時間がかかる．一部だけ書き換えた場合は

```javascript
element = document.getElementById('要素のID');
element.innerHTML = "2次方程式 $ax^2+bx+c=0$";
renderMathInElement(element);
```

とすればよい．もしくは，katex.render() や katex.renderToString() を用いる方法もある（[KaTeX API](https://katex.org/docs/api)）．
