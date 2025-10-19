---
title: "HTTP Signature Component for TLS Channel Binding"
category: info

docname: draft-hoypat-httpbis-message-signatures-ekm-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "HTTP"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "HTTP"
  type: "Working Group"
  mail: "ietf-http-wg@w3.org"
  arch: "https://lists.w3.org/Archives/Public/ietf-http-wg/"
  github: "cjpatton/draft-hoypat-httpbis-message-signatures-ekm"
  latest: "https://cjpatton.github.io/draft-hoypat-httpbis-message-signatures-ekm/draft-hoypat-httpbis-message-signatures-ekm.html"

author:
 -
    fullname: "Jonathan Hoyland"
    email: "jonathan.hoyland@gmail.com"

 -
    fullname: "Christopher Patton"
    organization: "Cloudflare"
    email: "chrispatton+ietf@gmail.com"

normative:

informative:

...

--- abstract

A derived component is specified for HTTP Message Signatures that binds the
signature to the underlying secure channel (TLS over TCP or QUIC), thereby
ensuring a signed message transmitted over one channel cannot be retransmitted
over another. The component consists of key material exported from TLS.

--- middle

# Introduction

HTTP Message Signatures {{!RFC9421}} allow various components of an HTTP
message to the authenticated by the sender either using a digital signature or
a message authentication code (MAC). The exact set of components to be signed
may very depending upon the application:

1. the components that need to be signed depend on the security considerations
   of the application; and

1. some components of the message may not available at the time of signing or
   verification.

To accommodate these limitations, {{!RFC9421}} defines a number of common
components and specifies rules for transforming components into the input to
the signature algorithm or MAC. The value of most components are extracted
directly from the bytes of the HTTP message; others are derived from the
message through a well-specified process.

All components are derived from the HTTP messages themselves. Consequentially,
an on-path attacker with access to the HTTP messages transmitted between the
client and server can replay a signed message at will.

The `nonce` parameter provides some defense against this, but this mechanism is
not applicable in all deployment scenarios. For example, it is common for two
TLS servers to be authoritative for the same DNS name. (This setup is commonly
referred to as "multi-CDN".) In this scenario, the first server can intercept a
signed request from a client, then replay that request to the second server,
thereby impersonating the client.

This goal of this document is to make replay protection more robust. A new
derived component is defined for HTTP Message Signatures whose value is set to
key material exported from TLS as defined in {{Section 7.5 of !RFC8446}}. This
binds the signed message to the underlying TLS channel, thereby ensuring the
signature is never accepted outside of that channel.

TODO Talk about WBA?

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The `@ekm` Derived Component

A new derived component is defined with the name `@ekm`.

The contents of this component are the output of a call to the exporter
interface of the underlying TLS connection encoded in base64. The label
parameter of the exporter function is set to "http-sig-ekm" and the context
value is empty. [[ Should we account for QUIC here?]]

If the `@ekm` component is present, but its value does not verify the signature
should be rejected. It is possible to architect a system such that the verifier
does not directly call the exporter interface, but is simply provided its output
on a trusted channel. This behaviour works, but requires the verifier and caller
to trust each other.

TODO

* Basically
  <https://github.com/rustls/rustls/blob/08cde8c57b1146752e188663de351ff2b2a69b67/rustls/src/tls13/key_schedule.rs#L858>.
  We should just need to define the label and context strings.

* MUST negotiate TLS 1.3 {{!RFC8446}}. We can probably do TLS 1.2 as well,
  but let's not bother right now.

* Warn users about when this mechanism can't be used. The signer and verifier
  have to share a TLS connection between them, or proxy the exported key
  material by some other means. We should probably to discourage this.

# Security Considerations

TODO

* Define a channel binding and say why it prevents replays between CDNs.

# IANA Considerations

TODO

* Update the "HTTP Signature Derived Component Names" registry.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
