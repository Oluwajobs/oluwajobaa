---
layout: single
title: "Assorted collection of other projects"
permalink: /projects/
collection: portfolio
author_profile: true
---

Here's a collection of my projects. I will try to keep this page as updated as possible. Please get in touch for more details about any of them.


<!--
## [Climate Proofing Cities: A Resilience Analysis](https://anamika255.github.io/portfolio/C40-Cities/)

100 climate proofing strategies were implemented by the global consortium of C40 cities. How did those strategies fare on a resilience landscape?
Here's how to add link to the pages (/assets/files/C40_report.pdf)

### Other projects coming up soon.
-->

{% include base_path %}

{% for post in site.portfolio %}
  {% include archive-single.html %}
{% endfor %}
