---
layout: default
---

{% for post in site.hundredwords %}
  <article>
    <header>
      <h1>{{ post.title }}</h1>
      <p>{{ post.date | date_to_string }}</p>
    </header>

    {{ post.content }}
  </article>
{% endfor %}
