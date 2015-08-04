## What is this?

This is a summary of common REST API design constraints and conventions intended as a guide for developers when introducing new or modifying existing REST API endpoints. &nbsp;It is derived from the [RESTful API Design Guidelines](README.md)&nbsp;but distilled down to the most essential points such that it can be read in just a few minutes and easily referenced.

## REST API Design Principles

### _REST Methods_

*M1.* The GET response body is a representation of the resource identified by the request URL.

*M2.* The GET method does not change the state of the resource identified by the request URL.

*M3.* The POST request creates a new resource as a leaf under the request URL path.

*M4.* The POST response body is either empty (status 204) or a representation of the resource created (status 201).

*M5.* The PUT request to a URL at which no resource yet exists may create a new resource at that URL.

*M6.* The PUT request to a URL at which a resource already exists overwrites the state of that resource with the representation provided in the request body.

*M7.* The PUT response body is a representation of the resource identified by the URL as it exists following any changes made as a result of the request.

*M8.* The DELETE request to a URL at which a resource exists is a request to remove the resource identified by that URL.

*M9.* The DELETE response body is either empty (204) or a representation of the resource deleted (200).

*M10.* The PATCH request to a URL partially updates the state of the resource at that URL using a provided json-patch (RFC-6902) body.

*M11.* The PATCH response returns a representation of the resource identified by the URL as it exists following any changes made as a result of the request.

### _URL Paths_

*P1.* The URL path represents a resource, or a collection of resources.

*P2.* The URL path root is of the form: `/{service}/{apiclass}/v{version}`.

*P3.* The last path segment of a collection URL is a plural noun, describing the type of resources it contains.

*P4.* The parent of the last path segment of a resource URL is a plural noun and represents a resource collection.

*P5.* The last path segment of a resource URL is a unique identifier of the resource within it's container.

*P6.* The URL path segments are alphanumeric and follow camelCase convention (first character is lowercase).

*P7.* The URL path segments are intuitive, unambiguous, and succinct.

### _URL Query Parts_

*Q1*. URL query parameters representing date/time are in RFC-3339 "iso-date-time" format.

*Q2*. URL query parameters representing duration are in RFC-3339 "duration" format.

*Q3*. URL query parameters representing temporal intervals are in RFC-3339 "period" format.

*Q4*. URL query parameters semantically equivalent to those found elsewhere in the API have matching names.

*Q5*. URL query parameters not semantically equivalent to those found elsewhere in the API have different names.

*Q6*. URL query parameters for selectively retrieving specific fields of a resource conform to the [partial retrieval template &sect; 3.6.3.6](README.md) .

*Q7*. URL query parameters pertaining to paginated retrieval of resource representations conform to the [paginated query form &sect; 3.6.3.3](README.md).

*Q8*. URL query parameter names are alphanumeric and follow camelCase convention (first character is lowercase).

*Q9*. URL query parameter names are intuitive, unambiguous, and succinct.

### _Resource Representations_

*R1*. A resource representation is encoded as application/json except where standard media-type representations are available (e.g. png, vcard, etc.).

*R2*. A collection representation is encoded as application/json with an array attribute containing representations of resources in the collection.

*R3*. When a collection supports pagination, a representation of that collection conforms to the [paginated collection template &sect; 3.6.3.4](README.md). 

*R4*. A resource representation as provided in a server response includes a "url" field that is the absolute and canonical URL of the resource itself.

*R5*. A resource representation does not contain data for which another REST service is the "source of truth".

*R6*. A response body does not include status of the operation so as to override or otherwise alter the semantics of the HTTP response status code.

*R7*. When returning an HTTP status code of 4xx or 5xx, response entity data conforms to the [error response template &sect; 3.9.2](README.md).

### _JSON Attributes_

*J1.*&nbsp;Attribute names are intuitive, unambiguous, and succinct.

*J2.* Attribute names are alphanumeric and follow camelCase convention (first character is lowercase).

*J3.* Attributes representing date/time are in RFC-3339 {{iso-date-time}} format and with a UTC offset.

*J4.* Attributes representing duration are in RFC-3339 {{duration}} format.

*J5.* Attributes representing temporal intervals are in RFC-3339 {{period}} format.

*J6.* Attributes representing binary values are encoded using the JSON native boolean type.

*J7.* Attributes representing arrays are named as plural nouns.

*J8.* Attributes representing non-arrays are named as singular nouns.

*J9.* Attributes representing links to other resources are provided as absolute and canonical URLs.

*J10.* Attributes representing data semantically equivalent to that found elsewhere have matching field names.

*J11.* Attributes representing data not semantically equivalent to that found elsewhere have different field names.

*J12.* Attributes representing enumerated types have well defined legal values and semantics.

*J13.* No semantic distinction is drawn between the absence of a JSON attribute and it's explicit assignment as `null`.

### _Security_

*S1.*&nbsp;A REST request uses a Common Identity OAuth2 flow for authentication, authorization.

*S2.*&nbsp;A REST request has a clear policy regarding the authorization scope required for access.

*S3.*&nbsp;A REST request containing personally identifiable information (PII) must be authenticated.

*S4.*&nbsp;A REST request does not allow unintentional content or PII leaks between users.

*S5.*&nbsp;A REST request that is authenticated must be made over a secure channel (HTTPS).

*S6.*&nbsp;A REST request does not contain authentication, authorization, or PII material in the URL.

### _API Documentation_

*D1.* API documentation is auto-generated from source and supplemental text files.

*D2.* The auto-generated documentation provides sufficient description of the resource and methods supported.

*D3.* The auto-generated documentation provides sufficient description of the policy for authorized access and invocation.

*D4.* The auto-generated documentation provides sufficient description of the URL path and query parameters.

*D5.* The auto-generated documentation provides sufficient description of all JSON attributes of the resource representation.

*D6.* The auto-generated documentation provides sufficient description of all anticipated error conditions.
