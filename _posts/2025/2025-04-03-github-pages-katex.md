---
title: "GitHub Pages に KaTeX で数式を表示させる方法（Jekyll の Minimal Mistakes 導入）"
date: 2025-04-03T12:00:00+09:00
categories:
  - Web
tags:
  - GitHub Pages
  - Jekyll
  - KaTeX
latex: true
---

GitHub Pages に Jekyll の Minimal Mistakes テーマを導入し，KaTeX で数式を表示させる方法についてのメモ

## はじめに

Jekyll の [Minimal Mistakes remote theme starter](https://github.com/mmistakes/mm-github-pages-starter) のテンプレートを用いると，記事を Markdown で書いて [GitHub Pages](https://docs.github.com/ja/pages) で手軽に公開できる．そのままでは記事に $\LaTeX$ の数式を表示させることができないので，$\KaTeX$ を用いて数式を表示させる．

## 手順

[Minimal Mistakes remote theme starter](https://github.com/mmistakes/mm-github-pages-starter) で作成したリポジトリにおいて，`_data/` や `_pages/`，`_posts/` と同じ階層に `_includes/head/custom.html` を新たに作成し，以下を入力する．

```html
{% if page.katex %}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.css" integrity="sha384-Xi8rHCmBmhbuyyhbI88391ZKP2dmfnOl4rT9ZfRI7mLTdk1wblIUnrIq35nqwEvC" crossorigin="anonymous">
<!-- The loading of KaTeX is deferred to speed up page rendering -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.js" integrity="sha384-X/XCfMm41VSsqRNQgDerQczD69XqmjOOOwYQvr/uuC+j4OPoNhVgjdGFwhvN02Ja" crossorigin="anonymous"></script>
<!-- To automatically render math in text elements, include the auto-render extension: -->
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

注）Minimal Mistakes 以外の他のテーマだと，最初から `_includes/head-custom.html` というファイルが存在している場合があり，その場合は `head-custom.html` に上を追記すればよい．

あとは適当な記事において，Markdown 中に

```markdown
$$
i\hbar\frac{d\Psi(\boldsymbol{r},\ t)}{dt}=H\Psi(\boldsymbol{r},\ t)
$$
```
のように入力すると

$$
i\hbar\frac{d\Psi(\boldsymbol{r},\ t)}{dt}=H\Psi(\boldsymbol{r},\ t)
$$

のように数式が表示される．

## 数式があるページにだけ KaTeX を読み込む

数式のないページに KaTeX を読み込んでも，読込速度が落ちるだけで意味がないので，数式があるページだけに読み込むようにするために，数式を表示させたい Markdown ファイルの front matter で以下のようにページ変数 `katex: true` を設定する．

```markdown
---
title: "タイトル"
data: 2025-04-03T12:00:00+09:00
...
katex: true
---
```

次に，先ほど作成した `_includes/head-custom.html` において，以下のように追記して `page.katex == true` のときだけ KaTeX を読み込むようにする．

```html
{% if page.katex %}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.css" integrity="sha384-Xi8rHCmBmhbuyyhbI88391ZKP2dmfnOl4rT9ZfRI7mLTdk1wblIUnrIq35nqwEvC" crossorigin="anonymous">
<!-- The loading of KaTeX is deferred to speed up page rendering -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.0/dist/katex.min.js" integrity="sha384-X/XCfMm41VSsqRNQgDerQczD69XqmjOOOwYQvr/uuC+j4OPoNhVgjdGFwhvN02Ja" crossorigin="anonymous"></script>
<!-- To automatically render math in text elements, include the auto-render extension: -->
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

注）{% if ... %}の詳しい使い方は [Liquid](https://shopify.github.io/liquid/) を参照．
