# Response Body Expectations
This article presents the rules pertaining to the responses from RESTful requests made to the GraphTalk AIA implementation of these guidelines. In general these examples should apply to almost all non-GraphTalk AIA contexts as well.

Generally the HTTP entity/body contained in the response to a REST API request will be of
one of two types:

1. A resource representation - i.e. a formatted view of the state of the addressed resource
2. An [outcome report](#the-outcome-report) - i.e. a report detailing the outcome (success or failure) of the request

The OPTIONS request is a slightly special case because it returns interaction information pertaining to the
addressed resource in its current state, but even that can be deemed as a view of a different aspect of that
state and thus can also be classed as a resource representation.

It is also possible (but rare) that the response does not contain an entity - in which case the entire outcome
can be derived from the status code and other metadata (i.e. headers) accompanying the response.

This document summarizes what a client of the AIA REST API can expect in a response body in various
circumstances.

  * [Request or response in error](#request-or-response-in-error)
  * [GET](#get)
    * [GET of a resource (except printed documents and schema documents)](#get-of-a-resource-except-printed-documents-and-schema-documents)
    * [GET of a printed document](#get-of-a-printed-document)
    * [GET of a schema document](#get-of-a-schema-document)
  * [OPTIONS](#options)
  * [DELETE and non-creation POST](#delete-and-non-creation-post)
  * [Creation (POST-create or PUT)](#creation-post-create-or-put)
  * [Update (PATCH and PUT-update)](#update-patch-and-put-update)
  * [The Outcome Report](#the-outcome-report)
  * [Side Effect Notification Pattern](#side-effect-notification-pattern)

---

## Request or response in error
In all cases, for all requests, if an error is encountered by any layer within the REST implementation,
the client will receive

##### HTTP Status Code
    >=400
##### Significant HTTP headers
    Content-Type: application/json; charset=utf8
##### Response entity
    Outcome report

All other sections in this document assume a successful request.

---

## GET
#### GET of a resource (except printed documents and schema documents)
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/json; charset=utf8
    Link: headers related to resource type and navigation
##### Response entity
    Outcome report

#### GET of a printed document
##### Request contains
    URI of a printable resource
    Accept: application/pdf
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/pdf
    Link <headers related to resource type and navigation>
##### Response entity
    “Resource representation” in PDF bytes
##### Notes
Obviously this will only be returned if a PDF representation of the
resource is available or synchronously build-able, otherwise the JSON
representation of the resource will be returned.
#### GET of a schema document
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/json; charset=utf8
##### Response entity
    Resource representation of a schema, in JSON Hyperschema format
##### Notes
* Schemas are in JSON Hyperschema format but, at present, there
is no specific registered media type for that format - this will be addressed
* **Currently schemas don’t have any Link headers. This is 
shortcoming of the AIA API and should be fixed - there needs
to be the same Link headers as found for any business**
resource (type, up, self etc).

---

## OPTIONS
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/json; charset=utf8
##### Response entity
    OPTIONS response for the target resource in JSON Hyperschema format
##### Notes
OPTIONS responses are in JSON Hyperschema format but, at
present, there is no specific registered media type for that format -
this will be addressed

---

## DELETE and non-creation POST
### Response does not contain a body
##### HTTP Status Code
    204
##### Significant HTTP headers
    X-CSC-Modified (optional - see below)
    X-CSC-Deleted (optional - see below)
##### Response entity
    None
##### Notes
An empty response is possible in the case that
* The server has nothing to report other than possibly [side effect notifications](#side-effect-notification-pattern), *AND*
* The size of the side effect notification header values does not exceed a configured maximum (currently 25 items).

### Response contains a body
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/json; charset=utf8
##### Response entity
    Outcome report
##### Notes
* An [outcome report](#the-outcome-report) will be returned if
  * AIA returns an outcome report, *OR*
  * The size of the side effect notification header values exceeds a configured maximum (currently 25 items)
* Side effect notification headers are never returned when the response entity is an outcome report. [See below for details](#side-effect-notification-pattern).

---

## Creation (POST-Create or PUT)
### Request contains `Accept: application/vnd.hal+json`
##### HTTP Status Code
    201
##### Significant HTTP headers
    Content-Type: application/vnd.hal+json; charset=utf8
    Location: <value is the URI of the created resource>
    Content-Location: <value is the URI of the created resource>
    Link: <headers related to resource type and navigation>
    X-CSC-Modified (optional - see below)
    X-CSC-Deleted (optional - see below)
##### Response entity
    Resource representation in HAL JSON format
##### Notes
* This is the [Content-Location pattern response](Update-Responses-and-CLP.md)
* It is only sent if:
  * The request contains the specified `Accept` header, *AND*
  * The server responds with a resource representation, *AND*
  * The size of the [side effect notification](#side-effect-notification-pattern) header values does not exceed a configured maximum (currently 25 items).

### Request does NOT contain `Accept: application/vnd.hal+json`
##### HTTP Status Code
    201
##### Significant HTTP headers
    Content-Type: application/vnd.hal+json; charset=utf8
    Location: <value is the URI of the created resource>
##### Response entity
    Outcome report
##### Notes
* This is the NON-Content-Location pattern response, comprising an [outcome report](#the-outcome-report)
* It is sent if:
  * The request does not contain the specified `Accept` header, *OR*
  * The server responds without a resource representation, *OR*
  * The size of the side effect notification header values exceeds a configured maximum (currently 25 items)
* Side effect notification headers are never returned when the response entity is an outcome report. [See below for details](#side-effect-notification-pattern).

---

## Update (PATCH and PUT-update)
### Request contains `Accept: application/vnd.hal+json`
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/vnd.hal+json; charset=utf8
    Content-Location: <value is the same as the request URI>
    Link: <headers related to resource type and navigation>
    X-CSC-Modified (optional - see below)
    X-CSC-Deleted (optional - see below)
##### Response entity
    Resource representation in HAL JSON format
##### Notes
* This is the [Content-Location pattern response](Update-Responses-and-CLP.md)
* It is only sent if:
  * The request contains the specified `Accept` header, *AND*
  * The server responds with a resource representation, *AND*
  * The size of the [side effect notification](#side-effect-notification-pattern) header values does not exceed a configured maximum (currently 25 items).

### Request does NOT contain `Accept: application/vnd.hal+json`
##### HTTP Status Code
    200
##### Significant HTTP headers
    Content-Type: application/vnd.hal+json; charset=utf8
##### Response entity
    Outcome report
##### Notes
* This is the NON-Content-Location pattern response, comprising an [outcome report](#the-outcome-report)
* It is sent if:
  * The request does not contain the specified `Accept` header, *OR*
  * The server responds without a resource representation, *OR*
  * The size of the side effect notification header values exceeds a configured maximum (currently 25 items)
* Side effect notification headers are never returned when the response entity is an outcome report. [See below for details](#side-effect-notification-pattern).

---

## The Outcome Report
An outcome report returned in response to a request will have the following format:
```
{
    "outcome": an outcome description,
    "messages": [
        an array of message objects further detailing the outcome
    ]
}
```
The value of the `outcome` property is a string with one the following values
* `success`: the request succeeded unconditionally
  * the accompanying HTTP status code will be less than 300.
  * any items in the `messages` array in the report will have a severity property value of `informational`.
* `warning`: the request succeeded but AIA needs to convey some alert information to the client
  * the accompanying HTTP status code will be less than 300.
  * there will be at least one message in the messagesarray that has a severity of `warning`, but none with a severity of `error`.
* `failure`: the request failed
  * the accompanying HTTP response code will be 400 or greater.
  * there will be at least one message in the messagesarray that has a severity of error.

The default value of the outcome property is `success` if the HTTP status code is less than 300, otherwise
the default value is `failure` (but it should always be present). So responses that lack a response body
can be assumed to be successful responses with no informational messages.

In a successful outcome report, the `messages` property may be absent or have an empty array as its
value.

The objects in the `messages` array each have the following general format:
```
{
    "severity": a string indicating the severity of the message,
    "context": a string describing the context of the message,
    "message": a JSON value containing the message detail
}
```
The `context` property contains a string that provides some application context for the message which
helps the client understand more precisely the message data - for example, if a PATCH request fails
because a property setting in the request body is invalid then the context string might be the name of that
property.

The definition of the `context` property value is left deliberately vague (for now) to allow for extension in the
future. Each individual allowed interaction for a resource may define its own semantics for this property, as
may the API for general messages. A (CSC-wide) scheme of documenting context strings should probably
be devised in the future.

The `message` property can contain any JSON value applicable to the message `context`.

More often than not, it will contain a message string that the client will likely write to a report or display to
the user (in the case of warning or error messages), but some contexts may need to exploit a more
structured value for the `message` property - for example, the "side effect notification pattern" described below.

---

## Side Effect Notification Pattern
The server communicates that a request has affected the state of resources other than the
addressed resource by listing the URIs of those resources in either an `X-CSC-Modified` or an `X-CSC-Deleted`  notification (note that GraphTalk AIA still uses `X-GraphTalk-Modified` instead of `X-CSC-Modified`). The former is used for resources whose state was modified, the later for resources that were deleted.

This notification is returned to the client in one of two ways:
* In a response header named `X-CSC-Modified` or `X-CSC-Deleted`, OR
* In message objects in a standard outcome report structure, which will resemble this:

   ```
   {
       ...
       "messages": [
           ....
           {
               "context": "X-CSC-Modified",
               "message": [
                   "http://localhost:10104/csc/insurance/this/one",
                   "http://localhost:10104/csc/insurance/that/one"
               ]
           },
           {
               "context": "X-CSC-Deleted",
               "message": [
                   "http://localhost:10104/csc/insurance/another/one"
               ]
           },
           ...
       ],
       ...
   }
   ```

The detection of which mechanism has been used is fairly simple:
* if the response entity is an outcome report, then the notifications (if any) are found in that report and NOT in the headers
* otherwise, the notifications (if any) are found in headers - so:
  * any response that contains a resource representation (i.e. a [Content-Location pattern response](Update-Responses-and-CLP.md) - because a GET response will *never* convey modification notifications because GET is defined as “Safe and Idempotent” and so must not modify anything that affects the client!)
  * a response without an entity (e.g. DELETE)


So the trick is:
* Determine what the response entity contains (see all the previous sections)
* Get the side effect notifications from the appropriate place for that type of response.
