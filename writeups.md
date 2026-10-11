---
layout: page
title: Write-ups
permalink: /writeups/
---

Hands-on labs, CTF rooms, and bug bounty notes. TryHackMe flags are redacted, and bug bounty targets are anonymized.

<ul class="post-list">
  {% for post in site.posts %}
  <li>
    <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
    <h3><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
    {% if post.excerpt %}<p>{{ post.excerpt | strip_html }}</p>{% endif %}
    {% if post.tags %}<p class="post-meta">{{ post.tags | join: " · " }}</p>{% endif %}
  </li>
  {% endfor %}
</ul>
