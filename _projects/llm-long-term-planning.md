---
title: "SakinaSim"
permalink: /projects/llm-long-term-planning/
excerpt: "A simulation platform for training mental-health providers — leveraging long-horizon LLM reasoning to evaluate whole sessions and give feedback on performance."
image: /images/projects/topas-architecture.png
status: "Ongoing"
order: 2
---

**SakinaSim** is a training platform for mental-health providers. Trainees practice with a realistic, AI-simulated patient, and an AI **evaluator** observes the session and gives structured feedback on their performance — across dimensions such as content, cultural sensitivity, risk and safety, and diagnostic reasoning.

What makes this hard — and what the project is really about — is **long-horizon reasoning**. Good clinical feedback can't be given turn-by-turn in isolation; it depends on the *whole* trajectory of a session: how the conversation was opened, how rapport built, whether risk was assessed at the right moment, and how the trainee adapted as the patient's state changed. SakinaSim leverages the long-horizon capabilities of large language models to assess this full arc and surface feedback a supervisor would give.

Directions we are exploring include:

- Evaluating multi-turn sessions as a whole, with models that reason over the entire trajectory rather than scoring single replies.
- Structured, multi-dimensional feedback — content, stylistic, cultural, risk & safety, diagnostic, and alternative actions.
- Realistic patient simulation grounded in clinically meaningful cases.

*This work prioritizes safety and human oversight; it is intended to augment supervised clinical training, not replace trained professionals. Details and publications will be added as the project develops.*
