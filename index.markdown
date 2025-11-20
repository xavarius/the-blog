---
layout: default
title: Home
---

<div class="home">
  {% assign categories = "Tech,Android,Routes,Poetry" | split: "," %}
  
  {% for category in categories %}
    {% assign category_posts = site.posts | where: "category", category %}
    {% if category_posts.size > 0 %}
      <section class="category-section">
        <h2 class="category-header">{{ category }}</h2>
        <div class="post-list">
          {% for post in category_posts %}
            <div class="post-item">
              <a href="{{ post.url | relative_url }}" class="post-link">
                {{ post.title }}
              </a>
              <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
            </div>
          {% endfor %}
        </div>
      </section>
    {% endif %}
  {% endfor %}
</div>
