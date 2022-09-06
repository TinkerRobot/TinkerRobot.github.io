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
\newcommand{\size}[1]{\lvert #1 \rvert}
\begin{aligned}
  \size{\mathsf{Const}(n)} &= 1 \\
  \size{\mathsf{Var}(x)} &= 1 \\
  \size{\mathsf{Plus}(e_1, e_2)} &= 1 + \size{e_1} + \size{e_2} \\
  \size{\mathsf{Times}(e_1, e_2)} &= 1 + \size{e_1} + \size{e_2}
\end{aligned}
$$

让我们继续行使我们的创造许可证，使用 $\lceil e \rceil$ 表示 $e$ 的 *深度（depth）*，即在语法树中从根节点到任意叶子节点的向下路径长度的最大值。

$$
\newcommand{\depth}[1]{\lceil #1 \rceil}
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

> **定理2.1.** 对任意 $e \in \mathsf{Exp}$, $\newcommand{\size}[1]{\lvert #1 \rvert} \newcommand{\depth}[1]{\lceil #1 \rceil} \depth{e} \leq \size{e}$.
> 
> **证明** 对 $e$ 的结构进行归纳可得定理成立。

像这样极度简单的证明往往让初次接触的读者感到惊讶和受挫。我们的观点是，对证明进行检查是一项更适合机器而非人的活动，所以对于上文中以及与本章相关的其它许多定理，我们倾向于在书中省略这些令人厌烦的证明细节，而将它们放在随附的Coq代码中。事实上，即使是发表在论文当中的证明，也倾向于使用像上文一样的简短“证明”，而依赖于读者的经验进行“填空”。因此，在这种论证中经常性地出现逻辑错误，从而导致人们接受了伪定理，也就不足为奇了。出于这个原因，我们在此坚持使用机器检查的证明，书中的章节主要用于介绍概念、推理法则以及陈述关键的定理和引理。

## 2.4 可判定理论

然而，我们确实需要以某种方式填入所有的证明细节。对我们而言最简单的一种情形是，证明目标能够被归入某个 *可判定理论（decidable theory）*。我们遵循可计算性理论的理解，将一些判定问题视作公式的集合 $F$ （通常是无限集合）与为 *真* 的公式的子集 $T \subseteq F$，并且可能只考虑使用有限的推断规则能够证明的公式。此时，一个判定问题是可 *判定* 的，等价于存在这样一些必然终止的程序：其输入为 $f \in F$，当且仅当 $f \in T$ 时，输出为“true”。理论的可判定性的便利之处在于，只要我们的证明目标属于某个可判定理论的集合 $F$，就可以通过运行必然存在的判定程序来完成。

一个常见的可判定理论是 *线性运算（linear arithmetic）*，其集合 $F$ 可通过下面的文法 $\phi$ 生成：

$$
\begin{aligned}
  \textrm{常量} \quad & n \in \mathbb Z \\
  \textrm{变量} \quad & x \in \mathsf{Strings} \\
  \textrm{术语} \quad & e ::= x \mid n \mid e + e \mid e - e \\
  \textrm{命题} \quad & \phi ::= e = e \mid e < e \mid \neg \phi \mid \phi \land \phi
\end{aligned}
$$

这里所使用的运算是 *线性（linear）* 的，线性在这里的含义与在 *线性代数* 中的含义一致：即不会对两个含有变量的词语使用乘法。事实上，乘法是被完全禁止的，但我们允许使用与常数的乘法作为重复加法的缩写（从逻辑上而言）。命题是由作用于词语上的相等性测试和小于关系测试组成的，此外还有布尔取反（“not”）运算符 $\neg$ 和连接（“and”）运算符 $\land$。这一组命题运算符足以编码其它常见的不相等和命题运算符，因此我们也允许使用这些运算符作为便捷的缩写。

在Coq这类证明辅助工具中使用可判定理论的重点是理解怎样将一个理论应用到在字面上实际并不满足其语法的的公式中。例如，我们可能试图证明 $f(x) - f(x) = 0$，但一些复杂的函数 $f$ 已经远远超出了上文中语法的定义。然而，我们只需引入一个新的变量 $y$，并且定义等式 $y = f(x)$，即可得到新的证明目标 $y - y = 0$。此时一个线性运算的程序即可轻松地完成这个目标，之后再代入 $y$ 即可得出原本的目标。Coq的策略就是在可判定理论的基础上，替我们做了所有这些艰苦的工作。

另一个重要的可判定理论是关于 *未解释函数的相等性* 的。

$$
\begin{aligned}
  \textrm{变量} \quad & x \in \mathsf{Strings} \\
  \textrm{函数} \quad & f \in \mathsf{Strings} \\
  \textrm{术语} \quad & e ::= x \mid f(e, \ldots, e) \\
  \textrm{命题} \quad & \phi ::= e = e \mid \neg \phi \mid \phi \land \phi
\end{aligned}
$$

在这个理论中，我们对变量和函数的详细特性一无所知。相反地，我们只能从最基本的相等关系的性质开始推理：

$$
\mathsf{自反性} \frac{}{e = e}
\quad 
\mathsf{对称性} \frac{e_2 = e_1}{e_1 = e_2}
\quad 
\mathsf{传递性} \frac{e_1 = e_3 \quad e_3 = e_2}{e_1 = e_2}
$$

$$
\mathsf{全等性} \frac{
  f = f'
  \quad e_1 = e'_1
  \quad \ldots
  \quad e_n = e'_n
}{f(e_1, \ldots, e_n) = f'(e'_1, \ldots, e'_n)}
$$

作为可判定理论的另一个例子，让我们来考虑 *半环* 的代数结构，它可以被理解为“类似于自然数的类型”。一个集合如果含有标记为 0 和 1 的两个元素，且对二元运算符 $+$ 和 $\times$ 封闭，那么就是一个半环。这些符号是提示性的，但事实上我们可以自由选择这里的集合、元素和运算符，只要它们满足以下公理[^1]：

[^1]: 这些等式几乎原文来自 [https://en.wikipedia.org/wiki/Semiring](https://en.wikipedia.org/wiki/Semiring)。

$$
\begin{aligned}
  (a + b) + c &= a + (b + c) \\
  0 + a &= a \\
  a + 0 &= a \\
  a + b &= b + a \\
  (a \times b) \times c &= a \times (b \times c) \\
  1 \times a &= a \\
  a \times 1 &= a \\
  a \times (b + c) &= (a \times b) + (a \times c) \\
  (a + b) \times c &= (a \times c) + (b \times c) \\
  0 \times a &= 0 \\
  a \times 0 &= 0
\end{aligned}
$$

形式化的理论如下，这里我们只将从上述公理中得到的相等性判定为“真”。

$$
\begin{aligned}
  \textrm{变量} \quad & x \in \mathsf{Strings} \\
  \textrm{术语} \quad & e ::= 0 \mid 1 \mid x \mid e + e \mid e \times e \\
  \textrm{命题} \quad & \phi ::= e = e
\end{aligned}
$$

请注意半环理论和线性运算理论的适用性是不可相比的。也就是说，虽然有些目标可以通过这两种理论中的任意一种证明，但是一些目标只能使用半环理论进行证明，而另一些目标只能使用线性运算理论。例如，我们可以使用半环理论证明 $x(y + z) = xy + xz$，而使用线性运算理论证明 $x - x = 0$。


\section{Simplification and Rewriting}

While we leave most proof details to the accompanying Coq code, it does seem important to introduce two key principles that are often implicit in proofs on paper.

The first is \emph{algebraic simplification}\index{algebraic simplification}, where we apply the defining equations of a recursive definition to simplify a goal.
For example, recall that our definition of expression size included this clause.
\begin{eqnarray*}
  \size{\mathsf{Plus}(e_1, e_2)} &=& 1 + \size{e_1} + \size{e_2}
\end{eqnarray*}
Now imagine that we are trying to prove this formula.
$$\size{\mathsf{Plus}(e, \mathsf{Const}(7))} = 2 + \size{e}$$
We may apply the defining equation to rewrite into a different formula, where we have essentially pushed the definition of $\size{\cdot}$ through the $\mathsf{Plus}$.
$$1 + \size{e} + \size{\mathsf{Const}(7)} = 2 + \size{e}$$
Another application of a different defining equation, this time for $\mathsf{Const}$, takes us to here.
$$1 + \size{e} + 1 = 2 + \size{e}$$
From here, the goal follows by linear arithmetic.

\medskip

Such a proof establishes a theorem $\forall e \in \mathsf{Exp}. \; \size{\mathsf{Plus}(e, \mathsf{Const}(7))} = 2 + \size{e}$.
We may use already-proved theorems via a more general \emph{rewriting}\index{rewriting} mechanism, applying whenever we know some quantified equality.
Within a new goal we are proving, we find some subterm that matches the lefthand side of that equality, after we choose the proper values of the quantified variables.
The process of finding those values automatically is called \emph{unification}\index{unification}.
Rewriting enables us to take the subterm we found and replace it with the righthand side of the equation.

As an example, assume that, for some $P$, we know $P(2 + \size{\mathsf{Var}(x)})$ and are trying to prove $P(\size{\mathsf{Plus}(\mathsf{Var}(x), \mathsf{Const}(7))})$.
We may use our earlier fact to rewrite the argument of $P$ in what we are trying to show, so that it now matches the argument from what we already know, at which point the proof is trivial to finish.
Here, unification found the assignment $e = \mathsf{Var}(x)$.

\medskip

\encoding
\label{metalanguage}
We close the chapter with an important note on terminology.
A formula like $P(\size{\mathsf{Plus}(\mathsf{Var}(x), \mathsf{Const}(7))})$ combines several levels of notation.
We consider that we are doing our mathematical reasoning in some \emph{metalanguage}\index{metalanguage}, which is often applicable to a wide variety of proof tasks.
We also happen to be applying it here to reason about some \emph{object language}\index{object language}, a programming language whose syntax is defined formally, here the language of arithmetic expressions.
We have $x$ as a variable of the metalanguage, while $\mathsf{Var}(x)$ is a variable expression of the object language.
It is difficult to use English to explain the distinction between the two in complete formality, but be on the lookout for places where formulas mix concepts of the metalanguage and object language!
The general patterns should soon become clear, as they are somehow already familiar to us from natural-language sentences like:
\begin{quote}
  The wise man said, ``it is time to prove some theorems.''
\end{quote}
The quoted remark could just as well be in Spanish instead of English, in which case we have two languages nested in a nontrivial way.