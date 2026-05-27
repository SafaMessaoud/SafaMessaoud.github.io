---
layout: archive
title: "Projects"
permalink: /projects/
author_profile: true
---

{% include base_path %}

A selection of research projects across generative modeling, reinforcement learning, safe AI, healthcare analytics, and systems. Each item links to the corresponding publication or patent.

Generative Models & Reinforcement Learning
======
* **SVGD Scalability & Entropy Estimation.** Studied whether the entropy of arbitrary distributions known only up to a normalization constant can be tractably estimated via Stein Variational Gradient Descent.
  *(Under submission, 2025.)* — See [publication]({{ base_path }}/publication/2025-svgd-entropy).

* **S2AC: Stein Soft Actor-Critic.** Proposed a new variational distribution leveraging Stein Variational Gradient Descent dynamics that enables learning multi-modal policies for Max-Entropy Reinforcement Learning.
  *ICLR 2024.* — See [publication]({{ base_path }}/publication/2024-s2ac).

* **DeepQAMVS — Query-Aware Multi-Video Summarization.** A reinforcement-learning method that trains a pointer network with hierarchical attention, achieving state-of-the-art results on query-aware multi-video summarization.
  *ACM SIGIR 2021; DRL4IR 2021 (Oral).* — See [publication]({{ base_path }}/publication/2021-deepqamvs).

* **Learned Heuristics for Graphical Model Inference.** A reinforcement-learning engine for inference in energy-based models with traditionally intractable higher-order potentials, applied to semantic segmentation.
  *CVPR 2020 (Oral); WiCV 2020 (Oral); DeepVision 2020.* — See [publication]({{ base_path }}/publication/2020-graphical-model-rl).

* **Diverse & Controllable Image Colorization.** Extended Gaussian conditional random fields, traditionally unimodal and pairwise, to model multi-modal distributions with high-order dependencies — enabling exact inference and runtime constraints.
  *ECCV 2018; WiML 2017.* — See [publication]({{ base_path }}/publication/2018-diverse-colorization).

Large Language Models
======
* **Fanar — Arabic-Centric Multimodal Generative AI Platform.** Contributor to QCRI's Fanar platform, an Arabic-centric multimodal foundation model spanning language, speech, and vision.
  *arXiv 2501.13944, 2025.* — See [technical report]({{ base_path }}/publication/2025-fanar).

Safe & Robust AI (Adversarial Training)
======
* **Intrinsic Dimensionality in Adversarial Training.** Explains how the intrinsic dimensionality of the data manifold drives the robustness–generalization tradeoff in adversarially trained models.
  *ICML 2025.* — See [publication]({{ base_path }}/publication/2025-intrinsic-dim-adv).

* **A3T — Accuracy-Aware Adversarial Training.** Improves the robustness/generalization tradeoff by leveraging the data's intrinsic dimensionality and misclassification accuracy.
  *Machine Learning journal, 2023 — selected among top 3 papers.* — See [publication]({{ base_path }}/publication/2023-a3t).

* **Adversarial Training in Language Models.** Studies how adversarial training affects robustness and generalization in pretrained language models.
  *Findings of ACL 2022.* — See [publication]({{ base_path }}/publication/2022-adv-lm).

Healthcare Analytics
======
* **Disease-Based Problem List Generation.** Variants of autoencoders that learn customized features per disease category for problem-list generation from electronic medical records. *U.S. Patent.* — See [patent]({{ base_path }}/publication/patent-medical-record).

* **Transcriptomic Analysis of Alzheimer's.** Unsupervised technique to identify genes that discriminate temporal-cortex expression data of Alzheimer patients from control subjects.
  *AAIC 2017.* — See [publication]({{ base_path }}/publication/2017-alzheimer).

* **Reaction–Diffusion Modelling for Microphysiometry.** Extended a 3D finite-element simulation of diffusion and metabolic reaction in cellular specimens, with a model of the sensor effect, validated experimentally.
  *Med. Biol. Eng. Comput., 2013.* — See [publication]({{ base_path }}/publication/2013-reaction-diffusion).

Automated Tools for Cyber-Physical Systems
======
* **Aircraft Electrical Power Systems Synthesis.** Two optimization-oriented methodologies to synthesize cost-effective and reliable aircraft electrical power-system topologies: (1) Mixed Integer–Linear Programming modulo reliability; (2) ILP with an approximate reliability algebra. *M.S. thesis at UC Berkeley.*

* **SIMULINK → SIGNAL Translator.** A semantic translator from discrete-time SIMULINK models into SIGNAL programs, enabling correct-by-design and multi-threaded code generation.
  *e-TI, 2015.* — See [publication]({{ base_path }}/publication/2015-simulink-signal).

Computer Architecture (FPGA)
======
* **FPGA Accelerator for Genomic Data Parsing.** SAM → BAM conversion accelerated 10× over a single-threaded software baseline.
  *U.S. Patent.* — See [patent]({{ base_path }}/publication/patent-genomic-fpga).

* **FPGA Monitor Aggregator for "Invasive Computing".** A resource-aware scheduling monitor for multicore architectures, designed at TUM's Integrated Systems department.

Community & Education
======
* **[MenaML](https://www.menaml.com/) — Middle East and North Africa Machine Learning Winter School.** Co-founder and co-director of a regional ML education initiative that brings together students, researchers, and leading scientists across MENA.

* **Mentorship Platform.** Lead PI on a *Social Innovation Fund* project (QAR 50,000) building a platform for professional roadmap construction, sharing, and merging.
