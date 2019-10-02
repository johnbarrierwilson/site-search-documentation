---
title: "Component Reference"
slug: "/component-reference"
---

Note that all components in this library are Pure Components. Read more
about that [here](#performance).

The following Components are available:

- [SearchBox](#searchbox)
- [Results](#results)
- [Result](#result)
- [ResultsPerPage](#resultsperpage)
- [Facet](#facet)
- [Sorting](#sorting)
- [Paging](#paging)
- [PagingInfo](#paginginfo)
- [ErrorBoundary](#errorboundary)

## SearchBox

Input element which accepts search terms and triggers a new search query.

### Example

```jsx

import { SearchBox } from "@elastic/react-search-ui";

...

<SearchBox />
```

### Configuring search queries

The input from `SearchBox` will be used to trigger a new search query. That query can be further customized
in the `SearchProvider` configuration, using the `searchQuery` property. See the
[Advanced Configuration](#advanced-configuration) guide for more information.

### Example of passing custom props to text input element

```jsx
<SearchBox inputProps={{ placeholder: "custom placeholder" }} />
```

### Example of view customizations

You can customize the entire view. This is useful to use an entirely different
autocomplete library (we use [downshift](https://github.com/downshift-js/downshift)). But for making small
customizations, like simply hiding the search button, this is often overkill.

```jsx
<SearchBox
  view={({ onChange, value, onSubmit }) => (
    <form onSubmit={onSubmit}>
      <input value={value} onChange={e => onChange(e.currentTarget.value)} />
    </form>
  )}
/>
```

You can also just customize the input section of the search box. Useful for things
like hiding the submit button or rearranging dom structure:

```jsx
<SearchBox
  inputView={({ getAutocomplete, getInputProps, getButtonProps }) => (
    <>
      <div className="sui-search-box__wrapper">
        <input
          {...getInputProps({
            placeholder: "I am a custom placeholder"
          })}
        />
        {getAutocomplete()}
      </div>
      <input
        {...getButtonProps({
          "data-custom-attr": "some value"
        })}
      />
    </>
  )}
/>
```

Note that `getInputProps` and `getButtonProps` are
[prop getters](https://kentcdodds.com/blog/how-to-give-rendering-control-to-users-with-prop-getters).
They are meant return a props object to spread over their corresponding UI elements. This lets you arrange
elements however you'd like in the DOM. It also lets you pass additional properties. You should pass properties
through these functions, rather directly on elements, in order to not override base values. For instance,
adding a `className` through these functions will assure that the className is only appended, not overriding base class values on those values.

`getAutocomplete` is used to determine where the autocomplete dropdown will be shown.

Or you can also just customize the autocomplete dropdown:

```jsx
<SearchBox
  autocompleteView={({ autocompletedResults, getItemProps }) => (
    <div className="sui-search-box__autocomplete-container">
      {autocompletedResults.map((result, i) => (
        <div
          {...getItemProps({
            key: result.id.raw,
            item: result
          })}
        >
          Result {i}: {result.title.snippet}
        </div>
      ))}
    </div>
  )}
/>
```

### Example using autocomplete results

"Results" are search results. The default behavior for autocomplete
results is to link the user directly to a result when selected, which is why
a "titleField" and "urlField" are required for the default view.

```jsx
<SearchBox
  autocompleteResults={{
    titleField: "title",
    urlField: "nps_link"
  }}
/>
```

### Example using autocomplete suggestions

"Suggestions" are different than "results". Suggestions are suggested queries. Unlike an autocomplete result, a
suggestion does not go straight to a result page when selected. It acts as a regular search query and
refreshes the result set.

```jsx
<SearchBox autocompleteSuggestions={true} />
```

### Example using autocomplete suggestions and autocomplete results

The default view will show both results and suggestions, divided into
sections. Section titles can be added to help distinguish between the two.

```jsx
<SearchBox
  autocompleteResults={{
    sectionTitle: "Suggested Results",
    titleField: "title",
    urlField: "nps_link"
  }}
  autocompleteSuggestions={{
    sectionTitle: "Suggested Queries"
  }}
/>
```

### Configuring autocomplete queries

Autocomplete queries can be customized in the `SearchProvider` configuration, using the `autocompleteQuery` property.
See the [Advanced Configuration Guide](#advanced-configuration) for more information.

```jsx
<SearchProvider
    config={{
      ...
      autocompleteQuery: {
        // Customize the query for autocompleteResults
        results: {
          result_fields: {
            // Add snippet highlighting
            title: { snippet: { size: 100, fallback: true } },
            nps_link: { raw: {} }
          }
        },
        // Customize the query for autocompleteSuggestions
        suggestions: {
          types: {
            // Limit query to only suggest based on "title" field
            documents: { fields: ["title"] }
          },
          // Limit the number of suggestions returned from the server
          size: 4
        }
      }
    }}
>
    <SearchBox
      autocompleteResults={{
        sectionTitle: "Suggested Results",
        titleField: "title",
        urlField: "nps_link"
      }}
      autocompleteSuggestions={{
        sectionTitle: "Suggested Queries",
      }}
    />
</SearchProvider>
```

### Example using multiple types of autocomplete suggestions

"Suggestions" can be generated via multiple methods. They can be derived from
common terms and phrases inside of documents, or be "popular" queries
generated from actual search queries made by users. This will differ
depending on the particular Search API you are using.

**Note**: Elastic App Search currently only supports type "documents", and Elastic Site Search
does not support suggestions. This is purely illustrative in case a Connector is used that
does support multiple types.

```jsx
<SearchProvider
    config={{
      ...
      autocompleteQuery: {
        suggestions: {
          types: {
            documents: { },
            // FYI, this is not a supported suggestion type in any current connector, it's an example only
            popular_queries: { }
          }
        }
      }
    }}
>
    <SearchBox
      autocompleteSuggestions={{
        // Types used here need to match types requested from the server
        documents: {
          sectionTitle: "Suggested Queries",
        },
        popular_queries: {
          sectionTitle: "Popular Queries"
        }
      }}
    />
</SearchProvider>
```

### Example using autocomplete in a site header

This is an example from a [Gatsby](https://www.gatsbyjs.org/) site, which overrides "submit" to navigate a user to the search
page for suggestions, and maintaining the default behavior when selecting a result.

```jsx
<SearchBox
  autocompleteResults={{
    titleField: "title",
    urlField: "nps_link"
  }}
  autocompleteSuggestions={true}
  onSubmit={searchTerm => {
    navigate("/search?q=" + searchTerm);
  }}
  onSelectAutocomplete={(selection, {}, defaultOnSelectAutocomplete) => {
    if (selection.suggestion) {
      navigate("/search?q=" + selection.suggestion);
    } else {
      defaultOnSelectAutocomplete(selection);
    }
  }}
/>
```

### Properties

| Name                          | type                                                                         | Required? | Default                                                            | Options | Description                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------- | ---------------------------------------------------------------------------- | --------- | ------------------------------------------------------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| className                     | String                                                                       | no        |                                                                    |         |                                                                                                                                                                                                                                                                                                                                                              |
| inputProps                    | Object                                                                       | no        |                                                                    |         | Props for underlying 'input' element. I.e., `{ placeholder: "Enter Text"}`.                                                                                                                                                                                                                                                                                  |
| searchAsYouType               | Boolean                                                                      | no        | false                                                              |         | Executes a new search query with every key stroke. You can fine tune the number of queries made by adjusting the `debounceLength` parameter.                                                                                                                                                                                                                 |
| debounceLength                | Number                                                                       | no        | 200                                                                |         | When using `searchAsYouType`, it can be useful to "debounce" search requests to avoid creating an excessive number of requests. This controls the length to debounce / wait.                                                                                                                                                                                 |
| view                          | Render Function                                                              | no        | [SearchBox](packages/react-search-ui-views/src/SearchBox.js)       |         | Used to override the default view for this Component. See the [Customization: Component views and HTML](#component-views-and-html) section for more information.                                                                                                                                                                                             |
| autocompleteResults           | Boolean or [AutocompleteResultsOptions](#AutocompleteResultsOptions)         | Object    | no                                                                 |         | Configure and autocomplete search results. Boolean option is primarily available for implementing custom views.                                                                                                                                                                                                                                              |
| autocompleteSuggestions       | Boolean or [AutocompleteSuggestionsOptions](#AutocompleteSuggestionsOptions) | Object    | no                                                                 |         | Configure and autocomplete query suggestions. Boolean option is primarily available for implementing custom views. Configuration may or may not be keyed by "Suggestion Type", as APIs for suggestions may support may than 1 type of suggestion. If it is not keyed by Suggestion Type, then the configuration will be applied to the first type available. |
| autocompleteMinimumCharacters | Integer                                                                      | no        | 0                                                                  |         | Minimum number of characters before autocompleting.                                                                                                                                                                                                                                                                                                          |
| autocompleteView              | Render Function                                                              | no        | [Autocomplete](packages/react-search-ui-views/src/Autocomplete.js) |         | Provide a different view just for the autocomplete dropdown.                                                                                                                                                                                                                                                                                                 |
| inputView                     | Render Function                                                              | no        | [SearchInput](packages/react-search-ui-views/src/SearchInput.js)   |         | Provide a different view just for the input section.                                                                                                                                                                                                                                                                                                         |
| onSelectAutocomplete          | Function(selection. options, defaultOnSelectAutocomplete)                    | no        |                                                                    |         | Allows overriding behavior when selected, to avoid creating an entirely new view. In addition to the current `selection`, various helpers are passed as `options` to the second parameter. This third parameter is the default `onSelectAutocomplete`, which allows you to defer to the original behavior.                                                   |
| onSubmit                      | Function(searchTerm)                                                         | no        |                                                                    |         | Allows overriding behavior when submitted. Receives the search term from the search box.                                                                                                                                                                                                                                                                     |

#### AutocompleteResultsOptions

| Name                    | type          | Required? | Default | Options | Description                                               |
| ----------------------- | ------------- | --------- | ------- | ------- | --------------------------------------------------------- |
| linkTarget              | String        | no        | \_self  |         | Used to open links in a new tab.                          |
| sectionTitle            | String        | no        |         |         | Title to show in section within dropdown.                 |
| shouldTrackClickThrough | Boolean       | no        | true    |         | Only applies to Results, not Suggestions.                 |
| clickThroughTags        | Array[String] | no        |         |         | Tags to send to analytics API when tracking clickthrough. |
| titleField              | String        | yes       |         |         | Field within a Result to use as the "title".              |
| urlField                | String        | yes       |         |         | Field within a Result to use for linking.                 |

#### AutocompleteSuggestionsOptions

| Name         | type   | Required? | Default | Options | Description                              |
| ------------ | ------ | --------- | ------- | ------- | ---------------------------------------- |
| sectionTitle | String | no        |         |         | Title to show in section within dropdown |

---

## Results

Displays all search results.

### Example

```jsx

import { Results } from "@elastic/react-search-ui";

...

<Results titleField="title" urlField="nps_link" />
```

### Configuring search queries

Certain aspects of search results can be configured in `SearchProvider`, using the `searchQuery` configuration, such as
term highlighting and search fields. See the [Advanced Configuration](#advanced-configuration) guide
for more information.

### Properties

| Name                    | type            | Required? | Default                                                  | Options | Description                                                                                                                                          |
| ----------------------- | --------------- | --------- | -------------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className               | String          | no        |                                                          |         |                                                                                                                                                      |
| resultView              | Render Function | no        | [Result](packages/react-search-ui-views/src/Result.js)   |         | Used to override individual Result views. See the Customizing Component views and html section for more information.                                 |
| titleField              | String          | no        |                                                          |         | Name of field to use as the title from each result.                                                                                                  |
| shouldTrackClickThrough | Boolean         | no        | true                                                     |         | Whether or not to track a clickthrough event when clicked.                                                                                           |
| clickThroughTags        | Array[String]   | no        |                                                          |         | Tags to send to analytics API when tracking clickthrough.                                                                                            |
| urlField                | String          | no        |                                                          |         | Name of field to use as the href from each result.                                                                                                   |
| view                    | Render Function | no        | [Results](packages/react-search-ui-views/src/Results.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |

---

## Result

Displays a search result.

### Example

```jsx

import { Result } from "@elastic/react-search-ui";

...

<SearchProvider config={config}>
  {({ results }) => {
    return (
      <div>
        {results.map(result => (
          <Result key={result.id.raw}
            result={result}
            titleField="title"
            urlField="nps_link"
          />
        ))}
      </div>
    );
  }}
</SearchProvider>
```

### Configuring search queries

Certain aspects of search results can be configured in `SearchProvider`, using the `searchQuery` configuration, such as
term highlighting and search fields. See the [Advanced Configuration](#advanced-configuration) guide
for more information.

### Properties

| Name                    | type                                                         | Required? | Default                                                | Options | Description                                                                                                                                          |
| ----------------------- | ------------------------------------------------------------ | --------- | ------------------------------------------------------ | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className               | String                                                       | no        |                                                        |         |                                                                                                                                                      |
| titleField              | String                                                       | no        |                                                        |         | Name of field to use as the title from each result.                                                                                                  |
| shouldTrackClickThrough | Boolean                                                      | no        | true                                                   |         | Whether or not to track a clickthrough event when clicked.                                                                                           |
| clickThroughTags        | Array[String]                                                | no        |                                                        |         | Tags to send to analytics API when tracking clickthrough.                                                                                            |
| urlField                | String                                                       | no        |                                                        |         | Name of field to use as the href from each result.                                                                                                   |
| view                    | Render Function                                              | no        | [Result](packages/react-search-ui-views/src/Result.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |
| result                  | [Result](packages/react-search-ui-views/src/types/Result.js) | no        |                                                        |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |

---

## ResultsPerPage

Shows a dropdown for selecting the number of results to show per page.

Uses [20, 40, 60] as default options. You can use `options` prop to pass custom options.

**Note:** When passing custom options make sure one of the option values match
the current [`resultsPerPage`](#resultsPerPageProp) value, which is 20 by default.
To override `resultsPerPage` default value [refer to the custom options example](#Example-using-custom-options).

### Example

```jsx

import { ResultsPerPage } from "@elastic/react-search-ui";

...

<ResultsPerPage />
```

### Example using custom options

```jsx

import { SearchProvider, ResultsPerPage } from "@elastic/react-search-ui";

<SearchProvider
    config={
        ...
        initialState: {
            resultsPerPage: 5
        }
    }
>
    <ResultsPerPage options={[5, 10, 15]} />
</SearchProvider>
```

### Properties

| Name      | type            | Required? | Default                                                                | Options | Description                                                                                                                                          |
| --------- | --------------- | --------- | ---------------------------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className | String          | no        |                                                                        |         |                                                                                                                                                      |
| options   | Array[Number]   | no        | [20, 40, 60]                                                           |         | Dropdown options to select the number of results to show per page.                                                                                   |
| view      | Render Function | no        | [ResultsPerPage](packages/react-search-ui-views/src/ResultsPerPage.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |

---

## Facet

Show a Facet filter for a particular field.

Must configure the corresponding field in the `SearchProvider` [facets](#advanced-configuration) object.

### Example

```jsx
import { Facet } from "@elastic/react-search-ui";
import { MultiCheckboxFacet } from "@elastic/react-search-ui-views";

...

<SearchProvider config={{
  ...otherConfig,
  searchQuery: {
    facets: {
     states: { type: "value", size: 30 }
    }
  }
}}>
  {() => (
    <Facet field="states" label="States" view={MultiCheckboxFacet} />
  )}
</SearchProvider>
```

### Example of an OR based Facet filter

Certain configuration of the `Facet` Component will require a "disjunctive" facet to work
correctly. "Disjunctive" facets are facets that do not change when a selection is made. Meaning, all available options
will remain as selectable options even after a selection is made.

```jsx
import { Facet } from "@elastic/react-search-ui";
import { MultiCheckboxFacet } from "@elastic/react-search-ui-views";

...

<SearchProvider config={{
  ...otherConfig,
  searchQuery: {
    disjunctiveFacets: ["states"],
    facets: {
     states: { type: "value", size: 30 }
    }
  }
}}>
  {() => (
    <Facet field="states" label="States" view={MultiCheckboxFacet} filterType="any" />
  )}
</SearchProvider>
```

### Properties

| Name         | type            | Required? | Default                                                                        | Options                                                                                                                                                       | Description                                                                                                                                                                                                                                                |
| ------------ | --------------- | --------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| className    | String          | no        |                                                                                |                                                                                                                                                               |                                                                                                                                                                                                                                                            |
| field        | String          | yes       |                                                                                |                                                                                                                                                               | Field name corresponding to this filter. This requires that the corresponding field has been configured in `facets` on the top level Provider.                                                                                                             |
| filterType   | String          | no        | "all"                                                                          | "all", "any", "none"                                                                                                                                          | The type of filter to apply with the selected values. I.e., should "all" of the values match, or just "any" of the values, or "none" of the values. Note: See the example above which describes using "disjunctive" facets in conjunction with filterType. |
| label        | String          | yes       |                                                                                |                                                                                                                                                               | A static label to show in the facet filter.                                                                                                                                                                                                                |
| show         | Number          | no        | 5                                                                              |                                                                                                                                                               | The number of facet filter options to show before concatenating with a "more" link.                                                                                                                                                                        |
| view         | Render Function | no        | [MultiCheckboxFacet](packages/react-search-ui-views/src/MultiCheckboxFacet.js) | [SingleLinksFacet](packages/react-search-ui-views/src/SingleLinksFacet.js) <br/> [SingleSelectFacet](packages/react-search-ui-views/src/SingleSelectFacet.js) [BooleanFacet](packages/react-search-ui-views/src/BooleanFacet.js) | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information.                                                                                                       |
| isFilterable | Boolean         | no        | false                                                                          |                                                                                                                                                               | Whether or not to show Facet quick filter.                                                                                                                                                                                                                 |

---

## Sorting

Shows a dropdown for selecting the current Sort.

### Example

```jsx

import { Sorting } from "@elastic/react-search-ui";

...

<Sorting
  sortOptions={[
    {
      name: "Relevance",
      value: "",
      direction: ""
    },
    {
      name: "Title",
      value: "title",
      direction: "asc"
    }
  ]}
/>
```

### Properties

| Name        | type                                                                  | Required? | Default                                                  | Options | Description                                                                                                                                          |
| ----------- | --------------------------------------------------------------------- | --------- | -------------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className   | String                                                                | no        |                                                          |         |                                                                                                                                                      |
| label       | Array[[SortOption](packages/react-search-ui/src/types/SortOption.js)] | no        |                                                          |         | A static label to show in the Sorting Component.                                                                                                     |
| sortOptions | Array[[SortOption](packages/react-search-ui/src/types/SortOption.js)] | yes       |                                                          |         |                                                                                                                                                      |
| view        | Render Function                                                       | no        | [Sorting](packages/react-search-ui-views/src/Sorting.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |

---

## Paging

Navigate through pagination.

### Example

```jsx

import { Paging } from "@elastic/react-search-ui";

...

<Paging />
```

### Properties

| Name      | type            | Required? | Default                                                | Options | Description                                                                                                                                          |
| --------- | --------------- | --------- | ------------------------------------------------------ | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className | String          | no        |                                                        |         |                                                                                                                                                      |
| view      | Render Function | no        | [Paging](packages/react-search-ui-views/src/Paging.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |

---

## PagingInfo

Paging details, like "1 - 20 of 100 results".

### Example

```jsx

import { PagingInfo } from "@elastic/react-search-ui";

...

<PagingInfo />
```

### Properties

| Name      | type            | Required? | Default                                                        | Options | Description                                                                                                                                          |
| --------- | --------------- | --------- | -------------------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className | String          | no        |                                                                |         |                                                                                                                                                      |
| view      | Render Function | no        | [PagingInfo](packages/react-search-ui-views/src/PagingInfo.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |

---

## ErrorBoundary

Handle unexpected errors.

### Example

```jsx
import { ErrorBoundary } from "@elastic/react-search-ui";

...

<ErrorBoundary>
  <div>Some Content</div>
</ErrorBoundary>
```

### Properties

| Name      | type            | Required? | Default                                                              | Options | Description                                                                                                                                          |
| --------- | --------------- | --------- | -------------------------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| className | String          | no        |                                                                      |         |                                                                                                                                                      |
| children  | React node      | yes       |                                                                      |         | Content to show if no error has occurred, will be replaced with error messaging if there was an error.                                               |
| view      | Render Function | no        | [ErrorBoundary](packages/react-search-ui-views/src/ErrorBoundary.js) |         | Used to override the default view for this Component. See [Customization: Component views and HTML](#component-views-and-html) for more information. |
