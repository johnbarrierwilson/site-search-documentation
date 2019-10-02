---
title: "Customization"
slug: "/customization"
---

## Custom Styles and Layout

Styling is up to you.

You can choose use the out of the box styles, or customize them.

To provide custom styles:

1. Write your own styles that target the class names in the individual Components. Do **NOT** include `styles.css`.
2. Override the default styles. Include `styles.css`, and then overwrite with your own styles.

For layout, provide your own layout instead of using the `Layout` Component.

For views and HTML, see the next section.

## Component Views and HTML

All Components in this library can be customized by providing a `view` prop.

Each Component's `view` will have a custom signature.

This follows the [React Render Props](https://reactjs.org/docs/render-props.html) pattern.

The clearest way to determine a Component's `view` function signature is to
look at the corresponding view Component's source code in
[react-search-ui-views](packages/react-search-ui-views/). Each Component in that
library implements a `view` function for a Component in the React library, so it
serves as a great reference.

For example, if we were to customize the `PagingInfo` Component...

We'd look up the default view from the [Components Reference](#component-reference) section for the `PagingInfo` Component.

The corresponding view is [PagingInfo](packages/react-search-ui-views/src/PagingInfo.js) -- see how the naming matches up?

After viewing that Component's source, you'll see it accepts 4 props:

1. `end`
2. `searchTerm`
3. `start`
4. `totalResults`

In our case, we care about the `start` and `end` values.

We provide a view function that uses those two props:

```jsx
<PagingInfo
  view={({ start, end }) => (
    <div className="paging-info">
      <strong>
        {start} - {end}
      </strong>
    </div>
  )}
/>
```

We could also accomplish this with a functional Component:

```jsx
const PagingInfoView = ({ start, end }) => (
  <div className="paging-info">
    <strong>
      {start} - {end}
    </strong>
  </div>
);

return <PagingInfo view={PagingInfoView} />;
```

## Component Behavior

**It will be helpful to read the [Headless Core](#headless-core) section first.**

We have two primary recommendations for customizing Component behavior:

1. Override state and action props before they are passed to your Component, using the `mapContextToProps` param. This
   will override the default [mapContextToProps](#mapContextToProps) for the component.
2. Override props before they are passed to your Component's view.

### Override mapContextToProps

Every Component supports a `mapContextToProps` prop, which allows you to modify state and actions
before they are received by the Component.

**NOTE** This MUST be an immutable function. If you directly update the props or context, you will have major issues in your application.

A practical example might be putting a custom sort on your facet data.

This example orders a list of states by name:

```jsx
<Facet
  mapContextToProps={context => {
    if (!context.facets.states) return context;
    return {
      ...context,
      facets: {
        ...(context.facets || {}),
        states: context.facets.states.map(s => ({
          ...s,
          data: s.data.sort((a, b) => {
            if (a.value > b.value) return 1;
            if (a.value < b.value) return -1;
            return 0;
          })
        }))
      }
    };
  }}
  field="states"
  label="States"
  show={10}
/>
```

### Overriding view props

An example of this is modifying the `onChange` handler of the `Paging` Component
view. Hypothetically, you may need to know every time a user
pages past page 1, indicating that they are not finding what they need on the first page
of search results.

```jsx
import { Paging } from "@elastic/react-search-ui";
import { Paging as PagingView } from "@elastic/react-search-ui-views";

function reportChange(value) {
  // Some logic to report the change
}

<Paging
  view={props =>
    PagingView({
      ...props,
      onChange: value => {
        reportChange(value);
        return props.onChange(value);
      }
    })
  }
/>;
```

In this example, we did the following:

1. Looked up what the default view is for our Component in the
   [Component Reference](#component-reference) guide.
2. Imported that view as `PagingView`.
3. Passed an explicit `view` to our `Paging` Component, overriding
   the `onChange` prop with our own implementation, and ultimately rendering
   `PagingView` with the updated props.
