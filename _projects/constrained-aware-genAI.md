---
layout: page
title: Constraint-Aware Diffusion for Scientific Applications
description: Teaching diffusion models to obey physics, geometry, and safety constraints — without retraining.
img: assets/img/projects/constrained-diffusion/highlight-constrained-aware.png
importance: 1
highlight: true
category: work
giscus_comments: true
---

<div class="constrained-aware-page" markdown="1">

## Problem Statement and Motivation

Generative AI — especially diffusion and flow-matching models — has achieved transformative performance across image synthesis, protein structure prediction, and materials design. Yet a critical gap limits their deployment in high-stakes scientific and engineering domains: **these models generate statistically plausible outputs that routinely violate physical laws, geometric constraints, and safety specifications**.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/Fig1.png" title="Standard vs. constrained diffusion" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 1</strong>: Standard diffusion trajectories diverge from the constraint manifold, producing outputs that may look realistic but violate physical requirements. Constrained diffusion <em>projects and adheres</em> to the manifold throughout generation — guaranteeing feasibility at every step.
</div>
</div>

Consider the challenge. A diffusion model trained on protein structures generates smooth, realistic-looking backbones — but some violate bond-angle constraints critical for function. A trajectory-planning model produces elegant robot paths, but some pass through solid obstacles. A materials-discovery model proposes crystal lattices with impossible interatomic distances. The models *look* right, but *aren't*.

This reflects a fundamental mismatch: diffusion models optimize for statistical faithfulness to training data, not constraint satisfaction. Retrofitting constraints post-hoc (filtering, rejection sampling) wastes computation and degrades diversity. Retraining massive foundational models with constraint-aware objectives is prohibitively expensive and domain-specific.

Our group addresses this from a fundamentally different angle: **making constraint enforcement a native, training-free operation inside the diffusion sampling loop**.

### Key Takeaways

<div class="takeaways-grid">
    <div class="takeaway-card">Constraints are enforced <em>training-free</em> by integrating projection and proximal steps directly into the reverse diffusion process.</div>
    <div class="takeaway-card">The framework is plug-and-play: it wraps any pre-trained diffusion or flow-matching model without touching model weights.</div>
    <div class="takeaway-card">Hard constraint satisfaction is provably guaranteed in convex settings and efficiently approximated for non-convex domains.</div>
    <div class="takeaway-card">Applications span protein design, materials discovery, robotics and motion planning, and chemical molecule generation.</div>
</div>


## The Core Framework: Diffusion as Sequential Constrained Optimization

To understand our approach, we first observe that the reverse diffusion process can be recast as a sequence of optimization steps. Denoising Diffusion Probabilistic Models define a forward process that corrupts data $$x_0 \sim p_{\text{data}}(x_0)$$ into noise $$x_T$$. The learned reverse process recovers data via Gradient Langevin Dynamics (GLD):

$$x_t \leftarrow x_t + \gamma_t \nabla_{x_t} \log p(x_t) + \sqrt{2\gamma_t}\,\epsilon$$

As the variance schedule decreases toward $$t \to 0$$, GLD transitions toward deterministic gradient ascent on the learned log-likelihood. This allows us to view each reverse step as solving a local optimization problem — and, critically, to **augment each step with a feasibility projection**:

$$\mathcal{P}_C(x_t) = \arg\min_{y \in C} \|y - x_t\|_2^2$$

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/obj_scheme.png" title="Iterative constraint enforcement in diffusion sampling" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 2</strong>: Constraint-Aware Diffusion alternates between score-guided denoising steps (following the learned density) and constraint enforcement steps (projecting back into the feasible region). Applied iteratively across the entire Markov chain, this guarantees that the final sample lies within the specified feasible region.
</div>
</div>

Our **Constrained-Aware Diffusion Models (CADM)** apply this projection at every step of the reverse Markov chain. The result is a formal guarantee: for convex constraint sets, the distance to the feasible region decreases at rate $$1 - 2\beta\gamma_t$$ per step, reaching an $$\varepsilon$$-feasible sample in $$\mathcal{O}(\gamma_{\min}^{-1} \log(1/\varepsilon))$$ steps — with no degradation of sample diversity.

### Scaling to Non-Convex and Latent-Space Constraints

Scientific constraints are rarely convex. We address this through two extensions:

- **Augmented Lagrangians**: For non-convex constraint sets, we introduce dual multipliers $$\lambda_j \geq 0$$ and solve a local augmented problem at each step, enabling rapid inference without exact non-convex projections.

- **Proximal updates in latent space**: Stable Diffusion operates in a compressed latent space $$z_t$$. Constraint evaluation (e.g., porosity, stress) requires decoding to pixel space. We construct a differentiable computational graph that evaluates $$g(\mathcal{D}(z_t))$$ and propagates gradients back through the decoder:

$$\text{prox}_\lambda(g(\mathcal{D}(z_t))) = \arg\min_{y} \left\{ g(\mathcal{D}(y)) + \frac{1}{2\lambda} \|\mathcal{D}(y) - \mathcal{D}(z_t)\|_2^2 \right\}$$

- **Simulators-in-the-loop**: When constraint evaluators (e.g., Finite Element Analysis) are non-differentiable, we use calibrated Monte Carlo perturbations to estimate pseudo-gradients, enabling the diffusion model to adjust outputs until simulated constraints are met.


## Applications Across Scientific Domains

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/Fig2.png" title="Constrained generative AI across scientific domains" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 3</strong>: Our constraint-aware generative framework has been applied across four scientific domains, each requiring strict adherence to domain-specific constraints: geometric and stability constraints in protein design, valency and chemical structure in molecular generation, thermodynamic constraints in materials discovery, and collision-free physical limits in robotics and control.
</div>
</div>

### Protein Design with Hard Structural Constraints

Designing novel proteins requires strict adherence to geometric constraints — bond lengths, angles, and inter-residue distances — to ensure structural stability and functional affinity. Standard diffusion-based protein design tools (e.g., RFdiffusion) can hallucinate backbone geometries that fail costly downstream validation.

<div class="project-figure project-figure--float-right">
    {% include figure.html path="assets/img/projects/constrained-diffusion/Fig3.png" title="Protein motif scaffolding with hard geometric constraints" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 4</strong>: Our constrained diffusion framework generates protein backbones that precisely preserve critical functional motifs (here: a binding pocket with a distance constraint of 3.2 Å) while freely designing the surrounding scaffold.
</div>
</div>

Our work ([Christopher et al., ICLR 2026](https://arxiv.org/abs/2510.14989)) integrates proximal feasibility updates with ADMM decomposition into the RFdiffusion generative process, enforcing hard structural constraints while preserving geometric and stereochemical diversity. We introduce a curated benchmark for **motif scaffolding in the PDZ domain** and demonstrate:

- **100% constraint satisfaction** on bonding and geometric constraints — compared to **0% for all three baselines** (Standard RFdiffusion, Recentering, Constraint-Guided Diffusion) across nearly 100,000 generated samples.
- **97.8% usable samples** on molecule encapsulation / pocket design — approximately **4× the nearest baseline** (24.2%).
- **21% viable designs** on motif scaffolding overall, reaching **83% for well-posed ligands**, while baselines produced zero feasible designs.

The approach is the first to solve pocket-design problems at full backbone resolution with provable constraint adherence.

<div class="project-figure project-figure--wide">
    {% include figure.html path="assets/img/projects/constrained-diffusion/protein_design_slide.png" title="Protein design comparison: RFdiffusion baselines and CADM" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 5</strong>: Protein-design benchmark from the AI-SCORE presentation. CADM reaches 100% constraint satisfaction and produces usable structures where RFdiffusion baselines fail to satisfy the hard global constraints.
</div>
</div>

<div style="clear:both"></div>

### Materials Discovery: Constrained Generation for Physical Properties

Discovering structural materials often requires satisfying dynamic physical constraints — target porosity, specific stress-strain responses, interatomic repulsion, or boundary conditions — that are expensive to verify computationally.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/Fig4.png" title="Constrained diffusion relaxation for point defects" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 6</strong>: Constrained diffusion relaxation directly generates the stable, low-energy atomic configuration of a crystal with a point defect — bypassing thousands of DFT calculation steps.
</div>
</div>

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/constrained_ai4mat.png" title="ML surrogate-guided constraint enforcement in crystal denoising" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 7</strong>: At each denoising step, an ML surrogate evaluates constraint violations (repulsion, long-range forces, boundary, inter-atomic potentials) and feeds corrections back into the diffusion trajectory.
</div>
</div>

Our work ([Zampini et al., 2025](https://arxiv.org/abs/2502.05625)) evaluates the framework on three distinct materials-science benchmarks:

- **Morphometric constraints** (microstructure porosity): **0% constraint violations** with FID of 13.5, versus 68.4% violation rate for conditional diffusion — achieving strict constraint adherence while maintaining competitive generation quality.
- **Metamaterial inverse design** (stress-strain response): **MSE of 1.4 ± 0.6**, a **4.6× improvement** over the specialized Bastek & Kochmann baseline (MSE 6.4 ± 4.6), with only 5% invalid shapes versus 55% for conditional diffusion.
- **Copyright-safe generation**: **90% safety compliance** versus 67% for conditional diffusion and 71% for projected diffusion, with an FID of 65.1 maintaining image quality.

<div class="project-figure project-figure--wide">
    {% include figure.html path="assets/img/projects/constrained-diffusion/metamaterials_results_slide.png" title="Mechanical meta-material inverse design results" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 8</strong>: Inverse design for mechanical meta-materials. The latent constrained method steers diffusion samples toward target stress-strain behavior while reducing physically invalid shapes to 5%.
</div>
</div>


### Robotics and Multi-Robot Motion Planning

Motion planning in autonomous systems requires finding collision-free trajectories governed by ODE dynamics for velocity, acceleration, and spatial bounds. Standard generative models that ignore these constraints produce trajectories that collide with obstacles or violate kinematic limits.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/Fig5.png" title="Neuro-symbolic safe trajectory generation" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 9</strong>: Standard generative models produce unsafe trajectories that collide with obstacles. Our neuro-symbolic approach bakes physical boundaries and logic rules directly into the diffusion process, guaranteeing collision-free, physically grounded trajectories.
</div>
</div>

We have addressed robotics planning through two complementary systems:

**SMD — Simultaneous Multi-Robot Motion Planning** ([Liang et al., 2025](https://arxiv.org/abs/2502.03607)) integrates constrained optimization into the diffusion sampling process to produce simultaneously collision-free, kinematically feasible trajectories. On dense maps with 9 robots, SMD achieves **96% success** — compared to just **27% for the best baseline (MMD)**, a **3.6× improvement**. In structured environments (corridor maps), SMD reaches **100% success** while all baselines score 0%. Evaluated across 4,000 test instances spanning six environment types.

**DGD — Discrete-Guided Diffusion** ([Liang et al., AAAI 2026](https://arxiv.org/abs/2508.20095)) couples discrete MAPF solvers with constrained generative diffusion, decomposing the nonconvex multi-robot planning problem into tractable convex subproblems. DGD achieves **over 92% success** across standard benchmarks, scales to **100 robots** in environments with 104 obstacles — a **2.5× improvement in robot count** over prior diffusion methods — and is **20× faster** than SMD on comparable settings (12.2s vs 254.3s).

<div class="project-video-grid">
  <div class="project-video-item">
    <video autoplay muted loop playsinline>
      <source src="/assets/video/projects/constrained-diffusion/smd_baisc.mp4" type="video/mp4">
    </video>
    <p class="project-video-label">Basic map</p>
  </div>
  <div class="project-video-item">
    <video autoplay muted loop playsinline>
      <source src="/assets/video/projects/constrained-diffusion/smd_dense.mp4" type="video/mp4">
    </video>
    <p class="project-video-label">Dense map</p>
  </div>
  <div class="project-video-item">
    <video autoplay muted loop playsinline>
      <source src="/assets/video/projects/constrained-diffusion/smd_corridor.mp4" type="video/mp4">
    </video>
    <p class="project-video-label">Corridor map</p>
  </div>
  <div class="project-video-item">
    <video autoplay muted loop playsinline>
      <source src="/assets/video/projects/constrained-diffusion/smd_shelf.mp4" type="video/mp4">
    </video>
    <p class="project-video-label">Shelf map</p>
  </div>
  <div class="project-video-item">
    <video autoplay muted loop playsinline>
      <source src="/assets/video/projects/constrained-diffusion/smd_room.mp4" type="video/mp4">
    </video>
    <p class="project-video-label">Room map</p>
  </div>
</div>
<div class="caption" style="margin-top:0.5rem">
    <strong>Figure 10</strong>: SMD planning demonstrations across five environment types — basic, dense, corridor, shelf, and room — all generated with constraint-aware diffusion, achieving 96–100% success rates. See the <a href="https://multi-robot-constrained-diffusion.github.io" target="_blank">project page</a> and <a href="https://github.com/RAISELab-atUVA/Diffusion-MRMP" target="_blank">code</a>.
</div>

Notably, because we explicitly encode physics, these models can **generalize out-of-distribution**: generating valid trajectories under gravitational conditions never seen during training (e.g., lunar gravity).


### Chemistry: Constrained Discrete Diffusion for Molecular Generation

Chemical design is fundamentally discrete. Generating molecules in SMILES format requires selecting tokens from a finite vocabulary while satisfying valency rules, structural filters, and molecular property targets — simultaneously.

<div class="project-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/Fig6.png" title="Step-by-step discrete diffusion for valid molecule generation" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 11</strong>: Constrained Discrete Diffusion enforces chemical valency constraints at every denoising step, building up a valid molecule token-by-token. Each intermediate state satisfies the constraint; the final molecule is guaranteed chemically valid.
</div>
</div>

Our **Constrained Discrete Diffusion (CDD)** framework ([Cardei et al., 2025](https://arxiv.org/abs/2503.09790)) adapts the CADM principle to discrete sequence spaces by defining projections over the probability simplex that minimize the KL divergence between original logits and feasibility-constrained logits. Key results:

- **Zero constraint violations** on toxicity mitigation across all threshold levels, versus 33.2%, 21.6%, and 13.1% for GPT-2 and 17–32% for MDLM/UDLM baselines.
- **Zero violations** on counting and lexical constraints, versus 54.5% and 97.5% for unconstrained MDLM.
- **392 novel non-toxic molecules** in the AI4D3 molecular-generation benchmark, versus 108 for MDLM and 5 for autoregressive generation.
- Minimal computational overhead compared to PPLM (85× slowdown) and FUDGE (134–143× slowdown).

**Extending to code and structured text**: ([Shao et al., 2026](https://arxiv.org/abs/2605.16829)) applies constrained discrete diffusion to code generation, enforcing syntactic and semantic constraints on program structure directly in the generation process.


## The Frontier: Constraint-Aware Flow Matching

Training-free constraint enforcement introduces a fundamental mismatch: the model is trained on unconstrained trajectories but evaluated on constrained ones, inducing distributional shift that can degrade sample quality. Our most recent work eliminates this gap.

**Constraint-Aware Flow Matching (CAFM)** ([Christopher et al., 2026](https://arxiv.org/abs/2605.12754)) proposes an **end-to-end framework** that explicitly incorporates constraint projections into the *training objective* of flow-matching models. By aligning the model's learned velocity field with the constrained sampling procedure, CAFM eliminates distributional shift at its root, achieving higher constraint satisfaction rates and better sample quality than training-free methods — with strict per-sample feasibility guarantees.

<div class="project-figure project-figure--wide">
    {% include figure.html path="assets/img/projects/constrained-diffusion/cafm_overview_slide.png" title="Constraint-aware flow matching overview" class="img-fluid rounded z-depth-1 project-figure__img" %}
<div class="caption">
    <strong>Figure 12</strong>: The distribution-shift problem in post-hoc constraint correction. CAFM trains the velocity field to anticipate the constrained projection.
</div>
</div>

**Search-Augmented Masked Diffusion (SearchDiff)** ([Ta et al., 2026](https://arxiv.org/abs/2602.02727)) takes a complementary approach for discrete domains: augmenting masked diffusion with tree search to explore the token space and find constraint-satisfying sequences, achieving superior performance on constrained generation benchmarks.

<div class="project-figure-grid">
  <div class="project-figure project-figure--grid-item">
      {% include figure.html path="assets/img/projects/constrained-diffusion/cafm_results_slide.png" title="CAFM feasibility and accuracy results" class="img-fluid rounded z-depth-1 project-figure__img" %}
  </div>
  <div class="project-figure project-figure--grid-item">
      {% include figure.html path="assets/img/projects/constrained-diffusion/cafm_coherence_slide.png" title="CAFM spatiotemporal coherence results" class="img-fluid rounded z-depth-1 project-figure__img" %}
  </div>
</div>
<div class="caption" style="margin-top:0.5rem">
    <strong>Figure 13</strong>: Flow-matching results from the slide deck. CAFM reduces constraint violations by orders of magnitude while preserving accuracy and suppressing physically impossible high-variance structures.
</div>


## Key Results

<div class="project-table-card project-table-card--results">
  <div class="project-table-card__header">
    <div>
      <span class="project-table-eyebrow">Summary across scientific domains</span>
      <h4>What constraint-aware generative modeling changes</h4>
    </div>
    <span class="project-table-legend">Use this as the fast-reading map for the project.</span>
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
          <th scope="row" data-label="Domain"><span class="domain-pill">Protein design</span></th>
          <td data-label="Method">CADM + ADMM <a href="https://arxiv.org/abs/2510.14989">ICLR 2026</a></td>
          <td data-label="Result">100% constraint satisfaction vs. 0% for all baselines; 97.8% usable samples, about 4× the nearest baseline.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Materials science</span></th>
          <td data-label="Method">Latent constrained diffusion <a href="https://arxiv.org/abs/2502.05625">NeurIPS 2025</a></td>
          <td data-label="Result">4.6× improvement in stress-strain MSE; 0% morphometric violations vs. 68.4% for conditional diffusion.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Multi-robot planning</span></th>
          <td data-label="Method">SMD <a href="https://arxiv.org/abs/2502.03607">ICML 2025</a></td>
          <td data-label="Result">96% success on dense maps vs. 27% for the best baseline; 100% vs. 0% in corridor maps.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Scalable planning</span></th>
          <td data-label="Method">DGD <a href="https://arxiv.org/abs/2508.20095">AAAI 2026</a></td>
          <td data-label="Result">Scales to 100 robots, a 2.5× increase; 20× faster than SMD with 92%+ success across benchmarks.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Molecular generation</span></th>
          <td data-label="Method">CDD <a href="https://arxiv.org/abs/2503.09790">2025</a></td>
          <td data-label="Result">0% toxicity and lexical violations; 392 novel non-toxic molecules vs. 108 for MDLM.</td>
        </tr>
        <tr>
          <th scope="row" data-label="Domain"><span class="domain-pill">Flow matching</span></th>
          <td data-label="Method">CAFM <a href="https://arxiv.org/abs/2605.12754">2026</a></td>
          <td data-label="Result">Eliminates the training-sampling distribution shift and improves the quality-satisfaction trade-off.</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

Beyond individual results, this body of work establishes a general principle: **constraint satisfaction can be treated as a first-class citizen in generative modeling**, achieved through the integration of differentiable optimization into the generative process — rather than bolted on as post-processing.


## Relevant Citations

- Christopher, J. K., Seamann, A., Cui, J., Khare, S., Fioretto, F. (2026). Constrained Diffusion for Protein Design with Hard Structural Constraints. *ICLR 2026*. [arXiv:2510.14989](https://arxiv.org/abs/2510.14989)
- Zampini, S., Christopher, J. K., Oneto, L., Anguita, D., Fioretto, F. (2025). Training-Free Constrained Generation With Stable Diffusion Models. *NeurIPS 2025 Spotlight*. [arXiv:2502.05625](https://arxiv.org/abs/2502.05625)
- Cardei, M., Christopher, J. K., Hartvigsen, T., Kailkhura, B., Fioretto, F. (2025). Constrained Discrete Diffusion. *NeurIPS 2025*. [arXiv:2503.09790](https://arxiv.org/abs/2503.09790)
- Liang, J., Christopher, J. K., Koenig, S., Fioretto, F. (2025). Simultaneous Multi-Robot Motion Planning with Projected Diffusion Models. *ICML 2025*. [arXiv:2502.03607](https://arxiv.org/abs/2502.03607)
- Liang, J., Koenig, S., Fioretto, F. (2026). Discrete-Guided Diffusion for Scalable and Safe Multi-Robot Motion Planning. *AAAI 2026*. [arXiv:2508.20095](https://arxiv.org/abs/2508.20095)
- Christopher, J. K., Warner, J. E., Fioretto, F. (2026). Constraint-Aware Flow Matching: Decision Aligned End-to-End Training for Constrained Sampling. [arXiv:2605.12754](https://arxiv.org/abs/2605.12754)
- Ta, H. B., Cardei, M., Velasquez, A., Fioretto, F. (2026). Search-Augmented Masked Diffusion Models for Constrained Generation. [arXiv:2602.02727](https://arxiv.org/abs/2602.02727)
- Shao, L., Cardei, M., Xie, Z., Fioretto, F., Wang, W. (2026). Constrained Code Generation with Discrete Diffusion. [arXiv:2605.16829](https://arxiv.org/abs/2605.16829)

</div>
