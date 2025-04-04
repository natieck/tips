---
title: "GitHub Pages に Jekyll の Minimal Mistakes テーマを導入し KaTeX で数式を表示させる方法"
date: 2025-04-03T12:00:00+09:00
last_modified_at: 2025-04-04T01:30:30+09:00
categories:
  - Web
tags:
  - GitHub Pages
  - Jekyll
  - KaTeX
katex: true
---

GitHub Pages に Jekyll の Minimal Mistakes テーマを導入し，KaTeX で数式を表示させる方法についてのメモ

## はじめに

[GitHub Pages](https://docs.github.com/ja/pages) に Jekyll の [Minimal Mistakes remote theme starter](https://github.com/mmistakes/mm-github-pages-starter) テンプレートを導入すると，ローカル環境で Jekyll ビルドすることなく，記事を Markdown で書いて [GitHub Pages](https://docs.github.com/ja/pages) で手軽にWeb上に公開できる．ただ，そのままでは記事に $\LaTeX$ の数式を表示させることができないので，数式表示のJavaScriptライブラリ $\KaTeX$ を用いて $\LaTeX$ の数式を表示できるように設定する．

## 手順

[Minimal Mistakes remote theme starter](https://github.com/mmistakes/mm-github-pages-starter) で作成したリポジトリにおいて，`_data/` や `_pages/`，`_posts/` と同じ階層に `_includes/head/custom.html` を新たに作成し，以下の KaTeX 導入設定（CDN）を入力して保存する．

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.css" integrity="sha384-Xi8rHCmBmhbuyyhbI88391ZKP2dmfnOl4rT9ZfRI7mLTdk1wblIUnrIq35nqwEvC" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.js" integrity="sha384-X/XCfMm41VSsqRNQgDerQczD69XqmjOOOwYQvr/uuC+j4OPoNhVgjdGFwhvN02Ja" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/contrib/auto-render.min.js" integrity="sha384-+XBljXPPiv+OzfbB3cVmLHf4hdUFHlWNZN5spNQ7rmHTXpd7WvJum6fIACpNNfIR" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>
<script>
  document.addEventListener("DOMContentLoaded", function() {
    renderMathInElement(document.body, {
      delimiters: [
        {left: '$$', right: '$$', display: true},
        {left: '$', right: '$', display: false},
        {left: '\\(', right: '\\)', display: false},
        {left: '\\[', right: '\\]', display: true}
      ],
      throwOnError : false
    });
  });
</script>
```

注）Minimal Mistakes 以外の他のテーマだと，最初から `_includes/head-custom.html` というファイルが存在している場合があり，その場合は `head-custom.html` に上記の KaTeX 導入設定を追記すればよい．

あとは適当な Markdown 中の数式を表示させたい箇所に

```
$$
i\hbar\frac{d\Psi(\boldsymbol{r},\ t)}{dt}=\hat{H}\Psi(\boldsymbol{r},\ t)
$$
```
のように入力すると

$$
i\hbar\frac{d\Psi(\boldsymbol{r},\ t)}{dt}=\hat{H}\Psi(\boldsymbol{r},\ t)
$$

のように数式が表示される．

## 数式があるページにだけ KaTeX を読み込む

数式のないページに KaTeX を読み込んでも，読込速度が落ちるだけで意味がない．数式があるページだけに読み込むようにするために，数式を表示させたい Markdown ファイルの front matter で以下のようにページ変数 `katex: true` を設定する（[参考ページ](https://tex2e.github.io/blog/latex/mathjax-to-katex)）

```markdown
---
title: "タイトル"
data: 2025-04-03T12:00:00+09:00
...
katex: true
---
```

次に，先ほど作成した `_includes/head/custom.html` において，以下のように {% raw %}{% if ... %}{% endif %}{% endraw %} で囲って `page.katex == true` のときだけ KaTeX を読み込むようにする．

{% raw %}
```html
{% if page.katex %}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.css" integrity="sha384-Xi8rHCmBmhbuyyhbI88391ZKP2dmfnOl4rT9ZfRI7mLTdk1wblIUnrIq35nqwEvC" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.js" integrity="sha384-X/XCfMm41VSsqRNQgDerQczD69XqmjOOOwYQvr/uuC+j4OPoNhVgjdGFwhvN02Ja" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/contrib/auto-render.min.js" integrity="sha384-+XBljXPPiv+OzfbB3cVmLHf4hdUFHlWNZN5spNQ7rmHTXpd7WvJum6fIACpNNfIR" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>
<script>
  document.addEventListener("DOMContentLoaded", function() {
    renderMathInElement(document.body, {
      delimiters: [
        {left: '$$', right: '$$', display: true},
        {left: '$', right: '$', display: false},
        {left: '\\(', right: '\\)', display: false},
        {left: '\\[', right: '\\]', display: true}
      ],
      throwOnError : false
    });
  });
</script>
{% endif %}
```
{% endraw %}
