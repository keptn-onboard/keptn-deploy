# deploy

# CloudEvent Specification

## Abstract

CloudEvents is a vendor-neutral specification for defining the format
of event data.

## Status of this document

This document is a working draft.

## Table of Contents
- [Overview](#overview)
- [Type System](#type-system)
- [Data Attribute](#data-attribute)
- [Specifications](#specifications)

## Overview

## Type System

### type
* Type: `String`
* Description: Type of occurrence which has happened. Often this
  attribute is used for routing, observability, policy enforcement, etc.
  The format of this is producer defined and might include information such
  as the version of the `eventtype` - see
  [Versioning of Attributes in the Primer](primer.md#versioning-of-attributes)
  for more information.
* Constraints:
   * REQUIRED
   * MUST be a non-empty string
   * SHOULD be prefixed with a reverse-DNS name. The prefixed domain dictates
            the organization which defines the semantics of this event type.
* Examples
   * com.github.pull.create
   * com.example.object.delete.v2

### specversion
* Type: `String`
* Description: The version of the CloudEvents specification which the event
  uses. This enables the interpretation of the context. Compliant event 
  producers MUST use a value of `0.2` when referring to this version of
  the specification.
* Constraints:
  * REQUIRED
  * MUST be a non-empty string

### source
* Type: `URI-reference`
* Description: This describes the event producer. Often this will include
  information such as the type of the event source, the organization
  publishing the event, the process that produced the event, and some unique
  identifiers. The exact syntax and semantics behind the data encoded in the URI
  is event producer defined.
* Constraints:
  * REQUIRED
* Examples
    * https://github.com/cloudevents/spec/pull/123
    * /cloudevents/spec/pull/123
    * urn:event:from:myapi/resourse/123
    * mailto:cncf-wg-serverless@lists.cncf.io

### id
* Type: `String`
* Description: ID of the event. The semantics of this string are explicitly
  undefined to ease the implementation of producers. Enables deduplication.
* Examples:
  * A database commit ID
* Constraints:
  * REQUIRED
  * MUST be a non-empty string
  * MUST be unique within the scope of the producer

### time
* Type: `Timestamp`
* Description: Timestamp of when the event happened.
* Constraints:
  * OPTIONAL
  * If present, MUST adhere to the format specified in
    [RFC 3339](https://tools.ietf.org/html/rfc3339)

### schemaurl
* Type: `URI`
* Description: A link to the schema that the `data` attribute adheres to.
  Incompatible changes to the schema SHOULD be reflected by a different URL.
  See
  [Versioning of Attributes in the Primer](primer.md#versioning-of-attributes)
  for more information.
* Constraints:
  * OPTIONAL
  * If present, MUST adhere to the format specified in
    [RFC 3986](https://tools.ietf.org/html/rfc3986)

### contenttype
* Type: `String` per [RFC 2046](https://tools.ietf.org/html/rfc2046)
* Description: Content type of the `data` attribute value. This attribute
  enables the `data` attribute to carry any type of content, whereby format
  and encoding might differ from that of the chosen event format. For example,
  an event rendered using the [JSON envelope](./json-format.md#3-envelope)
  format might carry an XML payload in its `data` attribute, and the
  consumer is informed by this attribute being set to "application/xml". The
  rules for how the `data` attribute content is rendered for different
  `contenttype` values are defined in the event format specifications; for
  example, the JSON event format defines the relationship in
  [section 3.1](./json-format.md#31-special-handling-of-the-data-attribute).

  When this attribute is omitted, the "data" attribute simply follows the
  event format's encoding rules. For the JSON event format, the "data"
  attribute value can therefore be a JSON object, array, or value.

  For the binary mode of some of the CloudEvents transport bindings,
  where the "data" content is immediately mapped into the payload of the
  transport frame, this field is directly mapped to the respective transport
  or application protocol's content-type metadata property. Normative rules
  for the binary mode and the content-type metadata mapping can be found
  in the respective transport mapping specifications.

* Constraints:
  * OPTIONAL
  * If present, MUST adhere to the format specified in
    [RFC 2046](https://tools.ietf.org/html/rfc2046)
* For Media Type examples see [IANA Media Types](http://www.iana.org/assignments/media-types/media-types.xhtml)

## Data Attribute

As defined by the term [Data](#data), CloudEvents MAY include domain-specific
information about the occurrence. When present, this information will be
encapsulated within the `data` attribute.

### data
* Type: `Any`
* Description: The event payload. The payload depends on the `type` and
  the `schemaurl`. It is encoded into a media format
  which is specified by the `contenttype` attribute (e.g. application/json).
* Constraints:
  * OPTIONAL

## Specifications

The following example shows a CloudEvent serialized as JSON:

``` JSON
{
    "specversion" : "0.2",
    "type" : "com.github.pull.create",
    "source" : "https://github.com/cloudevents/spec/pull/123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleextension2" : {
        "othervalue": 5
    },
    "contenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```

### Pull Request Event - (GitHub Example)

``` JSON
{
    "specversion" : "0.2",
    "type" : "sh.keptn.events.pullrequest.merged",
    "source" : "https://github.com/keptn/sockshop/carts/pull/[pr-id]",
    "id" : "ADi3-DFDF-6785",
    "time" : "2018-04-05T17:31:00Z",
    "contenttype" : "application/json",
    "data" : {
        "payload" : {
            "pullrequest" : "pr-1",
            "service" : "carts",
            "branch" : "master"
        }
    }
}
```

### Push Event - (GitHub Example)

``` JSON
{
    "specversion" : "0.2",
    "type" : "sh.keptn.events.push",
    "source" : "https://github.com/keptn/sockshop/config/",
    "id" : "ADi3-DFDF-6784",
    "time" : "2018-04-05T17:31:00Z",
    "contenttype" : "application/json",
    "data" : {
        "payload" : {
            "service" : "carts",
            "branch" : "dev"
        }
    }
}
```

### Deploy Event

``` JSON
{
    "specversion" : "0.2",
    "type" : "sh.keptn.events.deploy.finished",
    "source" : "https://https://us-central1-sai-research.cloudfunctions.net/githubWebhookListener/",
    "id" : "ADi3-DFDF-6712",
    "time" : "2018-04-05T17:31:00Z",
    "contenttype" : "application/json",
    "data" : {
        "payload" : {
            "status" : "success",
            "environment" : "dev",
            "service" : "carts",
            "artifact" : "10.43.249.155:5000/library/sockshop/carts:pr-33"
        }
    }
}
```