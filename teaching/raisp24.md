---
layout: default
title: Reponsilble AI
---

# Responsible AI: Privacy, Fairness, and Robustness Seminar

## Course Description

This seminar-style course delves into the ethical dimensions of Artificial Intelligence (AI), with a particular focus on the intersectionality of privacy, fairness, and robustness. The course is structured around reading, discussing, and critically analyzing seminal and state-of-the-art papers in the field. Participants will engage in intellectual discourse to understand the challenges, methodologies, and emerging trends related to responsible AI. The course is designed for graduate students with good ML, stats, and optimization background.

## Course Objectives

- Critically assess and discuss the literature on privacy, fairness, and robustness in AI.
- Identify challenges and propose potential solutions for responsible AI.
- Foster interdisciplinary discussions to explore the ethical dimensions of AI.
- Engage in a deep intellectual exploration of the field through paper discussions and presentations.

## Prerequisites

- Basic understanding of machine learning.
- Basic understanding of optimization.
  
## Syllabus

This is a tentative calendar and it is subject to change.

| Date       | Topic                     | Subtopic                           | Papers                       | Presenting   |
|------------|-----------------|------------------------------------|------------------------------|--------------|
| Wed Jan 17 | Intro to class  |                                    | [class slides](../raisp24_files/lec01.pdf)           | Fioretto     |
| Mon Jan 22 | Intro to class  | Safety and Alignment               | [class slides](../raisp24_files/lec02.pdf)           | Fioretto     |
| Wed Jan 24 | Intro to class  | Privacy (settings and attacks)     | [class slides](../raisp24_files/lect-3-and-4.pdf)           | Fioretto     |
| Mon Jan 29 | Intro to class  | Privacy (cont)                     | [class slides](../raisp24_files/lect-3-and-4.pdf)           | Fioretto     |
| Wed Jan 31 | Intro to class  | Privacy and Fairness               | [class slides](../raisp24_files/lec05.pdf)           | Fioretto     |
| Mon Feb 5  | Fairness        | Intro and bias sources             | [[1](#r1)] -- [[4](#r4)]     | Group 1      |
| Wed Feb 7  | Fairness        | Statistical measures               | [[5](#r5)] -- [[8](#r8)]     | Group 2      |
| Mon Feb 12 | Fairness        | Tradeoffs                          | [[9](#r9)] -- [[12](#r12)]   | Group 3      |
| Wed Feb 14 | Fairness        | LLMs: Toxicy and Bias              | [[13](#r13)] -- [[16](#r16)] | Group 4      |
| Mon Feb 19 | Fairness        | LLMs: Fairness                     | [[17](#r17)] -- [[19](#r19)] | Group 5      |
| Wed Feb 21 | Fairness        | Policy aspects                     | [[20](#r20)] -- [[22](#r22)] | Group 6      |
| Mon Feb 26 | No class (AAAI) |                                    |                              |              |
| Wed Feb 28 | Safety          | Distribution shift                 | [[23](#r23)] --  [[25](#r25)]| Group 1      |
| Mon Mar 4  | Spring break    |                                    |                              |              |
| Wed Mar 6  | Spring break    |                                    |                              |              |
| Mon Mar 11 | Safety          | Poisoning                          | [[26](#r26)] -- [[29](#r29)] | Group 2      |
| Wed Mar 13 | Safety          | Adversarial Robustness             | [[30](#r30)] -- [[34](#r34)] | Group 3      |
| Mon Mar 18 | Safety          | Adversarial Robustness             | [[35](#r35)] -- [[39](#r39)] | Group 4      |
| Wed Mar 20 | Safety          | LLMs: Prompt injection             | [[40](#r40)] -- [[45](#r45)] | Group 5      |
| Mon Mar 25 | Safety          | LLMs: Jailbreaking                 | [[46](#r46)] -- [[50](#r50)] | Group 6      |
| Wed Mar 27 | Privacy         | Differential Privacy 1             | [[51](#r51)] -- [[55](#r55)] | Group 1      |
| Mon Apr 1  | Privacy         | Differential Privacy 2             | [[56](#r56)] -- [[58](#r58)] | Group 2      |
| Wed Apr 3  | Privacy         | Differentially Private ML          | [[59](#r59)] -- [[61](#r61)] | Group 3      |
| Mon Apr 8  | Privacy         | Auditing and Membership inference  | [[62](#r62)] -- [[65](#r65)] | Group 4      |
| Wed Apr 10 | Privacy         | Privacy and Fairness               | [[66](#r66)] -- [[69](#r69)] | Group 5      |
| Mon Apr 15 | Privacy         | LLMs: Privacy in LLMs              | [[70](#r70)] -- [[73](#r73)] | Group 6      |
| Wed Apr 17 | Evaluation      | Model cards                        | [[74](#r74)] -- [[77](#r77)] | Group 1      |
| Mon Apr 22 | Evaluation      | LLMs: evaluation                   |                        	  | Group 2      |
| Wed Apr 24 | Unlearning      | Unlearning 1                       |                        	  | Group 3      |
| Mon Apr 29 | Unlearning      | LLMs: Targeted unlearning          |                        	  | Group 4          

<br>
### Bibliography

<!-- Fairnes: Definitions  -->
- <a id="r1">[1].</a> [Fairness and Machine Learning, Ch 1](https://fairmlbook.org/introduction.html). S. Barocas, M. Hardt, A. Narayanan, 2023
- <a id="r2">[2].</a> [Big Data: A Report on Algorithmic Systems, Opportunity, and Civil Rights](https://obamawhitehouse.archives.gov/sites/default/files/microsites/ostp/2016_0504_data_discrimination.pdf). The White House, 2016
- <a id="r3">[3].</a> [Big Data’s Disparate Impact](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2477899). S. Barocas, A. Selbst, 2014
- <a id="r4">[4].</a> [Semantics derived automatically from language corpora contain human-like biases](https://www.science.org/doi/10.1126/science.aal4230) A. Caliskan, J.J. Bryson, A. Narayanan, 2017

<!-- Fairness: Statistical Meausres -->
- <a id="r5">[5].</a> [Fairness and Machine Learning, Ch 3](https://fairmlbook.org/classification.html). S. Barocas, M. Hardt, A. Narayanan, 2023
- <a id="r6">[6].</a> [Fairness Through Awareness](https://arxiv.org/abs/1104.3913). C. Dwork, M. Hardt, T. Pitassi, O. Reingold, R. Zemel, 2011
- <a id="r7">[7].</a> [Learning Fair Representations](https://www.cs.toronto.edu/~toni/Papers/icml-final.pdf). R. Zemel, Y Wu, K. Swersky, T. Pitassi, C Dwork, 2013
- <a id="r8">[8].</a> [Equality of Opportunity in Supervised Learning](https://arxiv.org/abs/1610.02413). M. Hardt, E. Price, N. Srebro, 2016

<!-- Fairness: Tradeoffs -->
- <a id="r9">[9].</a>  [Fair prediction with disparate impact: A study of bias in recidivism prediction instruments](https://arxiv.org/abs/1610.07524). A. Chouldechova, 2016
- <a id="r10">[10].</a> [Algorithmic decision making and the cost of fairness](https://arxiv.org/abs/1701.08230). S. Corbett-Davies, E. Pierson, A. Feller, S. Goel, A. Huq, 2017
- <a id="r11">[11].</a> [Inherent Trade-Offs in the Fair Determination of Risk Scores](https://arxiv.org/abs/1609.05807). J. Kleinberg, S. Mullainathan, M. Raghavan, 2017
- <a id="r12">[12].</a> [On the (im)possibility of fairness](https://arxiv.org/abs/1609.07236). S.A. Friedler, C. Scheidegger, S. Venkatasubramanian, 2017

<!-- Fairness: Bias and Toxicity in LLMs  -->
- <a id="r13">[13].</a> [On the Dangers of Stochastic Parrots: Can Language Models Be Too Big?](https://dl.acm.org/doi/pdf/10.1145/3442188.3445922). E.M. Bender, T. Gebru, A. McMillan-Major, S. Shmitchell, 2021.
- <a id="r14">[14].</a> [RealToxicityPrompts: Evaluating Neural Toxic Degeneration in Language Models](https://arxiv.org/abs/). S. Gehman, S. Gururangan, M. Sap, Y. Choi, N.A. Smith, 2020
- <a id="r15">[15].</a> [OPT: Open Pre-trained Transformer Language Models](https://arxiv.org/abs/2205.01068). Zhang et al., 2022
- <a id="r16">[16].</a> [StereoSet: Measuring stereotypical bias in pretrained language models](https://aclanthology.org/2021.acl-long.416/). M. Nadeem, A. Bethke, S. Reddy, 2021

<!-- Fairness: Fairness in LLMs  -->
- <a id="r17">[17].</a> [Whose Language Counts as High Quality? Measuring Language Ideologies in Text Data Selection](https://arxiv.org/pdf/2201.10474.pdf). S. Gururangan et al., 2022
- <a id="r18">[18].</a> [Social Bias Frames: Reasoning about Social and Power Implications of Language](https://aclanthology.org/2020.acl-main.486/). M. Sap, S. Gabriel, L. Qin, D. Jurafsky, N.A. Smith, Y. Choi, 2020.
- <a id="r19">[19].</a> [Bias and Fairness in Large Language Models: A Survey](https://arxiv.org/abs/2309.00770). I.O. Gallegos et al. 2023

<!-- Fairness: Policy aspects -->
- <a id="r20">[20].</a> [Fairness and Machine Learning, Ch 6](https://fairmlbook.org/legal.html). S. Barocas, M. Hardt, A. Narayanan, 2023
- <a id="r21">[21].</a> [Big Data's Disparate Impact](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2477899). S. Barocas, A.D. Selbst, 2016
- <a id="r22">[22].</a> [How Copyright Law Can Fix Artificial Intelligence's Implicit Bias Problem](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3024938). A. Levendowski, 2022

<!-- Distribution shift -->
- <a id="r23">[23].</a> [Failing Loudly: An Empirical Study of Methods for Detecting Dataset Shift](https://arxiv.org/abs/1810.11953) S. Rabanser et al, 2018
- <a id="r24">[24].</a> [Revisiting the Calibration of Modern Neural Networks](https://arxiv.org/abs/2106.07998). Minderer et al., 2021
- <a id="r25">[25].</a> [Deep Gamblers: Learning to Abstain with Portfolio Theory](https://arxiv.org/abs/1907.00208). Ziyin et al., 2019

<!-- Poisoning attacks -->
- <a id="r26">[26].</a> [Poisoning attacks against support vector machines](https://arxiv.org/abs/1206.6389). Biggio et al. 2012
<!-- This paper is a cornerstone in the study of poisoning attacks, particularly against support vector machines (SVMs). It provides a thorough analysis of the attack strategies and their implications, offering foundational insights into the vulnerabilities of machine learning models. -->
- <a id="r27">[27].</a> [Manipulating machine learning: Poisoning attacks and countermeasures for regression learning](https://arxiv.org/abs/1804.00308). Jagielski et al. 2018
<!-- This paper extends the concept of poisoning attacks to regression models, which are widely used in various applications. It not only explores the attack methodologies but also discusses countermeasures, making it a critical read for understanding both sides of the coin. -->
- <a id="r28">[28].</a> [Certified defenses for data poisoning attacks](https://arxiv.org/abs/1706.03691). Steinhardt et al. 2017
<!-- This work proposes certified defenses against data poisoning attacks, offering a novel perspective on how to protect machine learning models. It is pivotal for understanding the defense mechanisms that can be put in place to ensure the integrity of machine learning systems. -->
- <a id="r29">[29].</a> [Poison frogs! Targeted clean-label poisoning attacks on neural networks](https://arxiv.org/abs/1804.00792). Shafahi et al. 2018
<!-- This paper introduces a sophisticated form of poisoning attack that is hard to detect, known as the "clean-label" poisoning attack. It's essential for comprehending the evolving nature of poisoning attacks and the challenges they pose to defense mechanisms. -->

<!-- Adversarial Examples -->
- <a id="r30">[30].</a> [Intriguing properties of neural networks](https://arxiv.org/abs/1312.6199). Szegedy et al. 2013
- <a id="r31">[31].</a> [Explaining and Harnessing Adversarial Examples](https://arxiv.org/abs/1412.6572). Goodfellow et al. 2014
- <a id="r32">[32].</a> [Towards Evaluating the Robustness of Neural Networks](https://arxiv.org/abs/1608.04644). Carlini and Wagner. 2017
- <a id="r33">[33].</a> [Adversarial examples in the physical world](https://arxiv.org/abs/1607.02533). Kurakin et al. 2018
- <a id="r34">[34].</a> [Adversarial Examples Are Not Bugs, They Are Features](https://arxiv.org/abs/1905.02175). Ilyas et al. 2019

<!-- Adversarial Exampless: Defenses -->
- <a id="r35">[35].</a> [Provable defenses against adversarial examples via the convex outer adversarial polytope](https://arxiv.org/abs/1711.00851) Wong and Kolter, 2017
- <a id="r36">[36].</a> [Scaling provable adversarial defenses](https://proceedings.neurips.cc/paper_files/paper/2018/file/358f9e7be09177c17d0d17ff73584307- ) Wong et al. 2018
- <a id="r37">[37].</a> [Towards Deep Learning Models Resistant to Adversarial Attacks](https://arxiv.org/abs/1706.06083). Madry et al. 2018
- <a id="r38">[38].</a> [Distillation as a Defense to Adversarial Perturbations against Deep Neural Networks](https://arxiv.org/abs/1511.04508). Papernot et al. 2016
- <a id="r39">[39].</a> [Theoretically Principled Trade-off between Robustness and Accuracy](https://arxiv.org/abs/1901.08573).  Zhang et al. 2019

<!-- LLMs: injection -->
- <a id="r40">[40].</a> [Universal Adversarial Triggers for Attacking and Analyzing NLP](https://arxiv.org/abs/1908.07125). Wallace et al. 2019
- <a id="r41">[41].</a> [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165). Browns et al. 2020 
<!-- Although not exclusively about prompt injection, this paper by OpenAI introduces GPT-3 and demonstrates the power of carefully designed prompts in guiding the model to perform a wide range of tasks without task-specific training data, laying the groundwork for understanding the potential of prompt engineering in LLMs.  -->
- <a id="r42">[42].</a> [How Can We Know What Language Models Know?](https://arxiv.org/abs/1911.12543) Jiang. 2020
- <a id="r43">[43].</a> [On the Dangers of Stochastic Parrots: Can Language Models Be Too Big?](https://dl.acm.org/doi/10.1145/3442188.3445922). Bender et al., 2021
- <a id="r44">[44].</a> [Prompt Injection attack against LLM-integrated Applications](https://arxiv.org/abs/2306.05499). Liu et al, 2023
- <a id="r45">[45].</a> [Nvidia Blog - securing against prompt injection attacks](https://developer.nvidia.com/blog/securing-llm-systems-against-prompt-injection/). 2023
<!-- A survey  on prompt injection attacks in LLMs -->

<!-- Jailbreaking -->
- <a id="r46">[46].</a> [Universal and Transferable Adversarial Attacks on Aligned Language Models](https://arxiv.org/abs/2307.15043) Zou et al. 2023
- <a id="r46">[47].</a> [LLM Censorship: A Machine Learning Challenge or a Computer Security Problem?](https://arxiv.org/abs/2307.10719) Glukhov et al. 2023
- <a id="r46">[48].</a> ["Do Anything Now": Characterizing and Evaluating In-The-Wild Jailbreak Prompts on Large Language Models](https://arxiv.org/abs/2308.03825) Shen et al. 2023
- <a id="r46">[49].</a> [Visual Adversarial Examples Jailbreak Aligned Large Language Models](https://arxiv.org/abs/2306.13213) Qi et al. 2023
- <a id="r46">[50].</a> [Coercing LLMs to do and reveal (almost) anything](https://arxiv.org/abs/2402.14020) Geiping et al. 2024


<!-- DP -->
- <a id="51">[51].</a> <a href="http://www.gautamkamath.com/courses/CS860-fa2022-files/" rel="external nofollow noopener" target="_blank">Lectures 2 to 4 (notes)</a> by Gautam Kamath.
- <a id="52">[52].</a> <a href="https://ecommons.cornell.edu/items/046034b9-9365-436b-88aa-e8c3fae94b7c" rel="external nofollow noopener" target="_blank">Understanding Database Reconstruction Attacks on Public Data</a> by S Garfinkel, JM Abowd, C Martindale.
- <a id="53">[53].</a> <a href="https://www.pnas.org/doi/10.1073/pnas.2300976120" rel="external nofollow noopener" target="_blank">  Database reconstruction does compromise confidentiality</a> by SA Keller and JM Abowd.
- <a id="54">[54].</a> <a href="https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf" rel="external nofollow noopener" target="_blank">Sections 2, 3.1, 3.2 of the Algorithmic Foundations of Differential Privacy</a> by Cynthia Dwork and Aaron Roth.
- <a id="55">[55].</a> <a href="https://programming-dp.com/cover.html" rel="external nofollow noopener" target="_blank">Programming Differential Privacy</a> Joseph P. Near and Chiké Abuah (additional resources)

<!-- DP -->
- <a id="56">[56].</a> <a href="http://www.gautamkamath.com/courses/CS860-fa2022-files/" rel="external nofollow noopener" target="_blank">Lectures 5 to 8 (notes)</a> by Gautam Kamath.
- <a id="57">[57].</a> <a href="https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf" rel="external nofollow noopener" target="_blank">Sections  3.3, 3.4, 10.1-10.2 of the Algorithmic Foundations of Differential Privacy</a> by Cynthia Dwork and Aaron Roth.
- <a id="58">[58].</a> <a href="https://programming-dp.com/cover.html" rel="external nofollow noopener" target="_blank">Programming Differential Privacy</a> Joseph P. Near and Chiké Abuah (additional resources)

- <a id="59">[59].</a> [Differentially Private Empirical Risk Minimization](https://www.jmlr.org/papers/volume12/chaudhuri11a/chaudhuri11a.pdf). Chaudhuri et al 2011.
- <a id="60">[60].</a> [Deep Learning with Differential Privacy](https://arxiv.org/abs/1607.00133). Abadi et al, 2016
- <a id="61">[61].</a> [Semi-supervised Knowledge Transfer for Deep Learning from Private Training Data](https://arxiv.org/abs/1610.05755). Papernot et al, 2016

<!-- Membership inference -->
- <a id="62">[62].</a> [Membership Inference Attacks against Machine Learning Models](https://arxiv.org/abs/1610.05820) Shokri et al. 2017
- <a id="63">[63].</a> [Membership Inference Attacks From First Principles](https://arxiv.org/abs/2112.03570) Carlini et al. 2021
- <a id="64">[64].</a> [The Secret Sharer: Evaluating and Testing Unintended Memorization in Neural Networks](https://arxiv.org/abs/1802.08232) Carlini et al. 2018
- <a id="65">[65].</a> [Auditing Differentially Private Machine Learning: How Private is Private SGD?](https://arxiv.org/abs/2006.07709)  Jagielski et al 2020


- <a id="66">[66].</a> [Differential Privacy and Fairness in Decisions and Learning Tasks: A Survey](https://arxiv.org/abs/2202.08187) Fioretto et al, 2022.
- <a id="67">[67].</a> [On the Compatibility of Privacy and Fairness](https://rachelcummings.com/wp-c<a id="65">[65].</a> ontent/uploads/2019/03/FairPrivate.pdf) Cummings et al. 2019
- <a id="68">[68].</a> [Differential Privacy Has Disparate Impact on Model Accuracy](https://arxiv.org/abs/1905.12101) Bagdasaryan 2019
- <a id="69">[69].</a> [Differentially Private Empirical Risk Minimization under the Fairness Lens](https://arxiv.org/abs/2106.02674) Tran et al 2021

- <a id="70">[70].</a> [Scalable Extraction of Training Data from (Production) Language Models](https://arxiv.org/abs/2311.17035) Nasar et al 2023.
- <a id="71">[71].</a> [Can LLMs Keep a Secret? Testing Privacy Implications of Language Models via Contextual Integrity Theory](https://arxiv.org/abs/2310.17884?context=cs) Mireshghallah 2023
- <a id="72">[72].</a> [Beyond Memorization: Violating Privacy Via Inference with Large Language Models](https://arxiv.org/abs/2310.07298v1) Staab et al 2023
- <a id="73">[73].</a> [Privacy issues in Large Language Models: A Survey](https://arxiv.org/pdf/2312.06717.pdf). Sections 3,4, and 5. Neel 2024. 

- <a id="74">[74]</a> [Model Cards for Model Reporting](1810.03993) Mitchell et al. 2018.
- <a id="75">[75]</a> [Datasheets for Datasets](https://arxiv.org/abs/1803.09010) Gebru et al. 2018.
- <a id="76">[76]</a> [The Values Encoded in Machine Learning Research](https://arxiv.org/abs/2106.15590) Birhane, 2021.
- <a id="77">[77]</a> [Data Cards: Purposeful and Transparent Dataset Documentation for Responsible AI](https://dl.acm.org/doi/abs/10.1145/3531146.3533231) Pushkarna, 2022

<!-- Evaluation: https://fairmlbook.org/testing.html -->
- <a id="78">[78]</a> [On the Opportunities and Risks of Foundation Models](https://arxiv.org/abs/2108.07258) Bommasani et al. 2022.

## Assessment

Each group will be assessed through the following activities:

- Paper Summaries (blogging): 33.3%
- Presentation: 33.3%
- Discussion Lead: 33.3%


##### 1. Paper Summaries (Blogging) – 33.3%

**Objective:** To develop the ability to critically analyze and summarize AI research papers in a clear and accessible manner.

**Expectations:**
- Each group will reivew all paper from the provided list, and they may propose additional ones for approval.
- Summaries should be written in Markdown format (supporting images and formulas) and committed to the course's [GitHub repository](https://github.com/uva-responsibleai/fall-24).
- The summary should include the following sections: Introduction and Motivations, Methods, Key Findings, and Critical Analysis.
- The Critical Analysis section should evaluate the strengths, weaknesses, potential biases, and ethical considerations of the paper.
- Summaries must be submitted **four days** prior to the presentation for review and potential feedback.

**Assessment Criteria:**
- Clarity and coherence of the written summary.
- Depth of critical analysis and understanding of the paper's content.
- Proper use of formatting and adherence to submission guidelines.
- Timeliness of submission.

##### 2. Presentation – 33.3%

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

##### 3. Discussion Lead – 33.3%

**Objective:** To cultivate skills in leading intellectual discourse and fostering collaborative learning.

**Expectations:**
- 30-minute discussion session following the presentation.
- Groups should prepare and facilitate a discussion based on their presentation.
- Use of supplementary materials (e.g., videos, code snippets) to enrich the discussion is encouraged.
- The discussion should engage the audience (with active questions), encouraging diverse viewpoints and deeper understanding of the topic.

**Assessment Criteria:**
- Ability to foster an inclusive and constructive discussion.
- Relevance and depth of prepared questions and discussion points.
- Engagement level of the audience during the discussion.
- Use of supplementary materials to enhance understanding.

### General Notes:
- All group members are expected to contribute equally to each component, but two to three members are expected to lead one of the three components.
- Peer evaluation within groups may be used to ensure fair contribution.

## Recommended Reading

- A curated list of papers will be provided at the start of the course.

## Groups

| Group     |  Members                                                                                |
|-----------|-----------------------------------------------------------------------------------------|
| Group 1   | Lei Gong, Archit Uniyal, Luke Benham, Chien-Chen Huang, Stuart Paine                    | 
| Group 2   | Saswat Das, Wenqian Ye, Benny Bigler-Wang, Parker Hutchinson, Linyun Wei, Zhiyang Yuan  | 
| Group 3   | Nibir Mandal, Guangzhi Xiong, Neh Joshi, Sree Esshaan Mahajan, Esshaan Mahajan          |
| Group 4   | Sarvin Motamen, Parth Kandharkar, Ellery Yu, Hongyan Wu, Kefan Song,                    |
| Group 5   | Mati Ur Rehman, Jeffrey Chen, Candace Chen, Kaylee Liu, Robert Bao                      |
| Group 6   | Stephanie Schoch, Aidan Hesselroth,  Joseph Moretto, Jonathan McGee, ShiHe Wang         |

 

## Instructor

Ferdinando Fioretto
Assistant Professor in Computer Science
University of Virgina

---

This syllabus is subject to changes to meet the learning needs of the course participants.