---
layout: page
title: Conor Patrick
tagline: Hardware projects, security
nocover: true
---

{% include JB/setup %}


* [Resume](/resume)
* [Publications and talks](/research)

To contact me, send me a [tweet](https://twitter.com/_conorpp) or an email at {{site.author.email}}

<br>

{% assign posts =  site.posts  %}

{% for post in posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | [{{post.tags[0]}}]({{BASE_PATH}}/tags.html#{{post.tags[0]}}) | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) | {% endfor %}

<br>

{% include JB/signup %}

<br>
<div>
<a href="/archive">Archive</a>
</div>

