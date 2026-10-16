---
layout: research_area_preview
title: Constraint-Aware Generative AI — Preview
description: A research-area guide to generative models that obey scientific constraints.
permalink: /preview/constrained-aware-genai/
nav: false
giscus_comments: false
---

<div class="research-area-page" markdown="1">

<header class="research-hero">
  <div class="research-hero__copy">
    <div class="research-kicker">Research area guide <span>·</span> Fioretto Lab</div>
    <h1>Constraint-aware<br>generative AI</h1>
    <p class="research-deck">How to make diffusion and flow-matching models obey physics, geometry, logic, and safety requirements—without giving up the diversity that makes generative models useful.</p>
    <div class="research-meta" aria-label="Article metadata">
      <span>8 projects synthesized</span>
      <span>4 application areas</span>
      <span>~18 min read</span>
    </div>
    <div class="research-actions">
      <a class="research-button research-button--primary" href="#start-here">Start with the intuition</a>
      <a class="research-button" href="#research-landscape">Explore the landscape</a>
    </div>
  </div>
  <div class="research-hero__visual" aria-label="A sample alternates between denoising and constraint correction until it becomes feasible">
    <div class="constraint-loop">
      <div class="constraint-loop__label">The recurring idea</div>
      <div class="constraint-loop__stage">
        <div class="constraint-loop__sample constraint-loop__sample--raw"><span></span><span></span><span></span><span></span></div>
        <div class="constraint-loop__arrow"><span>Denoise</span>→</div>
        <div class="constraint-loop__sample constraint-loop__sample--plausible"><span></span><span></span><span></span><span></span></div>
        <div class="constraint-loop__arrow constraint-loop__arrow--accent"><span>Correct</span>→</div>
        <div class="constraint-loop__sample constraint-loop__sample--feasible"><span></span><span></span><span></span><span></span></div>
      </div>
      <div class="constraint-loop__legend"><span>Learned prior</span><span>Constraint set</span></div>
      <p>Let the model propose. Let optimization enforce. Repeat throughout generation.</p>
    </div>
  </div>
</header>

<div class="research-shell">
  <aside class="research-toc" aria-label="On this page">
    <div class="research-toc__title">On this page</div>
    <a href="#start-here">The problem</a>
    <a href="#core-idea">Core idea</a>
    <a href="#research-landscape">Research landscape</a>
    <a href="#protein-design">Protein design</a>
    <a href="#materials">Materials</a>
    <a href="#robotics">Robotics</a>
    <a href="#chemistry">Chemistry & code</a>
    <a href="#frontier">The frontier</a>
    <a href="#limitations">Limits & open questions</a>
    <a href="#papers">Papers</a>
  </aside>

  <article class="research-article">

<section class="research-section research-section--lead" id="start-here" markdown="1">

<div class="section-label">01 · Start here</div>

## A realistic sample is not necessarily a valid one

Diffusion models are excellent at learning what their training data *looks like*. Scientific and engineering systems demand something stricter: every output may also need to satisfy equations, geometric relationships, safety margins, or logical rules.

That distinction is easy to miss. A protein backbone can look natural while violating a bond-angle requirement. A robot path can be smooth while crossing an obstacle. A generated material can resemble the training distribution while having impossible interatomic distances.

<div class="research-callout research-callout--thesis">
  <span class="research-callout__eyebrow">The thesis</span>
  <strong>Generation should be treated as a constrained decision process—not as “sample first, repair later.”</strong>
  <p>Our work inserts constraint enforcement inside the generative trajectory so that plausibility and feasibility are negotiated at every step.</p>
</div>

<div class="research-compare" aria-label="Comparison of common constraint strategies">
  <div>
    <span class="research-compare__index">A</span>
    <h3>Generate, then filter</h3>
    <p>Simple, but wastes samples and can collapse diversity when feasible outputs are rare.</p>
  </div>
  <div>
    <span class="research-compare__index">B</span>
    <h3>Condition or retrain</h3>
    <p>Powerful when data and compute are abundant, but expensive and tied to a particular constraint family.</p>
  </div>
  <div class="research-compare__active">
    <span class="research-compare__index">C</span>
    <h3>Constrain while sampling</h3>
    <p>Wrap a pretrained model with projections, proximal steps, solvers, or simulators that steer each generation.</p>
  </div>
</div>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/constrained-diffusion/Fig1.png" title="Standard and constrained diffusion trajectories" class="research-figure__img" %}
  <figcaption><span>Figure 1</span> Standard sampling can leave the feasible set even when the output remains visually plausible. Constraint-aware sampling repeatedly returns the trajectory toward the set of valid solutions.</figcaption>
</figure>

</section>

<section class="research-section" id="core-idea" markdown="1">

<div class="section-label">02 · Core idea</div>

## Turn every denoising step into a small constrained optimization problem

A diffusion model gradually transforms noise $$x_T$$ into a structured sample $$x_0$$. One view of a reverse step is noisy ascent along the model's learned score:

$$x_t \leftarrow x_t + \gamma_t \nabla_{x_t} \log p(x_t) + \sqrt{2\gamma_t}\,\epsilon.$$

If valid outputs live in a set $$C$$, we can pair that model step with a projection:

$$\mathcal{P}_C(x_t) = \arg\min_{y \in C} \lVert y-x_t \rVert_2^2.$$

<div class="plain-language">
  <div class="plain-language__mark">In plain language</div>
  <p>The model says, “this direction looks more like the data.” The constraint operator replies, “this nearby point is valid.” Repeating that conversation lets the pretrained model supply realism while optimization supplies guarantees.</p>
</div>

<figure class="research-figure">
  {% include figure.html path="assets/img/projects/constrained-diffusion/obj_scheme.png" title="Alternating denoising and constraint enforcement" class="research-figure__img" %}
  <figcaption><span>Figure 2</span> The reusable pattern: a learned generative update alternates with an explicit feasibility update throughout the reverse process.</figcaption>
</figure>

<details class="research-details">
  <summary>Technical depth: what changes when constraints are difficult?</summary>
  <div markdown="1">

  **Convex constraints.** Exact projections are often efficient, and the distance to the feasible set can be shown to contract. Under the assumptions in our analysis, an $$\varepsilon$$-feasible sample is reached in $$\mathcal{O}(\gamma_{\min}^{-1}\log(1/\varepsilon))$$ steps.

  **Non-convex constraints.** Augmented Lagrangian and ADMM-style updates replace a single exact projection with local primal–dual corrections.

  **Latent-space models.** A constraint such as porosity or stress is evaluated after decoding $$z_t$$, then differentiated back through the decoder. This makes the constraint visible to a model operating in a compressed representation.

  **Black-box simulators.** When a finite-element or physics simulator is not differentiable, calibrated perturbations estimate a useful correction direction.

  </div>
</details>

</section>

<section class="research-section" id="research-landscape" markdown="1">

<div class="section-label">03 · Research landscape</div>

## One principle, several kinds of constraints

The framework is best understood as a research program rather than a single algorithm. Different domains change the representation, the constraint operator, and the appropriate solver—but preserve the same division of labor between a learned prior and explicit domain knowledge.

<div class="landscape-grid">
  <a href="#protein-design"><span>Continuous · geometric</span><strong>Protein design</strong><small>Motifs, distances, bonding</small></a>
  <a href="#materials"><span>Continuous · simulator-based</span><strong>Materials</strong><small>Porosity, stress, energy</small></a>
  <a href="#robotics"><span>Hybrid · spatiotemporal</span><strong>Robotics</strong><small>Collision, dynamics, logic</small></a>
  <a href="#chemistry"><span>Discrete · symbolic</span><strong>Chemistry & code</strong><small>Valency, toxicity, syntax</small></a>
</div>

<div class="result-strip" aria-label="Selected headline results">
  <div><strong>100%</strong><span>protein constraint satisfaction</span></div>
  <div><strong>4.6×</strong><span>better stress–strain error</span></div>
  <div><strong>100</strong><span>robots planned jointly</span></div>
  <div><strong>0%</strong><span>toxicity-rule violations</span></div>
</div>

</section>

<section class="research-section research-domain" id="protein-design" markdown="1">

<div class="domain-heading"><span>Application 01</span><div class="domain-heading__dot domain-heading__dot--protein"></div></div>

## Protein design with hard structural constraints

<p class="section-standfirst">The challenge is not merely to generate a plausible backbone. It is to preserve the exact geometric relationships that make a functional motif work while allowing the surrounding scaffold to vary.</p>

Our constrained RFdiffusion work combines proximal feasibility updates with ADMM decomposition. On a benchmark for PDZ-domain motif scaffolding and pocket design, it achieved **100% satisfaction of bonding and geometric constraints**, compared with **0% for three baselines** across nearly 100,000 generated samples.

<div class="evidence-grid">
  <div><span>Constraint satisfaction</span><strong>100%</strong><small>vs. 0% for all three baselines</small></div>
  <div><span>Usable pocket designs</span><strong>97.8%</strong><small>about 4× the nearest baseline</small></div>
  <div><span>Well-posed ligands</span><strong>83%</strong><small>viable motif-scaffolding designs</small></div>
</div>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/constrained-diffusion/protein_design_slide.png" title="Protein design results with hard constraints" class="research-figure__img" %}
  <figcaption><span>Figure 3</span> Constraint-aware RFdiffusion preserves the required functional geometry and produces usable structures where unconstrained and guided baselines fail the global constraints.</figcaption>
</figure>

<p class="paper-link">Read the paper · <a href="https://arxiv.org/abs/2510.14989">Christopher et al., ICLR 2026 →</a></p>

</section>

<section class="research-section research-domain" id="materials" markdown="1">

<div class="domain-heading"><span>Application 02</span><div class="domain-heading__dot domain-heading__dot--materials"></div></div>

## Materials discovery with properties in the loop

<p class="section-standfirst">Material validity is often revealed only by an expensive evaluator: a stress–strain simulation, an energy model, or a structural surrogate. The constraint operator can turn that evaluator into feedback during sampling.</p>

Across microstructure generation, mechanical metamaterial design, and safety-aware image generation, latent constrained diffusion improves feasibility without retraining the base model. The key move is to evaluate a property after decoding and propagate the resulting correction back to the latent sample.

<div class="research-figure-pair">
  <figure class="research-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/constrained_ai4mat.png" title="Surrogate-guided crystal denoising" class="research-figure__img" %}
    <figcaption><span>Figure 4a</span> A learned surrogate checks physical requirements at every denoising step.</figcaption>
  </figure>
  <figure class="research-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/metamaterials_results_slide.png" title="Mechanical metamaterial inverse design" class="research-figure__img" %}
    <figcaption><span>Figure 4b</span> Samples move toward a target stress–strain response while invalid geometries fall to 5%.</figcaption>
  </figure>
</div>

<div class="evidence-grid">
  <div><span>Morphometric violations</span><strong>0%</strong><small>vs. 68.4% for conditional diffusion</small></div>
  <div><span>Stress–strain MSE</span><strong>1.4 ± 0.6</strong><small>4.6× better than a specialized baseline</small></div>
  <div><span>Safety compliance</span><strong>90%</strong><small>vs. 67% for conditional diffusion</small></div>
</div>

<p class="paper-link">Read the paper · <a href="https://arxiv.org/abs/2502.05625">Zampini et al., NeurIPS 2025 Spotlight →</a></p>

</section>

<section class="research-section research-domain" id="robotics" markdown="1">

<div class="domain-heading"><span>Application 03</span><div class="domain-heading__dot domain-heading__dot--robotics"></div></div>

## Safe motion planning at generative speed

<p class="section-standfirst">A multi-robot plan must satisfy many coupled constraints at once: obstacles, inter-robot collisions, velocity and acceleration limits, and the dynamics connecting one state to the next.</p>

**SMD** projects diffusion trajectories toward simultaneous feasibility. On dense nine-robot maps it reaches **96% success**, versus 27% for the best baseline. **DGD** adds guidance from a discrete multi-agent path-finding solver, decomposing a difficult non-convex problem into tractable subproblems and scaling to **100 robots**.

<div class="motion-reel" aria-label="Five looping multi-robot planning examples">
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_baisc.mp4" type="video/mp4"></video><span>Basic</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_dense.mp4" type="video/mp4"></video><span>Dense</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_corridor.mp4" type="video/mp4"></video><span>Corridor</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_shelf.mp4" type="video/mp4"></video><span>Shelf</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_room.mp4" type="video/mp4"></video><span>Room</span></div>
</div>
<p class="research-caption"><span>Figure 5</span> Motion is the one place where animation materially improves understanding: feasibility is about an entire trajectory, not a single final frame.</p>

<div class="evidence-grid">
  <div><span>Dense-map success</span><strong>96%</strong><small>vs. 27% for the best baseline</small></div>
  <div><span>Maximum scale</span><strong>100 robots</strong><small>with 104 obstacles</small></div>
  <div><span>DGD speedup</span><strong>20×</strong><small>over SMD on comparable settings</small></div>
</div>

<p class="paper-link">Read the papers · <a href="https://arxiv.org/abs/2502.03607">SMD, ICML 2025</a> · <a href="https://arxiv.org/abs/2508.20095">DGD, AAAI 2026 →</a></p>

</section>

<section class="research-section research-domain" id="chemistry" markdown="1">

<div class="domain-heading"><span>Application 04</span><div class="domain-heading__dot domain-heading__dot--chemistry"></div></div>

## Constraints in discrete spaces: molecules, text, and code

<p class="section-standfirst">In discrete diffusion, “move to the nearest feasible point” cannot mean nudging coordinates. It means changing probability mass over tokens without destroying the model's learned preferences.</p>

**Constrained Discrete Diffusion (CDD)** projects token probabilities onto a feasible region while minimizing KL divergence from the model's original logits. This allows rules such as chemical valency, toxicity thresholds, token counts, or syntax to be enforced during generation.

<figure class="research-figure">
  {% include figure.html path="assets/img/projects/constrained-diffusion/Fig6.png" title="Constrained discrete diffusion for molecular generation" class="research-figure__img" %}
  <figcaption><span>Figure 6</span> Constraint checks accompany the progressive unmasking process, so the final molecule respects the specified rules.</figcaption>
</figure>

<div class="evidence-grid">
  <div><span>Rule violations</span><strong>0%</strong><small>for toxicity, counting, and lexical tests</small></div>
  <div><span>Novel safe molecules</span><strong>392</strong><small>vs. 108 for MDLM and 5 autoregressively</small></div>
  <div><span>Beyond chemistry</span><strong>Code</strong><small>syntax and semantic structure constraints</small></div>
</div>

<p class="paper-link">Read the papers · <a href="https://arxiv.org/abs/2503.09790">CDD, NeurIPS 2025</a> · <a href="https://arxiv.org/abs/2605.16829">Constrained code generation, 2026 →</a></p>

</section>

<section class="research-section" id="frontier" markdown="1">

<div class="section-label">04 · The frontier</div>

## From correcting a pretrained model to training for constraints

Training-free guidance is flexible, but it creates a mismatch: the model learns unconstrained trajectories during training and encounters projected trajectories only at inference. Two newer directions attack that mismatch from opposite sides.

<div class="frontier-grid">
  <div>
    <span>Continuous models</span>
    <h3>Constraint-Aware Flow Matching</h3>
    <p>CAFM includes projection in the training objective, teaching the velocity field to anticipate the constrained sampler. The aim is a better quality–feasibility trade-off with strict per-sample guarantees.</p>
    <a href="https://arxiv.org/abs/2605.12754">Christopher et al., 2026 →</a>
  </div>
  <div>
    <span>Discrete models</span>
    <h3>Search-Augmented Diffusion</h3>
    <p>SearchDiff combines masked diffusion with tree search, spending extra computation to explore alternative token sequences when direct projection is not enough.</p>
    <a href="https://arxiv.org/abs/2602.02727">Ta et al., 2026 →</a>
  </div>
</div>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/constrained-diffusion/cafm_overview_slide.png" title="Constraint-aware flow matching" class="research-figure__img" %}
  <figcaption><span>Figure 7</span> CAFM aligns the learned velocity field with the constrained sampling path instead of correcting an unconstrained field only at inference.</figcaption>
</figure>

</section>

<section class="research-section" id="limitations" markdown="1">

<div class="section-label">05 · Limits and open questions</div>

## What guarantees do—and do not—buy us

Constraint satisfaction is only as meaningful as the constraint itself. A mathematically feasible output can still be undesirable if the evaluator is incomplete, misspecified, or only a surrogate for the real scientific objective.

<div class="questions-list">
  <div><span>01</span><p><strong>Evaluator fidelity.</strong> How should uncertainty in learned surrogates or approximate simulators propagate into a feasibility claim?</p></div>
  <div><span>02</span><p><strong>Competing constraints.</strong> What happens when requirements are mutually inconsistent, or feasibility leaves too little room for diversity?</p></div>
  <div><span>03</span><p><strong>Non-convex scale.</strong> Can local projection and decomposition methods retain useful guarantees on the largest scientific design spaces?</p></div>
  <div><span>04</span><p><strong>Training–sampling alignment.</strong> When is a training-free wrapper enough, and when is end-to-end constraint-aware training worth the added cost?</p></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">The broader principle</span>
  <strong>A generative model supplies a prior over plausible designs. Domain constraints turn that prior into a decision-making tool.</strong>
</div>

</section>

<section class="research-section research-section--papers" id="papers" markdown="1">

<div class="section-label">06 · Papers in this guide</div>

## Follow the research program

<div class="paper-list">
  <a href="https://arxiv.org/abs/2510.14989"><span>ICLR 2026</span><strong>Constrained Diffusion for Protein Design with Hard Structural Constraints</strong><small>Christopher, Seamann, Cui, Khare & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2502.05625"><span>NeurIPS 2025 Spotlight</span><strong>Training-Free Constrained Generation With Stable Diffusion Models</strong><small>Zampini, Christopher, Oneto, Anguita & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2503.09790"><span>NeurIPS 2025</span><strong>Constrained Discrete Diffusion</strong><small>Cardei, Christopher, Hartvigsen, Kailkhura & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2502.03607"><span>ICML 2025</span><strong>Simultaneous Multi-Robot Motion Planning with Projected Diffusion Models</strong><small>Liang, Christopher, Koenig & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2508.20095"><span>AAAI 2026</span><strong>Discrete-Guided Diffusion for Scalable and Safe Multi-Robot Motion Planning</strong><small>Liang, Koenig & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2605.12754"><span>2026</span><strong>Constraint-Aware Flow Matching</strong><small>Christopher, Warner & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2602.02727"><span>2026</span><strong>Search-Augmented Masked Diffusion Models for Constrained Generation</strong><small>Ta, Cardei, Velasquez & Fioretto</small></a>
  <a href="https://arxiv.org/abs/2605.16829"><span>2026</span><strong>Constrained Code Generation with Discrete Diffusion</strong><small>Shao, Cardei, Xie, Fioretto & Wang</small></a>
</div>

</section>

  </article>
</div>

</div>
