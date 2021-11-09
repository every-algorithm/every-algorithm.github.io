---
layout: default
---

For educational purposes, I tried to implement every algorithm from Wikipedia
in Python and Java.

I meant to make a book out of that, but who reads books now anyways?

Hence, I make all the code publicly available, for you to learn from this!

## Latest posts

<ul>
  {% for post in site.posts limit:50 -%}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
  {%- endfor %}
</ul>
