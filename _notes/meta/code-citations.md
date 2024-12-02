---
title: Code Citations
tags: meta
status: none
logs: false
published: true
---

## License: unknown
https://github.com/aloverso/anneloverso.com/tree/cad8a536545225beb3b2044babfb0db4761cc104/_plugins/bidirectional_links_generator.rb

```
= []
    graph_edges = []

    all_notes = site.collections['notes'].docs
    all_pages = site.pages

    all_docs = all_notes + all_pages

    link_extension = !!site.config["use_html_extension"] ? '.html' : ''

    # Convert all Wiki/Roam
```


## License: unknown
https://github.com/choisohan/Digital-Garden/tree/6f0f8e530cf681317c67da99eda024f7f911f0be/_plugins/bidirectional_links_generator.rb

```
BidirectionalLinksGenerator < Jekyll::Generator
  def generate(site)
    graph_nodes = []
    graph_edges = []

    all_notes = site.collections['notes'].docs
    all_pages = site.pages

    all_docs = all_notes + all_pages

    link_extension = !!site.config["use_html_extension"] ? '
```


## License: unknown
https://github.com/triger99/yoon_blog/tree/09a7910c52d7b74785b5292c57e3b8cf5751131e/_plugins/bidirectional_links_generator.rb

```
:Generator
  def generate(site)
    graph_nodes = []
    graph_edges = []

    all_notes = site.collections['notes'].docs
    all_pages = site.pages

    all_docs = all_notes + all_pages

    link_extension = !!site.config["use_html_extension"] ? '.html' :
```


## License: unknown
https://github.com/ryan-p-randall/ryan-p-randall.github.io/tree/1632e75f411298adabef786e129975c351745302/_plugins/bidirectional_links_generator.rb

```
(
            note_potentially_linked_to.basename,
            File.extname(note_potentially_linked_to.basename)
          )
        ).gsub('\_', '[ _]').gsub('\-', '[ -]').capitalize

        title_from_data = note_potentially_linked_to.data['title']
```


## License: MIT
https://github.com/HeySora/garden.heysora.net/tree/49db8aeca9b09a24404430519f8aaf4d9fe950cb/_plugins/bidirectional_links_generator.rb

```
).gsub('\_', '[ _]').gsub('\-', '[ -]').capitalize

        title_from_data = note_potentially_linked_to.data['title']
        if title_from_data
          title_from_data = Regexp.escape(title_from_data)
        end

        short_title_from_data = note_potentially_linked_to
```

