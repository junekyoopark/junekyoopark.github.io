---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: 아무말대잔치
permalink: /whatever/
---
{% for whatever in site.whatever %}
  <h2>
    <a href="{{ whatever.url }}">
      {{ whatever.title }}
    </a>
  </h2>
  <p> {{ whatever.content | markdownify | strip_html | truncatewords: 20 }}</p>
{% endfor %}