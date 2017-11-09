---
layout: post
---

##Design Overview*

Delete an object from ozone file system, and all data associated with this object will also be deleted. Ozone is a multi-layer distributed system that includes some primitive services, each layer needs to maintain a consistent view of under deletion keys.
Once a client calls KSM API to delete a key, the key is transited to `deleting`[1] state that indicating this key is currently under deletion. Once KSM is `acknowledged`[2] this state change, the key is removed from the namespace and becomes invisible from clients. That means it can no longer be query-able via any KSM API. However, at this point the actual data of this object is not purged yet, it happens in an `asynchronous`[3] manner. That means, each layer transited the key states relying on acknowledgements instead of actual deletion results.

Below is the overall architecture,

[design-architecture](http:)


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
