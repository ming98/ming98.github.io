# RESTful "update" Responses and Content-Location

This is an in-depth explanation of the constraints that apply to responses to update requests in a RESTful architecture, and the use of the Content-Location header to improve performance of update request cycles in the presence of those contraints - what is known as the COntent-Location Pattern (CLP).

It is structured as an FAQ in order to try and convey the information is as clear a fashion as possible.

Let's start with the basics:

## What is Content Negotiation?

"Proactive content negotiation" is where the HTTP client sends preferences to the server and the server selects an appropriate response format from those preferences.

The explanation of CLP presented here exploits this type of content negotiation, so we start with a brief clarification of what that is:

* It is a feature of HTTP that allows the client (e.g. a browser) to tell the server the preferred formats that it accepts in response to the current request.
* The client can tell the server "I prefer JSON, but will accept XML if you don't support JSON" for example, or "I prefer French responses but will accept English responses if you can't speak French"
* The client tells the server this using standard request headers whose names start with Accept.
  * The `Accept` header lists the the preferred media types. "JSON, but XML is OK", for example, can be conveyed like this (preference implied by order)
  
    `Accept: application/json, application/xml`
    
    or (preference explicitly declared, with the default being "1")
    
    `Accept: application/xml; q=0.5, application/json`
    
    
  * The `Accept-Language header` lists the preferred languages in much the same way
  * There are other `Accept` headers but we won't confuse the issue here any more
* The server will pick from the list, the highest priority format that it supports and return the response in that format.
* If the server doesn't support any of the formats, it can do one of two things:
  * It can reject the request with a `406 Not Acceptable` response, or
  * It can return a response in a default format
  
  The HTTP spec recommends the LATTER!
  
  
* BUT, and here's the wrinkle, the server CAN ignore the preferences altogether and just return a default format too.

## What is the default behavior of a POST/PATCH/PUT request?
* According the HTTP spec, the response to a POST/PATCH/PUT request is an outcome report
* It is NOT a resource representation
* That is, the client MUST assume that a 200/201 response means "the request worked, now RE-GET the resource to update your locally held representation of it".
* The body of the response can be used for some kind of reporting, but the client MUST NOT use it as a resource representation (security and all that!)
* PERIOD. NO ARGUMENT. THAT'S HOW IT WORKS.
* POST/PATCH/PUT followed by GET - the LAAAAWWWW!

## OK, that sucks! So we have to add GETs everywhere after POST/PATCH/PUT requests?

Yes you do!!!
 
Unless...

1. You code your client to properly recognize that a server has, in fact, sent you a resource representation
2. You code your client to properly ask the server to do that

## And that fixes all my "extra GET" problems then?

If you do it properly, yes!
 
And by that I mean that
1. You structure your requests properly
2. You recognize that the server may not support this optional feature
3. You recognize that even if the server does normally support the feature, under certain circumstances it can choose to ignore your "please send me a representation" preference.

## Alright, so what is the "feature" called?

It's the Content-Location Pattern.
 
It is a completely valid pattern defined in the HTTP specs as an optional feature, and whose rules are clear, cut and dried ...
... well at least for how a client can tell if the server has returned a representation instead of an outcome report.

## How do I know if the server has sent me a representation, then?

The name of the pattern gives you a clue - it's based on the presence and content of the `Content-Location` header in the response.
 
In general the header says "the body of this HTTP message contains a representation of the resource found at this location" - but, for good security reasons, the client must not simply rely on those semantics.
 
The rules are as follows:
 
* For a `201 Created` response:
  * IF
    * The `Content-Location` header is present in the response, AND
    * The value of the `Content-Location` header is equal to (byte-for-byte) the `Location` header value (which MUST be present in a `201` response)
  * THEN the response body contains a representation of the newly created resource
  * OTHERWISE the client MUST GET the resource whose URI is in the Location header in order to receive a representation of the newly created resource
 
* For a `200 OK` response,
  * IF
    * The `Content-Location` header is present in the response, AND
    *  The value of the `Content-Location` header is equal to (byte-for-byte) the original request URI
  * THEN the response body contains a representation of the modified resource
  * OTHERWISE the client MUST GET the resource whose URI was the target of the update request in order to receive a representation of the modified resource
 
The conditional code to implement these rules is almost trivial, so every client can use it!

## And that can apply to ALL POST/PATCH/PUT requests?

Almost!
 
The pattern can apply to any PATCH or PUT, but ONLY to POST requests that target "Factory resources" - i.e. resources who's sole or primary role is to create another resource - what we call POST-create requests in the Pragmatic Guide".

So, taking an extreme example, this would not apply to SOAP uses of POST, or to a POST request to an Controller resource that is not a Factory resource.
 
## Right, so how do I ask the server to apply the CLP?

The HTTP specs do not tell us this!
 
But, given that we have content negotiation and media types, we have have an effective scheme.
 
The basic approach is
* If you want the server to apply the CLP, then set the `Accept` header in the request to the media type of the resource representation of the affected resource.

  For a creation request (a POST-create or a PUT-create) this will be a media type supported by the GET method for the created resource type (listed in its documentation/schema)
  
  For an update request (a PATCH or a PUT-update) his will be a media type supported by the GET method for the target resource of the request (again, listed in its documentation/schema)

But remember three things:
1. The server may not support the CLP
2. The server may choose to not apply the CLP despite what you request
3. The server may ALWAYS apply the CLP despite what you request
 
## Why isn't the resource representation media type just listed in the POST/PATCH/PUT interaction descriptor in the schema and OPTIONS response?

Because that's not the correct DEFAULT behavior of POST/PATCH/PUT requests - see above.
 
The logic is
* The CLP is a macro for POST/PATCH/PUT followed by GET
* So if you want the pattern applied, `Accept` the response media type of a GET
* If not, `Accept` the response media type of the POST/PATCH/PUT
 
## Do you have an example?

Sure
 
Imagine an insurance application where, to create a quote, you send a POST request to the `/quotes` resource, with a body that contains at least the minimum required information to start the quote build process. Additionally, representations supported by the server are always of the media type `application/vnd.hal+json`.
 
If you send, THIS:

```
POST /quotes HTTP/1.1  
Accept: application/vnd.hal+json  
Content-Type: application/json  
<more headers>  
  
{"quote:product_id": "MC011", "quote:distributor_id": "a-distributor-id"}
```
the server recognizes that you are asking for the CLP to be applied and you get this in the response:
```
HTTP/1.1 201 Created  
Location: https://my.server.com/csc/insurance/quotes/ID-123456  
Content-Location: https://my.server.com/csc/insurance/quotes/ID-123456  
Content-Type: application/vnd.hal+json; charset=utf8  
<more headers>  
  
{  
    the hal representation of the created resource  
}
```
whereas if you use `application/json` in the `Accept` header of the request you get this:
```
HTTP/1.1 201 Created  
Location: https://my.server.com/csc/insurance/quotes/ID-123456  
Content-Type: application/json; charset=utf8  
<more headers but NOT Content-Location>  
  
{  
  "messages": {  
    "message": "Quote QT0000054321 - MC011 - gtmoni - Effective on ? created.",  
    "severity": "information"  
  },  
  "outcome": "success"  
}  
```
and so you would then need to GET the resource cited in the Location header.

## If the server does support the CLP, why wouldn't it always apply it when asked to?

Don't underestimate the importance of the content of an outcome report!
 
Mostly, when creating or updating a resource, you're going to get a simple "that worked" response as an outcome report. Since that's basically what the HTTP status code tells you anyway, the CLP makes sense.
 
But sometimes even though the request is successful, the server might decide that it has some important information related to the request to convey to the client (as opposed to information related to the targeted resource, which can be obtained from the resource representation itself). In this case, the server may force an outcome report containing relevant messages.

## Do you have an example of that?
Yes.
 
In our not-so-mythical insurance application we recognized the need to inform the client when an update request has had side-effects on other resources, so that the client can update any locally held representations of those resources. We do this using the ["Side Effect Notification Pattern"](Response-Content.md#side-effect-notification-pattern), implemented by returning the URIs of modified resources in a custom response header named `X-CSC-Modified`, while those of deleted resources are returned in a custom header named `X-CSC-Deleted`.
 
However, most HTTP clients and servers impose a limit on the length of an individual header and so in rare extreme cases we found that our responses were either causing an exception in the web server or in the client.
 
So we now place a limit on the number of URIs that can be returned in this way and, if an update request has more than this number of side effects, we list the URIs in a message in the outcome response instead, and force that through as the response to the request even when the client has asked for the CLP to be applied.
 
(Note that side effect notifications are associated with the request, NOT the addressed resource - for messages that pertain to the resource itself, we maintain a sub resource named status_report, which is always returned in the representation of the parent resource).

## In conclusion

I hope that clears up the content location pattern and its use in our REST API approach. It is an important pattern to master, and, when supported, must be applied uniformly by all implementations of our approach to REST.
 
Clients should always code for the possibility that the response to a create or update request conforms to the pattern , but should never expect that pattern unconditionally.
