# Distributed Text Services : Document API

The documents entry point is used to access the data for documents, as opposed to metadata (which is found in collections).  The representation of a document is up to the implementation.

## Default Scheme

- The document API can support as many response format as the content provider wishes.
- The document API must support an `application/tei+xml` response format that is made of at least the following structure : 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
  <dts:fragment xmlns:dts="DTS PURL ?">
    <!-- XML or string of the passage requested here -->
  </dts:fragment>
</TEI>
```
- The rest of the document can contain any node of the TEI
- The requested passage should be part of the `dts:fragment` Node

## URI 

### Query Parameters

The documents endpoint supports the following query parameters:

| name | description                              | methods |
|------|------------------------------------------|---------|
| id   | identifier for a document |  GET    |
| passage | passage identifier (used together with `id`) | n/a    |
| start | (For range) Start of the passage we want descendants of | GET |
| end |  (For range) End of the passage we want descendants of | GET |

### Response Headers

The response contains the following response headers:

| name | description |
|------|-------------|
| Link | Gives relation to next and previous pages |
| Content-Type | Content type of the response (By default : `application/tei+xml`)|

### URI Template

Here is a template of the URI for Collection API. The route itself (`/dts/api/navigation/`) is up to the implementer.

```json
{
  "@context": "http://www.w3.org/ns/hydra/context.jsonld",
  "@type": "IriTemplate",
  "template": "/dts/api/navigation/?id={collection_id}&passage={passage}&level={level}&start={start}&end={end}&page={page}",
  "variableRepresentation": "BasicRepresentation",
  "mapping": [
    {
      "@type": "IriTemplateMapping",
      "variable": "collection_id",
      "required": true
    },
    {
      "@type": "IriTemplateMapping",
      "variable": "passage",
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
    }
  ]
}
```

## Examples

### Retrieve a passage using `passage`

Retrieve the passage `1` of `bgu.11.2029`

#### Example of url : 

- GET `/dts/api/documents/?id=bgu.11.2029&passage=2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/documents/?id=bgu.11.2029&passage=1>; rel="prev", </dts/api/documents/?id=bgu.11.2029&passage=3>; rel="next" | 

#### Response

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <teiHeader>
      <fileDesc>
         <titleStmt>
            <title>bgu.11.2029</title>
         </titleStmt>
         <publicationStmt>
            <authority>Duke Collaboratory for Classics Computing (DC3)</authority>
            <idno type="filename">bgu.11.2029</idno>
            <idno type="ddb-perseus-style">0001;11;2029</idno>
            <idno type="ddb-hybrid">bgu;11;2029</idno>
            <idno type="HGV">9566</idno>
            <idno type="TM">9566</idno>
            <availability>
               <p>© Duke Databank of Documentary Papyri. This work is licensed under a <ref type="license" target="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 License</ref>.</p>
            </availability>
         </publicationStmt>
      </fileDesc>
   </teiHeader>
   <dts:fragment xmlns:dts="DTS PURL ?">
    <lb n="1"/><expan>τετελ<ex>ώνηται</ex></expan> <expan>δι<ex>ὰ</ex></expan> <expan>πύλ<ex>ης</ex></expan> Διονυσιάδος 
   </dts:fragment>
</TEI>
```

### Retrieve a passage using start and end

Retrieve the passages 1.1.1 to the passage 1.1.2

#### Example of url : 

- GET `/dts/api/documents/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.1.1&end=1.1.2`

#### Headers

| Key | Value | 
| --- | ----- |
| Content-Type | Content-Type: application/tei+xml |
| Link | </dts/api/documents/?id=urn:cts:latinLit:phi1318.phi001.perseus-lat1&start=1.2.1&end=1.2.2>; rel="next" | 

#### Response

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TEI xmlns="http://www.tei-c.org/ns/1.0">
   <dts:fragment xmlns:dts="DTS PURL ?">
       <text xml:lang="lat">
          <body>
             <div type="edition" xml:lang="lat" n="urn:cts:latinLit:phi1318.phi001.perseus-lat1">
                <div type="textpart" n="1" subtype="book">
                   <div type="textpart" n="1" subtype="letter">
                      <div type="textpart" n="1" subtype="section">
                         <p>Frequenter hortatus es, ut epistulas, si quas paulo curatius scripsissem, colligerem publicaremque. Collegi non servato temporis ordine - neque enim historiam componebam -, sed ut quaeque in manus venerat.</p>
                      </div>
                      <div type="textpart" n="2" subtype="section">
                         <p>Superest ut nec te consilii nec me paeniteat obsequii. Ita enim fiet, ut eas quae adhuc neglectae iacent requiram et si quas addidero non supprimam. Vale.</p>
                      </div>
                   </div>
                </div>
             </div>
          </body>
       </text>
   </dts:fragment>
</TEI>
```


### Retrieve a full document

[Waiting for solution in issue 82](https://github.com/distributed-text-services/collection-api/issues/82)