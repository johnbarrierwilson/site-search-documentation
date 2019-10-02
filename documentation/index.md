---
title: "API Overview"
slug: "/"
---

# API Overview

The **Site Search API** can be used to programmatically alter how your...

+ pages are crawled
+ documents are indexed
+ searches are completed
+ analytics are generated
+ ... and more!

This overview covers:

* [Language Optimization](#language-optimization)
* [Authentication](#authentication)
* [Engine Types: Crawler-based v. API-based](#engine_types)
* [Search: Public v. Private](#pub-v-priv)
* [Response Object](#response-object)
* [Free-text Query Syntax](#freetext_query_syntax)
* [Parameter Encoding](#parameter_encoding)
* [Summary of API endpoints](#api)
* [API Resources](#resources)

## Language Optimization [#language-optimization]

Each Engine can be optimized for a specific language.

By default, an Engine is considered Universal, or `null` according to the Engine's `language` parameter.

You can specify a language when creating an Engine via the dashboard or the API.

A language will match with its respective **Language Code**.

The codes adhere to an IETF subset: [RFC 5646](https://tools.ietf.org/html/rfc5646), which coincide with [ISO 639-1](https://en.wikipedia.org/wiki/ISO_639-1) and [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1).

Note that once a language has been configured, it cannot be reconfigured -- you will need to create a new Engine and transport your data.

{% table_block %}
  {% row **Language** | **Language Code, [ISO 639-1](https://en.wikipedia.org/wiki/ISO_639-1) and [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1).** %}
  {% row "Brazilian Portuguese" | `pt-br` %}
  {% row “Chinese" | `zh` %}
  {% row “Dutch" | `nl` %}
  {% row “English" | `en` %}
  {% row "French" | `fr` %}
  {% row “German" | `de` %}
  {% row "Italian" | `it` %}
  {% row "Japanese" | `ja` %}
  {% row "Korean | `ko` %}
  {% row “Portuguese" | `pt` %}
  {% row “Russian" | `ru` %}
  {% row “Spanish" | `es` %}
  {% row “Thai" | `th` %}
  {% row "Universal" | `null` %}
{% endtable_block %}

## Authentication [#authentication]

You may authenticate requests to the API using the following method:

<table class="table table-bordered">
  <thead>
    <tr>
      <th style="width: 120px;">Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Authentication Token</td>
      <td>
        Add the <code>auth_token</code> or <code>engine_key</code> parameter to each request. They take your <b>API Key</b> and your <b>Engine Key</b>, respectively.
      </td>
    </tr>
    <tr>
      <td>HTTP Basic Auth</td>
      <td>Use your <code>auth_token</code> or <code>engine_key</code> as the username. Do not include a password.</td>
    </tr>
  </tbody>
</table>

In addition to your secret credential, an **Engine Slug** or **Engine Key** is needed to make private API calls.

When you name an Engine we will assign your Engine a slug based on the name you provided.

eg: An Engine named **"Bookstore Search Engine"** would get the slug **"bookstore-search-engine"**.

You will use the slug as the `engine_id` to reference a particular Engine when sending requests to the API.

## Engine Types: Crawler-based v. API-based [#engine_types]

All Engine's are crawler Engines unless they are created via the [Engines API](/documentation/site-search/engines).

**You are unable to change the Engine type once it has been created.**

You can index your documents from one into the other, should you want to make the change.

<table class="table table-bordered">
    <thead>
      <tr>
        <th>Endpoint</th>
        <th>Supported Engines</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><a href="/documentation/site-search/indexing"> Document Indexing</a></td>
        <td><strong>API-based Engine only.</strong></td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/api-crawler-operations"> Crawler Operations</a></td>
        <td><strong>Crawler-based Engine only.</strong></td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/engines"> Engines</a></td>
        <td><strong>Account level.</strong></td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/searching"> Search</a></td>
        <td>Both.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/autocomplete"> Autocomplete</a></td>
        <td>Both.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/analytics"> Analytics</a></td>
        <td>Both.</td>
      </tr>
      </tbody>
</table>
</br>

## Search: Public v. Private [#pub-v-priv]

The [search](/documentation/site-search/searching) API endpoint has two available querying methods:

1. **Public** method
2. **Private** method

_Note: The documentation demonstrates **public** search queries._

#### Public Search Method [#public-search-method]

Public search uses the public **Engine Key**, a unique, read-only key associated with your Engine.

Public search requests require that the `engine_key` field contain the **Engine Key**.

All requests made with this key are **read only**.

It is used when:

* You are writing mobile applications.
* You are using client side JavaScript.
* Any case where you are comfortable with query information being exposed through a client.

{% endpoint GET "https://search-api.swiftype.com/api/v1/public/engines/search.json" %}
{% endpoint POST "https://search-api.swiftype.com/api/v1/public/engines/search.json" %}
</br>

{% examplecode %}
  {% description %}
    Using the <strong>public</strong> search method to search within the `bookstore` Engine for the query "brothers" across two DocumentTypes.
  {% code bash %}
  curl -XGET 'https://search-api.swiftype.com/api/v1/public/engines/search.json' \
    -H 'Content-Type: application/json' \
    -d '{
          "engine_key": "YOUR_ENGINE_KEY",
          "q": "brothers",
          "document_types": ["magazines", "comics"]
        }'
  {% endcode %}
{% endexamplecode %}

#### Private Search Method [#private-search-method]

Private search uses the private **API Key**.

The API Key a unique, permissive key which can be used as a credential against **all** API endpoints.

Private search requests require that the `auth_token` parameter contain the **API Key**.

The **Engine Key** is passed in via the URL to identify a specific Engine.

The **API Key** will perform reads during search, but can be used to write to other endpoints.

**It should be used with caution.**

It is used when:

* You want to use a single key for all API operations.
* Proxying requests through your infrastructure to hide queries from the client. [1]

{% endpoint GET "https://search-api.swiftype.com/api/v1/engines/{engine_id}/search.json" %}
{% endpoint POST "https://search-api.swiftype.com/api/v1/engines/{engine_id}/search.json" %}
</br>

{% examplecode %}
  {% description %}
    Using the <strong>private</strong> search method to search within the `bookstore` Engine for the query "brothers" across two DocumentTypes.
  {% code bash %}
  curl -XGET 'https://search-api.swiftype.com/api/v1/engines/bookstore/search.json' \
    -H 'Content-Type: application/json' \
    -d '{
          "auth_token": "YOUR_API_KEY",
          "q": "brothers",
          "document_types": ["magazines", "comics"]
        }'
  {% endcode %}
{% endexamplecode %}

[1] If you have search data you need to keep hidden, we recommend routing all search requests through your own servers via the private method. Read more about [protecting sensitive data](https://swiftype.com/questions/my-search-index-contains-sensitive-data-how-can-i-prevent-it-from-being-exposed).

## Response Object [#response-object]

Search results will contain an object with the following key/value parameters.

<table class="table table-bordered">
  <thead>
    <tr>
      <th>Key</th>
      <th>Value</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>records</code></td>
      <td>Hash of DocumentType slug to Array of result objects</td>
      <td>Contains the search results for each DocumentType.</td>
    </tr>
    <tr>
      <td><code>info</code></td>
      <td>Hash of DocumentType slug to a result info object</td>
      <td>Contains query metadata like number of results and facet counts.</td>
    </tr>
    <tr>
      <td><code>errors</code></td>
      <td>Hash of option name to Hash or Array of errors</td>
      <td>Contains details about query options that were used incorrectly. See <a href="#errors">Error messages</a> for more details.</td>
    </tr>
  </tbody>
</table>

In application, a response will appear as such:

```JSON
{
"records": {
  "books": [
    {
      "title": "The Brothers Karamazov",
      "author": "Fyodor Dostoyevsky",
      "price": "14.95"
    },
    ...
  ]
},
"info": {
  "books": {
    "query": "brothers",
    "current_page": 1,
    "num_pages": 1,
    "per_page": 20,
    "total_result_count": 18,
    "facets": {}
  }
},
"errors": {}
}
```

## Free-text Query Syntax [#freetext_query_syntax]

Text searches support a basic subset of the standard Lucene query syntax.

The supported functions are: double quoted strings, + and -, AND, OR, and NOT.

Visit the [Lucene documentation](http://lucene.apache.org/core/old_versioned_docs/versions/3_0_0/queryparsersyntax.html) for more information.

## Parameter Encoding [#parameter_encoding]

All **Site Search API** endpoints accept parameters as a JSON-encoded request body.

This is the recommended method and is shown in the examples in this documentation.

However, you can also submit parameters using form encoding.

For example, the JSON parameter:

```json
{
  "filters":
    {
      "videos":
      {
        "status": ["draft", "published"]
      }
    }
}
```

...is equivalent to:

```
/filters[videos][status][]=draft&filters[videos][status][]=published
```

## Summary of API Endpoints [#api]

Site Search offers a variety of useful endpoints...

<table class="table table-bordered">
    <thead>
      <tr>
        <th>Endpoint</th>
        <th>Details</th>
        <th>Engine Support</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><a href="/documentation/site-search/indexing">Document Indexing</a></td>
        <td>Add, remove, delete, or update documents. There are methods for bulk indexing, asynchronous indexing, and timed document expiry.</td>
        <td><strong>API-based Engines only.</strong></td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/api-crawler-operations">  Crawler Operations</a></td>
        <td>Add domains to your Engine, recrawl a domain, recrawl a URL -- this endpoint puts you in control of the <a href="/documentation/site-search/crawler">Site Search Crawler</a>.</td>
        <td><strong>Crawler-based Engines only.</strong></td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/engines">Engines</a></td>
        <td>Create an Engine, list Engines, and destroy Engines.</td>
        <td><strong>Account level.</strong></td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/searching">Search</a></td>
        <td>The endpoint of endpoints! Search a broad range of parameters to generate diverse search queries and relevant result sets.</td>
        <td>Both.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/autocomplete">Autocomplete</a></td>
        <td>Performs prefix matching to generate suggestions as the user is typing queries. With this endpoint, you can guide your users to the most relevant search keywords.</td>
        <td>Both.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/analytics">Analytics</a></td>
        <td>Search is powerful. But the insights that searches generate can help you get ahead of your users expectations. Provides useful insights into clicks and queries.</td>
        <td>Both.</td>
      </tr>
      </tbody>
</table>
</br>

## API Resources [#resources]

There are 3 primary resources of the **Site Search API**:

1. Engine
2. DocumentType
3. Document

<div class="image-center">
  <img src="/documentation/images/api/engine-diagram.png"/>
</div>

#### Engine [#engine]

An **Engine** is the top-level object in your search index.

It has a freeform name field which is translated into a slug identifier and used as the `ENGINE_ID`.

You must reference the Engine in all API requests.

#### DocumentType [#documenttype]

A **DocumentType** specify the structure of a set of documents in the Engine.

They act as entry points for searches.

They contain multiple fields types: string, text, enum, integer, float, date, and location.

The default DocumentType is called `page`. [Read more](/documentation/site-search/crawler-overview#schema).

#### Document [#document]

**Documents** represent all of the pieces of content in an Engine.

They are children of a DocumentType and conform to its field specification.

You do not need to specify the fields ahead of time, they will be inferred by a document contents.

All new fields will default to the `text` type.

When you perform a search on a DocumentType, you will receive document results.

The only required document field is `external_id`.

It can be any value, eg: the numeric ID you use to identify the object in your own data store.

## Document Field Types [#field_types]

Documents in your Engine may contain as many fields as you like.

There are three tables to help you navigate Field Types:

* [Field Types](#fieldtype)
* [Use Cases](#use-cases)
* [Specifications](#specifications)

### Field Types [#fieldtype]

To decide which field types to use, consider the tables below.

Another helpful resource is the [Site Search schema design guide](/documentation/site-search/guides/schema-design).

<table class="table table-bordered">
 <thead>
   <tr>
     <th>Field Type</th>
     <th>Description</th>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td><code>string</code></td>
     <td>Smaller pieces of text, such as a book title. These are used for autocomplete and full-text search. String fields can also be used for filtering, faceting, and sorting</td>
   </tr>
   <tr>
     <td><code>text</code></td>
     <td>Large bodies of text, such as a book chapter. These are only used for full-text search.</td>
   </tr>
   <tr>
     <td><code>enum</code></td>
     <td>String attributes of a document that should be used for exact comparisons, such as a URL or the genre of a book. Enum fields can be used for filtering, faceting, and sorting. Exact matches will be included in results of autocomplete and full-text search.</td>
   </tr>
   <tr>
     <td><code>integer</code></td>
     <td>An integer value, such as the number of sales of a book (e.g. 1234).</td>
   </tr>
   <tr>
     <td><code>float</code></td>
     <td>A floating point value, such as the price of a book (e.g. 3.99).</td>
   </tr>
   <tr>
     <td><code>date</code></td>
     <td>ISO 8601 compatible time strings, such as the publication date of a book.</td>
   </tr>
   <tr>
     <td><code>location</code></td>
     <td>A geographic location specified by latitude and longitude (e.g. lat: 53.2, lon: 27.6).</td>
   </tr>
 </tbody>
</table>

#### Field Type Use Cases [#use-cases]

<table class="table table-bordered">
 <thead>
   <tr>
     <th>Field Type</th>
     <th>Search</th>
     <th>Optimized for Autocomplete</th>
     <th>Functional Boosts</th>
     <th>Filtering</th>
     <th>Sorting</th>
     <th>Facets</th>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td><code>string</code></td>
     <td>Yes</td>
     <td>Yes</td>
     <td>No</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
   </tr>
   <tr>
     <td><code>text</code></td>
     <td>Yes</td>
     <td>No</td>
     <td>No</td>
     <td>No</td>
     <td>No</td>
     <td>No</td>
   </tr>
   <tr>
     <td><code>enum</code></td>
     <td>Yes</td>
     <td>No</td>
     <td>No</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
   </tr>
   <tr>
     <td><code>integer</code></td>
     <td>No</td>
     <td>No</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
   </tr>
   <tr>
     <td><code>float</code></td>
     <td>No</td>
     <td>No</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
   </tr>
   <tr>
     <td><code>date</code></td>
     <td>No</td>
     <td>No</td>
     <td>No</td>
     <td>Yes</td>
     <td>Yes</td>
     <td>Yes</td>
   </tr>
   <tr>
     <td><code>location</code></td>
     <td>No</td>
     <td>No</td>
     <td>No</td>
     <td>Yes</td>
     <td>No</td>
     <td>No</td>
   </tr>
 </tbody>
</table>

#### Field Type Specifications [#specifications]

<table class="table table-bordered">
 <thead>
   <tr>
     <th>Type</th>
     <th>Name</th>
     <th>Value</th>
     <th>JSON</th>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td><code>string</code></td>
     <td>title</td>
     <td>"My Post Title"</td>
     <td><code class="json_example">{"type":"string","name":"title","value":"My Post Title"}</code></td>
    </tr>
   <tr>
     <td><code>text</code></td>
     <td>body</td>
     <td>"This is the long content of my post..."</td>
     <td><code class="json_example">{"type":"text","name":"body","value":"This is the long content of my post..."}</code></td>
    </tr>
   <tr>
     <td><code>enum</code></td>
     <td>public</td>
     <td>true</td>
     <td><code class="json_example">{"type":"enum","name":"public","value":true}</code></td>
    </tr>
   <tr>
     <td><code>integer</code></td>
     <td>views</td>
     <td>387</td>
     <td><code class="json_example">{"type":"integer","name":"views","value":387}</code></td>
    </tr>
   <tr>
     <td><code>float</code></td>
     <td>price</td>
     <td>6.95</td>
     <td><code class="json_example">{"type":"float","name":"price","value":6.95}</code></td>
    </tr>
   <tr>
     <td><code>date</code></td>
     <td>posted_at</td>
     <td>"2012-07-02T20:44:05-07:00"</td>
     <td><code class="json_example">{"type":"date","name":"posted_at","value":"2012-07-02T20:44:05-07:00"}</code></td>
    </tr>
   <tr>
     <td><code>location</code></td>
     <td>posted_from</td>
     <td>{"lat":56.2,"lon":44.7}</td>
     <td><code class="json_example">{"type":"location","name":"posted_from","value":{"lat":56.2,"lon":44.7}}</code></td>
    </tr>

 </tbody>
</table>

#### Array values [#array]

When creating a document, pass the values as an array.

```
{"type": "enum", "name": "tags", "value": ["tag1", "tag2", "tag3"]}
```

#### Naming fields [#naming-field]

Document field names must be limited to alphanumeric ASCII characters.  Whitespace, special characters, and non-ASCII encoding are not allowed.

---

Stuck? Looking for help? [Contact support](mailto:support@swiftype.com) or check out the [Site Search community forum](https://discuss.elastic.co/c/site-search)!
