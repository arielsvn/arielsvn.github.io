---
title: Responsive component in React
date: 2018-12-08 09:00:01 Z
categories:
  - react
  - responsive
layout: post
summary: Responsive React? We discuss here an approach for creating a responsive React component. Includes an example of a search sidebar that becomes a side menu.
---

Responsiveness is the ability of a site to adapt to multiple screen resolutions, and it has become one integral part of any modern web application. In many situations, the web components can be transformed using CSS media queries, which is ideal since it's very easy to implement a mobile-first design with them. However in some cases, there are big differences between what needs to be displayed in big screens vs smaller ones, so we need to implement changes in the HTML code.

I maintain [refine.bio](https://www.refine.bio/), which is a React single page application. This article describes a solution for this problem, where we wanted to have a sidebar appear as a side menu for mobile devices.

<table>
  <tr>
    <td>
    <img src="https://user-images.githubusercontent.com/1882507/45521645-e4feac00-b78c-11e8-9d9d-ef56c8498c71.png" alt="404 page" style="max-height: 400px; margin-left: auto; margin-right: auto">
    </td>
    <td>
    <img src="https://user-images.githubusercontent.com/1882507/45521462-04490980-b78c-11e8-97a2-b02f41f2876a.png" alt="404 page" style="max-height: 400px; margin-left: auto; margin-right: auto">
    </td>
  </tr>
</table>

## Responsive component

Whatever we do, there has to be some logic to decide what to display depending on the screen resolution. This can be abstracted into a new component. [This article](https://goshakkk.name/different-mobile-desktop-tablet-layouts-react/) discusses an approach to do it, here I modified it to accept two render properties instead of putting the components there directly.

```js
const DESKTOP_LIMIT = 1024;

class ResponsiveSwitch extends React.Component {
  constructor() {
    super();

    this.state = {
      width: window.innerWidth
    };
  }

  handleWindowSizeChange = () => {
    this.setState({ width: window.innerWidth });
  };

  componentWillMount() {
    window.addEventListener('resize', this.handleWindowSizeChange);
  }

  componentWillUnmount() {
    window.removeEventListener('resize', this.handleWindowSizeChange);
  }

  render() {
    const { mobile, desktop } = this.props;
    let isMobile = this.state.width < DESKTOP_LIMIT;
    return isMobile ? mobile() : desktop();
  }
}
```

In this case, the component receives two [render properties](https://reactjs.org/docs/render-props.html), one will be rendered for mobile devices and the other one for larger ones.

```jsx
<ResponsiveSwitch
  mobile={() => <MobileSidebarComponent />}
  desktop={() => <DesktopSidebarComponent />}
/>
```

I only needed two states, but following this same pattern more slots can be added for other devices, such as tablets. In general, I'd rather keep the custom JS logic per device to a minimum, and push as many customizations as possible to CSS.

## SideMenu and Modal

If we want to reuse the same sidebar code that is used on desktop, we'll need some component to wrap it that takes care of putting it at a different place in the DOM, probably at the top level under the `body` element. We can easily achieve this using [portals](https://reactjs.org/docs/portals.html). To keep things simple, let's use the same code for modal as in the official docs example.

```js
class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(this._renderContent(), this.el);
  }

  _renderContent() {
    return <div>{this.props.isOpen && this.props.children}</div>;
  }
}
```

[refine.bio](https://github.com/AlexsLemonade/refinebio-frontend), uses [react-modal](http://reactcommunity.org/react-modal/), which provides a few more features including a backdrop out of the box.

With this we're ready to implement the side menu component, which will receive two render props:

- `component` will represent the element that is rendered in the tree, and that takes care of triggering the side menu
- `children` will contain the content of the child menu

This pattern allows us to abstract the state of whether the menu is open or not, and simplifies the usages of the `SideMenu` component.

```js
class SideMenu extends React.Component {
  state = {
    menuOpen: false
  };

  showMenu = () => this.setState({ menuOpen: true });
  hideMenu = () => this.setState({ menuOpen: false });

  render() {
    return (
      <React.Fragment>
        {this.props.component(this.showMenu)}

        <Modal isOpen={this.state.menuOpen} className="side-menu">
          {this.props.children({
            hideMenu: this.hideMenu
          })}
        </Modal>
      </React.Fragment>
    );
  }
}
```

## Content Layout

The filters component should take care of displaying the content we want on the desktop:

```js
function Filters() {
  ...
}
```

The mobile version is more interesting since it reuses it and wraps it into another component.

```js
function FiltersMobile() {
  return (
    <SideMenu component={showMenu => <a onClick={showMenu}>Show Filters</a>}>
      {({ hideMenu }) => (
        <div>
          <Filters />

          <a onClick={hideMenu}>Close Menu</a>
        </div>
      )}
    </SideMenu>
  );
}
```

And here is how we would use it in the main layout.

```html
<div className="layout">
  <div className="layout__side">
    <ReponsiveSwitch mobile={()=><FiltersMobile />} desktop={()=><Filters />} />
  </div>
  <div className="layout__main">... Main content</div>
</div>
```

### Conclusion

I like that this approach allows us to encapsulate all the logic into their own components so that it's easier to reuse.

- `ResponsiveSwitch` takes care of deciding what to display depending on the resolution

- `SideMenu` puts any component into a side menu, with a toggle that displays it

- `Filters` or however we want to name it, is the component that we re-use on both desktop and mobile.

With those components, it's not complicated to set up responsive sections in your application.
