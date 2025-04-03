---
title: "Katex sample page"
date: 2025-04-03T03:20:00+09:00
categories:
  - blog
tags:
  - Jekyll
  - katex
latex: true
---

数式を表示したいページだけ KaTeX を読み込むサンプルページ

オイラー角 $\alpha$, $\beta$, $\gamma$ についての回転行列 $R_z(\alpha)$, $R_x(\beta)$, $R_y(\gamma)$

$$
\tag{1}
R_z(\alpha)=\begin{pmatrix}
\cos\alpha & -\sin\alpha & 0 \\
\sin\alpha & \cos\alpha & 0 \\
0 & 0 & 1
\end{pmatrix}
$$

$$
\tag{2}
R_x(\beta)=\begin{pmatrix}
1 & 0 & 0 \\
0 & \cos\beta & -\sin\beta \\
0 & \sin\beta & \cos\beta    
\end{pmatrix}
$$

$$
\tag{3}
R_y(\gamma)=\begin{pmatrix}
\cos\gamma & 0 & \sin\gamma \\
0 & 1 & 0 \\
-\sin\gamma & 0 & \cos\gamma \\
\end{pmatrix}
$$

ここで，静止座標系で成分表示されたベクトル $ \boldsymbol{a}_0=\left(a_{0x},\,a_{0y},\,a_{0z}\right) $ から，$z$ - $x$ - $y$ 系のオイラー角により回転変換された座標系での成分表示 $ \boldsymbol{a}=\left(a_{x},\,a_{y},\,a_{z}\right) $ を得ること（座標変換）を考える．この場合，$ \boldsymbol{a}_0 $ を $z$ 軸周りに角 $\alpha$ だけ逆回転させ，次に $x$ 軸周りに角 $\beta$ だけ逆回転，さらに $y$ 軸周りに角 $\gamma$ だけ逆回転させればよいので

$$
\tag{4}
\begin{aligned}
\boldsymbol{a}&=R_y^{-1}(\gamma)R_x^{-1}(\beta)R_z^{-1}(\alpha)\boldsymbol{a}_0 \\
&=\left\lbrace R_z(\alpha)R_x(\beta)R_y(\gamma)\right\rbrace^{-1}\boldsymbol{a}_0
\end{aligned}
$$
