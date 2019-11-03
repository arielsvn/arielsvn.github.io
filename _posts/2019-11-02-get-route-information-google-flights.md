---
title: Get route information from Google Flights
date: 2019-11-02 09:00:01 Z
categories:
  - scrapper
  - javascript
  - cuba
  - google flights
layout: post
summary: Simple strategy to scrape information from google flights (with manual work). It's helpful if you don't need a lot of information.
---

This next December (12/2019), the US government announced that they will ban flights from the US to all cities in Cuba except for the US.

This gave me an idea to create a visualization to shows the impact this could have on those smaller airports.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Starting in Dec, the US will ban flights to all Cuban cities except Havana.<br><br>Created this <a href="https://twitter.com/hashtag/dataviz?src=hash&amp;ref_src=twsrc%5Etfw">#dataviz</a> with all direct flight destinations from <a href="https://twitter.com/hashtag/cuba?src=hash&amp;ref_src=twsrc%5Etfw">#cuba</a>&#39;s major airports, canceled routes are displayed in red <a href="https://t.co/wEXuAt0J5r">pic.twitter.com/wEXuAt0J5r</a></p>&mdash; Ariel Rodriguez Romero (@arielswn) <a href="https://twitter.com/arielswn/status/1190749939862818820?ref_src=twsrc%5Etfw">November 2, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

One of the main challenges was getting information about the flight routes that go into Cuba. I found a [route database](https://openflights.org/data.html), but it almost has no information about Cuba - which wasn't a surprise.

Google Flights was another alternative. If you add a starting location and zoom out, it gives you all the destinations in the world from that point.

![google flight search](/assets/img/2019-11-02-google-flights.png)

My first idea was to build a scrapper and get all the flight information, however that seemed overcomplicated, especially because I only needed the routes for the top 6 airports in Cuba. So I came up with a different strategy.

I created a function `scrape()` that when executed in the console, saves all the destination data from the sidebar and puts it into a global object named `result`.

```js
var result = {};
function scrape() {
  let code = document.querySelector('.gws-flights-form__iata-code').textContent;
  let date = document.querySelector('.gws-flights-form__date-content')
    .textContent;
  if (!result[code]) {
    result[code] = {};
  }
  result[code][date] = [
    ...document.querySelectorAll('[jsname="destinationCard"]')
  ].map(element => {
    let destination = element.querySelector('h3');
    let duration = element.querySelector('[class$="__duration"]');
    let price = element.querySelector('[class$="__price-row"]');
    return {
      price: price && price.textContent,
      destination: destination && destination.textContent,
      duration: duration && duration.textContent
    };
  });
}
```

This function is fairly simple, with 20 lines of code and some manual work was all I needed to get the information I wanted. After running it in the console, I had to execute `scrape()` every time I wanted to save something. Then, I copied the `result` object into the clipboard and saved it in a JSON file.

```js
copy(result);
```

I used the [copy](https://scottwhittaker.net/chrome-devtools/2016/02/29/chrome-devtools-copy-object.html) function to extract the object and copy it into the clipboard.

### Conclusions

There was some manual work involved to get the data using this strategy. Since I wanted to get information for 6 airports and 7 days. However, I'm sure that writing a scrapper would have taken me more time and this just involved calling `scrape()` 35 times.

I liked the resulting data visualization, and it was a nice way to learn how to work with geodata and ggplot. The entire notebook is published [here](http://arielrodriguezromero.com/cuba-plots/00-outgoing-flights.nb.html).

The number of destinations from Havana impressed me.

![Havana flight](/assets/img/2019-11-02-havana-flights.png)
