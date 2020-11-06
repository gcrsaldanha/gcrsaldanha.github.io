---
layout: post
title:  "Welcome!"
date:   2015-12-28 23:12:15 -0800
published: true
categories: jekyll cheatsheet
---
# Jekyll Cheatsheet/Guide


## Installation

```shell script
$ gem install jekyll
$ jekyll new myblog
$ cd myblog
$ jekyll serve
```


## Gems, Gemfile and Bundler

Later...
- https://jekyllrb.com/docs/ruby-101/


## Liquid

Liquid syntax includes:
- Objects: `{{ page.variable }}`
- Tags: `{% if %} ... {% endif %}`
- Filters: `{{ "gabriel" | capitalize }}`


## Front Matter

Add it to the top of every page where Liquid will be used.
```html
---
# This is called Front Matter
# snippet of YAML between two triple-dashed lines at the top of a file.
- variable: 5
---
```
Front Matter variables are available to Liquid via `page` object, e.g.: `{{ page.variable }}` would render `5`.


## Layouts

### `_layouts/default.html`
```html
<!doctype html>
<html>
    <title>{{ page.title }}</title>
  <body>
    {{ content }}  <!-- special variable -->
  </body>
</html>
```

`index.html`
```html
---
layout: default
title: Home  # This will replace {{ title }} in `_layouts/default.html`
---
<h1>This will replace {{ content }} in `_layouts/default.html`</h1>
```

`about.md`
```markdown
---
layout: default
title: About
---
~About page~

This page tells you a little bit about me.
```


## Includes

### `_includes/social_media.html`
```html
<ul>
  <li>Github</li>
  <li>Instagram</li>
  <li>LinkedIn</li>
</ul>
```

`index.html`
```html
<html>
  <footer>
    {% include social_media.html %}
  </footer>
</html>
```


## Data files

### `_data/social_media.yml`
```yaml
- media: GitHub
  link: /
- media: LinkedIn
  link: /
```

`index.html`
```html
{% for item in site.data.social_media %}
  <p>item.media</p>
  <p>item.link</p>
{% endfor %}
```


## Assets

### `assets/{css,images,js}`


## Posts

### `_posts/<YYYY>-<mm>-<dd>-[title].<ext>`
Variables available to `page` object:
- {{ page.title }}
- {{ page.date | date_to_string }}

Posts are available at `site.posts`


## Collections

Ref: https://jekyllrb.com/docs/step-by-step/09-collections/

In short, it's like `posts` but without the file name format.

Add to `_config.yml`:
```yaml
collections:
  mycollection:
```
Create folder: `_mycollection/`
Add files to it :)

Make each collection output a page:
```yaml
collections:
  mycollection:
    output: true
```

```html
{% for file in site.mycollection %}
  {{ file.url }} <!-- if output: true -->
  {{ file.custom_variable }}
  {{ file.content | markdownify }}  <!-- content is special variable, markdown filter -->
{% endfor %}
```
