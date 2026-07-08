---
layout: page
title: Constrained Discrete Diffusion for Language, Chemistry, and Code
description: Enforcing hard constraints on tokens, molecules, and programs — training-free, inside the denoising loop.
img: assets/img/projects/discrete-diffusion/highlight-discrete-diffusion.png
importance: 3
highlight: true
category: work
giscus_comments: true
---

<div class="discrete-diffusion-page" markdown="1">

## Problem Statement and Motivation

Discrete sequences — tokens in natural language, SMILES strings in chemistry, source code in programming — hold immense potential for scientific and engineering discovery. Yet deploying large language models and discrete generative models in these high-stakes domains exposes a fundamental gap: **the outputs of these models routinely violate critical constraints**, whether toxicity guidelines, chemical validity rules, functional correctness requirements, or security properties.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/discrete-diffusion/dllm_architecture.png" title="Discrete diffusion language models: forward masking and backward denoising" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 1</strong>: Discrete diffusion language models define a forward <em>masking</em> process that progressively corrupts a token sequence into noise, and a learned backward <em>denoising</em> process that reconstructs it. Unlike autoregressive models, the entire sequence is available at each denoising step — creating a natural intervention point for constraint enforcement. (Architecture following <a href="https://arxiv.org/abs/2406.07524" target="_blank">Sahoo et al., 2024</a>.)
</div>
</div>

Autoregressive models generate text left-to-right and are structurally constrained by this causal factorization: it is impossible to verify non-local properties (e.g., "does this molecule contain an aldehyde fragment?" or "does this code compile?") until generation is complete, when corrections are expensive or impossible. Discrete diffusion models break this bottleneck. Because they expose the **full, partially denoised sequence** at every reverse step, they offer something autoregressive models cannot: a global program or sequence state that can be inspected, scored, and corrected *before* committing to any final token.

Our work turns this structural advantage into a rigorous constraint-enforcement machinery — **without retraining the underlying model**.

### Key Takeaways

<div class="takeaways-grid">
    <div class="takeaway-card">Discrete diffusion exposes the full token sequence at every denoising step — a natural intervention point for enforcing non-local constraints that autoregressive models cannot natively handle.</div>
    <div class="takeaway-card">Constraints are enforced via a KL-minimizing proximal projection on the probability simplex — a training-free operation that steers token logits toward feasibility at each reverse step.</div>
    <div class="takeaway-card">The framework handles heterogeneous constraint types: toxicity filters, chemical valency rules, novelty requirements, syntactic correctness, and program-level security properties.</div>
    <div class="takeaway-card">Applications span natural language safety, drug-discovery molecular generation, symbolic reasoning, and secure code synthesis — consistently outperforming both autoregressive and unconstrained discrete diffusion baselines.</div>
</div>


## The Framework: Projecting Token Distributions onto the Feasible Set

Standard masked diffusion models (MDMs) operate over sequences of $$L$$ tokens drawn from a vocabulary $$\mathcal{V}$$. Each position $$\ell$$ in the sequence $$x_t \in \mathbb{R}^{L \times \lvert\mathcal{V}\rvert}$$ holds a probability distribution over vocabulary items. The forward process corrupts tokens by progressively replacing them with a special `[MASK]` token. The learned reverse process — a transformer network — predicts clean-state logits and iteratively unmasks the sequence.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/discrete-diffusion/cdd_framework.png" title="Constrained-aware discrete diffusion: KL proximal projection on token logits" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 2</strong>: At each reverse step, the model's predicted token distribution <em>x<sub>t</sub></em> is projected onto the constraint-feasible region by solving a KL-minimization problem over the probability simplex. The constraint is enforced at the <em>argmax</em> level — on the most likely token at each position — ensuring that the final decoded sequence satisfies the user-specified rules. Toxic or invalid tokens are gated out before they can be committed.
</div>
</div>

**Constrained Discrete Diffusion (CDD)** ([Cardei et al., NeurIPS 2025](https://arxiv.org/abs/2503.09790)) adapts the constrained diffusion principle to this discrete setting. At each reverse step $$t$$, instead of directly sampling from the unconstrained model prediction, CDD solves a local proximal problem over the probability simplex:

$$\text{prox}_{\mathcal{C}}(x_t) = \min_{y} \; \mathrm{KL}(y \,\|\, x_t) \quad \text{s.t.} \quad \arg\max(y) \to \mathcal{C}$$

Here $$y \in \Delta^{\lvert\mathcal{V}\rvert}$$ is the projected distribution and $$\mathcal{C}$$ encodes the constraint on the most-likely decoded token. This ensures two things simultaneously: **(1)** the projection stays close to the model's intended distribution (preserving fluency), and **(2)** the sequence that would result from greedily decoding the projected logits satisfies the constraint — **at the sequence level, not just position by position**.

Because the KL proximal step requires no gradient through model weights and no retraining, CDD is a **training-free, plug-and-play wrapper** for any masked diffusion backbone.

### Handling Non-Differentiable Constraints: Neuro-Symbolic Diffusion

Many real-world constraints involve non-differentiable checkers — SMARTS substructure filters in chemistry, compiler parse trees in code, or formal logic evaluators in reasoning. Our **Neuro-Symbolic Diffusion (NSD)** framework ([Christopher et al., NeuS 2025](https://arxiv.org/abs/2506.01121), **DARPA Disruptive Idea Award**) extends this principle by interleaving diffusion steps with symbolic optimization, enabling certifiably consistent generation under arbitrary functional and logic constraints — for both continuous and discrete domains.

**Search-Augmented Masked Diffusion (SearchDiff)** ([Ta et al., 2026](https://arxiv.org/abs/2602.02727)) takes a complementary approach: rather than projecting logits, it introduces a **tree search at each denoising step** that optimizes over the model's predicted proposal set under user-specified property satisfaction, yielding a modified reverse transition that steers sampling toward probable and feasible solutions.


## Application 1: Natural Language Safety — Toxicity Mitigation and Lexical Constraints

The most direct application is enforcing content-safety constraints on text generation. Large language models trained on internet-scale data can generate harmful, toxic, or factually incorrect content. CDD addresses this by treating toxicity and lexical rules as hard constraints on the token sequence.

At each reverse step, the KL-proximal projection gates out tokens whose presence would increase the probability of toxic continuations (evaluated by a classifier) or violate counting/lexical rules (e.g., "do not include the word 'kill' in the output"). Key results from ([Cardei et al., 2025](https://arxiv.org/abs/2503.09790)):

- **Zero constraint violations** on toxicity mitigation across all threshold levels (ω = 0.25, 0.50, 0.75), versus **33.2%, 21.6%, and 13.1%** violation rates for GPT-2 — and **17–32%** for MDLM and UDLM discrete diffusion baselines.
- **Zero violations** on character-level and sequence-level lexical constraints, versus **54.5%** and **97.5%** for MDLM without constraint enforcement.
- **Minimal computational overhead** compared to post-hoc alternatives: PPLM incurs an **85× slowdown**, FUDGE a **134–143× slowdown**; CDD adds negligible cost over standard discrete diffusion sampling.

Crucially, constraint satisfaction is achieved while **preserving semantic coherence and fluency** — perplexity remains competitive with unconstrained generation, as CDD only departs from the model's intent by the minimum required to satisfy constraints.


## Application 2: Molecular Generation for Drug Discovery

<div class="project-figure project-figure--float-right">
    {% include figure.html path="assets/img/projects/discrete-diffusion/molgen_task.png" title="Molecular generation with novelty and toxicity constraints on SMILES" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 3</strong>: Generating molecules in SMILES format requires simultaneously enforcing novelty (the molecule must not appear in the training set) and toxicity-avoidance (BRENK substructure filters for aldehyde flags, three-membered heterocycles, and other reactive fragments). CDD enforces these constraints directly on the denoising trajectory rather than via post-hoc filtering.
</div>
</div>

Chemical molecule generation is a natural discrete-diffusion application: SMILES strings are sequential, token-based representations of molecular structure. Generating valid drug candidates requires satisfying multiple simultaneous constraints — novelty (the molecule must not be in the training set), toxicity avoidance (substructure filters from medicinal chemistry), and chemical validity (SMILES grammar rules).

Our **CDD for Molecular Generation** ([Cardei et al., AI4D3 NeurIPS 2025](https://ai4d3.github.io/2025/papers/18_Constrained_Molecular_Gener.pdf), **Best Paper Award**) applies the KL proximal projection to enforce five BRENK substructure filters — ruling out aldehydes, three-membered heterocycles, and other toxicity-linked fragments — jointly with novelty constraints, all within a single training-free inference procedure.

<div style="clear:both"></div>

<div class="project-figure">
    {% include figure.html path="assets/img/projects/discrete-diffusion/molecular_design.png" title="Molecular design results: valid, novel, and non-toxic molecule generation" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 4</strong>: Molecule generation results comparing autoregressive (AR), unconstrained masked diffusion (MDLM, UDLM), and CDD with BRENK filters. CDD-based methods produce drastically more novel and non-toxic candidates while reducing constraint violations to zero — without any model retraining.
</div>
</div>

Quantitative results from the molecule generation benchmark:

<div class="project-table-card">
  <div class="project-table-card__header">
    <div>
      <span class="project-table-eyebrow">Molecule generation benchmark</span>
      <h4>Novel and non-toxic molecule discovery</h4>
    </div>
    <span class="project-table-legend">Higher is better for discovery; lower is better for violations.</span>
  </div>
  <div class="project-table-scroll">
    <table class="project-table project-table--metrics">
      <thead>
        <tr>
          <th scope="col">Model</th>
          <th scope="col">Novel <span class="metric-direction metric-direction--up">↑</span></th>
          <th scope="col">Novel &amp; non-toxic <span class="metric-direction metric-direction--up">↑</span></th>
          <th scope="col">Novelty violations <span class="metric-direction metric-direction--down">↓</span></th>
          <th scope="col">BRENK violations <span class="metric-direction metric-direction--down">↓</span></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row" data-label="Model">AR</th>
          <td data-label="Novel">10.3 ± 2.3</td>
          <td data-label="Novel & non-toxic">5.3 ± 1.4</td>
          <td data-label="Novelty violations">99.0%</td>
          <td data-label="BRENK violations">40.2%</td>
        </tr>
        <tr>
          <th scope="row" data-label="Model">MDLM</th>
          <td data-label="Novel">260.7 ± 16.4</td>
          <td data-label="Novel & non-toxic">108.0 ± 9.7</td>
          <td data-label="Novelty violations">53.9%</td>
          <td data-label="BRENK violations">35.3%</td>
        </tr>
        <tr>
          <th scope="row" data-label="Model">UDLM</th>
          <td data-label="Novel">279.7 ± 22.7</td>
          <td data-label="Novel & non-toxic">132.3 ± 3.7</td>
          <td data-label="Novelty violations">70.8%</td>
          <td data-label="BRENK violations">38.1%</td>
        </tr>
        <tr class="project-table__highlight">
          <th scope="row" data-label="Model">CDD + BRENK <span class="table-note">ours</span></th>
          <td data-label="Novel"><strong>451.7 ± 19.5</strong></td>
          <td data-label="Novel & non-toxic"><strong>392.0 ± 16.7</strong></td>
          <td data-label="Novelty violations">51.2%</td>
          <td data-label="BRENK violations"><strong>0.0%</strong></td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

By combining novelty enforcement with BRENK filters, our method produces **392 novel non-toxic molecules** per evaluation run — versus **108 for MDLM** and **5 for autoregressive generation** — while reducing BRENK violations to exactly zero.


## Search-Augmented Discrete Diffusion

<div class="project-figure">
    {% include figure.html path="assets/img/projects/discrete-diffusion/search_augmented.png" title="SearchDiff: tree search augments the denoising process for constrained generation" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 5</strong>: SearchDiff integrates informed tree search into the reverse denoising process. At each step, the model's predicted token distribution defines a proposal set; search optimizes over this set under user-defined property constraints, yielding a modified reverse transition that steers sampling toward probable <em>and</em> feasible completions.
</div>
</div>

Projection-based methods like CDD work well when constraint satisfaction can be efficiently evaluated and projected locally. But some constraints require evaluating global sequence properties — protein secondary structure, symbolic logic satisfiability, or pass/fail test suite execution — where local projections are insufficient.

**SearchDiff** ([Ta et al., 2026](https://arxiv.org/abs/2602.02727)) addresses this by replacing the projection step with a **search procedure**: at each denoising step, the model's prediction defines a proposal set of high-probability candidate tokens, and a search algorithm (informed by the constraint evaluator) selects the candidate that best satisfies the target property while remaining within the model's distribution. This yields a modified reverse transition that **steers sampling toward probable and feasible solutions**.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/discrete-diffusion/blackbox_evaluators.png" title="Discrete diffusion enables blackbox constraint evaluators at every step" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 6</strong>: A key advantage of discrete diffusion over autoregressive generation: because the <em>full</em> candidate sequence is available at every reverse step, any blackbox evaluator — a protein structure predictor, a logic solver, a test harness — can score the entire sequence and return a constraint signal, which SearchDiff uses to guide the next denoising step.
</div>
</div>

A key structural advantage exploited by SearchDiff is that discrete diffusion makes the **full candidate sequence accessible at every step**, even early in generation. This allows arbitrary blackbox evaluators — protein structure predictors, formal logic solvers, test suite harnesses, financial risk models — to score and critique a draft output at intermediate steps, enabling much earlier constraint correction than is possible with autoregressive decoding.

Across biological design (protein sequence) and symbolic reasoning benchmarks, SearchDiff **substantially improves constraint satisfaction and property adherence**, consistently outperforming both unconstrained discrete diffusion and autoregressive baselines.


## Application 3: Constrained Code Generation

<div class="project-figure">
    {% include figure.html path="assets/img/projects/discrete-diffusion/code_gen_results.png" title="Constrained code generation results: functional correctness, security, and syntax" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 7</strong>: Results for constrained code generation across functional correctness, security, and syntax benchmarks. CDC (Constrained Diffusion for Code) consistently improves constraint satisfaction over both discrete diffusion and autoregressive baselines, using <em>less</em> corrective computation and more localized edits to the token sequence.
</div>
</div>

Code generation imposes among the most demanding constraint combinations: programs must be syntactically valid, functionally correct with respect to a specification, and free of security vulnerabilities — simultaneously. Autoregressive code models (e.g., Codex, DeepSeek-Coder) frequently produce outputs that pass one criterion while failing another, because left-to-right generation cannot "look ahead" and globally reason about program structure.

**Constrained Diffusion for Code (CDC)** ([Shao et al., 2026](https://arxiv.org/abs/2605.16829)) applies the discrete diffusion constraint-enforcement principle to this domain. At each denoising step, CDC combines **mathematical optimization with static program analysis** to identify constraint-relevant regions of the intermediate program state — the specific tokens that, if modified, would most improve constraint satisfaction — and locally adjusts the denoising trajectory toward feasible programs.

CDC provides:

- **Functional correctness**: Enforcing that programs pass unit tests by guiding generation toward syntactically and semantically consistent completions.
- **Security constraints**: Ruling out code patterns (e.g., unsafe buffer operations, SQL injection vulnerabilities) via rule-based program analysis applied at each step.
- **Syntax validity**: Grounding token predictions in parse-tree structure, eliminating syntactically invalid intermediate states before they propagate.

Across all three constraint categories, CDC **consistently outperforms both discrete diffusion and autoregressive baselines** — with **less corrective computation** (more localized edits) than post-hoc repair methods.


## Improving the Backbone: Self-Conditioned Masked Diffusion

Constraint enforcement quality depends directly on the quality of the underlying discrete diffusion backbone — how accurately the model predicts clean-state logits from partially masked inputs. A key limitation of standard MDMs is that masked positions are inferred from scratch at each step, discarding useful intermediate predictions.

**Simple Self-Conditioning Adaptation for Masked Diffusion Models (SCMDM)** ([Cardei et al., 2026](https://arxiv.org/abs/2604.26985)) addresses this with a simple but effective **post-training adaptation**: each denoising step is conditioned on the model's own previous clean-state predictions. This allows still-masked positions to accumulate refinement signal across steps, without introducing a recurrent latent pathway or requiring retraining from scratch.

SCMDM achieves **nearly a 50% reduction in generative perplexity** on language modeling benchmarks (42.89 → 23.72 on OWT), alongside strong improvements in molecular generation and genomic distribution modeling — with no additional denoiser evaluations at inference time. Better backbone predictions translate directly into higher-quality starting points for CDD's constraint projection steps.


## Key Results

<div class="project-table-card project-table-card--results">
  <div class="project-table-card__header">
    <div>
      <span class="project-table-eyebrow">Summary across domains</span>
      <h4>Where constrained discrete diffusion helps</h4>
    </div>
    <span class="project-table-legend">Use this as the fast-reading map for the page.</span>
  </div>
  <div class="project-table-scroll">
    <table class="project-table project-table--results">
      <thead>
        <tr>
          <th scope="col">Domain</th>
          <th scope="col">Method</th>
          <th scope="col">Result to remember</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Toxicity mitigation</span></th>
          <td data-label="Method">CDD <a href="https://arxiv.org/abs/2503.09790">NeurIPS 2025</a></td>
          <td data-label="Result">0% violations at all thresholds vs. 13–33% for GPT-2 and 17–32% for MDLM/UDLM.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Lexical constraints</span></th>
          <td data-label="Method">CDD <a href="https://arxiv.org/abs/2503.09790">NeurIPS 2025</a></td>
          <td data-label="Result">0% violations on counting and lexical constraints vs. 54.5–97.5% for MDLM.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Molecular generation</span></th>
          <td data-label="Method">CDD + BRENK <a href="https://ai4d3.github.io/2025/papers/18_Constrained_Molecular_Gener.pdf">AI4D3 2025</a></td>
          <td data-label="Result">392 novel non-toxic molecules vs. 108 for MDLM, with 0% BRENK violations.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Symbolic reasoning</span></th>
          <td data-label="Method">SearchDiff <a href="https://arxiv.org/abs/2602.02727">2026</a></td>
          <td data-label="Result">Substantially improves satisfaction and outperforms autoregressive and discrete diffusion baselines.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Code generation</span></th>
          <td data-label="Method">CDC <a href="https://arxiv.org/abs/2605.16829">2026</a></td>
          <td data-label="Result">Improves correctness, security, and syntax with less computation than post-hoc repair.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">MDM backbone</span></th>
          <td data-label="Method">SCMDM <a href="https://arxiv.org/abs/2604.26985">2026</a></td>
          <td data-label="Result">About 50% perplexity reduction, from 42.89 to 23.72, with no extra inference cost.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Neuro-symbolic</span></th>
          <td data-label="Method">NSD <a href="https://arxiv.org/abs/2506.01121">NeuS 2025</a></td>
          <td data-label="Result">Certifiable constraint satisfaction across continuous and discrete domains; DARPA Disruptive Idea.</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

This body of work establishes a general principle: **discrete diffusion is the right architecture for constrained sequence generation**. The global token state at each step is not just a modeling choice — it is the mechanism that makes training-free, non-local constraint enforcement tractable. Autoregressive models, by design, cannot offer this.


## Relevant Citations

- Cardei, M., Christopher, J. K., Hartvigsen, T., Kailkhura, B., Fioretto, F. (2025). Constrained Discrete Diffusion. *NeurIPS 2025*. [arXiv:2503.09790](https://arxiv.org/abs/2503.09790)
- Cardei, M., Christopher, J. K., Hartvigsen, T., Kailkhura, B., Fioretto, F. (2025). Constrained Molecular Generation with Discrete Diffusion for Drug Discovery. *AI4D3 Workshop, NeurIPS 2025*. **Best Paper Award**. [PDF](https://ai4d3.github.io/2025/papers/18_Constrained_Molecular_Gener.pdf)
- Ta, H. B., Cardei, M., Velasquez, A., Fioretto, F. (2026). Search-Augmented Masked Diffusion Models for Constrained Generation. [arXiv:2602.02727](https://arxiv.org/abs/2602.02727)
- Shao, L., Cardei, M., Xie, Z., Fioretto, F., Wang, W. (2026). Constrained Code Generation with Discrete Diffusion. [arXiv:2605.16829](https://arxiv.org/abs/2605.16829)
- Cardei, M., Ta, H. B., Fioretto, F. (2026). Simple Self-Conditioning Adaptation for Masked Diffusion Models. [arXiv:2604.26985](https://arxiv.org/abs/2604.26985)
- Christopher, J. K., Cardei, M., Liang, J., Fioretto, F. (2025). Neuro-Symbolic Generative Diffusion Models for Physically Grounded, Robust, and Safe Generation. *NeuS 2025*. **DARPA Disruptive Idea Award**. [arXiv:2506.01121](https://arxiv.org/abs/2506.01121)
- Sahoo, S., et al. (2024). Simple and Effective Masked Diffusion Language Models. *NeurIPS 2024*. [arXiv:2406.07524](https://arxiv.org/abs/2406.07524)

</div>
