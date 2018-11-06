# Optional: Content Creation and Editing Extensions

## About

This page provides optional extensions to the DTS specification. The core specification is focused on consumption (retrieval) of documents and their metadata. These extensions provide for creating, updating, and deleting documents, sections of documents, and their metadata (full CRUD). An API that implements the DTS specification need not include these extensions if it will not be allowing these other kinds of interaction. If an implementation does allow for more than just consumption of data, however, it should do so using the relevant extensions.

## Collections Endpoint: Optional Methods

In addition to retrieving metadata on collections via the GET method, implementations may also support creation and management of collections through the methods POST, PUT, and DELETE.

This documentation assumes that you have read the core specification for the Collections endpoint.

### Extended URI

#### Extended Collection Query Parameters

Note that the core `page` and `nav` parameters are not accepted for any of the extended request methods. One additional parameter, `parent`, must be accepted if the POST method is supported.

Assuming that the implementation wants to maintain control over who can create and modify server data, the new `token` parameter also allows for token-based authentication (as with OAuth 2.0) if desired. It is up to the implementation to decide how such tokens should be generated and processed.

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a collection or document. Spaces and other problematic characters should be url encoded. |  GET, PUT, DELETE |
| page | page of the current collections members  |  GET    |
| nav  | whether members of the collection are its `children` (default)  or `parents` | GET |
| parent | identifier for the collection under which a new collection or document is to be created. Spaces and other problematic characters in the identifier should be url encoded. | POST |
| token | authentication token for access control | POST, PUT, DELETE |

#### Extended URI Collections Template

The following template is returned at the base collection endpoint, instructing clients how to build valid URL requests. The route itself (here `/dts/api/collections/`) is up to the implementer.
<!---TODO: Is there a way of indicating in the template which methods are supported for which parameters?--->

```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/collection/?id={collection_id}&page={page}&nav={nav}&parent={parent&token={token}}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "page",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "nav",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "parent",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "token",
      "required": false
    }    
  ]
}
```

### POST on the Collections Endpoint

The POST method of the Collections endpoint allows for creation of the metadata record for a new collection or new readable resource (i.e., document) within a collection. Only one top-level item (collection and/or resource) may be created in a single POST request, but this request may simultaneously create any number of children.

#### POST Query parameters

There are no required query parameters for a POST request to the Collections endpoint. The one optional parameter is `parent`. This should be the unique identifier of an existing collection. If a `parent` is supplied, then the new item will be created as a child member of that collection. Otherwise the new item will be created at the top level of the target repository.

The other parameter that may be accepted is `token`. This parameter carries the client's authentication token.

#### POST Request Body

The request body must be valid JSON-LD, following the scheme set out in the core specification for the Collections endpoint. The minimum required terms for each item to be created are:

| Term  | Description    |
| ----- |--------------- |
| `title`         | a single string                                |
| `@id`           | the unique identifier of the item being created|
| `@type`         | a string with one of the two values "Collection" and "Resource"                  |
| `totalItems`    | the number of children contained by the new item at its creation |
| `dts:citeDepth` | the maximum depth of a readable resource (required only when creating a Resource item). |

If children of the new top-level item are to be created, their metadata should be included as a list under the `member` term.

The JSON object must also begin with the `@context` property, setting the default vocabulary to Hydra and providing DCT, TEI and DTS namespace prefixes. This only needs to be included once at the top level of the JSON object. If children are included under `member`, the `@context` is not repeated for each child.

#### POST responses

##### Status codes

A successful POST request should return the status code `201(Created)`.

If a POST request is unsuccessful because of problems with the request content (i.e., parameters or request body), then the return status code should be `400(Bad Request)` or a custom status code in the 4XX series signaling a more specific error.

If a POST request is unsuccessful because the specified item `@id` is already in use, then the return status code should be `409(Conflict)`. An implementation must never allow the creation of items with duplicate `@id` values.

##### Successful response content

The response headers after a successful POST request should include a `Location` value. This should be the URL where the newly created record can be retrieved via a GET request. The response should also include a `Content-type` header with a value of "application/ld+json".

The response body after a successful POST request should contain a JSON-LD object representing the newly created metadata. This should be identical to the response a client would get using a GET request to the `Location` indicated in the response header. This will also mean that in most successful requests the JSON-LD contained in the response body will be identical to the object sent by the client in the POST.

##### Unsuccessful response content

In an unsuccessful request, the response headers should include a `Location` whose value is the URL for the Collections endpoint documentation. The response should also include a `Content-type` header with a value of "application/ld+json".

If a response returns an error code, the response body should contain a JSON-LD object following the hydra specification for status responses (https://www.hydra-cg.com/spec/latest/core/#description-of-http-status-codes-and-errors). For example, this would be a well-formed response body for a `400(Bad Request)` error:

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "Status",
  "statusCode": 400,
  "title": "Improperly Formet Request Body",
  "description": "The body of a POST request to the Collections endpoint must be properly formed JSON-LD."
}
```

For a `400(Bad Request)` error, the `description` should provide as much information as possible about which submitted data was missing or unacceptable. It should indicate whether:
- there were missing required query parameters
- any query parameters held unacceptable values
- the request body did not have properly formed JSON-LD
- the request body was properly formed but carried unacceptable values

If a response returns a status of `409(Conflict)` then the JSON `description` value should indicate that the requested `@id` already exists and cannot be created.

#### POST Example 1: Creating an Empty Top-Level Collection

In this example we will create a new, empty, top-level collection on a server where the Collection endpoint is at `/api/dts/collections`. The newly created collection will be identified by the string "general" and will have the title "Collection Générale de l'École Nationale des Chartes".

##### POST request URL

- `/api/dts/collections/?token=XXXXX`

##### POST request body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 0,
    "title": "Collection Générale de l'École Nationale des Chartes",
}
```
##### successful POST response status

- 201(Created)

##### successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/collections?id=general |
| Content-Type  | application/ld+json             |

##### successful POST response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 0,
    "title": "Collection Générale de l'École Nationale des Chartes",
}
```


#### POST Example 2: Creating a New Child Document in an Existing Collection

In this example we will create a new Readable Collection (i.e., a textual Resource) inside an existing collection "lasciva_roma". Again, this presumes that the Collection endpoint is found at `/api/dts/collections`.

##### POST request URL

- `/api/dts/collections/?parent=lasciva_roma&token=XXXXX`

##### POST request body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type": "Resource",
    "totalItems": 0,
    "title" : "Priapeia",
    "dts:citeDepth" : 2
}
```

##### successful POST response status

- 201(Created)

##### successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/collections?id=urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1 |
| Content-Type  | application/ld+json             |

##### successful POST response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "urn:cts:latinLit:phi1103.phi001.lascivaroma-lat1",
    "@type": "Resource",
    "totalItems": 0,
    "title" : "Priapeia",
    "dts:citeDepth" : 2
}
```


### PUT on the Collections Endpoint
The `PUT` method of the Collection endpoint allows for modification of the metadata for an existing collection or document. Only one item (collection or resource) may be modified in a single `PUT` request.

The request body must be valid JSON-LD. The simplest way to ensure this is by first retrieving a JSON-LD object (using `GET`) that represents the collection or document to be modified. The client may then modify that JSON-LD object and return it in the body of the `PUT` request.

#### PUT Query Parameters

The one required parameter for a `PUT` request is `id`. This should be the unique identifier of the collection or resource that the client wants to modify.

The other parameter that may be accepted is `token` which carries the client's authentication token.

#### PUT Request Body

The request body must be valid JSON-LD, following the scheme set out in the core specification for the Collection endpoint. There are, however, no minimum necessary terms to include in a PUT request (as there are in POST requests). The client may opt to include in the JSON-LD object submitted only those terms that are being modified. Terms left out of the submitted object are not deleted but simply left unchanged on the server. The submitted JSON-LD object still must, however, include the `@context` property, setting the default vocabulary to Hydra and providing DCT, TEI and DTS namespace prefixes.

If a client wishes to delete the value of a term, the client must include that term in the submitted JSON object but assign it an empty string ("") as its value. This API specification does not provide for complete removal of the term itself from a record.

#### PUT Responses

##### Status codes

If the specified item is updated successfully, the response status should be `200(OK)`.

If no item exists with the `@id` specified in the request URL, the response status should be `404(Not Found)`. If there is some other problem with the request data, the response status should be `400(Bad Request)`.

##### Successful response content

The response headers after a successful PUT request should include a `Location` value. This should be the URL where the newly created record can be retrieved via a GET request. The response should also include a `Content-type` header with a value of "application/ld+json".

In a successful PUT request, the response body should be a JSON-LD object representing the edited item. This object should always include the `@context` and `@id` properties. Aside from those, however, this object should only include those term/value pairs that were modified in the transaction. This allows the client to quickly recognize whether the correct information was updated on the server.

##### Unsuccessful response content

In an unsuccessful request, the response headers should include a `Location` whose value is the URL for the Collections endpoint documentation. The response should also include a `Content-type` header with a value of "application/ld+json".

If a response returns an error code, the response body should contain a JSON-LD object following the hydra specification for status responses (https://www.hydra-cg.com/spec/latest/core/#description-of-http-status-codes-and-errors). For example, this would be a well-formed response body for a `404(Not Found)` error:

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "Status",
  "statusCode": 404,
  "title": "Not Found",
  "description": "No item exists with the id specified in your request."
}
```
If a response returns a status of `404(Not Found)` then the `description` value should indicate that the requested `@id` value does not exist and so the requested item cannot be modified.

If the status code is `400(Bad Request)` then the `description` should clarify which parts of the request data were not acceptable.

#### PUT Example 1: Editing an Existing Collection

In this example we will use a `PUT` request to modify the metadata for the collection with the `id` value "general." We will shorten the `title` value to read just "Collection Générale".

##### PUT request URL

- `/api/dts/collections/?id=general&token=XXXXX`

##### PUT request body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "title": "Collection Générale",
}
```
##### Successful PUT response status

- `200(OK)`

##### Successful PUT response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/collections?id=general |
| Content-Type  | application/ld+json             |

##### successful PUT response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "title": "Collection Générale",
}
```

### DELETE on the Collections Endpoint  
The `DELETE` method of the Collection endpoint allows for removal of a collection or resource. Only one item (collection or resource) may be removed in a single `DELETE` request.

<!-- TODO: Should we allow deletion of non-empty collections, i.e. deletion of collection and children simultaneously? -->

#### DELETE Query Parameters

The only query parameter that *must* be accepted for a `DELETE` request is the unique `id` of the item to be removed.

The other parameter that *may* be accepted is `token` which carries the client's authentication token.

#### DELETE Request Body

Delete requests do not include a body, so the request body should be empty.

#### DELETE Responses

##### Status codes

With a `DELETE` request the server should provide different responses based on whether or not the operation has already concluded when the response is sent. If the deletion is done synchronously, and is finished at response time, the returned status code should be `200(OK)`. If the request simply began an asynchronous deletion process, then the response should return `202(Accept)`.

If a response returns a status of `404(Not Found)` then the response body should contain a JSON object indicating that the requested `@id` value does not exist and so the requested item cannot be modified.

##### Successful response content

Unlike with other methods, the response headers after a successful DELETE request should *not* include a `Location` value. The response should, though, include a `Content-type` header with the value "application/ld+json".

In a successful DELETE request, the response body should be a JSON-LD object representing the record that was deleted. This object should always include the `@context` and `@id` properties along with all other terms that carried associated values at the time of deletion.

##### Unsuccessful response content

In an unsuccessful request, the response headers should include a `Location` whose value is the URL for the Collections endpoint documentation. The response should also include a `Content-type` header with a value of "application/ld+json".

If a response returns an error code, the response body should contain a JSON-LD object following the hydra specification for status responses (https://www.hydra-cg.com/spec/latest/core/#description-of-http-status-codes-and-errors). For example, this would be a well-formed response body for a `404(Not Found)` error:

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "Status",
  "statusCode": 404,
  "title": "Not Found",
  "description": "No item exists with the id specified in your request."
}
```
If a response returns a status of `404(Not Found)` then the `description` value should indicate that the requested `@id` value does not exist and so the requested item cannot be modified.

#### DELETE Example 1: Deleting an Existing Collection

In this example we will use a `DELETE` request to remove from the server the metadata for the collection with the `id` value "general." This will be a synchronous `DELETE` operation, so the successful response code will be `200(OK)`.

##### DELETE request URL

- `/api/dts/collections/?id=general&token=XXXXX`

##### DELETE request body

No body should be sent with the `DELETE` request.

##### Successful DELETE response status

- 200(OK)

##### Successful DELETE response headers

| key | Value |
| --- | ----- |
| Content-Type  | application/ld+json             |

##### successful DELETE response body

```json
{
    "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
    },
    "@id": "general",
    "@type": "Collection",
    "totalItems": 0,
    "title": "Collection Générale de l'École Nationale des Chartes",
}
```
## Documents Endpoint: Optional Methods

In addition to retrieving the text of resources via the `GET` method, implementations may also support creation and modification of textual units through the methods POST, PUT, and DELETE.

This documentation assumes that you have read the core specification for the Documents endpoint.

### Default Scheme

As with the core `GET` specifications, the other methods of the Documents endpoint __may__ accept as many different text encoding schemes as the implementer wishes. Any implementation __must__, though, at least support XML data encoded using the Text Encoding Initiative (TEI) schema. The specification that follows will assume that information is being transferred in this format.

As with the core Documents specification, these extended methods also should expect to receive textual data as a `<TEI>` rootnode of the namespace http://www.tei-c.org/ns/1.0. If an entire document is being sent in the body of a single request (as when a new document is first created) the contents of that <TEI> rootnode need simply be compliant with the TEI specification. In all other cases, the text being submitted in the request body should be wrapped in a  `<fragment>` element of the DTS Namespace (https://w3id.org/dts/api#). So most request bodies will look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <!-- XML or string of the passage submitted here -->
  </dts:fragment>
</TEI>
```

Again, the text contained in the `<fragment>` element may be any part of a TEI-compliant XML document, as appropriate for the operation being performed.

The TEI schema often allows for multiple ways of encoding the same information. Projects implementing these extended Documents methods may want to provide further guidelines about how the TEI guidelines are to be implemented. Such further guidelines should be included or referenced in the API documentation provided at the root URL for the Documents endpoint.

Note that the core `GET` specification allows for arbitrary siblings to be included alongside the `<fragment>` element inside the `<TEI>` rootnode. For textual data being submitted via `POST` or `PUT` requests such siblings __should be ignored__. Contextual information (metadata) about the document should be submitted using the Collections or Docinfo endpoints. Contextual information about a particular passage of text should be submitted using the Annotations endpoint.

### Extended URI

#### Extended Documents Query Parameters

The extended Documents endpoint should accept the following query parameters, depending on which methods are being supported by the implementation:

| name	             | description	          | methods |
| -------------------| -----------------------| --------|
| id	             | (Required) identifier for a document	| GET, POST, PUT, DELETE |
| ref                | passage identifier (used together with id can’t be used with start and end)	| GET, PUT, DELETE |
| start	             | (For range) Start of a range of passages (can’t be used with ref)	| GET, PUT, DELETE |
| end	             | (For range) End of a range of passages (requires start and no ref)	| GET, PUT, DELETE |
| after              | (Either this or "before" required for POST) passage after which the new segment should be inserted | POST |
| before             | (Either this or "after" required for POST) passage after which the new segment should be inserted | POST |
| token	             | (May be required) Authentication token for access control	| POST, PUT, DELETE |

Note that, as in the core Documents specification, one must either provide a "ref" parameter __or__ a pair of "start" and "end" parameters. A request cannot combine "ref" with the other two. If, say, a "ref" and a "start" are both provided this should trigger an error response.

Two new parameters are introduced for the extended Documents endpoint: "after" and "before." These allow one to specify in a `POST` request where a new text segment should be inserted. One or the other of "after" and "post" must be included in a `POST` request.

As with the other extended methods, there is also the optional addition of the "token" parameter for authentication and access control.

#### Extended Documents URI Template
Here is a template of the URI for the Documents endpoint with support for the extended methods. The route itself (/dts/api/documents/) is up to the implementer. The only new parameter added to the core Documents URI template is the "token" parameter.
```json
{
  "@context": {
        "@vocab": "https://www.w3.org/ns/hydra/core#",
        "dc": "http://purl.org/dc/terms/",
        "dts": "https://w3id.org/dts/api#"
  },
  "@type": "IriTemplate",
  "template": "/dts/api/document/?id={collection_id}&ref={ref}&start={start}&end={end}&after={after}&before={before}&token={token}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "ref",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "start",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "end",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "after",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "before",
      "required": false
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "token",
      "required": false
    }
  ]
}
```

### POST on the Document Endpoint  

The `POST` method of the Documents endpoint allows for creation of new textual passages in a resource. This method requires that the document already has a metadata record accessible via the Collections endpoint.

Note that this method should __not__ be used to modify existing segments of text. In other words, if the specified reference label already exists in the document on the server, this method should return an error and refuse to perform the requested operation. If, for example, a document already has a line 6 following the existing line 5, then `POST` cannot be used to insert new or added text in that existing line 6. Modification of the text in existing document segments must be done using the `PUT` method instead.

#### POST Query parameters

The only strictly required parameters for a `POST` Documents request are "id" and (if supported) "token." If neither "before" nor "after" is supplied, the server should interpret the request as supplying the initial form of a new document. In that case, the server should return an error if (a) some text already exists for that document, or (b) the request body contains a `<dts:fragment>` element.

In most cases, though, the query will be inserting a new segment of text into a document that already contains some text. In such instances, the query must include either a "before" or an "after" parameter to indicate where the new text segment will be inserted.

Neither the "ref" parameter nor the pair of "start"/"end" parameters should be provided in a `POST` Documents request. The reference information for the inserted segment(s) should be included using the standard TEI attributes in the XML of the request body.

Note that the "after" or "before" references should also be used to determine the depth of the insertion in the document's citation structure. The lowest structural level specified in the "after" or "before" reference is the level at which the contents of the XML `<fragment>` will be inserted. In other words, the new segment will be taken to be a sibling to the segment specified in that reference.

#### POST Request Body

The body of the `POST` request must be a properly formed `<TEI>` root node containing TEI compliant XML. See the further specifications under "Default Scheme" above.

Note that in most cases the actual text to be inserted is the __contents__ of the inner `<fragment>` element in the request body. The only exception to this rule is where no text has yet been created for a document, in which case the entire `<TEI>` rootnode (along with all its contents) will be used as the initial form of the document.

#### POST Responses

##### Status codes

A successful POST request to the Documents endpoint should return the status code `201(Created)`.

If a POST request is unsuccessful because of problems with the request content (i.e., parameters or request body), then the return status code should be `400(Bad Request)`.

A POST request should also return `409(Conflict)` if it would create a new initial form of a document for which some text already exists. This would be when no "after" or "before" parameter is supplied, but at least one segment of the document already contains text (even if this text is simply an empty string).

A `POST` Documents request also may not result in a text segment whose reference identifier is the same as that of an existing segment. If, for example, a document already includes text at the location "4.23.8", then a `POST` request which would result in a *second* location "4.23.8" should fail and return `409(Conflict)`.

##### Successful response content

The response headers after a successful `POST` Documents request should include a `Location` value. This should be the URL where the newly inserted text segment(s) can be retrieved via a GET request. The response should also include a `Content-type` header with a value of "application/tei+xml".

The response should also include a `Link` header as detailed in the core documentation for the Documents endpoint. This `Link` gives URL references for the previous and next segments of the document, a URL for the document's navigation structure, and a URL for the document's Collections metadata.

The response body after a successful POST request should contain an XML object representing the newly created text segment(s). This should be identical to the response a client would get using a GET request to the `Location` indicated in the response header. This will also mean that in most successful requests the XML contained in the response body will be identical to the object sent by the client in the `POST`.

##### Unsuccessful response content

In an unsuccessful request, the response headers should include a `Location` whose value is the URL for the Documents endpoint documentation. The response should also include a `Content-type` header with a value of "application/tei+xml".

If a response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `400(Bad Request)` error:

```xml
<error statusCode="400" xmlns="https://w3id.org/dts/api">
  <title>Improperly Formet Request Body</title>
  <description>The body of a POST request to the Documents endpoint must be properly formed XML. If it is submitting the initial text for a new document, the request body must be a <TEI> rootnode with properly formed and schema-compliant children.</description>
</error>
```

For a `400(Bad Request)` error, the `description` should provide as much information as possible about which submitted data was missing or unacceptable. It should indicate whether:
- there were missing required query parameters
- any query parameters held unacceptable values
- the request body did not have properly formed XML
- the request body was properly formed but carried unacceptable values

If a response returns a status of `409(Conflict)` then the XML `<description>` element should contain an indication that the request would result in a duplicate segment in the document's citation structure.

#### POST Example 1: Creating the initial text for a new document

In this example we will submit the first text segments to constitute a new document. Prior to this operation no textual data exists for the document on the server. This first `POST` request only creates verses 1-2 of chapter 1 in the document `urn:cts:ancJewLit:1Enoch`.

Note that a metadata record for the document must already be added via the Collections endpoint. This is the only way to establish a valid identifier for the document, which is then used for the `POST` request.

Note too that no `<teiHeader>` element precedes the main `<text>` element. The kind of metadata contained in a `<teiHeader>` should be created either using the Collections endpoint (in JSON-LD format) or the Docinfo endpoint (where an XML TEI header can be uploaded).

##### POST request URL
Notice that this request omits the usual "before" or "after" parameters.

- `/api/dts/documents/?id=urn:cts:ancJewLit:1Enoch&token=XXXXX`

##### POST request body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
    <text xml:lang="Ethiopic">
      <body>
        <div n="1" xml:id="1En1" type="Chapter">
          <div n="1:1" xml:id="1En1:1" type="Verse">
            <app xml:id="1">
              <rdg wit="#Bertalotto #p"> ቃለ፡​በረከት፡​ዘሄኖከ፡​ዘከመ፡​ባረከ፡​ኅሩያነ፡​ወጻድቃነ፡​እለ፡​ሀለዉ፡​ይኩኑ፡​በዕለተ፡​ምንዳቤ፡​ለአሰስሎ፡​ኵሎ፡​እኩያን፡​ወረሲዓን፡​</rdg>
            </app>
          </div>
          <div n="1:2" xml:id="1En1:2" type="Verse">
            <app xml:id="2">
              <rdg wit="#Bertalotto #p">ወአውሥአ፡​ሄኖክ፡​ወይቤ፡​ብእሲ፡​ጻድቅ፡​ዘእምኀበ፡​እግዚአብሔር፡​እንዘ፡​አዕይንቲሁ፡​ክሡታት፡​ወይሬኢ፡​</rdg>
            </app>
            <app xml:id="3">
              <rdg wit="#p">ራዕየ፡​</rdg>
              <rdg wit="#Bertalotto">ራእየ፡​</rdg>
            </app>
            <app xml:id="4">
              <rdg wit="#Bertalotto #p">ቅዱሰ፡​ዘበሰማያት፡​ዘአርአዩኒ፡​መላእክት፡​ወሰማዕኩ፡​እምኀቤሆሙ፡​ኵሎ፡​ወአእመርኩ፡​አነ፡​ዘእሬኢ፡​ወአኮ፡​ለዝ፡​ትውልድ፡​አላ፡​ለዘ፡​ይመጽኡ፡​ትውልድ፡​ርሑቃን፡​</rdg>
            </app>
          </div>
        </div>
      </body>
    </text>
</TEI>
```

##### Successful POST response status

- 201(Created)

##### Successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/documents?id=urn:cts:ancJewLit:1Enoch |
| Content-Type  | application/tei+xml             |

##### Successful POST response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
    <text xml:lang="Ethiopic">
      <body>
        <div n="1" xml:id="1En1" type="Chapter">
          <div n="1:1" xml:id="1En1:1" type="Verse">
            <app xml:id="1">
              <rdg wit="#Bertalotto #p"> ቃለ፡​በረከት፡​ዘሄኖከ፡​ዘከመ፡​ባረከ፡​ኅሩያነ፡​ወጻድቃነ፡​እለ፡​ሀለዉ፡​ይኩኑ፡​በዕለተ፡​ምንዳቤ፡​ለአሰስሎ፡​ኵሎ፡​እኩያን፡​ወረሲዓን፡​</rdg>
            </app>
          </div>
          <div n="1:2" xml:id="1En1:2" type="Verse">
            <app xml:id="2">
              <rdg wit="#Bertalotto #p">ወአውሥአ፡​ሄኖክ፡​ወይቤ፡​ብእሲ፡​ጻድቅ፡​ዘእምኀበ፡​እግዚአብሔር፡​እንዘ፡​አዕይንቲሁ፡​ክሡታት፡​ወይሬኢ፡​</rdg>
            </app>
            <app xml:id="3">
              <rdg wit="#p">ራዕየ፡​</rdg>
              <rdg wit="#Bertalotto">ራእየ፡​</rdg>
            </app>
            <app xml:id="4">
              <rdg wit="#Bertalotto #p">ቅዱሰ፡​ዘበሰማያት፡​ዘአርአዩኒ፡​መላእክት፡​ወሰማዕኩ፡​እምኀቤሆሙ፡​ኵሎ፡​ወአእመርኩ፡​አነ፡​ዘእሬኢ፡​ወአኮ፡​ለዝ፡​ትውልድ፡​አላ፡​ለዘ፡​ይመጽኡ፡​ትውልድ፡​ርሑቃን፡​</rdg>
            </app>
          </div>
        </div>
      </body>
    </text>
</TEI>
```

#### POST Example 2: Creating a new section of text in an existing document

In this example we will create a new text segment in the existing document `urn:cts:ancJewLit:1Enoch`. Initially we created the document with just verses 1 and 2 of chapter 1. We will now add a third verse to that same first chapter.

##### POST request URL

- `/api/dts/documents?id=urn:cts:ancJewLit:1Enoch&after=1:2&token=XXXXX`

##### POST request body
Note that since some text already exists for this document, the new segment is submitted in a `<dts:fragment>` element.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

##### Successful POST response status

- `201(Created)`

##### Successful POST response headers

| key | Value |
| --- | ----- |
| Location      | /api/dts/documents?id=urn:cts:ancJewLit:1Enoch&ref=1:3 |
| Content-Type  | application/tei+xml               |

##### Successful POST response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

### PUT on the Document Endpoint  

The `PUT` method of the Documents endpoint allows for modification of existing textual passages in a resource.

Note that this method should __not__ be used to create new structural segments of text. For example, if the citation structure of the document already contains a section "5.7.31" then the `PUT` method may be used to modify that section. If section "5.7.31" does not already exist in the document on the server, an attempt to edit that segment using `PUT` will result in an error response. The `POST` method must first be used to add that segment to the document. Likewise, `PUT` should never result in the deletion of a segment in the document's citation structure. Of course, other XML entities below lowest level of a document's citation structure may be freely added or removed by the `PUT` method

#### POST Query parameters

The three required parameters for a `PUT` Documents request are "id," "token" (if supported by the implementation), and "ref." Only one structural segment of the document may be modified in a single `PUT` request.

The text/XML submitted in the body of a Documents `PUT` request will completely replace the XML entity representing the identified text segment. So the submitted fragment __must include the outer element__ identified by the "ref" value. If you identify your submitted text as a modified version of line "12", and if each line is represented by a TEI `<ln>` element, you would include the opening and closing tags `<ln xml:id="16">` and `</ln>` around the changed content. This is to allow modification of the attributes on the outer element tag.

The "ref" references also determines the structural depth of the modification. Suppose a document has a three-level citation structure represented in XML by nested `<div>` elements. A `PUT` request with a "ref" of "3.16.8" will replace just one `<div>` element at the lowest structural level. If the request is instead sent with a "ref" parameter of "3.16", the submitted `<div>` element will replace the second-level `<div>` with the identifier "3.16". In the latter case, the submitted `<div>` element will need to contain the series of nested `<div>` elements that represent the bottom-level segments "3.16.1", "3.16.2", "3.16.3", etc. It is recommended that implementations check the submitted text before performing upper-level modifications like this, to ensure that no lower-level structural segments are being created or destroyed. Alternately, a client may avoid this issue by only making `PUT` requests at the lowest level of the document's citation structure.

#### PUT Request Body

The body of the `PUT` request must be wrapped in a TEI rootnode containing a `<dts:fragment>` entity. See the further specifications under "Default Scheme" above.

The contents of this `<dts:fragment>` must be a single XML element (along with its enclosed text and/or children) representing the modified form of the target document section. This single element inside the `<dts:fragment>` will replace the corresponding XML element in the document on the server.   

#### PUT Responses

##### Status codes

If the specified section of the document is successfully modified, the response status should be `200(OK)`.

If no structural section exists with an identifier matching the "ref" parameter in the request, the response status should be `404(Not Found)`. If there is some other problem with the request data, the response status should be `400(Bad Request)`.

##### Successful response content

The response headers after a successful PUT request should include a `Location` value. This should be the URL where the newly modified document section can be retrieved via a GET request. The response should also include a `Content-type` header with a value of "application/tei+xml".

The response should also include a `Link` header as detailed in the core documentation for the Documents endpoint. This `Link` gives URL references for the previous and next segments of the document, a URL for the document's navigation structure, and a URL for the document's Collections metadata.

In a successful PUT request, the response body should be an XML object with the same structure as a `GET` response, but containing the the newly modified contents of the specified text section. This allows the client to quickly recognize whether the correct information was updated on the server.

##### Unsuccessful response content

In an unsuccessful `PUT` request, the response headers should include a `Location` whose value is the URL for the Documents endpoint documentation. The response should also include a `Content-type` header with a value of "application/tei+xml".

If a response returns an error code, the response body should contain an XML object following the DTS specification for XML status responses (https://w3id.org/dts/api). For example, this would be a well-formed response body for a `400(Bad Request)` error:
```xml
<error statusCode="400" xmlns="https://w3id.org/dts/api">
  <title>Improperly Formet Request Body</title>
  <description>The body of a POST request to the Documents endpoint must be properly formed XML. If it is submitting the initial text for a new document, the request body must be a <TEI> rootnode with properly formed and schema-compliant children.</description>
</error>
```

If a response returns a status of `404(Not Found)` then the `<description>` contents should indicate that no section with the requested "ref" identifier exists and that the section must be created before it can be modified.

If the status code is `400(Bad Request)` then the `description` should clarify which parts of the request data were not acceptable.

#### PUT Example 1: Changing the child elements in a bottom-level section

In this example we will change the contents of the section with the reference "1:3" in the document with the id "urn:cts:ancJewLit:1Enoch". Since this document is a critical edition, that section contains a series of TEI `<app>` elements, each of which contains a set of parallel `<rdg>` elements with alternate textual variants. In the middle `<app>` element we will add a third `<rdg>` option that is empty, representing a manuscript which lacks any corresponding words.

##### PUT request URL

- `/api/dts/documents?id=urn:cts:ancJewLit:1Enoch&ref=1:3&token=XXXXX`

##### PUT request body
If you compare this with POST Example 1 above, you will notice that the modification in this case involves changes to the XML child elements, not just to the text they contain.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
          <rdg wit="#a"></rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

##### Successful PUT response status

- `201(Created)`

##### Successful PUT response headers

| key | value |
| --- | ----- |
| Link          | </api/dts/documents?id=urn:cts:ancJewLit:1Enoch&ref=1:2>; rel="prev", </api/dts/documents?id=urn:cts:ancJewLit:1Enoch&ref=1:4>; rel="next", </api/dts/documents?id=urn:cts:ancJewLit:1Enoch&ref=145:12>; rel="last", </api/dts/navigation?id=urn:cts:ancJewLit:1Enoch>; rel="contents", </api/dts/collections?id=urn:cts:ancJewLit:1Enoch>; rel="collection"|
| Content-Type  | application/tei+xml               |

##### Successful PUT response body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="https://w3id.org/dts/api#">
    <div n="1:3" xml:id="1En1:3" type="Verse">
        <app xml:id="5">
          <rdg wit="#Bertalotto #p">በእንተ፡​ኅሩያን፡​እቤ፡​ወአውሣእኩ፡​በእንቲአሆሙ፡​ምስለ፡​ዘይወጽእ፡​ቅዱስ፡​</rdg>
        </app>
        <app xml:id="6">
          <rdg wit="#p">ወዓቢይ፡​</rdg>
          <rdg wit="#Bertalotto">ወዐቢይ፡​</rdg>
          <rdg wit="#a"></rdg>
        </app>
        <app xml:id="7">
          <rdg wit="#Bertalotto #p">እማኅደሩ</rdg>
        </app>
    </div>
  </dts:fragment>
</TEI>
```

### DELETE on the Document Endpoint  
