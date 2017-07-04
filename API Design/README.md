This documentation is intended as a guide for designing a robust REST API.

In order maximize usefulness, a well designed API must meet certain objectives:
* It must be lightweight (i.e. minimal external dependencies, configuration, and setup)
* It must _require_ no special tooling (even if tooling to facilitate its use is available)
* It must use loose integration contracts (i.e. promote loose coupling)
* It should be self-documenting and discoverable, assuming little or no pre-existing knowledge by API consumers
* It should be versionable in order to handle changes to underlying business objects (resources)

Taken together, these factors maximize both robustness and ease of use.

Although this guide will be useful to programmers, it is not a programmer's guide and will not recommend specific implementation frameworks. The practicalities related to implementing and running the resultant design are also not part of this document, but will be covered elsewhere.

While REST is described apart from any specific implementation, it is undeniable that HTTP both influenced that description and remains the only fully RESTful protocol. And now there is a new standard in the HTTP world - HTTP/2.

The idea of "version 2" is, however, a bit of a misnomer. The specification isn't _actually_ a new version of HTTP as a whole, it is "simply" a new line protocol that can significantly improve the performance and security of HTTP interactions between client and server.

Nevertheless, in order to coherently specify this new line protocol, the HTTP 1.1 specification, embodied mainly in RFC 2616, had to be revised to provide a cleaner separation of the line protocol from the application protocol.

As part of this revision, the authors took the opportunity to update, clarify and "tweak" the specification of HTTP 1.1 application features and, in particular, to address some of the uncertainties related to using HTTP in RESTful applications.

Consequently, where this document references HTTP, it is (unless otherwise stated) referring to the [new RFCs that describe HTTP 1.1](http://httpwg.org/specs/) and not to the (now deprecated) RFC 2616 description.
