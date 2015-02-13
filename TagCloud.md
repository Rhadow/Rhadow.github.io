---
layout: page
title: Tag Cloud
---
<ul class="tag-cloud">
{% assign sorted_tags = (site.tags | sort: 0) %}
{% for tag in sorted_tags %}
  <li style="font-size: {{ tag | last | size | times: 100 | divided_by: site.tags.size | plus: 35  }}%">
    <a href="/tags/{{ tag[0] }}">
      {{ tag | first }} ({{ tag | last | size }})
    </a>
  </li>
{% endfor %}
</ul>