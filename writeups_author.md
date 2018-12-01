---
layout: default
title: Writeups by author
show_in_nav: false
permalink: /writeups_author/
---

<style>
.main_handle, .alt_handle {
	-margin-right: 0.5rem;
}

.main_handle {
	font-size: 2em;
}
.main_handle + .alt_handle::before {
	display: inline-block;
	content: "AKA";
	font-size: 0.8em;
	margin: 0 0.5rem;
	font-weight: bold;
}

.alt_handle + .alt_handle::before {
	display: inline-block;
	content: ", ";
	margin-right: 0.3em;
}

.author ul h3 {
	display: inline-block;
	margin: unset;
}

</style>

# Writeups by author
<a href="../writeups/">Browse writeups by CTF</a>

{% assign authors = site.data.authors | sort: "name" %}
{% assign writeups = site.writeups | sort: "contest" %}

{% for author in authors %}

<div class='author'>

<span class='main_handle'> {{ author.handles | join: "</span><span class='alt_handle'>" }} </span><br>

<!--Writeups by {{ author.handles.first }}:-->
<ul>
{% for writeup in writeups %}
{% if writeup.authors contains author.handles.first %}
	<li><a href="{{site.baseurl}}{{writeup.url}}"><h3>{{ writeup.title }} - {{ writeup.contest }}</h3></a></li>
{% endif %}
{% endfor %}
</ul>

</div>

{% endfor %}
