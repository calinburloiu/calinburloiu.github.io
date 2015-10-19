---
layout: post
title:  How to write equations with Redcarpet without escaping
---

Use `<div>` tags for displayed equations:

<div>\[ s = a_2 + b_2 \]</div>

Escaping backslashes for inline equations sucks:

By calculating \\( s = a + b, \\forall a \\in A, \\exists b \\leq \\epsilon \\) we get...

So to avoid it without using a fancy Liquid plugin that makes the Markdown unportable, use `<p>` tags. `<div>` tags makes the next line too close.

<p>By calculating \( s = a + b, \forall a \in A, \exists b \leq \epsilon \) we get to...</p>

Advanced stuff:

<p>
\[
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
\]
</p>

