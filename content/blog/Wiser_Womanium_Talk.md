+++
author = "Reuben Demirdjian"
title = "WISER/Womanium Talk"
date = "2025-07-23"
description = "A talk that I gave on using quantum computers to solve differential equations"
+++

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$']],
    displayMath: [['$$','$$']],
    processEscapes: true,
    processEnvironments: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
    TeX: { equationNumbers: { autoNumber: "AMS" },
         extensions: ["AMSmath.js", "AMSsymbols.js"] }
  }
});
</script>

<script type="text/x-mathjax-config">
  MathJax.Hub.Queue(function() {
    // Fix <code> tags after MathJax finishes running. This is a
    // hack to overcome a shortcoming of Markdown. Discussion at
    // https://github.com/mojombo/jekyll/issues/199
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>

I gave this talk on July 14th, 2025 as a part of the <a href="https://www.thewiser.org/">WISER/Womanium</a> Quantum Program. I also recommend checking out the other talks on their channel because there are truly some amazing ones on there! 

The intent of this talk is to provide a bird's-eye view of a particular methodology for using quantum computers to solve nonlinear differential equations, and some of the challenges that come along with it. I hope that you enjoy! 

{{< youtube v_FzYD7HjoQ >}}
