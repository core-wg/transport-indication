---
title: CoAP Protocol Indication
docname: draft-amsuess-core-transport-indication-latest
stand_alone: true
ipr: trust200902
cat: std
pi:
  strict: 'yes'
  toc: 'yes'
  tocdepth: '3'
  symrefs: 'yes'
  sortrefs: 'yes'
  compact: 'yes'
  comments: yes
  subcompact: 'no'
  iprnotified: 'no'
area: Internet
wg: CoRE
kw: CoRE, Web Linking, Proxy, URI, aliasing
author:
- ins: C. Amsüss
  name: Christian Amsüss
  street: Hollandstr. 12/4
  code: '1020'
  country: Austria
  phone: "+43-664-9790639"
  email: christian@amsuess.com
normative:
informative:
  aliases:
    title: Architecture of the World Wide Web, Section 2.3.1 URI aliases
    author:
      org: W3C
    target: https://www.w3.org/TR/webarch/#uri-aliases
  cooluris:
    title: Cool URIs don't change
    author:
      name: Tim BL
    target: https://www.w3.org/Provider/Style/URI

--- abstract

The Constrained Application Protocol (CoAP) is available over different transports
(UDP / DTLS since {{?RFC7252}}, TCP / TLS / WebSockets since {{?RFC8323}}),
but lacks a way to unify these addresses.
This document provides terminology based on Web Linking {{?RFC8288}}
to express alternative transports available to a device,
and to optimize exchanges using these.

--- middle

# Introduction {#introduction}

\[ See Abstract for the moment \]

\[
This document is being developed at <https://gitlab.com/chrysn/transport-indication>.
\]

## Terminology

Same-host proxy

: A CoAP server that accepts forward proxy requests (i.e., requests carrying the Proxy-Scheme option)
  exclusively for URIs that it is the authoritative server for is defined as a "same-host proxy".

hosts

: The verb "to host" is used here in the sense of the link relation of the same name defined in {{?RFC6690}}.

  For resources discovered via CoAP's discovery interface,
  a hosting statement is typically provided by the defaults implied by {{?RFC6690}}
  where a link like `</sensor/temp>` is implied to have the relation "hosts" and the anchor `/`,
  such that a statement "coap://hostname hosts coap://hostname/sensor/temp" can be inferred.

  For many application it can make sense to assume that any resource has a "host" relation
  leading from the root path of the server
  without having performed that discovery explicitly.

  \[ TBD: The last paragraph could be a contentuous point;
  things should still work without it,
  and maybe that's even better because it precludes a dynamic resource created
  with one transport from use with another transport unless explicitly cleared.
  \]

When talking of proxy requests,
this document only talks of the Proxy-Scheme option.
Given that all URIs this is usable with can be expressed in decomposed CoAP URIs,
the need for using the Proxy-URI option should never arise.

## Goals

This document introduces provisions for the seamless use of different transport mechanisms for CoAP.
Combined, these provide:

* Enablement: Inform clients of the availability of other transports of servers.

* No Aliasing: Any URI aliasing must be opt-in by the server. Any defined mechanisms must allow applications to keep working on the canonical URIs given by the server.

* Optimization: Do not incur per-request overhead from switching protocls. This may depend on the server's willingness to create aliased URIs.

* Proxy usability: All information provided must be usable by aware proxies to reduce the need for duplicate cache entries.

* Proxy announcement: Allow third parties to announce that they provide alternative transports to a host.

For all these functions, security policies must be described that allow the client to use them as securely as the original transport.

This document will not concern itself with changes in transport availability over time,
neither in causing them ("Please take up your TCP interface, I'm going to send a firmware update")
nor in advertising them (other than by the server putting suitable Max-Age values on any of its statements).

# Indicating alternative transports

While CoAP can indicate the authority component of the requested URI in all requests (by means of Uri-Host),
indicating the scheme of a requested URI (by means of Proxy-Scheme) makes the request implicitly a proxy request.
However, this needs to be of only little practical concern:
Any device can serve as a proxy for itself (a "same-host proxy")
by accepting requests that carry the Proxy-Scheme option.
If it is to be a well-behaved as a proxy,
the device should then check whether it recognizes the name indicated in Uri-Host as one of its own
\[ TBD: Check whether 7252 makes this a stricter requirement \],
reject the request with 5.05 when it is not recognized,
and otherwise process it as it would process a request coming in on that protocol
(which, for many hosts, is the same as if the option were absent completely).

A server can indicate support for same-host proxying (or any kind of proxying, really)
by serving a Web Link with the "has-proxy" relation.

The semantics of a link from C to T with relations has-proxy ("C has-proxy T", `<T>;rel=has-proxy;anchor="C"`)
are that for any resource R hosted on C ("C hosts R"), T is can be used as a proxy to obtain R.

Note that HTTP and CoAP proxies are not located at a particular resource,
but at a host in general.
Thus, a proxy URI `T` in these protocols can not carry a path or query component.
This is true even for CoAP over WebSockets (which uses the concrete resource `/.well-known/coap`, but that can not expressed in "coap+ws" URI).
Future protocols for which CoAP proxying is defined may have expressible path components.

## Example

A constrained device at the address 2001:db8::1 that supports CoAP over TCP in addition to CoAP can self-describe like this:

~~~~~
Req: to [ff02::fd]:5683 on UDP
Code: GET
Uri-Path: /.well-known/core
Uri-Query: if=tag:example.com,sensor

Res: from [2001:db8::1]:5683
Content-Format: application/link-format
Payload:
</sensors/temp>;if="tag:example.com,sensor",
<coap+tcp://[2001:db8::1]>;rel=has-proxy;anchor="/"


Req: to [2001:db8::1]:5683 on TCP
Code: GET
Proxy-Scheme: coap
Uri-Path: /sensors/temp
Observe: 0

Res: 2.05 Content
Observe: 0
Payload:
39.1°C
~~~~~
{: #fig-has-proxy title='Follow-up request through a has-proxy relation'}

Note that generating this discovery file needs to be dynamic based on its available addresses;
only if queried using a link-local source address, it may also respond with a link-local address in the authority component of the proxy URI.

Unless the device makes resources discoverable at `coap+tcp://[2001:db8::1]/.well-known/core` or another discovery mechanism,
clients may not assume that `coap+tcp://[2001:db8::1]/sensors/temp` is a valid resource (let alone has any relation to the other resource on the same path).
The server advertising itself like this may reject any request on CoAP-over-TCP unless they contain a Proxy-Scheme option.

Clients that want to access the device using CoAP-over-TCP would send a request
by connecting to 2001:db8::1 TCP port 5683
and sending a GET with the options Proxy-Scheme: coap, no Uri-Host or -Port options (utilizing their default values),
and the Uri-Paths "sensors" and "temp".


# Elision of Proxy-Scheme and Uri-Host

A CoAP server may publish and accept multiple URIs for the same resource,
for example when it accepts requests on different IP addresses that do not carry a Uri-Host option,
or when it accepts requests both with and without the Uri-Host option carrying a registered name.
Likewise, the server may serve the same resources on different transports.
This makes for efficient requests (with no Proxy-Scheme or Uri-Host option),
but In general is discouraged {{aliases}}.

To make efficient requests possible without creating URI aliases that propagate,
the "has-unique-proxy" specialization of the has-proxy relation is defined.

If a proxy is unique, it means that it unconditionally forwards to the server indicated in the link context,
even if the Proxy-Scheme and Uri-Host options are elided.

\[ The following two paragraphs are both true but follow different approaches to explaining the observable and implementable behavior;
   it may later be decided to focus on one or the other in this document. \]

While this creates URI aliasing in the requests as they are sent over the network,
applications that discover a proxy this way should not "think" in terms of these URIs,
but retain the originally discovered URIs (which, because Cool URIs Don't Change{{cooluris}}, should be long-term usable).
They use the proxy for as long as they have fresh knowledge of the has-(unique-)proxy statement.

In a way, advertising `has-unique-proxy` can be viewed as a description of the link target in terms of SCHC {{?I-D.ietf-lpwan-coap-static-context-hc}}:
In requests to that target, the link source's scheme and host are implicitly present.

A client MAY use a unique-proxy like a proxy and still send the Proxy-Scheme and Uri-Host option;
such a client needs to recognize both relation types, as relations of the has-unique-proxy type
are a specialization of has-proxy and typically don't carry the latter (redundant) annotation.

Example:

~~~~~
Req: to [ff02::fd]:5683 on UDP
Code: GET
Uri-Path: /.well-known/core
Uri-Query: if=tag:example.com,sensor

Res: from [2001:db8::1]:5683
Content-Format: application/link-format
Payload:
</sensors/>;if="tag:example.com,collection",
<coap+tcp://[2001:db8::1]>;rel=has-proxy;anchor="/"


Req: to [2001:db8::1]:5683 on TCP
Code: GET
Uri-Path: /sensors/

Res: 2.05 Content
Content-Format: application/link-format
Payload:
</sensors/temperature>;if="tag:example.com,sensor"
~~~~~
{: #fig-has-unique-proxy title='Follow-up request through a has-unique-proxy relation. Compared to the last example, 5 bytes of scheme indication are saved during the follow-up request.'}

It is noteworthy that when the URI reference `/sensors/temperature` is resolved,
the base URI is `coap://device0815.example.com` and not its coap+ws counterpart --
as the request is implicitly forwarded there, which both the client and the server are aware of.
However, this detail is of little practical importance:
A simplistic client that uses `coap+ws://device0815.proxy.rd.example.com` as a base URI will still arrive at an identical follow-up request with no ill effect,
as long as it only uses the wrongly assembled URI for dereferencing resources,
and does not (for example) pass it on to other devices.

# Third party proxy services

A server that is aware of a suitable cross proxy may use the has-proxy relation to advertise that proxy.
If the protocol used towards the proxy provides name indication (as CoAP over TLS or WebSockets does),
or by using a large number of addresses or ports,
it can even advertise a (more efficient) has-unique-proxy relation.
This is particularly interesting when the advertisements are made available across transports,
for example in a Resource Directory.

How the server can discover such a proxy
(or, for the rare cases where this is possible, discover that it is suitable as a unique proxy)
is out of scope for this document,
but generally involves the same kind of links.


~~~~~
Req: GET http://rd.example.com/rd-lookup?if=tag:example.com,sensor

Res:
Content-Format: application/link-format
Payload:
<coap://device0815.example.com/sensors/>;if="tag:example.com,collection",
<coap+wss://device0815.proxy.rd.example.com>;rel=has-unique-proxy;anchor="coap://device0815.example.com/"


Req: to device0815.proxy.rd.example.com on WebSocket
Host (indicated during upgrade): device0815.proxy.rd.example.com
Code: GET
Uri-Path: /sensors/

Res: 2.05 Content
Content-Format: application/link-format
Payload:
</sensors/temperature>;if="tag:example.com,sensor"
~~~~~
{: #fig-external-proxy title='HTTP based discovery and CoAP-over-WS request to a CoAP resource through a has-unique-proxy relation'}

A third party proxy may advertise its availability to act as a proxy for arbitrary CoAP requests.
\[ TBD: Specify a mechanism for this; `<coap+ws://myself>;rel=has-proxy;anchor="coap://*"` for all supported protocols appears to be an obvious but wrong solution. \]

# Client picked proxies

When a resource is accessed through an "actual" proxy (i.e., a host between the client and the server, which itself may have a same-host proxy in addition to that),
the proxy's choice of the upstream server is originally (i.e., without the mechanisms of this document)
either configured (as in a "chain" of proxies)
or determined by the request URI (where a proxy picks CoAP over TCP for a request aimed at a coap+tcp URI).

A proxy that has learned,
by active solicitation of the information or by consulting links in its cache,
that the requested URI is available through a same-host proxy,
or that has learned of advertised URI aliasings,
may can that information.

For example, if a host at coap://h1.example.com has advertised `</res>,<coap+tcp://h1.example.com>;rel=has-proxy;anchor="/"`,
then a proxy that has an active CoAP-over-TCP connection to h1.example.com
can forward an incoming request for coap://h1.example.com/res through that CoAP-over-TCP connection
with a suitable Proxy-Scheme on that connection.

If the host had marked the proxy point as `<coap+tcp://h1.example.com>;rel=has-unique-proxy`,
then the proxy could elide the Proxy-Scheme and Uri-Host options,
and would (from the original CoAP caching rules) also be allowed
to use any fresh cache representation of coap+tcp://h1.example.com/res
to satisfy requests for coap://h1.example.com/res.

# Security considerations

\[ TBD; in key words: \]

* Always (ie. both with (D)TLS and OSCORE): Risk of attackers redirecting traffic for metadata analysis.

  Thus, only use transports you've obtained from either the server itself or someone you trust to make routing decisions for you.

  (If you have no other route, you may not be too picky about where you get your routes from).

* Without E2E (ie. (D)TLS):
  Advertised proxy must either present credentials that are good enough for you to use as a general forward proxy,
  or present credentials good enough to be the actual origin server (like Alt-Svc does).

* TBD: Protecting the indicated proxy (rather than the client)

# IANA considerations

## Link Relation Types

IANA is asked to add two entries into the Link Relation Type Registry last updated in {{!RFC8288}}:

| Relation Name     | Description                                                                 | Reference   |
| has-proxy         | The link target can be used as a proxy to reach the link context.           | RFCthis     |
| has-unique-proxy  | Like has-proxy, and using this proxy implies scheme and host of the target. | RFCthis     |
{: #tab-iana title='New Link Relation types' }


--- back

<!--
# Change log


-->

# Acknowledgements

This document heavily builds on concepts
explored by Bill Silverajan and Mert Ocak in {{?I-D.silverajan-core-coap-protocol-negotiation}},
and together with Ines Robles and Klaus Hartke inside T2TRG.
