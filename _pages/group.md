---
layout: default
permalink: /group/
title: group
description:
nav: true
nav_order: 3
display_categories: [Current Students, Former Students, Visiting Students and Interns]
horizontal: false
---

# **R**esponsible **AI** for **S**cience and **E**ngineering (RAISE) Group

I lead the RAISE group where I am fortunate to work with an amazing set of students and collaborators.
We work on foundational topics relating to machine learning and optimization, privacy and fairness.
We often ground our research in applications at the intersection of physical sciences and energy, as
well as policy and decision making.



<!-- pages/projects.md -->
<div class="projects">
{%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {%- for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {%- assign categorized_projects = site.group | where: "category", category -%}
  {%- assign sorted_projects = categorized_projects | sort: "importance" %}
  
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include group.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {% endfor %}

{%- else -%}
<!-- Display projects without categories -->
  {%- assign sorted_projects = site.group | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include group.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
{%- endif -%}
</div>


<!-- <div class="projects">
  <h2 class="category">Former Students and Visitors</h2>
</div>
  
  - **[Cuong Tran](https://cuongtran-syr.github.io)**, Phd @Syracuse (2020-2023); Postdoc @UVA (2024)<br>
    _Next position_: Research Scientist, Amazon
  - **[Eric Nguyen](https://www.joonhyukko.com)**  MS @UVA (Spring 2024)
  - **[Jayanta Mandi](https://jayantamandi.com)** PhD @VUB (visitor, Summer 2023)<br>
    _Next position_: Postdoc at Vrije Universiteit Brussel
  - **[Kyle Beiter]()** BS @Syracuse (NSF REU, Summer 2021)<br>
    _Next position_: Data Scientist at Optiver
  - **[Michele Marchiani]()** Visiting MS student (Winter 2022)<br>
    _Next position_: MS, University of Parma 
  - **[Pranav Putta](https://www.linkedin.com/in/pranav-putta-3512b47a/)** BS @Georgia Tech (NSF REU, Summer 2021)
  - **[Rakshit Naidu](https://rakshit-naidu.github.io)** MS @CMU (Intern, Summer 2022)<br>
    _Next position_: PhD student at Georgia Tech
  - **[Razane Tajeddine](https://scholar.google.com/citations?user=AyF89JgAAAAJ&hl=en)** Postdoc at University of Helsinki (Visitor, Fall 2023)
  - **[St John Grimbly](https://stjohngrimbly.com)** MS @University of Cape Town (Intern, Winter 2023)<br>
    _Next position_: PhD position, University of Cape Town
  - **[Yehya Farhat](https://yehya-farhat.github.io)**} MS, Syracuse University, (Thesis advisor 2022)\\
    _Next position_: PhD student at Rice University
  - **[Zhiyan Yao]()**, BS (Intern, Summer 2021)<br>
    _Next position_: Software Engineer at Microsoft
 -->
<!-- 
  \textbf{Shujun Xia} (BS, City University of Hong Kong, Summer 2024), 
  \textbf{Zarreen Reza} (BS, OpenMined, Spring 2024), 
  \textbf{Eric Nguyen} (BS, University of Virginia, Fall 2023),
  \textbf{Catherine Smolka} (HS, Deep Run High School, VA, 2023-2024), 
  %%
  \textbf{Pranav Putta} (BS, GaTech, Summer 2023) [NSF REU],
  \textbf{Winston Tsui} (BS, SU Summer 2023),
  \textbf{Zhongquan Cheng} (BS SU, Summer 2023), 
  %%
  \textbf{Adya Parida} (BS SU, Fall 2022) [NSF REU], 
  \textbf{Deniz Gursoy} (HS, Fayetteville High School, Summer 2022), 
  \textbf{Saswat Das} (BS, ITS, Summer 2022), 
  \textbf{Utsav Pathak} (BS, Alliance University, Bengaluru, Summer 2022),
  \textbf{Daiwei Shen} (BS, Northwestern, Summer 2022),
  \textbf{Sunisth Kumar} (BS, Bennett University, Summer 2022),
  %%
  \textbf{Kyle Beiter} (BS, SU, Summer 2021) [NSF REU],  %NSF REU
  \textbf{Shantanu Jhaveri} (BS, USC, Summer 2021) [NSF REU], % NSF REU
  \textbf{Dayong Gu} (BS, SU, Summer 2021),
  \textbf{Guoliang Chen} (BS, SU, Summer 2021),
  \textbf{Pradyumn Yadav} (BS, SU, Summer 2021),
  %%
  \textbf{Anudit Nagar} (BS, SU, Summer 2020 -- Current), 
  \textbf{Zhiyan Yao} (BS, SU, Summer 2020 -- Current),
  \textbf{Zifei Lu} (BS, SU, Summer 2020),
  \textbf{Thomas Montfort} (BS, SU, Summer 2020),
  \textbf{Cong Liu} (BS, SU, Summer 2020),
  \textbf{Pratik Paranjape}, (BS, SU, Summer 2020),
  % {\sc Research}: Generating datasets for preference elicitation. 
  % {\sc Next position:} \textit{Developer at OthersideAI}
  \textbf{Pavan Kumar Vaddineni} (BS, SU, Spring 2020),
  % {\sc Research}: Explainable and Fair Learning. 
  % {\sc Next position:} \textit{Same}
  \textbf{William Kluegel}, (BS, NMSU, 2016 -- 2018), 
  % {\sc Research}: \textit{Optimization and Preferences Elicitation for Smart Home Devices.}
  % {\sc Next position:} \textit{Sandia National Labs}
  \textbf{Lyndon Shi} (BS, UMich, 2018), 
  \textbf{Jiayu Chen} (BS, UMich, 2018), 
  \textbf{Eric Frechette} (BS, NMSU, 2016).
\medskip -->