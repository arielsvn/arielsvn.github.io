---
title: Hiding state, simplifying modal dialogs usages
date: 2018-09-19 09:00:01 Z
categories:
  - react
  - responsive
layout: post
summary: Render props is a very powerful technique in React, here it's applied to Modal Dialogs to simplify their usage. There're examples with `react-modal`.
---

Similarly to [lifting state up](https://reactjs.org/docs/lifting-state-up.html), in some scenarios you might not care how a piece of state is managed. Like for example, a modal dialog with an `isOpen` property that decides if the dialog is open or not. [react-modal](https://github.com/reactjs/react-modal) works like this:

```js
<Modal isOpen={true}>
  <div>... modal dialog content here</div>
</Modal>
```

This offers a great deal of flexibility, but a problem arises, whenever you need to use this modal component you'll also need to add logic to control whether the dialog is open or not. One solution could be creating a base class for all modal dialogs and manage the `isOpen` state there, but [inheritance is usually not recommended](https://reactjs.org/docs/composition-vs-inheritance.html) in React. There's a better way.

It should be possible to extract the logic that controls the `isOpen` property from the content of the dialog and the component that takes care of launching it. This seems like a good use case for [render props](https://reactjs.org/docs/render-props.html). The following `ModalManager` is a possible solution, it has a property `component` that returns a button or whatever element is responsible for displaying the modal. And the `children` property, also a render prop, will return the content of the modal.

```js
<ModalManager
  component={showModal => <button onClick={showModal}>Show Modal</button>}
>
  {({ hideModal }) => (
    <div>
      <h1>Modal content</h1>

      <button onClick={hideModal}>Hide modal</button>
    </div>
  )}
</ModalManager>
```

This will make using a modal dialog easier since it's completely abstracted from all the logic concerned with the state of the modal dialog. Assuming we have a separate `Modal` component (like the one provided by [react-modal](https://github.com/reactjs/react-modal)), implementing `ModalManager` is simple.

```js
class ModalManager extends React.Component {
  state = {
    modalOpen: false
  };

  showModal = () => this.setState({ modalOpen: true });

  hideModal = () => this.setState({ modalOpen: false });

  render() {
    return (
      <React.Fragment>
        {this.props.component(this.showModal)}
        <Modal isOpen={this.state.modalOpen} onRequestClose={this.hideModal}>
          {this.props.children({
            hideModal: this.hideModal
          })}
        </Modal>
      </React.Fragment>
    );
  }
}
```

Here's the [basic example](http://reactcommunity.org/react-modal/examples/minimal.html) from [react-modal](https://github.com/reactjs/react-modal) modified to use `ModalManager`.

<p data-height="401" data-theme-id="dark" data-slug-hash="QVJxpx" data-default-tab="result" data-user="arielsvn" data-pen-title="Sample: ModalManager, hiding state with render props" class="codepen">See the Pen <a href="https://codepen.io/arielsvn/pen/QVJxpx/">Sample: ModalManager, hiding state with render props</a> by Ariel Rodriguez Romero (<a href="https://codepen.io/arielsvn">@arielsvn</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script><br>

This class is used several times in [refinebio](https://github.com/AlexsLemonade/refinebio-frontend/blob/6d527419a39b297407e83fa594b71f721a91c7d9/src/components/Modal/ModalManager.js), there `ModalManager` has other properties to support more use cases. I should also thank [ramenhog](https://ramenhog.com/) since we came up with this together.

[Render props](https://reactjs.org/docs/render-props.html) are a very powerful technique in React, I can't stop using it now that I learned about it. The first person I heard mention it was Michael Jackson, one of the creators of [react-router](https://github.com/ReactTraining/react-router).

<center>
<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">I can do anything you&#39;re doing with your HOC using a regular component with a render prop. Come fight me.</p>&mdash; MICHAEL JACKSON (@mjackson) <a href="https://twitter.com/mjackson/status/885910701520207872?ref_src=twsrc%5Etfw">July 14, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script><br>
</center>
