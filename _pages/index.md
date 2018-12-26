---
layout: defaults/page
permalink: index.html
narrow: true
---

{% include components/intro.md %}

## Recent Posts

{% for post in site.posts limit:3 %}
{% include components/post-card.html %}
{% endfor %}

## All Posts

See all posts here! [all posts by year.]({{ site.baseurl }}{% link list/posts.html %})
