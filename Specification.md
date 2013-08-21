# Personal Avatar 0.3.0

This version:
: [http://pavatar.com/spec/pavatar-0.3.0](http://pavatar.com/spec/pavatar-0.3.0)

Latest version:
: [http://pavatar.com/spec](http://pavatar.com/spec)

Previous versions:
: [http://pavatar.com/spec/pavatar-0.2.0](http://pavatar.com/spec/pavatar-0.2.0)
: [http://jeenaparadies.net/specs/pavatar-0.1.2](http://jeenaparadies.net/specs/pavatar-0.1.2)
: [http://jeenaparadies.net/specs/pavatar-0.1.1](http://jeenaparadies.net/specs/pavatar-0.1.1)
: [http://jeenaparadies.net/specs/pavatar-0.1.0](http://jeenaparadies.net/specs/pavatar-0.1.0)

Editors
: [Jeena Paradies](http://jeenaparadies.net/contact/) <spec@pavatar.com>

© 2006 Jeena Paradies

## Abstract

The Personal Avatar (short Pavatar) system is a way to use small personalized images to standardize a providers profile picture through various websites when leaving comments and other content. Providers are	able to host their Personal Avatars themselves.

## Status of This Document

This is the 18 December 2006 Candidate Recommendation of the Personal Avatar. This Candidate Recommendation was published to indicate that the document is believed to be stable and to encourage implementation by the developer community.

Comments are welcome, please write to <spec@pavatar.com>.

## Table of Contents

1. Definitions
2. Personal Avatar Conformance Requirements
	1. Technical Definition
	2. Refusing Personal Avatar Requests
3. Autodiscovery
	1. HTTP Header
	2. Link Element
	3. Direct URL
	4. Precedence
4. Dealing with Personal Avatars
	1. Autodiscovery Algorithm
	2. Manipulating
	3. Caching
	4. Updating Cached Personal Avatars
5. Appendix
	1. Optimization
	2. Support of Conformance to Arbitrary Rules

## 1. Definitions  

**Provider**
: Provider is the person who leaves a comment, post, annotation or similar content on a publishers page and possesses the Personal Avatar. The provider and the consumer **may** be the same.

**Providers website**
: A providers website must be a valid HTTP URL according to [RFC 1738](http://www.faqs.org/rfcs/rfc1738.html), which the provider has specified as his website.

**Personal Avatar**
: A Personal Avatar is a picture stored on the provider's website conforming to the Personal Avatar conformance requirements described in [2][conformance-requirements].

**Consumer**
: The consumer provides the service for the provider to act. The consumer and the provider may be the same.

**The Personal Avatar implementation (short: *implementation*)**
: This is the implementation running the provider's service dealing the Personal Avatars.

**Now**
: Now refers to the time the HTTP request is finished.

The key words **must**, **must not**, **required**, **shall**, **shall not**, **should**, **should not**, **recommended**, **may**, and **optional** in this document are to be interpreted as described in [RFC 2119](http://www.faqs.org/rfcs/rfc2119.html).

## 2. Personal Avatar Conformance Requirements
### 2.a. Technical Definition
A valid Personal Avatar **must** be a 80x80 pixel sized image in GIF ([GIF89a](http://www.w3.org/Graphics/GIF/spec-gif89a.txt)), PNG ([PNG](http://www.w3.org/TR/PNG/)) or JPEG ([ISO/IEC IS 10918-1], [ISO/IEC IS 14495-1], [ISO/IEC IS 15444-1]) format.

A valid Personal Avatar **must not** exceed the size of 409600 Bit (the octuple size of an uncompressed 80x80pixel big picture with 8-bit color depth).

A valid Personal Avatar **must** be publicly accessible through a valid HTTP URL ([RFC 1738](http://www.faqs.org/rfcs/rfc1738.html)). This URL **must** be used to reference the Personal Avatar (see [Autodiscovery](autodiscovery)).

The Personal Avatar **must** have the correct `Content-Type` header.

### 2.b. Refusing Personal Avatar Requests
The provider **may** restrict access to the Personal Avatar. The provider **must** act accordingly to HTTP, e.g. **should** response with a 403 HTTP status code, if access is denied. It is **recommended** to use the keyword "**none**" of the HTTP-Header `X-Pavatar` as described in [3.a](3). to refuse the delivery of the Personal Avatar completely.

## 3. Autodiscovery
The URL of the Personal Avatar **must** be offered by the provider in one or more of the following ways to the consumer, the use of [3.a]. and [3.b]. is recommended. The consumer **must** accept all three ways.

### 3.a. HTTP Header
If chosen, a reference to a Personal Avatar must be returned with a X-Pavatar HTTP header, for example:

	X-Pavatar: http://example.com/path/to/my-own_pavatar.png

The value of the `X-Pavatar` header **must** be the absolute URL of the Personal Avatar or the keyword "**none**".

The server response **must not** include more than one such header.

### 3.b. Link Element
If chosen, an HTML or XHTML Personal Avatar enabled page **must** contain a `<link>` element. It **must** have one of the following forms:

**HTML**
: `<link rel="pavatar" href="URL">`

**XHTML**
: `<link rel="pavatar" href="URL" />`

The URL in the "`href`" attribute **must** be a valid and absolute HTTP URL according to [RFC 1738](http://www.faqs.org/rfcs/rfc1738.html) or the keyword "**none**".

### 3.c. Direct URL (the favicon.ico way)
If chosen, there **must** be a file named "*pavatar.png*" on the providers server. The URL for this file **must** conform to the following form. The EBNF keywords used here are similar to the ones used in [RFC 1738](http://www.faqs.org/rfcs/rfc1738.html).

	hostport = host [ ":" port ]
	hsegment = *[ uchar | ";" | ":" | "@" | "&" | "=" ]
	puri = "http://" hostport *[ "/" hsegment ] "/pavatar.png"

**Example in Pseudocode:**

1. IF the last `hsegmet` ends with a slash: GOTO 3
2. Remove the first `hsegment`
3. Append "*pavatar.png*" to the result
4. IF success (i.e. a 2xx answer to an HEAD request) EXIT(SUCCESS)
5. ELSE remove all `hsegments`
6. Append "*pavatar.png*" to the result
7. IF success (i.e. a 2xx answer to an HEAD request) EXIT(SUCCESS)
8. EXIT(FAILURE)

### Precedence
The HTTP Header ([3.a.](3)) method **must** have the highest precedence, the Direct URL ([3.c](3c)) method the lowest.

## Dealing with Personal Avatars
The implementation should ensure to use as little traffic as possible to deal with Personal Avatars (e.g., use conditional get HTTP headers like If-Modified-Since).

### 4.a. Autodiscovery Algorithm

Personal Avatar implementations, given a publisher's website URL, **should** follow the following steps to find the Personal Avatar URL (obs. if the implementation finds the keyword "**none**" instead of a valid URL it **must** abort the autodiscovery because there will be no Personal Avatar at all):

1. Examine the HTTP headers of the response. If there are any `X-Pavatar` headers then the first such header's value **should** be used as the Personal Avatar resource. Clients **must** examine the HTTP headers if they are able to. If for some reason the HTTP headers are not available to the implementation then this step **may** be skipped, however, implementers **should** be aware that this will reduce the usefulness of their application as link elements cannot be used for resources that are neither HTML nor XHTML, and HTTP headers are defined to override link elements when they differ.
2. If there is no `X-Pavatar` HTTP Header, search the entity body for the first match of the following regular expression:

	<link rel="pavatar" href="([^"]+)" ?/?>

	If the regular expression matched, clients **must** decode the allowed HTML entities (`&` for `&amp;`, `<` for `&lt;`, `>` for `&gt;`, and `"` for `&quot;`).

3. If no `X-Pavatar` HTTP Header and no `<link>` could be found the implementation **shall** check first if there is a Personal Avatar in the publisher's website directory. If not, it **shall** check if there is a Personal Avatar in the publisher's website document root.

### 4.b. Manipulating
The implementation **may** manipulate the cached Personal Avatar if necessary, e.g. changes in width and height, colors, etc. but it **must** respect the visual identity of the provider and therefore avoid destructive transformations of the Personal Avatar, such as those that fundamentally alter the content of the image.

### 4.c. Caching
The implementation **should** cache **all** Personal Avatars it wants to serve. It **should not** serve them remotely.

### 4.d. Updating Cached Personal Avatars
The implementation **may** check regularly for changed Personal Avatars using the information given in the `Cache-Control` or `Expires` headers. If the publisher's website doesn't send these headers, it is **recommended** that it checks weekly for changes.

The implementation **should** remember URLs for which the [autodiscovery](#autodiscovery-algorithm) didn't find a Personal Avatar and not check them for changes until the provider posts something new.

## Appendix

This chapter is informative. Useragents are not required to implement the properties of this chapter in order to conform to the Personal Avatar specification.

### 5.a. Optimization
Clients **may** optimize the search. For example:

1. The client **may** initally only send an HTTP HEAD request in the hope that the header will be found and the content will not have to be fetched.
2. Since `<link>` elements may only appear in the document's head, clients **may** abort when the strings `</head>` or `<body>` are seen (e.g. if the client reads the content one line at a time).
3. Since the Personal Avatar links are most likely to appear near the top of the document, clients **may** abort the search after passing a certain size threshold. Clients **may** similarly use the HTTP Content-Range header to only fetch the first few kilobytes of the target URI.

### 5.b. Support of Conformance to Arbitrary Rules
The implementation **may** include an automatic denial-of-publish mechanism if the Personal Avatar is unknown to the system. It **may** include a notification mechanism in case of an automatic denial-of-publish.