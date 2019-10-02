---
title: "Connectors and Handlers"
slug: "/connectors-and-handlers"
---

**Learn about the [Headless Core](#headless-core) concepts first!**

---

Search UI exposes a number of event hooks which need handlers to be implemented in order for Search UI
to function properly.

The easiest way to provide handlers for these events is via an out-of-the-box "Connector", which
provides pre-built handlers, which can then be configured for your particular use case.

While we do provide out-of-the-box Connectors, it is also possible to implement these handlers directly,
override Connector methods, or provide "middleware" to Connectors in order to further customize
how Search UI interacts with your services.

#### Event Handlers

| method                      | params                                                                  | return                            | description                                                                                                                                         |
| --------------------------- | ----------------------------------------------------------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `onResultClick`             | `props` - Object                                                        |                                   | This method logs a click-through event to your APIs analytics service. This is triggered when a user clicks on a result on a result page.           |
|                             | - `query` - String                                                      |                                   | The query used to generate the current results.                                                                                                     |
|                             | - `documentId` - String                                                 |                                   | The id of the result that a user clicked.                                                                                                           |
|                             | - `requestId` - String                                                  |                                   | A unique id that ties the click to a particular search request.                                                                                     |
|                             | - `tags` - Array[String]                                                |                                   | Tags used for analytics.                                                                                                                            |
| `onSearch`                  | `state` - [Request State](#request-state)                               | [Response State](#response-state) |                                                                                                                                                     |
|                             | `queryConfig` - [Query Config](#query-config)                           |                                   |                                                                                                                                                     |
| `onAutocompleteResultClick` | `props` - Object                                                        |                                   | This method logs a click-through event to your APIs analytics service. This is triggered when a user clicks on a result in an autocomplete dropdown |
|                             | - `query` - String                                                      |                                   | The query used to generate the current results.                                                                                                     |
|                             | - `documentId` - String                                                 |                                   | The id of the result that a user clicked.                                                                                                           |
|                             | - `requestId` - String                                                  |                                   | A unique id that ties the click to a particular search request.                                                                                     |
|                             | - `tags` - Array[String]                                                |                                   | Tags used for analytics.                                                                                                                            |
| `onAutocomplete`            | `state` - [Request State](#request-state)                               | [Response State](#response-state) |                                                                                                                                                     |
|                             | `queryConfig` - Object                                                  |                                   |                                                                                                                                                     |
|                             | - `results` - [Query Config](#query-config)                             |                                   | If this is set, results should be returned for autocomplete.                                                                                        |
|                             | - `suggestions` - [Suggestions Query Config](#suggestions-query-config) |                                   | If this is set, query suggestions should be returned for autocomplete.                                                                              |

### Implementing Handlers without a Connector

If you are using an API for search that there is no Connector for, it is possible to simply provide
handler implementations directly on the `SearchProvider`.

```jsx
<SearchProvider
  config={{
    onSearch: async state => {
      const queryForOtherService = transformSearchUIStateToQuery(state);
      const otherServiceResponse = await callSomeOtherService(
        queryForOtherService
      );
      return transformOtherServiceResponseToSearchUIState(otherServiceResponse);
    }
  }}
/>
```

This makes Search UI useful for services like `elasticsearch` which do not have a Connector
available.

For a thorough example of this, see the demo in [examples/elasticsearch](examples/elasticsearch/README.md)

### Overriding Connector Handlers

Explicitly providing a Handler will override the Handler provided by the Connector.

```jsx
<SearchProvider
  config={{
    apiConnector: connector,
    onSearch: async (state, queryConfig) => {
      const queryForOtherService = transformSearchUIStateToQuery(
        state,
        queryConfig
      );
      const otherServiceResponse = await callSomeOtherService(
        queryForOtherService
      );
      return transformOtherServiceResponseToSearchUIState(otherServiceResponse);
    }
  }}
/>
```

### Using middleware in Connector Handlers

Handler implementations can also be used as middleware for Connectors by leveraging
the `next` function.

```jsx
<SearchProvider
  config={{
    apiConnector: connector,
    onSearch: (state, queryConfig, next) => {
      const updatedState = someStateTransformation(state);
      return next(updatedState, queryConfig);
    }
  }}
/>
```

### Build your own Connector

An example of a connector is the [Site Search API Connector](./packages/search-ui-site-search-connector/README.md).

A connector simply needs to implement the Event Handlers listed above. The handlers typically:

1. Convert the current [Request State](#request-state) and [Query Config](#query-config) into the search semantics of
   your particular Search API.
2. Convert the response from your particular Search API into [Response State](#response-state).

While some handlers are meant for fetching data and performing searches, other handlers are meant for recording
certain user events in analytics services, such as `onResultClick` or `onAutocompleteResultClick`.

#### Errors

For error handling, a method must throw any error with a "message" field populated for any unrecoverable error. This
includes things like 404s, 500s, etc.
