---
title: "Long-horizon Planning with LLMs"
permalink: /projects/llm-long-term-planning/
excerpt: "Equipping LLMs with the ability to reason over long horizons beyond a single step generation."
image: /images/projects/topas-architecture.png
status: "Ongoing"
order: 2
---

<figure style="margin: 0.5rem 0 1.75rem 0;">
  <img src="{{ '/images/projects/topas-architecture.png' | relative_url }}"
       alt="TOPAS architecture: textbooks and unlabeled data feed the TOPAS framework, which builds a proactive agentic system comprising a hierarchical planner, a graph-based memory, a dialog LLM, and a user simulator."
       style="width:100%; height:auto; border:1px solid #e7e3d8; border-radius:8px;">
  <figcaption style="margin-top:0.5rem; font-size:0.85rem; font-style:italic; color:#777; text-align:center;">
    TOPAS builds a structured agentic system for long-horizon dialogue planning from intervention-oriented domain textbooks.
  </figcaption>
</figure>

TOPAS: Textbooks to Proactive Agentic Systems
======

Large language models (LLMs) excel at generating locally coherent responses but remain fundamentally *reactive*, lacking the ability to plan and steer interactions over long horizons. Existing approaches attempt to induce proactivity through prompting or reinforcement learning with verifiable rewards (RLVR), which we show degrade on long-horizon dialogue tasks. A natural solution is to augment LLMs with planners, memory, and user simulators — however, specifying these components and their interactions requires substantial expert design effort, thereby limiting scalability.

We address this challenge by introducing **TOPAS** (**T**extbooks t**O** **P**roactive **A**gentic **S**ystems), a framework for automatically constructing executable agentic systems from intervention-based domain textbooks. Given unstructured instructional textbooks, TOPAS jointly derives:

- a **hierarchical planner** over a structured decision space (states, actions, and rewards),
- a **user simulator**, and
- a **graph-based memory** structure,

thereby defining a complete interaction loop for training and evaluation. It further generates structured supervision via **uncertainty-aware self-annotation** and learns hierarchical policies over the extracted structure using **reinforcement learning**.

Experiments on **psychotherapy** and **persuasive dialogue** benchmarks show that TOPAS achieves state-of-the-art performance across multiple metrics and enables planning over longer horizons — outperforming prompting-based, RLVR, and manually designed planning approaches.

*This work prioritizes safety and human oversight; it is intended to augment, not replace, trained professionals. Details and publications will be added as the project develops.*
