---
layout: post
title:  "Modify Bootstrap's default grid system padding"
date:   2015-09-28 09:00:01
categories: bootstrap3
---

Using Bootstrap column grid is extremely helpful, however they come with a default `30px` margin between each column. Although this padding can be modified by changing the variable `@grid-gutter-width` sometimes its useful to be able to use different values of this padding.

That's what this mixin does, allows the creation of a class to modify the padding within columns inside a row.

```css
.row-padding(@padding) {
    margin-left: -@padding/2;
    margin-right: -@padding/2;

	.col-xs-1, .col-sm-1, .col-md-1, .col-lg-1, 
	.col-xs-2, .col-sm-2, .col-md-2, .col-lg-2, 
	.col-xs-3, .col-sm-3, .col-md-3, .col-lg-3, 
	.col-xs-4, .col-sm-4, .col-md-4, .col-lg-4, 
	.col-xs-5, .col-sm-5, .col-md-5, .col-lg-5, 
	.col-xs-6, .col-sm-6, .col-md-6, .col-lg-6, 
	.col-xs-7, .col-sm-7, .col-md-7, .col-lg-7, 
	.col-xs-8, .col-sm-8, .col-md-8, .col-lg-8, 
	.col-xs-9, .col-sm-9, .col-md-9, .col-lg-9, 
	.col-xs-10, .col-sm-10, .col-md-10, .col-lg-10, 
	.col-xs-11, .col-sm-11, .col-md-11, .col-lg-11, 
	.col-xs-12, .col-sm-12, .col-md-12, .col-lg-12 {
	    padding-left: @padding/2;
	    padding-right: @padding/2;
	}
}
```

Using it is pretty straightforward, just declare the class name and include the mixin.

```css
.row--xs {
	.row-padding(10px);
}
```

And then on the html, add it as usual

```html
<div class="row row--xs">
	<div class="col-sm-4"></div>
	<div class="col-sm-4"></div>
	<div class="col-sm-4"></div>
</div>
```
