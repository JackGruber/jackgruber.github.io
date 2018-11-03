---
layout: default
title: "Search site"
css: "/css/search.css"
---

<!-- HTML elements for search -->
<input type="text" id="search-input" placeholder="Search blog posts..">
<ul id="results-container"></ul>

<!-- script pointing to jekyll-search.js -->
<script src="{{ site.baseurl }}/simple-jekyll-search.js"></script>

<script>
SimpleJekyllSearch({
  searchInput: document.getElementById('search-input'),
  resultsContainer: document.getElementById('results-container'),
  json: '/search.json',
  searchResultTemplate: '<li><a href="{{ site.url }}{url}">{title}</a></br>{info}</br>{preview}</br></br></li>',
  noResultsText: 'Sorry no search result',
  limit: 10,
  fuzzy: false
})
</script>

