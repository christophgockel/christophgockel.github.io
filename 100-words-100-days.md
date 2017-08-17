---
layout: default
collection: hundredwords
---

{% for post in site.hundredwords %}
  <article {% if post.title contains "Challenge" %} class="challenge" {% endif %}>
    {% if post.title contains "Challenge" %}
      <header>
        <h1>{{ post.title }}</h1>
      </header>
      <em class="center">{{ post.content }}</em>
    {% else %}
      <header>
        <h1>{{ post.title }}</h1>
        <p>{{ post.date | date_to_string }}</p>
      </header>
      {{ post.content }}
    {% endif %}
  </article>
{% endfor %}

