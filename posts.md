---
layout: default
title: Posts
permalink: /posts/
---

<div class="home">
   <ul class="post-list">
{% for post in site.posts %}
       <li>
         <h2>
           <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
         </h2>
	  <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
       </li>
   <div class="post-content" itemprop="articleBody">
     {{ post.excerpt }}
   </div>
{% endfor %}
   </ul>
   <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>
{{ paginator.total_pages }}
{{ paginator.total_posts }}
{{ paginator.previous_page_path }}
{{ paginator.page }}
</div> 
