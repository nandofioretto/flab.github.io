---
layout: research_area_project
title: Constrained Discrete Diffusion for Language, Chemistry, and Code
description: Enforcing hard constraints on tokens, molecules, and programs — training-free, inside the denoising loop.
img: assets/img/projects/discrete-diffusion/highlight-discrete-diffusion-v2.png
importance: 3
highlight: true
category: work
giscus_comments: true
---

<section class="research-section research-section--lead" id="start-here" markdown="1">

<div class="section-label">01 · Start here</div>

## From tokens to masks—and back again

<p class="section-standfirst">Discrete diffusion generates text, molecular strings, and code by repeatedly refining an entire sequence. That global, editable state is what makes constraint enforcement possible before the output is final.</p>

Let $$x_0=(x_0^1,\ldots,x_0^L)$$ be a sequence of $$L$$ tokens drawn from a vocabulary $$\mathcal{V}$$. A masked diffusion model defines a **forward process** that progressively replaces clean tokens with a special `[MASK]` symbol. At noise level $$t$$, each position is retained with probability $$\alpha_t$$ and masked otherwise:

$$q(x_t^\ell\mid x_0^\ell)=\operatorname{Cat}\!\left(\alpha_t x_0^\ell+(1-\alpha_t)m\right),$$

where $$m$$ denotes the one-hot representation of `[MASK]`. As $$t$$ increases, more of the sequence disappears. The learned reverse process starts from a fully masked state and predicts clean-token distributions until a complete sequence is recovered.

<div class="diffusion-steps" aria-label="Forward masking and reverse denoising for discrete diffusion">
  <div><span>Forward process</span><strong>Tokens → masks</strong><small>A fixed corruption schedule progressively removes information.</small></div>
  <div class="diffusion-steps__arrow">⇄</div>
  <div><span>Learned reverse process</span><strong>Masks → sequence</strong><small>A transformer repeatedly predicts the clean token at every position.</small></div>
</div>

### What the denoiser learns

Given a partially masked sequence $$x_t$$, the denoiser predicts a distribution over the vocabulary for **every position**:

$$x_\theta(x_t,t)\in\Delta^{L\times|\mathcal{V}|}.$$

Unlike a left-to-right model, it does not commit permanently to one next token. It exposes a global proposal for the entire clean sequence at each reverse step. Tokens that remain uncertain can be reconsidered as information elsewhere in the sequence becomes available.

<div class="plain-language">
  <div class="plain-language__mark">What the model sees</div>
  <p>A draft of the whole output—not merely its prefix. That draft can be inspected for a distant word, a forbidden molecular substructure, a failed test, or another sequence-level requirement while there is still time to revise it.</p>
</div>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/dllm_architecture.png" title="Discrete diffusion language models: forward masking and backward denoising" class="research-figure__img" %}
  <figcaption><span>Figure 1</span> The forward process masks a token sequence; the learned reverse process reconstructs all positions through iterative refinement.</figcaption>
</figure>

<p class="paper-link">Mathematical foundation · <a href="https://arxiv.org/abs/2406.07524">Sahoo et al., NeurIPS 2024 →</a></p>

### Why a coherent sequence is not necessarily a valid one

Discrete sequences—tokens in natural language, SMILES strings in chemistry, and source code in programming—carry rules that likelihood alone does not guarantee. A fluent sentence may violate a lexical instruction. A plausible molecular string may contain an unwanted structural alert. A program can look natural while failing its tests or introducing a vulnerability.

Autoregressive models generate left to right, so many non-local properties cannot be evaluated until the sequence is complete. Post-hoc filtering can catch failures, but only after the generation cost has been paid. Discrete diffusion offers a different intervention point: inspect and correct the global draft throughout denoising.

<div class="research-compare" aria-label="Three approaches to constraints in sequence generation">
  <div>
    <span class="research-compare__index">A</span>
    <h3>Prompt and hope</h3>
    <p>Ask the model for a rule, but receive no per-sample assurance that it will be followed.</p>
  </div>
  <div>
    <span class="research-compare__index">B</span>
    <h3>Generate, then filter</h3>
    <p>Reject invalid outputs after completion, which becomes inefficient when feasible sequences are rare.</p>
  </div>
  <div class="research-compare__active">
    <span class="research-compare__index">C</span>
    <h3>Constrain while denoising</h3>
    <p>Evaluate the full draft and minimally redirect token probabilities before the output is committed.</p>
  </div>
</div>

<div class="research-callout research-callout--thesis">
  <span class="research-callout__eyebrow">The thesis</span>
  <strong>The global token state in discrete diffusion is more than a modeling choice—it is an interface for enforcing non-local rules during generation.</strong>
  <p>Our work turns that interface into training-free projection, symbolic feedback, and search mechanisms for language, molecules, reasoning, and code.</p>
</div>

</section>

<section class="research-section" id="core-idea" markdown="1">

<div class="section-label">02 · Core idea</div>

## Project token distributions onto the feasible set

<p class="section-standfirst">In continuous diffusion, a projection moves a point in Euclidean space. In discrete diffusion, the object being corrected is a probability distribution over possible tokens.</p>

The continuous constraint-aware foundation views reverse diffusion as a sequence of optimization steps and augments each one with a feasibility correction. **Constrained Discrete Diffusion (CDD)** carries that idea to the probability simplex. At each reverse step, instead of sampling directly from the unconstrained prediction $$x_t$$, CDD solves a local proximal problem:

$$\operatorname{prox}_{\mathcal{C}}(x_t)
= \arg\min_y\;D_{\mathrm{KL}}(y\,\|\,x_t)
\quad\text{subject to}\quad \arg\max(y)\in\mathcal{C}.$$

Here $$y\in\Delta^{L\times|\mathcal{V}|}$$ is the corrected token distribution and $$\mathcal{C}$$ is the feasible set. The KL objective changes the model's proposal only as much as needed, while the constraint acts on the sequence that would be obtained by decoding the corrected probabilities.

<p class="paper-link">Constraint-aware foundation · <a href="https://arxiv.org/abs/2402.03559">Christopher, Baek &amp; Fioretto, NeurIPS 2024 →</a></p>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/cdd_framework.png" title="Constraint-aware discrete diffusion: KL proximal projection on token logits" class="research-figure__img" %}
  <figcaption><span>Figure 2</span> At every reverse step, CDD finds a nearby token distribution whose most likely decoded sequence satisfies the specified rule.</figcaption>
</figure>

<div class="plain-language">
  <div class="plain-language__mark">What projection changes</div>
  <p>The denoiser still decides which sequences are plausible. The projection only redistributes enough probability mass to prevent the current draft from violating the encoded requirement.</p>
</div>

Because the proximal step does not update model weights, CDD is a **training-free wrapper** for a pretrained masked-diffusion backbone. The same sampler can support different constraints by changing the evaluator and projection rather than retraining the generator.

<p class="paper-link">Discrete constraint framework · <a href="https://arxiv.org/abs/2503.09790">Cardei et al., NeurIPS 2025 →</a></p>

### Beyond differentiable constraints

Many important checks are black boxes: SMARTS or BRENK substructure filters in chemistry, compilers and static analyzers in code, protein structure predictors, or formal logic solvers. **Neuro-Symbolic Diffusion (NSD)** interleaves denoising with symbolic optimization, while **Search-Augmented Masked Diffusion (SearchDiff)** explores the model's high-probability token proposals with tree search when a local projection is insufficient.

<div class="current-work-grid">
  <div>
    <span>Symbolic feedback</span>
    <h3>Neuro-Symbolic Diffusion</h3>
    <p>External logic and functional evaluators critique intermediate samples, allowing the sampler to enforce rules that are not naturally differentiable.</p>
    <a href="https://arxiv.org/abs/2506.01121">Christopher et al., NeuS 2025 →</a>
  </div>
  <div>
    <span>Search-based correction</span>
    <h3>SearchDiff</h3>
    <p>Tree search evaluates alternative high-probability tokens and selects transitions that improve global property satisfaction.</p>
    <a href="https://arxiv.org/abs/2602.02727">Ta et al., 2026 →</a>
  </div>
</div>

</section>

<section class="research-section" id="research-landscape" markdown="1">

<div class="section-label">03 · Research landscape</div>

## One sampling principle, several kinds of rules

<p class="section-standfirst">The evaluator changes from one domain to another, but the recurring loop remains the same: propose a full sequence, evaluate it, and correct the token distribution before denoising continues.</p>

<div class="landscape-grid">
  <a href="#language"><span>Application 01</span><strong>Language safety</strong><small>Toxicity thresholds, counting, and lexical placement</small></a>
  <a href="#molecules"><span>Application 02</span><strong>Molecular generation</strong><small>Novelty, structural alerts, and chemical validity</small></a>
  <a href="#searchdiff"><span>Method extension</span><strong>Black-box search</strong><small>Protein properties, logic, and arbitrary evaluators</small></a>
  <a href="#code"><span>Application 03</span><strong>Code generation</strong><small>Functionality, syntax, and security</small></a>
</div>

<div class="result-strip" aria-label="Selected headline results">
  <div><strong>0%</strong><span>toxicity-rule violations</span></div>
  <div><strong>392</strong><span>novel BRENK-screened molecules</span></div>
  <div><strong>65.2%</strong><span>HumanEval-X functionality with CDC</span></div>
  <div><strong>~50%</strong><span>backbone perplexity reduction</span></div>
</div>

</section>

<section class="research-section research-domain" id="language" markdown="1">

<div class="domain-heading"><span>Application 01</span><div class="domain-heading__dot domain-heading__dot--language"></div></div>

## Natural-language safety and exact lexical rules

<p class="section-standfirst">Language constraints range from learned safety scores to exact symbolic instructions. CDD treats both as requirements on the complete token sequence rather than preferences attached to individual next-token choices.</p>

For toxicity mitigation, the current draft is evaluated by a classifier and the token distribution is projected toward a specified threshold. For counting and lexical tasks, the feasible set encodes exact requirements such as how many times a character occurs or which word must appear at a particular position.

Across all evaluated toxicity thresholds ($$\omega=0.25,0.50,0.75$$), CDD produced **zero constraint violations**, compared with violation rates of 33.2%, 21.6%, and 13.1% for GPT-2 and 17–32% for the evaluated discrete-diffusion baselines. It also reached **zero violations** on counting and positional lexical constraints, versus 54.5% and 97.5% for unconstrained MDLM.

<div class="evidence-grid">
  <div><span>Toxicity thresholds</span><strong>0%</strong><small>violations across all evaluated settings</small></div>
  <div><span>Counting rules</span><strong>0%</strong><small>vs. 54.5% violations for MDLM</small></div>
  <div><span>Lexical placement</span><strong>0%</strong><small>vs. 97.5% violations for MDLM</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Exact sequence-level control can be added to a masked diffusion model without retraining it for every new instruction.</strong>
  <p>The guarantee is tied to the supplied rule or classifier, so it should be interpreted as reliable benchmark compliance rather than a complete definition of safe language. Fluency and perplexity remain competitive, although constraint enforcement introduces task-dependent tradeoffs.</p>
</div>

<p class="paper-link">Language constraints · <a href="https://arxiv.org/abs/2503.09790">Cardei et al., NeurIPS 2025 →</a></p>

</section>

<section class="research-section research-domain" id="molecules" markdown="1">

<div class="domain-heading"><span>Application 02</span><div class="domain-heading__dot domain-heading__dot--molecules"></div></div>

## Molecular generation for drug discovery

<p class="section-standfirst">A SMILES string is a discrete sequence, but its value depends on the chemical structure it represents. Useful generation therefore requires sequence validity, novelty, and domain filters to hold together.</p>

Chemical molecule generation is a natural discrete-diffusion application: SMILES strings are token-based representations of molecular structure. The benchmark combines novelty constraints with five BRENK substructure filters, ruling out selected aldehydes, three-membered heterocycles, and other reactive or undesirable fragments during generation rather than after it.

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/molgen_task.png" title="Molecular generation with novelty and toxicity constraints on SMILES" class="research-figure__img" %}
  <figcaption><span>Figure 3</span> CDD evaluates novelty and selected medicinal-chemistry structural alerts throughout the SMILES denoising trajectory.</figcaption>
</figure>

Our **CDD for Molecular Generation** applies KL-proximal corrections for novelty and BRENK constraints within a single training-free inference procedure. The generated sequence remains close to the model's proposal while being redirected away from encoded violations.

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/molecular_design.png" title="Molecular design results: valid, novel, and non-toxic molecule generation" class="research-figure__img" %}
  <figcaption><span>Figure 4</span> CDD produces more novel candidates that pass the selected BRENK screens than the evaluated autoregressive and unconstrained diffusion baselines.</figcaption>
</figure>

<div class="project-table-card">
  <div class="project-table-card__header">
    <div>
      <span class="project-table-eyebrow">Molecule generation benchmark</span>
      <h4>Novel and BRENK-screened molecule discovery</h4>
    </div>
    <span class="project-table-legend">Higher is better for discovery; lower is better for violations.</span>
  </div>
  <div class="project-table-scroll">
    <table class="project-table project-table--metrics">
      <thead>
        <tr>
          <th scope="col">Model</th>
          <th scope="col">Novel <span class="metric-direction metric-direction--up">↑</span></th>
          <th scope="col">Novel &amp; screened <span class="metric-direction metric-direction--up">↑</span></th>
          <th scope="col">Novelty violations <span class="metric-direction metric-direction--down">↓</span></th>
          <th scope="col">BRENK violations <span class="metric-direction metric-direction--down">↓</span></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row" data-label="Model">AR</th>
          <td data-label="Novel">10.3 ± 2.3</td>
          <td data-label="Novel & screened">5.3 ± 1.4</td>
          <td data-label="Novelty violations">99.0%</td>
          <td data-label="BRENK violations">40.2%</td>
        </tr>
        <tr>
          <th scope="row" data-label="Model">MDLM</th>
          <td data-label="Novel">260.7 ± 16.4</td>
          <td data-label="Novel & screened">108.0 ± 9.7</td>
          <td data-label="Novelty violations">53.9%</td>
          <td data-label="BRENK violations">35.3%</td>
        </tr>
        <tr>
          <th scope="row" data-label="Model">UDLM</th>
          <td data-label="Novel">279.7 ± 22.7</td>
          <td data-label="Novel & screened">132.3 ± 3.7</td>
          <td data-label="Novelty violations">70.8%</td>
          <td data-label="BRENK violations">38.1%</td>
        </tr>
        <tr class="project-table__highlight">
          <th scope="row" data-label="Model">CDD + BRENK <span class="table-note">ours</span></th>
          <td data-label="Novel"><strong>451.7 ± 19.5</strong></td>
          <td data-label="Novel & screened"><strong>392.0 ± 16.7</strong></td>
          <td data-label="Novelty violations">51.2%</td>
          <td data-label="BRENK violations"><strong>0.0%</strong></td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

<div class="evidence-grid">
  <div><span>Novel &amp; screened</span><strong>392</strong><small>vs. 108 for MDLM and 5 for AR</small></div>
  <div><span>BRENK violations</span><strong>0%</strong><small>for the encoded structural alerts</small></div>
  <div><span>Model retraining</span><strong>None</strong><small>constraints are imposed at inference</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Constraint-aware denoising can focus molecular exploration on novel candidates that pass specified structural screens.</strong>
  <p>BRENK filters are useful early screens, not a complete assessment of toxicity, synthesizability, stability, or biological activity. The outputs remain computational candidates that require broader cheminformatics and experimental validation.</p>
</div>

<p class="paper-link">Molecular constraints · <a href="https://ai4d3.github.io/2025/papers/18_Constrained_Molecular_Gener.pdf">Cardei et al., AI4D3 at NeurIPS 2025 — Best Paper →</a></p>

</section>

<section class="research-section research-domain" id="searchdiff" markdown="1">

<div class="domain-heading"><span>Method extension</span><div class="domain-heading__dot domain-heading__dot--search"></div></div>

## Search-augmented discrete diffusion

<p class="section-standfirst">Projection works well when feasibility can be evaluated and corrected locally. SearchDiff addresses global black-box objectives for which the useful correction is not available as a gradient.</p>

At each denoising step, the model's token distribution defines a proposal set. Search explores high-probability alternatives, evaluates them with a user-provided property function, and selects a transition that balances likelihood and constraint satisfaction.

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/search_augmented.png" title="SearchDiff: tree search augments the denoising process for constrained generation" class="research-figure__img" %}
  <figcaption><span>Figure 5</span> SearchDiff uses the denoiser to propose likely tokens and tree search to identify transitions that better satisfy a global property.</figcaption>
</figure>

The key architectural advantage is that the **full candidate sequence is accessible at every reverse step**. A protein structure predictor, logic solver, test harness, or other black-box evaluator can score the current draft and return feedback long before generation is complete.

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/blackbox_evaluators.png" title="Discrete diffusion enables blackbox constraint evaluators at every step" class="research-figure__img" %}
  <figcaption><span>Figure 6</span> Global intermediate states allow arbitrary evaluators to critique a sequence and guide subsequent denoising.</figcaption>
</figure>

Across biological design and symbolic-reasoning benchmarks, SearchDiff substantially improves property adherence over unconstrained discrete diffusion and the evaluated autoregressive baselines.

<p class="paper-link">Search-based constraints · <a href="https://arxiv.org/abs/2602.02727">Ta et al., 2026 →</a></p>

</section>

<section class="research-section research-domain" id="code" markdown="1">

<div class="domain-heading"><span>Application 03</span><div class="domain-heading__dot domain-heading__dot--code"></div></div>

## Constrained code generation

<p class="section-standfirst">Programs must satisfy syntax, functionality, and security requirements simultaneously. Diffusion's editable global state provides a place to localize a failure and revise only the responsible region.</p>

**Constrained Diffusion for Code (CDC)** combines mathematical optimization with program analysis. At each denoising step, it decodes an intermediate program, evaluates the relevant requirement, identifies the tokens most responsible for a violation, and locally adjusts or remasks that region while keeping the remainder close to the base model's proposal.

For functional correctness, CDC uses differentiable surrogate execution feedback to guide generations toward programs that pass unit tests. For security, static analysis identifies vulnerability paths and supplies targeted symbolic feedback. The same localized corrections also improve syntactic validity.

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/code_gen_results.png" title="Constrained code generation results: functional correctness, security, and syntax" class="research-figure__img" %}
  <figcaption><span>Figure 7</span> CDC improves benchmark pass rates for functionality, syntax, and security across DiffuCoder 7B and Dream-Coder 7B.</figcaption>
</figure>

With Dream-Coder 7B, CDC raises HumanEval-X C++ functionality from **34.1% to 65.2%** and MBPP-C++ functionality from **27.7% to 59.2%**. Syntax correctness rises from 67.1% to 79.2% and from 41.1% to 72.0%, respectively. On CWEval, joint functionality-and-security success increases from **12.04% to 34.26%**.

<div class="evidence-grid">
  <div><span>HumanEval-X function</span><strong>65.2%</strong><small>vs. 34.1% for the base model</small></div>
  <div><span>HumanEval-X syntax</span><strong>79.2%</strong><small>vs. 67.1% for the base model</small></div>
  <div><span>CWEval joint success</span><strong>34.26%</strong><small>vs. 12.04% for the baseline</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Program-level feedback can be integrated during denoising to improve correctness and security with focused edits rather than whole-program regeneration.</strong>
  <p>The result is only as complete as the tests, surrogate, and static analyzer used to evaluate the program. Passing these benchmarks does not rule out untested behavior or unknown vulnerabilities, so consequential code still requires review and execution in an appropriate validation environment.</p>
</div>

<p class="paper-link">Program constraints · <a href="https://arxiv.org/abs/2605.16829">Shao et al., 2026 →</a></p>

</section>

<section class="research-section" id="current-work" markdown="1">

<div class="section-label">04 · Current and ongoing work</div>

## Improving the masked-diffusion backbone

Constraint enforcement depends on the quality of the base model's clean-token predictions. Standard masked diffusion models infer still-masked positions largely from scratch at every step, discarding useful information from earlier predictions.

**Simple Self-Conditioning Adaptation for Masked Diffusion Models (SCMDM)** conditions each denoising step on the model's previous clean-state prediction. This allows uncertain positions to accumulate information across steps without introducing an additional recurrent latent state or increasing the number of denoiser evaluations at inference.

SCMDM reduces generative perplexity on OpenWebText from **42.89 to 23.72**, alongside improvements in molecular generation and genomic distribution modeling. Better clean-state predictions also give constraint projections a stronger proposal from which to start.

<div class="evidence-grid">
  <div><span>OWT perplexity</span><strong>23.72</strong><small>down from 42.89</small></div>
  <div><span>Reduction</span><strong>~50%</strong><small>in generative perplexity</small></div>
  <div><span>Extra denoiser calls</span><strong>0</strong><small>at inference time</small></div>
</div>

<p class="paper-link">Backbone adaptation · <a href="https://arxiv.org/abs/2604.26985">Cardei, Ta &amp; Fioretto, 2026 →</a></p>

</section>

<section class="research-section" id="key-results" markdown="1">

<div class="section-label">05 · Results overview</div>

## Where constrained discrete diffusion helps

<div class="project-table-card project-table-card--results">
  <div class="project-table-card__header">
    <div>
      <span class="project-table-eyebrow">Summary across domains</span>
      <h4>Selected results from the research program</h4>
    </div>
    <span class="project-table-legend">A fast-reading map of methods, domains, and reported outcomes.</span>
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
          <td data-label="Result">392 novel BRENK-screened molecules vs. 108 for MDLM, with 0% BRENK violations.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Symbolic reasoning</span></th>
          <td data-label="Method">SearchDiff <a href="https://arxiv.org/abs/2602.02727">2026</a></td>
          <td data-label="Result">Improves property satisfaction over autoregressive and discrete-diffusion baselines.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Code generation</span></th>
          <td data-label="Method">CDC <a href="https://arxiv.org/abs/2605.16829">2026</a></td>
          <td data-label="Result">Improves functionality, security, and syntax through localized denoising-time edits.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">MDM backbone</span></th>
          <td data-label="Method">SCMDM <a href="https://arxiv.org/abs/2604.26985">2026</a></td>
          <td data-label="Result">About 50% perplexity reduction, from 42.89 to 23.72, with no extra inference calls.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Neuro-symbolic</span></th>
          <td data-label="Method">NSD <a href="https://arxiv.org/abs/2506.01121">NeuS 2025</a></td>
          <td data-label="Result">Symbolic constraint feedback across continuous and discrete domains.</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

<div class="research-callout research-callout--thesis">
  <span class="research-callout__eyebrow">The broader principle</span>
  <strong>Discrete diffusion turns sequence generation into an editable process in which global requirements can be checked before the final output is committed.</strong>
  <p>Projection is one implementation of that idea. Symbolic feedback, search, and self-conditioned backbones extend it to constraints and domains that require richer evaluators.</p>
</div>

</section>

<section class="research-section research-section--papers" id="papers" markdown="1">

<div class="section-label">06 · Papers in this guide</div>

## Follow the research program

<div class="paper-list">
  <a href="https://arxiv.org/abs/2503.09790"><span>NeurIPS 2025</span><strong>Constrained Discrete Diffusion</strong><small>Cardei, Christopher, Hartvigsen, Kailkhura &amp; Fioretto</small></a>
  <a href="https://ai4d3.github.io/2025/papers/18_Constrained_Molecular_Gener.pdf"><span>AI4D3 2025 · Best Paper</span><strong>Constrained Molecular Generation with Discrete Diffusion for Drug Discovery</strong><small>Cardei, Christopher, Hartvigsen, Kailkhura &amp; Fioretto</small></a>
  <a href="https://arxiv.org/abs/2602.02727"><span>2026</span><strong>Search-Augmented Masked Diffusion Models for Constrained Generation</strong><small>Ta, Cardei, Velasquez &amp; Fioretto</small></a>
  <a href="https://arxiv.org/abs/2605.16829"><span>2026</span><strong>Constrained Code Generation with Discrete Diffusion</strong><small>Shao, Cardei, Xie, Fioretto &amp; Wang</small></a>
  <a href="https://arxiv.org/abs/2604.26985"><span>2026</span><strong>Simple Self-Conditioning Adaptation for Masked Diffusion Models</strong><small>Cardei, Ta &amp; Fioretto</small></a>
  <a href="https://arxiv.org/abs/2506.01121"><span>NeuS 2025</span><strong>Neuro-Symbolic Generative Diffusion Models</strong><small>Christopher, Cardei, Liang &amp; Fioretto</small></a>
  <a href="https://arxiv.org/abs/2406.07524"><span>NeurIPS 2024</span><strong>Simple and Effective Masked Diffusion Language Models</strong><small>Sahoo et al.</small></a>
</div>

</section>
