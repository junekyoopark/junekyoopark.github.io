---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: Ideas
permalink: /ideas/
---
{% for idea in site.ideas %}
  <h2>
    <a href="{{ idea.url }}">
      {{ idea.title }}
    </a>
  </h2>
  <p> {{ idea.content | markdownify | strip_html | truncatewords: 20 }}</p>
{% endfor %}