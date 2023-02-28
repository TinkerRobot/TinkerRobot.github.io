---
title: "【FRAP】第三章：数据抽象"
date: 2022-08-23 01:00:00
tags:
- FRAP
- 形式化验证
- 读书笔记
categories:
- FRAP
---

本书中所有完全形式化的证明都是在配套的Coq代码中进行的。因此，在我们进一步讨论更多有关程序语义及其证明的内容之前，培养一些基本的Coq能力是很重要的。为此，本书配套了若干带有详细注释的示例代码文件。本书不会在“附录A”以外的章节中详细描述Coq形式化证明的细节。然而，在这些Coq代码中所展示的一种可能性值得我们注意、尽管我们尚未将它与编程语言的形式化语法联系起来，但其本身就是语义学中的一个重要概念，这就是 *数据抽象*，它是程序结构化中最核心的概念之一。我们首先研究数据结构中 *封装* 在数学上的含义。

<!-- more -->

## 3.1 抽象数据类型的代数接口

我们从一个最简单的队列开始。队列是一种可以将数据元素依次入队并将它们按照与入队时相同的顺序依次出队的经典数据结构。也许会让人惊讶的是，要实现一个高效的队列确实是有一定的复杂性的，但是使用队列的所谓 *客户端代码（client code）* 并不需要了解这些实现细节。因此，我们可以隐藏其实现细节，将“队列”表述为一个 *抽象数据类型（abstract data type）*。在诸如Coq一类的纯函数式编程的环境中，我们使用如下的方法将一个数据类型表示成一组类型和操作的集合，这看上去有些类似于Java中的接口。在下面的表达式中，类型 $\mathsf{t}(\alpha)$ 表示数据元素的类型为 $\alpha$ 的队列。

$$
\begin{aligned}
\mathsf{t}(\alpha) &: \mathsf{Set} \\
\mathsf{empty} &: \mathsf{t}(\alpha) \\
\mathsf{enqueue} &: \mathsf{t}(\alpha) \times \alpha \to \mathsf{t}(\alpha) \\
\mathsf{dequeue} &: \mathsf{t}(\alpha) \rightharpoonup \mathsf{t}(\alpha) \times \alpha
\end{aligned}
$$

这里有一些需要注意的符号用法：首先，我们通过将 $\mathsf{t}(\alpha)$ 指定为类型 `Set` 来将其声明为一个类型，这里的 `Set` 包含了编程中的所有标准类型。对于任意的类型 $\alpha$，均存在空队列(empty)和入队(enqueue)与出队(dequeue)操作。其次，dequeue使用箭头 $\rightharpoonup$ 表示这是一个偏函数：对于空队列而言，出队操作不产生任何结果。对于偏函数 $f: A \rightharpoonup B$，我们使用 $f(x) = \cdot$ 表示函数 $f$ 在 $x \in A$ 上没有映射。

在定义抽象数据类型时，通常我们就止步于这一层面了。但如果要进行形式化的正确性证明，就必须为数据类型增加若干 *规范（specifications）* (specs)。*代数规则* 是规范的一种重要形式，即定义一组量化的等式，这些等式中包含数据类型上的操作。对于队列，有以下两个规则：

$$
\begin{aligned}
\mathsf{dequeue}(\mathsf{empty}) = \cdot \\
\forall q. \; \mathsf{dequeue}(q) = \cdot \Rightarrow q = \mathsf{empty} \\
\end{aligned}
$$

实际上，上一章使用的推断规则记号能让代数规则更易读，因此上面的式子可重写为

$$
\frac{}{\mathsf{dequeue}(\mathsf{empty}) = \cdot}
\quad 
\frac{\mathsf{dequeue}(q) = \cdot}{q = \mathsf{empty}}
$$

下面这条规则使用了数学中分段函数的符号，从而完整地描述入队出队操作的行为：

$$
\frac{}{\mathsf{dequeue}(\mathsf{enqueue}(q, x)) = \begin{cases}
    (\mathsf{empty}, x), & \mathsf{dequeue}(q) = \cdot \\
    (\mathsf{enqueue}(q', x), y), & \mathsf{dequeue}(q) = (q', y)
\end{cases}}
$$

现在这个队列的功能有几种方式可以实现。首先是两个“显而易见”的方案，其一是使用一个列表，从列表首端入队、末端出队。我们使用 $\mathsf{list}(\alpha)$ 表示含有类型为 $\alpha$ 的数据元素的列表，使用 $\ell_1 \bowtie \ell_2$ 表示列表 $\ell_1$ 的 $\ell_2$ 的连接，使用在方括号内以逗号分隔的列表表示一个列表字面量。

$$
\begin{aligned}
    \mathsf{t(\alpha)} &= \mathsf{list}(\alpha) \\
    \mathsf{empty} &= [] \\
    \mathsf{enqueue}(q, x) &= {[x]} \bowtie {q} \\
    \mathsf{dequeue}([]) &= \cdot \\
    \mathsf{dequeue}({[x]} \bowtie {q}) &= ([], x)\textrm{, 若 $\mathsf{dequeue}(q) = \cdot$.} \\
    \mathsf{dequeue}({[x]} \bowtie {q}) &= ({[x]} \bowtie {q'}, y)\textrm{, 若 $\mathsf{dequeue}(q) = (q', y)$.}
\end{aligned}
$$

相对地，另一种实现方式是从列表末端入队、首端出队。

$$
\begin{aligned}
    \mathsf{t(\alpha)} &= \mathsf{list}(\alpha) \\
    \mathsf{empty} &= [] \\
    \mathsf{enqueue}(q, x) &= {q} \bowtie {[x]} \\
    \mathsf{dequeue}([]) &= \cdot \\
    \mathsf{dequeue}({[x]} \bowtie {q}) &= (q, x)
\end{aligned}
$$

对以上两种实现的代数规则的证明，均可在配套的Coq代码中找到。若假定连接操作需要花费与第一个列表的长度呈线性关系的时间，则这两种实现实际上均需花费平方级别的时间。有一种更为著名和巧妙的实现，能够实现均摊的常数时间复杂度（即对于连续的一系列操作花费线性级别的时间），但首先我们需要扩充我们的代数规则，才能描述这一实现。

## 3.2 具有自定义相等关系的代数接口

我们可以为队列的基础接口新增一个数学上的“运算”

$$
\begin{aligned}
\mathsf{t}(\alpha) &: \mathsf{Set} \\
\mathsf{empty} &: \mathsf{t}(\alpha) \\
\mathsf{enqueue} &: \mathsf{t}(\alpha) \times \alpha \to \mathsf{t}(\alpha) \\
\mathsf{dequeue} &: \mathsf{t}(\alpha) \rightharpoonup \mathsf{t}(\alpha) \times \alpha \\
\mathsf{\approx} &: \mathcal P(\mathsf{t}(\alpha) \times \mathsf{t}(\alpha))
\end{aligned}
$$


我们使用“幂集”运算 $\mathcal P$ 表示 $\approx$ 是一个作用于（相同类型的）队列的二元关系运算符。我们的目的是使用 $\approx$ 表示 *相等* 关系，其形式化定义如下：

$$
\frac{}{a \approx a} \mathsf{自反性}
\quad \frac{b \approx a}{a \approx b} \mathsf{对称性}
\quad \frac{a \approx b \quad b \approx c}{a \approx c} \mathsf{传递性}
$$

现在我们就可以使用 $\approx$ 代替等号重写原有的规则。这里我们隐式地扩大了 $\approx$ 的含义以使其能够作用在偏函数 $\mathsf{dequeue}$ 的结果上：不存在的结果 $\cdot$ 总是和自身相同；$(q_1, x_1)$ 和 $(q_2, x_2)$ 相同，当且仅当 $q_1 \approx q_2$ and $x_1 = x_2$。

$$
\frac{}{\mathsf{dequeue}(\mathsf{empty}) = \cdot}
\quad \frac{\mathsf{dequeue}(q) = \cdot}{q \approx \mathsf{empty}}
$$

$$
\frac{}{\mathsf{dequeue}(\mathsf{enqueue}(q, x)) \approx 
\begin{cases}
    (\mathsf{empty}, x), & \mathsf{dequeue}(q) = \cdot \\
    (\mathsf{enqueue}(q', x), y), & \mathsf{dequeue}(q) = (q', y)
\end{cases}}
$$

我们能够从这样的改写中得到什么回报呢？首先，通过使用 $\approx$ 表示相等性，我们能够对上面两个队列的实现进行合理性检查，即这两个实现均符合队列的规范。更重要的是，现在我们还可以开始研究经典的 *双栈队列（two-stack queue）*。下面是它的实现，这里用到了将列表反转的函数 $\mathsf{rev}$ （这一函数具有线性复杂度）。

$$
\begin{aligned}
    \mathsf{t(\alpha)} &= \mathsf{list}(\alpha) \times \mathsf{list}(\alpha) \\
    \mathsf{empty} &= ([], []) \\
    \mathsf{enqueue}((\ell_1, \ell_2), x) &= ({[x]} \bowtie {\ell_1}, \ell_2) \\
    \mathsf{dequeue}(([], [])) &= \cdot \\
    \mathsf{dequeue}((\ell_1, {[x]} \bowtie {\ell_2})) &= ((\ell_1, \ell_2), x) \\
    \mathsf{dequeue}((\ell_1, [])) &= (([], q'_1), x)\textrm{, 且 $\mathsf{rev}(\ell_1) = {[x]} \bowtie {q'_1}$.}
\end{aligned}
$$

这里的技巧在于使用一对列表 $(\ell_1, \ell_2)$ 来表示一个队列。入队时，我们向 $\ell_1$ 的队首加入元素，此时这一操作仅有常数复杂度；出队时，我们从 $\ell_2$ 的队首移除元素，此时也仅有常数复杂度。但是，如果 $\ell_2$ 中的元素被全部移除，就必须 *反转* $\ell_1$ 并将结果移至 $\ell_2$。

我们可以用下面的相等关系将上述规则形式化。

$$
\begin{aligned}
    \mathsf{rep}((\ell_1, \ell_2)) &= {\ell_1} \bowtie {\mathsf{rev}(\ell_2)} \\
    q_1 \approx q_2 &= \mathsf{rep}(q_1) = \mathsf{rep}(q_2)
\end{aligned}
$$

我们可以证明：这里定义的 $\approx$ 确实是相等关系，同时这里的实现也满足队列的所有规范。因此，客户端代码（及其正确性证明）在使用这一代码时，实际上可以将其视为一个普通的队列，隐藏底层的双栈实现。

我们为什么要大费周章地引入自定义相等关系这一操作呢？考虑下面的两个队列，它们是否相等？（这里的函数 $\pi_1$ 表示取出一个对组中的第一个元素。）

$$
\begin{aligned}
    \mathsf{enqueue}(\mathsf{empty}, 2) &\stackrel{?}{=} \pi_1(\mathsf{dequeue}(\mathsf{enqueue}(\mathsf{enqueue}(\mathsf{empty}, 1), 2)))
\end{aligned}
$$

答案是这两个队列并不相等！对双栈队列而言，第一个表达式的结果为 $([2], [])$，而第二个表达式的结果为 $([], [2])$。我们称这种数据结构为 *非规范化的（noncanonical）*，指的是逻辑上相同的值可以对应于多个实际表示。自定义的相等关系允许我们指定哪些实际表示是相等的。

## 3.3 表示函数

最后一种定义相等关系的方法基于 *表示函数（representation functions）*。我们可以要求每个队列的实现包含一个函数，这个函数将队列转换成一个标准、规范化的表示形式。实际运行的程序一般不应当调用这个函数，它的主要作用是帮助我们更好地描述代数规则。或许会令读者惊讶的是，只要符合这一要求的函数存在，就足以表明队列实现的正确性，并且这一方法可以推广到基本上所有其它作为抽象数据类型的数据结构。

修改后的队列的类型签名如下：

$$
\begin{aligned}
    \mathsf{t}(\alpha) &: \mathsf{Set} \\
    \mathsf{empty} &: \mathsf{t}(\alpha) \\
    \mathsf{enqueue} &: \mathsf{t}(\alpha) \times \alpha \to \mathsf{t}(\alpha) \\
    \mathsf{dequeue} &: \mathsf{t}(\alpha) \rightharpoonup \mathsf{t}(\alpha) \times \alpha \\
    \mathsf{rep} &: \mathsf{t}(\alpha) \to \mathsf{list}(\alpha)
\end{aligned}
$$

修改后的公理如下：

$$
\frac{}{\mathsf{rep}(\mathsf{empty}) = []}
\quad \frac{}{\mathsf{rep}(\mathsf{enqueue}(q, x)) = {[x]} \bowtie {\mathsf{rep}(q)}}
$$

$$
\frac{\mathsf{rep}(q) = []}{\mathsf{dequeue}(q) = \cdot}
\quad \frac{\mathsf{rep}(q) = {\ell} \bowtie {[x]}}{\exists q'. \; \mathsf{dequeue}(q) = (q', x) \land \mathsf{rep}(q') = \ell}
$$

值得注意的是，上述规范也可以视作 *为数据类型提供了参考实现*，此时 $\mathsf{rep}$ 表示了在任意时刻如何将数据类型转换回参考实现。

## 3.4 固定抽象数据类型的参数类型

下面是另一个经典的抽象数据结构：有限集合，其中 $\mathbb B$ 表示布尔类型的集合。

$$
\begin{aligned}
  \mathsf{t}(\alpha) &: \mathsf{Set} \\
  \mathsf{empty} &: \mathsf{t}(\alpha) \\
  \mathsf{add} &: \mathsf{t}(\alpha) \times \alpha \to \mathsf{t}(\alpha) \\
  \mathsf{member} &: \mathsf{t}(\alpha) \times \alpha \to \mathbb B
\end{aligned}
$$

下面的几条规则描述了这一数据结构的预期行为，其中 $\top$ 和 $\bot$ 分别表示 $\mathbb B$ 中的 “真（true）”和“假（false）”。

$$
\frac{}{\mathsf{member}(\mathsf{empty}, k) = \bot}
\quad \frac{}{\mathsf{member}(\mathsf{add}(s, k), k) = \top}
\quad \frac{k_1 \neq k_2}{\mathsf{member}(\mathsf{add}(s, k_1), k_2) = \mathsf{member}(s, k_2)}
$$

下面这个简单的方法使用无序列表实现了有限集合。

$$
\begin{aligned}
  \mathsf{t} &= \mathsf{list} \\
  \mathsf{empty} &= [] \\
  \mathsf{add}(s, k) &= {[k]} \bowtie {s} \\
  \mathsf{member}([], k) &= \bot \\
  \mathsf{member}({[k']} \bowtie {s}, k) &= k = k' \lor \mathsf{member}(s, k)
\end{aligned}
$$

然而，我们可以为特定的元素类型和使用场景构造特定的有限集。例如，假设我们正在处理的是自然数的集合，并且我们知道其中大多数集合包含的是连续的数字。在这种情况下，只需要存储集合中最小和最大的元素就足够了，这样所有的集合操作都只花费常数时间。假设我们现在已经有一个有限集的基本实现，其类型为 $t_0$，包含 $\mathsf{empty}_0$、$\mathsf{add}_0$ 和 $\mathsf{member}_0$ 三个操作。在这个基础上，我们可以用下面的方法实现一个优化版本的有限集，其中 $\mathsf{fromRange} : \mathbb N \times \mathbb N \to \mathsf{t}_0$ 将一个范围转换成一个临时的集合。

$$
\begin{aligned}
  \mathsf{t} &= \mathsf{Empty} \mid \mathsf{Range}(\mathbb N \times \mathbb N) \mid \mathsf{AdHoc}(\mathsf{t}_0) \\
  \mathsf{empty} &= \mathsf{Empty} \\
  \mathsf{add}(\mathsf{Empty}, k) &= \mathsf{Range}(k, k) \\
  \mathsf{add}(\mathsf{Range}(n_1, n_2), k) &= \mathsf{Range}(n_1, n_2)\textrm{, 若 $n_1 \leq k \leq n_2$} \\
  \mathsf{add}(\mathsf{Range}(n_1, n_2), n_1-1) &= \mathsf{Range}(n_1-1, n_2)\textrm{, 若 $n_1 \leq n_2$} \\
  \mathsf{add}(\mathsf{Range}(n_1, n_2), n_2+1) &= \mathsf{Range}(n_1, n_2+1)\textrm{, 若 $n_1 \leq n_2$} \\
  \mathsf{add}(\mathsf{Range}(n_1, n_2), k) &= \mathsf{AdHoc}(\mathsf{add}_0(\mathsf{fromRange}(n_1, n_2), k))\textrm{, 否则} \\
  \mathsf{add}(\mathsf{AdHoc}(s), k) &= \mathsf{AdHoc}(\mathsf{add}_0(s, k)) \\
  \mathsf{member}(\mathsf{Empty}, k) &= \bot \\
  \mathsf{member}(\mathsf{Range}(n_1, n_2), k) &= n_1 \leq k \leq n_2 \\
  \mathsf{member}(\mathsf{AdHoc}(s), k) &= \mathsf{member}_0(s, k)
\end{aligned}
$$

可以证明，如果底层的临时集合满足有限集的规范，那么上述的实现也满足有限集的规范。对于只需要对连续的数字构建集合的应用场景，这一实现的效率远高于常规的基于列表的实现，将复杂度由二次级别降低至线性级别。
