---
layout: default
title: Writeups
permalink: /writeups/
---

<style>
ul h3 {
	display: inline-block;
	margin: unset;
}
</style>

# Writeups by CTF
Browse writeups <a href="../writeups_author/">by author</a> / <a href="../writeups_latest/">latest</a>

{% assign contests = site.writeups | group_by: "contest" %}

{% for contest in contests %}

# {{contest.name}}
<ul>
{% for writeup in contest.items %}
<li><a href="{{site.baseurl}}{{writeup.url}}"><h3>{{ writeup.title }} - {{ writeup.authors }}</h3></a></li>
{% endfor %}
</ul>

{% endfor %}
