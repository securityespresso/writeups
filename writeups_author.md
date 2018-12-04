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
Browse writeups <a href="../writeups/">by CTF</a> / <a href="../writeups_latest/">latest</a>

{% assign authors = site.data.authors | sort: "handles" %}
{% assign writeups = site.writeups | sort: "contest" %}

{% for author in authors %}

<div class='author'>

{%- for handle in author.handles -%}
{%- if handle == author.handles[0] -%}
<span class='main_handle'>{{ handle | escape }}</span>
{%- else -%}
<span class='alt_handle'>{{ handle | escape }}</span>
{%- endif -%}
{%- endfor -%}

{%- if author.email -%}
<span class='contact'> - <a href='mailto:{{ author.email }}'>{{ author.email | escape }}</a></span>
{%- endif -%}
<br>

<ul>
{%- for writeup in writeups -%}
{%- if writeup.authors contains author.handles.first -%}
	<li><a href="{{site.baseurl}}{{writeup.url}}"><h3>{{ writeup.title | escape }} - {{ writeup.contest | escape }}</h3></a></li>
{%- endif -%}
{%- endfor -%}
</ul>

</div>

{% endfor %}
