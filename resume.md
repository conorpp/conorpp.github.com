---
layout: page
title: Conor Patrick
tagline: Resume
nocover: true
---

Check out my [Github profile]({{site.social[1].url}}) for some of my active and old projects.  

Check out links below for press and videos.

Résumé
======

###### (Last updated {{ site.data.resume.updated | date: '%B, %Y' }})

**{{site.data.resume.name}}**

**{{site.data.resume.contact}}**    (pgp key: {{site.data.resume.fingerprint}})

Current residence: {{site.data.resume.location}}

[Website]({{site.data.resume.website}}), [Twitter](https://twitter.com/{{site.data.resume.twitter}})


{% for b in site.data.resume.subjects %}


{{ b.name }}
===========


    {% for i in b.items %}
        {% unless i.hide %}

        {% if i.type == "plain" %}

* {{i.content}}

        {% elsif i.type == "label" %}

* **{{i.name}}**: {{i.content}}
        
        {% elsif i.type == "link" %}

* **[{{ i.name }}]({{ i.content }})**

        {% elsif i.type == "list" %}

* {{i.name}}

            {% for li in i.content %}
  * {{li.content}}
            {% endfor %}

        {% else %}
    
<h1 style="color:red;">invalid format type: {{i.type}}<h1>

        {% endif %}
        {% endunless %}

    {% endfor %}


{% endfor %}

{% include JB/setup %}
