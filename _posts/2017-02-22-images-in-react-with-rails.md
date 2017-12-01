---
layout: post
title:  "Display images in React application with Rails backend"
date:   2017-02-25 09:00:01
categories: rails, js, react, redux
---

I'm not sure if it's a good idea to have a React app with Rails in the backend, because a few issues arise with this combination... One of them is for example how to be able to reference images in the front-end.

In Rails all assets are precompiled when the app is deployed, and a hash is inserted in the name of the files. This has different advantages that are outlined [here](http://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark); but also prevents us from knowing the final path of the files.

My solution to this problem, was to create a hash with the urls of the images and pass it down to the React application, inspired by [this comment](https://github.com/reactjs/react-rails/issues/211#issuecomment-172884120).

An interesting idea was to create a `DataProvider` and place the hash with the images in the context of React, so that all components can access it without having to pass it down.

```js
class DataProvider extends Component {
  static propTypes = {
    images: PropTypes.object,
    children: PropTypes.element.isRequired,
  };

  getChildContext() {
    return {
      images: this.props.images,
    };
  }

  render() {
    return Children.only(this.props.children);
  }
}
```

This component can be used at the Root component, to wrap the entire application, so that the images hash is passed down. Here's an example of a compoent that access the images hash:

```js
class SomeComponent extends Component {
  render() {
    let images = this.context.images;
  }
}
SomeComponent.contextTypes = {
  images: React.PropTypes.object,
};
```

One thing we can do now is create a component that renders an image, given it's path on the server, for example:

```js
function imagePath(url, images) {
  let path = images[url];
  if (!path) {
    throw new Error(`Image url not found: ${path}`);
  }
  return path;
}

function Image(props, { images }) {
  return <img {...props} src={ imagePath(props.src, images) } />;
}
Image.contextTypes = { images: React.PropTypes.object };
Image.propTypes = {
  src: React.PropTypes.string.isRequired,
};
```

So now, inserting an image in the application can be as simple as:

```js
<Image src="images/logo.png" />
```
