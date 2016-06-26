---
layout: category
permalink: /categories/
---
<div class="home">
   <ul class="post-list">
{% for category in site.categories-list %}
{% if site.categories[category] != null %}
  <h3 class="category-title">
    {{ category }}
  </h3>
{% endif %}
<br />
  {% for post in site.categories[category] %}
  	<h3>
  	<a class="post-link category-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
	</h3>
  {% endfor %}
{% endfor %}
   </ul>
</div> 
