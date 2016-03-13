---
layout: page
title: Conor Patrick
tagline: Security, Privacy
---

{% include JB/setup %}

You can see my resume [here](/resume.html).
                            
I'm currently [looking for work in the government](/work).

To contact me, [tweet](https://twitter.com/_conorpp) at me or send me an email at {{site.author.email}}


||||
|:--------|----|---|-|{% for post in site.posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | [{{post.tags[0]}}]({{BASE_PATH}}/tags.html#{{post.tags[0]}}) | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) |{% endfor %}


