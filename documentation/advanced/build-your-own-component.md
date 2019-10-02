---
title: "Build Your Own Components"
slug: "/build-your-own-components"
---

**Learn about the [Headless Core](#headless-core) concepts first!**

---

We provide a variety of Components out of the box.

There might be cases where we do not have the Component you need.

In this case, we provide a [Higher Order Component](https://reactjs.org/docs/higher-order-components.html)
called [withSearch](./packages/react-search-ui/src/withSearch.js).

It gives you access to work directly with Search UI's [Headless Core](#headless-core).

This lets you create your own Components for Search UI.

Ex. Creating a Component for clearing all filters

```jsx
import React from "react";
import { withSearch } from "@elastic/react-search-ui";

function ClearFilters({ filters, clearFilters }) {
  return (
    <div>
      <button onClick={() => clearFilters()}>
        Clear {filters.length} Filters
      </button>
    </div>
  );
}

export default withSearch(({ filters, clearFilters }) => ({
  filters,
  clearFilters
}))(ClearFilters);
```

Note that `withSearch` accepts a `mapContextToProps` function as the first parameter. Read more about that
in the [mapContextToProps](#mapContextToProps) section.

Also note that all components created with `withSearch` will be Pure Components. Read more
about that [here](#performance).
