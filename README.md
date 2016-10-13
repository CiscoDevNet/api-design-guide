# Introduction

# Construct

Resources should describe the “objects” that your API describes, with an identifier that uniquely identifies each object;
HTTP methods are used to “change” the state of the object, with a mandated list of supported methods;
HTTP headers are used for passing mandatory arguments such as for authentication, accepted content types, etc.;
Query parameters are used for optional arguments and can be omitted as required;
Return codes should mirror the meaning and semantics of the core HTTP specification, ensuring the code is informative and embellished with extra information in the payload where required.

# Description

You should use a description language

# Data

What data should be returned: For example, how a unique identifier for a given member of a collection should be expressed, support for pagination, filtering of large GETs, etc.;

# Encoding

What encoding should be supported: The majority of new web APIs tend to implement JSON only, but there are cases where APIs support alternative encoding specifications such as XML. The style guide should define what is mandatory, what alternatives can be implemented, and how to deal with the transposition between different specifications;

# Links

The use of links: Hypermedia APIs have long been considered by many in the industry as being the way to implement data linkages across multiple endpoints or APIs, with identifiers to related data items implemented as links rather than as a simple surrogate or canonical identifier. However, doing so cohesively across an organization demands a singularity of purpose that a style guide can help deliver. Moreover, there are times when including rather than linking to data for the purposes of efficiency is desirable. The style guide therefore needs to define an organizational approach to hypermedia APIs and, where they are being promoted, define practical steps for making them work.

# Versioning

