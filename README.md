# Class Query

## A simple method to help manage responsive content *(and an alternative to element queries as an added bonus)*

With all of the responsive web sites I've built, one particular issue arises after it's handed over to the client to manage: how does their content editor manage elements/modules that need to respond at arbitrary breakpoints?

###Take this simple example:

1. The client adds a table of data with 5 narrow columns to a page. At 520px, the columns don't fit, so need to be linearised so the cells stack vertically.
2. On another page, the client adds a table with 11 columns. At 850px, they wish to linearise the rows in to groups of 3 columns, and at 480px linearise them fully.
3. On the final page, they add another table, again with 5 columns, but this time they're wider and no longer fit at 610px.

###How can we, or the client manage this?

We can't set a global rule for tables to linearise at an arbitrary breakpoint as the example above demonstrates that it clearly won't suit all scenarios.

The [BBC has blogged about this exact issue](http://responsivenews.co.uk/post/52382349921/tables), which finishes with this excellent quote:

> "Making responsive web design work with content that comes from a CMS is hard. Making small sites is simpler, as you are able to build custom solutions for data in a limited set of pages. But with a CMS you need to apply patterns that any data can fit into."

> "Because of this, CMS's will need to improve previewing for specific content types, and maybe give the user different layout options for them. As a technologist the worst thing you could do in this situation is to limit the type of data that the user can make. If the technology is dictating the content, then you’re doing it wrong."

###Sounds like a job for CSS classes!

Classes were designed for this kind of thing, right? For the above example, we could create two classes: `.linearise-table-all` and `.linearise-table-3col`. Perfect! But media queries **can't** apply classes to elements. Damn!

Element queries seem like another possible solution, but they don't exist either - and their performance and complexity issues may mean they never will.

###Enter Class Query!

####Here's how it works:

1. Instead of writing media queries for elements in CSS, you write the styles as a generic class, prefixed with `.classquery-`
2. In your markup, you add a simple media query-like syntax in a `[data-classquery]` attribute for when you want it to apply, such as `[data-classquery="(max-width: 520px), .classquery-linearise-table-all"]`, which would appy the declarations of the `.classquery-linearise-table-all` class at the `@media (max-width: 520px)` breakpoint for that element.
3. Include the tiny (~1 KB minified & gzipped) vanilla JavaScript class.query.js before the closing `</body>`.
4. Voila! The script will loop through all of your `[data-classquery]` elements, give them a unique `[data-classquery-id]` and write the relevant media queries in a new `<style>` block in the head. If there aren't any `[data-classquery]` elements on the page, then it does nothing.

If your elements already have IDs and classes applied, these will be added to the new media query selector to combat any specificity issues.

It will also add a class of `.classquery-init` to `<html>` when it starts, and replace it with `.classquery-complete` at the end, should you need finer control of the elements' display while they're being made responsive.

####What about browsers that don't support media queries?

Just add an additional selector to your `.classquery-` rule targeting them with a global attribute selector, similar to:

```
.classquery-breakpoint-modifier, .ltie9 [data-classquery*=".classquery-breakpoint-modifier"] { }
```

Incidentally, [Suzi](https://github.com/izilla/Suzi/), my Sass UI Framework has a [`classquery` mixin](https://github.com/izilla/Suzi/#class-mixins) that automatically generates both the standard and `.ltie9` selectors.

###That's great, but what about content editors?

This plugin just handles the front-end side of things. If you want content editors to use it, you'll need to provide a way for them to apply the `[data-classquery]` attributes in your preferred the CMS.

I've written a custom dialog for [Telerik's RadEditor](http://www.telerik.com/products/aspnet-ajax/editor.aspx), but that's pretty well integrated in to our CMS, [Cognition](http://www.cognitionecm.com).

I'm also in the process of writing support as a custom plugin for [CK Editor](http://ckeditor.com), as this handles inline editing, which will make managing the responsive content even easier.

###You mentioned Element Queries…

Media queries are great, but they're not portable. Developer A can't create a responsive module and have it integrated flawlessly in Developer B's project as the "framework" of the site/app will no doubt be different – meaning that the media queries will fire at incorrect sizes.

Element queries are the solution to this, but as I mentioned earlier, they don't exist… The best "polyfill" I've found is [Tyson Matanich's elementQuery](https://github.com/tysonmatanich/elementQuery), however, this has performance issues:

1. It requires [jQuery](http://jquery.com) or [Sizzle](http://sizzlejs.com).
2. An onresize event listener is added to every elementQuery element, which in turn alters the DOM.

Class Query is different in that, apart from the initial looping and constructing of the media queries, the browser handles all responsiveness natively meaning there is no additional performance overhead.

If you created a responsive module with Class Query **and** named your classes sensibly, it's instantly apparent what will happen at a particular breakpoint, and how it could be changed to suit an individual project.

###Are there any downsides to Class Query?

1. We're requiring JavaScript to implement responsive design. However, consider it as progressive enhancement. If you build "mobile-first" then your small screen users will get the elements' default layout regardless of whether they have JavaScript enabled.

2. Coupling media queries and markup. Re-styling/re-building a site would also require changing the `[data-classquery]` values in content pages. While some may see this as an issue, we're already doing this for responsive `<audio>` and `<video>`, as well as any [PictureFill](https://github.com/scottjehl/picturefill) "picture" elements.

3. It doesn't support `@import` CSS - but then, [you shouldn't be using `@import` anyway](http://www.stevesouders.com/blog/2009/04/09/dont-use-import/)…

With that said, I wouldn't use Class Query to replace standard "layout" media queries. It should only be used for responsive content which can't be styled generically or prototyping with element queries.

---

### OK, I'm convinced; show me!

#### Here's a basic example, which would change a `<div>`'s `background-colour` from red to green at 460px

##### HTML:

```
<div data-classquery="(min-width: 460px), .classquery-w460"></div>
```

##### CSS:

```
div {
    background-color: red;
    height: 100px;
}

.classquery-w460, .ltie9 [data-classquery*=".classquery-w460"] {
    background-color: green;
}
```

#### And here's an example with multiple media queries:

##### HTML:

```
<div data-classquery="(min-width: 460px), .classquery-w460; (min-width: 600px), .classquery-w600"></div>
```

##### CSS:

```
div {
    background-color: red;
    height: 100px;
}

.classquery-w460, .ltie9 [data-classquery*=".classquery-w460"] {
    background-color: green;
}

.classquery-w600, .ltie9 [data-classquery*=".classquery-w600"] {
    background-color: blue;
}
```

#### And one more using media query operators and existing IDs and classes:

##### HTML:

```
<div id="foo" class="bar baz" data-classquery="(min-width: 460px) and (max-width: 550px), .classquery-w460_550"></div>
```

##### CSS:

```
div {
    background-color: red;
    height: 100px;
}

.classquery-w460_550, .ltie9 [data-classquery*=".classquery-w460_550"] {
    background-color: orange;
}
```

#### And the all-important live demo:

**http://stowball.github.io/Class-Query/**

---

Copyright (c) 2013 Matt Stow    
Licensed under the MIT license (see LICENSE for details)
Minified version created with Online YUI Compressor: http://www.refresh-sf.com/yui/