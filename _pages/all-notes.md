---
layout: archive
title: All Notes
permalink: /all-notes/
---

<ul id="notes-list">
  {% assign notes = site.notes | sort: "last_modified_at" | reverse %}
  {% for note in notes %}
    <li class="note-item" data-updated="{{ note.last_modified_at | date: '%Y-%m-%d' }}" data-status="{{ note.status }}" data-logs="{% if note.logs %}has-logs{% else %}no-logs{% endif %}">
      <a href="{{ note.url }}">{{ note.title }}</a>
      <p>Last updated: {{ note.last_modified_at | date: '%B %-d, %Y' }}</p>
      <p>Status: {{ note.status }}</p>
      <p>Logs: {% if note.logs %}Yes{% else %}No{% endif %}</p>
    </li>
  {% endfor %}
</ul>