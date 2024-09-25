Title: Pelican Static Site Generator
Date: 2024-09-21 17:00
Author: Ryan McKay
Tags: python, ssg, pelican
Status: published

I'm migrating my blogs on various sites (primarily [blogspot](https://againstentropy.blogspot.com/)) to github pages.  I like specifying the content in markdown and having revision control.

The default static site generator on github pages is Jekyll, which is written in Ruby.
I tried it out briefly, but decided to go for a generator written in the more familiar Python,
so I can hack it if necessary - thus [Pelican](https://docs.getpelican.com/en/latest/index.html).
Let's explore some of the features.
The source for this site is at [https://github.com/ryanmckaytx/ryanmckaytx.github.io](https://github.com/ryanmckaytx/ryanmckaytx.github.io)

# Themes
I pretty quickly found a theme I like, and [forked](https://github.com/ryanmckaytx/pelican-mediumfox)
it to switch out the background image, fix a bug, and make a few styling improvements. 
I pulled the forked theme into this repo with a git submodule.

# Github Pages
Pelican comes with a github action to publish your site to github pages.  But it uses pip and I like to use poetry, primarily for the integrated virtual env and lockfile management.  So I [tweaked](https://github.com/ryanmckaytx/ryanmckaytx.github.io/blob/main/.github/workflows/pelican_github_pages.yml) it a bit.

# Python-Markdown
The markdown processor in Pelican is [Python-Markdown](https://python-markdown.github.io/).
It has several [officially supported extensions](https://python-markdown.github.io/extensions/#officially-supported-extensions0) and many [third-party extensions](https://github.com/Python-Markdown/markdown/wiki/Third-Party-Extensions).

## Table of Contents
[TOC]

This is one of the built-in extensions.
It adds the permalink to each heading and creates the TOC component.
The permalink and TOC look a lot better after some styling.

## Code Blocks {: style="clear: both"}
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
![Bluebonet]({static}/images/bluebonnet-7837830_300.jpg "Bluebonnet")

Image by <a href="https://pixabay.com/users/ray_shrewsberry-7673058/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=7837830">Ray Shrewsberry •</a> from <a href="https://pixabay.com//?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=7837830">Pixabay</a>

## Table
First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell

It rendered to html correctly.  The template styling for tables needed some tweaking.

# Import from Blogspot
Pelican provides the [pelican-import](https://docs.getpelican.com/en/latest/importer.html) tool for importing content from various sources.
The exported content from blogspot was about 400KB of xml.  After running the tool, all of my blog entries became pages as expected, but there were some unexpected aspects:

* The comments from the original blog entries each became another page
* The page content had a lot of embedded html that didn't look good ([example](/drafts/docker-java-example-part-1-initializing-original-import))
* The images were not transferred

So there is a fair amount of cleanup/translation to markdown required.