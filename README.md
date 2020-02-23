# lazy-style

To keep CSS to a minimum it would be ideal to only load CSS that matches the current media query.  The browser will download all of them that are linked to the page and then look at the media query to choose which to apply.  Therefore it is loading the stylesheets which are not yet being used and in many cases would never be used.

Lazy-style is a simple technique to only load stylesheets that match the media query.  This is done by writing the `<link>` tag using data attributes and a small JavaScript block to register those tags and setup an event listener to link them only when their media query is matched.

## The link tags
```html
<head>
    <link rel="stylesheet" href="core.css">
    <link rel="stylesheet" data-href="medium.css" data-media="screen and (min-width: 768px) and (max-width: 1280px)">
    <link rel="stylesheet" data-href="small.css" data-media="screen and (max-width: 767px)">
</head>
```
The first link tag will be downloaded and applied.  The second and third tags do not have a proper href, therefore will not download any content.  Instead, they have data-href and data-media attributes for use by the JavaScript

## the script
```javascript
(function(){
  var links = [].slice.call(document.querySelectorAll('link[data-media]')),
    lazy = links.reduce((sum, link) =>{
        var query = window.matchMedia(link.dataset.media);
        link.dataset.media = query.media; // creating the object might reorder the query elements but not change the meaning
        if (query.matches) {
            mediaMatched(query); // eager load
        } else {
            query.addListener(mediaMatched); // attach listener to lazy load if query ever matched
            sum.push(query); // keep reference
        }
        return sum;
    },[]);

  function mediaMatched(query) {
    var link = links.find((link) => link.dataset.media === query.media);
    if (link) {
        link.setAttribute('href', link.dataset.href);
        link.setAttribute('media', link.dataset.media);
        link.removeAttribute('data-href');
        link.removeAttribute('data-media');
    }
    query.target && query.target.removeListener(mediaMatched);
  }
}());
```
The self-involking function does the following
1. Make an array of a <link> tags with an attribute of data-media (aka lazy-styles)
2. Use reduce() to loop through the lazy-style <link> tags and check if their media query is currently met
  a. if yes then call internal function to convert lazy-style <link> tag to a regular <link> tag and therefore causing it to download
  b. if not yet then add a listener for it to become matched, registering the <link> rewrite function as the handler
  
The mediaMatched() handler re-writes the data attributes to proper href and media attributes causing immediate download and native CSS handling for when to apply those styles.
