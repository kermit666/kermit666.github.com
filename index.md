---
layout: page
title: kermit666's GitHub pages
tagline: kre kre...
---
{% include JB/setup %}

Welcome to my GitHub pages. My primary blog can be found 
at [kermit.epska.org](http://kermit.epksa.org). This site 
contains some of my notes which are easier to maintain 
in Markdown.

## News

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Resources

This site is powered by Jekyll Bootstrap

[quick_start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html) [docs](http://jekyllbootstrap.com) [code](http://github.com/plusjade/jekyll-bootstrap)

The contents of this site are available under the [Creative Commons Attribution-NonCommercial-ShareAlike 3.0 license](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.en_US). Help yourself to the Markdown [source code](https://github.com/kermit666/kermit666.github.com).

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/deed.en_US"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-nc-sa/3.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">kermit666's GitHub page</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="kermit666.github.com" property="cc:attributionName" rel="cc:attributionURL">Dražen Lučanin</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/deed.en_US">Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License</a>.
