---
layout: page
title: Tech
categories: tech
permalink: /tech/
---
<h2>{{ page.list_title | default: "Posts" }}</h2>
<div>
  <ul>
    {% assign page_category = page.categories %}
    {% for post in site.posts %}
      {% if post.categories contains page_category %}
      <li>
        {%- assign date_format = site.minima.date_format | default: "%B %-d, %Y" -%}
        {{ post.date | date: date_format }}:
        <a href="{{ post.url | relative_url }}">
          {{ post.title | escape }}
        </a>
      </li>
      {% endif %}
    {% endfor %}
  </ul>
</div>
