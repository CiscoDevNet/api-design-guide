
# 1 Preface

Several Cisco business units have teamed up to create this RESTful API design guide.  Collectively, this includes [DevNet](https://developer.cisco.com), [Collaboration](http://www.cisco.com/c/en/us/solutions/collaboration/index.html), and the Application Platform Group.  The objective of this document is to act as a guide to define the future, it does not represent the current state of all REST APIs at Cisco.

tl;dr: These guidelines are also distilled into a set of [API Principles](principles.md).

**Note: the latest recommendations to design APIs at Cisco are published at https://apistyleguide.cisco.com, and are accessible from Cisco internal network. These recommendations cover both REST and YANG styles, addressing Web APIs and Network Devices APIs respectively.**


# 2 Overview

This document serves as a high-level description of our API Design Guidelines.


## 2.1 Goals

A primary goal of these guidelines is to establish a cohesive look and feel for all Cisco product and service network APIs ("web services" herein). Additionally, we wish to instill an "API-only" development approach where all communications between separate modules must happen via APIs.

## 2.2 Non-Goals

"API" can mean a lot of things; in the context of this document, we're referring to service-level APIs that are exposed over a network. We are not discussing the look and feel of Java libraries, for example.

These guidelines also omit discussion regarding the means by which an application might implement a RESTful API.  No assumptions are made regarding programming language or the use of application server frameworks.


## 2.3 "API-Only" Communications

Why is it valuable to route all communications between web services through well-defined APIs? For intra-Cisco communications, why not just use glass-box knowledge of the internals of other services to tie things together?

Any time communication happens between services, there is by definition an API involved. By using internal hooks (knowledge of underlying database schemas or disk layout; special undocumented parameters in HTTP requests; etc.), we are simply codifying those hooks as the de facto APIs. This makes the system more brittle as a whole, and to no real benefit.  Exposing functionality through a well-thought-out API should not be much more up-front work, and certainly saves on downstream effort.

But what about "advanced" functionality that is needed for our intra-Cisco service communication, but shouldn't be exposed to third parties? While there may be some business reasons not to expose certain APIs to third parties, the level of advancement of the functionality does not justify softer API treatment. Creating an internal set of APIs (de facto or explicitly) and a second set of "partner APIs" is a recipe for weak partner products, bugs in third-party integrations, and wasted effort developing and testing multiple pathways to perform the same tasks.

We also recognize that in a complex system, composed of many components evolving on separate schedules, it is imperative that the interfaces between these components be robust, well understood, and as loosely coupled as possible.  By defining high quality APIs we can address these concerns by providing rules of engagement that reduce the risk of breakage when deploying component upgrades.  In a RESTful API, these typically allow for the continuous introduction of new features, feature extensions, and resources, but disallow the elimination or destructive alteration of those previously supported (outside of major API changes, which are versioned and intentionally rare).  With this essential strategy, system components can evolve at their own pace without the burden of complicated rollout schedule dependencies which can significantly delay the release of important products and features.


## 2.4 Open Standards

Open standards are a key part of the Cisco strategy. When a service implements a standard, the APIs defined by the standard should be used whenever possible, and extensions should be designed in a way that dovetails with the standard itself. When a service is in a space with existing standards, they should be implemented whenever appropriate. This document is primarily focused on how to build an API to expose functionality that is **not** standards-based, and should **not** be interpreted as a justification to avoid standards-based development.


## 2.5 RESTful

APIs to a Cisco product or service should be primarily RESTful. That is, in keeping with Roy Fielding's principles of representational state transformations. Briefly, this means that the API should be designed around identifiable resources which can be manipulated by a small handful of pre-defined actions (creation, modification, deletion).  If the reader wishes to “brush up” on the subject, there are many excellent resources.  We would recommend starting with [Dr. Roy Fielding's paper](#references) is the seminal tome on the subject of REST.  However, that paper is too academic in nature to be applied as a practical guideline.  There are many excellent practical resources on the subject, probably the best of which is the O'Reilly book [“RESTful Web Services”](#references) written by Richardson and Ruby.

Note that SOAP is not RESTful.


## 2.6 Aesthetics Matter

REST is an architectural style, not a particular set of rules. As a result, there are alternative ways to construct a RESTful system using HTTP. The aesthetics of an API are important, however, and consistency among Cisco products is the objective.  In a sense, API design is as much about user experience as it is technical completeness and correctness, as we must consider how the design impacts the experience of developers working to write high quality applications that consume our services.  In that context, it is clear that good API design requires a stewardship of the API UE with the same seriousness as we apply to our end users' UE.

# 3 Guidelines

The following guidelines are derived from best practices, as described in a variety of sources including some prior art at Cisco.  However, in the interest of reducing ambiguity and promoting consistency among Cisco applications, this section uses somewhat more normative (prescriptive) language than is typically used in a best-practices discussion.

The following definitions apply for all specifications of section 3.

*Reference Representation*  Refers to a JSON representation containing no more and no less than that data required to uniquely identify and globally locate the resource.  The URLs provided in reference representations are the canonical URLs of the resources they represent, and MUST be treated as opaque by clients.

```
{
    "url" : "https://widgets.example.com/files/v3/documents/e23af9"
}
```


*Narrow Representation*  Refers to a JSON representation containing data belonging only to the resource represented by the root object.  If that resource contains internal references to other resources, these are represented as reference representations (defined above).

```
{
    "url" : "https://widgets.example.com/files/v3/documents/e23af9",
    "name" : "Awesome Document.docx",
    "comment" : {
        "url" : "https://widgets.example.com/files/v3/comments/a29f31",
    }
}
```


*Wide Representation*  Refers to any JSON representation containing data belonging to one or more resources referenced by the resource represented by the root object.

```
{
    "url" : "https://widgets.example.com/files/v3/documents/e23af9",
    "name" : "Awesome Document.docx",
    "comment" : {
        "url" : "https://widgets.example.com/files/v3/comments/a29f31",
        "text": "Wow, that's an awesome document.",
    }
}
```



## 3.1 Security

**3.1.1** Services MUST require the use of HTTPS for RESTful API traffic.  Services MUST NOT redirect non-secure HTTP requests to their secure HTTPS equivalents, but instead should result in a hard failure.

**3.1.4** Services MUST NOT use wildcard or self-signed certificates when deployed in production.

**3.1.5** Services MUST use certificates issued by widely trusted certificate authorities, exclusively, when deployed in production.

**3.1.6** Services MUST follow industry best practices for guarding private keys.  This includes, but is not limited to, storing keys only on security-hardened and fully patched systems.

**3.1.7** Services MUST NOT run as root.

**3.1.8** Services MUST perform input validation on all REST request parameters (see https://www.owasp.org/index.php/Top_10_2010-A1-Injection).

**3.1.9** Services MUST NOT accept or transmit sensitive data or user privacy data (PII) such as credentials, keys, passwords, SSN, credit card numbers, etc., in URLs.  Services SHOULD accepts and transmit such data as part of request and response entity-body data.

**3.1.10** Services MUST NOT expose through their API any predictable internal key references to objects.  That is, build the API to secure against Direct Object Reference vulnerabilities (see https://www.owasp.org/index.php/Top_10_2010-A4-Insecure_Direct_Object_References text).

* invalid example:


```
https://example.com/account/325365436/transfer?amount=$100.00&toAccount=473846376
```


**3.1.11** Services MUST ensure that, for any given request, the requesting user is always authenticated and authorized to perform the requested operation.

* invalid example:

A service accepting the following as a successfully authenticated and authorized request:


```
https://example.com/user/AccountInfo?email=alice@cisco.com
```


must protect against unintentionally accepting the following unauthorized request:


```
https://example.com/user/AccountInfo?email=bob@cisco.com
```


**3.1.12** Service endpoints that respond to GET requests MUST have no business logic side affects.  This is necessary to mitigate against CSRF attacks.

## 3.2 Authentication and Authorization

**3.2.1** The REST API MUST use OAuth2 implementation for user authentication and authorization, exclusively.  Specific mechanisms and guidelines for use of this implementation are defined in the architectural artifacts of that project and related standards documentation.  Additional references will be provided here as implementation progresses.

**3.2.2** Where any guideline in this document diverges from a standard protocol employed for authentication or authorization, the standard protocol MUST be followed.

**3.2.3** A service MUST NOT accept authentication material or authorization tokens provided as components of a request URL path or query parameters.

## 3.3 Representations

**3.3.1** Resource representations MUST be based on established standards when such standards exist for the resource type.

**3.3.2** Where resource representation standards do not apply, structured data resources MUST support update and retrieval based on JSON representations. The media type to be used in HTTP requests and responses which contain entity data in JSON format MUST be `application/json`, and optionally qualified with a `charset=UTF-8` parameter.  Absent this qualification, however, it MUST be assumed that the entity data is encoded as UTF-8.

**examples**:

```
Content-Type: application/json
Content-Type: application/json; charset=UTF-8
```


**3.3.3** A resource identifier labeled "url" MUST be present in all RESTful API resource representations, the value for which MUST be the absolute and canonical URL for the resource itself.

**example**:

```
{
    "url" : "https://widgets.example.com/files/v3/documents/e23af9"
     ...
}
```


**3.3.4** It is RECOMMENDED that services return narrow representations by default.

**3.3.5** Services MAY support the return of wide representations when explicitly requested by the client, and when the maximum size of the returned representation is bounded.

**3.3.6**  Services SHOULD NOT return narrow or wide representations of resources hosted on other services, but rather SHOULD return only reference representations of those resources. That is, a service SHOULD NOT act as a proxy for RESTful resource state for which the source of truth is another service.  Exceptions MUST demonstrably improve user experience or system performance.  In any such exceptional case, a service MUST provide a documented upper bound for how long cached data may remain out of sync.

*  **invalid example:**

```
{
    "url" : "https://widgets.example.com/files/v3/documents/e23af9",
    "name" : "Awesome Document.docx",
    "comment" : {
        "url" : "https://widgets.example.com/files/v3/comments/a29f31",
        "text": "Wow, that's an awesome document.",
    }
    "author" : {
        "url" : "https://widgets.example.com/users/v3/authors/b569fe",
        "first": "Benvolio",
        "last": "Montague"
    }
}
```


Assuming the above representation was returned by the Files service, it is invalid because the "author" object which is homed on the remote User service includes the first and last names of the author.  A valid equivalent of the above example would be:

**valid example**:

```
{
    "url" : "https://widgets.example.com/files/v3/documents/e23af9",
    "name" : "Awesome Document.docx",
    "comment" : {
        "url" : "https://widgets.example.com/files/v3/comments/a29f31",
        "text": "Wow, that's an awesome document.",
    }
    "author" : {
        "url" : "https://widgets.example.com/users/v3/authors/b569fe"
    }
}
```


The above is valid because the author is represented using a reference representation.

**3.3.7** In addition to the required "url" field, a service MAY include additional identification properties as part of a resource representation, such as an "id" field.  If present, any such field MUST be advertised to clients as opaque, non-canonicalizable to the resource URL, and potentially unstable over an extended period of time.

**3.3.8** Date and time fields MUST be represented as strings and formatted according to RFC-3339, specifically the ABNF syntax specification for `iso-date-time`.

**3.3.9** Duration fields MUST be represented as either integers representing lengths of time in whole seconds, or strings and formatted according to RFC-3339, specifically the ABNF syntax specification for `duration`.

**3.3.10** Temporal interval fields MUST be represented as strings and formatted according to RFC-3339, specifically the ABNF syntax specification for `period`.

**3.3.11** With regard to JSON representation property names, and URL query parameters, services SHOULD:
*  choose meaningful and succinct names,
*  not reuse any names reserved for other purposes by these guidelines,
*  avoid internal naming conflicts by reusing names for dissimilar purposes,
*  use plural nouns for arrays,
*  use singular nouns for non-arrays,
*  begin with lowercase letters,
*  prefer camelCase over under_scores,&nbsp;
*  follow SCIM Schema naming when the field represents data from the directory, and
*  be case-sensitive.

**3.3.12** With regard to JSON representation attribute types, services SHOULD prefer the native JSON boolean type over strings containing "true", "false", "yes", "no", etc.

**3.3.13** The success or failure of an API request MUST be reflected in the HTTP result code of the response, and therefore services SHOULD NOT return JSON entity data in HTTP responses for the purpose of indicating the success or failure of an API request, as this would be redundant and potentially conflict with the HTTP result code. However, JSON entity data may be returned in such a way as to augment the status provided in the HTTP result code, providing additional detail, as would be appropriate for request failures, and described in the section on (status codes).

## 3.4 URL Format

### 3.4.1 General

**3.4.1.1**  The root of a REST resource URL path part MUST be a registered name representing the service hosting the resource.  Services MAY include an additional level of identification to represent a subset, or API-class, of the service's overall API.  Service names and API-class names SHOULD be chosen carefully such that these need not change when products are versioned or rebranded.  The unqualified hostname (or bottom level subdomain) of the domain part of a REST resource URL MAY match the service API name, otherwise it MUST be "api".

**template**:

```
https://{domain}/{service}/{...}
https://{domain}/{service}/{apiclass}/{...}
```


**examples**:

```
https://widgets.example.com/files/{...}
https://api.example.com/contacts/scim/{...}
```



**3.4.1.2** A resource MAY be accessible through any number of URLs, however a resource MUST have one canonical URL.  A resource's canonical URL MUST NOT contain context dependent tokens (e.g. `@me`), and SHOULD NOT contain query parameters, or fragment parts.

**example:**

```
https://widgets.example.com/files/v1/documents/1111
```


**invalid example:**

```
https://social.example.com/social/v1/@me/documentlibrary?filterBy=id&filterValue=1111
```


Both URLs above may return the same resource representation, however the first represents a valid canonical URL whereas the second does not.

**3.4.1.3** A resource's canonical URL MUST be globally unique in space and time, such that it never represents a resource other than that for which it was originally defined.

**3.4.1.4** A resource's canonical URL MUST be immutable, such that it may be used to access its associated resource at any point in time for at long as that resource exists.

**3.4.1.5** A service MUST NOT apply any semantic interpretation of the domain portion of the URL or HTTP `Host:` header when servicing a request.  That is, the host and domain portion of a request MUST be regarded as opaque.

**3.4.1.6** For the purpose of these guidelines, we define two types of HTTP endpoints. **UI endpoints** are URLs that provide access to resources intended for direct use by a web browser (e.g. HTML, graphics, JavaScript, etc.). **API endpoints** are URLs that provide access to resources intended for use by application clients and other services (e.g. structured data in the form of JSON documents, vCards, etc.).  Further, we define **vanity domain** to mean any FQDN containing a customer-specific identifier (e.g. api.company.example.com).  Given these definitions, the following applies for the usage of vanity domains in cloud-based service deployments:

*  A service deployment MAY support vanity domains in the domain part of UI endpoints.

*  A service deployment MAY support vanity domains in the domain part of API endpoints if and only if the service deployment is already in production and in use by customers outside of Cisco.

*  In all other cases, a service deployment MUST NOT support vanity domains.

**3.4.1.7** When representing an array of values through URL query parameters, a service MUST support discrete representation of each value of the array as a separate name/value pair in the query part of the URL.

**example:**

```
https://widgets.example.com/files/v1/documents/id=1111&id=2222&id=3333
```


**3.4.1.8** When representing an array of values through URL query parameters a service MAY, in addition to the format described in 3.4.1.7, also support a single string comma-delimited concatenation of the values of the array as a single name/value pair in the query part of the URL.  This is provided the valid values of the attribute do not allow for embedded commas.

**example:**

```
https://widgets.example.com/files/v1/documents/id=1111,2222,3333
```


### 3.4.2 User Scoped Endpoints

**3.4.2.1** A service MAY support the use of a `@me` token in any URL, and interpret it as the user ID corresponding to the authenticated request originator.

**template**:

```
https://{domain}/{service}/{version}/{...}/@me/{...}
```


**example**:

```
https://widgets.example.com/files/v3/documents/@me/public
```


may be interpreted as:

```
https://widgets.example.com/files/v3/documents/b569fe/public
```


where `b569fe` is the `userId` of the request originator.

**3.4.2.2** In the absence of an explicit user ID or `@me` token in the URL, a service MUST NOT assume an endpoint is intended to refer to a resource related to the authenticated request originator.

**example**:

```
https://widgets.example.com/files/v3/documents
```


The above cannot be assumed to represent documents owned by the request originator.

**3.4.2.3** A service MUST NOT require the use of a `@me` token in any URL.


## 3.5 HTTP Headers

### 3.5.1 Standard Headers

**3.5.1.1** Services SHOULD support the `ETag` header in any HTTP response where it is reasonable for clients or proxies to cache the associated resource representation. In cases where `ETag` is supported, such resources SHOULD also support `If-Match` and `If-None-Match` headers.
Where caching is not appropriate, services MUST include a `Cache-Control` header (e.g. max-age=0, no-cache, no-store, must-revalidate) and MUST NOT include an ETag header.

**3.5.1.2** Services SHOULD support standard HTTP compression content codings, as outlined in section 3.5 of the [HTTP spec](#references).

**3.5.1.3** If a request includes an HTTP `Accept` header, the service MUST return a resource representation corresponding to a type presented in that header, or return an appropriate error code.  Exceptions include the following list of formats, which MUST be regarded as acceptance by the client of responses with JSON formatted resource representations.

```
application/x-www-form-urlencoded
text/plain
```


**3.5.1.4** Absence of an `Accept` header in a request MUST be regarded as acceptance by the client of responses with JSON formatted resource representations.

**3.5.1.5** When responding to a request with an error code, the service MAY return the JSON formatted response payload described in section 3.9 regardless of the presence or contents of an `Accept` header in the original request.

**3.5.1.6** If the last segment of the path of a request URL contains a "." (dot) the service MUST regard the portion of the URL following that dot as a "format extension", and the preceding portion of the URL as identifying the actual resource to be operated upon.

**3.5.1.7** A request made on a URL with a format extension MUST be treated as though the corresponding format were provided as an `Accept:` header in the request (based on Apache MIME to file extension mappings [MIME2EXT](#references).  If the request contains an explicit `Accept:` header, the explicit `Accept` header MUST be disregarded in favor of the format extension.

**3.5.1.8** A request made on a URL with a format extension, and including entity content, MUST result in an error response if the request's `Content-Type:` header does not match the format indicated by the format extension (based on Apache MIME to file extension mappings [MIME2EXT](#references).  Exceptions include cases where the `Content-Type` header is either missing from the request, or indicates one or more of the following formats:

```
application/x-www-form-urlencoded
text/plain
```

In these exception cases, the explicit `Content-Type` header MUST be disregarded in favor of the format extension.

**3.5.1.9** A service MUST support [CORS](#references) simple and preflight request flows.  Services SHOULD return "**" as the `Access-Control-Allow-Origin` header, unless the request is accompanied by an `Origin` header, in which case the service SHOULD return an `Access-Control-Allow-Origin` header with a value equal to that of the received `Origin` header.  Services MUST NOT return an `access-control-allow-credentials` header in any HTTP response.


### 3.5.2 TrackingID Header

**3.5.2.1** A service MUST include a unique `TrackingID` header in each REST API request it sends to another service. The value of the `TrackingID` MUST include a `sendertype` and a `uuid` part, and MAY include one or more `nvpair` parts, and MAY include one or more `sequence` parts.  The format of the `TrackingID` value MUST be structured as defined in the following ABNF template:

**template:**

```
trackingid = sendertype uuid *nvpair *sequence
sendertype = 1*ALPHA
uuid = "_" 8HEXDIG "-" 4HEXDIG "-" 4HEXDIG "-" 4HEXDIG "-" 12HEXDIG
nvpair = "_" name ":" value
sequence = "_" 1*DIGIT
name = 1*ALPHA
value = 1*ALPHA
```


**examples:**

```
WX2_550e8400-e29b-41d4-a716-446655440000
WX2_550e8400-e29b-41d4-a716-446655440000_0
WX2_550e8400-e29b-41d4-a716-446655440000_1_1_3
WX2_550e8400-e29b-41d4-a716-446655440000_locus:1234
WX2_550e8400-e29b-41d4-a716-446655440000_locus:1234_0_1
WX2_550e8400-e29b-41d4-a716-446655440000_locus:1234_calliope:5678
WX2_550e8400-e29b-41d4-a716-446655440000_locus:1234_calliope:5678_1_2
```


**3.5.2.2** The `sendertype` part of a `TrackingID` MUST uniquely identify the immediate sender of the HTTP request within which the `TrackingID` is embedded.

**3.5.2.3** The `uuid` part of a `TrackingID` MUST be a standard 8-4-4-4-12 hex string representation of a unique 128 bit value.

**3.5.2.4** The optional `nvpair` part of a `TrackingID`, if present, MUST take the form of one or more name/value pairs, the semantics of which are left to the discretion of the service. Note, due to the use of _ as a delimiter for the components of a `TrackingID`, services MUST NOT use _ within the name or value fields of the `nvpair`.

**3.5.2.5** A service MUST recognize when a received REST request contains a `TrackingID` header, and SHOULD include the `uuid`, `nvpair`, and `sequence` parts of the received `TrackingID` value when making downstream REST requests to other services while acting on behalf of the received upstream request.  Note that when carrying `TrackingID` information from an upstream request to a downstream request, a service MUST replace the `sendertype` with its own, MAY append additional entries to the `nvpair` part, and MAY append additional call sequence information to the `sequence part`.  If additional call sequence information is appended, the values SHOULD represent the count of all REST calls made thus far by that service as part of satisfying the received request (beginning with 0).

**example:**

_(note: for clarity we don't include any `nvpair` data in this example)_

[tracking id](trackingid-flow.png)

**3.5.2.6** When logging activities related to the production of a response to a REST API call, a service SHOULD include the corresponding `TrackingID` as part of the log message.  This permits for back-end correlation of log information useful for troubleshooting.

## 3.6 HTTP Verbs

### 3.6.1 POST

**3.6.1.1** A POST operation on a URL including `method` as a query parameter MUST be regarded as an [alternative forms](#references) request.

**3.6.1.2** A POST operation for which the last segment of the URL path is `invoke` MUST be regarded as an [action resource](#references) request.

**3.6.1.3** A POST operation for which neither **3.6.1.1** nor **3.6.1.2** apply, MUST be regarded as a request to create a new resource within the collection endpoint identified by the request URL.  A success response to such a request MUST be accompanied by a unique service-generated canonical URL, subordinate to the collection endpoint, and referring to the newly created resource.  This URL MUST be returned to the request originator in the form of both a `Location` header in the HTTP response, as well as the JSON formatted narrow representation of the resource.

**example request**:

```
POST /files/v3/documents HTTP/1.1
...
```


**example response**:

```
HTTP/1.1 201 Created
Location: http://widgets.example.com/files/v3/documents/35bd3e
...

{
    "url" : "http://widgets.example.com/files/v3/documents/35bd3e"
}
```



### 3.6.2 PUT

**3.6.2.1** The PUT verb MAY be used in any case where a client requests the direct modification of a resource's state.  Such a request MUST be idempotent with respect to the resource state.  In the event that no resource exists at the given URL, the service MAY regard a PUT as a request to create such a resource at that URL, with an initial state determined by the request payload.

**3.6.2.2** A field omitted by a JSON representation submitted as part of a PUT request, but recognized by the server as a field of the resource that is modifiable by the client, MUST be regarded by the server as a request to remove that field from the resource if present.

**3.6.2.3** A field omitted by a JSON representation submitted as part of a PUT request, but recognized by the server as a field of the resource that is not modifiable by the client, MUST be regarded by the server as constituting no requested change for that field.

**3.6.2.4** A field included in a JSON representation submitted as part of a PUT request, but recognized  by the server as a field of the resource that is not modifiable by the client, MUST be regarded by the server as an illegal request if and only if the field value differs from the resource's current value for that field.


### 3.6.3 GET

**3.6.3.1** The GET verb MAY be used for retrieving representational state from a resource. Such a request MUST have no apparent affect on the state of the resource.  However, side effects MAY include incidental changes in state of the server, other resources, or client-invisible fields of the target resource itself, as might be expected in cases where view counts or client metrics are being tracked. &nbsp;

**3.6.3.2**  The GET verb MAY be used to retrieve the states of multiple resources in a single request.  The selection of resources MAY be implicit as with a GET on a collection resource, or explicit as with a GET on a collection or search resource accompanied by selection criteria as query parameters.

**examples**:

```
GET /files/v3/documents
GET /files/v3/documents?id=1111,2222,3333
GET /files/v3/search?author=shakespeare
```


In response to such a request, a service SHOULD return JSON structured data containing only the [reference representation](#references) of each resource matching the request.  The service SHOULD NOT, by default, return the [narrow](#references) or [wide](#references) representational state of each resource in the response unless explicitly requested by the client through additional query parameters 3.6.3.6.

**[template](#references)**:

```
reference_representation {
    "url": uri full http
}

root {
   "items": [ 0* reference_representation ]
}
```


**example**:

```
{
    "items":
    [
        { "url": "https://widgets.example.com/files/v3/documents/234e" },
        { "url": "https://widgets.example.com/files/v3/documents/98ef" },
        { "url": "https://widgets.example.com/files/v3/documents/d4b3" }
    ]
}
```

**3.6.3.3** Services MAY support pagination for GET operations on collection endpoints.  If supported, the request MUST accept limit and offset parameters where the limit is the maximum number of resource to be returned, and offset is the index within the result set of the first resource to be returned.

**template**:

```
/{service}/{version}/{collectionpath}?limit={limit}&offset={offset}
```


**example**:

```
GET /files/v3/documents?limit=25&offset=0
```


**3.6.3.4** When returning paginated results, a service MUST support JSON formatted responses based on the following template and field descriptions.

**[template](#references)**:

```
reference_representation {
    "url": uri full http
}

root {
   "items" [ 0* reference_representation ],
   "paging": {
        "next" : [ 0* : uri format http ],
        ?"prev" : [ 0* : uri format http ],
        ?"limit" : integer,
        ?"offset" : integer,
        ?"pages" : integer,
        ?"count" : integer
   }
}
```


The **next** field is an array in which the entry at index 0 is a URL for the immediately subsequent page, the entry at index 1 is a URL for the next subsequent page, and so on.  If present, the "next" field MUST have a length of at least 1 in any case where there exists a subsequent page in the result set, but MAY include more.  Services supporting pagination MUST support the "next" field.

The **prev** field is an array in which the entry at index 0 is a URL for the immediately previous page, the entry at index 1 is a URL for the next previous page, and so on.  If present, the "prev" field MUST have a length of at least 1 in any case where there exists a previous page in the result set, but MAY include more.  It is OPTIONAL for services supporting pagination to also support the "prev" field.

The **limit**&nbsp;field is an integer representing the maximum number of items requested by the client for the current page.  It is OPTIONAL for services supporting pagination to also support the "limit" field.

The **offset**&nbsp;field is an integer representing the 0-based index of the first item on the current page within the context of the overall requested result set.  It is OPTIONAL for services supporting pagination to also support the "offset" field.

The **pages**&nbsp;field is an integer representing the total number of pages in the paginated result set, based on the current value of "limit".  It is OPTIONAL for services supporting pagination to also support the "pages" field.

The **count** field is an integer representing the total number of elements in the collection, regardless of "offset" or "limit".  It is OPTIONAL for services supporting pagination to also support the "count" field.

**minimal example**:

```
{
    "items":
    [
        { "url": "https://widgets.example.com/files/v3/documents/234e" },
        { "url": "https://widgets.example.com/files/v3/documents/98ef" },
        { "url": "https://widgets.example.com/files/v3/documents/d4b3" }
    ],
    "paging":
    {
        "next": [ "https://widgets.example.com/files/v3/documents?limit=3&offset=9" ]
    }
}
```


**full example**:

```
{
    "items":
    [
        { "url": "https://widgets.example.com/files/v3/documents/234e" },
        { "url": "https://widgets.example.com/files/v3/documents/98ef" },
        { "url": "https://widgets.example.com/files/v3/documents/d4b3" }
    ],
    "paging":
    {
        "next":
         [
             "https://widgets.example.com/files/v3/documents?limit=3&offset=9",
             "https://widgets.example.com/files/v3/documents?limit=3&offset=12",
             "https://widgets.example.com/files/v3/documents?limit=3&offset=15"
         ],
        "prev":
         [
             "https://widgets.example.com/files/v3/documents?limit=3&offset=3",
             "https://widgets.example.com/files/v3/documents?limit=3&offset=0",
         ],
        "limit": 3,
        "offset": 6,
        "pages": 42
    }
}
```


Note that, in the examples above, the "next" and "prev" URLs assume the same format as the user's original pagination query.  This is used for demonstrative purposes only, as services are free to construct these URLs in whatever way is most convenient for the pagination implementation.  Clients MUST treat these URLs as opaque.

**3.6.3.5** Service MAY support paging in a non-transactional manner, allowing for the possibility that changes made to a resource collection's membership may affect the responses received by clients in the process of iterating over the collection's paginated contents.  This implies that clients MUST be prepared to handle race conditions resulting in the apparent duplication or omission of resources in paginated results.

**3.6.3.6**  Services MAY support partial retrieval of resource state, based on GET query parameters.  If partial retrieval is supported, the request MUST be formatted according to the following template, and MUST return only those fields requested.  Field identifiers MUST conform to valid URL query component syntax as defined in [RFC-3986](#references), specifically the ABNF syntax specification for `pchar+`.

template:
```
/{service}/{version}/{resourcepath}?fields={fieldId,*}
```

example request:

```
GET /meetings/v3/files/34d3a?fields=url,name
```

example response:

```
HTTP/1.1 200 OK
Content-Type: application/json
...
 
{
    "url": "https://meetings.example.com/meetings/v3/files/34d3a",
    "name": "Discuss project Ice House"
}
```

If a service supports the fields parameter, then a value of @reference, @narrow, or @wide SHOULD be interpreted as a request for the set of fields constituting the reference, narrow, or wide representation of the resource, respectively. Services may define additional special values preceded by an @ (e.g. @basic) to describe specific field sets as warranted. Note, this specification does not obviate the considerations put forward in 3.3.6 regarding the return of representational state belonging to resources hosted by remote services.

Services MAY also recognize the fields query parameter when servicing requests on collection resources. In that context, the fields parameter represents an explicit request by the client that the named fields be returned for each resource included in the response.

example request:

`GET /meetings/v3/files?fields=url,name`

example response:

```
HTTP/1.1 200 OK
Content-Type: application/json
...
 
{
    "items": [
        {
            "url": "https://meetings.example.com/meetings/v3/files/34d3a",
            "name": "Discuss project Ice House"
        },
        {
            "url": "https://meetings.example.com/meetings/v3/files/64fe1",
            "name": "Discuss project Wynkoop"
        }
    ]
}
```

**3.6.3.7** Any entity data accompanying a GET request MUST be ignored by the service.

### 3.6.4 DELETE

**3.6.4.1** The DELETE verb MUST be interpreted as a request to delete the specified resource. The request must fail in the event no resource exists with the given URL.

**3.6.4.2** The DELETE verb MAY be implemented as a soft delete by default. In this case, a subsequent GET request on the same resource, or any collection having contained that resource, MUST NOT return the resource unless accompanied by an include_deleted flag.

example request:

`DELETE /files/v3/documents/ab34de`

followed by:

`GET /files/v3/documents/ab34de`

fails, whereas:

`GET /files/v3/documents/ab34de?include_deleted=true`

returns the soft deleted resource, and:

`GET /files/v3/documents?include_deleted=true`

returns the contents of documents, including ab34de and all other soft deleted resources formerly contained within that collection.

**3.6.4.3** The POST verb MAY be used to reverse the soft-delete of a resource, using an "undelete" parameter.

example request:

`POST /files/v3/documents/ab34de?undelete=true`

**3.6.4.4** The DELETE verb MAY accept a "purge" parameter to infer that a delete operation MUST be irreversible.

example request:

`DELETE /files/v3/documents/ab34de?purge=true`

**3.6.4.5** Any entity data accompanying a DELETE request MUST be ignored by the service.

### 3.6.5 PATCH

**3.6.5.1** The PATCH verb MAY be used for submitting partial updates to a resource. A PATCH request SHOULD have a Content-Type declared as application/json-patch, and if so MUST conform to a JSON Patch document as defined in [JP]. A server MAY implement a subset of the operations defined in [JP], provided any limitations be clearly documented in the API reference.

### 3.6.6 HEAD

**3.6.6.1** A service MAY support HEAD in conformance with the HTTP standard for this method.

**3.6.6.2** Any entity data accompanying a HEAD request MUST be ignored by the service.

### 3.6.7 OPTIONS

**3.6.7.1** A service MUST support OPTIONS in conformance with the HTTP standard for this method.

**3.6.7.2** A service SHOULD NOT require authorization credentials for OPTIONS requests.

**3.6.7.3** Any entity data accompanying an OPTIONS request MUST be ignored by the service.

### 3.6.8 Safe and Non-Safe Methods
For the purposes of these guidelines, the term "safe" is used in the same sense as in [HTTP]. This implies HTTP requests of type GET, HEAD, and OPTIONS are safe, while those of type PUT, POST, PATCH, and DELETE are not.

**3.6.8.1** A resource field included in a JSON representation submitted as part of a PUT, POST, or PATCH, if not recognized as both a valid and client-visible field of the resource, MUST be ignored by the server.

**3.6.8.2** Any non-safe request MUST be permitted solely through the resource's canonical URL.

**3.6.8.3** Any non-safe request MUST require OAuth2 authorization.

**3.6.8.4** Any safe request MUST require OAuth2 authorization, unless anonymous access to the resource is an explicitly intended feature of the API.


## 3.7 Alternative Forms

**3.7.1** To offer clients a means for mitigating against non-compliant browsers and badly-behaved HTTP proxies, services MUST, in addition to the standard verb invocation, allow clients to submit any standard HTTP request using the POST verb, providing the intended verb as a method query parameter. Such requests MUST conform to the following template.

`POST /{service}/{version}/{collectionpath|resourcepath}?_method={verb}`

example requests:

```
POST /files/v3/documents/ab34de?_method=delete
POST /files/v3/documents/ab34de?_method=put
POST /files/v3/documents/ab34de?_method=patch
POST /files/v3/documents/ab34de?_method=head
POST /files/v3/documents/ab34de?_method=patch
POST /files/v3/documents/ab34de?_method=get
```

Note that a service MUST NOT support custom verbs in this way, the value of _method MUST always be a standard HTTP verb.


## 3.8 Action Resources
**3.8.1** Services SHOULD avoid using action resources. In most cases it will be possible to define object resources which represent actions and use the standard RESTful verbs to operate on those objects. Exceptions are permissible, however, where this approach reveals no acceptable RESTful alternative.

**3.8.2** An action resource request URL MUST have the following form.

template:

`https://{domain}/{service}/{version}/{resourcepath}/actions/{action}/invoke?{action-resource-parameters}`

or

`https://{domain}/{service}/{version}/{resourcepath}/actions/{action}?{action-resource-parameters}`

example:

`https://ramp.example.com/ramp/v3/actions/sleep/invoke?timeout=60`

example:

`https://ramp.example.com/ramp/v3/sources/23fed2a/actions/monitor/invoke`

where 23fed2a identifies a particular resource on which the service should begin monitoring.

**3.8.3** Action resource requests MUST be submitted using the POST verb.

example:

`POST /ramp/v3/actions/sleep/invoke?timeout=60`

## 3.9 Status Codes
**3.9.1** A service MUST return HTTP response codes in conformance with RFC-2616 and common usage.


**3.9.2** For 4xx and 5xx series status codes, the service MAY return additional information regarding status as part of the response payload. If such additional information is provided, it MUST support responses with the following JSON format.

template [JCR]:

```
messages_array  [ *:string ]
 
messages_with_codes_array [ *:{
    "description" : string,
    ?"code" : string,
    ?"location" : string
}]
 
root {
    "error" {
        "key" : string,
        "message" : messages_array / messages_with_codes_array
    },
    "trackingId" : string
}
```

Where key is an application defined error code, message contains a list of human readable explanations for the error, and trackingId is an opaque identifier for mapping protocol failures to service internal codes. The message field is not intended for end-user consumption, and is not expected to be localized. A service MAY return a message in a form where each array element is just a string, or in a form where each array element is an object containing a description, an optional application defined code, and an optional location indicator. Note that values for key and code MUST NOT be used in such a way as to override or alter the semantic meaning of the response message's HTTP status code. The value of location, if present, MUST be a string formatted according to the "JSONPath" syntax as described in JSONPATH.

example 1:

```
{
    "error" : {
        "key" : "404",
        "message" : [
            "File kittens.jpg not found.",
            "Database connection failure."
        ]
    },
    "trackingId" : "S1_12345678-90ab-cdef-1234-567890abcdef_0"
}
```

example 2:

```
{
    "error" : {
        "key" : "404",
        "message" : [
            {
                "description": "File kittens.jpg not found.",
            },
            {
                "description": "Database connection failure.",
                "code": "ORA-01506"
            }
        ]
    },
    "trackingId" : "S1_12345678-90ab-cdef-1234-567890abcdef_0"
}
```

**3.9.3** Information returned response payloads MUST NOT reveal internal implementation details of the server, for example Java class names or stack traces.

**3.9.4** Documentation of response payloads MUST NOT reveal internal implementation details of the server.

## 3.10 API Proxy

**3.10.1** New RESTful web services MUST NOT be designed to require the relay and/or translation of API requests through other services (i.e. proxying).

**3.10.2** Existing web services with extensive non-RESTful API's MAY utilize a proxy model to relay and/or translate API requests (e.g. CWCAPI Gateway).

## 3.11 Bulk, Batch, and Multi-Result Operations
For the purposes of these guidelines, a "batch" operation is defined as any API request designed for the express purpose of encapsulating potentially dissimilar HTTP requests for non-atomic processing by the server and independent status code reporting for each operation.

For the purposes of these guidelines, a "bulk" operation is defined as an atomic operation consisting of one HTTP method, one URL, one set of HTTP request headers, and applied uniformly across two or more resources of the same type and contained within the same resource collection. The HTTP status code of a response to a bulk request MUST indicate either complete success or complete failure of the operation, with a 4xx or 5xx code implying there has been no net change to the state of any of the resources targeted by the operation. In the event of a failure response, it is permissible to indicate within the response entity data (as defined in section 3.9) which resource(s) are responsible for the operation failure.

**3.11.1** Services MUST NOT implement batch operations unless and until there exists empirical evidence that such operations would improve user experience or system performance. If such evidence arises, a specification for such operations MUST be developed, documented, and reviewed prior to its use.

**3.11.2** Services MUST NOT implement any type of multi-result response strategy unless and until a use case is identified that cannot be satisfied by extension of the existing single-result status response described in section 3.9. In such event that multi-result responses are needed (for example, if batch operations were to be specified), a specification for such responses MUST be developed, documented, and reviewed prior to its use. Note this should not be construed as a restriction on the standard use of the 207 status code (RFC-4918, section 13) to reflect the results of a single non-batched operation.

**3.11.3** Services MAY implement bulk operations, as defined at the beginning of this section.

**3.11.4** When implementing a bulk operation of any type other than POST, the service MUST accept requests with URLs structured in a manner identical to that of the equivalent non-bulk requests, save for the absence of a singular resource identifier. In place of the singular resource identifier, the service MUST accept query parameters sufficient to unambiguously identify that set of resources to which the operation is to be applied.  When implementing a bulk operation of type POST, the service MUST accept requests with URLs structured in a manner identical to that of the equivalent non-bulk requests (no additional query parameters are required).

examples:

Given the following non-bulk REST operation for updating a single file resource:

`PATCH /files/v3/documents/132 HTTP/1.1`

The following would be valid bulk request equivalents to that operation:

```
PATCH /files/v3/documents?id=132&id=212&id=343 HTTP/1.1
PATCH /files/v3/documents?author=bob HTTP/1.1
PATCH /files/v3/documents?filter=age%20gt%203600 HTTP/1.1
```

The following, however, would not be valid bulk request equivalents to that operation:

```
PATCH /files/v3/some/other/collection?id=132&id=212&id=343 HTTP/1.1
PATCH /files/v3/documents/132?id=212&id=343 HTTP/1.1
PATCH /files/v3/documents HTTP/1.1
POST /files/v3/documents?id=132,212,343 HTTP/1.1
```

**3.11.5** When implementing a POST, PUT, or PATCH bulk operation, a service MAY expect request entity data containing multiple resource representations. If a service does expect request entity data containing multiple resource representations, the general format of that entity data MUST conform to the following template, where resource_representation signifies request entity data that would be legal for the corresponding non-bulk operation.

template [JCR]:

```
resource_representation { ... }
 
root {
    "items" [ *:resource_representation ]
}
```

Note, a service MAY use the specific name of the resource collection rather than the generic "items" as the name for the array.

example:
A bulk create of domains in an identity service might look something like this:

```
POST /organization/acme/v1/domains HTTP/1.1
Host: identity.example.com
 
{
    "items": [
        {
            "domain": "example1.com",
        },
        {
            "domain": "example2.com",
        },
        {
            "domain": "example3.com",
        }
    ]
}
```

or

```
POST /organization/acme/v1/domains HTTP/1.1
Host: identity.example.com
 
{
    "domains": [
        {
            "domain": "example1.com",
        },
        {
            "domain": "example2.com",
        },
        {
            "domain": "example3.com",
        }
    ]
}
```

The response, if successful, might then have the following format, consistent with what a bulk GET would return:

```
HTTP/1.1 201 Created
Location: https://identity.example.com/organizations/{orgid}/v1/domains?id=9876&id=5432&id=6789
 
{
    "domains": [
        {
            "url": "https://identity.example.com/organizations/{orgid}/v1/domains/9876",
            "domain": "example1.com",
        },
        {
            "url": "https://identity.example.com/organizations/{orgid}/v1/domains/5432",
            "domain": "example2.com",
        },
        {
            "url": "https://identity.example.com/organizations/{orgid}/v1/domains/6789",
            "domain": "example3.com",
        }
    ]
}
```

## 3.12 Performance
**3.12.1** The API of a service MUST NOT be designed in such a way as to require or encourage the frequent polling of any resource by consuming clients or services.


## 3.13 Versioning
The versioning of REST APIs is necessary to accommodate the sometimes unavoidable introduction of non-backward compatible changes. That is, changes which are likely to break compatibility with clients developed to work with earlier versions of the API. The recommended practice for the continuous evolution of a REST API is to avoid making non-backward compatible changes, but it is practical to consider the case where necessary changes cannot meet this standard.

The following represent backward compatible API changes.

* Introduction of new URL path structures and endpoints.
* Introduction of new optional query parameters, and entity fields.
* Disregarding previously recognized query parameters and entity fields (provided the expected behavior remains consistent).
* Removal of access restrictions on existing resources.
* Introduction of support for new media types.
* Introduction of new error response keys and codes (as defined in STATUSCODES).
* Deprecation of existing error response keys and codes.

The following represent non-backward compatible API changes.

* Changes in URL path structure for previously existing resources (example).
* Introduction of new required request query parameters (example).
* Introduction of new required request entity fields (example).
* Change in expected behavior in the absence of new fields (example).
* Change in interpretation of previously recognized query parameters or entity data.
* Change in value type or format of previously recognized query parameters or entity data (example).
* New resource access restrictions due to authorization policy changes.
* Removal of previously existing resource endpoints.
* Removal of fields previously included in response entities (example).
* Rejection of previously recognized query parameters and entity fields.
* Discontinued support of previously recognized media types.
* Redefinition of existing error response keys and codes.

**3.13.1** A sevice API version token MUST be incorporated into URL paths as shown below, and MUST have the format "v#" where # is a single integer value representing the API version number.

template:

`https://{domain}/{service}/{version}/{...}`

example:

`[https://widgets.example.com/files/v3/]{...}`

**3.13.2** Services SHOULD evolve their APIs through backward compatible changes. Services SHOULD NOT make non-backward compatible API changes when there exist reasonable backward compatible alternatives.

**3.13.3** Services MAY increment their API version number if and only if non-backward compatible changes have been introduced.

**3.13.4** Upon deploying with a new API version number, a service MAY mark previous versions of an endpoint as deprecated (see endpoint classification).  Once deprecated, a service MUST support requests on a deprecated version of the endpoint for the duration of its deprecation period.

**3.13.5** A service SHOULD return an HTTP 301 or 307 response, with appropriate location header, when a request URL indicates a version that is not currently deployed for the requested resource.

If the requested version is lesser than the least current deployed version of the endpoint, a 301 SHOULD be returned with a URL pointing to the least current deployed version of the endpoint.

If the requested version is greater than the most current deployed version of the endpoint, a 307 SHOULD be returned with a URL pointing to the most current deployed version of the endpoint.

examples:

```
GET /files/v1/documents/1111 HTTP/1.1
Host: widgets.example.com
```

Should trigger a response of:

```
HTTP/1.1 301 Moved Permanently
Location: [https://widgets.example.com/files/v2/documents/1111]
GET /files/v3/documents/1111 HTTP/1.1
Host: widgets.example.com
```

Should trigger a response of:

```
HTTP/1.1 307 Temporary Redirect
Location: [https://widgets.example.com/files/v2/documents/1111]
```

**3.13.6** Services that host resources containing links to remotely hosted resources MUST provide a means whereby an existing deployment can efficiently update the domain and version components of those remote links. This is in consideration of the need to update remote links when a remote service has rev'd its API or changed its service domain.

**3.13.7** It is RECOMMENDED that a service implement an API endpoint for flexibly updating remote links contained within its resources. An endpoint for this purpose SHOULD be an action resource such as:

`https://{domain}/{service}/{version}/actions/updatelinks/invoke`

and SHOULD accept a JSON formatted request entity based on the following template:

```
root {
"template": string,
"match": {
        ...
    }
"replace": {        ...    }
}
```

Where the template field is an RFC-6570 URI template, which when combined with the properties of the match object form an expansion on which to match remote links with that particular base URL. Those matched URLs are updated by the service such that the match fields are replaced using the corresponding properties of replace.

example:

```
POST /meetings/v1/actions/updatelinks/invoke HTTP/1.1
Host: meetings.example.com
Content-type: application/json; charset=utf-8
...
 
{
"template": "https://identity.example.com/identity/{version}",
"match": {
        "version": "v1"
    }
"replace": {
        "version": "v2"
    }
}
```

The above request instructs the server to update all links contained within locally hosted resources and beginning with:

`[https://identity.example.com/identity/v1]`

to now begin with:

`[https://identity.example.com/identity/v2]`
 
# References

* [RWS]	RESTful Web Services <http://shop.oreilly.com/product/9780596529260.do>.
* [ROS]	RESTful Objects Specification <http://restfulobjects.org>.
* [DISC1]	Google APIs Discovery Service <https://developers.google.com/discovery/v1/using#discovery-doc-apiproperties>.
* [DISC2]	Aggregated Service Discovery <http://tools.ietf.org/html/draft-daboo-aggregated-service-discovery-01>.
* [MIME2EXT]	Apache MIME to File Extension Mapping <shttp://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types>.
* [JCR]	A Language for Rules Describing JSON Content <http://tools.ietf.org/html/draft-newton-json-content-rules-00>.
* [JP]	JSON Patch <http://tools.ietf.org/html/rfc6902>.
* [URI]	URI Template <http://tools.ietf.org/html/rfc6570>.
* [HTTP]	Hypertext Transfer Protocol – HTTP/1.1 <http://www.ietf.org/rfc/rfc2616.txt>.
* [OAUTH4]	The OAuth 2.0 Authorization Framework: Bearer Token Usage <http://tools.ietf.org/html/rfc6750>.
* [RFC3986]	RFC-3986 <http://www.ietf.org/rfc/rfc3986.txt>.
* [CORS]	Cross-Origin Resource Sharing <http://www.w3.org/TR/cors/>.
* [RFC3339]	Date and Time on the Internet: Timestamps<http://www.ietf.org/rfc/rfc3339.txt>.
* [SCIMSCHEMA]	SCIM Schema <http://tools.ietf.org/html/draft-ietf-scim-core-schema>.
* [JSONPATH]	JSONPath <http://goessner.net/articles/JsonPath/>.

Copyright 2015 Cisco Systems, Inc
