---
layout: page
title: Conor Patrick
tagline: Security, Privacy
nocover: true
---

{% include JB/setup %}

# Archive

Posts that I consider either out of date or just not helpful.

{% assign posts = site.archive %}

||||
|:--------|----|---|-|{% for post in posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | [{{post.tags[0]}}]({{BASE_PATH}}/tags.html#{{post.tags[0]}}) | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) | {% endfor %}


