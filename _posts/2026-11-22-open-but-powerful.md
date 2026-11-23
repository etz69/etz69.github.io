---
layout: "post"
title: "Open but powerful"
date: 2026-11-22 12:29:40 +0000
description: "Open-weight models offer unusual scale and flexibility, but using them well means understanding their architecture, limits and licenses."
categories: ["builders"]
tags: ["ai", "llm", "open-source", "research"]
---

# Open but powerful

Open-weight models can be large, capable systems—not just smaller substitutes for hosted AI. But access to model weights is only a starting point. To use these models well, teams need to understand what is actually released, what resources the model needs, and what its license permits.

## The idea

“Open-weight” means that a model’s trained weights are available. It does not automatically mean that its training data, training code or evaluation process is also available. Nor does it mean every model has the same terms of use.

The difference matters in practice. A team evaluating a model for an agent needs more than a headline benchmark or a parameter count. It needs to know whether the model fits its hardware, performs on its task, handles the required context, and can be used under its license.

## How it works

One reason some open-weight models can have very large total parameter counts is a design called a **mixture of experts** (MoE). Instead of using every part of the model for every token, an MoE model routes each token through a subset of its internal experts. The research report compares models by both total parameters and active parameters—the approximate number used per token.

For example, the report lists DeepSeek-V3 at 671 billion total parameters and 37 billion active parameters. It lists Llama 4 Maverick at 400 billion total and 17 billion active. OpenAI’s gpt-oss-120b is listed at about 116.83 billion total and 5.13 billion active. Those figures describe different aspects of a model; total parameters alone do not tell you the compute required for every step, or whether a model will run efficiently in a particular deployment.

Other models in the report are dense: Qwen3-32B and Gemma 3 27B have the same total and active parameter counts. This can make their resource requirements more predictable than an MoE model’s, though actual performance and operating cost still depend on the hardware and setup.

Context length is another headline number that needs interpretation. The report lists a 128K-token context for gpt-oss and DeepSeek-V3, and a 10-million-token advertised context for Llama 4 Scout. That spread is large enough to matter. A chart of the flagship models’ context lengths makes the contrast easy to see. But a large advertised window is not a guarantee that a model will use every part of a long input equally well. The report cautions that effective context can be lower and depends on the model and inference setup.

## What it means

**Choose for the job, not the biggest number.** A model with many total parameters may activate only a fraction per token. A dense model may be easier to reason about operationally. Neither fact alone determines which is best. Test candidates on representative tasks, with the prompts, tools and data your agent will actually use.

**Treat license as part of the technical decision.** The report lists gpt-oss under Apache 2.0, while Llama 4 uses the Llama 4 Community License and Gemma 3 uses Google’s terms. These are not interchangeable. Before deployment, check the exact license and any applicable usage terms for the specific model version. “Weights available” is not the same as “no conditions.”

**Read benchmark comparisons carefully.** The report finds that benchmark coverage is sparse and drawn from mixed sources. Scores may use different methods or versions, so they should not be read as a clean head-to-head ranking across model families. Benchmarks can help narrow a shortlist. Evaluation on your own tasks should decide what you deploy.

## Why it matters

Open weights create room to choose where and how a model runs. That flexibility can be valuable for teams building agents, but it comes with responsibility: understand the model’s architecture, test its behavior in the intended setup, and check the terms before putting it into an operation.

The useful question is not simply “How powerful is this model?” It is “What can this version do for our task, under our constraints, and how will we know?”

## Sources

- [Open-Weight LLM Flagship Landscape: Key Comparisons](workspace research report, revision 3; prepared in this research space)
- [Open-Weights LLM Release History and Timeline](https://hidekazu-konishi.com/entry/open_weights_llm_release_history_and_timeline.html)
