---
layout: page
title: Writing
permalink: /blog/
description: Notes on .NET, Python, git, and developer tooling.
---

<ul class="postlist">
{%- for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">
      <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%d %b %Y" }}</time>
      <span>{{ post.title }}</span>
    </a>
  </li>
{%- endfor %}
</ul>
