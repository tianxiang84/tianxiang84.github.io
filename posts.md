---
layout: page
comments: true
---

<div class="posts">
  {% for post in site.posts %}
    <article class="post">

      <h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1>
      
      <div class="date">
         Written on {{ post.date | date: "%B %e, %Y" }}
      </div>

      {% if post.content contains '<!--more-->' %}
        <div>
          {{ post.content | split:'<!--more-->' | first }}
        </div>
        <input type="checkbox" class="read-more-state" id="{{ site.baseurl }}{{ post.url }}"/>
        <div class="read-more-1">
          {{ post.content | split:'<!--more-->' | last }}
        </div>
        <label for="{{ site.baseurl }}{{ post.url }}" class="read-more-trigger"></label>
      {% else %}
        {{ post.content }}
      {% endif %}

    </article>
  {% endfor %}
</div>