---
layout: default.liquid

title: Martijn's blog
data:
    last_updated_date: 2020-07-30
---

{% for post in collections.posts.pages %}
{{ post.data.published_date }}
[**{{ post.title }}**]({{ post.permalink }})
{% endfor %}
