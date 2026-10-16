---
layout: page
permalink: /research/
title: research
nav: true
nav_order: 0
---

<div class="research-overview">
  <p class="research-overview__lead">We develop learning and generative systems that are <strong>controllable, constraint-aware, steerable, and grounded in the requirements of science and engineering.</strong></p>

  <p>I work across three connected themes:</p>

<section class="research-theme" id="foundational-ml">
<div class="research-theme__content" markdown="1">

## Foundational ML and Generative AI under Constraints

We develop the algorithmic foundations for predictive and generative models whose outputs are not only likely, but feasible, safe, and physically meaningful. This line of work builds on our earlier foundations for differentiable optimization, decision-focused learning, and learning-to-optimize systems, including Lagrangian methods for constrained deep learning ([AAAI-20](https://ojs.aaai.org/index.php/AAAI/article/view/5403), [ECML-20](https://doi.org/10.1007/978-3-030-67670-4_8), [AAAI-22](https://ojs.aaai.org/index.php/AAAI/article/view/20685)), end-to-end prediction and optimization ([IJCAI-22](https://doi.org/10.24963/ijcai.2022/815), [JAIR-24](https://arxiv.org/abs/2307.13565), [ECAI-24](https://arxiv.org/abs/2311.13087)), and differentiable parametric programming and bilevel learning-to-optimize ([CDC-24](https://arxiv.org/abs/2404.00882), [L4DC-26](https://arxiv.org/abs/2507.09050)). We now extend these ideas to constrained continuous diffusion and flow matching ([NeurIPS-25a](https://arxiv.org/abs/2502.05625), [ArXiv:26a](https://arxiv.org/abs/2605.12754)), constrained discrete diffusion and search-augmented masked diffusion ([NeurIPS-25b](https://arxiv.org/abs/2503.09790), [ArXiv:26b](https://arxiv.org/abs/2602.02727)), and constrained code generation and diffusion-based speculative decoding ([ArXiv:26c](https://arxiv.org/abs/2605.16829), [MLSys-26](https://arxiv.org/abs/2511.00606)).
</div>
</section>

<section class="research-theme" id="ai-for-science">
<div class="research-theme__content" markdown="1">

## AI for Science and Engineering

We use these foundations to support scientific discovery and engineering design, including inverse design of protein binders ([ICLR-26](https://www.biorxiv.org/content/10.1101/2025.08.06.668648v1)), materials with defects and micro-material design ([AI4Mat-25](https://openreview.net/pdf?id=L3LYjJJoL5), [NeurIPS-24](https://arxiv.org/abs/2402.03559), [NeurIPS-25a](https://arxiv.org/abs/2502.05625)), safe multi-robot motion planning ([ICML-25](https://arxiv.org/abs/2502.03607), [AAAI-26](https://arxiv.org/abs/2508.20095)), and molecular generation workflows ([NeurIPS-25b](https://arxiv.org/abs/2503.09790), [ArXiv:26](https://arxiv.org/abs/2508.12331)). Our physics-informed and constraint-aware methods have also been used to accelerate energy optimization, operational planning, and scheduling tasks (e.g., [AAAI-20](https://arxiv.org/abs/1909.10461), [AAAI-22](https://ojs.aaai.org/index.php/AAAI/article/view/20685), [NeurIPS-21](https://openreview.net/forum?id=2zO2lb7ykMD), [EPSR-23](https://www.sciencedirect.com/science/article/pii/S0378779623004406), [NeurIPS-23](https://arxiv.org/abs/2308.01436), [ICLR-25](https://arxiv.org/abs/2410.01786)).
</div>
</section>

<section class="research-theme" id="responsible-ai">
<div class="research-theme__content" markdown="1">

## Responsible AI

We analyze the assurance of AI systems in support of decision-making and learning tasks, focusing especially on privacy, safety, and fairness. Our work has uncovered new theoretical and algorithmic understanding at the intersection of privacy and fairness, including fairness-aware post-processing of private data releases, differentially private and fair learning, and fairness issues in socio-demographic data processes (e.g., [IJCAI-21](https://doi.org/10.24963/ijcai.2021/312), [IJCAI-22](https://doi.org/10.24963/ijcai.2022/559), [AAAI-21](https://ojs.aaai.org/index.php/AAAI/article/view/17193), [AAAI-25](https://arxiv.org/abs/2408.08471)). We also study how safety and fairness can emerge when machine learning and agentic LLM pipelines are constrained by systems or robustness considerations, including pruning at initialization ([NeurIPS-22](https://openreview.net/forum?id=VXXTBdy4FA)), hardware failures and deployment choices ([ICML-24](https://openreview.net/forum?id=1Lj2XfljS8)), and adversarial robustness requirements ([IJCAI-24](https://doi.org/10.48550/arXiv.2211.11835)).
</div>
</section>

  <p class="research-overview__more">For more details, please see our <strong><a href="{{ '/publications/' | relative_url }}">publications</a></strong> and my <strong><a href="{{ '/bio/' | relative_url }}">short bio</a></strong>.</p>

  <section class="research-talks" aria-labelledby="recorded-talks-title">
    <div class="research-talks__heading">
      <div>
        <h2 id="recorded-talks-title">Recorded Talks</h2>
      </div>
      <p>Selected talks and tutorials spanning our research foundations and applications.</p>
    </div>
    {% include research_talks.html %}
  </section>

  {% include recruitment.html %}
</div>
