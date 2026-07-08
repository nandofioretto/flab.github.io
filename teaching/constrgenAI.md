---
layout: default
title: Constrained-Aware Generative AI
permalink: /teaching/constrgenAI/
---

# **Constrained-Aware Generative AI**

#### Computer Science department, University of Virginia

- **Course number**: CS 6501-005
- **Term**: Spring, 2026
- **Meeting time and location:** TuTh 9:30AM - 10:45AM, Rice Hall 340
- **Instructor:** **Ferdinando Fioretto**, [Email](fioretto@virginia.edu), Office: Rice Hall 307, Office hours: Fridays 3:30PM
- **Teaching staff:** Michael Cardei, [Email](ntr2rm@virginia.edu), Office hours:

<!-- **Course website and LMS:** [URL] -->

---

## Course description

Generative AI systems are increasingly deployed in settings where correctness is defined by explicit constraints and verifiable properties: physical feasibility in robotics, structural validity in proteins and materials, syntactic and semantic correctness in code, and safety and policy compliance in language.
This course develops the algorithmic and mathematical foundations of constrained-aware generative AI. The central question is how to turn powerful probabilistic generators into reliable components for scientific and engineering workflows, where outputs must satisfy hard or verifiable requirements such as physical laws, discrete structure rules, safety specifications, and system-level constraints (for example, feasibility of a robot trajectory, validity of a molecular graph, or compliance with a policy).

Students will learn the algorithmic foundations of likelihood-based modeling, latent-variable methods, autoregressive Transformers, diffusion models, and flow matching, and then connect these models to optimization and control mechanisms such as projection and proximal steps, constrained decoding, differentiable optimization layers, and reinforcement learning for alignment. A short optimization bootcamp is included to support students who have had a first machine learning course but limited exposure to convex optimization.


## Prerequisites

Students are expected to have completed a first graduate-level machine learning course (or equivalent). Familiarity with linear algebra, probability, and gradient-based optimization in ML is assumed. Some degree of knowledge on convex optimization is desirable. We will introduce optimization first as an operational tool (projection, penalties, prox), then formalized (duality, KKT, splitting), and finally used for differentiable layers and alignment.

## Learning objectives

By the end of the course, students should be able to:

1. Formalize constraint-aware generation as approximate inference, sampling, or optimization under hard and soft constraints.
2. Derive and interpret core objectives for generative modeling, including maximum likelihood, ELBO-based objectives, and diffusion and flow-based training objectives.
3. Explain and implement constraint enforcement mechanisms.
4. Use core optimization concepts to analyze modern constrained generation algorithms.
5. Critically evaluate constraint-aware generative systems using appropriate metrics for validity, constraint satisfaction, robustness, and generalization of constraints.

## Course structure

The course is organized as follows:

- The first 4 weeks will be instructor-led _lecture_s to establish a shared technical language
- Weeks 5 to 7 will feature short invited-lecture focused on scientific and engineering case studies
- Weeks 8 to 14 are run as a “research-based lab”: each lecture begins with a short instructor “theory injection” followed by student paper presentations and structured discussion.

A recurring course template is the constrained target distribution:

$$
\text{sample} x \sim p_\theta(x | c) \quad \text{subject to} \quad x \in \mathcal{C}\ \text{(hard)}
  \text{or} \, g(x)\le 0\ \text{(soft)},
$$

where constraints may be symbolic (grammars, automata, SAT/SMT), geometric (equivariance, manifolds, SE(3)), physical (PDEs, energy minimization, stability), or preference-based (toxicity, helpfulness). Each paper and method in the course will be analyzed by (i) how it represents constraints, (ii) where constraints enter (training, architecture, inference, post-processing), and (iii) how it performs constrained inference.

## **Course Schedule**


### Part 1: Instructor-led bootcamp

Symbols [R] denotes ``required'' reading.

##### **L1: Course overview and taxonomy of constraints**

**Tuesday, January 13, 2026**

**[Lecture notes](../constrGenAI26_files/lecture1_notes.pdf)**

We will review why constraints matter in generative AI: validity, safety, and controllability.
Then define constrained-aware generation by specifying a target distribution that blends a base model with constraint terms, for example

$$
\pi(x \mid c)\ \propto\ p_\theta(x \mid c)\,\exp(-\lambda \phi(x,c))\,\mathbf{1}\{x \in \mathcal{C}(c)\}.
$$

Every method in the course chooses where to implement $$\phi$$ or $$\mathcal{C}$$ (training, architecture, inference, post-processing), and how to approximate sampling or optimization under $$\pi$$.

**Suggested readings**:
- Christopher, Baek, Fioretto, [Constrained Synthesis with Projected Diffusion Models](https://arxiv.org/abs/2402.03559) (NeurIPS 2024) [R].
- Mandi et al., [Decision-Focused Learning: Foundations, State of the Art, Benchmark and Future Opportunities](https://arxiv.org/abs/2307.13565) (JAIR 2024).
- R: LeCun, Chopra, Hadsell (2006). [A Tutorial on Energy-Based Learning](http://yann.lecun.com/exdb/publis/pdf/lecun-06.pdf). (Chapter in Predicting Structured Data).
- Hyvarinen (2005). [Estimation of Non-Normalized Statistical Models by Score Matching](https://jmlr.org/papers/volume6/hyvarinen05a/hyvarinen05a.pdf). JMLR.
- O: Blei, Kucukelbir, McAuliffe (2017). [Variational Inference: A Review for Statisticians](https://arxiv.org/pdf/1601.00670). JASA.
- Bubeck (2015). [Convex Optimization: Algorithms and Complexity](https://arxiv.org/pdf/1405.4980). Foundations and Trends in Machine Learning.

##### **L2: Likelihood and latent-variable modeling for control**

**Thursday, January 15, 2026**

**[Lecture notes](../constrGenAI26_files/lecture2_notes.pdf)**

We will review maximum likelihood, conditional modeling, ELBO for latent-variable models. The goal is to approach approximate inference because later we will need to ``add constraints'' within this framework.

**Suggested readings**:

- Rezende, Mohamed, Wierstra (2014). [Stochastic Backpropagation and Approximate Inference in Deep Generative Models](https://arxiv.org/abs/1401.4082). ICML [R].
- O: Burda, Grosse, Salakhutdinov (2015). [Importance Weighted Autoencoders](https://arxiv.org/abs/1509.00519). ICLR.
- Ranganath, Gerrish, Blei (2014). [Black Box Variational Inference](https://arxiv.org/abs/1401.0118). AISTATS.


##### **L3: VAEs and GANs**

**Tuesday, January 20, 2026**

**[Lecture notes](../constrGenAI26_files/lecture3_notes.pdf)**


We will cover conditional VAEs, structured priors, and posterior regularization as early ''weak control'' strategies. Then GANs as implicit control, and why feasibility constraints are awkward without explicit likelihood. This will give us the necessary context for why iterative refinement methods are so attractive for constraints.

**Suggested readings**:
- R: Kingma, Welling (2013). [Auto-Encoding Variational Bayes](https://arxiv.org/abs/1312.6114). [R]
- R: Goodfellow et al. (2014). [Generative Adversarial Nets](https://arxiv.org/abs/1406.2661). NeurIPS. [R]
- Arjovsky, Chintala, Bottou (2017). [Wasserstein GAN](https://arxiv.org/abs/1701.07875). ICML.
- O: Gulrajani et al. (2017). [Improved Training of Wasserstein GANs](https://arxiv.org/abs/1704.00028). NeurIPS.
- Nowozin, Cseke, Tomioka (2016). [f-GAN: Training Generative Neural Samplers using Variational Divergence Minimization](https://arxiv.org/abs/1606.00709). NeurIPS.


##### **L4: Autoregressive Transformers and decoding**

**Thursday, January 22, 2026**

**[Lecture notes](../constrGenAI26_files/lecture4_notes.pdf)**

We will review transformers, factorization, and basic sampling. Then decoding methods (beam search, top-p, reranking). We will also cover constrained decoding via grammars or finite-state constraints as the first concrete example of ''generation as search subject to constraints''.

**Suggested readings**:
- R: Vaswani et al. (2017). [Attention Is All You Need](https://arxiv.org/abs/1706.03762). NeurIPS. [R]
- Brown et al. (2020). [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165). NeurIPS [R].
- R: Holtzman et al. (2019). [The Curious Case of Neural Text Degeneration](https://arxiv.org/abs/1904.09751). ICLR [R].
- O: Hokamp, Liu (2017). [Lexically Constrained Decoding for Sequence Generation using Grid Beam Search](https://arxiv.org/abs/1704.07138). ACL [R].
- Post, Vilar (2018). [Fast Lexically Constrained Decoding with Dynamic Beam Allocation](https://arxiv.org/abs/1804.06609). NAACL.
- Li et al. (2022). [Contrastive Decoding: Open-ended Text Generation as Optimization](https://arxiv.org/abs/2210.15097). ACL.
- O: Radford et al. (2019). [Language Models are Unsupervised Multitask Learners](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) (GPT-2 technical report).
- Devlin et al. (2019). [BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/abs/1810.04805). NAACL.

<!-- Pretraining objectives, scaling laws, and data/compute tradeoffs -->
- R: Kaplan et al. (2020). [Scaling Laws for Neural Language Models](https://arxiv.org/abs/2001.08361).
- Hoffmann et al. (2022). [Training Compute-Optimal Large Language Models (Chinchilla)](https://arxiv.org/abs/2203.15556).
- O: Touvron et al. (2023). [LLaMA: Open and Efficient Foundation Language Models](https://arxiv.org/abs/2302.13971).
- Wei et al. (2022). [Emergent Abilities of Large Language Models](https://arxiv.org/abs/2206.07682).


##### **L5: Architectures for control**

**January 27, 2026**

**[Lecture notes](../constrGenAI26_files/lecture5_notes.pdf)**

This lecture will review the recent progress on geometric deep learning, equivariance, and inductive biases.

**Suggested readings**:
- R: Bronstein et al. (2021). [Geometric Deep Learning: Grids, Groups, Graphs, Geodesics, and Gauges](https://arxiv.org/abs/2104.13478).
- Satorras, Hoogeboom, Welling (2021). [E(n) Equivariant Graph Neural Networks](https://arxiv.org/abs/2102.09844). ICML.
- O: Fuchs et al. (2020). [SE(3)-Transformers: 3D Roto-Translation Equivariant Attention Networks](https://arxiv.org/abs/2006.10503). NeurIPS.
- Thomas et al. (2018). [Tensor Field Networks: Rotation- and Translation-Equivariant Neural Networks for 3D Point Clouds](https://arxiv.org/abs/1802.08219).

##### **L6: Optimization essentials**

**Thursday, January 29, 2026**

**[Lecture notes](../constrGenAI26_files/lecture6_notes.pdf)**


We will cover constraint sets, projections, why projections solve a least-squares problem, and what a penalty method is. We will introduce proximal operators for nonsmooth penalties and review duality.

- R: Boyd, Vandenberghe (2004). [Convex Optimization](https://stanford.edu/~boyd/cvxbook/bv_cvxbook.pdf). Cambridge University Press. (Chapters 2-5).
- R: Parikh, Boyd (2014). [Proximal Algorithms](https://web.stanford.edu/~boyd/papers/pdf/prox_algs.pdf). Foundations and Trends in Optimization.
- Boyd et al. (2011). [Distributed Optimization and Statistical Learning via the Alternating Direction Method of Multipliers](https://web.stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf). Foundations and Trends in Machine Learning.
- O: Beck, Teboulle (2009). [A Fast Iterative Shrinkage-Thresholding Algorithm for Linear Inverse Problems](https://www.ceremade.dauphine.fr/~carlier/FISTA) (FISTA). SIAM J. Imaging Sciences.
- Combettes, Pesquet (2011). [Proximal Splitting Methods in Signal Processing](https://arxiv.org/abs/0912.3522).
- Mandi et al., [Decision-Focused Learning: Foundations, State of the Art, Benchmark and Future Opportunities](https://arxiv.org/abs/2307.13565) (JAIR 2024).


##### **L7: Diffusion models and Guidance**

**Tuesday, February 3, 2026**

**[Lecture notes](../constrGenAI26_files/lecture7_notes.pdf)**


We will cover DDPM objective and the conceptual score view. The emphasis will be algorithmic: the reverse process is a sequence of updates, so it has natural insertion points for constraint forces, projection, or repair.

- R: Ho, Jain, Abbeel (2020). [Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239). NeurIPS. [R]
- Song et al. (2021). [Score-Based Generative Modeling through Stochastic Differential Equations](https://arxiv.org/abs/2011.13456). ICLR. [R]
- O: Nichol, Dhariwal (2021). [Improved Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2102.09672). ICML.
- Sohl-Dickstein et al. (2015). [Deep Unsupervised Learning using Nonequilibrium Thermodynamics](https://arxiv.org/abs/1503.03585). ICML.
<!-- Conditional diffusion and guidance: classifier guidance, classifier-free guidance, and constraint energy shaping -->
- Ho, Salimans (2022). [Classifier-Free Diffusion Guidance](https://arxiv.org/abs/2207.12598).
- O: Song et al. (2020). [Denoising Diffusion Implicit Models (DDIM)](https://arxiv.org/abs/2010.02502). ICLR.
- Kawar et al. (2022). [Denoising Diffusion Restoration Models (DDRM)](https://arxiv.org/abs/2201.11793). NeurIPS.


##### **L8: Flow matching and rectified flows**

**Thursday, February 5, 2026**

**[Lecture notes](../constrGenAI26_files/lecture8_notes.pdf)**


Flow matching as learning a vector field; rectified flows as simplified transport. The point is that ODE form makes constraint injection feel like adding a control term, and it motivates later splitting methods.

- R: Lipman et al. (2022). [Flow Matching for Generative Modeling](https://arxiv.org/abs/2210.02747). [R]
- Liu, Gong, Liu (2022). [Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow](https://arxiv.org/abs/2209.03003).
- O: Miller et al. (2024). [FlowMM: Generating Materials with Riemannian Flow Matching](https://arxiv.org/abs/2406.04713). ICML.
- Chen et al. (2018). [Neural Ordinary Differential Equations](https://arxiv.org/abs/1806.07366). NeurIPS.


##### **L9: Discrete diffusion models.**

**Tuesday, February 10, 2026**

**[Lecture notes](../constrGenAI26_files/lecture9_notes.pdf)**

Masked or denoising diffusion for tokens, joint distribution control versus AR conditionals, and how constraints can act as (i) constrained unmasking policies, (ii) constrained decoding on intermediate representations, or (iii) projection onto probability simplices with constraints. This sets up your lab phase on discrete constraints without needing duality yet.

- R: Austin et al. (2021). [Structured Denoising Diffusion Models in Discrete State-Spaces (D3PM)](https://arxiv.org/abs/2107.03006). NeurIPS. [R]
- Lou, Meng, Ermon (2023). [Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution (SEDD)](https://arxiv.org/abs/2310.16834). [R]
- O: Hoogeboom et al. (2021). [Argmax Flows and Multinomial Diffusion: Learning Categorical Distributions](https://arxiv.org/abs/2102.05379). NeurIPS.
- Simple and Effective Masked Diffusion Language Models (MDLM). [arXiv:2406.07524](https://arxiv.org/abs/2406.07524). [R]
<!-- Constraints -->
- R: Cardei et al. (2025). Constrained Language Generation with Discrete Diffusion Models. arXiv:2503.09790. [R]
- R: Cardei et al. (2025). [Constrained Language Generation with Discrete Diffusion Models](https://arxiv.org/abs/2503.09790). [R]
- Post, Vilar (2018). [Fast Lexically Constrained Decoding with Dynamic Beam Allocation](https://arxiv.org/abs/1804.06609). NAACL.
- O: Schiff et al. (2025). [Simple Guidance Mechanisms for Discrete Diffusion Models](https://arxiv.org/abs/2412.10193). ICLR.
- Arriola et al. (2025). [Block Diffusion: Interpolating Between Autoregressive and Diffusion Language Models](https://arxiv.org/abs/2503.09573).


### **Part 2: Invited lectures**

##### **L11: Generative AI for Protein Design**

**Thursday, February 12, 2026**

- **Invited Speaker**: [Sagar Khare](https://chem.rutgers.edu/people/faculty/faculty-details/164-khare-sagar-d) - Rutgers University
- **Talk title**: TBA
- **Talk abstract**: TBA

##### **L12: Generative AI for Weather Prediction**

**Tuesday, February 17, 2026**

- **Invited Speaker**: James E. Warner - NASA
- **Talk title**: TBA
- **Talk abstract**: TBA

##### **L13: Generative AI for Material Science**

**Thursday, February 19, 2026**

- **Invited Speaker**: Amir Ziabari - ORNL
- **Talk title**: TBA
- **Talk abstract**: TBA

##### **L14: Preliminary Projects and Teams.**

**Tuesday, February 24, 2026**

We will use this time to discuss and refine project ideas proposed by various teams.

Teams are required to:

- **Form a team (2–4 students)** and choose a tentative project direction.
- **Prepare a short in-class pitch** (5 minutes + 2 minutes Q&A). Bring 2–4 slides.
- **Submit a one-page project intent** (PDF) within 48 hours after class. It should include:
  - the **task/domain** (e.g., language, molecules, robotics, materials, code),
  - the **constraint(s)** (hard and/or soft) and how they are represented,
  - the **verification or measurement** procedure for constraint satisfaction,
  - a **baseline generator** you will reproduce (with a citation),
  - the **constraint-aware mechanism** you plan to implement (projection/prox, constrained decoding, reranking/verifier, differentiable layer, RL/alignment, etc.),
  - **data and compute needs** (dataset or simulator; expected runtime),
  - an **evaluation plan** (validity/feasibility rate, violation magnitude, quality/diversity, and at least one ablation),
  - key risks and a **fallback plan**.
<!-- - **Schedule one 15-minute check-in** with the instructor/TA during the following week to lock scope and feasibility. -->


##### **L15: Preliminary Projects and Teams.**

**Tuesday, February 26, 2026**

We will continue the discussion, brainstorm and provide feedback to project ideas.


##### **L16: Generative AI for Robotics**

**Tuesday, March 10, 2026**

- **Invited Speaker**: Nicola Bezzo - University of Virginia
- **Talk title**: TBA
- **Talk abstract**: TBA


### Part 3: Research-based lab (Proposed -- will be subject to change)

Each lab lecture begins with a short micro-lecture followed by two student paper presentations and discussion.


| **Class** | **Date** | **Notes** |
| ---------- | -------- | --------- |
| — | *Spring Recess* | February 28 - March 8 |
| 17 | Thursday, March 12, 2026 | Paper & Group TBD |
| 18 | Tuesday, March 17, 2026 | Paper & Group TBD |
| 19 | Thursday, March 19, 2026 | Paper & Group TBD |
| 20 | Tuesday, March 24, 2026 | Paper & Group TBD |
| 21 | Thursday, March 26, 2026 | Paper & Group TBD |
| 22 | Tuesday, March 31, 2026 | Paper & Group TBD |
| 23 | Thursday, April 2, 2026 | Paper & Group TBD |
| 24 | Tuesday, April 7, 2026 | Paper & Group TBD |
| 25 | Thursday, April 9, 2026 |  Paper & Group TBD |
| 26 | Tuesday, April 14, 2026 |  Paper & Group TBD1 |
| 27 | Thursday, April 16, 2026 |  Paper & Group TBD |
| 28 | Tuesday, April 21, 2026 | Paper & Group TBD |
| 29 | Thursday, April 23, 2026 | Paper & Group TBD |
| 30 | Tuesday, April 28, 2026 | Final project presentations (**class starts at 8:30AM!**)|

<!--
**W7 - L13. Micro-lecture: optimization essentials for constrained generation.**
Constraint sets, projections, penalties, and proximal operators as reusable primitives.

**W7 - L14. Micro-lecture: Lagrangians and duality for constraint enforcement.**
Dual variables as constraint pressures; interpreting soft penalties.

**W8 - L15. Micro-lecture: splitting methods.** Proximal gradient, Douglas–Rachford intuition, ADMM as “prior step plus constraint step.”

**W8 - L16. Micro-lecture: constrained sampling and energy shaping.** Guidance as adding energy terms; proxy constraints and pitfalls.

**W10 - L19. Micro-lecture: differentiable optimization layers.** Implicit differentiation through KKT systems; sensitivity analysis. Core: OptNet; Differentiable Convex Optimization Layers (CVXPYLayers).

**W10 - L15 Geometry and equivariance for constraints. SO(3)/SE(3) intuition, invariances, why this is “architecture-level constraint enforcement.”

<!-- **W10 - L20. Micro-lecture: practical differentiation through solvers.** Approximate solves, nonsmoothness, stability, and implementation patterns. -->
<!--
**W11 - L20. Micro-lecture: constrained decoding beyond the basics.** Grammar constraints, lexical constraints, reranking, verifier-in-the-loop generation. Grid Beam Search.

**W11 - L21. Micro-lecture: search and verification inside generation.** Classical search, repair loops, and executable constraints, compilers, theorem provers, simulators.

**W12 - L22. Micro-lecture: RL for control and alignment.** KL-regularized RL and reward modeling as soft constraints. InstructGPT?

**W12 - L23. Micro-lecture: preference optimization and constraint tradeoffs.** Direct preference optimization as a practical alignment mechanism. DPO.

**W13 - L24. Micro-lecture: zeroth-order and simulator-only constraints.** Bandit feedback, evolutionary strategies, sample efficiency, and robustness. -->
<!--
**W13 - L26. Micro-lecture: evaluation and stress testing for constraint satisfaction.** Validity rates, constraint violation magnitudes, diversity under constraints, proxy hacking, distribution shift. -->

<!-- **W14 - L25. Final project presentations I.**

**W14 - L26. Final project presentations II and course synthesis.** -->

---

## **Assessment and grading**



##### Paper Presentation – 40.0%

**Objective:** To enhance students' ability to communicate complex AI concepts and engage in public speaking.

**Expectations:**
- 45-minute presentation per group.
- Presentations can include slides, code demonstrations, videos, or other creative methods.
- The presentation should cover the key aspects of the paper, including its contribution to responsible AI.
- A critical evaluation of the paper is essential, including discussing its limitations and implications.
- Preparation of thought-provoking questions to stimulate audience engagement.

**Assessment Criteria:**
- Effectiveness of communication and presentation skills.
- Accuracy and depth of content presented.
- Creativity and engagement in the presentation method.
- Ability to provoke thoughtful discussion through prepared questions.

##### Final Project - 60%

(proposal + milestones + report + presentation)

The final project is the main deliverable and should include a reproducible baseline and a constraint-aware extension.

**Objective:** To design, implement, and evaluate a constraint-aware generative system that is technically sound, empirically validated, and reproducible.

**Expectations:**
- Define a clear **task/domain** and one or more explicit **constraints** (hard and/or soft), including how constraints are represented and verified.
- Reproduce a **baseline generator** (e.g., autoregressive, diffusion, flow matching) with a documented training/inference pipeline.
- Implement a **constraint-aware method** (e.g., projection/prox steps, constrained decoding, reranking/verifier-in-the-loop, differentiable optimization layer, or alignment/RL-style objective).
- Use appropriate **constraint-satisfaction metrics** (e.g., validity/feasibility rate, violation magnitude) alongside **quality/diversity** metrics.
- Provide a short **written report** that describes the method, experimental setup, ablations, and main findings, plus a clear description of limitations/failure modes.
- Provide a **reproducibility package** (code, instructions, and configs) sufficient for another student to run core experiments and regenerate the main plots/tables.

**Assessment Criteria:**
- Problem formulation quality (clarity of constraints, verification procedure, and evaluation metrics).
- Technical quality of the approach (correctness, soundness of the constraint mechanism, and connection to course concepts).
- Empirical rigor (baseline strength, ablations, and robustness/stress tests for constraints).
- Results and analysis (interpretability of outcomes, error analysis, and discussion of tradeoffs).
- Reproducibility (clean code organization, documentation, and ability to rerun key results).
- Presentation quality (clarity, structure, and ability to answer questions).


---


## **Course policies (summary)**

**Collaboration:** Discussion is encouraged. Submitted work must be written independently unless an assignment explicitly permits collaboration.

**Late policy:** TBA

**Academic integrity:** TBA

**Accessibility:** TBA

**Use of generative AI tools:** Permitted for brainstorming, debugging, and editing with attribution, unless a specific assignment forbids it. Students are responsible for correctness and for documenting tool use.


<!-- # **Reading plan**

Diffusion models: DDPM; score-based SDE unification.

Flow methods: Flow Matching; Rectified Flow.

Discrete diffusion: D3PM.

Constrained decoding: Grid Beam Search.

Robotics: Diffusion Policy; Diffuser.

Differentiable optimization: OptNet; Differentiable Convex Optimization Layers.

Alignment: InstructGPT; DPO.

Geometry: Geometric Deep Learning survey.

## **Application anchors (for invited and lab discussions)**

Proteins: RFdiffusion.

Docking: DiffDock.

Materials: CDVAE; DiffCSP; MatterGen. -->
