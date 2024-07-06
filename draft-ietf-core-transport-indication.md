---
v: 3

title: CoAP Transport Indication
docname: draft-ietf-core-transport-indication-latest
stand_alone: true

venue:
  mail: core@ietf.org
  github: core-wg/transport-indication
  latest: "https://core-wg.github.io/transport-indication/"

ipr: trust200902
cat: std
stream: IETF
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
  country: Austria
  email: christian@amsuess.com
- fullname: Martine Sophie Lenders
  org: TUD Dresden University of Technology
  abbrev: TU Dresden
  street: Helmholtzstr. 10
  city: Dresden
  code: D-01069
  country: Germany
  email: martine.lenders@tu-dresden.de
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
  lwm2m:
    title: White Paper – Lightweight M2M 1.1
    author:
      org: OMA SpecWorks
    target: https://omaspecworks.org/white-paper-lightweight-m2m-1-1/
  I-D.ietf-lake-edhoc:
  w3address:
    title: "W3 address syntax: BNF"
    author:
      name: Tim BL
    target: http://info.cern.ch/hypertext/WWW/Addressing/BNF.html#43
    date: 1992-06-29
  noproxy:
    title: "We need to talk: Can we standardize NO_PROXY?"
    author:
      name: Stan Hu
    target: https://about.gitlab.com/blog/2021/01/27/we-need-to-talk-no-proxy/
    date: 2021-01-27
  evossl:
    title: The Evolution of SSL and TLS
    author:
      name: Elizabeth Baier
    target: https://www.digicert.com/blog/evolution-of-ssl
    date: 2015-02-02

--- abstract

The Constrained Application Protocol (CoAP, {{!RFC7252}}) is available over different transports
(UDP, DTLS, TCP, TLS, WebSockets),
but lacks a way to unify these addresses.
This document provides terminology and provisions based on Web Linking {{?RFC8288}}
to express alternative transports available to a device,
and to optimize exchanges using these.

--- middle

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) provides transports mechanisms
(UDP and DTLS since {{?RFC7252}}, TCP, TLS and WebSockets since {{?RFC8323}}),
with some additional being used in LwM2M {{lwm2m}}
and even more being explored ({{?I-D.bormann-t2trg-slipmux}}, {{?I-D.amsuess-core-coap-over-gatt}}).
These are mutually incompatible on the wire,
but CoAP implementations commonly support several of them,
and proxies can translate between them.

CoAP currently lacks a way to indicate which transports are available for a given resource,
and to indicate that a device is prepared to serve as a proxy;
this document solves both by introducing the "has-proxy" terminology to Web Linking {{!RFC8288}} that expresses the former through the latter.
The additional "has-unique-proxy" term is introduced
to negate any per-request overhead that would otherwise be introduced in the course of this.

CoAP also lacks a unified scheme to label a resource in a transport-independent way.
This document does *not* attempt to introduce any new scheme here,
or raise a scheme to be the canonical one.
Instead, each host or application can pick a canonical address for its resources,
and advertise other transports in addition.

## Terminology

Readers are expected to be familiar with the terms and concepts
described in CoAP {{RFC7252}}
and link format {{!RFC6690}}
(or, equivalently, web links as described in {{RFC8288}}).

Same-host proxy:

: A CoAP server that accepts forward proxy requests (i.e., requests carrying the Proxy-Scheme option)
  exclusively for URIs that it is also the authoritative server for is defined as a "same-host proxy".

: The distinction between a same-host and any other proxy is only relevant on a practical, server-implementation and illustrative level;
  this specification does not use the distinction in normative requirements,
  and clients need not make the distinction at all.

When talking of proxy requests,
this document only talks of the Proxy-Scheme option.
Given that all URIs this is usable with can be expressed in decomposed CoAP URIs,
the need for using the Proxy-URI option should never arise.
The Proxy-URI option is still equivalent to the decomposed options,
and can be used if the server supports it.

### Using URIs to identify transport endpoints {#identifying}

The URI `coap://[2001:db8::1]` identifies a particular resource, possibly a "welcome" text.
It is, colloquially, also used to identify the combination
of a CoAP transport and the transport specific details.

For precision, this document uses the term
"the transport address indicated by (a URI)" to refer to the transport and its details (in the example, CoAP over UDP with an IPv6 address and the default port),
but otherwise no big deal is made of it.

The transport indicated by a URI is not only influenced by the URI scheme,
but also by the authority component.
The transports and resolution mechanisms currently specified
make little use of this possibility,
mainly because the most prominent resolution mechanism (SVCB records) has not been avaialble when {{?RFC8323}} was published
(see also {{althist}}),
end because it can not be expressed in IP literals (see {{newlit}}).

When the resolution mechanism used for a registered name authority component yields multiple addresses,
all of those are possible ways to interact with the resource.
The resolution mechanism or other underlying transport can give guidance on how to find the best usable one.
With the currently specified transports and resolution mechanisms,
the most prominent example of making use of that information
is applying {{?RFC8305}}'s Happy Eyeballs mechanism to establish a TCP connection
when a name resolves to both IPv4 and IPv6 addresses,

#### Paths in URIs that indicate transport addresses

For the CoAP schemes (coap, coaps, coap+tcp, coaps+tcp, coap+ws, coaps+ws),
URIs indicating a transport are always given with an empty path
(which under their URI normalization rules is equivalent to a path containing a single slash).
For the coap and coap+tcp schemes, URIs with different host names
can indicate the same transport as long as the names resolve to the same addresses.
For the others, the given host name informs the name set in TLS's Server Name Indication (SNI)
and/or the host sent in the "Host" header of the underlying HTTP request.

If an update to this document extends the list,
for new schemes it might be allowed to have paths, queries or fragment identifiers present in the URI indicating the transport address.
No guidance can be given here for these,
as no realistic example is known.
(Note that while the coap+ws scheme does use the well-known path `/.well-known/coap` internally,
that is used purely on the HTTP side, and not part of the CoAP URI, not even for indicating the transport address).
[^svcbpathparam]

[^svcbpathparam]: It is conceivable that a path such as the `/.well-known/coap` of CoAP-over-WebSockets would be indicated in an SVCB discovery's parameters similar to dohpath. This does not immediately help with the question of whether a URI indicating a transport can have a path, though. --CA

#### Existing use

A similar concept is used in {{?I-D.ietf-core-observe-multicast-notifications}} (expressed as pieces of its `tp_info` parameter),
but not expressed with URIs yet.
As that document migrates towards using CRIs ({{I-D.ietf-core-href}}),
it is expected that its transport addresses coincide with the URIs (CRIs, equivalently) indicating a transport.

URIs indicating a transport are especially useful when talking about proxies;
this use is aligned with the way they are expressed in the conventional environment variables `http_proxy` etc.
{{noproxy}}.
Furthermore, URIs processing is widespread in CoAP systems,
and when that changes (e.g. through the introduction of {{?I-D.ietf-core-href}}),
URIs indicating a transport will still be efficient to encode.
And last but not least, it lines up well with the colloquial identity mentioned above.
(An alternative would be using a dedicated naming scheme, say, `transport:coap:device.example.com:port`,
but that would needlessly introduce implementation complexity).

Note that this mechanism can only used with proxies that use CoAP's native address indication mechanisms.
Proxies that perform URI mapping
(as described in Section 5 of {{?RFC8075}}, especially using URI templates)
are not supported in this document.

\[ TBD: Do we want to extend this to HTTP proxies? Probably just not, and if so, only to those that can just take coap://... for a URI. \]

## Goals

This document introduces provisions for the seamless use of different transport mechanisms for CoAP.
Combined, these provide:

1. Enablement: Inform clients of the availability of other transports of servers.

2. No Aliasing: Any URI aliasing must be opt-in by the server. Any defined mechanisms must allow applications to keep working on the canonical URIs given by the server.

3. Optimization: Do not incur per-request overhead from switching transports. This may depend on the server's willingness to create aliased URIs.

4. Proxy usability: All information provided must be usable by aware proxies to reduce the need for duplicate cache entries.

5. Proxy announcement: Allow third parties to announce that they provide alternative transports to a host.

For all these functions, security policies must be described that allow the client to use them as securely as the original transport.

This document will not concern itself with changes in transport availability over time,
neither in causing them ("Please take up your TCP interface, I'm going to send a firmware update")
nor in advertising their availability in advance.
Hosts whose transport's availability changes over time can utilize
any suitable mechanism to keep client updated,
such as placing a suitable Max-Age value on their resources
or having them observable.

# Indicating alternative transports

While CoAP can set the authority component of the requested URI in all requests (by means of Uri-Host and Uri-Port),
setting the scheme of a requested URI (by means of Proxy-Scheme) makes the request implicitly a proxy request.
However, this needs to be of only little practical concern:
Any device can serve as a proxy for itself (a "same-host proxy")
by accepting requests that carry the Proxy-Scheme option.
{{Section 5.7.2 of RFC7252}} already mandates that a proxy recognize its own addresses.
A minimal same-host proxy supports only those and respond with 5.05 (Proxying Not Supported).
In many cases (precisely: on hosts that alias their resources across transports),
this is equivalent to ignoring the Proxy-Scheme option in that request.

A server can advertise a recommended proxy
by serving a Web Link with the "has-proxy" relation to a URI indicating its transport address.
In particular (and that is a typical case),
it can indicate its own transport address on an alternative transport when implementing same-host proxy functionality.

The semantics of a link from S to P with relations has-proxy ("S has-proxy P", `<P>;rel=has-proxy;anchor="S"`)
are that for any resource that has the same origin as S, the transport address indicated by P can be used to obtain that resource.

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
{: #fig-has-proxy title='Discovery and follow-up request through a has-proxy relation'}

Note that generating this discovery file needs to be dynamic based on its available addresses;
only if queried using a link-local source address, the server may also respond with a link-local address in the authority component of the proxy URI.

Unless the device makes resources discoverable at `coap+tcp://[2001:db8::1]/.well-known/core` or another discovery mechanism,
clients may not assume that `coap+tcp://[2001:db8::1]/sensors/temp` is a valid resource (let alone is equivalent to the other resource on the same path).
The server advertising itself like this may reject any request on CoAP-over-TCP unless it contains a Proxy-Scheme option.

Clients that want to access the device using CoAP-over-TCP would send a request
by connecting to 2001:db8::1 TCP port 5683
and sending a GET with the options Proxy-Scheme: coap, no Uri-Host or -Port options (utilizing their default values),
and the Uri-Paths "sensors" and "temp".

## Security context propagation {#secctx-propagation}

Any security requirements posed by a server or client application on a CoAP request
MUST be applied independently of the transport that is used to perform the request.
If a transport can not be used to satisfy the requirements,
it is ineligible for use with the request (from a client's point of view),
and unauthorized (from a server's point of view).

If the requirements contain transport layer security,
the proxy needs to present the credentials required of the server to the client,
and those of the client to the server;
this is only practical when the proxy is a same-host proxy.

Some applications have requirements
exceeding the requirements of a secure connection,
e.g., (explicitly or implicitly) requiring that
name resolution happen through a secure process
and packets are only routed into networks where it trusts that they will not be intercepted on the path to the server.
Such applications need to extend their requirements to the source of the `has-proxy` statement;
a sufficient (but maybe needlessly strict) requirement is to only follow `has-proxy` statements
that are part of the same resource that advertises the link currently being followed.
Section {{proxy-foreign-advertisement}} adds further considerations.

## Choice of transports

It is up to the client whether to use an advertised proxy transport,
or (if multiple are provided) which to pick.

Links to proxies may be annotated with additional metadata that may help guide such a choice;
defining such metadata is out of scope for this document.

Clients MAY switch between advertised transports as long as the document describing them is fresh;
they may even do so per request.
(For example, they may perform individual requests using CoAP-over-UDP,
but choose CoAP-over-TCP for requests with large expected responses).
When the describing document approaches expiry,
the client can use the representation's ETag to efficiently renew its justification for using the alternative transport.

## Selection of a canonical origin

While a server is at liberty to provide the same resource independently on different transports (i.e. to create aliases),
it may make sense for it to pick a single scheme and authority under which it announces its resources.
Using only one address helps proxies keep their caches efficient,
and makes it easier for clients to avoid exploring the same server twice from different angles.

When there is a predominant scheme and authority through which an existing service is discovered,
it makes sense to use these for the canonical addresses.

Otherwise,
it is suggested to use the `coap` or `coaps` scheme
(given that these are the most basic and widespread ones),
and the most stable usable name the host has.

### Unreachable canonical origin addresses

For devices that are not generally reachable at a stable address,
it may make sense to use a scheme and authority as the canonical address that can not actually be dereferenced.

The registered names available for that purpose depend on the locally defined host or service name registry.
When the Domain Name System (DNS) is used,
such names would not be associated with any A or AAAA records
(but may still use, for example, TLSA records).

Such URIs are *only* usable to clients that discover a suitable proxy along with the URI,
and which can place sufficient trust in that proxy.


## Advertisement through a Resource Directory

In the Resource Directory specification {{?rfc9176}},
protocol negotiation was anticipated to use multiple base values.
This approach was abandoned since then,
as it would incur heavy URI aliasing.

Instead, devices can submit their has-proxy links to the Resource Directory like all their other metadata.

A client performing resource lookup can ask the RD to provide available (same-host-)proxies in a follow-up request
by asking for `?anchor=<the-discovered-host>&rel=has-proxy`.
<!-- We don't say that the RD can not do that, right? -->
The RD may also volunteer that information during resource lookups even though the has-proxy link itself does not match the search criteria.

\[

It may be useful to define RD parameters for use with lookup here, which'd guide which available proxies to include.
For example, asking `?if=tag:example.com,sensor&proxy-links=tcp` could give as a result:

`<coap://[2001:db8::1]/s>;rt=tag:example.com,sensor,<coap+tcp://[2001:db8::1]/>;rel=has-proxy;anchor="coap://[2001:db8::1]/"`

This is similar to the extension suggested in {{Section 5 of ?I-D.amsuess-core-resource-directory-extensions}}.

\]

# Elision of Proxy-Scheme and Uri-Host

A CoAP server may publish and accept multiple URIs for the same resource,
for example when it accepts requests on different IP addresses that do not carry a Uri-Host option,
or when it accepts requests both with and without the Uri-Host option carrying a registered name.
Likewise, the server may serve the same resources on different transports.
This makes for efficient requests (with no Proxy-Scheme or Uri-Host option),
but in general is discouraged {{aliases}}.

To make efficient requests possible without creating URI aliases that propagate,
the "has-unique-proxy" specialization of the has-proxy relation is defined.

If a proxy is unique,
it means that requests arriving at the proxy are treated the same
no matter whether the scheme, authority and port of the link context are set in the Proxy-Scheme, Uri-Host and Uri-Port options, respectively,
or whether all of them are absent.

\[ The following two paragraphs are both true but follow different approaches to explaining the observable and implementable behavior;
   it may later be decided to focus on one or the other in this document. \]

While this creates URI aliasing in the requests as they are sent over the network,
applications that discover a proxy this way should not "think" in terms of these URIs,
but retain the originally discovered URIs (which, because Cool URIs Don't Change{{cooluris}}, should be long-term usable).
They use the proxy for as long as they have fresh knowledge of the has-(unique-)proxy statement.

In a way, advertising `has-unique-proxy` can be viewed as a description of the link target in terms of SCHC {{?I-D.ietf-lpwan-coap-static-context-hc}}:
In requests to that target, the link source's scheme and host are implicitly present.

While applications retain knowledge of the originally requested URI
(even if it is not expressed in full on the wire),
the original URI is not accessible to caches both within the host and on the network
(for the latter, see {{actualproxies}}).
Thus, cached responses to the canonical and any aliased URI are mutually interchangeable
as long as both the response and the proxy statement are fresh.

A client MAY use a unique-proxy like a proxy and still send the Proxy-Scheme and Uri-Host option;
such a client needs to recognize both relation types, as relations of the has-unique-proxy type
are a specialization of has-proxy and typically don't carry the latter (redundant) annotation.
\[ To be evaluated -- on one hand, supporting it this way means that the server needs to identify all of its addresses and reject others.
   Then again, is a server that (like many now do) fully ignore any set Uri-Host correct at all? \]

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
<coaps+ws://[2001:db8::1]>;rel=has-unique-proxy;anchor="/"


Req: to [2001:db8::1] via WebSockets over HTTPS
Code: GET
Uri-Path: /sensors/

Res: 2.05 Content
Content-Format: application/link-format
Payload:
</sensors/temperature>;if="tag:example.com,sensor"
~~~~~
{: #fig-has-unique-proxy title='Follow-up request through a has-unique-proxy relation. Compared to the last example, 5 bytes of scheme indication are saved during the follow-up request.'}

It is noteworthy that when the URI reference `/sensors/temperature` is resolved,
the base URI is `coap://device0815.example.com` and not its coaps+ws counterpart --
as the request is still for that URI, which both the client and the server are aware of.
However, this detail is of little practical importance:
A simplistic client that uses `coaps+ws://device0815.proxy.rd.example.com` as a base URI will still arrive at an identical follow-up request with no ill effect,
as long as it only uses the wrongly assembled URI for dereferencing resources,
the security context is the same,
the state is kept no longer than the has-unique-proxy statement is fresh,
and it does not (for example) pass the URI on to other devices.

## Impact on caches

\[ This section is written with the "there is implied URI aliasing" mindset;
it should be possible to write it with the "compression" mindset as well
(but there is no point in having both around in the document at this time).

It is also slightly duplicating, but also more detailed than,
the brief note on the topic in {{actualproxies}}
\]

When a node that performs caching learns of a `has-unique-proxy` statement,
it can utilize the information about the implied URI aliasing:
As long as the `has-unique-proxy` statement is fresh and trusted,
requests for either of the origins can be served from the cache of the other origin.

## Using unique proxies securely {#unique-security}

The elision of the host name afforded by the `unique-proxy` relation
is only possible if the required security mechanisms verify the scheme and host of the server.

This is given for OSCORE based mechanisms,
where "unprotected message fields (including Uri-Host \[...]) MUST not lead to an OSCORE message becoming verified".

With TLS based security mechanisms,
name and scheme can not be completely elided in general.
While the use of the SNI HostName field sets the default Uri-Host already,
the scheme still needs to be sent in a Proxy-Scheme option
to satisfy the requirement of {{secctx-propagation}}.

\[
It may be possible to relax this requirement
if the host publishes a *trustworthy* statement about serving the same content on all schemes;
however, no urgent need for this optimization is currently known that warrants the extra scrutiny.
\]

## Self-description as a unique proxy

The level of assurance a client needs from a server
to elide the Uri-Host option in a request that was created from a URI with no IP address literal
has been a controversial topic.
\[ Should we dig up old conversations, link to https://github.com/core-wg/wiki/wiki/CoAP-FAQ#q4,
or just let the weight of a WG consensus-document-to-be do its work? \]

The `has-unique-proxy` relation provides an easy way for a server
to indicate that this is in fact allowed:
A server can publish a statement such as `</>;rel=has-unique-proxy` in its `/.well-known/core` file.
A client that receives and understands it can thus elide the Uri-Host option in requests to the server
as per the definition of the relation.

# Third party proxy services {#thirdparty}

A server that is aware of a suitable cross proxy may use the has-proxy relation to advertise that proxy.
If the transport used towards the proxy provides name indication (as CoAP over TLS or WebSockets does),
or by using a large number of addresses or ports,
it can even advertise a (more efficient) has-unique-proxy relation.
This is particularly interesting when the advertisements are made available across transports,
for example in a Resource Directory.

How the server can discover and trust such a proxy
is out of scope for this document,
but generally involves the same kind of links.
In particular, a server may obtain a link to a third party proxy from an administrator as part of its configuration.

The proxy may advertise itself without the origin server's involvement;
in that case, the client needs to take additional care (see {{proxy-foreign-advertisement}}).

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

## Generic proxy advertisements {#generic-advertisements}

A third party proxy may advertise its availability to act as a proxy for arbitrary CoAP requests.
This use is not directly related to the transport indication in other parts of this document,
but sufficiently similar to warrant being described in the same document.

The resource type "CPA-core.proxy" can be used to describe such a proxy.

~~~~~
Req: GET coap://[fe80::1]/.well-known/core?rt=CPA-core.proxy

Res:
Content-Format: application/link-format
Payload:
<>;rt=CPA-core.proxy

Req: to [fe80::1] via CoAP
Code: GET
Proxy-Scheme: http
Uri-Host: example.com
Uri-Path: /motd
Accept: text/plain

Res: 2.05 Content
Content-Format: text/plain
Payload:
On Monday, October 25th 2021, there is no special message of the day.
~~~~~
{: #fig-6lbr-proxy title='A CoAP client discovers that its border router can also serve as a proxy, and uses that to access a resource on an HTTP server.'}

The considerations of {{proxy-foreign-advertisement}} apply here.

A generic advertised proxy is always a forward proxy,
and can not be advertised as a "unique" proxy as it would lack information about where to forward.

A proxy may be limited in the URIs it can service,
for technical reasons (e.g. when none of the URI's transports are supported by the server)
or for policy reasons (only accessing servers inside an organizational structure).
Future documents (or versions of this document) may add target attributes
that allow specifying the capabilities of a proxy.
\[
An earlier version of this document contained a proxy-schemes attribute.
This was discontinued because it could already not express whether a proxy could access IPv4 or IPv6 peers,
and because the use of schemes is becoming less useful given the new recommendation of incorporating details from registered name resolution into the transport selection.
\]

The use of a generic proxy can be limited to a set of devices that have permission to use it.
Clients can be allowed by their network address if they can be verified,
or by using explicit client authentication using the methods of {{?I-D.tiloca-core-oscore-capable-proxies}}.

# Client picked proxies {#actualproxies}

This section is purely informative,
and serves to illustrate that the mechanisms introduced in this document do not hinder the continued use of existing proxies.

When a resource is accessed through an "actual" proxy (i.e., a host between the client and the server, which itself may have a same-host proxy in addition to that),
the proxy's choice of the upstream server is originally (i.e., without the mechanisms of this document)
either configured (as in a "chain" of proxies)
or determined by the request URI (where a proxy picks CoAP over TCP and resolves the given name for a request aimed at a coap+tcp URI).

A proxy that has learned,
by active solicitation of the information or by consulting links in its cache,
that the requested URI is available through a (possibly same-host) proxy,
may use that information
in choosing the upstream transport,
to correct the URI associated with a cached response,
and to use responses obtained through one transport to satisfy requests on another.

For example, if a host at coap://h1.example.com has advertised `</res>,<coap+tcp://h1.example.com>;rel=has-proxy;anchor="/"`,
then a proxy that has an active CoAP-over-TCP connection to h1.example.com
can forward an incoming request for coap://h1.example.com/res through that CoAP-over-TCP connection
with a suitable Proxy-Scheme and Uri-Host on that connection.

If the host had marked the proxy point as `<coap+tcp://h1.example.com>;rel=has-unique-proxy` instead,
then the proxy could elide the Proxy-Scheme and Uri-Host options,
and would (from the original CoAP caching rules) also be allowed
to use any fresh cache representation of coap+tcp://h1.example.com/res
to satisfy requests for coap://h1.example.com/res.

A client that uses a forward proxy
and learns of a different proxy advertised to access a particular resource
will not change its behavior if its original proxy is part of its configuration.
If the forward proxy was only used out of necessity
(e.g., to access a resource whose indicated transport not supported by the client)
it can be practical for the client to use the advertised proxy instead.

# Transport indication from other sources: Service Binding Parameters

Discovery mechanisms that exist in DNS {{?RFC9460}}, DHCP or Router Advertisements {{?RFC9463}}
can provide details already that would otherwise only be discovered later through proxy links.
For when those details are provided in the shape of Service Binding Parameters,
this section describes their interpretation in the context of CoAP transport indication.

## Discovering transport indication details from name resolution {#svcb-discovery}

When a host finds a CoAP transport from a URI
and the URI's authority component does not contain a precise address literal,
the resolution mechanisms which it may try generally depend on the CoAP transports and their variation which it supports.
For example, if it supports CoAP-over-UDP and IPv6,
it requests AAAA records through DNS and look them up in a host file.

This document extends this and registers the `_coap` and `_coaps` attrleaf labels
in {{iana-underscored}},
using the pattern described as described in {{Section 10.4.5 of RFC9460}}
and thus enabling the use of SVCB records.
This path is chosen over the creation of a new SVCB RR pair "COAP" / "COAPS"
because it is considered unlikely that DNS implementations would update their code bases to apply SVCB behavior;
this assumption will be revisited before registration.

Some SVCB parameters have defaults; for those new services, these are:
* port: 5683 for `_coap`, 5684 for `_coaps`
* ALPN: empty for `_coap`, "co" for `_coaps`
* coaptransport: "udp" for `_coap`, empty for `_coaps`

As SVCB records were not specified for the existing CoAP transports originally,
generic CoAP clients are not required to use the SVCB lookup mechanism,
but MAY attempt it opportunistically in order to obtain a usable transport
(or to obtain it faster).
Applications built on CoAP MAY require clients to perform this kind of discovery.
Adding such a requirement is particularly useful if the application frequently advertises URIs with a scheme that defaults to a transport which its clients may not support,
or when the application makes use of functionality afforded by {{?RFC9460}} such as apex domain redirection.

The effects on a client of seeing SVCB parameters are similar
to those of seeing a "has-proxy" link from the origin to the URI built using {#svcblit}.
They differ in that SVCB parameters describe the server itself:
Credentials expressed apply end to end
(as opposed to credentials that describe the proxy in a "has-proxy" link),
and the client could conclude that the implied proxy is a same-host proxy
(if that had any impact on the client, which it does not).

## Service Parameters

Several parameters are relevant in the context of CoAP,
independently of whether they are used with SVCB records or Service Binding Parameters transported outside of SVCB records,
and independently of whether they apply to the `_coap` / `_coaps` service or another service that can be used on top of CoAP (such as `_dns`):

* `port`: The CoAP service using the transport described in this parameter is reachable on this port
  (described in {{RFC9460}}).

* `alpn`: The ALPN "coap" has been defined for CoAP-over-TLS {{?RFC8323}}, and "co" for CoAP-over-DTLS in {{?I-D.lenders-core-coap-dtls-svcb}}.

  If an ALPN service parameter is found, this indicates that the ALPN(s) and thus the CoAP transport that can be used on this address / port.
  For example, "co" indicates that DTLS (and thus UDP) is used.

* `coaptransport`: This is a new parameter defined in this document, and similar to the ALPN parameter.

  If a `coaptransport` parameter is present, the indicated transport(s) can be used on this address / port.

  The names registered for existing transports are identical to the URI schemes that indicate their use in the absence of Service Binding Parameters.

  \[ It is left for review by SVCB experts whether these are a separate parameter space or we should just take ALPNs for them, like eg. h2c does. \]

* `cred`: This is a new parameter defined in this document, and describes credentials usable to authenticate the server.

  The `cred` parameter's value is a CBOR sequence of COSE Header maps as defined in {{!RFC9052}}.
  If the parameter is present, it indicates that a COSE based security layer such as EDHOC can be used on the transport,
  and that the server can be authenticated by any credential expressed in the sequence.
  This is similar to the TLSA records specified in {{?RFC6698}}.

  A COSE Header map can express many properties, not all of which are sufficient to authenticate a peer on any given security mechanism.
  Without excluding applications that may process other entries,
  a practical criterion for whether a header map is suitable for EDHOC is that the header map could also be used in EDHOC as `ID_CRED_R`
  if the credential is sent by value.

  For example, a header map with a `kccs` entry can be used to indicate a public key including a Key ID (`kid`),
  and that public key does not need to be sent during the EDHOC exchange.
  Alternatively, a header map with an `x5t` identifies the end entity certificate the server presents by a thumbprint (hash).

  It is up to the application to define requirements for the provenance of the `cred` parameter,
  whether it needs to be provided through secure mechanism,
  or whether the server is strictly required to present that credential.

  This is unlike TLSA, which needs to be transported through DNSSEC,
  because a `cred` parameter may be sent using other means than DNS
  (for example in DHCPv6 responses or Router Advertisements).

  \[
  The current phrasing of this is rather general toward COSE,
  but we only have two security mechanisms based on it for CoAP,
  and only one of those is asymmetric enough to be practical.
  Should we try to stay generic, or just call this "edhoc" and be done with it?
  (Applications where a server is identified by OSCORE kid are very limited in that the OSCORE context needs to be pre-established, and the ).
  \]

* `edhoc-info`: This is a new parameter defined in this document, describing how EDHOC can be used on the server.

  The value of the parameter is a CBOR map following the `EDHOC_Information` structure defined in {{?I-D.ietf-ace-edhoc-oscore-profile}}
  and extended in {{?I-D.tiloca-lake-app-profiles}}.

  It is optional to provide and optional to process, but can help speed up the establishment of a security context.

### Examples of using name resolution discovery and parameters

#### Generic client discovering transport options

A generic client is directed to obtain `coap://dev1.example.com/log`
requests the name to be resolved using the system's resolution mechanisms,
resulting in a DNS query for these records:

```
_coap.dev1.example.com IN SVCB
dev1.example.com       IN AAAA
```

The following records are returned:

```
_coap.dev1.example.com IN SVCB 1 . coaptransport=tcp,udp
_coap.dev1.example.com IN SVCB 1 . alpn=co,coap port=5684
_coap.dev1.example.com IN SVCB 1 . coaptransport=udp port=61616
dev1.example.com       IN AAAA 2001:db8:1::1
```

Exceeding the single option it had with just the IP address,
it may now also choose to establish a TCP connection on the default port,
to use port 61616 for UDP (which results in more compact frames on a 6LoWPAN link),
or to use either of the TLS based options.

#### Application mandating SVCB

An application's policy is to mandate client support for SVCB records,
and to require that a security mechanism must be used where credentials are backed either by DNSSEC or by the Web PKI.

A server operator is running in a legacy network that only provides an IPv4 address behind NAT with a dynamic public address,
but has PCP {{?RFC7291}} available.
After running PCP to open a UDP port,
it learns that 1.2.3.4:5678 will be available for some time.

It therefore updates its DNS record like this:

```
_coap.host.example.net 600 IN SVCB 1 publicudp.host.example.net       \
                       port=5678                                      \
                       cred={14:{... /KCCS containing its public key/}}
```

When a client starts using `coap://host.example.net/interactive`,
it looks up that record and verifies it using DNSSEC.
It then proceeds to send EDHOC requests over CoAP to 1.2.3.4 port 5678,
setting the Uri-Host option to "host.example.net".

The client could also have initiated an EDHOC session if no cred parameter had been present,
but then,
it would have required that the server present some credential that could be verified through the Web PKI,
for example an x5chain containing a Let's Encrypt certificate.


## Expressing Service Parameters as literals {#svcblit}

TBD

### Examples

## Producing a URI from a discovered service {#svcb2uri}

If a service's discovery process does not produce a URI but an address, host name and/or Service Binding Parameters,
those can be converted to a CoAP URI,
for which transport hints are already encoded in the parameters the URI is constructed from.
An example of this is DNS server discovery {{?I-D.lenders-core-dnr}}.

TBD

### Examples

# Guidance to upcoming transports {#upcomingtransports}

When new transports are defined for CoAP,
it is recommended to use the "coap" scheme
(or "coaps" for TLS based transports).

{:ml: source="Martine"}

If the transport's identifiers are IP based and have identifiers typically resolved through DNS,
authors of new transports are encouraged to specify Service Binding records ({{?RFC9460}}) for CoAP,
e.g., using an `alpn` or `coaptransport` parameter.
and if IP literals are relevant to the transport, to follow up on {{newlit}}.

If the transport's native identifiers are compatible with the structure of the authority component of a URI,
those identifiers can be used as an authority as-is.
To help the host decide the resolution mechanism,
it may be helpful to register a subdomain of .arpa as described in {{?rfc3172}}.
The guidence for users is to never attempt to resolve such a name,
and for the zone's implementation is to return NXDOMAIN unconditionally.

For the purpose of specifying a transport protocol via Service Binding records, and to encourage
future authors more, this document specifies the `coaptransport` Service Parameter Key (SvcParamKey)
with the initial legal values "udp" and "tcp" which indicate either CoAP over UDP and CoAP over
TCP as the transport. The present of transport security is indicated by the `alpn` SvcParamKey. If
it the `alpn` SvcParamKey is not provided, but `coaptransport` is, the transport is unencrypted.[^1]{:ml:}

[^1]: Wondering if "udp" or "tcp" should be strings or numeric representations as value. The later
      would need an extra table or is there something we could reuse, e.g. from
      {{I-D.ietf-core-href}}?


If the transport's native identifiers are incompatible with that structure
(e.g. because they contain colons),
the document may define some transformation.

If a transport's native identifiers are only local,
the zone .alt {{?rfc9476}} may be used instead.

For example,
CoAP over GATT {{?I-D.amsuess-core-coap-over-gatt}}
removes the colons from Bluetooth Low Energy MAC addresses like 00:11:22:33:44:55
and combines them into authority compoennts such as `001122334455.ble.arpa`.
Slipmux {{?I-D.bormann-t2trg-slipmux}}
might use the locally significant device name `/dev/ttyUSB0`
as `coap://ttyUSB0.dev.alt/`.

URIs created from such names may not indicate the protocol uniquely:
Additional transports specified later may also provide CoAP services for the same name.
In the sense of {{identifying}},
both transport would be identified by that URI.
That is not an issue as long as the protocols underneath the CoAP transport
provide a means of advertising the precise protocol used.
For example,
a hypothetical CoAP transport for BLE that is not GATT based
can be selected for the same scheme and authority based on data in the BLE advertisement.

# Security considerations

## Security context propagation

Clients need to strictly enforce the rules of {{secctx-propagation}}.
Failure to do so,
in particular using a thusly announced proxy based on a certificate that attests the proxy's name,
would allow attackers to circumvent the client's security expectation.

When security is terminated at proxies (as is in DTLS and TLS),
a third party proxy can usually not satisfy this requirement;
these transports are limited to same-host proxies.

## Traffic misdirection {#proxy-foreign-advertisement}

Accepting arbitrary proxies,
even with security context propagation performed properly,
would allow attackers to redirect traffic through systems under their control.
Not only does that impact availability,
it also allows an attacker to observe traffic patterns.

This affects both OSCORE and (D)TLS,
as neither protect the participants' network addresses.

Other than the security context propagation rules,
there are no hard and general rules about when an advertised proxy is a suitable candidate.
Aspects for consideration are:

* When no direct connection is possible
  (e.g. because the resource to be accessed is served as coap+tcp and TCP is not implemented in the client,
  or because the resource's host is available on IPv6 while the client has no default IPv6 route),
  using a proxy is necessary if complete service disruption is to be avoided.

  While an adversary can cause such a situation (e.g. by manipulating routing or DNS entries),
  such an adversary is usually already in a position to observe traffic patterns.

* A proxy advertised by the device hosting the resource to be accessed is less risky to use than one advertised by a third party.

  The `/.well-known/core` resource is regarded as a source of authoritative information on the endpoint's CoAP related metadata,
  and can be queried early in the discovery process,
  or queried for verification (with filtering applied) after discovery through an RD.
  Other resources may be less trustworthy as they may be controlled by entities not trusted with the endpoint's traffic.

<!-- Note that these aspects' applicability is mutually exclusive:
When the advertisement was obtained from the target host,
that needs to be reachable. -->

{{ead}} describes an extension to {{I-D.ietf-lake-edhoc}} by which the client can verify that the proxy used by the client is recognized by the server.
This is similar to querying `/.well-known/core` for any proxies advertised there,
but happens earlier in the connection establishment,
and leaves the decision whether the proxy is legitimate to the server.

It only conveys information about the URI of the proxy.
The mapping of any host name inside it to an IP address,
or of an IP address to a routing decision,
is left to the security mechanisms of the respective layers.

## Protecting the proxy

A widely published statement about a host's availability as a proxy can cause many clients to attempt to use it.

This is mitigated in well-behaved clients by observing the rate limits of {{RFC7252}},
and by ceasing attempts to reach a proxy for the Max-Age of received errors.

Operators can further limit ill-effects
by ensuring that their client systems do not needlessly use proxies advertised in an unsecured way,
and by providing own proxies when their clients need them<!-- in a sense, avoid having starving clients that grab any straw at connectivity -->.


# IANA considerations

## Link Relation Types

IANA is asked to add two entries into the Link Relation Type Registry last updated in {{!RFC8288}}:

| Relation Name     | Description                                                                                  | Reference   |
| has-proxy         | The link target can be used as a proxy to reach URIs inside the origin of the link context.  | RFCthis     |
| has-unique-proxy  | Like has-proxy, and using this proxy implies scheme and host of the target.                  | RFCthis     |
{: #tab-iana title='New Link Relation types' }

## Resource Types

IANA is asked to add an entry into the "Resource Type (rt=) Link Target Attribute Values" registry under the Constrained RESTful Environments (CoRE) Parameters:

\[ The RFC Editor is asked to replace any occurrence of CPA-core.proxy with the actually registered attribute value. \]

Attribute Value: core.proxy

Description: Forward proxying services

Reference: \[ this document \]

## Service Parameter Key (SvcParamKey)


This document adds the following entry to the SVCB Service Parameters
registry ({{?RFC9460}}). The definition of this parameter can be found in {{upcomingtransports}}.

| Number  | Name           | Meaning                            | Reference       |
| ------- | -------------- | ---------------------------------- | --------------- |
| 10 (suggested)      | coaptransport        | CoAP transport protocol        | \[TBD-this-spec\] {{upcomingtransports}} |

## Underscored and Globally Scoped DNS Node Names {#iana-underscored}

IANA is NOT YET requested to add the following entries to the Underscored and Globally Scoped DNS Node Names registry
(in the DNS Parameters group)
established in {{?RFC8552}}
and thus enables its use with SVCB records:

* SVCB, `_coap`, {{svcb-discovery}} of this document
* SVCB, `_coaps`, {{svcb-discovery}} of this document

The request for registration is deliberately not expressed at this point
because it is yet to be revisited whether the creation of a "COAP" / "COAPS" RR pair
similar to the "HTTPS" RR would be preferable.

--- back

# Change log

Since draft-ietf-core-transport-indication-04:

* Not just the scheme, but also the authority value influences the transport selection.
  - Add guidance section for new transports.
  - Point out that registerd names already can fan out to different addresses.
* Rephrase and simplify security considerations, especially by limiting unique proxying for TLS.
* Add recommendation to new scheme authors to use "coap"/"coaps" and let the resolution process guide the selection.
  - Remove proxy-schemes attribute from core.proxy because of its greatly reduced value.
* Update "Related work" appendix to cover SVCB instead of SRV records
* Rename to "Transport Indication", using "protocol" only for other protocols, in established phrases, or when referring to CoAP as a general protocol.
* Add note linking CoAP-over-WS's .well-known/coap to dohpath
* Remove OSCORE vs. unique-proxy open point
* EDHOC EAD: Describe response option content
* Editorial updates

Since draft-ietf-core-transport-indication-03:

* Added appendices on alternative history and Literals beyond IP addresses.
  The remaining document was not brought in sync with those new parts.

Since draft-ietf-core-transport-indication-02:

* Added EAD appendix, adjusted security considerations to match.

Since draft-ietf-core-transport-indication-01:

* Simplify same-host proxy behavior by referring to existing RFC7252 mandate.
* proxy-links= lookup: Refer to prior art.

Since draft-ietf-core-transport-indication-00:

* Add section on canonical URIs that are not necessarily reachable.
* Clarify that the the "hosts" relation is followed transitively.
* Cross reference with compatible multicast-notifications concept.

Since draft-amsuess-core-transport-indication-03:

* No changes (merely changing the name after WG adoption)

Since -02 (mainly processing reviews from Marco and Klaus):

* Acknowledge that 'coap://hostname/' is not the proxy but a URI that (in a particular phrasing) is used to stand in for the proxy's address (while it regularly identifies a resurce on the server)
* Security: Referencing traffic misdirection already in the first security block.
* Security: Add (incomplete) considerations for unique-proxy case.
* Narrow down "unique" proxy semantics to those properties used by the client, allowing unique proxies to be co-hosted with forward proxies.
* "Client picked proxies" clarified to merely illustrate how this is compatible with them.
* Use of "hosts" relation sharpened.
* Precision on how this does and does not consider changing transports.
* "Related work" section demoted to appendix.
* Add note on DTLS session resumption.
* Variable renaming.
* Various editorial fixes.

Since -01:

* Removed suggestion for generally trusted proxies;
  now stating that with (D)TLS,
  "a third party proxy can usually not satisfy \[the security context propagation requirement\]".
* State more clearly that valid cache entries for resources aliased through has-unique-proxy can be used.
* Added considerations for Multipath TCP.
* Added concrete suggestion and example for advertisement of general proxies.
* Added concrete suggestion for RD lookup extension that provides proxies.
* Minor editorial and example changes.

Since -00:

* Added introduction
* Added examples
* Added SCHC analogy
* Expanded security considerations
* Added guidance on choice of transport, and canonical addresses
* Added subsection on interaction with a Resource Directory
* Added comparisons with related work, including rdlink and DNS-SD sketches
* Added IANA considerations
* Added section on open questions

# Related work and applicability to related fields

## On HTTP

The mechanisms introduced here are similar to the Alt-Svc header of {{?RFC7838}}
in that they do not create different application-visible addresses,
but provide dispatch through lower transport implementations.

In HTTP, different versions of the protocol (i.e., different transports)
are distinguished using a protocol identifier equivalent to an ALPN.
This works well because all relevant transports use transport layer security and thus can use ALPNs.
In contrast, the currently specified CoAP transports predate ALPNs,
and specified per-transport schemes, which enable the use of URIs that indicate transports.

To accommodate the message size constraints typical of CoRE environments,
and accounting for the differences between HTTP headers and CoAP options,
information is delivered once at discovery time.

Using the has-proxy and has-unique-proxy with HTTP URIs as the context is NOT RECOMMENDED;
the HTTP provisions of the Alt-Svc header and ALPN are preferred.

## Using DNS

DNS Service Binding resource records (SVCB RRs)
described in {{?RFC9460}} can carry many of the details otherwise negotiated using the proxy relations.
In HTTP, they can be used in a way similar to Alt-Svc headers.

SVCB records were not specified when CoAP was specified for TCP,
but could have been (see {{althist}}).

If at any point SVCB records for CoAP are defined,
name resolution produces a set of transport details that can be used immediately
without the need for a `has-proxy` link.
Explicit `has-proxy` links would still be relevant for third party advertised proxies.

## Using names outside regular DNS

Names that are resolved through different mechanisms than DNS,
or names which are defined within the scope of DNS but have no universally valid answers to A/AAAA requests,
can be advertised using the relation types defined here and CoAP discovery.

In {{fig-rdlink}}, a server using a cryptographic name as described in {{?I-D.amsuess-t2trg-rdlink}} is discovered and used.

~~~~
Req: to [ff02::fd]:5683 on UDP
Code: GET
Uri-Path: /.well-known/core
Uri-Query: rel=has-proxy
Uri-Query: anchor=coap://nbswy3dpo5xxe3denbswy3dpo5xxe3de.ab.rdlink.arpa

Res: from [2001:db8::1]:5683
Content-Format: application/link-format
Payload:
<coap+tcp://[2001:db8::1]>;rel=has-unique-proxy;
  anchor="coap://nbswy3dpo5xxe3denbswy3dpo5xxe3de.ab.rdlink.arpa"


Req: to [2001:db8::1]:5683 on TCP
Code: GET
OSCORE: ...
Uri-Path: /sensors/temp
Observe: 0

Res: 2.05 Content
OSCORE: ...
Observe: 0
Payload:
39.1°C
~~~~
{: #fig-rdlink title='Obtaining a sensor value from a local device with a global name'}

## Multipath TCP

When CoAP-over-TCP is used over Multipath TCP
and no Uri-Host option is sent,
the implicit assumption is that there is aliasing between URIs containing any of the endpoints' addresses.

As these are negotiated within MPTCP,
this works independently of this document's mechanisms.
As long as all the server's addresses are equally reachable,
there is no need to advertise multiple addresses that can later be discovered through MPTCP anyway.
When advertisements are long-lived and there is no single more stable address,
several available addresses can be advertised (independently of whether MPTCP is involved or not).
If a client uses an address that is merely a proxy address (and not a unique proxy address),
but during MPTCP finds out that the network location being accessed is actually an MPTCP alternative address of the used one,
the client MAY forego sending of the Proxy-Scheme and Uri-Path option.

\[ This follows from multiple addresses being valid for that TCP connection;
at some point we may want to say something about what that means for the default value of the Uri-Host option --
maybe something like "has the default value of any of the associated addresses, but the server may only enable MPTCP if there is implicit aliasing between all of them" (similar to OSCORE's statement)?  \]

\[ TBD: Do we need a section analog to this that deals with (D)TLS session resumption in absence of SNI? \]

# Open Questions / further ideas

* Advertising under a stable name:

  If a host wants to advertise its host name rather than its IP address during multicast, how does it best do that?

  Options, when answering from 2001:db8::1 to a request to ff02::fd are:

  ~~~~
  <coap://myhostname/foo>,...,
  <coap://[2001:db8::1]>;rel=has-unique-proxy;anchor="coap://myhostname"
  ~~~~

  which is verbose but formally clear, and

  ~~~~
  </foo>,...,
  <coap://[2001:db8::1]>;rel=has-unique-proxy;anchor="coap://myhostname"
  ~~~~

  which is compatible with unaware clients,
  but its correctness with respect to canonical URIs needs to be argued by the client, in this sequence

  * understanding the has-unique-proxy line,
  * understanding that the request that went to 2001:db8::1 was really a Proxy-Scheme/Uri-Host-elided version of a request to coap://myhostname, and then
  * processing any relative reference with this new base in mind.

  (Not that it'd need to happen in software in that sequence,
  but that's the sequence needed to understand how the `/foo` here is really `coap://myhostname/foo`).

  If CoRAL is used during discovery, a base directive or reverse relation to has-unique-proxy would make this easier.

# EDHOC EAD for verifying legitimate proxies {#ead}

This document sketches an extension to {{I-D.ietf-lake-edhoc}} that informs the server of the public address the client is using,
allowing it to detect undesired reverse proxies.

\[ This section is immature, and written up as a discussion starting point. Further research into prior art is still necessary. ]

The External Authorization Data (EAD) item with name "Proxy CRI", label 24-CPA, is defined for use with messages 1, 2 and 3.

A client can set this label in uncritical form, followed by a CRI ({{I-D.ietf-core-href}}) that is CBOR-encoded in a byte string as a CBOR sequence.
The transport indicated by the URI is the proxy the client chose from information advertised about the server.

If a server can not determine its set of legitimate proxies,
it ignores the option (as does any EDHOC implementation that is unaware of it).

If it recognizes the CRI as belonging to a legitimate proxy,
it places the empty label in its non-critical form in the next message to confirm the proxy choice.
Otherwise, it places the label in its critical form, either empty or containing a recommended CRI.
The client may then decide to discontinue using the proxy,
or to use more extensive padding options to sidestep the attack.
Both the client and the server may alert their administrators of a possible traffic misdirection.

# Alternative History: What if SVCB had been around before CoAP over TCP? {#althist}

This appendix explores a hypothetical scenario in which Service Binding (SVCB, {{?RFC9460}}) was around and supported before the controversial decision to establish the "coap+tcp" scheme.
It serves to provide a fresh perspective of what parts are logically necessary,
and to ease the exploration of how it may be used in the future.

## Hypothetical retrospecification

CoAP is specified for several transports:
CoAP over UDP, over DTLS, over TCP, over TLS and over (secure or insecure) WebSockets.
URIs of all these are expressed using the "coap" or "coaps" scheme,
depending on whether a (D)TLS connection is to be used.
\[ It is currently unclear whether the two schemes should also be unified; the rest of the text is left intentionally vague on that distinction. \]

Any server providing CoAP services
announces not only its address
but also its SVCB Service Parameters,
including at least one of `alpn` and `coaptransport`.

For example, a host serving "coap://sensor.example.com" and "coaps://sensor.example.com"
might have these records:

```
_coap.sensor.example.com IN SVCB 1 . alpn=coap,co coaptransport=tcp,udp port=61616
sensor.example.com IN AAAA 2001:db8::1
```

A client connecting to the server loops up the name's service parameters using its system's discovery mechanisms.

For example, if DNS is used, it obtains SVCB records for \_coap.sensor.example.com,
and receives the corresponding AAAA record either immediately from an SVCB aware resolver
or through a second query.
It learns that the service is available through CoAP-over-DTLS (ALPN "co"), CoAP-over-TLS (ALPN "coap"),
or through unencrypted TCP or UDP, and that port 61616 needs to be used in all cases.

If the server and the client do not have a transport in common,
or if one of them supports only IPv4 and the other only IPv6,
no exchange is possible;
the client may resort to using a proxy.

## Shortcomings

While the mechanism above would have unified the CoAP transports under a pair of schemes,
it would have rendered the use of IP literals impossible:
The URI `coap://[2001:db8::1]` would be ambiguous as to whether CoAP-over-UDP or CoAP-over-TCP should be used.
{{newlit}} provides a solution for this issue.

# Literals beyond IP addresses {#newlit}

\[
This section is placed here preliminarily:
After initial review in CoRE, this may be better moved into a separate document aiming for a wider IETF audience.
\]

## Motivation for new literal-ish names

IP literals were part of URIs from the start {{w3address}}.
Initially, they were equivalent to host names in their expressiveness,
save for their inherent difference that the former can be used without a shared resolver,
and the latter can be switched to a different network address.

This equivalence got lost gradually:
Certificates for TLS (its precursor SSL has been available since 1995 {{evossl}})
<!-- TBD cite - https://en.wikipedia.org/wiki/HTTPS or  ? -->
have only practically been available to host names.
The Host header introduced in HTTP 1.1 {{Section 14.23 of ?RFC2616}}
introduced name based virtual hosting in 1999.
DANE {{?RFC6698}}, which provides TLS public keys augmenting the or outside of the public key infrastructure,
is only available for host names resolved through DNSSEC.
SVCB records {{?RFC9460}} introduced in 2023
allow starting newer HTTP transports without going through HTTP/1.1 first,
enables load balancing, fail-over,
and enable Encrypted Client Hello --
again, only for host names resolved through DNS.

This document proposes an expression for the host component of a URI
that fills that gap.
Note that no attempt is yet made to register `service.arpa` in the .ARPA Zone Management;
that name is used only for the purpose of discussion.

[^prelim]

[^prelim]:
    The structure and even more the syntax used here is highly preliminary.
    They serves to illustrate that the desired properties can be obtained,
    and do not claim to be optimal.
    As one of many aspects, they are missing considerations for normalization
    and for internationalization.

## Structure of `service.arpa`

Names under service.arpa are structured into
an optional custom prefix,
an ordered list of key-value component pairs,
and the common suffix `service.arpa`.

The custom prefix can contain user defined components.
The intended use is labelling, and to differentiate services provided by a single host.
Any data is allowed within the structure of a URI (ABNF reg-name) and DNS name rules (63 bytes per segment).
(While not ever carried by DNS,
this upholds the constraints of DNS for names.
That decision may be revised later,
but is upheld while demonstrating that the desired properties can be obtained).

Component pairs consist of a registered component type
(no precise registry is proposed at this early point)
followed by encoded data.
The component type "--" is special in that it concatenates the values to its left and to its right,
creating component values that may exceed 63 bytes in length.

Initial component types are:

* "6": The value encodes an IPv6 address
  in {{?RFC5952}} format, with the colon character (":") replaced with a dash ("-").

  It indicates an address of a host providing the service.

  If any address information is present,
  a client SHOULD use that information to access the service.

* "4": The value encodes an IPv4 address
  in dotted decimal format {{?RFC1123}}, with the dot character (".") replaced with a dash ("-").

  It indicates an address of a host providing the service.

* "tlsa": The data of a DNS TLSA record {{?RFC6698}} encoded in base32 {{?RFC4648}}.

  Depending on the usage, this describes TLS credentials through which the service can be authenticated.

  If present,
  a client MUST establish a secure connection,
  and MUST fail the connection if the TLSA record's requirements are not met.

* "s": Service Parameters {{?RFC9460}}).
  SvcbParams in base32 encoding of their wire format.

  TBD: There is likely a transformation of the parameters' presentation format that is compatible with the requirements of the authority component,
  but this will require some more work on the syntax.

  If present,
  a client SHOULD use these hints to establish a connection.

  TBD: Encoding only the SvcParams and not priorities and targets appears to be the right thing to do for the immediate record,
  but does not enable load balancing and failover.
  Further work is required to explore how those can be expressed,
  and how data pertaining to the target can be expressed,
  possibly in a nested structure.

## Syntax of `service.arpa`

~~~abnf
name = [ custom ".-." ] *(component) "service.arpa"

custom = reg-name
component = 1*63nodot "." comptype "."
comptype = nodotnodash /  2*63nodot

; unreserved/subdelims without dot
nodot  = nodotnodash / "-"
; unreserved/subdelims without dot or dash
nodotnodash = ALPHA / DIGIT / "_" / "~" / sub-delims

; reg-name and sub-delims as in RFC3986
~~~

Due to {{?RFC3986}}'s rules,
all components are case insensitive and canonically lowercase.

Note that while using the IPvFuture mechanism of {{?RFC3986}} would avoid compatibility issues around the 63 character limit
and some of the character restrictions,
it would not resolve the larger limitation of case insensitivity.

## Processing `service.arpa`

A client accessing a resource under a service.arpa name
does not consult DNS,
but obtains information equivalent to the results of a DNS query from parsing the name.

DNS resolvers should refuse to resolve service.arpa names.
(They would have all the information needed to produce sensible results,
but operational aspects would need a lot of consideration;
future versions of this document may revise this).

## Examples

TBD: For SvcParams, the examples are inconsistent with the base32 recommendation;
they serve to explore the possible alternatives.

* http://s.alpn_h2c.6.2001-db8--1.service.arpa/ -- The server is reachable on 2001:db8::1 using HTTP/2

* https://mail.-.tlsa.amaqckrkfivcukrkfivcukrkfivcukrkfivcukrkfivcukrkfivcukrk.service.arpa/ -- No address information is provided, the client needs to resort to other discovery mechanisms.
  Any server is eligible to serve the resource if it can present a (possibly self-signed) certificate whose public key SHA256 matches the encoded value.
  The "mail.-." part is provided to the server as part of the Host header,
  and can be used for name based virtual hosting.

* coap://s.coaptransfer_tcp_coapsecurity_edhoc.6.2001-db8--1.service.arpa/ -- The server is reachable using CoAP over TCP with EDHOC security at 2001:db8::1. (The SVCB parameters are experimental values from {{?I-D.lenders-core-dnr}}).

# Acknowledgements

This document heavily builds on concepts
explored by Bill Silverajan and Mert Ocak in {{?I-D.silverajan-core-coap-protocol-negotiation}},
and together with Ines Robles and Klaus Hartke inside T2TRG.

\[ TBD: reviewers
Marco
Klaus
\]
