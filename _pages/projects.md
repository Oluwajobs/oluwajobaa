---
permalink: /projects/
layout: collection
collection: projects
title: "Projects"
author_profile: true
entries_layout: grid
---

<!--
title: Portfolio
layout: collection
permalink: /portfolio/
collection: portfolio
entries_layout: grid
classes: wide
![](/assets/images/webPageConstruction.jpg)
-->


{% include base_path %}


{% for post in site.portfolio %}
  {% include archive-single.html %}
{% endfor %}
