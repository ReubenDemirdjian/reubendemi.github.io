+++
author = "Reuben Demirdjian"
title = "Data Loading on a Quantum Computer: Part 3"
date = "2025-06-25"
description = "An overview of a recent journal article of mine"
draft = true
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

### Introduction
This post is part 3 of a series where I discuss data loading for solving linear systems of equations. I highly recommend that you first read <a href="https://reubendemirdjian.github.io/data_loading_part1/">Part 1</a> and <a href="https://reubendemirdjian.github.io/data_loading_part2/">Part 2</a>, otherwise this post may not make much sense. 

In this post, I will combine the concepts introduced in parts 1 and 2 to create explicit circuits that encode the matrix from the Carleman linearized and Burgers' equation. The end result is a linear combination of unitaries with not only a polylogarithmic number of terms, but each term may be encoded with polylogarithmic circuit depths. In plain language, this means that we can encode our exponentially sized matrix onto a quantum computer using only a polynomial number of classical resources. 

### Decomposition of $L^{(e)}$
In words, the $L^{(e)}$ matrix is the zero-padded Carleman linearized Burgers' equation and is defined in <a href="https://reubendemirdjian.github.io/data_loading_part2/#mjx-eqn-eqnLe">eq. (5) of Part 2</a>. As we saw in Example 2 of Part 2, this matrix can be split into two types of terms given by

$$
L^{(e)} :=
\begin{pmatrix}
  I  & 0 & \dots & 0 \\\ 
  -I & I-\Delta t A^{(e)} & \dots & 0 \\\
  \vdots & \ddots & \ddots & \vdots \\\
  0  & \dots & -I & I-\Delta t A^{(e)}
  \end{pmatrix}
= L_1^{(e)} - \Delta t L_2^{(e)}
$$
where
$$ \tag{1} \label{eqn:L1eL2e}
L_1^{(e)} = 
\begin{pmatrix}
    I  & 0 & \dots & 0 \\\
    -I & I & \dots & 0 \\\
    \vdots & \ddots & \ddots & \vdots \\\
    0  & \dots & -I & I
\end{pmatrix}
,\quad
L_2^{(e)} = 
\begin{pmatrix}
    0  & 0 & \dots & 0 \\\
    0 & A^{(e)} & \dots & 0 \\\
    \vdots & \ddots & \ddots & \vdots \\\
    0  & \dots & 0 & A^{(e)}
\end{pmatrix} .
$$
We will find the explicit circuits for $L_1^{(e)}$ and $L_2^{(e)}$ in the following subsections.

#### Decomposition of $L_1^{(e)}$
The $L_1^{(e)}$ matrix is simple and may be decomposed exactly as
$$
L_1^{(e)} = \bigg(\rho_4^{\otimes \log n_t} - \rho_4^{\otimes (\log(n_t)-1)}\otimes \rho_2
-\sum_{j=2}^{\log n_t} \rho_4^{\otimes (j-2)} \otimes \rho_2 \otimes \rho_1^{\otimes (\log(n_t)-j+1)}\bigg)
\otimes \rho_4^{\otimes\log(\alpha n_x^\alpha)} ,
$$
for truncation order $\alpha$, number of time steps $n_t$, and $\rho_j$ for $j\in\\{0,\dots,4\\}$ defined in Part 1. This linear combination has exactly $\log n_t +1$ terms. Since these terms are composed purely of $\rho_j$ components, we can use the methods introduced in Part 1 to generate their circuits as we see in the following example. 

<div style="border-radius: 10px; background: beige; padding: 10px; color: black">
	<h4 style="margin:0.3em 0"><u>Example 1</u></h4> 
	Let $\alpha=2$, $n_t=4$, and $n_x=4$, then 
  $$ \tag{2}\label{eqn:L1}
  L_1^{(e)} = 
  (\rho_4^{\otimes 2} - \rho_4\otimes\rho_2 - \rho_2\otimes\rho_1)\otimes\rho_4^{\otimes 5} .
  $$
  Following the strategy introduced in Part 1, we will block encode each of the 3 terms into a unitary matrix of the form 
  $$
  U := \begin{pmatrix} A^c & A \\\ A & A^c \end{pmatrix} = U_1 U_2
  $$
  where $A$ would be one of the 3 terms in the decomposition above. 
  <br><br>
	As in Part 1, I will use the <a href="https://github.com/Strilanc/Quirk/wiki/How-to-use-Quirk#view-the-unitary-matrix-of-a-circuit-via-the-state-channel-duality">EPR pairs</a> trick so that the state display shows the corresponding matrix for each circuit. Note that the parts of the circuits below labelled "EPR Pairs" exist simply to implement the EPR pairs trick and are not part of our block encodings.
  <br><br>
  The first term in eq. \eqref{eqn:L1} is $\rho_4^{\otimes 7}$, which has a trivial block encoding. The full circuit is given <a href="https://algassert.com/quirk#circuit={%22cols%22:[[%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22],[%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,%22X%22]]}">here in Quirk</a> and reproduced below.  
	<p class="aligncenter">
		<img src="Le1_1.png" alt="centered image" /></p>
	<style>
	.aligncenter {
		text-align: center;
	}
 	</style>	 
  This component is clearly just the main diagonal from the $L_1^{(e)}$ matrix in eq. \eqref{eqn:L1eL2e}.
  <br><br>
  The second circuit is $\rho_2\otimes\rho_1\otimes\rho_4^{\otimes 5}$ and its circuit is given
  <a href="https://algassert.com/quirk#circuit={%22cols%22:[[%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22],[%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,1,1,1,1,1,1,%22%E2%80%A2%22,1,%22X%22]]}">here in Quirk</a> and reproduced below.  
	<p class="aligncenter">
		<img src="Le1_2.png" alt="centered image" /></p>
	<style>
	.aligncenter {
		text-align: center;
	}
 	</style>	 
  This circuit fills in part of the subdiagonal $-I$ block from eq. \eqref{eqn:L1eL2e}, but not all of it. To complete this subdiagonal block we need one more term given <a href="https://algassert.com/quirk#circuit={%22cols%22:[[%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22,%22H%22],[%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,%22%E2%80%A2%22,1,1,1,1,1,1,1,%22X%22],[1,1,1,1,1,1,1,1,1,1,1,1,1,%22X%22,%22X%22],[1,1,1,1,1,1,1,1,1,1,1,1,1,%22%E2%97%A6%22,%22%E2%80%A2%22,%22X%22]]}">here in Quirk</a> and reproduced below. 
	<p class="aligncenter">
		<img src="Le1_3.png" alt="centered image" /></p>
	<style>
	.aligncenter {
		text-align: center;
	}
 	</style>
  And that's it! Verification is straightforward. Simply add each of them together and compare the resulting matrix's upper right block with $L_1^{(e)}$ from eq. \eqref{eqn:L1eL2e}. Note that this is a qualitative verification since we are only looking at the matrix structure, and not its values. However, the proper values are simply obtained by multiplying by the correct coefficients, which is easy to do.
</div>
<br>
The principles from Example 1 can be used to very easily encode the $L_1^{(e)}$ matrix of arbitrary size. In fact, we can encode exponentially sized $L_1^{(e)}$ matrices with only polynomially many classical resources!

#### Decomposition of $L_2^{(e)}$

### Explicit Circuits



__Questions or comments?__ ReubenDemi [at] gmail [dot] com

## References
<a id="1">[1]</a>
A. Gnanasekaran and A. Surana, "Efficient Variational Quantum Linear Solver for Structured Sparse Matrices," 2024 IEEE International Conference on Quantum Computing and Engineering (QCE), Montreal, QC, Canada, 2024, pp. 199-210, <a href="https://arxiv.org/abs/2404.16991">doi: 10.1109/QCE60285.2024.00033</a>

<a id="2">[2]</a> 
Demirdjian, Reuben, Thomas Hogancamp, and Daniel Gunlycke. "An Efficient Decomposition of the Carleman Linearized Burgers' Equation." arXiv preprint <a href="https://arxiv.org/abs/2505.00285">arXiv:2505.00285</a> (2025).

<a id="3">[3]</a> 
Bravo-Prieto, Carlos, et al. "Variational quantum linear solver." Quantum 7 (2023): 1188.

<a id="4">[4]</a> 
Liu, Jin-Peng, et al. "Efficient quantum algorithm for dissipative nonlinear differential equations." Proceedings of the National Academy of Sciences 118.35 (2021): e2026805118.