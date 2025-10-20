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

To accommodate these limitations, HTTP Message Signatures defines a number of
HTTP Message Components ({{Section 2 of RFC 9421}}) and specifies rules for transforming components into the
input to the signature algorithm ({{Section 3.1 of RFC9421}}). The value of most components are extracted
directly from the bytes of the HTTP message; others are derived from the
message through a well-specified process.

All components are derived from the HTTP messages themselves. Consequentially,
an on-path attacker with access to the HTTP messages transmitted between the
client and server can replay a signed message at will.
This is described in {{Sectoin 7.2.2 of RFC9421}}.

The `nonce` parameter provides some defense against replay attacks, but this
mechanism is not applicable in all deployment scenarios. For example, it is
common for two TLS servers to be authoritative for the same DNS name. (This
setup is commonly referred to as "multi-CDN".) In this scenario, the first
server can intercept a signed request from a client, then replay that request
to the second server, thereby impersonating the client.

The goal of this document is to make replay protection more robust. A new
derived component is defined for HTTP Message Signatures whose value is set to
key material exported from TLS as defined in {{Section 7.5 of !RFC8446}}. This
binds the signed message to the underlying TLS channel, thereby ensuring the
signature is never accepted outside of that channel.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The `@ekm` Derived Component

A new derived component is defined with the name `@ekm`.

The contents of this component are the output of a call to the exporter
interface of the underlying TLS connection as defined in {{Section 7.5 of
!RFC8446}}, encoded in base64 {{!RFC4648}}. The label parameter of the exporter
function is set to "http-sig-ekm" and the context value is the version of TLS.
For TLS 1.3 (i.e., {{!RFC8446}}) its value is `0x0304`.

TLS 1.3 or later is REQUIRED. This derived component is not compatible with
HTTP messages sent in plaintext or over TLS 1.2 and below.

> NOTE We could in principal specify this for TLS 1.2, if we need to.

If the signer and verifier do not agree on the value of `@ekm`, then the
signature will not verify. If the signer and verifier share a TLS connection
between them, then they will compute the same value. If they do not share a
direct TLS connection, it is possible to architect a system such that the
verifier does not directly call the exporter interface, but is simply provided
its output on a trusted channel. This behaviour works, but requires the
verifier and caller to trust each other.

> OPEN ISSUE How do we negotiate usage of `@ekm`? Both the signer and verifier
> need to support the new derived component into generate message signatures
> that use it, but the signer might not know if the verifier uses it.

# Security Considerations

> TODO Define a channel binding and say why it prevents replays between CDNs.

# IANA Considerations

> TODO Update the "HTTP Signature Derived Component Names" registry.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
