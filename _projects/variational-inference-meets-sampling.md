---
title: "Variational Inference Meets Sampling"
permalink: /projects/variational-inference-meets-sampling/
excerpt: "Bridging variational inference and sampling-based methods for scalable, multi-modal posterior approximation."
image: /images/projects/vi-sampling.svg
status: "Ongoing"
order: 3
---

This ongoing project explores the interface between **variational inference** and **sampling-based methods** (MCMC, Stein Variational Gradient Descent, diffusion-based samplers). The goal is to design algorithms that combine the speed and amortization of VI with the asymptotic correctness and multi-modality of sampling.

Topics we are currently investigating include:

- Scalable entropy estimation for unnormalized distributions via SVGD dynamics.
- Energy-based reinforcement learning with Stein-style policies (see *S2AC*, ICLR 2024).
- Connections between score-based diffusion samplers and amortized variational methods.

Relevant publications:

- [Can We Compute The Entropy Of Arbitrary Distributions Known Up To A Normalization Constant? A Tale of SVGD Scalability]({{ "/publication/2025-svgd-entropy" | relative_url }}) — *Under submission, 2025.*
- [S2AC: Energy-Based Reinforcement Learning with Stein Soft Actor Critic]({{ "/publication/2024-s2ac" | relative_url }}) — *ICLR 2024.*
