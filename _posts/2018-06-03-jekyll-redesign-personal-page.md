---
title: "Jekyll: Re-designing my personal blog."
date: 2018-06-03 09:00:01 Z
categories:
- jekyll
- blog
layout: post
summary: Finally! I updated the blog. I talk about the entire process here.
---

It's been long overdue, but today I finally updated my blog! The previous version was up since 2015, I implemented it quickly after I graduated from my university and it was always my intention to get back and change it.

![Before and After image of the blog](https://user-images.githubusercontent.com/1882507/40889416-26671f78-6734-11e8-8303-3882e9a08120.png)

This is certainly a big improvement, and hopefully, this new design will live for several years. I like how simple it is, and yet still looks professional. This post describes the process I followed for the re-design, it's a bit long, if you want you can jump to either [Design](#design) or [Implementation](#implementation), depending on what interest you the most.

## Design

The first part was coming up with the design, it's always better to have some mockups before starting with the implementation. I decided early that I wanted two main templates: one for the landing page and another one the blog post page. This would ease the process of implementing everything, and since I don't have too many blog posts it doesn't really make sense to have a separate page with the list of articles.

To organize my thoughts, I wrote a few key points to consider while creating the design. In general:

- I wanted something very simple.
- Responsive.
- With a single column. I wasn't able to think of anything important enough to deserve a separate column.
- Top bar with the site name ("Ariel Rodriguez Romero's Blog" for now)

In the case of the Landing Page:

- Starts with a nice image.
- Include a short bio.
- Include links to social networks: Github, Twitter, Linkedin, etc.
- Include a list of articles, with summaries for each of them. In the future, if the list of articles grows it may make sense to bring back the `/articles` page.

And for the Article Page:

- Integrate well with markdown, especially for images and code. I don't want to deal with any formatting issues when writing new posts.
- [Disqus comments](https://disqus.com/home/forum/arielrodriguezromero/) at the bottom.

Then, I started brainstorming to create several proposals for the home page. I took some inspiration from several blogs, including [nathanbarry.com](http://nathanbarry.com/) and [brunomarinho.com](http://brunomarinho.com/) among others.

![landing page proposals](https://user-images.githubusercontent.com/1882507/36406297-d363e200-15c3-11e8-8ee0-ecbcfa886569.png)

Initially the 4th one seemed like the best option. But, a few weeks later I started playing with Sketch and created a Mid-Fi design that ended up being the one selected. It looks more like the 2nd one.

<img src="https://user-images.githubusercontent.com/1882507/40617879-aee70902-625d-11e8-80eb-7b06a84c780e.png" alt="Mid-fi design of arielrodriguezromero.com" style="max-height: 650px; margin-left: auto; margin-right: auto">

The image at the top was taken by [@isabeldiazalanis](https://www.instagram.com/isabeldiazalanis/), from our recent trip to [St. Augustine](https://en.wikipedia.org/wiki/St._Augustine,_Florida). They claim to be the oldest city in the US, although everything seemed fairly new.

## Implementation

I'm fond of the combination of [Jekyll and Github Pages](https://pages.github.com/), it's free hosting and the resulting site loads really fast since it's only static HTML. So, I didn't thought much about finding another technology for this.

I briefly considered using webpack and react, I used a similar approach for the [Sci-Hub app](https://greenelab.github.io/scihub/#/) ([code here](https://github.com/greenelab/scihub/tree/master/webapp)), but in that case, we needed an application more sophisticated. For my personal blog, this didn't seem to have any clear advantages, and it would have prevented me from the automatic compilation performed by Github after every commit.

There's always the option to implement the site from scratch, but there's no reason to do that when so many resources are available for free. I looked into [jekyllthemes.org](http://jekyllthemes.org/), [jekyllthemes.io](https://jekyllthemes.io/) and [jekyll-themes.com](https://jekyll-themes.com) to find a good base theme. In the end, I decided to extend [Poole](http://getpoole.com/). I also liked called [Hyde](http://hyde.getpoole.com/) but it seemed too sophisticated for my use case.


#### SEO: Choosing the correct link structure for the blog

I read [this article](https://moz.com/blog/15-seo-best-practices-for-structuring-urls) with tips for choosing the best URL structure for the site. It's important to keep the URLs short and include as much information as possible. So I decided to go with the following format:

```yaml
permalink: /blog/:slug-:year:output_ext
```

That format is the [permalink configuration](https://jekyllrb.com/docs/permalinks/) from Jekyll, some things to note:

1. All posts will be under `/blog/`, which doesn't seem to have any bad effects and makes the URLs look better IMO.

2. A sluggified version of the title is included in the URL, this will allow me to separate the title from the keywords in the URL.

3. The `:year` is at the end of the URL. Some of these posts will probably be outdated at some point, so the date, when they were written is important.

4. All blog posts end with `.html`. I'm not sure if this is still valid, I remember reading somewhere that having the extension at the end was good for SEO purposes.

So for example, the URL of this blog post looks like:

<big>{{ site.url }}{{ page.url }}<big>

Another thing I included was [this tip](https://jamiegoodwin.uk/seo-friendly-nofollow-links-jekyll-github-pages/) to add `rel="nofollow"` to all links inside posts, since most of the time they'll be linking to external sources.


## Conclusions

This turned out to be more complicated than what I expected, given how simple the site looks. Except for the home page, everything else reuses most of the styles from Poole. The hardest part I think was finding a design that satisfied my requirements.

I don't plan to spend more resources into this for now. Hopefully I will still like this design in a few years.


*Additional notes for this post can be seen on [#2](https://github.com/arielsvn/arielsvn.github.io/issues/2).*


## Extra

Almost forgot, another thing I really liked was the [404 page]({{site.url}}/404). I created a custom drawing for it:

<img src="https://user-images.githubusercontent.com/1882507/40891169-118902f4-674f-11e8-92b0-8abb253a3b36.png" alt="404 page" style="max-height: 400px; margin-left: auto; margin-right: auto">

I think I'll start using more images like that one in future posts.



