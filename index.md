---
layout: page
title: Conor Patrick
tagline: Hardware projects, security
nocover: true
---

{% include JB/setup %}

I love to meet others and keep in touch.  

Feel free to [reach out](/contact), whether if you have questions, want some advice, or just want to chat.



{% include JB/signup %}

* [Publications and talks](/research)

<br>

{% assign posts =  site.posts  %}

{% for post in posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | [{{post.tags[0]}}]({{BASE_PATH}}/tags.html#{{post.tags[0]}}) | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) | {% endfor %}

<br>


<br>
<div>
<a href="/archive">Archive</a>
</div>

