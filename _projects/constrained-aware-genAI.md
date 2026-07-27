---
layout: research_area
title: Constraint-Aware Generative AI for Science and Engineering
description: How RAISE Lab makes generative models obey physics, geometry, logic, and safety requirements.
img: assets/img/projects/constrained-diffusion/highlight-constrained-aware-v2.png
importance: 1
highlight: true
category: work
giscus_comments: true
giscus_term: Constraint-Aware Diffusion for Scientific Applications
giscus_strict: 0
---

<div class="research-area-page" markdown="1">

<header class="research-hero">
  <div class="research-hero__copy">
    <div class="research-lab-brand">
      <img src="{{ '/assets/img/projects/constrained-diffusion/raise-lab-logo.png' | relative_url }}" alt="RAISE Lab — Responsible AI for Science and Engineering">
      <div><strong>RAISE Lab</strong><span>Responsible AI for Science & Engineering</span></div>
    </div>
    <div class="research-kicker">Research area guide <span>·</span> University of Virginia</div>
    <h1>Constraint-aware<br>generative AI</h1>
    <p class="research-deck">How to make diffusion and flow-matching models obey physics, geometry, logic, and safety requirements—without giving up the diversity that makes generative models useful.</p>
    <div class="research-meta" aria-label="Article metadata">
      <span>9 papers synthesized</span>
      <span>4 application areas</span>
      <span>~22 min read</span>
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
    <a href="#start-here">Diffusion primer</a>
    <a href="#core-idea">Core idea</a>
    <a href="#research-landscape">Research landscape</a>
    <a href="#protein-design">Protein design</a>
    <a href="#materials">Materials</a>
    <a href="#robotics">Robotics</a>
    <a href="#chemistry">Chemistry & code</a>
    <a href="#current-work">Current work</a>
    <a href="#limitations">Limits & open questions</a>
    <a href="#papers">Papers</a>
  </aside>

  <article class="research-article">

<section class="research-section research-section--lead" id="start-here" markdown="1">

<div class="section-label">01 · Start here</div>

## From data to noise—and back again

A diffusion model learns to generate by first studying how data is destroyed. Given a clean sample $$x_0 \sim p_{\mathrm{data}}$$, the **forward process** adds a small amount of Gaussian noise at each step:

$$q(x_t \mid x_{t-1}) = \mathcal{N}\!\left(\sqrt{1-\beta_t}\,x_{t-1},\,\beta_t I\right).$$

Writing $$\alpha_t = 1-\beta_t$$ and $$\bar\alpha_t=\prod_{s=1}^{t}\alpha_s$$ lets us jump directly to any noise level:

$$x_t = \sqrt{\bar\alpha_t}\,x_0 + \sqrt{1-\bar\alpha_t}\,\epsilon, \qquad \epsilon \sim \mathcal{N}(0,I).$$

Early states retain most of the structure in $$x_0$$; late states are almost indistinguishable from Gaussian noise. The generative problem is to learn the reverse direction: starting from $$x_T \sim \mathcal{N}(0,I)$$, recover a sequence of progressively cleaner states until reaching a sample $$x_0$$.

<div class="diffusion-steps" aria-label="Forward noising and reverse denoising processes">
  <div><span>Forward process</span><strong>Data → noise</strong><small>A fixed variance schedule gradually erases structure.</small></div>
  <div class="diffusion-steps__arrow">⇄</div>
  <div><span>Learned reverse process</span><strong>Noise → data</strong><small>A neural score estimates which direction increases probability.</small></div>
</div>

### Score-based denoising

Rather than learning the normalized density $$p_t(x)$$ itself, a score-based model learns its spatial gradient:

$$s_\theta(x_t,t) \approx \nabla_{x_t}\log p_t(x_t).$$

The score is a vector field. At any noisy point $$x_t$$, it points toward nearby configurations that have higher probability under the perturbed data distribution. During training, denoising score matching provides the target analytically because the conditional corruption kernel is Gaussian:

$$\nabla_{x_t}\log q(x_t\mid x_0)
= -\frac{x_t-\sqrt{\bar\alpha_t}x_0}{1-\bar\alpha_t}
= -\frac{\epsilon}{\sqrt{1-\bar\alpha_t}}.$$

The network is therefore trained across noise levels to minimize a weighted objective of the form

$$\mathcal{L}_{\mathrm{score}}(\theta)=
\mathbb{E}_{t,x_0,\epsilon}\!\left[
\lambda(t)\left\|s_\theta(x_t,t)+\frac{\epsilon}{\sqrt{1-\bar\alpha_t}}\right\|_2^2
\right].$$

<div class="plain-language">
  <div class="plain-language__mark">What the score learns</div>
  <p>The network does not memorize a single route from noise to data. It learns a direction field at every noise level: “from this corrupted point, move this way to become more typical of the data.”</p>
</div>

### Sampling with Langevin dynamics

Once the score is learned, we can sample by following that field while retaining controlled stochasticity. A discretized Langevin update at noise level $$t$$ is

$$x_t^{k+1}=x_t^k+\gamma_t s_\theta(x_t^k,t)+\sqrt{2\gamma_t}\,\xi^k,
\qquad \xi^k\sim\mathcal{N}(0,I).$$

The **score term** moves the sample toward regions of higher learned density. The **Gaussian term** preserves exploration, preventing every trajectory from collapsing onto the same local mode. Sampling is annealed from large to small noise levels: the process explores broadly when $$t$$ is large, then becomes increasingly precise as $$t\to0$$ and the variance schedule shrinks.

In the small-noise limit, the stochastic term becomes negligible and Langevin dynamics approaches gradient ascent on $$\log p_t(x)$$. Equivalently, denoising approximately minimizes the negative log-likelihood $$-\log p_t(x)$$. This optimization view is the bridge to constraint-aware generation.

<p class="paper-link">Score-based generative modeling · <a href="https://proceedings.neurips.cc/paper/2019/hash/3001ef257407d5a371a96dcd947c7d93-Abstract.html">Song & Ermon, NeurIPS 2019 →</a></p>

### Why a realistic sample is not necessarily a valid one

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

A diffusion model gradually transforms noise $$x_T$$ into a structured sample $$x_0$$. The key observation behind our constraint-aware foundation is that each reverse step already behaves like a stochastic optimization step.

Recall that the learned score

$$s_\theta(x_t,t) \approx \nabla_{x_t}\log p_t(x_t)$$

points toward configurations with higher probability under the model's distribution at noise level $$t$$. A Langevin reverse update follows that direction while adding Gaussian noise:

$$x_t \leftarrow x_t + \gamma_t \nabla_{x_t} \log p(x_t) + \sqrt{2\gamma_t}\,\epsilon.$$

Because ascending $$\log p_t(x)$$ is equivalent to descending the negative log-likelihood $$f_t(x)=-\log p_t(x)$$, this is a **noisy descent step on the model's own generative objective**. At each noise level, ordinary reverse diffusion can therefore be viewed as approximately solving

$$\min_x\; -\log p_t(x).$$

The noise is important early in sampling because it lets trajectories explore multiple high-probability regions. As $$t\to0$$ and the variance schedule shrinks, the stochastic term becomes smaller and the update approaches ordinary gradient descent on $$-\log p_t(x)$$. This is what makes the connection to constrained optimization natural: we do not replace the diffusion objective—we restrict where its solution is allowed to live.

If scientifically valid outputs form a feasible set $$C$$, the reverse step becomes

$$\min_{x\in C}\; -\log p_t(x).$$

Projected optimization gives a direct way to implement this constrained step. First, let the score model take its usual noisy denoising update:

$$\widetilde{x}_t = x_t + \gamma_t s_\theta(x_t,t) + \sqrt{2\gamma_t}\,\epsilon.$$

Then map that proposal to its nearest feasible point:

$$x_t^{+}=\mathcal{P}_C(\widetilde{x}_t),
\qquad
\mathcal{P}_C(z)=\arg\min_{y\in C}\lVert y-z\rVert_2^2.$$

This produces an alternating process: the **score step** preserves alignment with the learned data distribution, while the **projection step** enforces the explicit scientific or engineering requirements. Crucially, projection is applied throughout the reverse chain—not once after generation. Each feasible iterate becomes the starting point for the next denoising step, so the model and the constraint operator progressively negotiate a sample that is both likely and valid.

<div class="plain-language">
  <div class="plain-language__mark">In plain language</div>
  <p>The model says, “move this way to look more like the data.” The projection replies, “this is the nearest point that also obeys the rules.” Repeating that exchange lets the pretrained model supply realism while constrained optimization supplies feasibility.</p>
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

<p class="paper-link">Constraint-aware foundation · <a href="https://arxiv.org/abs/2402.03559">Christopher, Baek & Fioretto, NeurIPS 2024 →</a></p>

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

<p class="section-standfirst">The challenge is not merely to generate a plausible protein backbone. It is to preserve the exact three-dimensional relationships that support molecular recognition while allowing the surrounding structure to change.</p>

### Why PDZ domains matter

Many proteins work not as isolated machines, but as parts of larger molecular assemblies. **PDZ domains**—named after the first proteins in which the shared domain was recognized: PSD-95, Discs large, and ZO-1—are small, modular protein-interaction domains that help organize such assemblies. They commonly recognize a short motif at the end of a partner protein, binding it through a groove in which the peptide forms β-sheet-like hydrogen-bond contacts with the PDZ domain.

In cells, PDZ-containing proteins act as molecular organizers. They help localize receptors, ion channels, adhesion proteins, and signaling molecules at the places where they are needed. PSD-95, for example, uses PDZ-mediated interactions to assemble and regulate protein complexes at neuronal synapses. The broader importance is not that every PDZ interaction is individually strong—many are modest-affinity and somewhat promiscuous—but that a modular domain can recruit the right partners into a larger, spatially organized signaling complex.

This modularity also makes PDZ domains useful test cases for protein engineering. One established strategy is to attach a second small domain that contacts portions of the peptide not recognized by the PDZ groove, creating an **affinity clamp** with a larger interaction surface and potentially improved specificity ([Huang et al., 2009](https://doi.org/10.1016/j.jmb.2009.07.067)). Designing such a construct computationally is difficult: the added scaffold must make the intended cross-chain hydrogen bonds while preserving realistic local bond lengths, angles, and secondary structure throughout the new backbone.

### What the design task asks

Our benchmark begins with a resolved PDZ domain bound to its peptide motif. The task is **not to redesign the natural PDZ interaction from scratch**. Instead, the model must generate an additional domain, connected to the PDZ, that reaches the opposite face of the peptide and forms complementary β-sheet-like contacts. This requires satisfying local stereochemistry and global relationships between atoms on different chains at the same time.

The benchmark was built from experimentally resolved PDZ–peptide complexes in the Protein Data Bank. After structural curation and computational filtering, it contains **31 design targets**. Six include short or geometrically restrictive peptide ligands—particularly proline-containing cases—that make the requested backbone arrangement poorly posed. Keeping these difficult cases in the benchmark provides a more realistic measure of where the method succeeds and where the design specification itself leaves little room for a viable structure.

### What we found

Our constrained RFdiffusion method combines proximal feasibility updates with ADMM decomposition. Across 31,000 generations from each of three baselines—93,000 baseline samples in total—none simultaneously satisfied all of the specified inter-chain bond-distance and angle constraints. The constrained method satisfied those encoded constraints in every generated sample.

Constraint satisfaction alone, however, is not the same as a successful protein design. A generated backbone was counted as **usable** only when it also passed structural-realism checks. Under that stricter measure, **21% of generations were usable across the complete 31-target benchmark**. For the subset with well-posed peptide geometries, the usable rate reached **83%**. The three baselines produced no usable samples under the same combined criteria.

**Figure 3** summarizes this comparison and shows representative constrained backbones: the method preserves the requested contacts across the peptide while still producing candidates that pass the benchmark's structural-realism checks.

<div class="evidence-grid">
  <div><span>Encoded constraints</span><strong>100%</strong><small>vs. 0% across 93,000 baseline samples</small></div>
  <div><span>Usable designs</span><strong>21%</strong><small>across the complete 31-target benchmark</small></div>
  <div><span>Well-posed ligands</span><strong>83%</strong><small>usable designs on the favorable subset</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Constraint-aware sampling turns demanding geometric specifications that defeated the evaluated baselines into a meaningful pool of computational protein candidates.</strong>
  <p>This is an important step toward making functional geometry a controllable part of generative protein design. The resulting backbones are promising starting points rather than finished proteins: expression, stability, binding affinity, specificity, and biological function still require experimental validation, and the 21% overall usability rate leaves clear room for improvement.</p>
</div>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/constrained-diffusion/protein_design_slide.png" title="Protein design results with hard constraints" class="research-figure__img" %}
  <figcaption><span>Figure 3</span> In the computational benchmark, constraint-aware RFdiffusion preserves the requested inter-chain geometry and yields structurally usable candidates where the evaluated baselines fail the combined global constraints.</figcaption>
</figure>

On a separate vacancy-constrained pocket-design benchmark, the same framework produced **97.8% usable samples**, compared with **24.2% for the nearest baseline**. That result supports the method's applicability beyond PDZ scaffolding, but it measures a different geometric task and should not be conflated with the 21% overall PDZ result.

<p class="paper-link">Read the paper · <a href="https://arxiv.org/abs/2510.14989">Christopher et al., ICLR 2026 →</a></p>

</section>

<section class="research-section research-domain" id="materials" markdown="1">

<div class="domain-heading"><span>Application 02</span><div class="domain-heading__dot domain-heading__dot--materials"></div></div>

## Materials discovery with properties in the loop

<p class="section-standfirst">In materials design, a sample can look plausible while having the wrong porosity, mechanical response, or physical geometry. The constraint operator turns those measurable properties—and, when needed, an external simulator—into feedback during generation.</p>

### Why material structure matters

A material's behavior depends not only on what it is made of, but also on how its internal structure is organized. Pore size and connectivity influence transport through a porous medium; in cellular metamaterials, the arrangement and thickness of the struts determine how the structure bends, buckles, and carries load. Small geometric changes can therefore produce large changes in the resulting physical response.

Generative models could help explore these large design spaces and augment datasets that are costly to obtain through high-resolution imaging, fabrication, or simulation. But visual realism is not enough. A generated microstructure may resemble the training data while missing the requested porosity, and a convincing cellular pattern may fail to reproduce a target stress–strain curve—or may not define a physically valid structure at all.

### What the design task asks

We study two complementary tasks. In **porous-microstructure generation**, the model must create an image at a prescribed porosity, including values that are sparsely represented in the data. Because porosity can be computed directly from the decoded image, the method can project each intermediate sample toward the required value. In **mechanical-metamaterial inverse design**, the input is instead a target nonlinear stress–strain response under compression. Here, the relevant property is evaluated by an external finite-element simulator, so the method uses simulator queries to estimate a useful correction and then applies it in the latent space of the diffusion model.

The common pattern is **generate, evaluate, and correct** without retraining the underlying diffusion model. **Figure 4a** illustrates this broader constraint-in-the-loop architecture in an atomistic setting, where a surrogate checks physical interactions during denoising. **Figure 4b** shows the mechanical-metamaterial experiment: as the latent sample is corrected, its simulated stress–strain curve moves toward the target and the geometry becomes physically valid.

### What we found

For porous microstructures, conditional diffusion produced visually strong samples but missed the requested porosity by more than 10% in **68.4%** of cases. The constrained method reduced that violation rate to **0%**. Its Fréchet Inception Distance was **13.5**, compared with **10.8** for conditional diffusion, indicating a modest image-quality tradeoff in exchange for much more reliable property control.

For mechanical metamaterials, the constrained method reached a stress–strain mean-squared error of **1.4 ± 0.6**, compared with **6.4 ± 4.6** for a specialized inverse-design baseline and **7.1 ± 4.5** for conditional diffusion. Only **5%** of its outputs were judged physically invalid, versus **20%** and **55%** for those two baselines, respectively. These results show that simulator feedback can improve both response matching and geometric validity on this computational benchmark.

<div class="research-figure-pair">
  <figure class="research-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/constrained_ai4mat.png" title="Surrogate-guided crystal denoising" class="research-figure__img" %}
    <figcaption><span>Figure 4a</span> The broader constraint-in-the-loop pattern for atomistic generation: a surrogate evaluates interactions and returns a correction during denoising.</figcaption>
  </figure>
  <figure class="research-figure">
    {% include figure.html path="assets/img/projects/constrained-diffusion/metamaterials_results_slide.png" title="Mechanical metamaterial inverse design" class="research-figure__img" %}
    <figcaption><span>Figure 4b</span> Simulator-guided samples move toward the target stress–strain response; the final benchmark reports 5% physically invalid geometries.</figcaption>
  </figure>
</div>

<div class="evidence-grid">
  <div><span>Porosity error &gt;10%</span><strong>0%</strong><small>vs. 68.4% for conditional diffusion</small></div>
  <div><span>Stress–strain MSE</span><strong>1.4 ± 0.6</strong><small>4.6× lower than a specialized baseline</small></div>
  <div><span>Invalid geometries</span><strong>5%</strong><small>vs. 20% specialized and 55% conditional</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Constraint-in-the-loop generation makes target material properties substantially more reliable while retaining competitive sample quality.</strong>
  <p>The results demonstrate a practical way to use image measurements and finite-element feedback during generation, opening a larger and more controllable computational design space. The next step is experimental validation: fabrication constraints, defects, robustness to loading conditions, measured stress–strain behavior, and other application-specific properties remain outside these benchmarks. Simulator-guided sampling also introduces a meaningful computational cost.</p>
</div>

<p class="paper-link">Read the paper · <a href="https://arxiv.org/abs/2502.05625">Zampini et al., NeurIPS 2025 Spotlight →</a></p>

</section>

<section class="research-section research-domain" id="robotics" markdown="1">

<div class="domain-heading"><span>Application 03</span><div class="domain-heading__dot domain-heading__dot--robotics"></div></div>

## Safe motion planning at generative speed

<p class="section-standfirst">A multi-robot plan is feasible only if every robot reaches its goal without striking an obstacle or another robot—and does so along a trajectory that its motion limits can support.</p>

### Why coordinated motion matters

Multi-robot motion planning appears wherever autonomous systems share space: warehouse fleets moving inventory, coordinated drones, autonomous vehicles, and robots operating in search-and-rescue settings. Planning each path independently is not enough. A route that is safe for one robot can block another, create a collision several seconds later, or produce a deadlock in a narrow corridor. The difficulty grows quickly because each additional robot adds a trajectory whose position must remain compatible with every other trajectory over time.

Classical grid-based multi-agent path finding scales well, but its coarse spatial and temporal discretization can produce paths that are difficult to execute smoothly. Continuous trajectory optimization offers finer motion, but the joint problem becomes high-dimensional and strongly non-convex as robots and obstacles are added. A generative model provides a learned prior over plausible paths; the central question is whether that prior can be converted into a coordinated, feasible plan.

### What the planning task asks

Each robot is assigned a start and goal position in a shared two-dimensional workspace. The planner must produce all trajectories simultaneously while satisfying four coupled requirements: avoid static obstacles, maintain separation between robots at every time step, respect velocity and acceleration limits, and preserve the assigned start and goal states. Practical benchmark layouts include warehouse-like shelves, connected rooms, dense obstacle fields, and corridors where robots must negotiate passage rather than simply follow locally shortest routes.

**Simultaneous MRMP Diffusion (SMD)** addresses the joint continuous problem directly. During reverse diffusion, it repeatedly corrects the full set of trajectories using an augmented-Lagrangian projection, so obstacle, kinematic, and inter-robot constraints influence generation before the paths are finalized. **Discrete-Guided Diffusion (DGD)** tackles the scaling bottleneck differently: a discrete multi-agent path-finding solution supplies a coordination schedule, the continuous workspace is decomposed into tractable regions, and constrained diffusion refines the motion within those regions. A lightweight repair step handles residual violations.

The animations in **Figure 5** make the constraint visible over time. They show that feasibility depends on the entire joint trajectory—particularly in corridors, shelves, and rooms—not merely on collision-free final positions.

### What we found

On the SMD benchmark's most difficult random setting—nine robots moving among 20 obstacles in a dense map—SMD solved **96%** of instances, compared with **27.2%** for the strongest prior multi-robot diffusion baseline. In the corridor task, SMD solved every instance while the evaluated baselines found no feasible trajectory. These results measure whether the generated plan is collision-free and satisfies the encoded motion constraints within the benchmark.

DGD extends the approach to larger problems. On the standard benchmark it achieved at least **92% success** in all but the two hardest 18-robot settings, where it still led at **76%**. On dense six-robot maps, DGD matched SMD's 100% success while using **4% of SMD's runtime**. The scalability study generated feasible plans for **100 robots among 104 obstacles**, compared with the 40-robot, obstacle-free scale reported for prior diffusion-based planning.

<div class="motion-reel" aria-label="Five looping multi-robot planning examples">
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_baisc.mp4" type="video/mp4"></video><span>Basic</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_dense.mp4" type="video/mp4"></video><span>Dense</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_corridor.mp4" type="video/mp4"></video><span>Corridor</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_shelf.mp4" type="video/mp4"></video><span>Shelf</span></div>
  <div><video autoplay muted loop playsinline preload="metadata"><source src="/assets/video/projects/constrained-diffusion/smd_room.mp4" type="video/mp4"></video><span>Room</span></div>
</div>
<p class="research-caption"><span>Figure 5</span> Simulated multi-robot trajectories in five benchmark environments. The animation exposes coordination events and possible conflicts that a final frame would hide.</p>

<div class="evidence-grid">
  <div><span>Dense-map success</span><strong>96%</strong><small>vs. 27.2% for the strongest prior baseline</small></div>
  <div><span>Maximum scale</span><strong>100 robots</strong><small>with 104 obstacles</small></div>
  <div><span>DGD runtime</span><strong>4%</strong><small>of SMD's time on dense six-robot maps</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Constraint-aware diffusion can coordinate many robots in cluttered environments where prior generative planners frequently fail.</strong>
  <p>The gains in feasibility and scale make diffusion-based planning a credible option for difficult coordination problems, especially when multiple valid trajectories may exist. Moving from simulation to physical deployment will require models of perception error, moving obstacles, tracking and control error, communication limits, and hardware dynamics. SMD's inference cost and consistency across DGD's decomposed regions also remain useful targets for improvement.</p>
</div>

<p class="paper-link">Read the papers · <a href="https://arxiv.org/abs/2502.03607">SMD, ICML 2025</a> · <a href="https://arxiv.org/abs/2508.20095">DGD, AAAI 2026 →</a></p>

</section>

<section class="research-section research-domain" id="chemistry" markdown="1">

<div class="domain-heading"><span>Application 04</span><div class="domain-heading__dot domain-heading__dot--chemistry"></div></div>

## Constraints in discrete spaces: molecules, text, and code

<p class="section-standfirst">In discrete diffusion, “move to the nearest feasible point” cannot mean nudging coordinates. It means changing probability mass over tokens without destroying the model's learned preferences.</p>

### Why discrete constraints matter

Text, molecular strings, and source code are built from discrete symbols. A single misplaced token can reverse the meaning of a sentence, break a SMILES representation, or introduce a software vulnerability. Many important requirements are also global: a word may need to appear at a particular position, a molecular sequence must decode to a valid and previously unseen structure, and a program must behave correctly across an entire test suite. These constraints cannot always be enforced by choosing the most likely next token in isolation.

Masked diffusion offers a useful intervention point because it repeatedly predicts the **whole sequence** rather than committing permanently from left to right. Intermediate tokens can be reconsidered as the global structure becomes clearer. This makes discrete diffusion particularly well suited to constraints that depend on relationships between distant parts of a sequence.

### What the generation task asks

**[Constrained Discrete Diffusion (CDD)](https://arxiv.org/abs/2503.09790)** operates on the probability distribution over tokens. At each reverse step, it finds a nearby distribution whose most likely decoded sequence satisfies the constraint, using KL divergence to remain close to the base model's proposal. The same principle supports several types of evaluator: a toxicity classifier for controlled text, exact counting or positional rules for instruction following, and property or novelty checks for molecular SMILES.

The molecule sequence in **Figure 6** provides the intuition: constraint checks accompany generation instead of being applied only after a final candidate appears. In the actual sequence benchmarks, the projection acts on token probabilities and can encode properties such as synthetic accessibility, novelty, or the absence of selected BRENK structural alerts—not only local valency.

Code requires a richer version of this loop. **[Constrained Diffusion for Code (CDC)](https://arxiv.org/abs/2605.16829)** decodes an intermediate program, evaluates functional or security requirements, identifies the token regions most responsible for a violation, and locally revises or remasks those regions. Differentiable surrogate feedback guides functional correctness, while static program analysis supplies concrete vulnerability information. This uses diffusion's editable global state to repair a developing program before the final sample is committed.

### What we found

Across the CDD text experiments, the constrained sampler produced **0% violations** at all evaluated toxicity thresholds and on both counting and positional lexical constraints. The corresponding unconstrained MDLM violated 54.5% of counting requirements and 97.5% of lexical requirements. This exact compliance came with task-dependent quality and computational tradeoffs; for example, constrained lexical generation improved the paper's coherence score but had somewhat higher perplexity than unconstrained MDLM.

In the **[molecular-generation extension](https://ai4d3.github.io/2025/papers/18_Constrained_Molecular_Gener.pdf)**, CDD generated an average of **392 novel candidates that passed the encoded BRENK screens**, compared with 108 for MDLM and 5 for an autoregressive baseline, while recording no BRENK-rule violations. These are useful filters for exploring chemical space, but they are not a complete assessment of toxicity, synthesizability, stability, or biological activity.

For code, CDC increased functional correctness with Dream-Coder 7B from **34.1% to 65.2%** on HumanEval-X C++ and from **27.7% to 59.2%** on MBPP-C++. It also raised syntax correctness from 67.1% to 79.2% and from 41.1% to 72.0% on those benchmarks. On CWEval, joint functional-and-security success increased from **12.04% to 34.26%**. **Figure 7** summarizes these improvements across functionality, syntax, and security tests.

<figure class="research-figure">
  {% include figure.html path="assets/img/projects/constrained-diffusion/Fig6.png" title="Constrained discrete diffusion for molecular generation" class="research-figure__img" %}
  <figcaption><span>Figure 6</span> Conceptual view of checking molecular validity throughout generation. CDD implements this principle over discrete sequence probabilities and supports both local and sequence-level rules.</figcaption>
</figure>

<figure class="research-figure research-figure--bleed">
  {% include figure.html path="assets/img/projects/discrete-diffusion/code_gen_results.png" title="Constrained code generation results" class="research-figure__img" %}
  <figcaption><span>Figure 7</span> CDC improves benchmark pass rates for program functionality, syntax, and security across two seven-billion-parameter diffusion code models.</figcaption>
</figure>

<div class="evidence-grid">
  <div><span>Rule violations</span><strong>0%</strong><small>for toxicity, counting, and lexical tests</small></div>
  <div><span>BRENK-screened candidates</span><strong>392</strong><small>novel molecules vs. 108 for MDLM</small></div>
  <div><span>Code functionality</span><strong>65.2%</strong><small>vs. 34.1% for the Dream-Coder base model</small></div>
</div>

<div class="research-callout">
  <span class="research-callout__eyebrow">What these results show</span>
  <strong>Intervening during denoising can materially improve rule adherence across language, molecular design, and code without retraining the base generator.</strong>
  <p>This makes discrete diffusion a promising platform for applications where outputs must be both plausible and verifiably structured. The guarantees remain tied to the supplied evaluator: toxicity classifiers, molecular screens, tests, and static analyzers each cover only part of the real requirement. Constraint optimization also adds computation, so human review and domain-specific validation remain important for consequential uses.</p>
</div>

<p class="paper-link">Explore the full research-area guide · <a href="{{ '/projects/discrete-diffusion/' | relative_url }}">Constrained Discrete Diffusion for Language, Chemistry, and Code →</a></p>

</section>

<section class="research-section" id="current-work" markdown="1">

<div class="section-label">04 · Current and ongoing work</div>

## From correcting a pretrained model to training for constraints

Training-free guidance is flexible, but it creates a mismatch: the model learns unconstrained trajectories during training and encounters projected trajectories only at inference. Two newer directions attack that mismatch from opposite sides.

<div class="current-work-grid">
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
  <figcaption><span>Figure 8</span> CAFM aligns the learned velocity field with the constrained sampling path instead of correcting an unconstrained field only at inference.</figcaption>
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
  <a href="https://arxiv.org/abs/2402.03559"><span>NeurIPS 2024</span><strong>Constrained Synthesis with Projected Diffusion Models</strong><small>Christopher, Baek & Fioretto</small></a>
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
