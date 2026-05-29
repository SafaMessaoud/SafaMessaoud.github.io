---
title: "Long-horizon Planning with LLMs"
permalink: /projects/llm-long-term-planning/
excerpt: "Equipping LLMs with the ability to reason over long horizons beyond a single step generation."
image: /images/projects/topas-architecture.png
status: "Ongoing"
order: 2
---

Modern large language models are excellent at single-step generation but struggle to **reason over long horizons** — to plan a sequence of actions, anticipate how a conversation will unfold, and adjust course based on what happens many steps later. This ongoing project develops methods that equip LLMs with explicit planning, search, and credit-assignment mechanisms so they can act as **goal-directed agents** rather than next-token predictors.

We study these questions in the context of **mental-health support**, where long-horizon reasoning matters most: a supportive agent must hold a goal over an entire conversation (and across sessions), choose what to say now in light of where the interaction should go, and adapt gently as the person's state changes — never optimizing for a single reply in isolation.

Directions we are exploring include:

- Planning and search over multi-turn dialogue, guided by learned value functions that reflect long-term wellbeing rather than immediate engagement.
- Reinforcement learning over long-horizon trajectories with sparse, delayed feedback.
- Safeguards and credit assignment that keep an agent's long-term plan aligned with the person's needs.

*This work prioritizes safety and human oversight; it is intended to augment, not replace, trained professionals. Details and publications will be added as the project develops.*
