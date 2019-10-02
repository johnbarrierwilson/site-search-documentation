---
title: "Advanced Configuration"
slug: "/advanced-configuration"
---

All configuration for Search UI is provided in a single configuration object.

```jsx
const configurationOptions = {
  apiConnector: connector,
  searchQuery: {
    disjunctiveFacets: ["acres"],
    disjunctiveFacetsAnalyticsTags: ["Ignore"],
    search_fields: {
      title: {},
      description: {}
    },
    result_fields: {
      title: {
        snippet: {
          size: 100,
          fallback: true
        }
      },
      nps_link: {
        raw: {}
      },
      description: {
        snippet: {
          size: 100,
          fallback: true
        }
      }
    },
    facets: {
      states: { type: "value", size: 30 },
      acres: {
        type: "range",
        ranges: [
          { from: -1, name: "Any" },
          { from: 0, to: 1000, name: "Small" },
          { from: 1001, to: 100000, name: "Medium" },
          { from: 100001, name: "Large" }
        ]
      }
    }
  },
  hasA11yNotifications: true,
  a11yNotificationMessages: {
    searchResults: ({ start, end, totalResults, searchTerm }) =>
      `Searching for "${searchTerm}". Showing ${start} to ${end} results out of ${totalResults}.`
  }
};

return (
  <SearchProvider config={configurationOptions}>
    {() => (
      <div className="App">
        <Layout
          header={<SearchBox />}
          bodyContent={<Results titleField="title" urlField="nps_link" />}
        />
      </div>
    )}
  </SearchProvider>
);
```

**It is helpful to [read the section on the headless core](#headless-core) first!**

| option                      | type                                                                    | required? | default | description                                                                                                                                                                                                                  |
| --------------------------- | ----------------------------------------------------------------------- | --------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `apiConnector`              | APIConnector                                                            | optional  |         | Instance of a Connector. For instance, [search-ui-app-search-connector](packages/search-ui-app-search-connector).                                                                                                            |
| `onSearch`                  | function                                                                | optional  |         | You may provide individual handlers instead of a Connector, override individual Connector handlers, or act as middleware to Connector methods. See [Connectors and Handlers](#connectors-and-handlers) for more information. |
| `onAutocomplete`            | function                                                                | optional  |         | You may provide individual handlers instead of a Connector, override individual Connector handlers, or act as middleware to Connector methods. See [Connectors and Handlers](#connectors-and-handlers) for more information. |
| `onResultClick`             | function                                                                | optional  |         | You may provide individual handlers instead of a Connector, override individual Connector handlers, or act as middleware to Connector methods. See [Connectors and Handlers](#connectors-and-handlers) for more information. |
| `onAutocompleteResultClick` | function                                                                | optional  |         | You may provide individual handlers instead of a Connector, override individual Connector handlers, or act as middleware to Connector methods. See [Connectors and Handlers](#connectors-and-handlers) for more information. |
| `autocompleteQuery`         | Object                                                                  | optional  | {}      | Configuration options for the main search query.                                                                                                                                                                             |
|                             | - `results` - [Query Config](#query-config)                             |           |         | Configuration options for results query, used by autocomplete.                                                                                                                                                               |
|                             | - `suggestions` - [Suggestions Query Config](#suggestions-query-config) |           |         | Configuration options for suggestions query, used by autocomplete.                                                                                                                                                           |
| `debug`                     | Boolean                                                                 | optional  | false   | Trace log actions and state changes.                                                                                                                                                                                         |
| `initialState`              | Object                                                                  | optional  |         | Set initial [State](#state) of the search. Any [Request State](#request-state) can be set here. This is useful for defaulting a search term, sort, etc.<br/><br/>Example:<br/>`{ searchTerm: "test", resultsPerPage: 40 }`   |
| `searchQuery`               | [Query Config](#query-config)                                           | optional  | {}      | Configuration options for the main search query.                                                                                                                                                                             |
| `trackUrlState`             | Boolean                                                                 | optional  | true    | By default, [Request State](#request-state) will be synced with the browser url. To turn this off, pass `false`.                                                                                                             |
| `urlPushDebounceLength`     | Integer                                                                 | optional  | 500     | The amount of time in milliseconds to debounce/delay updating the browser url after the UI update. This, for example, prevents excessive history entries while a user is still typing in a live search box.                  |
| `hasA11yNotifications`      | Boolean                                                                 | optional  | false   | Search UI will create a visually hidden live region to announce search results & other actions to screen reader users. This accessibility feature will be turned on by default in our 2.0 release.                           |
| `a11yNotificationMessages`  | Object                                                                  | optional  | {}      | You can override our default screen reader [messages](packages/search-ui/src/A11yNotifications.js#L49) (e.g. for localization), or create your own custom notification, by passing in your own key and message function(s).  |

## Query Config

Query configuration for Search UI largely follows the same API as the [App Search Search API](https://swiftype.com/documentation/app-search/api/search).

For example, if you add a `search_fields` configuration option, it will control which fields are actually returned from the API.

| option                             | type                     | required? | default | description                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------- | ------------------------ | --------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `facets`                           | Object                   | optional  |         | [App Search Facets API Reference](https://swiftype.com/documentation/app-search/api/search/facets). Tells Search UI to fetch facet data that can be used to build [Facet](#facet) Components. <br /><br />Example, using `states` field for faceting:<br/>`facets: {states: { type: "value", size: 30 }`                                                                                                                       |
| `disjunctiveFacets`                | Array[String]            | optional  |         | An array of field names. Every field listed here must have been configured in the `facets` field first. It denotes that a facet should be considered disjunctive. When returning counts for disjunctive facets, the counts will be returned as if no filter is applied on this field, even if one is applied. <br /><br />Example, specifying `states` field as disjunctive:<br/>`disjunctiveFacets: ['states']`               |
| `disjunctiveFacetsAnalyticsTags`   | Array[String]            | optional  |         | Used in conjunction with the `disjunctiveFacets` parameter. Adding `disjunctiveFacets` can cause additional API requests to be made to your API, which can create deceiving analytics. These queries will be tagged with "Facet-Only" by default. This field lets you specify a different tag for these. <br /><br />Example, use `junk` as a tag on all disjunctive API calls:<br/>`disjunctiveFacetsAnalyticsTags: ['junk']` |
| `conditionalFacets`                | Object[String, function] | optional  |         | This facet will only be fetched if the condition specified returns `true`, based on the currently applied filters. This is useful for creating hierarchical facets.<br/><br/>Example: don't return `states` facet data unless `parks` is a selected filter.<br/> `{ states: filters => isParkSelected(filters) }`                                                                                                              |
| `search_fields`                    | Object[String, Object]   | optional  |         | Fields which should be searched with search term.<br/><br/>[App Search search_fields API Reference](https://swiftype.com/documentation/app-search/api/search/search-fields)                                                                                                                                                                                                                                                    |
| `result_fields`                    | Object[String, Object]   | optional  |         | Fields which should be returned in results.<br/><br/>[App Search result_fields API Reference](https://swiftype.com/documentation/app-search/api/search/result-fields)                                                                                                                                                                                                                                                          |
| \* [Request State](#request-state) |                          | optional  |         | Any request state value can be provided here. If provided, it will ALWAYS override the value from state.                                                                                                                                                                                                                                                                                                                       |

## Suggestions Query Config

Suggestions Query configuration for Search UI largely follows the same API as the [App Search Search API](https://swiftype.com/documentation/app-search/api/query-suggestion).

Ex.

```
{
  "types": {
    "documents": {
      "fields": [
        "title",
        "states"
      ]
    }
  },
  "size": 4
}
```

| option  | type    | required? | source                                                                                       |
| ------- | ------- | --------- | -------------------------------------------------------------------------------------------- |
| `types` | Object  | required  | Object, keyed by "type" of query suggestion, with configuration for that type of suggestion. |
| `size`  | Integer | optional  | Number of suggestions to return.                                                             |

## API Config

Search UI makes all of the search API calls for your application.

You can control what these API calls look like with options such as `search_fields`, `result_fields`, and `facets`.

But there may be cases where certain API operations are not supported by Search UI.

For example, [App Search](https://www.elastic.co/cloud/app-search-service) supports a "grouping" feature, which Search UI does not support out of the box.

We can work around that by using the `beforeSearchCall` hook on the App Search Connector. This acts as a middleware
that gives you an opportunity to modify API requests and responses before they are made.

```js
const connector = new AppSearchAPIConnector({
  searchKey: "search-371auk61r2bwqtdzocdgutmg",
  engineName: "search-ui-examples",
  hostIdentifier: "host-2376rb",
  beforeSearchCall: (existingSearchOptions, next) =>
    next({
      ...existingSearchOptions,
      group: { field: "title" }
    })
});
```
