---
layout: defaults/page
permalink: index.html
narrow: false
---

{% for post in site.posts limit:5 %}
{% include components/post-card.html %}
{% endfor %}

---

More posts [here]({{ site.baseurl }}{% link list/posts.html %}) ...
