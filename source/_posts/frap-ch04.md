---
title: "【FRAP】第四章：使用解释器生成语义"
date: 2022-08-26 01:00:00
tags:
- FRAP
- 形式化验证
- 读书笔记
categories:
- FRAP
---

关于程序应当 *长什么样* 的内容就到此为止。让我们把注意力转移到程序的 *含义* 上来。

<!-- more -->

## 4.1 使用有限映射表示算数表达式的语义

要阐述第二章中定义的算数表达式的语义，我们需要一种方法表示每个变量的值。*有限映射* 在这里十分有用。在本书中，我们将使用以下符号：

$$
\newcommand{\mempty}[0]{\bullet}
\newcommand{\msel}[2]{#1(#2)}
\newcommand{\mupd}[3]{#1[#2 \mapsto #3]}

\begin{aligned}
  \mempty \quad & 空映射，其定义域为 \emptyset \\
  \msel{m}{k} \quad & 键 k 在映射 m 中对应的值 \\
  \mupd{m}{k}{v} \quad & 扩展映射 m，将键 k 映射至值 v
\end{aligned}
$$

顾名思义，有限映射是具有有限定义域的函数，我们可以通过扩展操作扩大映射的定义域。下面的两个公理阐述了这些基本操作在共同作用时的性质。

$$
\newcommand{\mempty}[0]{\bullet}
\newcommand{\msel}[2]{#1(#2)}
\newcommand{\mupd}[3]{#1[#2 \mapsto #3]}

\frac{}{\msel{\mupd{m}{k}{v}}{k} = v}
\quad
\frac{
  k_1 \neq k_2
}{\msel{\mupd{m}{k_1}{v}}{k_2} = m(k_2)}
$$

有了上述这些运算，我们就能写出算数表达式的语义，它是一个 *将变量的值映射到数字上* 的递归函数。这里，我们使用 $\llbracket e \rrbracket$ 表示 $e$ 的含义，这种记号通常又被称作 *牛津括号*。回顾一下，我们允许使用这种符号作为任意函数的句法糖，即使是在定义这些函数的等式内部也是如此。此外，我们使用 $v$ 表示求值（有限映射）。

$$
\newcommand{\denote}[1]{{\llbracket #1 \rrbracket}}

\begin{aligned}
  \denote{n}v &= n \\
  \denote{x}v &= v(x) \\
  \denote{e_1 + e_2}v &= \denote{e_1}v + \denote{e_2}v \\
  \denote{e_1 \times e_2}v &= \denote{e_1}v \times \denote{e_2}v
\end{aligned}
$$

这一定义中的部分内容看上去似乎仅仅是“将记号移入了括号内”。但请注意，括号 *内* 的加号是语法的一部分，而括号 *外* 的加号表示数学中的加法运算！

为了测试我们所定义的语义，我们可以定义一个 *变量代换* 函数。$[e'/x]e$ 表示遍历表达式 $e$ 并将其中所有的变量 $x$ 替换为表达式 $e'$ 后所得到的结果。

$$
\newcommand{\subst}[3]{[#3/#2]#1}

\begin{aligned}
  \subst{n}{x}{e} &= n \\
  \subst{x}{x}{e} &= e \\
  \subst{y}{x}{e} &= y ，若 y \neq x \\
  \subst{(e_1 + e_2)}{x}{e} &= \subst{e_1}{x}{e} + \subst{e_2}{x}{e} \\
  \subst{(e_1 \times e_2)}{x}{e} &= \subst{e_1}{x}{e} \times \subst{e_2}{x}{e}
\end{aligned}
$$

We can prove a key compatibility property of these two recursive functions.

> **定理 4.1**
> 
> For all $e$, $e'$, $x$, and $v$, $\denote{\subst{e}{x}{e'}}{v} = \denote{e}{(\mupd{v}{x}{\denote{e'}{v}})}$.