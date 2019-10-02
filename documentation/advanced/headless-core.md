---
title: "Headless Core"
slug: "/headless-core"
---

## Headless Core Concepts

```
                                    |
    @elastic/react-search-ui        |   @elastic/search-ui
                                    |
                                    |
          SearchProvider <--------------- SearchDriver
              |     |               |          |
   State /    |     |               |          | State /
   Actions    |     |               |          | Actions
              |     |               |          |
        Components  |               |          |
              |     |               |          |
              v     v               |          v
------------------------------------+----------------------------
              |     |                          |
              v     v                          v
          Using     Headless Usage       Headless Usage outside
     Components     in React             of React
```

The core is a separate, vanilla JS library which can be used for any JavaScript based implementation.

> [@elastic/search-ui](https://github.com/elastic/search-ui/tree/master/packages/search-ui)

The Headless Core implements the functionality behind a search experience, but without its own view. It provides the underlying "state" and "actions" associated with that view. For instance, the core provides a `setSearchTerm` action, which can be used to save a `searchTerm` property in the state. Calling `setSearchTerm` using the value of an `<input>` will save the `searchTerm` to be used to build a query.

All of the Components in this library use the Headless Core under the hood. For instance, Search UI provides a `SearchBox` Component for collecting input from a user. But you are not restricted to using just that Component. Since Search UI lets you work directly with "state" and "actions", you could use any type of input you want! As long as your input or Component calls the Headless Core's `setSearchTerm` action, it will "just work". This gives you maximum flexibility over your experience if you need more than the Components in Search UI have to offer.

The `SearchProvider` is a React wrapper around the Headless Core, and makes state and actions available to Search UI
and in a React [Context](https://reactjs.org/docs/context.html), and also via a
[Render Prop](https://reactjs.org/docs/render-props.html).

It looks like this:

```jsx
<SearchProvider config={config}>
  {/*SearchProvider exposes the "Context"*/}
  {context => {
    // Context contains state, like "searchTerm"
    const searchTerm = context.searchTerm;
    // Context also contains actions, like "setSearchTerm"
    const setSearchTerm = context.setSearchTerm;
    return (
      <div className="App">
        {/*An out-of-the-box Component like SearchBox uses State and Actions under the hood*/}
        <SearchBox />
        {/*We could work directly with those State and Actions also */}
        <input value={searchTerm} onChange={setSearchTerm} />
      </div>
    );
  }}
</SearchProvider>
```

## Working with the Headless Core

If you wish to work with Search UI outside of a particular Component, you'll work
directly with the core.

There are two methods for accessing the headless core directly, `withSearch` and
`WithSearch`. They use the [HOC](https://reactjs.org/docs/higher-order-components.html) and
[Render Props](https://reactjs.org/docs/render-props.html) patterns, respectively. The two methods
are similar, and choosing between the two is mostly personal preference.

Both methods expose a `mapContextToProps` function which allows you to pick which state and actions
from context you need to work with.

### mapContextToProps

`mapContextToProps` allows you to pick which state and actions
from Context you need to work with. `withSearch` and `WithSearch` both use [React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent),
and will only re-render when the picked state has changed.

| name    | type   | description         |
| ------- | ------ | ------------------- |
| context | Object | The current Context |
| props   | Object | The current props   |

ex.:

```jsx
// Selects `searchTerm` and `setSearchTerm` for use in Component
withSearch(({ searchTerm, setSearchTerm }) => ({
  searchTerm,
  setSearchTerm
}))(Component);

// Uses current `props` to conditionally modify context
withSearch(({ searchTerm }, { someProp }) => ({
  searchTerm: someProp ? "" : searchTerm
}))(Component);
```

### withSearch

This is the [HOC](https://reactjs.org/docs/higher-order-components.html) approach to working with the
core.

This is typically used for creating your own Components.

See [Build Your Own Component](#build-your-own-component).

### WithSearch

This is the [Render Props](https://reactjs.org/docs/render-props.html) approach to working with the core.

One use case for that would be to render a "loading" indicator any time the application is fetching data.

For example:

```jsx
<SearchProvider config={config}>
  <WithSearch mapContextToProps={({ isLoading }) => ({ isLoading })}>
    {({ isLoading }) => (
      <div className="App">
        {isLoading && <div>I'm loading now</div>}
        {!isLoading && (
          <Layout
            header={<SearchBox />}
            bodyContent={<Results titleField="title" urlField="nps_link" />}
          />
        )}
      </div>
    )}
  </WithSearch>
</SearchProvider>
```

## Headless Core Reference

### SearchProvider

The `SearchProvider` is a top-level Component which is essentially a wrapper around the core.

It exposes the [State](#state) and [Actions](#actions) of the core in a [Context](#context).

Params:

| name     | type       | description                                                        |
| -------- | ---------- | ------------------------------------------------------------------ |
| config   | Object     | See the [Advanced Configuration](#advanced-configuration) section. |
| children | React Node |                                                                    |

### Context

The "Context" is a flattened object containing, as keys, all [State](#state) and [Actions](#actions).

We refer to it as "Context" because it is implemented with a [React Context](https://reactjs.org/docs/context.html).

ex.

```js
{
  resultsPerPage: 10, // Request State
  setResultsPerPage: () => {}, // Action
  current: 1, // Request State
  setCurrent: () => {}, // Action
  error: '', // Response State
  isLoading: false, // Response State
  totalResults: 1000, // Response State
  ...
}
```

### Actions

| method              | params                                                                                                                                                                                                                                                                                                                                                                                                                                                           | return | description                                                                                                                               |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `addFilter`         | `name` String - field name to filter on<br/>`value` String - field value to filter on<br/>`filterType` String - type of filter to apply: "all", "any", or "none"                                                                                                                                                                                                                                                                                                 |        | Add a filter in addition to current filters values.                                                                                       |
| `setFilter`         | `name` String - field name to filter on<br/>`value` String - field value to filter on<br/>`filterType` String - type of filter to apply: "all", "any", or "none"                                                                                                                                                                                                                                                                                                 |        | Set a filter value, replacing current filter values.                                                                                      |
| `removeFilter`      | `name` String - field to remove filters from<br/>`value` String - (Optional) Specify which filter value to remove<br/>`filterType` String - (Optional) Specify which filter type to remove: "all", "any", or "none"                                                                                                                                                                                                                                              |        | Removes filters or filter values.                                                                                                         |
| `reset`             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |        | Reset state to initial search state.                                                                                                      |
| `clearFilters`      | `except` Array[String] - List of field names that should NOT be cleared                                                                                                                                                                                                                                                                                                                                                                                          |        | Clear all filters.                                                                                                                        |
| `setCurrent`        | Integer                                                                                                                                                                                                                                                                                                                                                                                                                                                          |        | Update the current page number. Used for paging.                                                                                          |
| `setResultsPerPage` | Integer                                                                                                                                                                                                                                                                                                                                                                                                                                                          |        |                                                                                                                                           |
| `setSearchTerm`     | `searchTerm` String<br/> `options` Object<br/>`options.refresh` Boolean - Refresh search results on update. Default: `true`.<br/>`options.debounce` Number - Length to debounce any resulting queries<br/>`options.autocompleteSuggestions` Boolean - Fetch query suggestions for autocomplete on update, stored in `autocompletedSuggestions` state<br/>`options.autocompleteResults` Boolean - Fetch results on update, stored in `autocompletedResults` state |        |                                                                                                                                           |
| `setSort`           | `sortField` String - field to sort on<br/>`sortDirection` String - "asc" or "desc"                                                                                                                                                                                                                                                                                                                                                                               |        |                                                                                                                                           |
| `trackClickThrough` | `documentId` String - The document ID associated with the result that was clicked<br/>`tag` - Array[String] Optional tags which can be used to categorize this click event                                                                                                                                                                                                                                                                                       |        | Report a clickthrough event, which is when a user clicks on a result link.                                                                |
| `a11yNotify`        | `messageFunc` String - object key to run as function<br/>`messageArgs` Object - Arguments to pass to form your screen reader message string                                                                                                                                                                                                                                                                                                                      |        | Reads out a screen reader accessible notification. See `a11yNotificationMessages` under [Advanced Configuration](#advanced-configuration) |

### State

State can be divided up into a few different types.

1. Request State - State that is used as parameters on Search API calls.
2. Result State - State that represents a response from a Search API call.
3. Application State - The general state.

Request State and Result State will often have similar values. For instance, `searchTerm` and `resultSearchTerm`.
`searchTerm` is the current search term in the UI, and `resultSearchTerm` is the term associated with the current
`results`. This can be relevant in the UI, where you might not want the search term on the page to change until AFTER
a response is received, so you'd use the `resultSearchTerm` state.

#### Request State

State that is used as parameters on Search API calls.

Request state can be set by:

- Using actions, like `setSearchTerm`
- The `initialState` option.
- The URL query string, if `trackUrlState` is enabled.

| option                                            | type                                                            | required? | source                                 |
| ------------------------------------------------- | --------------------------------------------------------------- | --------- | -------------------------------------- |
| `current`                                         | Integer                                                         | optional  | Current page number                    |
| `filters`                                         | Array[[Filter](./packages/react-search-ui/src/types/Filter.js)] | optional  |                                        |
| <a name="resultsPerPageProp"></a>`resultsPerPage` | Integer                                                         | optional  | Number of results to show on each page |
| `searchTerm`                                      | String                                                          | optional  | Search terms to search for             |
| `sortDirection`                                   | String ["asc" \| "desc"]                                        | optional  | Direction to sort                      |
| `sortField`                                       | String                                                          | optional  | Name of field to sort on               |

#### Response State

State that represents a response from a Search API call.

It is not directly update-able.

It is updated indirectly by invoking an action which results in a new API request.

| field                               | type                                                                                   | description                                                                                                                                                                                                                                                               |
| ----------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `autocompletedResults`              | Array[[Result](./packages/react-search-ui/src/types/Result.js)]                        | An array of results items fetched for an autocomplete dropdown.                                                                                                                                                                                                           |
| `autocompletedResultsRequestId`     | String                                                                                 | A unique ID for the current autocompleted search results.                                                                                                                                                                                                                 |
| `autocompletedSuggestions`          | Object[String, Array[[Suggestion](./packages/react-search-ui/src/types/Suggestion.js)] | A keyed object of query suggestions. It's keyed by type since multiple types of query suggestions can be set here.                                                                                                                                                        |
| `autocompletedSuggestionsRequestId` | String                                                                                 | A unique ID for the current autocompleted suggestion results.                                                                                                                                                                                                             |
| `facets`                            | Object[[Facet](./packages/react-search-ui/src/types/Facet.js)]                         | Will be populated if `facets` configured in [Advanced Configuration](#advanced-configuration).                                                                                                                                                                            |
| `requestId`                         | String                                                                                 | A unique ID for the current search results.                                                                                                                                                                                                                               |
| `results`                           | Array[[Result](./packages/react-search-ui/src/types/Result.js)]                        | An array of result items.                                                                                                                                                                                                                                                 |
| `resultSearchTerm`                  | String                                                                                 | As opposed the the `searchTerm` state, which is tied to the current search parameter, this is tied to the searchTerm for the current results. There will be a period of time in between when a request is started and finishes where the two pieces of state will differ. |
| `totalResults`                      | Integer                                                                                | Total number of results found for the current query.                                                                                                                                                                                                                      |

#### Application State

Application state is the general application state.

| field         | type    | description                                                                                                        |
| ------------- | ------- | ------------------------------------------------------------------------------------------------------------------ |
| `error`       | String  | Error message, if an error was thrown.                                                                             |
| `isLoading`   | boolean | Whether or not a search is currently being performed.                                                              |
| `wasSearched` | boolean | Has any query been performed since this driver was created? Can be useful for displaying initial states in the UI. |
