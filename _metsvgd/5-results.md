---
title: "Results"
nav_title: "Results"
permalink: /met-svgd/results/
nav_order: 5
---

## Energy-Based Models

Energy-based models (EBMs) are generative models that learn a density {::nomarkdown}$p_\phi${:/nomarkdown} parametrized by an energy function {::nomarkdown}$f_\phi${:/nomarkdown}, so that {::nomarkdown}$p_\phi(x) = \bar p_\phi(x) / Z_\phi${:/nomarkdown}, with {::nomarkdown}$\bar p_\phi(x)=\exp(f_\phi(x))${:/nomarkdown}.
Typically, EBMs are trained via maximum likelihood estimation (MLE) by estimating the gradient of the maximum likelihood loss, since direct computation of the loss is not feasible due to the intractability of {::nomarkdown}$Z_\phi${:/nomarkdown}.
However, when a sampler with a tractable density {::nomarkdown}$q_\theta${:/nomarkdown} is given, a tight lower bound on the MLE loss can be computed:

{::nomarkdown}
$$
\mathcal{L}_\text{ELBO}(\phi, \theta)
=
\mathbb{E}_{x \sim q_\theta}\left[ \log \bar p_\phi(x) \right]
-
\mathbb{E}_{x \sim p_\text{data}}\left[ \log \bar p_\phi(x) \right]
+
\mathcal{H}(q_\theta).
$$
{:/nomarkdown}


The EBM is then trained by alternating between maximizing {::nomarkdown}$\mathcal{L}_\text{ELBO}(\phi, \theta)${:/nomarkdown} with respect to {::nomarkdown}$\theta${:/nomarkdown} to further tighten the lower bound, and minimizing it with respect to {::nomarkdown}$\phi${:/nomarkdown}.

In [the figure](/met-svgd/results/#cifar), we show the Fréchet Inception Distance (FID) averaged over 5 random seeds for CIFAR10 image generation.
We compare against P-SVGD [[1]](#ref-s2ac) and Glow (Normalizing Flow) [[2]](#ref-glow).

<figure id="cifar">
  <img src="/images/met-svgd/fid.svg" alt="FID on CIFAR-10; bold marks changes between configurations.">
  <figcaption>FID on CIFAR-10; bold marks changes between configurations.</figcaption>
</figure>

Removing either the [trace-of-Hessian term](/met-svgd/density/#sec3-tr) or the step-size bound causes the training to diverge, as the violet and gray curves show.
[the figure](/met-svgd/results/#cifar-score) shows that adding the trace-of-Hessian term produces smoother energy landscapes that are much easier to sample from.
The step-size bound plays a different but equally important role, as it ensures that the entropy estimation remains valid.

<figure id="cifar-score">
  <img src="/images/met-svgd/cifar_score.svg" alt="Smoothness of the learnt EBM as measured by $||\nabla \log \bar p_\phi||$ throughout training iterations. The configurat">
  <figcaption>Smoothness of the learnt EBM as measured by $||\nabla \log \bar p_\phi||$ throughout training iterations. The configurations with the best FID ([](#cifar)) exhibit smoother landscapes.</figcaption>
</figure>

Replacing the median heuristic bandwidth, {::nomarkdown}$\sigma_\text{med}${:/nomarkdown}, with a learnable bandwidth greatly improves stability and leads to significantly better FID scores compared to P-SVGD (green versus orange).
As shown in [the figure](/met-svgd/results/#cifar-sigma), the median heuristic bandwidth is on average more than an order of magnitude larger than the learned value {::nomarkdown}$\sigma_{\theta_2}${:/nomarkdown}.
This overly large bandwidth causes particles to become spuriously correlated, hurting sample quality.

Learning the step size as well (red curve) allows the method to converge to the target distribution much faster.
[the figure](/met-svgd/results/#cifar-step) illustrates that the learned step size {::nomarkdown}$\epsilon_{\theta_3}${:/nomarkdown} is much larger than the fixed step size {::nomarkdown}$\epsilon${:/nomarkdown}.

<figure id="sigma-step-legend">
  <img src="/images/met-svgd/cifar_sigma_step_legend.svg" alt="">
  
</figure>

<figure id="cifar-sigma-step">
<div class="ms-figrow">
<figure id="cifar-sigma">
  <img src="/images/met-svgd/cifar_sigma.svg" alt="Kernel bandwidth across training iterations.">
  <figcaption>Kernel bandwidth across training iterations.</figcaption>
</figure>
<figure id="cifar-step">
  <img src="/images/met-svgd/cifar_step.svg" alt="Step-size across training iterations">
  <figcaption>Step-size across training iterations</figcaption>
</figure>
</div>
<figcaption>Learnt kernel bandwidth and step-size across training iterations. The trends exhibited by the learnt kernel bandwidth and step-size in MET-SVGD configurations are significantly different from $\sigma_\text{med}$ and constant $\epsilon$.</figcaption>
</figure>

Using an adaptive number of update steps {::nomarkdown}$L_c${:/nomarkdown} further stabilizes training, as shown by the brown curve.

In contrast to the other MET-SVGD optimizations, experiments that incorporate MH diverge, as indicated by the pink curve.
This instability is not inherent to MET-SVGD itself, but instead comes from the interaction between MH rejection dynamics and a the loss function, {::nomarkdown}$\mathcal{L}_\text{ELBO}${:/nomarkdown}, not being lower-bounded.
The stability of the training objective depends critically on the quality of the samples used to estimate the expectation with respect to the sampler.
When the energy landscape is highly complex, MH acceptance rates can become very low [[3]](#ref-cifar-rej), preventing particles from moving toward high-density regions of the target distribution.
As a result, sample quality degrades and training eventually diverges.

<figure id="cifar-rej">
  <img src="/images/met-svgd/cifar_mh_rej.svg" alt="MH rejection rate across training iterations.">
  <figcaption>MH rejection rate across training iterations.</figcaption>
</figure>

Glow-NF exhibits a similar failure mode, struggling to produce good samples early in training and ultimately diverging as well.

## Maximum Entropy Reinforcement Learning

MaxEnt RL learns a stochastic policy over actions given a state by maximizing the sum of expected rewards and entropies:

{::nomarkdown}
$$
\pi^*_\theta
=
\underset{\pi_\theta}{\mathrm{argmax}}
\sum_t
\mathbb{E}_{(s_t, a_t)} \left[
r(s_t, a_t)
+
\alpha \mathcal{H}(\pi_\theta(\cdot | s_t))
\right]
$$
{:/nomarkdown}


We model the sampler and estimate the entropy using the MET-SVGD framework and compare against SAC [[4]](#ref-sac), which models the policy as a diagonal Gaussian, SAC-NF, which models the policy as an auto-regressive normalizing flow [[5]](#ref-maf), and P-SVGD [[1]](#ref-s2ac).
In [the figure](/met-svgd/results/#walker), we report the average return on {::nomarkdown}$10${:/nomarkdown} rollouts every {::nomarkdown}$1000${:/nomarkdown} steps on Walker2d-v2 averaged over {::nomarkdown}$5${:/nomarkdown} seeds.

<figure id="walker-legend">
  <img src="/images/met-svgd/walker_return_legend.svg" alt="">
  
</figure>

<figure id="walker">
<div class="ms-figrow">
<figure id="walker-curve">
  <img src="/images/met-svgd/walker_return_curve.svg" alt="Return IQM across training iterations.">
  <figcaption>Return IQM across training iterations.</figcaption>
</figure>
<figure id="walker-bar">
  <img src="/images/met-svgd/walker_return_bar.svg" alt="Return IQM at the end of training.">
  <figcaption>Return IQM at the end of training.</figcaption>
</figure>
</div>
<figcaption>Return IQM on Walker2d-v2.</figcaption>
</figure>

Learning the kernel bandwidth (orange) and adding the trace-of-Hessian correction (green) both lead to substantial improvements over the P-SVGD baseline.
In contrast, removing the bound on the step size (pale green) causes the method to diverge, which is expected.

Learning the step size without the trace-of-Hessian term (peach) performs worse than all baselines.
In this case, particles tend to drift into non-smooth regions of the landscape [[6]](#ref-walker-score).

<figure id="walker-score">
  <img src="/images/met-svgd/walker_score.svg" alt="Smoothness as measured by $||\nabla \log \bar p_\phi||$ throughout training iterations.">
  <figcaption>Smoothness as measured by $||\nabla \log \bar p_\phi||$ throughout training iterations.</figcaption>
</figure>

The best results come from incorporating MH with an {::nomarkdown}$\epsilon${:/nomarkdown}-greedy schedule (purple):
applying MH with high probability later in training and low probability early on helps balance exploration in the beginning with exploitation as learning progresses.
These improvements become even more pronounced when we increase the number of particles from 10 to 64 (brown).

## References {#references}

1. <a id="ref-s2ac"></a>Messaoud, Safa, Mokeddem, Billel, Xue, Zhenghai, Pang, Linsey, An, Bo, Chen, Haipeng, Chawla, Sanjay. *S 2 AC: Energy-Based Reinforcement Learning with Stein Soft Actor Critic*. ICLR (2024)
2. <a id="ref-glow"></a>Kingma, Durk P, Dhariwal, Prafulla. *Glow: Generative flow with invertible 1x1 convolutions*. NeurIPS (2018)
3. <a id="ref-cifar-rej"></a>.  ()
4. <a id="ref-sac"></a>Haarnoja, Tuomas, Zhou, Aurick, Abbeel, Pieter, Levine, Sergey. *Soft actor-critic: Off-policy maximum entropy deep reinforcement learning with a stochastic actor*. ICML (2018)
5. <a id="ref-maf"></a>Papamakarios, George, Pavlakou, Theo, Murray, Iain. *Masked autoregressive flow for density estimation*. NeurIPS (2017)
6. <a id="ref-walker-score"></a>.  ()

