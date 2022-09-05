---
title: "【FRAP】第五章：归纳关系与规则推理"
date: 2022-08-30 01:00:00
tags:
- FRAP
- 形式化验证
- 读书笔记
categories:
- FRAP
---

让我们先在此处暂停一下，引入另一个重要的数学工具，它虽然在语义学的研究之外并不常用，但对于我们之后将要定义的几乎所有语言的语义而言都是必不可少的。这个工具类似于我们在第二章所见到的归纳 *集合* 或归纳 *类型*。但是这一次我们要归纳定义的是 *关系*（和 *断言*，单参数关系的通俗说法）。我们先从简单的例子开始，随后再深入研究和语义更加相关的案例。

<!-- more -->

## 5.1 使用有限集合作为归纳断言

任何有限集合都可以简单地定义为断言，即一组不含前提的推断规则。举例来说，假设我最喜欢的数字是17、23和42，我们可以定义断言 $\mathsf{favorites}$ 如下：

$$
\newcommand{\favs}[1]{\mathsf{favorites}(#1)}

\frac{}{\favs{17}}
\quad \frac{}{\favs{23}}
\quad \frac{}{\favs{42}}
$$

如同我们将归纳集合定义为满足给定的推断规则的 *最小* 集合，我们也将归纳断言定义为满足该规则的 *最小* 断言。这里所给出的规则只接受17、23和42，因此这些也正是断言所能接受的值。

任何归纳断言的定义都有一个对应的归纳法则，我们可以仿照第二章中的做法对其进行推导。具体来说，当我们归纳地定义了 $P$，并且想要从中得出结论 $Q$ 时，我们实际上是在证明 $\forall x. \; P(x) \Rightarrow Q(x)$。

Any inductive predicate definition has an associated induction principle\index{inductive principle}, which we can derive much as we derived induction principles in Chapter \ref{syntax}.
Specifically, when we defined $P$ inductively and want to conclude $Q$ from it, we want to prove $\forall x. \; P(x) \Rightarrow Q(x)$.
We transform each rule into one obligation within the induction, by replacing $P$ with $Q$ in the conclusion, in addition to taking each premise $P(e)$ and pairing it with a new premise $Q(e)$ (an \emph{induction hypothesis}\index{induction hypothesis}).

任何归纳式谓词定义都有一个相关的归纳原则，我们可以像在第二章中推导归纳原则一样推导出这个原则。具体来说，当我们归纳地定义了P并想从它得出Q的结论时，我们要证明@x。Ppxq æ Qpxq。我们将每条规则转化为归纳法中的一项义务，在结论中用Q代替P，此外还将每个前提Ppeq与一个新的前提Qpeq（一个归纳假设）配对。