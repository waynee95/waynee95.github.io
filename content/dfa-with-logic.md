---
title: "Doing Dataflow Analysis with Logic"
date: 2026-03-22
draft: true
---

Since I have been studying the modal $\mu$-calculus a lot in the last couple of 
weeks, I recently came across a [a neat little paper](https://doi.org/10.1145/268946.268950).
It fits perfectly into my areas of interest right at the intersection of 
compilers and logics.

## The modal $\mu$-calculus

The logic we use here is the modal $\mu$-calculus extended with reverse modalities.

The syntax is defined by the following grammar.

$$
\varphi ::= p \mid \neg p \mid \varphi \land \varphi \mid \varphi \lor \varphi 
  \mid \square \varphi \mid \overline{\square} \varphi \mid \lozenge \varphi 
  \mid \overline{\lozenge} \varphi
  \mid \mu Z. \varphi(Z) \mid \nu Z. \varphi(Z)
  \mid Z
$$

where $p$ is a _primitive property_ and $Z$ is a second-order variable.

A judgement is of the form $s \models_t \varphi$, where $t$ is a tree, $s$ is a 
state, and $\varphi$ is a formula meaning state $s$ satisfies $\varphi$ given a 
tree $t$.

A primitive property is a first-order property about states, e.g., "variable
$x$'s value at state $s$ is positive", written formally as $s \models_t (x > 0)$.
The modalities $\square$ (called box) and $\lozenge$ (called diamond) are used to 
express second-order properties:

- "There exists a path starting from $s$
such that variable $x$ has a positive value in two transitions": $s \models_t \lozenge\lozenge(x > 0)$.
- "All immediate predecessors to $s$ have $x$ with a positive value": $s \models_t \overline{\square} (x > 0)$.

The operators $\mu$ and $\nu$ are _fixpoint operators_. The _least fixpoint_ operator
$\mu$ is used to state properties that hold finitely many steps in the future (or past).
The _greatest fixpoint_ operator $\nu$ is used to state properties that hold
indefinitely.

- "At some state now or in the past, $x$ was positive": 
$s \models_t \mu Z. (x > 0) \lor \overline{\lozenge} Z$.
- "From now on, the value of $x$ is always positive": $s \models_t \nu Z. (x > 0) \land \square Z$.

The semantics are defined inductively on the structure of the formula. 

$$
\begin{align*}
s & \models_t q & \iff\quad & q \in L(s) \\\\
s & \models_t \neg q & \iff\quad & q \not\in L(s) \\\\
s & \models_t \varphi_1 \land \varphi_2 & \iff\quad & s \models_t \varphi_1 \text{ and } s \models_t \varphi_2 \\\\
s & \models_t \varphi_1 \lor \varphi_2 & \iff\quad & s \models_t \varphi_1 \text{ or } s \models_t \varphi_2 \\\\
s & \models_t \square \varphi & \iff\quad & \text{for all } s' \text{ such that } s \rightarrow s', s' \models_t \varphi \\\\
s & \models_t \overline{\square} \varphi & \iff\quad & \text{for all } s' \text{ such that } s' \rightarrow s, s' \models_t \varphi \\\\
s & \models_t \lozenge \varphi & \iff\quad & \text{there exists } s' \text{ such that } s \rightarrow s', s' \models_t \varphi \\\\
s & \models_t \overline{\lozenge} \varphi & \iff\quad & \text{there exists } s' \text{ such that } s' \rightarrow s, s' \models_t \varphi \\\\
s & \models_t \mu Z. \varphi & \iff\quad & \text{there exists } i \geq 0 \text{ such that }
s \models_t \varphi_i, \text{ where } \begin{cases}\varphi_0 = false \\\\ \varphi_{i + 1} = [Z \mapsto \varphi_i]\varphi\end{cases} \\\\
s & \models_t \nu Z. \varphi & \iff\quad & \text{for all } i \geq 0 \text{ such that }
s \models_t \varphi_i, \text{ where } \begin{cases}\varphi_0 = true \\\\ \varphi_{i + 1} = [Z \mapsto \varphi_i]\varphi\end{cases} \\\\
\end{align*}
$$

## Dataflow Analysis

$$
LiveVars(p) = Used(p) \cup \left( NotModified(p) \cap \bigcup_{p' \in succ(p)} LiveVars(p') \right)
$$

which translates into a formula of the modal $\mu$-calculus as

$$
isLive(x) \equiv \mu Z. isUsed(x) \lor (\neg isModified(x) \land \lozenge Z)
$$

We can do the same for the available expressions

- $AE(p) = \bigcap_{p' \in pred(p)} ((AE(p') \cap notModified(p')) \cup Gen(p'))$
- $isAvail(e) = \nu Z. \overline{\square}((Z \land \neg isModified(e)) \lor isGen(e))$

## References

- David A. Schmidt. 1998. Data flow analysis is model checking of abstract interpretations.
- Bernhard Steffen. 1991. Data Flow Analysis as Model Checking.
