---
layout: page
title: All Notes
permalink: /all-notes/
---

<h1>All Notes</h1>

<div>
  <label for="filter-updated">Filter by update time:</label>
  <select id="filter-updated" onchange="filterNotes()">
    <option value="all">All</option>
    <option value="recent">Recently Updated</option>
    <option value="old">Updated Long Ago</option>
  </select>

  <label for="filter-status">Filter by status:</label>
  <select id="filter-status" onchange="filterNotes()">
    <option value="all">All</option>
    <option value="resonates">Resonates</option>
    <option value="discussing">Discussing</option>
    <option value="applicable">Applicable</option>
    <option value="archived">Archived</option>
  </select>

  <label for="filter-logs">Filter by logs:</label>
  <select id="filter-logs" onchange="filterNotes()">
    <option value="all">All</option>
    <option value="has-logs">Has Logs</option>
    <option value="no-logs">No Logs</option>
  </select>
</div>

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

<script>
  function filterNotes() {
    const updatedFilter = document.getElementById('filter-updated').value;
    const statusFilter = document.getElementById('filter-status').value;
    const logsFilter = document.getElementById('filter-logs').value;

    const notes = document.querySelectorAll('.note-item');

    notes.forEach(note => {
      const updated = note.getAttribute('data-updated');
      const status = note.getAttribute('data-status');
      const logs = note.getAttribute('data-logs');

      let show = true;

      if (updatedFilter === 'recent' && new Date(updated) < new Date(new Date().setDate(new Date().getDate() - 30))) {
        show = false;
      } else if (updatedFilter === 'old' && new Date(updated) >= new Date(new Date().setDate(new Date().getDate() - 30))) {
        show = false;
      }

      if (statusFilter !== 'all' && status !== statusFilter) {
        show = false;
      }

      if (logsFilter !== 'all' && logs !== logsFilter) {
        show = false;
      }

      note.style.display = show ? '' : 'none';
    });
  }
</script>