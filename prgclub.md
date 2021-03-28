---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: 프로그래밍 공부 동아리
permalink: /prgclub/
---
{% for stuff in site.prgclub %}
  <h2>
    <a href="{{ stuff.url }}">
      {{ stuff.title }}
    </a>
  </h2>
  <p> {{ stuff.content | markdownify | strip_html | truncatewords: 20 }}</p>
{% endfor %}