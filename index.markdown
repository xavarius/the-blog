---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Home
---

<div class="home">
  <ul class="post-list">
    {% for post in site.posts %}
        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
        
        <div class="post-content-preview">
          {{ post.content }}
        </div>
        {%- unless forloop.last -%}
          <hr class="post-divider">
        {%- endunless -%}
    {% endfor %}
  </ul>
</div>

