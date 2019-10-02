---
title: "Performance"
slug: "/performance"
---

This library is optimized to avoid full sub-tree re-rendering, and so that components only re-render when state changes
that are relevant to those particular components occur.

In order to accomplish this, all components within
this library are "Pure Components". You can read more about the concept and potential pitfalls
of Pure Components in the React
[Optimizing Performance](https://reactjs.org/docs/optimizing-performance.html#avoid-reconciliation) guide.

The thing to be most cautious of is not to
[mutate state](https://reactjs.org/docs/optimizing-performance.html#the-power-of-not-mutating-data) that you will
pass as props to any of these components.

Example of what not to do:

```jsx
class SomeComponent extends React.Component {
  changeSorting = () => {
    const { options } = this.state;
    // Mutating an existing array in state rather than creating a new one is bad. Since Sorting component is "Pure"
    // it won't update after calling `setState` here.
    options.push("newOption");
    this.setState({ options });
  };

  render() {
    const { options } = this.state;
    return <Sorting options={options} />;
  }
}
```

Instead, do:

```jsx
// Create a new options array and copy the old values into that new array.
this.setState(prevState => ({ options: [...prevState.options, "newOption"] }));
```

If you ever need to debug performance related issues, see the instructions in the Optimizing Performance guide for
enabling the "Highlight Updates" feature in the
[React Developer tools for Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi).
