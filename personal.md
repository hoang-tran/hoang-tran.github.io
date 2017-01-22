---
layout: default
---
{% assign personal_posts = site.tags.personal %}
{% assign posts_count = (personal_posts | size) %}

<div class="home">
  {% if posts_count > 0 %}
    <div class="posts">
      {% for post in personal_posts %}
        <div class="post py3">
          <p class="post-meta">{{ post.date | date: site.date_format }}</p>
          <a href="{{ post.url | prepend: site.baseurl }}" class="post-link"><h3 class="h1 post-title">{{ post.title }}</h3></a>
          <p class="post-summary">
            {% if post.summary %}
              {{ post.summary }}
            {% elsif post.meta_description %}
              {{ post.meta_description }}
            {% else %}
              {{ post.excerpt }}
            {% endif %}
          </p>
        </div>
      {% endfor %}
    </div>
  {% else %}
    <h1 class='center'>{{ site.text.index.coming_soon }}</h1>
  {% endif %}
</div>
