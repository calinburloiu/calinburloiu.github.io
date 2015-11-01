---
layout: post
title:  Meta, Using Jekyll to Create This Blog
---
This post explains how I configured Jekyll to build this blog in order to have some extra features like  commenting, social media sharing buttons, table of contents for some posts or adding math formulas with \\( \\LaTeX\\).

This blog was created by using [Jekyll](http://jekyllrb.com/) a tool for geeks which allows you to write your blog or website in plain text (as [Markdown](http://daringfireball.net/projects/markdown/) or [Textile](Textile)) by transforming your text into static HTML pages.

Choosing the Markdown Processor
-------------------------------

A chose *Redcarpet* instead of *Kramdown* as the Markdown processor, because it better fit my needs:

* This is a tech blog and I am a programmer, so I often see myself adding code snippets in my posts. The processor should make this process easy.
* I sometimes add math formulas, so being able to LaTeX for this is a plus. However, it is more likely to add code than formulas, as I said, I am a programmer, not a mathematician.
* Avoid using Liquid as much as possible, because it makes the content unportable to other platforms which use Markdown.

### Fenced Code Blocks

By default, code blocks in Jekyll are created by using Liquid.

* It allows me to use GitHub flavored *fenced code blocks*, so I can easily create with triple backticks.

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

