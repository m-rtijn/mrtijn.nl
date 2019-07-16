---
layout: default.liquid

title: Martijn's blog
---

{% for post in collections.posts.pages %}
{{ post.data.published_date }}
[**{{ post.title }}**]({{ post.permalink }})
{% endfor %}
