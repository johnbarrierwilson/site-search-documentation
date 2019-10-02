---
title: "Getting started 🐣"
slug: "/getting-started"
---

## Install

Install **React Search UI** and the **App Search** connector.

```sh
# Install React Search UI and a Connector, like the Elastic App Search Connector
npm install --save @elastic/react-search-ui @elastic/search-ui-app-search-connector
```

## Creating a search experience

<a id="search-ui"></a>

Use out of the box components, styles, and layouts to build a search experience in a matter of minutes.

```jsx
import React from "react";
import AppSearchAPIConnector from "@elastic/search-ui-app-search-connector";
import { SearchProvider, Results, SearchBox } from "@elastic/react-search-ui";
import { Layout } from "@elastic/react-search-ui-views";

import "@elastic/react-search-ui-views/lib/styles/styles.css";

const connector = new AppSearchAPIConnector({
  searchKey: "search-371auk61r2bwqtdzocdgutmg",
  engineName: "search-ui-examples",
  hostIdentifier: "host-2376rb"
});

export default function App() {
  return (
    <SearchProvider
      config={{
        apiConnector: connector
      }}
    >
      <div className="App">
        <Layout
          header={<SearchBox />}
          bodyContent={<Results titleField="title" urlField="nps_link" />}
        />
      </div>
    </SearchProvider>
  );
}
```

Or go "headless", and take complete control over the look and feel of your search experience.

```jsx
<SearchProvider config={config}>
  <WithSearch
    mapContextToProps={({ searchTerm, setSearchTerm, results }) => ({
      searchTerm,
      setSearchTerm,
      results
    })}
  >
    {({ searchTerm, setSearchTerm, results }) => {
      return (
        <div>
          <input
            value={searchTerm}
            onChange={e => setSearchTerm(e.target.value)}
          />
          {results.map(r => (
            <div key={r.id.raw}>{r.title.raw}</div>
          ))}
        </div>
      );
    }}
  </WithSearch>
</SearchProvider>
```

A search experience built with Search UI is composed of the following layers:

1. [A Search API](#1-search-api)
2. [A Connector](#2-connectors)
3. [A SearchProvider](#3-searchprovider)
4. [Components](#4-components)
5. [Styles and Layout](#5-styles-and-layout)

```
Styles and Layout -> Components -> SearchProvider -> Connector -> Search API
```

---

### 1. Search API

A Search API is any API that you use to search data.

We recommend [**Elastic App Search**](https://www.elastic.co/cloud/app-search-service?ultron=searchui-repo&blade=readme&hulk=product).

It has Elasticsearch at its core, offering refined search UIs, robust documentation, and accessible dashboard tools.

You can start a [14 day trial of the managed service](https://www.elastic.co/cloud/app-search-service?ultron=searchui-repo&blade=readme&hulk=product) or [host the self managed package for free](https://www.elastic.co/downloads/app-search?ultron=searchui-repo&blade=readme&hulk=product).

Once your data is indexed into App Search or a similar service, you're good to go.

### 2. Connectors

A connector is a module that tell Search UI how to connect and communicate with your Search API.

It generates Search API calls for you so that Search UI will "just work", right out of the box.

```js
const connector = new AppSearchAPIConnector({
  searchKey: "search-371auk61r2bwqtdzocdgutmg",
  engineName: "search-ui-examples",
  hostIdentifier: "host-2376rb"
});
```

_Read the [advanced README](./ADVANCED.md#build-your-own-connector) to learn how to build a connector for any Search API._

### 3. SearchProvider

`SearchProvider` is the top level component in your Search UI implementation.

It is where you configure your search experience and it ties all of your components together, so that they work as a cohesive application.

```jsx
<SearchProvider
  config={{
    apiConnector: connector
  }}
>
  <div className="App">{/* Place Components here! */}</div>
</SearchProvider>
```

While components can be handy, a search experience can have requirements that don't quite fit what components provide "out of the box". Use `WithSearch` to access "actions" and "state" in a [Render Prop](https://reactjs.org/docs/render-props.html), giving you maximum flexibility over the experience.

```jsx
<SearchProvider
  config={{
    apiConnector: connector
  }}
>
  <WithSearch
    mapContextToProps={({ searchTerm, setSearchTerm }) => ({
      searchTerm,
      setSearchTerm
    })}
  >
    {({ searchTerm, setSearchTerm }) => (
      <div className="App">{/* Work directly with state and actions! */}</div>
    )}
  </WithSearch>
</SearchProvider>
```

_Read the [Advanced Configuration Guide](./ADVANCED.md#advanced-configuration) or learn more about the state management and the [Headless Core](./ADVANCED.md#headless-core)._

### 4. Components

Components are the building blocks from which you craft your search experience.

Each Component - like `SearchBox` and `Results` - is a child of the `SearchProvider` object:

```jsx
<SearchProvider
  config={{
    apiConnector: connector
  }}
>
  <div className="App">
    <div className="Header">
      <SearchBox />
    </div>
    <div className="Body">
      <Results titleField="title" urlField="nps_link" />
    </div>
  </div>
</SearchProvider>
```

The following Components are available:

- SearchBox
- Results
- Result
- ResultsPerPage
- Facet
- Sorting
- Paging
- PagingInfo
- ErrorBoundary

_Read the [Component Reference](./ADVANCED.md#component-reference) for a full breakdown._

### 5. Styles and Layout

For basic styles, include:

```jsx
import "@elastic/react-search-ui-views/lib/styles/styles.css";
```

For a basic layout, which helps quickly get a UI bootstrapped, use the [Layout](packages/react-search-ui-views/src/layouts/Layout.js) Component.

```jsx
import { Layout } from "@elastic/react-search-ui-views";

<Layout header={<SearchBox />} bodyContent={<Results />} />;
```

The provided styles and layout can be found in the [react-search-ui-views](packages/react-search-ui-views) package.

_Read the [Customization guide](./ADVANCED.md#customization) for more design details._
