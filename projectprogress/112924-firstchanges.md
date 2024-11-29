# November 29, 2024

## Small changes

- Changed footer
- Changed README
- Assigned license
- Built website

//TODO

### TO DO LIST

- [ ] Change license in Github mismo
- [ ] Change assets
- [ ] Write Now page content
- [ ] Build Now archive to be accessed later
- [ ] Address errors
- [ ] Create page with all notes that has filter

I want the filter to have the following
- Recently updated posts / posts that were updated long ago
- Status
-   Resonates
-   Discussing
-   Applicable
-   Archived
- Notes that have logs / notes don't have logs

## Each note has a blog of its own

1. Created a `_layouts/note_with_blog.html`

```
---
layout: note
---

<article>
  <div>
    <h1>{{ page.title }}</h1>
    {% if page.subtitle %}
      <h3 style="margin-top: 0.5rem;">{{ page.subtitle }}</h3>
    {% endif %}
    <time datetime="{{ page.last_modified_at | date_to_xmlschema }}">
      {% if page.type != 'pages' %}
        Last updated on {{ page.last_modified_at | date: "%B %-d, %Y" }}

        {% if page.tags.size > 0 %}
          <br />
          <small>
            Tags:
            {% for tag in page.tags %}
              <span class="tag">{{ tag | strip }}</span>
            {% endfor %}
          </small>
        {% endif %}
      {% endif %}
    </time>
  </div>

  <div id="notes-entry-container">
    <content>
      {{ content }}
    </content>

    <side style="font-size: 0.9em">
      <h3 style="margin-bottom: 1em">Notes mentioning this note</h3>
      {% if page.backlinks.size > 0 %}
        <div style="display: grid; grid-gap: 1em; grid-template-columns: repeat(1fr);">
        {% for backlink in page.backlinks %}
          <div class="backlink-box">
          <a class="internal-link" href="{{ site.baseurl }}{{ backlink.url }}{%- if site.use_html_extension -%}.html{%- endif -%}">{{ backlink.title }}</a><br>
          <div style="font-size: 0.8em; word-break: break-word">{{ backlink.content | strip_html | truncatewords: 20 }}</div>
          </div>
        {% endfor %}
        </div>
      {% else %}
      <div style="font-size: 0.9em">
        <p>
          There are no notes linking to this note.
        </p>
      </div>
      {% endif %}
    </side>
  </div>

  {% if page.blog_posts %}
  <div id="blog-posts-container">
    <h2>Related Blog Posts</h2>
    <ul>
      {% for post in page.blog_posts %}
        <li>
          <a href="{{ post.url }}">{{ post.title }}</a>
          <p>{{ post.excerpt }}</p>
        </li>
      {% endfor %}
    </ul>
  </div>
  {% endif %}
</article>
```

2. Modified the note layout to include a section for blog posts if they exist.

```
{% if page.blog_posts %}
<div id="blog-posts-container">
  <h2>Related Blog Posts</h2>
  <ul>
    {% for post in page.blog_posts %}
      <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        <p>{{ post.excerpt }}</p>
      </li>
    {% endfor %}
  </ul>
</div>
{% endif %}
```

3. Add blog posts to notes
```
---
title: My Note
layout: note_with_blog
blog_posts:
  - title: "First Blog Post"
    url: "/blog/first-post"
    excerpt: "This is the first blog post."
  - title: "Second Blog Post"
    url: "/blog/second-post"
    excerpt: "This is the second blog post."
---
```

## Excluding blog posts from graph generation

I modified the `bidirectional_links_generator.rb` plugin to filter out the blog posts when generating the graph nodes and edges.

I put the blog posts in the `_posts` directory so I can filter them based on the path. 

Here is the modified code:
```
# frozen_string_literal: true
class BidirectionalLinksGenerator < Jekyll::Generator
  def generate(site)
    graph_nodes = []
    graph_edges = []

    all_notes = site.collections['notes'].docs
    all_pages = site.pages

    all_docs = all_notes + all_pages

    link_extension = !!site.config["use_html_extension"] ? '.html' : ''

    # Convert all Wiki/Roam-style double-bracket link syntax to plain HTML
    # anchor tag elements (<a>) with "internal-link" CSS class
    all_docs.each do |current_note|
      next if current_note.path.include?('_posts') # Exclude blog posts

      all_docs.each do |note_potentially_linked_to|
        next if note_potentially_linked_to.path.include?('_posts') # Exclude blog posts

        note_title_regexp_pattern = Regexp.escape(
          File.basename(
            note_potentially_linked_to.basename,
            File.extname(note_potentially_linked_to.basename)
          )
        ).gsub('\_', '[ _]').gsub('\-', '[ -]').capitalize

        title_from_data = note_potentially_linked_to.data['title']
        if title_from_data
          title_from_data = Regexp.escape(title_from_data)
        end

        short_title_from_data = note_potentially_linked_to.data['short-title']
        if short_title_from_data
          short_title_from_data = Regexp.escape(short_title_from_data)
        end

        new_href = "#{site.baseurl}#{note_potentially_linked_to.url}#{link_extension}"
        anchor_tag = "<a class='internal-link' href='#{new_href}'>\\1</a>"

        # Replace double-bracketed links with label using note title
        current_note.content.gsub!(
          /\[\[#{title_from_data}\|(.+?)(?=\])\]\]/i,
          anchor_tag
        )

        if note_potentially_linked_to.data['short-title']
          # Replace double-bracketed links with label using note short-title
          current_note.content.gsub!(
            /\[\[#{short_title_from_data}\|(.+?)(?=\])\]\]/i,
            anchor_tag
          )
        end

        # Replace double-bracketed links using note filename
        current_note.content.gsub!(
          /\[\[#{note_title_regexp_pattern}\|(.+?)(?=\])\]\]/i,
          anchor_tag
        )

        # Replace double-bracketed links using note title
        current_note.content.gsub!(
          /\[\[(#{title_from_data})\]\]/i,
          anchor_tag
        )

        if note_potentially_linked_to.data['short-title']
          # Replace double-bracketed links using note short-title
          current_note.content.gsub!(
            /\[\[(#{short_title_from_data})\]\]/i,
            anchor_tag
          )
        end
      end

      # Identify note backlinks and add them to each note
      notes_linking_to_current_note = all_notes.filter do |e|
        e.data['title'] != current_note.data['title'] &&
        (e.content.include?("'" + current_note.url) || e.content.include?("\"" + current_note.url))
      end

      # Nodes: Graph
      graph_nodes << {
        id: note_id_from_note(current_note),
        path: "#{site.baseurl}#{current_note.url}#{link_extension}",
        label: current_note.data['short-title'] || current_note.data['title'],
      } unless current_note.path.include?('_notes/index.html')

      # Edges: Graph
      notes_linking_to_current_note.each do |n|
        graph_edges << {
          source: note_id_from_note(n),
          target: note_id_from_note(current_note),
        }
      end
    end

    File.write('_includes/notes_graph.json', JSON.dump({
      edges: graph_edges,
      nodes: graph_nodes,
    }))
  end

  def note_id_from_note(note)
    note.data['title'].bytes.join
  end
end
```

In this updated code, the `next if current_note.path.include?('_posts')` line ensures that any document within the `_posts` directory is excluded from the graph generation.

## Generate page where all notes are listed and can be filtered according to criteria

The following criteria are:
- Recently updated or were updated long ago
- Status
    - Resonates
    - Discussing
    - Applicable
    - Archived
- Notes that have logs or don't have logs

1. Create a new page template

Can be found on `_pages/all_notes.md`. Here is the code:

```
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
```

2. Ensure that each note has the necessary front matter fields for status and logs
```
---
title: My Note
status: resonates
logs: true
last_modified_at: 2023-10-01
---
```


