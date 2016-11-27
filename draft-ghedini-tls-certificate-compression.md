---
title: Transport Layer Security (TLS) Certificate Compression
docname: draft-ghedini-tls-certificate-compression-latest

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
 -
    ins: A. Ghedini
    name: Alessandro Ghedini
    org: Cloudflare, Inc.
    email: alessandro@cloudflare.com

normative:
  RFC2119:
  RFC5226:
  RFC5246:
  RFC4366:

informative:
  RFC7924:

--- abstract

In Transport Layer Security (TLS) handshakes, certificate chains often take up
the majority of the bytes transmitted.

This document describes how certificate chains can be compressed to reduce the
amount of data transmitted and avoid some round trips.

--- middle

# Introduction

In order to reduce latency and improve performance it can be useful to reduce
the amount of data exchanged during a Transport Layer Security (TLS) handshake.

[RFC7924] describes a mechanism that allows a client and a server to avoid
transmitting certificates already shared in an earlier handshake, but it
doesn't help when the client connects to a server for the first time and
doesn't already have knowledge of the server's certificate chain.

This document describes a mechanism that would allow certificates to be
compressed during full handshakes.

## Notational Conventions

The words "MUST", "MUST NOT", "SHOULD", and "MAY" are used in this document.
It's not shouting; when they are capitalized, they have the special meaning
defined in [RFC2119].

# Certificate Compression Extension

*TODO: Should this use RFC7250 instead?*

This document defines a new extension type (compress_certificates(TBD)), which
is used in ClientHello and ServerHello messages. The extension type is
specified as follows.

~~~
    enum {
         compress_certificates(TBD), (65535)
    } ExtensionType;
~~~

This allows TLS clients and servers to negotiate that the server sends its
certificate chain in compressed form.

In order to negotiate the use of compressed certificates, clients MAY include
an extension of type "compress_certificates" in the extended client hello. The
"extension_data" field of this extension SHALL be empty.

Servers that receive an extended hello containing a "compress_certificates"
extension MAY agree to use compressed certificates by including an extension
of type "compress_certificates", with empty "extension_data", in the extended
server hello.

# Server Certificate Message

When a ClientHello message contains the "compress_certificates" and the server
agrees to use compressed certificates by sending the same extension back, then
the server MUST send the certificate message shown in Figure 2.

~~~
    struct {
         TBD
    } Certificate;
~~~

*TODO: Define CertificateEntry for TLS 1.3?*

# Security Considerations

TBD

# IANA Considerations

## New Entry to the TLS ExtensionType Registry

Create an entry, compress_certificates(TBD), in the existing registry for
ExtensionType (defined in [RFC5246]).

--- back
