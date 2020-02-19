---
layout: default
title: Home
nav_order: 1
description: "Welcome to the Blog Posts Home. This is Sudheesh's home for research work."
permalink: /
has_children: false
---

# Blog Posts
{: .fs-9 }

{% assign pages_list = site.html_pages | sort:"date" %}

{% for node in pages_list %}
	{% if node.date %}
### {{node.date | date_to_string }} : [{{node.title}}]({{node.url}})
#### {{node.summary | truncatewords: 40}}
	{% endif %}
{% endfor %}

