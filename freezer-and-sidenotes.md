Some text was in the main document that is detracting from the main story,
providing rationale or unexplored roads.

These snippets are taken here for safekeeping.

----

## Terminology

### Using URIs to identify transport endpoints {#identifying}

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
