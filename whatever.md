---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: 아무말대잔치
permalink: /whatever/
---
{% for stuff in site.whatever %}
  <h2>
    <a href="{{ stuff.url }}">
      {{ stuff.title }}
    </a>
  </h2>
  <p> {{ stuff.content | markdownify | strip_html | truncatewords: 20 }}</p>
{% endfor %}