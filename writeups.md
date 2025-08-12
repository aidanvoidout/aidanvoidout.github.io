---
layout: page
title: Writeups
permalink: /writeups/
---

<ul>
  {% for writeup in site.writeups %}
    <li>
      <a href="{{ writeup.url | relative_url }}">{{ writeup.title }}</a>
    </li>
  {% endfor %}
</ul>
