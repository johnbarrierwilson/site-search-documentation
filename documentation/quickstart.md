---
title: "API Quick Start"
slug: "/quickstart"
---

# Site Search API Quick Start

An API-based Engine does not use the crawler to index documents.

Instead, you index documents via an API endpoint.

**Engines created using this quick start are API-based Engines**.

<table class="table table-bordered">
    <thead>
      <tr>
        <th>Endpoint</th>
        <th>Supported?</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><a href="/documentation/site-search/indexing"> Document Indexing</a></td>
        <td>Yes.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/api-crawler-operations"> Crawler Operations</a></td>
        <td>No.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/engines"> Engines</a></td>
        <td>Engines are account level.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/searching"> Search</a></td>
        <td>Yes.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/autocomplete"> Autocomplete</a></td>
        <td>Yes.</td>
      </tr>
      <tr>
        <td><a href="/documentation/site-search/analytics"> Analytics</a></td>
        <td>Yes.</td>
      </tr>
      </tbody>
</table>
</br>

## 1. Create an Account [#create-account]

Before getting started, you'll need to [create a Site Search account](/users/sign_up) and confirm your email address.

Create a new API Engine. The callout is under text box for URL entry.

The Site Search API uses an **API Key** to authenticate requests.

You can find it in the corner of your **Account Settings** page.

If you're already signed up, you can also use the API to create a new Engine:

```bash
curl -XPOST 'https://api.swiftype.com/api/v1/engines.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "engine":{"name":"bookstore"}
      }'
```

We will use a `bookstore` as an example Engine.

## 2. Create a Foundation [#create-foundation]

We will create a DocumentType for each type of object we'd like to make searchable.

In other words, a DocumentType is a top-level directory for different segments of search engine content.

Since this is a bookstore, we'll create DocumentTypes for both `books` and `magazines`.

```bash
curl -XPOST 'https://api.swiftype.com/api/v1/engines/bookstore/document_types.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "document_type":{"name":"books"}
      }'
```

```bash
curl -XPOST 'https://api.swiftype.com/api/v1/engines/bookstore/document_types.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "document_type":{"name":"magazines"}
      }'
```

## 3. Indexing [#indexing]

Your Engine is prepared!

Now to index a few documents in the `books` DocumentType.

```bash
curl -XPOST 'https://api.swiftype.com/api/v1/engines/bookstore/document_types/books/documents.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "document": {
          "external_id": "1",
          "fields": [
            {"name": "title", "value": "The Great Gatsby", "type": "string"},
            {"name": "author", "value": "F. Scott Fitzgerald", "type": "string"},
            {"name": "genre", "value": "fiction", "type": "enum"}
          ]
        }
      }'

curl -XPOST 'https://api.swiftype.com/api/v1/engines/bookstore/document_types/books/documents.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "document": {
          "external_id": "2",
          "fields": [
            {"name": "title", "value": "The Brothers Karamazov", "type": "string"},
            {"name": "author", "value": "Fyodor Dostoevsky", "type": "string"},
            {"name": "genre", "value": "fiction", "type": "enum"}
          ]
        }
      }'
```

And a few documents into the `magazines` DocumentType...

```bash
curl -XPOST 'https://api.swiftype.com/api/v1/engines/bookstore/document_types/magazines/documents.json' \

-H 'Content-Type: application/json' \
-d '{
      "auth_token":"YOUR_API_KEY",
      "document": {
        "external_id": "1",
        "fields": [
          {"name": "name", "value": "The Economist", "type": "string"},
          {"name": "category", "value": "news-current-affairs", "type": "enum"},
          {"name": "periodicity", "value": "weekly", "type": "enum"}
        ]
      }
    }'

curl -XPOST 'https://api.swiftype.com/api/v1/engines/bookstore/document_types/magazines/documents.json' \
-H 'Content-Type: application/json' \
-d '{
      "auth_token":"YOUR_API_KEY",
      "document": {
        "external_id": "2",
        "fields": [
          {"name": "name", "value": "Dwell", "type": "string"},
          {"name": "category", "value": "interior-design", "type": "enum"},
          {"name": "periodicity", "value": "weekly", "type": "enum"}
        ]
      }
    }'
```

We only used the **string** and **enum** field types, but the API supports many additional types.

See [API Overview, Field Types](overview#field_types) for details.

The documents are in the Engine -- now, time to search.

## 4. Searching [#searching]

 By default, search queries will match any fields that are of the type **string** or **text**.

 You may search each DocumentType individually:

```bash
curl -XGET 'https://search-api.swiftype.com/api/v1/engines/bookstore/document_types/books/search.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "q": "brothers"
      }'

curl -XGET 'https://search-api.swiftype.com/api/v1/engines/bookstore/document_types/magazines/search.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "q":"dwell"
      }'
```

Or you may search the entire Engine, over all DocumentTypes at once:

```bash
curl -XGET 'https://search-api.swiftype.com/api/v1/engines/bookstore/search.json' \
  -H 'Content-Type: application/json' \
  -d '{
        "auth_token":"YOUR_API_KEY",
        "q":"the"
      }'
```

## 5. Autocomplete Searches [#autocomplete]

Finally, as with full-text searches, you may perform autocomplete (prefix match) searches as well:

```bash
curl -XGET 'https://search-api.swiftype.com/api/v1/engines/bookstore/suggest.json' \
  -H 'Content-Type: application/json' \
  -d '{
       "auth_token":"YOUR_API_KEY",
       "q":"fitz"
     }'
```

### What next?

We have plenty of resources to help you get the most out of your Engine:

* Checkout our [clients, libraries, and SDKs](https://swiftype.com/documentation/site-search/clients).
* Explore [Synonyms](/documentation/site-search/guides/synonyms), [Result Rankings](/documentation/site-search/guides/result-rankings), and [Weights](/documentation/site-search/guides/weights) guides to master relevance tuning.

---

Stuck? Looking for help? [Contact support](mailto:support@swiftype.com) or check out the [Site Search community forum](https://discuss.elastic.co/c/site-search)!
