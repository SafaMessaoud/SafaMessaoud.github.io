---
title: "The SVGD-Induced Density"
nav_title: "Density"
permalink: /met-svgd/density/
nav_order: 3
---

As a particle-based variational inference method, SVGD [[1]](#ref-svgd) evolves a set of {::nomarkdown}$M${:/nomarkdown} particles {::nomarkdown}$\{ x_i^0 \}_{i=0}^{M-1}${:/nomarkdown} sampled from an initial distribution {::nomarkdown}$q^0${:/nomarkdown} to collectively match any arbitrarily complex target distribution {::nomarkdown}$p${:/nomarkdown}, as long as it is possible compute the score of {::nomarkdown}$p${:/nomarkdown}.
However, SVGD does not provide an explicit expression for the density it induces: while we can generate samples, we do not know the value of the corresponding probability density at those sampled points.
That is, if we evolve {::nomarkdown}$\{ x_i^0 \}_{i=0}^{M-1}${:/nomarkdown} for {::nomarkdown}$L${:/nomarkdown} steps according to the SVGD update rule, we would obtain {::nomarkdown}$\{ x_i^L \}_{i=0}^{M-1} \sim q^L${:/nomarkdown} with {::nomarkdown}$q^L \approx p${:/nomarkdown}, but we wouldn't know the value of {::nomarkdown}$q^L(x_i^L)${:/nomarkdown} for a given {::nomarkdown}$x_i^L${:/nomarkdown}.
This is problematic when we are interested in downstream tasks such as likelihood-based evaluation, uncertainty quantification, or entropy estimation, as these require access to the density itself rather than just samples.

## Derivation of the SVGD-induced Density {#sec3-1}

<figure id="density-evolution">
  <img src="/images/met-svgd/density_evolution.gif" alt="Density' evolution starting from $q^0$ and following the SVGD velocity field.">
  <figcaption>Density' evolution starting from $q^0$ and following the SVGD velocity field.</figcaption>
</figure>

<details markdown="1"><summary>Code for [the figure](/met-svgd/density/#density-evolution).</summary>

```python
from svgd.sampler import SVGD
from svgd.distributions import TorchDistribution
from svgd.kernels import RBF
from svgd.kernels.parameters import HeuristicKP
from svgd.lrs import ParameterLR
from svgd.callbacks import Logger

import torch
from torch.distributions import MixtureSameFamily, Categorical, Independent, Normal

import matplotlib
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

torch.manual_seed(0)

target_distribution = TorchDistribution(
    MixtureSameFamily(
        Categorical(torch.ones(2)),
        Independent(Normal(torch.tensor([[-1.0, 0.0], [1.0, 0.0]]), 1 / 3), 1),
    )
)
initial_distribution = TorchDistribution(Independent(Normal(torch.zeros(2), 1.0), 1))
kernel = RBF(HeuristicKP("median"))
lr = ParameterLR(torch.tensor(0.5))
logger = Logger(log_x=True)
logger.activated = True
svgd = SVGD(
    target_distribution=target_distribution,
    initial_distribution=initial_distribution,
    kernel=kernel,
    lr=lr,
    callbacks=[logger],
)

n_particles = 100
n_steps = 50
x, _, _ = svgd.sample(n_particles=n_particles, n_steps=n_steps)
x = x.detach()

bound = 2.5

grid = torch.arange(-bound, bound, 0.01)
xg, yg = torch.meshgrid(grid, grid, indexing="ij")
grid = torch.cat((xg.reshape(-1)[:, None], yg.reshape(-1)[:, None]), dim=-1)
zg = target_distribution.log_prob(grid).exp().view(xg.shape)

fig, ax = plt.subplots()
ax.pcolormesh(xg, yg, zg, cmap="Oranges")
ax.set_xlim(-bound, bound)
ax.set_ylim(-bound, bound)
ax.axis("off")
scatter = ax.scatter([], [], color="black", alpha=0.6)


def animate(frame):
    for artist in ax.collections:
        if isinstance(artist, matplotlib.contour.QuadContourSet):
            artist.remove()

    scatter.set_offsets(logger.x[frame])
    sns.kdeplot(
        x=logger.x[frame][:, 0],
        y=logger.x[frame][:, 1],
        ax=ax,
        alpha=0.4,
        color="black",
    )

    return (scatter,)


animation = FuncAnimation(
    fig,
    animate,
    len(logger.x),
    interval=1000 / 60,
    blit=False,
)
animation.save("density_evolution.gif", fps=120)
```

</details>

Suppose that, after {::nomarkdown}$l${:/nomarkdown} SVGD steps, the particle {::nomarkdown}$x^l${:/nomarkdown} is distributed according to {::nomarkdown}$q^l${:/nomarkdown}.
Then, using the change of variable formula for densities (CVF), the distribution of {::nomarkdown}$x^{l+1} = x^l + \epsilon \phi(x^l)${:/nomarkdown}, where {::nomarkdown}$\phi${:/nomarkdown} is the [SVGD velocity field](/met-svgd/svgd/#the-svgd-update-rule), is:


<div class="ms-aside" markdown="1" id="cvf-aside">

<p class="ms-aside-title">Change of Variable Formula (CVF)</p>

Given a random variable {::nomarkdown}$X \in \mathbb{R}^D${:/nomarkdown} and an invertible, deterministic transformation {::nomarkdown}$T:\mathbb{R}^D\to\mathbb{R}^D${:/nomarkdown}, the CVF gives the expression of the density of {::nomarkdown}$Y=T(X)${:/nomarkdown}: {::nomarkdown}$q_Y(T(x)) = q_X(x)\left|\det \nabla_{x}T(x)\right|^{-1}${:/nomarkdown}.

</div>



{::nomarkdown}
$$
\begin{align*}
q^{l+1}(x^{l+1})
&=
q^{l}(x^{l})
\left|\det \nabla_{x^l} \left( x^l + \epsilon \phi(x^l) \right)\right|^{-1}
\\
&=
q^{l}(x^{l})
\left|\det \left( I + \epsilon \nabla_{x^l} \phi(x^l) \right)\right|^{-1}.
\end{align*}
$$
{:/nomarkdown}


However, to apply the CVF, the SVGD transformation must be invertible.
To ensure this, MET-SVGD adapts a sufficient conditions for the invertibility of transformations of the form {::nomarkdown}$f(x)=x+g(x)${:/nomarkdown} from [[2]](#ref-behrinv) to SVGD.
Namely, {::nomarkdown}$f(x^l) = x^l + \epsilon \phi(x^l)${:/nomarkdown} is invertible if {::nomarkdown}$\epsilon \sup_{x^l} ||\nabla_{x^l} \phi(x^l)||_2 < 1${:/nomarkdown}, where {::nomarkdown}$||\nabla_{x^l} \phi(x^l)||_2${:/nomarkdown} denotes the spectral norm of {::nomarkdown}$\nabla_{x^l} \phi(x^l)${:/nomarkdown}.


<div class="ms-aside" markdown="1">

<p class="ms-aside-title">Spectral Norm</p>

The spectral norm of a real-valued matrix {::nomarkdown}$A${:/nomarkdown} is defined as {::nomarkdown}$||A||_2 = \sigma_\text{max}(A)${:/nomarkdown}, where {::nomarkdown}$\sigma_\text{max}(A)${:/nomarkdown} is the largest singular value of {::nomarkdown}$A${:/nomarkdown}.
An equivalent and more intuitive definition is {::nomarkdown}$||A||_2 = \max_{||v||=1} ||Av||${:/nomarkdown}, i.e. {::nomarkdown}$||A||_2${:/nomarkdown} measures the maximum amount by which {::nomarkdown}$A${:/nomarkdown} can stretch a unit vector.

</div>


In practice, it is easier to work with the log of the density, also called the log-likelihood:

{::nomarkdown}
$$
\log q^{l+1}(x^{l+1})
=
\log q^{l}(x^{l})
-
\log \left|\det \left( I + \epsilon \nabla_{x^l} \phi(x^l) \right)\right|.
$$
{:/nomarkdown}


Unfortunately, this expression involves the log of the determinant of a jacobian, which is expensive to compute.
To avoid this, under the condition
{::nomarkdown}$\epsilon |\lambda_\text{max}(\nabla_{x^l} \phi(x^l))| < 1${:/nomarkdown},
MET-SVGD accurately estimates
{::nomarkdown}$\log \left|\det \left( I + \epsilon \nabla_{x^l} \phi(x^l) \right)\right|${:/nomarkdown}
by
{::nomarkdown}$\epsilon \mathrm{Tr}\left( \nabla_{x^l} \phi(x^l) \right)${:/nomarkdown},
giving:


{::nomarkdown}
$$
\log q^{l+1}(x^{l+1})
\approx
\log q^{l}(x^{l})
-
\epsilon \mathrm{Tr}\left( \nabla_{x^l} \phi(x^l) \right).
$$
{:/nomarkdown}


For samples {::nomarkdown}$\{ x_i^l \}_{i=0}^{M-1} \sim q^l${:/nomarkdown}, the trace term evaluated at {::nomarkdown}$x_i^l${:/nomarkdown} is:

{::nomarkdown}
$$
\begin{align*}
\mathrm{Tr}\left( \nabla_{x_i^l} \phi(x_i^l) \right)
&=
\frac{1}{M}
\sum_{j=0}^{M-1}
\left[
\nabla_{x_i^l} \kappa(x_i^l, x_j^l)^\top
\nabla_{x_j^l} \log \bar p(x_j^l)
+
\mathrm{Tr}\left( \nabla_{x_i^l} \nabla_{x_j^l} \kappa(x_i^l, x_j^l) \right)
\right]
\\
&+
\frac{1}{M}
\mathrm{Tr}\left( \nabla_{x_i^l}^2 \log \bar p(x_i^l) \right).
\end{align*}
$$
{:/nomarkdown}

When {::nomarkdown}$\kappa${:/nomarkdown} is the RBF kernel, the first term in the above expression can be efficiently computed using only vector dot products.
However, the second term is computationally expensive because it involves computing the trace of a hessian, which has {::nomarkdown}$\mathcal{O}(D^2)${:/nomarkdown} complexity.

One way to altogether bypass computing this term is to estimate the expectation in {::nomarkdown}$\phi(x_i^l)${:/nomarkdown} using {::nomarkdown}$\{ x_i^l \}_{i=0}^{M-1} - \{ x_i^ l\}${:/nomarkdown}.
However, this turns out to be suboptimal in the finite particle case.
Instead, MET-SVGD efficiently estimates it as:


<div class="ms-aside" markdown="1">

- {::nomarkdown}$(i)${:/nomarkdown} Using Hutchinson's estimator [[3]](#ref-hutch)
- {::nomarkdown}$(ii)${:/nomarkdown} Using the double differentiation trick [[4]](#ref-ddt)

</div>



{::nomarkdown}
$$
\begin{align*}
\frac1M\mathrm{Tr}\left( \nabla_{x_i^l}^2 \log \bar p(x_i^l) \right)
&\stackrel{(i)}{=}
\frac1M\mathbb{E}_{v \sim p_v}\left[
v^\top
\nabla_{x_i^l}^2 \log \bar p(x_i^l)
v
\right] \\
&\stackrel{(ii)}{=}
\frac1M\mathbb{E}_{v \sim p_v}\left[
\nabla_{x_i^l} \left( v^\top \nabla_{x_i^l} \log \bar p(x_i^l) \right)
v
\right] \\
&\approx
\frac1{MV}
\sum_{k=0}^{V-1}
\nabla_{x_i^l} \left(
v_k^\top \nabla_{x_i^l} \log \bar p(x_i^l)
\right)
v_k,
\end{align*}
$$
{:/nomarkdown}


where {::nomarkdown}$v \sim p_v${:/nomarkdown} satisfy {::nomarkdown}$\mathbb{E}_{v \sim p_v}[v]=0${:/nomarkdown} and {::nomarkdown}$\mathbb{E}_{v \sim p_v}[vv^\top]=I${:/nomarkdown}, and {::nomarkdown}$V${:/nomarkdown} is the number of {::nomarkdown}$v_k${:/nomarkdown} samples.

Since the estimator is weighted by {::nomarkdown}$\frac1M${:/nomarkdown}, its variance is greatly reduced, and, in practice, only one {::nomarkdown}$v${:/nomarkdown} is sufficient.

## Unifying the Step-Size Conditions {#sec3-eps}

The correctness of the previous derivation depends on two conditions:
- {::nomarkdown}$\epsilon \sup_{x^l} ||\nabla_{x^l} \phi(x^l)||_2 < 1${:/nomarkdown} to ensure that the SVGD transformation is invertible
- {::nomarkdown}$\epsilon |\lambda_\text{max}(\nabla_{x^l} \phi(x^l))| < 1${:/nomarkdown} for the approximation of {::nomarkdown}$\log \left|\det \left( I + \epsilon \nabla_{x^l} \phi(x^l) \right)\right|${:/nomarkdown} to be accurate

While these are separate conditions, they can be unified by considering the order relation between the spectral norm of a real-valued square matrix {::nomarkdown}$A${:/nomarkdown} and the magnitude of {::nomarkdown}$\lambda_\text{max}(A)${:/nomarkdown}.

According to [[5]](#ref-matrixbounds), for {::nomarkdown}$A \in \mathbb{R}^{d \times d}${:/nomarkdown}:

{::nomarkdown}
$$
|\lambda_i(A)|
\leq
\sigma_i(A)
\leq
\sqrt{ \mathrm{Tr}(A^\top A) }
\quad
\forall i \in [1 \dots d],
$$
{:/nomarkdown}

where {::nomarkdown}$\lambda_i(A)${:/nomarkdown} and {::nomarkdown}$\sigma_i(A)${:/nomarkdown} are the {::nomarkdown}$i${:/nomarkdown}-th eigenvalue and singular value of {::nomarkdown}$A${:/nomarkdown}, respectively.

Given that {::nomarkdown}$||\nabla_{x^l} \phi(x^l)||_2 = \sigma_\text{max}(\nabla_{x^l} \phi(x^l))${:/nomarkdown}, we have:

{::nomarkdown}
$$
\epsilon |\lambda_\text{max}(\nabla_{x^l} \phi(x^l))|
\leq
\epsilon \sup_{x^l} ||\nabla_{x^l} \phi(x^l)||_2
\leq
\epsilon \sup_{x^l} \sqrt{ \mathrm{Tr}\left( \nabla_{x^l} \phi(x^l)^\top \nabla_{x^l} \phi(x^l) \right) }
.
$$
{:/nomarkdown}


Therefore, in order to satisfy both conditions, it is sufficient to choose the step-size such that:

{::nomarkdown}
$$
\epsilon
<
\left(
\sup_{x^l}
\sqrt{
\mathrm{Tr}\Big( \nabla_{x^l} \phi(x^l)^\top \nabla_{x^l} \phi(x^l) \Big)
}
\right)^{-1}
.
$$
{:/nomarkdown}


Note that
{::nomarkdown}$\mathrm{Tr}\Big( \nabla_{x^l} \phi(x^l)^\top \nabla_{x^l} \phi(x^l) \Big)${:/nomarkdown}
can be efficiently computed using only vector dot products and first-order derivatives, similarly to
{::nomarkdown}$\mathrm{Tr}\left( \nabla_{x^l} \phi(x^l) \right)${:/nomarkdown}.
And, in practice, MET-SVGD solves {::nomarkdown}$\sup_{x^l}${:/nomarkdown} by taking the maximum over particles {::nomarkdown}$\{ x_i^l \}_{i=0}^{M-1}${:/nomarkdown} at iteration {::nomarkdown}$l${:/nomarkdown}.

## References {#references}

1. <a id="ref-svgd"></a>Liu, Qiang, Wang, Dilin. *Stein variational gradient descent: A general purpose bayesian inference algorithm*. NeurIPS (2016)
2. <a id="ref-behrinv"></a>Behrmann, Jens, Grathwohl, Will, Chen, Ricky TQ, Duvenaud, David, Jacobsen, Jorn-Henrik. *Invertible residual networks*. ICML (2019)
3. <a id="ref-hutch"></a>Hutchinson, Michael F. *A stochastic estimator of the trace of the influence matrix for Laplacian smoothing splines*. Commun. Stat. Simul. Comput. (1989)
4. <a id="ref-ddt"></a>Song, Yang, Garg, Sahaj, Shi, Jiaxin, Ermon, Stefano. *Sliced score matching: A scalable approach to density and score estimation*. UAI (2020)
5. <a id="ref-matrixbounds"></a>Wolkowicz, Henry, Styan, George PH. *Bounds for eigenvalues using traces*. Linear Algebra Appl. (1980)

