---
layout: post
title: "Support LaTex in Octopress"
date: 2012-08-18 00:57
comments: true
categories: Octopress
tags: octopress latex kramdown mathjax
keywords: latex in octopress, kramdown, mathjax
description: How to add support of latex math in octopress
---

It took me some time to finally get latex math formulas working in Octopress.
If you googled 'Octopress latex', you can get quite a few online resources about
how to support latex in octopress, with various levels of complexity. In this
post, I will write down how I achieve this.

The initial attempt
-------------------
As I installed the [jekyll-rst](https://github.com/xdissent/jekyll-rst) plugin
to use rst to write my posts, I thought it should be easy to write latex math
because docutils has native support for it since version 0.8 (A :math: role and
also a .. math: directive introduced for that). However, after I tried to use
these in a octopress post, I found that the post will be rendered to empty. For
example, if I insert the following rst code into my post, the whole post becomes
empty; but after removing this line, everything is fine.


```
The area of a circle is :math:`A_\text{c} = (\pi/4) d^2`.
```

I also verified that the exact same code can be successfully converted to valid
html using the 'rst2html.py' script on my system, so I guess maybe something is
wrong in 'RbST'. I found that in RbST, it has its own copies of rst2html and
rst2latex tools under ** /gems/RbST-0.1.3/lib/rst2parts **,

``` console
chuchao@chuchao:~/.rvm/gems/ruby-1.9.2-p320/gems/RbST-0.1.3/lib/rst2parts$ ls
__init__.py  rst2html.py  rst2latex.py  transform.py  transform.pyc
chuchao@chuchao:~/.rvm/gems/ruby-1.9.2-p320/gems/RbST-0.1.3/lib/rst2parts$ ls ..
rbst.rb  rst2parts
```

which will be used in rbst.rb. I have even tried to change rbst.rb to use the
rst2html.py installed on my system, but this also didn't get any luck.

``` ruby
class RbST
  
  @@executable_path = File.expand_path(File.join(File.dirname(__FILE__), "rst2parts"))
  @@executables = {
    # :html  => File.join(@@executable_path, "rst2html.py"),
    :html  => "/usr/local/bin/rst2html.py",
    :latex => File.join(@@executable_path, "rst2latex.py")
  }
...
```

Finally, I gave up on this and opened an
[issue](https://github.com/xdissent/jekyll-rst/issues/6) on this for the jekyll-rst
plugin. Hope the author can fix this.

Switch back to markdown and using kramdown
------------------------------------------
After a google search about this issue, seems that the simplest solution is to use
kramdown, which has built-in support for latex.

First, install kramdown:
`gem install karmdown`

Then, add mathjax configs into <head></head> tag, in octopress, just add the
below code into `/source/_includes/custom/head.html`:

``` html
<!-- mathjax config similar to math.stackexchange -->

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>

<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
      }
    });
</script>

<script type="text/x-mathjax-config">
    MathJax.Hub.Queue(function() {
        var all = MathJax.Hub.getAllJax(), i;
        for(i=0; i < all.length; i += 1) {
            all[i].SourceElement().parentNode.className += ' has-jax';
        }
    });
</script>

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

See [here](http://kramdown.rubyforge.org/syntax.html#math-blocks) for more details.
After this, we are ready to test latex math in our post. For example:

``` latex
$$
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
$$
```

will render as

$$
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
$$

And for inline latex code, just use `$\exp(-\frac{x^2}{2})$`, which will give
$\exp(-\frac{x^2}{2})$.
