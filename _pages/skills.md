---
title: Skills
permalink: /skills/
---
<p>
{% for skill in site.data.skills %}
    <span class="btn btn--info btn--small">{{ skill }}</span>
{% endfor %}
</p>
