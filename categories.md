---
layout: category
permalink: /categories/
---
<div class="home">
   <ul class="post-list">
{% for category in site.categories-list %}
{% if site.categories[category] != null %}
  <h3 class="category-title">
    //{{ category }}
  </h3>
{% endif %}
<br />
  {% for post in site.categories[category] %}
  	<h3 class="category-link">
  	<a class="post-link" href="{{ post.url | prepend: site.baseurl }}">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{{ post.title }}</a>
	</h3>
  {% endfor %}
{% endfor %}
   </ul>
</div> 
