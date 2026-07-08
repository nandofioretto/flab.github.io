---
layout: page
title: Massively Speeding up LLM inference with Speculative Diffusion Decoding
description: Faster LLM inference with discrete diffusion drafting and alignment.
img: assets/img/projects/specdiff2.jpeg
importance: 2
highlight: true
category: work
giscus_comments: true
---

## Problem Statement and Motivation

Modern Large Language Models (LLMs) are marvels of reasoning, yet they suffer from a persistent architectural bottleneck: they are "autoregressive." Every word generated is the result of a sequential dependency chain where the model must finalize token n before it can even begin to calculate token n+1. This "one-token-at-a-time" pace creates a significant latency tax, making high-reasoning models feel sluggish and limiting the depth of "thinking" an AI can perform within a fixed time budget.

To circumvent this, Leviathan et al. introduced **Speculative Decoding** ([Leviathan et al., 2023](https://arxiv.org/abs/2211.17192)), a technique that uses a small "drafter" model to guess several tokens ahead, which a larger "verifier" model then checks in parallel. If the drafter is accurate, the system leaps forward; if not, it restarts. However, existing approaches face two critical bottlenecks:

- **Autoregressive Dependency in Drafting**: Most drafter models are themselves autoregressive, generating tokens sequentially and limiting the parallelism benefits. This sequential dependency adds latency to the drafting process, especially for longer draft sequences.
- **Drafter-Verifier Misalignment**: When the drafter's proposals don't align well with the verifier's preferences, many tokens get rejected, requiring regeneration and negating potential speed gains. This misalignment is particularly problematic when using diffusion-based drafters, as they learn joint distributions that can be miscalibrated with autoregressive verifiers.

Two works from RAISELab, **SpecDiff** ([Christopher et al., 2025](https://arxiv.org/abs/2408.05636)) and **SpecDiff-2** ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)), have addressed these bottlenecks with a fundamental new methodology. The idea is to replace sequential drafters with discrete diffusion models which generate tokens in parallel, allowing decoupling generation speed from sequence length. The work also introduces novel alignment mechanisms to improve acceptance rates.


### Takeaways

<div class="takeaways-grid">
    <div class="takeaway-card">Discrete diffusion drafters enable genuine parallel multi-token generation.</div>
    <div class="takeaway-card">Alignment should optimize accepted streaks, not just token-level accuracy.</div>
    <div class="takeaway-card">Faster inference increases reasoning quality within fixed wall-time budgets.</div>
    <div class="takeaway-card">Position-wise marginals enable low-cost draft self-selection at test time.</div>
</div>


<div class="project-figure">
    {% include figure.html path="assets/img/projects/specdiff.png" title="SpecDiff speed-ups across verifier models" class="img-fluid rounded z-depth-1 project-figure__img" %}

<div class="caption">
    <strong>Figure 1</strong>: SpecDiff replaces the autoregressive drafter with a non-autoregressive diffusion model. Instead of generating tokens sequentially, a diffusion drafter generates an entire block of draft tokens simultaneously. This is achieved by starting with ``masked'' tokens and denoising them into a coherent sequence in a small, fixed number of steps.
</div>
</div>



### Parallelizing the "Thinker"

The core innovation of [SpecDiff](https://arxiv.org/abs/2408.05636) ([Christopher et al., 2025](https://arxiv.org/abs/2408.05636)), which was presented at **NAACL 2025**, is the move from sequential to parallel drafting. In a traditional setup, generating a draft of 32 tokens requires 32 sequential passes through the drafter. SpecDiff replaces this with a discrete diffusion model that generates the entire block of tokens simultaneously through global denoising.

For the reader, the "so what" here lies in the relationship between $$T$$ (diffusion steps) and $$\gamma$$ (the draft window size). In autoregressive models, latency scales linearly with $$\gamma$$. In SpecDiff, latency is tied to the number of diffusion steps ($$T$$), which can be as low as 1 or 2 steps. This allows the system to scale $$\gamma$$ to 32 tokens or more without a linear time penalty. As shown in SpecDiff, modern diffusion models can require 32x fewer function evaluations than autoregressive models to produce text ([Christopher et al., 2025](https://arxiv.org/abs/2408.05636)).

This architecture led to a staggering 7.2x speedup on the OpenWebText benchmark (using GPT-NEO) and up to 5.45x on summarization tasks, proving that "global thinking" is inherently faster than "local guessing" ([Christopher et al., 2025](https://arxiv.org/abs/2408.05636)).


### The Alignment Bottleneck: The Size Paradox

While SpecDiff-1 solved the speed problem, it exposed a second challenge: Misalignment. Diffusion models look at global patterns (joint distributions), while autoregressive verifiers look at local prefix-conditional probabilities. If the drafter’s "global" vision doesn't align with the verifier’s "local" logic, the verifier rejects the draft, and speed gains evaporate.

|-----------------------------------------------------------------------------------------------------------|
| Bottleneck                | 	Technical Driver                    |	Consequence                         |
|-----------------------------------------------------------------------------------------------------------|
| **Drafter Latency**	        | Sequential dependency of AR drafters. | 	Drafting takes too long relative to verification. |
| **Drafter-Verifier Alignment** | Statistical divergence between global denoising and local conditionals.	| Frequent rejections force the model back into slow sequential mode. |
|-----------------------------------------------------------------------------------------------------------|

This leads to a paradox addressed in [SpecDiff-2](https://arxiv.org/abs/2511.00606) ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)). Traditionally, drafters are tiny (e.g., 80M parameters) to remain faster than the verifier. But SpecDiff-2 uses a 7B parameter drafter (like DiffuLLaMA) to accelerate a 70B model. In an autoregressive world, a 7B drafter would be too slow to be useful. Because diffusion is parallel, we can use "Large Drafters" that are more intelligent and better aligned without sacrificing the speed advantage.


### Discrete diffusion for parallel drafting

SpecDiff-2 employs Masked-Discrete Diffusion Models (MDMs) as drafters to eliminate autoregressive dependencies ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)). Unlike traditional autoregressive models, diffusion models generate text by iteratively refining corrupted sequences over a fixed number of denoising steps. Critically, each denoising step updates all token positions in parallel.

The process works by extending a prefix $$s$$ with $$\gamma$$ masked tokens [MASK], then using a single denoising step to generate a joint proposal:

$$
x_{1:\gamma} \sim Q_{diff}(\cdot \mid s \circ [MASK]_{1:\gamma})
$$

This parallel generation means drafting cost depends primarily on the fixed number of denoising steps rather than the draft length, providing substantial efficiency gains for longer sequences.

### Dual alignment strategy

<div class="project-figure">
    <div class="row">
        <div class="col-md-6 mt-3 mt-md-0">
            {% include figure.html path="assets/img/projects/specdiff2-fig2.jpeg" title="Train-time streak-distillation" class="img-fluid rounded z-depth-1 project-figure__img" %}
        </div>
        <div class="col-md-6 mt-3 mt-md-0">
            {% include figure.html path="assets/img/projects/specdiff2-fig3.jpeg" title="Test-time self-selection" class="img-fluid rounded z-depth-1 project-figure__img" %}
        </div>
    </div>
</div>
<div class="caption">
    <strong>Figure 2 (left)</strong>: Train-time streak-distillation aligns the diffusion drafter with the verifier by
    optimizing for long accepted token sequences across the entire draft window.
    <strong>(Right)</strong>:
    Test-time self-selection leverages diffusion parallelism to generate multiple draft candidates
    and select the most promising one using verifier-based scoring.
</div>

**Train-time streak-distillation.** 
The drafter is optimized for long accepted streaks, not just individual token likelihoods, using a greedy acceptance proxy
$$\Pr(accept\ x_i \mid s) = P(x_i \mid s)$$. In practice, the objective treats the verifier as a
teacher and encourages the diffusion drafter to generate sequences that maximize expected
committed tokens across the full draft window. This shifts training away from myopic
token-level matches and toward sequences that the verifier is likely to accept end-to-end,
which directly improves throughput.

$$
    	ext{Tokens}_{\text{Draft}}(\gamma, s) = \mathbb{E}_{x \sim Q_{diff}(\cdot \mid s)}\left[\sum_{i=1}^{\gamma} \prod_{j=1}^{i} P(x_j \mid s_{<j})\right]
$$

The training uses verifier-guided trajectories to estimate gradients, reinforcing long
accepted streaks instead of only the first few tokens. The result is a drafter that is
better calibrated to the verifier across the entire draft horizon.


**Test-time self-selection acceptance.** Position-wise marginals allow sampling $$K$$ joint drafts from a single denoising pass. Candidates are ranked using a streak-oriented verifier score. Because diffusion updates all positions in parallel, multiple candidate drafts can be generated with minimal overhead, and the highest-scoring draft is selected for verification to maximize expected committed tokens ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)).

$$
    	ext{Score}(x^{(k)}) = \sum_{i=1}^{\gamma} \prod_{j=1}^{i} P(x^{(k)}_j \mid s_{<j})
$$

This test-time selection complements training alignment: higher draft diversity at larger $$K$$ improves acceptance rates without changing the verifier or sacrificing output quality.


## Key Results and Performance


<div class="project-figure project-figure--float-right">
    {% include figure.html path="assets/img/projects/specdiff2.jpeg" title="SpecDiff-2 speed-ups across verifier models" class="img-fluid rounded z-depth-1 project-figure__img" %}

<div class="caption">
    <strong>Figure 2</strong>: SpecDiff-2 achieves superior speed-ups compared to existing speculative decoding
    methods across different verifier models, with improvements of over 30% compared to EAGLE-2.
</div>
</div>

SpecDiff-2 demonstrates significant improvements across multiple benchmarks and metrics ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)):

- **Speed-up Performance**: The method achieves an average 4.22x speed-up across all tested settings, representing over 30% improvement compared to EAGLE-2 ([Li et al., 2024](https://arxiv.org/abs/2406.16858)). On specialized tasks like code generation, speed-ups reach 5.51x compared to vanilla autoregressive generation.

- **Accepted streak length**: SpecDiff-2 produces substantially longer accepted sequences. For example, with Qwen2.5-72B at temperature 0, it achieves 5.98 tokens per draft on average compared to EAGLE-2's 4.41 tokens ([Li et al., 2024](https://arxiv.org/abs/2406.16858)).

- **Scaling Properties**: The self-selection mechanism shows smooth test-time scaling, with increasing the number of parallel drafts from 1 to 8 yielding up to 20% additional throughput, particularly effective at higher drafter temperatures where draft diversity is greater.


<div class="project-figure">
    {% include figure.html path="assets/img/projects/specdiff2-fig4.jpeg" title="Self-selection scaling with K drafts" class="img-fluid rounded z-depth-1 project-figure__img" %}

<div class="caption">
    <strong>Figure 4</strong>: Test-time self-selection scales smoothly with the number of parallel drafts,
    yielding up to 20% additional throughput at higher drafter temperatures.
</div>
</div>

## Acceleration-Compute Paradigm

A significant contribution is introducing "acceleration-compute" as a new scaling dimension for LLMs. The research demonstrates that faster inference directly translates to higher accuracy on complex reasoning tasks within fixed wall-time budgets. On Math500 with Chain-of-Thought prompting, SpecDiff-2 achieves a +63% boost in accuracy over vanilla models and +11% increase over unaligned SpecDiff within a 15-second reasoning budget ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)).
This paradigm shift suggests that optimizing for throughput isn't just about speed—it's about improving the effective intelligence of LLMs within practical time constraints by enabling more reasoning steps.


<div class="project-figure">
    {% include figure.html path="assets/img/projects/specdiff2-fig5.jpeg" title="Acceleration-compute improves reasoning" class="img-fluid rounded z-depth-1 project-figure__img" %}

<div class="caption">
    <strong>Figure 5</strong>: Faster inference enables better reasoning within fixed time budgets. SpecDiff-2
    provides the highest accuracy gains among acceleration methods.
</div>
</div>

## Ablation Studies and Analysis

**Individual Component Analysis**: Ablations reveal that streak-distillation alone provides substantial gains, with approximately +30% speed-up improvement over base diffusion drafters. The combination of both alignment mechanisms results in 40-50% performance improvement over unaligned SpecDiff ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)).

**Position-wise Acceptance Analysis**: The research includes detailed analysis showing how acceptance rates degrade with distance from the prefix. Streak-distillation significantly improves this degradation pattern compared to standard autoregressive distillation, maintaining higher acceptance rates throughout the draft window ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)). Related distillation-based alignment approaches include DistillSpec ([Zhou et al., 2023](https://arxiv.org/abs/2310.08461)).

<div class="project-figure">
    {% include figure.html path="assets/img/projects/specdiff2-fig6.jpeg" title="Streak-distillation training progression" class="img-fluid rounded z-depth-1 project-figure__img" %}

<div class="caption">
    <strong>Figure 6</strong>: Streak-distillation training yields monotonic speed-up improvements with
    increasing training steps.
</div>
</div>


## Significance and Impact

SpecDiff-2 establishes a new state-of-the-art for lossless LLM inference acceleration, with several important implications:

- **Architectural Innovation**: The work expands the design space for speculative decoding by demonstrating the superiority of discrete diffusion models over traditional autoregressive drafters, opening new research directions for non-autoregressive generation methods ([Christopher et al., 2025](https://arxiv.org/abs/2408.05636); [Sandler et al., 2025](https://arxiv.org/abs/2511.00606)).
- **Practical Deployment Benefits**: The significant speed improvements without accuracy loss enable more responsive user applications, reduced computational costs, and feasibility of real-time LLM systems. The ability to generate more tokens within fixed time budgets allows for deeper reasoning capabilities.
- **Research Paradigm**: The introduction of "acceleration-compute" as a scaling factor provides a new lens for understanding LLM performance optimization, suggesting that inference efficiency improvements can directly enhance problem-solving capabilities rather than just reducing latency ([Sandler et al., 2025](https://arxiv.org/abs/2511.00606)).

The comprehensive approach of jointly addressing both drafting latency and alignment challenges through specialized techniques tailored for diffusion models represents a significant advancement in making powerful LLMs more practical and accessible for real-world applications.

## Relevant citations

- Leviathan, Y., Kalman, M., and Matias, Y. (2023). Fast inference from transformers via speculative decoding. ICML 2023. https://arxiv.org/abs/2211.17192
- Christopher, J. K., Bartoldson, B. R., Ben-Nun, T., Cardei, M., Kailkhura, B., and Fioretto, F. (2025). Speculative diffusion decoding: Accelerating language generation through diffusion. NAACL 2025. https://arxiv.org/abs/2408.05636
- Sandler, J., Christopher, J. K., Hartvigsen, T., and Fioretto, F. (2025). SpecDiff-2: Scaling diffusion drafter alignment for faster speculative decoding. arXiv:2511.00606. https://arxiv.org/abs/2511.00606
- Li, Y., Wei, F., Zhang, C., and Zhang, H. (2024). EAGLE-2: Faster inference of language models with dynamic draft trees. arXiv:2406.16858. https://arxiv.org/abs/2406.16858
- Zhou, Y., Lyu, K., Rawat, A. S., Menon, A. K., Rostamizadeh, A., Kumar, S., Kagy, J.-F., and Agarwal, R. (2023). DistillSpec: Improving speculative decoding via knowledge distillation. arXiv:2310.08461. https://arxiv.org/abs/2310.08461
