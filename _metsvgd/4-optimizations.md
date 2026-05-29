---
title: "MET-SVGD Optimizations"
nav_title: "Optimizations"
permalink: /met-svgd/optimizations/
nav_order: 4
---

<figure id="metsvgd-overview">
  <img src="/images/met-svgd/metsvgd.svg" alt="Overview of the MET-SVGD framework.">
  <figcaption>Overview of the MET-SVGD framework.</figcaption>
</figure>

## End-to-End SVGD Parameter Learning via Reverse KL Minimization

Although we consistently observe the particle converging to the high density regions of the target distributions for different values of the kernel bandwidth {::nomarkdown}$\sigma${:/nomarkdown}, using the derived closed-form expression of the SVGD-induced density to estimate the entropy of the target is very sensitive to the choice of {::nomarkdown}$\sigma${:/nomarkdown}, in the sense that the estimate only converges to the true entropy {::nomarkdown}$\mathcal{H}(p)${:/nomarkdown} for specific {::nomarkdown}$\sigma${:/nomarkdown} values.

<figure id="sigma-sensitivity">
<div class="ms-figrow">
<figure>
  <img src="/images/met-svgd/sensitivity_1.gif" alt="Particle and entropy evolution with $\sigma=1$.">
  <figcaption>Particle and entropy evolution with $\sigma=1$.</figcaption>
</figure>
<figure>
  <img src="/images/met-svgd/sensitivity_5.gif" alt="Particle and entropy evolution with $\sigma=5$.">
  <figcaption>Particle and entropy evolution with $\sigma=5$.</figcaption>
</figure>
</div>
<figcaption>Particle and entropy evolution for different $\sigma$ values. The convergence of the entropy estimate to the true value depends on the choice of $\sigma$ despite particles' convergence to the target.</figcaption>
</figure>

<details markdown="1"><summary>Code for [the figure](/met-svgd/optimizations/#sigma-sensitivity).</summary>

```python
from svgd.sampler import SVGD
from svgd.distributions import TorchDistribution, Gaussian
from svgd.kernels import RBF
from svgd.kernels.parameters import HeuristicKP, ParameterKP
from svgd.lrs import ParameterLR
from svgd.callbacks import Logger

import torch
from torch.distributions import MultivariateNormal

import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

torch.manual_seed(0)

sigma_choice = "1"

mu_x, mu_y = -0.6871, 0.8010
target_distribution = TorchDistribution(
    MultivariateNormal(
        torch.Tensor([mu_x, mu_y]),
        torch.Tensor([[0.2260, 0.1652], [0.1652, 0.6779]]).mul(5),
    )
)
initial_distribution = Gaussian(torch.zeros(2), torch.ones(2).mul(6).sqrt())
sigma = (
    HeuristicKP("median")
    if sigma_choice == "median"
    else ParameterKP(torch.tensor(float(sigma_choice)))
)
kernel = RBF(sigma)
lr = ParameterLR(torch.tensor(0.1))
logger = Logger(log_x=True, log_log_q=True)
logger.activated = True
svgd = SVGD(
    target_distribution=target_distribution,
    initial_distribution=initial_distribution,
    kernel=kernel,
    lr=lr,
    callbacks=[logger],
)

n_particles = 200
n_steps = 2000
x, _, _ = svgd.sample_with_log_q(n_particles=n_particles, n_steps=n_steps)
x = x.detach()

bound = 5
grid_x = torch.arange(mu_x - bound, mu_x + bound, 0.01)
grid_y = torch.arange(mu_y - bound, mu_y + bound, 0.01)
xg, yg = torch.meshgrid(grid_x, grid_y, indexing="ij")
grid = torch.cat((xg.reshape(-1)[:, None], yg.reshape(-1)[:, None]), dim=-1)
zg = target_distribution.log_prob(grid).exp().view(xg.shape)

fig, ax = plt.subplots(1, 2, figsize=(6.4 * 2, 4.8))

ax[0].pcolormesh(xg, yg, zg, cmap="Oranges")
ax[0].set_xlim(mu_x - bound, mu_x + bound)
ax[0].set_ylim(mu_y - bound, mu_y + bound)
ax[0].set_axis_off()
scatter = ax[0].scatter([], [], color="black", alpha=0.6)
scatter = ax[0].scatter(*x.T, color="black", alpha=0.6)

ax[1].plot(
    [target_distribution.distribution.entropy() for _ in range(n_steps)],
    color="black",
    ls="--",
    label="Ground-Truth",
)
plot = ax[1].plot([-log_q for log_q in logger.log_q], label="$q^l$")[0]
plot.set_xdata([])
plot.set_ydata([])
ax[1].set_ylabel("$\\mathcal{H}(q)$")
ax[1].legend()

x_data = list(range(len(logger.x)))
y_data = [-log_q for log_q in logger.log_q]


def animate(frame):
    scatter.set_offsets(logger.x[min(frame, len(logger.x) - 1)])
    plot.set_xdata(x_data[:frame])
    plot.set_ydata(y_data[:frame])
    return (scatter, plot)


animation = FuncAnimation(
    fig,
    animate,
    list(range(0, n_steps + 20, 20)),
    interval=1000 / 60,
    blit=False,
)
animation.save(f"sensitivity_{sigma_choice}.gif", fps=120)
```

</details>

The target's entropy estimate based on {::nomarkdown}$q^L${:/nomarkdown} is:

{::nomarkdown}
$$
\begin{align*}
\mathcal{H}(p)
&\approx
-\mathbb{E}_{x^L \sim q^L} \left[ \log q^L(x^L) \right] \\
&\approx
-\mathbb{E}_{x^{L-1} \sim q^{L-1}} \left[ \log q^{L-1}(x^{L-1}) \right]
+\epsilon\mathbb{E}_{x^{L-1} \sim q^{L-1}} \left[ \mathrm{Tr}\left( \nabla_{x^{L-1}} \phi(x^{L-1}) \right) \right]
.
\end{align*}
$$
{:/nomarkdown}


As {::nomarkdown}$L\to\infty${:/nomarkdown}, we expect {::nomarkdown}$q^L=q^{L-1}${:/nomarkdown} and the trace term above to converge zero in expectation.
However, this is not trivial from the perspective of {::nomarkdown}$\sigma${:/nomarkdown} selection.
It can be shown via a taylor expansion that this trace term corresponds to an {::nomarkdown}$8${:/nomarkdown}-th degree polynomial in {::nomarkdown}$\sigma${:/nomarkdown}, with coefficients that do not guarantee the existence of real roots.

To resolve this, MET-SVGD transforms the hyperparameter selection problem into a parameter learning one by leveraging the computed {::nomarkdown}$\{ \log q^L(x_i^L) \}_{i=0}^{M-1}${:/nomarkdown} to minimize the KL divergence between {::nomarkdown}$q^L${:/nomarkdown} and {::nomarkdown}$p${:/nomarkdown}, thus obtaining both the optimal kernel bandwidth {::nomarkdown}$\sigma_{\theta_2^*}^l${:/nomarkdown} and step-size {::nomarkdown}$\epsilon_{\theta_3^*}^l${:/nomarkdown} for every step {::nomarkdown}$l${:/nomarkdown}:

{::nomarkdown}
$$
\theta^*
=
\underset{\theta}{\mathrm{argmin}}
-\mathcal{H}(q^L_\theta)
-\mathbb{E}_{x^L \sim q^L_\theta}\left[ \log \bar p(x^L) \right]
\quad
\text{s.t.}
\quad
\epsilon_{\theta_3}^l \leq \epsilon_\text{UB}^l
\quad
\forall l \in [0 \dots L-1],
$$
{:/nomarkdown}

where {::nomarkdown}$\epsilon_\text{UB}^l${:/nomarkdown} is the step-size upper bound derived in [Section 3.2](/met-svgd/density/#sec3-eps).

Learning the step-size alongside the kernel bandwidth is necessary, since, from the argument above, the kernel bandwidth alone might not be enough for the trace term to converge to zero.

<figure id="sensitivity-metsvgd">
  <img src="/images/met-svgd/sensitivity_metsvgd.gif" alt="Particle and entropy evolution with learnable kernel bandwidth and step-size.">
  <figcaption>Particle and entropy evolution with learnable kernel bandwidth and step-size.</figcaption>
</figure>

<details markdown="1"><summary>Code for [the figure](/met-svgd/optimizations/#sensitivity-metsvgd).</summary>

```python
from svgd.sampler import SVGD
from svgd.distributions import TorchDistribution, Gaussian
from svgd.kernels import RBF
from svgd.kernels.parameters import StepParameterKP
from svgd.lrs import StepLR
from svgd.callbacks import Logger

import torch
from torch.optim.adam import Adam
from torch.distributions import MultivariateNormal

from tqdm import tqdm
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

torch.manual_seed(0)

device = "cuda:0"
n_epochs = 100
n_particles = 200
n_steps = 200

mu_x, mu_y = -0.6871, 0.8010
target_distribution = TorchDistribution(
    MultivariateNormal(
        torch.Tensor([mu_x, mu_y]).to(device),
        torch.Tensor([[0.2260, 0.1652], [0.1652, 0.6779]]).mul(5).to(device),
    )
)
initial_distribution = Gaussian(
    torch.zeros(2), torch.ones(2).mul(6).sqrt()
).requires_grad_(False)
sigma = StepParameterKP(torch.zeros(n_steps), lambda x: x.exp())
kernel = RBF(sigma)
lr = StepLR(torch.tensor(0.1), torch.tensor(n_steps - 20), torch.tensor(1e-6))
lr._log_step_size.requires_grad_(False)
lr._log_decay_rate.requires_grad_(False)
logger = Logger(log_x=True, log_log_q=True)
svgd = SVGD(
    target_distribution=target_distribution,
    initial_distribution=initial_distribution,
    kernel=kernel,
    lr=lr,
    bound_lr=True,
    ij_term_density=True,
    callbacks=[logger],
).to(device)
optimizer = Adam(svgd.parameters(), 1e-2)

for epoch in tqdm(range(n_epochs)):
    logger.activated = epoch == n_epochs - 1
    x, _, log_q = svgd.sample_with_log_q(n_particles=n_particles, n_steps=n_steps)
    log_q = log_q.mean()
    log_p = target_distribution.log_prob(x).mean()
    kld = log_q.sub(log_p)
    kld.backward()
    optimizer.step()
    optimizer.zero_grad()

bound = 5
grid_x = torch.arange(mu_x - bound, mu_x + bound, 0.01)
grid_y = torch.arange(mu_y - bound, mu_y + bound, 0.01)
xg, yg = torch.meshgrid(grid_x, grid_y, indexing="ij")
grid = torch.cat((xg.reshape(-1)[:, None], yg.reshape(-1)[:, None]), dim=-1)
zg = target_distribution.log_prob(grid.to(device)).exp().view(xg.shape).cpu()

fig, ax = plt.subplots(1, 2, figsize=(6.4 * 2, 4.8))

ax[0].pcolormesh(xg, yg, zg, cmap="Oranges")
ax[0].set_xlim(mu_x - bound, mu_x + bound)
ax[0].set_ylim(mu_y - bound, mu_y + bound)
ax[0].set_axis_off()
scatter = ax[0].scatter([], [], color="black", alpha=0.6)
scatter = ax[0].scatter(*x.detach().cpu().T, color="black", alpha=0.6)

ax[1].plot(
    [target_distribution.distribution.entropy().item() for _ in range(n_steps)],
    color="black",
    ls="--",
    label="Ground-Truth",
)
plot = ax[1].plot([-log_q for log_q in logger.log_q], label="$q^l$")[0]
plot.set_xdata([])
plot.set_ydata([])
ax[1].set_ylabel("$\\mathcal{H}(q)$")
ax[1].legend()

x_data = list(range(len(logger.x)))
y_data = [-log_q for log_q in logger.log_q]


def animate(frame):
    scatter.set_offsets(logger.x[min(frame, len(logger.x) - 1)])
    plot.set_xdata(x_data[:frame])
    plot.set_ydata(y_data[:frame])
    return (scatter, plot)


animation = FuncAnimation(
    fig,
    animate,
    list(range(0, n_steps + 2, 2)),
    interval=1000 / 60,
    blit=False,
)
animation.save(f"sensitivity_metsvgd.gif", fps=120)
```

</details>

## Faster Convergence via Learning the Initial Distribution

To accelerate convergence, MET-SVGD also learns the initial distribution {::nomarkdown}$q^0_{\theta_1}${:/nomarkdown}.
This is done via reverse KL minimization, similar to learning the kernel bandwidth and step-size.

<figure id="learn-distribution">
<div class="ms-figrow">
<figure id="learn-distribution-yes">
  <img src="/images/met-svgd/particle_evolution_learnable_1.gif" alt="Particle's evolution with a learned initial distribution.">
  <figcaption>Particle's evolution with a learned initial distribution.</figcaption>
</figure>
<figure id="learn-distribution-no">
  <img src="/images/met-svgd/particle_evolution_learnable_2.gif" alt="Particle's evolution with a pre-specified initial distribution.">
  <figcaption>Particle's evolution with a pre-specified initial distribution.</figcaption>
</figure>
</div>
<figcaption>Learning the initial distribution leads to faster convergence.</figcaption>
</figure>

<details markdown="1"><summary>Code for [the figure](/met-svgd/optimizations/#learn-distribution).</summary>

```python
from svgd.sampler import SVGD
from svgd.distributions import TorchDistribution, Gaussian
from svgd.kernels import RBF
from svgd.kernels.parameters import ParameterKP
from svgd.lrs import StepLR
from svgd.callbacks import Logger

import torch
from torch.optim.adam import Adam
from torch.distributions import MixtureSameFamily, Categorical, Independent, Normal

from tqdm import tqdm
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

from copy import deepcopy

torch.manual_seed(0)

device = "cuda:0"

n_epochs = 300
n_particles = 50
n_steps = 100

target_distribution = TorchDistribution(
    MixtureSameFamily(
        Categorical(torch.ones(2).to(device)),
        Independent(
            Normal(torch.tensor([[-1.0, 0.0], [1.0, 0.0]]).to(device), 1 / 3), 1
        ),
    )
)
initial_distribution = Gaussian(torch.zeros(2), torch.ones(2).mul(2.0)).to(device)
initial_distribution.mu.requires_grad_(False)
sigma = ParameterKP(torch.tensor(0.0), lambda x: x.exp()).to(device)
kernel = RBF(sigma)
lr = StepLR(
    torch.tensor(0.1),
    torch.tensor(n_steps / 2),
    torch.tensor(1e-9),
).to(device)
lr._log_decay_rate.requires_grad_(False)
lr._log_initial_lr.requires_grad_(False)
logger = Logger(log_x=True)
logger.activated = True
svgd = SVGD(
    target_distribution=target_distribution,
    initial_distribution=initial_distribution,
    kernel=kernel,
    lr=lr,
    bound_lr=True,
    ij_term_density=True,
    leaky_lr_clamp=True,
    callbacks=[logger],
).to(device)
svgd_fixed = deepcopy(svgd)
svgd_fixed.initial_distribution.requires_grad_(False)
logger_fixed: Logger = svgd_fixed.callbacks[0]
optimizer = Adam(svgd.parameters(), 1e-2)
optimizer_fixed = Adam(svgd_fixed.parameters(), 1e-2)


def optimize_svgd(svgd: SVGD, n_steps: int, optimizer: Adam):
    x, _, log_q = svgd.sample_with_log_q(n_particles=n_particles, n_steps=n_steps)
    log_q = log_q.mean()
    log_p = target_distribution.log_prob(x).mean()
    kld = log_q.sub(log_p)
    kld.backward()
    optimizer.step()
    optimizer.zero_grad()


for epoch in tqdm(range(n_epochs)):
    optimize_svgd(svgd, n_steps, optimizer)
    optimize_svgd(svgd_fixed, n_steps, optimizer_fixed)

grid = torch.arange(-2, 2, 0.01)
xg, yg = torch.meshgrid(grid, grid, indexing="ij")
grid = torch.cat((xg.reshape(-1)[:, None], yg.reshape(-1)[:, None]), dim=-1)
zg = target_distribution.log_prob(grid.to(device)).exp().view(xg.shape).cpu()

fig, ax = plt.subplots(1, 1, figsize=(6.4 * 1, 4.8 * 1))
ax.pcolormesh(xg, yg, zg, cmap="Oranges")
ax.set_xlim(-2.0, 2.0)
ax.set_ylim(-2.0, 2.0)
ax.axis("off")
scatter = ax.scatter([], [], color="black", alpha=0.6)


def animate(frame, logger):
    scatter.set_offsets(logger.x[frame])
    return (scatter,)


FuncAnimation(
    fig,
    lambda frame: animate(frame, logger),
    range(len(logger.x)),
    interval=1000 / 60,
    blit=False,
).save("particle_evolution_learnable_1.gif", fps=120)
FuncAnimation(
    fig,
    lambda frame: animate(frame, logger_fixed),
    range(len(logger.x)),
    interval=1000 / 60,
    blit=False,
).save("particle_evolution_learnable_2.gif", fps=120)
```

</details>

## Stein-Identity as a Stopping Criterion

In SVGD, the number of steps {::nomarkdown}$L${:/nomarkdown} for which the algorithm is run is specified is a hyperparameter that must be tuned for each target distribution {::nomarkdown}$p${:/nomarkdown}.
This is also typically the case for some MCMC algorithms such as Langevin Dynamics.
MET-SVGD on the other hand employs an adaptive number of steps {::nomarkdown}$L_c${:/nomarkdown} by checking at each step {::nomarkdown}$l${:/nomarkdown} whether convergence has been achieved by measuring the violation of Stein's Identity:

{::nomarkdown}
$$
SI(q^l_\theta, p)
=
\sqrt{
\mathbb{E}_{x^l \sim q^l_\theta}
\left[
\phi_\theta(x^{l})^\top
\nabla_{x^l} \log p(x^l)
+
\mathrm{Tr}\left(
\nabla_{x^l} \phi_\theta(x^l)
\right)
\right]
},
$$
{:/nomarkdown}


which only incurs a vector dot product to compute, as {::nomarkdown}$\nabla_{x^l} \log p(x^l)${:/nomarkdown} and {::nomarkdown}$\mathrm{Tr}\left( \nabla_{x^l}\phi_\theta(x^l) \right)${:/nomarkdown} have already been computed.

Based on this, MET-SVGD can be viewed as a normalizing flow model with a full-rank Jacobian and an adaptive number of layers, retaining computational efficiency without sacrificing expressivity.

## Divergence Control via Metropolis-Hastings

In many application, the unnormalized density {::nomarkdown}$\bar p${:/nomarkdown} may exhibit abrupt gradient changes or contain highly non-smoothness regions, which can lead to instability or divergence during sampling.

<figure id="non-smooth-evolution">
  <img src="/images/met-svgd/particle_evolution_non_smooth.gif" alt="Particles' evolution in the presence of non-smoothness.">
  <figcaption>Particles' evolution in the presence of non-smoothness.</figcaption>
</figure>

<details markdown="1"><summary>Code for [the figure](/met-svgd/optimizations/#non-smooth-evolution).</summary>

```python
from svgd.sampler import SVGD
from svgd.distributions import TorchDistribution
from svgd.kernels import RBF
from svgd.kernels.parameters import HeuristicKP
from svgd.lrs import ParameterLR
from svgd.callbacks import Logger

import torch
from torch.distributions import MultivariateNormal, Independent, Normal

import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

torch.manual_seed(0)

phi = torch.tensor(torch.pi / 4)
cos = torch.cos(phi)
sin = torch.sin(phi)
rotation = torch.tensor([[cos, -sin], [sin, cos]])
cov = torch.tensor([[1.0, 0.0], [0.0, 1e-2]])
cov = rotation @ cov @ rotation.T
target_distribution = TorchDistribution(MultivariateNormal(torch.zeros(2), cov))
initial_distribution = TorchDistribution(Independent(Normal(torch.zeros(2), 1.0), 1))
kernel = RBF(HeuristicKP("median"))
lr = ParameterLR(torch.tensor(0.1))
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
n_steps = 25
x, _, _ = svgd.sample(n_particles=n_particles, n_steps=n_steps)
x = x.detach()

bound = 3.0

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
    scatter.set_offsets(logger.x[frame])
    return (scatter,)


animation = FuncAnimation(
    fig,
    animate,
    len(logger.x),
    interval=1000 / 60,
    blit=False,
)
animation.save("particle_evolution_non_smooth.gif", fps=120)
```

</details>

MET-SVGD addresses this issue by introducing a principled divergence control mechanism based on Metropolis-Hastings [[1]](#ref-mh).
At each step {::nomarkdown}$l${:/nomarkdown}, the SVGD update is interpreted as a proposal distribution, yielding a proposed state {::nomarkdown}$\tilde x^{l+1}${:/nomarkdown}.
This proposal is accepted with probability {::nomarkdown}$\alpha^l${:/nomarkdown}, in which case {::nomarkdown}$x^{l+1} = \tilde x^{l+1}${:/nomarkdown}.
Otherwise, the proposal is rejected and the previous state is retained, i.e. {::nomarkdown}$x^{l+1} = x^l${:/nomarkdown}.

<figure id="non-smooth-evolution-mh">
  <img src="/images/met-svgd/particle_evolution_non_smooth_mh.gif" alt="Particles' evolution in the presence of non-smoothness with MH divergence control.">
  <figcaption>Particles' evolution in the presence of non-smoothness with MH divergence control.</figcaption>
</figure>

<details markdown="1"><summary>Code for [the figure](/met-svgd/optimizations/#non-smooth-evolution-mh).</summary>

```python
from svgd.sampler import SVGD
from svgd.distributions import TorchDistribution
from svgd.kernels import RBF
from svgd.kernels.parameters import HeuristicKP
from svgd.lrs import ParameterLR
from svgd.callbacks import Logger

import torch
from torch.distributions import MultivariateNormal, Independent, Normal

import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

torch.manual_seed(0)

phi = torch.tensor(torch.pi / 4)
cos = torch.cos(phi)
sin = torch.sin(phi)
rotation = torch.tensor([[cos, -sin], [sin, cos]])
cov = torch.tensor([[1.0, 0.0], [0.0, 1e-2]])
cov = rotation @ cov @ rotation.T
target_distribution = TorchDistribution(MultivariateNormal(torch.zeros(2), cov))
initial_distribution = TorchDistribution(Independent(Normal(torch.zeros(2), 1.0), 1))
kernel = RBF(HeuristicKP("median"))
lr = ParameterLR(torch.tensor(0.1))
logger = Logger(log_x=True)
logger.activated = True
svgd = SVGD(
    target_distribution=target_distribution,
    initial_distribution=initial_distribution,
    kernel=kernel,
    lr=lr,
    divergence_control="metropolis-hastings",
    callbacks=[logger],
)

n_particles = 100
n_steps = 25
x, _, _ = svgd.sample(n_particles=n_particles, n_steps=n_steps)
x = x.detach()

bound = 3.0

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
    scatter.set_offsets(logger.x[frame])
    return (scatter,)


animation = FuncAnimation(
    fig,
    animate,
    len(logger.x),
    interval=1000 / 60,
    blit=False,
)
animation.save("particle_evolution_non_smooth_mh.gif", fps=120)
```

</details>

To inherit the convergence guarantees of Metropolis–Hastings (MH), MET-SVGD augments the SVGD state with a simple auxiliary random variable {::nomarkdown}$r${:/nomarkdown} that can take one of two values, {::nomarkdown}$−1${:/nomarkdown} or {::nomarkdown}$1${:/nomarkdown}.
At each step {::nomarkdown}$l${:/nomarkdown}, {::nomarkdown}$r^{l+1}${:/nomarkdown} is sampled at random and determines how the proposal is constructed.
If the value is {::nomarkdown}$1${:/nomarkdown}, the SVGD transformation is applied in the usual forward direction; if it is {::nomarkdown}$−1${:/nomarkdown}, the inverse SVGD transformation is applied.
This construction allows MET-SVGD to inherit MH convergence guarantees while still leveraging the efficiency of SVGD.

When {::nomarkdown}$r${:/nomarkdown} is a Rademacher random variable, the probability of acceptance is computed as:


{::nomarkdown}
$$
\log \alpha^l_\theta
=
\min \left[
0,
\log \bar p (\tilde x^{l+1})
-
\log \bar p (x^{l})
+
r^{l+1}
\epsilon^{l}_{\theta_3}
\mathrm{Tr}\left(
\nabla_{x^l} \phi_\theta(x^l)
\right)
\right]
$$
{:/nomarkdown}


With the inclusion of the MH correction, the induced distribution given {::nomarkdown}$r^{l+1}${:/nomarkdown} becomes:

{::nomarkdown}
$$
q_\theta^{\text{MH},l+1}(x^{l+1}|r^{l+1}=c)
=
\alpha_\theta^l
q_\theta^{\text{MH},l}(x^l)
\left|\det \left(I + \epsilon^l_{\theta_3} \nabla_{x^l} \phi_\theta(x^l) \right)\right|^{-c}
+
(1-\alpha_\theta^l)
q_\theta^{\text{MH},l}(x^l),
$$
{:/nomarkdown}

and {::nomarkdown}$q_\theta^\text{MH,l+1}(x^{l+1})${:/nomarkdown} is obtained via marginalization, with {::nomarkdown}$p(r=1)=p(r=-1)=\frac12${:/nomarkdown}.

Given that MET-SVGD inherits the convergence guarantees of MH, {::nomarkdown}$q_\theta^\text{MH,L}${:/nomarkdown} converges to the target distribution {::nomarkdown}$p${:/nomarkdown} as {::nomarkdown}$L\to\infty${:/nomarkdown}, independently of the number of particles {::nomarkdown}$M${:/nomarkdown}.
By contrast, existing convergence guarantees for SVGD in the literature require both {::nomarkdown}$L,M\to\infty${:/nomarkdown} [[2]](#ref-svgdconv).

## References {#references}

1. <a id="ref-mh"></a>Luke Tierney. *Markov Chains for Exploring Posterior Distributions*. The Annals of Statistics (1994)
2. <a id="ref-svgdconv"></a>Sun, Lukang, Karagulyan, Avetik, Richtarik, Peter. *Convergence of Stein variational gradient descent under a weaker smoothness condition*. AISTATS (2023)

