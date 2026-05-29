---
title: "MET-SVGD in a Nutshell"
nav_title: "Introduction"
permalink: /met-svgd/introduction/
nav_order: 1
---

## Motivation and Problem Setup

We consider the setting where we only have access to a target density {::nomarkdown}$p${:/nomarkdown} known up to a normalization constant

{::nomarkdown}
$$
p(x) = \frac{\bar{p}(x)}{Z},
$$
{:/nomarkdown}

where {::nomarkdown}$\bar{p}${:/nomarkdown} is the unnormalized density and {::nomarkdown}$Z${:/nomarkdown} is the normalization constant, which is generally intractable to compute.

Our aim is to perform a range of inference tasks with respect to {::nomarkdown}$p${:/nomarkdown}, such as:
- generating high-quality samples
- estimating {::nomarkdown}$p(x)${:/nomarkdown} for a given sample {::nomarkdown}$x${:/nomarkdown}
- estimating the entropy of {::nomarkdown}$p${:/nomarkdown}

To accomplish these goals, in practice, we typically rely on a parameterized sampling mechanism, such as a learned sampler, a neural transformation, or a parameterized particle update rule.
The quality of inference hinges on how well this mechanism can represent the target density.

## Problem Significance

This setup arises frequently in machine learning.
A motivating example is maximum-entropy reinforcement learning (MaxEnt RL), where policies are defined through unnormalized energy-based distributions over actions.

Policies trained under the maximum-entropy reinforcement learning framework tend to be more robust, as the agent learns to capture multiple modes of high-reward behavior rather than committing to a single deterministic trajectory.
Consequently, if the environment or the state is perturbed at test time, the agent is more likely to recover by exploiting alternative high-reward strategies.

<figure id="rl-agent">
<div class="ms-figrow">
<figure id="rl-agent-train">
  <img src="/images/met-svgd/figure_2a_maze_one_path.png" alt="Environment at train time.">
  <figcaption>Environment at train time.</figcaption>
</figure>
<figure id="rl-agent-test">
  <img src="/images/met-svgd/figure_2b_maze-two-paths.png" alt="Environment at test time.">
  <figcaption>Environment at test time.</figcaption>
</figure>
</div>
<figcaption><a href="https://bair.berkeley.edu/blog/2017/10/06/soft-q-learning/.">https://bair.berkeley.edu/blog/2017/10/06/soft-q-learning/.</a></figcaption>
</figure>

This is illustrated in the figure above, where the test time environment includes an additional obstacle that the agent hasn't seen during training.
A standard RL agent that has learned a deterministic policy would not be able to reach the goal, whereas a MaxEnt RL agent would be able to find the lower passage towards the goal.

## The Challenge

The core difficulty lies in the fact that the normalization constant {::nomarkdown}$Z${:/nomarkdown} is unknown, which renders many standard inference methods inapplicable.
While {::nomarkdown}$Z${:/nomarkdown} can be computed in closed form for certain distributions, such as Gaussians, this is generally not feasible for more complex distributions.

Some methods attempt to approximate {::nomarkdown}$Z${:/nomarkdown}, for example via importance sampling, but the variance of these estimates tends to grow with dimensionality, limiting their practicality.

Traditional MCMC methods (e.g., HMC, Langevin dynamics) bypass the normalization constant entirely by using the score function {::nomarkdown}$\nabla_x \log p(x) = \nabla_x \log \bar{p}(x)${:/nomarkdown}. However, these methods require careful hyperparameter tuning, produce only samples, and often need many iterations to yield high-quality results.

Normalizing flows, by contrast, provide both samples and densities, which allows direct estimation of {::nomarkdown}$p(x)${:/nomarkdown} for a generated sample {::nomarkdown}$x${:/nomarkdown}, and hence also of {::nomarkdown}$\mathcal{H}(p)${:/nomarkdown}.
Yet, they do not directly leverage the unnormalized density {::nomarkdown}$\bar{p}${:/nomarkdown}, which limits their expressivity, and are prone to issues such as mode collapse.

What we ultimately seek is a method that constructs a distribution that:
- is expressive enough to capture complex, multimodal targets
- utilizes the unnormalized density {::nomarkdown}$\bar{p}${:/nomarkdown}
- is computationally tractable
- allows efficient sampling

## MET-SVGD

Metropolis-Hastings Stein Variational Gradient Descent (MET-SVGD) satisfies the above criteria by extending Parameterized SVGD (P-SVGD) [[1]](#ref-s2ac), a particle-based parametric variational inference method based on SVGD [[2]](#ref-svgd) that derives a closed-form expression of the SVGD-induced density.

MET-SVGD bridges the gap between Stein Variational Gradient Descent (SVGD) [[2]](#ref-svgd), parametric variational inference (P-VI), and Metropolis-Hastings (MH), inheriting the strengths of each:
- the ability to approximate arbitrarily complex distributions, convergence detection, and particle efficiency form SVGD
- scalability from P-VI
- convergence guarantees from MH

<figure id="bridge-image">
  <img src="/images/met-svgd/bridge.png" alt="MET-SVGD bridges the gap between P-VI, SVGD, and MCMC methods." style="max-width:300px;">
  <figcaption>MET-SVGD bridges the gap between P-VI, SVGD, and MCMC methods.</figcaption>
</figure>

<figure id="bridge-table">
<table>
<thead><tr><th></th><th>P-VI</th><th>MCMC</th><th>SVGD</th><th>P-SVGD</th><th>MET-SVGD</th></tr></thead>
<tbody>
<tr><td>Expressivity</td><td>✗</td><td>✓</td><td>✓</td><td>✓</td><td>✓✓</td></tr>
<tr><td>Convergence Detection</td><td>✓</td><td>✗</td><td>✓</td><td>✓</td><td>✓</td></tr>
<tr><td>Convergence Guarantees</td><td>✗</td><td>✓</td><td>✗</td><td>✗</td><td>✓</td></tr>
<tr><td>Sampling Efficiency</td><td>✓</td><td>✗</td><td>✓</td><td>✓</td><td>✓</td></tr>
<tr><td>Tractable Entropy</td><td>✓</td><td>✗</td><td>✗</td><td>✓</td><td>✓</td></tr>
<tr><td>Parameter Efficiency</td><td>✓</td><td>—</td><td>—</td><td>✓✓</td><td>✓✓</td></tr>
</tbody></table>
<figcaption>MET-SVGD inherits the advantages of different approximate inference methods</figcaption>
</figure>

In addition, MET-SVGD unprecedentedly scales SVGD to high-dimensional spaces, while retaining computational efficiency.

Moreover, unlike traditional approaches that rely on grid search for hyperparameter tuning, MET-SVGD enables end-to-end learning of sampler parameters via KL-divergence minimization, solving a long-standing challenge in machine learning.

Finally, MET-SVGD can be viewed as a full-rank Jacobian normalizing flow model with an adaptive number of layers controlled by a convergence check, ensuring flexibility and expressivity.

## References {#references}

1. <a id="ref-s2ac"></a>Messaoud, Safa, Mokeddem, Billel, Xue, Zhenghai, Pang, Linsey, An, Bo, Chen, Haipeng, Chawla, Sanjay. *S 2 AC: Energy-Based Reinforcement Learning with Stein Soft Actor Critic*. ICLR (2024)
2. <a id="ref-svgd"></a>Liu, Qiang, Wang, Dilin. *Stein variational gradient descent: A general purpose bayesian inference algorithm*. NeurIPS (2016)

