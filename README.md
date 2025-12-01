# STAC API - Collection Search Endpoint

- **Title:** Collection Search Endpoint
- **Conformance Classes:**
  - `https://api.stacspec.org/v1.0.0-beta.1/collection-search-endpoint`
- **Scope:** STAC API - Core
- **Extension Maturity Classification:** Proposal
- **Dependencies:**
  - [STAC API - Core](https://github.com/radiantearth/stac-api-spec/blob/main/core)
  - [STAC API - Collection Search](https://github.com/stac-api-extensions/collection-search)
- **Owner**: @jonhealy1

## Introduction

This extension defines a dedicated **Collection Search** endpoint (`/collections-search`) that supports both `GET` and `POST` methods.

It provides a safe, conflict-free mechanism to perform advanced searching (filtering, sorting) on Collections without colliding with the **Transaction Extension** on the root `/collections` path.

## Motivation

The standard [STAC API - Collection Search](https://github.com/stac-api-extensions/collection-search) extension relies on the `/collections` endpoint. While this works well for simple `GET` requests, it introduces a significant architectural conflict when `POST` is required for advanced search (e.g., complex CQL filters or large bodies).

As noted in the official Collection Search documentation:
> *"STAC may add behavior for POST /collections in the future, but due to a potential conflict with the Transaction Extension, specific rules for content negotiation might be required."*

In practice, implementing Content-Type negotiation to distinguish between "Creating a Collection" (Transaction) and "Searching Collections" (Search) on the same URL (`POST /collections`) is difficult for many API frameworks and confusing for client libraries.

This extension resolves the conflict by moving search operations to a dedicated resource: **`/collections-search`**.

## Endpoints

| Method | URI | Description |
| :--- | :--- | :--- |
| `GET` | `/collections-search` | **Simple Search.** Retrieves collections matching query parameters. Functionally identical to `GET /collections`. |
| `POST` | `/collections-search` | **Advanced Search.** Retrieves collections matching the JSON body filter. |

## Query Parameters and Fields

This extension supports all query parameters and extension behaviors defined in the standard [Collection Search](https://github.com/stac-api-extensions/collection-search) specification, including:
* **Filter** (CQL2)
* **Sort**
* **Fields**
* **Free-Text Search** (`q`)

### GET Request
Behaves exactly like the standard `GET /collections` endpoint, accepting parameters via the URL.

```http
GET /collections-search?limit=10&q=sentinel&sortby=-created
```

### POST Request
Accepts a JSON body containing search parameters. This allows for complex queries that exceed URL length limits or require structured data.

#### Header: `Content-Type: application/json`

#### Body Schema:

```json
{
  "q": ["sentinel", "radar"],
  "datetime": "2020-01-01/2021-01-01",
  "limit": 10,
  "sortby": [
    { "field": "created", "direction": "desc" }
  ],
  "filter": {
    "op": "and",
    "args": [
      { "op": "=", "args": [{ "property": "platform" }, "sentinel-1"] }
    ]
  }
}
```

### Response
The response format for both GET and POST is a standard STAC Collections object.

```json
{
  "collections": [
    {
      "type": "Collection",
      "id": "sentinel-1-grd",
      "stac_version": "1.0.0",
      "description": "Sentinel-1 Ground Range Detected",
      "links": [],
      "extent": {
        "spatial": { "bbox": [[-180, -90, 180, 90]] },
        "temporal": { "interval": [["2014-10-03T00:00:00Z", null]] }
      }
    }
  ],
  "links": [
    {
      "rel": "root",
      "href": "https://api.example.com/"
    },
    {
      "rel": "self",
      "href": "https://api.example.com/collections-search"
    }
  ]
}
```

### Relation to Item Search 
This endpoint mirrors the design pattern of **Item Search**, creating symmetry in the API:
Resource,Simple Search (GET),Advanced Search (POST)
Items,/search,/search
Collections,/collections-search,/collections-search

