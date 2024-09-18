Title: Pelican Static Site Generator
Date: 2024-09-15 17:00

I'm migrating my blogs on various sites to github pages.  
The default static site generator on github pages is Jekyll, which is written in Ruby.  
I tried it out briefly, but decided to go for a generator written in the more familiar Python, so I can hack it if necessary.  
Thus Pelican.  So far I like it.  I pretty quickly found a theme I like, and forked it to switch out the background image.  I pulled the forked theme into this repo with a git submodule.

Two things I want to make sure will work with Pelican are formatted code blocks and mermaid diagrams.  Lets start with a code block.

```python
print("Hello Python")
```

Not bad - I wish it had a copy button though.

Ok how about mermaid?  Yes, it works with [this](https://github.com/Lee-W/markdown-mermaidjs) python-markdown extension.

```mermaid
  graph TD;
      A-->B;
      A-->C;
      B-->D;
      C-->D;
```