---
title: "【FRAP】第二章：程序语法的形式化"
date: 2022-08-20 01:00:00
tags:
- FRAP
- 形式化验证
- 读书笔记
categories:
- FRAP
---

程序的定义始于编程语言的定义，而编程语言的定义又始于其 *语法（syntax）* 的定义。语法规定了哪些短语是符合其规定的。我们会在后面的章节中再讨论 *语义（semantics）*，也就是程序的 *含义*，在这一过程中我们将应用更多的正确性条件。

<!-- more -->

## 2.1 具体语法

让我们转到具体的例子上，首先值得关注的是 *具体语法*，它规定了哪些字符序列是可以被接受的。对于下面这个简单的语言——算数表达式，以下的字符串都是合法的：

$$
\begin{array}{l}
  3 \\
  x \\
  3 + x \\
  y * (3 + x)
\end{array}
$$

而其它许多字符串均是不合法的，例如：

$$
\begin{array}{l}
  1 + + \; 2 \\
  x \; y \; z
\end{array}
$$

与其求助于我们对小学算术的直觉印象，我们更倾向于使用 *文法（grammar）* 对这一具体语法进行形式化定义。*巴科斯——诺尔范式（BNF）* 是一种描述文法的形式，我们使用一组 *非终结符*（如下文中的 $e$）表示可被接受的串的集合。其中一些定义会使用已有的集合，例如在下面这个例子当中，我们使用众所周知的自然数（非负数）集合 $\mathbb{N}$ 定义常数 $n$。

$$
\begin{aligned}
  \textrm{常量} \quad & n \in \mathbb N \\
  \textrm{变量} \quad & x \in \mathsf{Strings} \\
  \textrm{表达式} \quad & e ::= n \mid x \mid e + e \mid e \times e
\end{aligned}
$$

用自然语言来描述这个语法：我们首先基于众所周知的自然数和字符串的集合分别定义了常量和变量的集合，随后定义了表达式，包括常量、变量、加法和乘法。重点在于，最后两种情形是 *递归* 定义的：从这个例子中可以看到如何用较小的表达式构建更大的表达式。

顺带一提，我们已经在形式化证明的讨论中见到了各种各样的记号。所有这些内容都是使用 $\LaTeX$ 进行排版的，有兴趣的读者可以翻阅本书的TeX源代码，了解这些符号是如何实现的。

*归纳定义（inductive definition）* 是贯穿整个主题最重要的工具之一，它描述了如何在较小的集合的基础上构建更大的集合。上文的文法的递归性质实际上隐式地给出了一个归纳定义。归纳定义的更一般的记法是通过一系列的 *推断规则（inference rule）* 定义一个集合——准确地说，是 *能够满足所有规则* 的最小集合。每个规则都包括 *前提（premise）* 和 *结论（conclusion）*。我们用下面的四条规则来演示，这四条规则等价于上文的BNF文法，共同定义了一个表达式的集合 $\mathsf{Exp}$。

$$
\frac{n \in \mathbb N}{n \in \mathsf{Exp}}
\quad 
\frac{x \in \mathsf{Strings}}{x \in \mathsf{Exp}}
\quad 
\frac{
  e_1 \in \mathsf{Exp}
  \quad e_2 \in \mathsf{Exp}
}{e_1 + e_2 \in \mathsf{Exp}}
\quad 
\frac{
  e_1 \in \mathsf{Exp}
  \quad e_2 \in \mathsf{Exp}
}{e_1 \times e_2 \in \mathsf{Exp}}
$$

推断规则的一般读法是：**如果** 横线上方的所有事实都为真，那么横线下方的事实也为真。推断规则默认需要对其中出现的所有 *元变量（metavariable）*（如 $n$和$e1$）的所有值都成立；我们可以用一种更为通用的自顶向下的量化方法来更明确地表示这些变量。初次接触语义的读者在看到这种定义风格时往往不太适应，但大家很快就会发现这是一个能够表达许多概念的十分紧凑的记号。读者可以将其视作一种用于数学定义的特定领域编程语言，这种类比在相关的Coq代码中会变得非常具体。

## 2.2 抽象语法

在简单介绍了具体语法之后，本书余下的部分将不再对其进行形式化的讨论。相反，我们转向关注语言定义的真正核心内容，*抽象语法（abstract syntax）*。从现在开始，程序变成了 *抽象语法树*（*abstract syntax trees，AST*），对应于Coq中的归纳类型定义或者Haskell中的代数数据类型定义。我们可以通过枚举带有类型的 *构造函数（constructor）* 来定义这些类型。

$$
\begin{aligned}
  \mathsf{Const} &: \mathbb{N} \to \mathsf{Exp} \\
  \mathsf{Var} &: \mathsf{Strings} \to \mathsf{Exp} \\
  \mathsf{Plus} &: \mathsf{Exp} \times \mathsf{Exp} \to \mathsf{Exp} \\
  \mathsf{Times} &: \mathsf{Exp} \times \mathsf{Exp} \to \mathsf{Exp}
\end{aligned}
$$

请注意这里的“$\times$”并非具体语法中的乘法符号，而是集合论中的笛卡尔积运算符，表示一个类型的对组。

这些构造函数共同定义了集合 $\mathsf{Exp}$，这个集合包含所有能够通过这些构造函数生成的元素。使用推断规则记号书写如下：

$$
\frac{
  n \in \mathbb N
}{\mathsf{Const}(n) \in \mathsf{Exp}}
\quad 
\frac{
  x \in \mathsf{Strings}
}{\mathsf{Var}(x) \in \mathsf{Exp}}
\quad 
\frac{
  e_1 \in \mathsf{Exp}
  \quad e_2 \in \mathsf{Exp}
}{\mathsf{Plus}(e_1, e_2) \in \mathsf{Exp}}
\quad 
\frac{
  e_1 \in \mathsf{Exp}
  \quad e_2 \in \mathsf{Exp}
}{\mathsf{Times}(e_1, e_2) \in \mathsf{Exp}}
$$

实际上，书写这种冗长的描述让语义学家们不胜其烦，所以文本中的证明往往更倾向于采用我们在书写具体语法时所使用的记法。这就需要我们在大脑中将具体语法的记法解构为抽象语法的符号！一般我们不会详述这一过程的具体细节。相反地，我们会同时使用以抽象语法为起点的Coq代码，以及本书中应用具体语法的基于 $\LaTeX$ “代码”，通过这些例子反复演示这一过程。

抽象语法十分适用于书写函数的 *递归定义（recursive definition）*。下面是一个Haskell的子句式的函数定义：

$$
\begin{aligned}
  \mathsf{size}(\mathsf{Const}(n)) &= 1 \\
  \mathsf{size}(\mathsf{Var}(x)) &= 1 \\
  \mathsf{size}(\mathsf{Plus}(e_1, e_2)) &= 1 + \mathsf{size}(e_1) + \mathsf{size}(e_2) \\
  \mathsf{size}(\mathsf{Times}(e_1, e_2)) &= 1 + \mathsf{size}(e_1) + \mathsf{size}(e_2)
\end{aligned}
$$

请注意 *对于归纳类型的每个构造函数都应当有一个对应的子句*，否则函数是不 *完备（total）* 的。同时需要注意保证函数能够 *终止*，即让递归调用只作用于构造函数的参数。这一终止判定准则被称为 *原始递归*，也是Coq所采用的判定准则。

为递归函数定义一个新的记号也是一种常见的做法。例如，我们可以使用 $\lvert e \rvert$ 表示 $\mathsf{size}(e)$ 如下：

$$
\newcommand{\size}[1]{{\lvert #1 \rvert}}

\begin{aligned}
  \size{\mathsf{Const}(n)} &= 1 \\
  \size{\mathsf{Var}(x)} &= 1 \\
  \size{\mathsf{Plus}(e_1, e_2)} &= 1 + \size{e_1} + \size{e_2} \\
  \size{\mathsf{Times}(e_1, e_2)} &= 1 + \size{e_1} + \size{e_2}
\end{aligned}
$$

让我们继续行使我们的创造许可证，使用 $\lceil e \rceil$ 表示 $e$ 的 *深度（depth）*，即在语法树中从根节点到任意叶子节点的向下路径长度的最大值。

$$
\newcommand{\depth}[1]{{\lceil #1 \rceil}}

\begin{aligned}
  \depth{\mathsf{Const}(n)} &= 1 \\
  \depth{\mathsf{Var}(x)} &= 1 \\
  \depth{\mathsf{Plus}(e_1, e_2)} &= 1 + \max(\depth{e_1}, \depth{e_2}) \\
  \depth{\mathsf{Times}(e_1, e_2)} &= 1 + \max(\depth{e_1}, \depth{e_2})
\end{aligned}
$$

## 2.3 结构化归纳法则

我们更偏好抽象语法的主要原因是，虽然文本字符串 *看上去* 更自然和容易理解，但要对其进行完全形式化的处理时会遇到非常多的困难。相对地，归纳语法树较为容易操作。顾名思义，我们主要想做的操作就是 *归纳（induction）*，这个操作最常出现的场景可能是应用于自然数的 *数学归纳法（mathematical induction）*。在本书中，我们不会过多涉及和自然数有关的证明，而是将自然数视为一个简单的归纳定义的集合，在这个基础上，介绍更具一般性、更强大的 *结构化归纳（structural induction）* 思想，它在形式化的意义上包含了数学归纳法。

我们可以通过一个具有一般性的方法从归纳定义中得出其归纳法则。当我们归纳地定义集合 $S$ 时，我们同时也得到了一个归纳法则，它可以用于证明某个断言 $P$ 对 $S$ 中的所有元素都成立。为了得出这个结论，我们必须对归纳定义的每一条规则履行一次证明义务。回顾上文中我们为 $\mathsf{Exp}$ 的抽象语法所做的基于规则的归纳定义，为了推导出 $\mathsf{Exp}$ 的结构化归纳法则，我们定义一套新的规则，它在原有规则的基础上做了两个关键的修改：

1. 将所有形式满足 $E \in S$ 的结论 $E$ 替换为结论 $P(E)$。也就是说，证明义务要求 *证明* $P$ 对特定的项能够成立。
2. 对每个形式满足 $E \in S$ 的前提 $E$，添加一个伴随的前提 $P(E)。也就是说，证明义务允许 *假定* $P$ 对特定的项能够成立。这里的每个假定被称作一个 *归纳假设（inductive hypothesis，IH）*。

对上文的抽象语法执行这一机械的流程将产生下面四个证明义务，以及相关的归纳证明 $\forall x \in \mathsf{Exp}. \; P(x)$。

$$
\frac{n \in \mathbb N}{P(\mathsf{Const}(n))}
\quad 
\frac{x \in \mathsf{Strings}}{P(\mathsf{Var}(x))}
$$

$$
\frac{
  e_1 \in \mathsf{Exp}
  \quad P(e_1)
  \quad e_2 \in \mathsf{Exp}
  \quad P(e_2)
}{P(\mathsf{Plus}(e_1, e_2))}
\quad 
\frac{
  e_1 \in \mathsf{Exp}
  \quad P(e_1)
  \quad e_2 \in \mathsf{Exp}
  \quad P(e_2)
}{P(\mathsf{Times}(e_1, e_2))}
$$

换句话说，为了证明 $\forall x \in \mathsf{Exp}. \; P(x)$，我们必须证明上述每个推断规则均成立。

为了展示归纳法的作用，我们证明下文中的定理，对上文的的两个递归定义进行合理性检查：depth 不能超过 size。

> **定理2.1.** 对任意 $e \in \mathsf{Exp}$, $\newcommand{\size}[1]{{\lvert #1 \rvert}} \newcommand{\depth}[1]{{\lceil #1 \rceil}} \depth{e} \leq \size{e}$.
> 
> **证明** 对 $e$ 的结构进行归纳可得定理成立。

像这样极度简单的证明往往让初次接触的读者感到惊讶和受挫。我们的观点是，证明检查是一项更适合机器而非人的活动，所以对于上文中以及与本章相关的其它许多定理，我们倾向于在书中省略这些令人厌烦的证明细节，而将它们放在随附的Coq代码中。事实上，即使是发表在论文当中的证明，也倾向于使用像上文一样的简短“证明”，而依赖于读者的经验进行“填空”。因此，在这种论证中经常性地出现逻辑错误，从而导致人们接受了伪定理，也就不足为奇了。出于这个原因，我们在此坚持使用机器检查的证明，书中的章节主要用于介绍概念、推理法则以及陈述关键的定理和引理。

## 2.4 可判定定理

