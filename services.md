---
layout: default
title: Services
permalink: /services
---

<div class="posts">
  <h1 class="page-heading">{{page.title}}</h1>
  <ul class="posts">
  	{% for post in site.categories.services %}
  	  	<li class="post">
 				<span class="special-colour">>> </span>
   				<a href="{{ site.baseurl }}{{ post.url }}" class="post-entry">{{ post.title }} </a>
    			<a class="post-date"> ({{ post.date | date: "%B %e, %Y" }}) </a>
    	</li>
  {% endfor %}
	</ul>
</div>
