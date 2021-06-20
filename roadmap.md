---
layout: page
group: roadmap
title: Roadmap
permalink: /roadmap/
---

# Roadmap
{: .rl-article-title }

Crochet is still under active development. An experimental implementation is
planned to be released in 2021.

{% include roadmap-status.html %}


## Milestones

{% for milestone in site.data.roadmap.milestones %}
{% assign is_current = false %}
{% assign current_milestone = forloop.index %}

{% if site.data.roadmap.milestone > forloop.index %}
  <h3><strike>{{ milestone.title }}</strike></h3>
{% elsif site.data.roadmap.milestone == forloop.index %}
  {% assign is_current = true %}
  <h3>{{ milestone.title }}</h3>
{% else %}
  <h3>{{ milestone.title }}</h3>
{% endif %}

| **Expected release:** | {{ milestone.release }} |
{: .common-table .rl-skip-small }

{{ milestone.goal }}

<ul>
  {% for step in milestone.steps %}
    {% if ((current_milestone < site.data.roadmap.milestone) or ((site.data.roadmap.milestone == current_milestone) and (site.data.roadmap.step > forloop.index))) %}
      <li><strike>{{ step }}</strike></li>
    {% elsif is_current and site.data.roadmap.step == forloop.index %}
      <li><strong>{{ step }}</strong></li>
    {% else %}
      <li>{{ step }}</li>
    {% endif %}
  {% endfor %}
</ul>

{% endfor %}
