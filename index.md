---
layout: page
title: Conor Patrick
tagline: Security, Privacy
---
{% include JB/setup %}

Hi, I'm Conor.  I'm a senior Computer Engineering major at Virginia Tech.  I have experience working as a software engineeer for Rockwell Collins
and as a security researcher for the FTC.  I like going to hackathons and doing security research in my spare time.  

If you'd like to chat, tweet at me or send me an PGP encrypted message at {{site.author.email}}



||||
|:--------|----|---|-|{% for post in site.posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | {{post.category}} | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) |{% endfor %}



