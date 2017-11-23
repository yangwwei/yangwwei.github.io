---
layout: defaults/page
permalink: index.html
narrow: true
---

{% include components/intro.md %} More about the site author [Weiwei]({{ site.baseurl}}{% link _pages/about.md %}). I worked on large scale computation and storage systems for manay years. [See my experiences here.]({{ site.baseurl }}{% link list/portfolio.html %})

## Recent Posts

{% for post in site.posts limit:3 %}
{% include components/post-card.html %}
{% endfor %}

## All Posts

See all posts here! [all posts by year.]({{ site.baseurl }}{% link list/posts.html %})
