---
layout: default
---

# In Order To

<ul id="posts">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> <span>{{ post.date | date: '%B %Y' }}</span>
    </li>
  {% endfor %}
</ul>

<footer>
  &copy; 2016 Andrii Ponomarov.
</footer>
