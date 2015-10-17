---
layout: page
title: Conor Patrick
tagline: Security, Privacy
---
{% include JB/setup %}

Hi, I'm Conor.  I'm a senior Computer Engineering major at Virginia Tech.  I have experience working as a software engineeer for Rockwell Collins
and as a security researcher for the FTC.  I like going to hackathons and doing security research in my spare time.  

You can see my resume [here](/resume.html).

If you'd like to chat, [tweet](https://twitter.com/_conorpp) at me or send me an email at {{site.author.email}}



||||
|:--------|----|---|-|{% for post in site.posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | [{{post.tags[0]}}]({{BASE_PATH}}/tags.html#{{post.tags[0]}}) | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) |{% endfor %}



