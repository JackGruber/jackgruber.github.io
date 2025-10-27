---
layout: post
title: Embed Mermaid in Jekyll without plugin
subtitle:
show-img: true
tags: [Jekyll, Mermaid, Github, howto]
mermaid: true
cover-img: "/assets/img/head/code.jpg"
thumbnail-img: "/img/posts/2021-05-09/mermaid.jpg"
---

Mermaid is a simple markdown-like script language for generating charts from text via javascript.
I would like to show you how you can enable rendering of mermaid diagrams without plugin in your Jekyll blog/site which is useful for Github pages.

## Page Template

Create a file `mermaid.html` in `_include` directory with the following content:

```html
<script src="https://unpkg.com/mermaid@8.9.3/dist/mermaid.min.js"></script>
<script>
  $(document).ready(function () {
    mermaid.initialize({
      startOnLoad:true,
      theme: "default",
    });
    window.mermaid.init(undefined, document.querySelectorAll('.language-mermaid'));
  });
</script>
```

Add the following content to the end of file `footer-scripts.html` in directory `_include`.

{% raw %}
```html
{% if page.mermaid %} 
  {% include mermaid.html %} 
{% endif %}
```
{% endraw %}

## Enable mermaid rendering

With `mermaid: true` you can now enable the rendering of mermaid diagrams in your post or page.
This has the advantage that the relatively large `mermaid.min.js` is only loaded when it is needed.

## Writing a mermaid diagram

You can now write a mermaid diagram like this in your site or posts.

<pre>
```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```  
</pre> 

Which are rendered automatically

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
