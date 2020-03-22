# Ariel's personal blog

The blog http://arielrodriguezromero.com, hosted on Github Pages and uses Jeckyll to generate the HTML files.

## Running Locally

Run the following commands:

```
bundle install
jekyll s
```

More info on how to [set up Jekyll with Github Pages](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/).

Alternatively, it's also possible to run it in a docker container with:

```
docker run -p 4000:4000 -v $(pwd):/site bretfisher/jekyll-serve
```