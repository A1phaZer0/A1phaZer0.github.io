---
layout: category
permalink: /categories/
---
<div class="home">
   <ul class="post-list">
{% for category in site.categories-list %}
{{ category }}
<br />
  {% for post in site.categories[category] %}
  	<h3>
  	<a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
	</h3>
  {% endfor %}
{% endfor %}
   </ul>
</div> 
