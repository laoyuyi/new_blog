- 基本语法（以下的markdown为common markdown）

|内容|语法|备注|
|:---:|:---:|:---:|
|标题|`# 一级标题 ## 二级标题 ### 三级标题`|最多6级标题|
|斜体|`*斜体*`||
|粗体|`**粗体**`||
|粗斜体|`***粗斜体***`||
|分割线|`***或---`||
|删除线|`~~删除线~~`||
|无序列表|`- 列表 + 列表 * 列表`|缩进嵌套|
|有序列表|`1. 列表 2. 列表 3. 列表`|缩进嵌套|
|引用|`> 文字`|多个\>层级更深|
|链接|`[链接名称](链接地址)`||
|行内公式|`$mathjax$`||
|行间公式|`$$mathjax$$`||

- Callout语法（obsidian特有的语法）

> [!note]

> [!abstract] abstract, summary, tldr

> [!info]

> [!todo]

> [!tip] tip, hint, important

> [!success] success, check, done

> [!question] question, help, faq

> [!warning] warning, caution, attention

> [!failure] failure, fail, missing

> [!danger] danger, error

> [!bug]

> [!example]

> [!quote] quote, cite

- 各种上下左右标：

$$\underset{e}{\overset{f}{_a^bM_c^d}}$$

```latex
\underset{e}{\overset{f}{_a^bM_c^d}}
```

- 级数：

$$\sum_{i=1}^{n}f(x)$$

```latex
\sum_{i=1}^{n}f(x)
```

- 极限：

$$\lim_{x \to 0}f(x)=A$$

```latex
\lim_{x \to 0}f(x)=A
```

- 偏导数：

$$\frac{\partial f(x)}{\partial x}$$

```latex
\frac{\partial f(x)}{\partial x}
```

- 矩阵：

$$
\begin{cases}
  &这是第一行\\\\
  &这是第二行
	\begin{cases}
		可以多层嵌套\\\\
		这里是嵌套第二行
	\end{cases}
\end{cases}
$$

- 希腊字母：

|小写希腊字母|语法|大写希腊字母|语法|
|:--------:|:-------:|:--------:|:-------:|
|α        | `\alpha` |||
|β        | `\beta`  |||
|γ        | `\gamma` |Γ        | `\Gamma` |
|δ        | `\delta` |Δ        | `\Delta` |
|ε        | `\epsilon`|Ε        | `E`     |
|ζ        | `\zeta`  |Ζ        | `Z`      |
|η        | `\eta`   |Η        | `H`      |
|θ        | `\theta` |Θ        | `\Theta` |
|ι        | `\iota`  |Ι        | `I`      |
|κ        | `\kappa` |Κ        | `\Kappa` |
|λ        | `\lambda` |Λ        | `\Lambda`|
|μ        | `\mu`    |Μ        | `\Mu`    |
|ν        | `\nu`    |Ν        | `\Nu`    |
|ξ        | `\xi`    |Ξ        | `\Xi`    |
|ο        | `o`      |Ο        | `O`      |
|π        | `\pi`    |Π        | `\Pi`    |
|ρ        | `\rho`   |Ρ        | `\Rho`   |
|σ        | `\sigma` |Σ        | `\Sigma` |
|τ        | `\tau`   |Τ        | `\Tau`   |
|υ        | `\upsilon`|Υ        | `\Upsilon`|
|φ        | `\phi`   |Φ        | `\Phi`   |
|χ        | `\chi`   |Χ        | `\Chi`   |     
|ψ        | `\psi`   |Ψ        | `\Psi`   |
|ω        | `\omega` |Ω        | `\Omega` |