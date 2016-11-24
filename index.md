---
layout: page
title: Conor Patrick
tagline: Security, Privacy
nocover: true
---

{% include JB/setup %}


I'm currently [looking for work in the government](/work).

* [Resume](/resume)
* [Research](/research)

To contact me, send me a [tweet](https://twitter.com/_conorpp) or an email at {{site.author.email}}

{% assign posts =  site.posts | where: "archive", "" %}

||||
|:--------|----|---|-|{% for post in posts %}
| <span class="nowrap">{{ post.date | date: "%b '%y" }}</span> | [{{post.tags[0]}}]({{BASE_PATH}}/tags.html#{{post.tags[0]}}) | [{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) | {% endfor %}

<br>
<div>
<a href="/archive">Archive</a>
</div>

