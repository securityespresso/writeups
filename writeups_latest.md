---
layout: default
title: Latest writeups
permalink: /writeups_latest/
---

<style>
ul h3 {
	display: inline-block;
	margin: unset;
}
</style>

# Latest writeups
Browse writeups <a href="../writeups/">by CTF</a> / <a href="../writeups_author/">by author</a>

{%- assign writeups = site.writeups | sort: "last_modifed_at" | reverse -%}
{%- assign n = writeups | size | minus: 1 -%}
{%- for i in (0..n) -%}

{%- comment -%}
{%- are you fucking serious... -%}
{%- endcomment -%}
{%- assign j = i | minus: 1 -%}

{%- assign writeup = writeups[i] -%}
{%- assign prevwriteup = writeups[j] -%}

{%- comment -%}
{%- https://stackoverflow.com/a/18674804 sarut mana -%}
{%- endcomment -%}

{%- capture day -%}{{ writeup.last_modified_at | date: '%Y%m%d' }}{%- endcapture -%}
{%- capture pday -%}{{ prevwriteup.last_modified_at | date: '%Y%m%d' }}{%- endcapture -%}

{% if day != pday %}

## {{ writeup.last_modified_at | date: "%Y-%m-%d" }} 

{% endif %}

<h3><a href="{{site.baseurl}}{{writeup.url}}">{{ writeup.contest }} - {{ writeup.title }}</a></h3>

{%- endfor -%}
 