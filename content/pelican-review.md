Title: Pelican Static Site Generator
Date: 2024-09-15 17:00

I'm migrating my blogs on various sites to github pages. The default static site generator on github pages is Jekyll, which is written in Ruby.  I tried it out briefly, but decided to go for a generator written in the more familiar Python, so I can hack it if necessary.  Thus Pelican.  Let's explore some of the features.

# Themes
I pretty quickly found a theme I like, and forked it to switch out the background image.  I pulled the forked theme into this repo with a git submodule.

# Python-Markdown
The markdown processor in Pelican is [Python-Markdown](https://python-markdown.github.io/).  It has several [officially supported extensions](https://python-markdown.github.io/extensions/#officially-supported-extensions0) and many [third-party extensions](https://github.com/Python-Markdown/markdown/wiki/Third-Party-Extensions).

## Table of Contents
[TOC]

This is one of the built-in extensions.  I'd like the permalink to be more subtle, but probably that can be achieved with some styling work.  The subheadings should be indented.

## Code Blocks
```python
print("Hello Python")
```

Another built-in extension.  Not bad - I wish it had a copy button though.

## Mermaid Diagrams
Support added through the third-party [markdown-mermaidjs](https://github.com/Lee-W/markdown-mermaidjs) python-markdown extension.

```mermaid
  graph TD;
      A-->B;
      A-->C;
      B-->D;
      C-->D;
```

## Inline Image
![Alt Text]({static}/images/bluebonnet-7837830_300.jpg "Bluebonnet")

Image by <a href="https://pixabay.com/users/ray_shrewsberry-7673058/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=7837830">Ray Shrewsberry •</a> from <a href="https://pixabay.com//?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=7837830">Pixabay</a>

## Table
First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell

It rendered to html correctly, but need to add some style to this table.