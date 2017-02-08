---
title: Transport Layer Security (TLS) Certificate Compression
docname: draft-ghedini-tls-certificate-compression-latest
abbrev: TLS Certificate Compression

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
 -
    ins: A. Ghedini
    name: Alessandro Ghedini
    org: Cloudflare, Inc.
    email: alessandro@cloudflare.com
 -
    ins: V. Vasiliev
    name: Victor Vasiliev
    org: Google
    email: vasilvv@google.com

normative:
  RFC1950:
  RFC2119:
  RFC5226:
  RFC5246:
  RFC4366:
  RFC7932:

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

# Notational Conventions

The words "MUST", "MUST NOT", "SHALL", "SHOULD", and "MAY" are used in this
document.  It's not shouting; when they are capitalized, they have the special
meaning defined in [RFC2119].

# Negotiating Certificate Compression

This document defines a new extension type (compress_certificates(TBD)), which
is used by the client and the server to negotiate the use of compression as well
as the choice of the compression algorithm.

By sending the compress_certificates message, the client indicates to the server
the certificate compression algorithms it supports.  The "extension_data" field
of this extension in the ClientHello SHALL contain a
CertificateCompressionAlgorithms value:

~~~
    enum {
        zlib(0),
        brotli(1),
        (255)
    } CertificateCompressionAlgorithm;

    struct {
        CertificateCompressionAlgorithm algorithms<1..2^8>;
    } CertificateCompressionAlgorithms;
~~~

If the server supports any of the algorithms offered in the ClientHello, it MAY
respond with an extension indicating which compression algorithm it chose.  In
that case, the extension_data SHALL be a CertificateCompressionAlgorithm value
corresponding to the chosen algorithm.  If the server has chosen to not use any
compression, it MUST NOT send the compress_certificates extension.

# Server Certificate Message

If the server picks a compression algorithm and sends it in the ServerHello, the
format of the Certificate message is altered as follows:

~~~
    struct {
         uint16 uncompressed_length;
         opaque compressed_certificate_message<1..2^16-1>;
    } Certificate;
~~~

compressed_certificate_message
: The compressed body of the Certificate message, in the same format as the
  server would normally express it. The compression algorithm defines how the
  bytes in the compressed_certificate_message are converted into the
  Certificate message.

uncompressed_length
: The length of the Certificate message once it is uncompressed.  If after
  decompression the specified length does not match the actual length, the
  client MUST abort the connection with bad_certificate message.

If the specified compression algorithm is zlib, then the Certificate message
MUST be compressed with the ZLIB compression algorithm, as defined in [RFC1950].
If the specified compression algorithm is brotli, the Certificate message MUST
be compressed with the Brotli compression algorithm as defined in [RFC7932].

If the client cannot decompress the received Certificate message from the
server, it MUST tear down the connection with the "bad_certificate" alert.

The extension only affects the Certificate message from the server.  It does not
change the format of the Certificate message sent by the client.

# Security Considerations

After decompression, the Certificate message MUST be processed as if it were
encoded without being compressed.  This way, the parsing and the verification
have the same security properties as they would have in TLS normally.

Since certificate chains are typically presented on a per-server name basis, the
attacker does not have control over any individual fragments in the Certificate
message, meaning that they cannot leak information about the certificate by
modifying the plaintext.

The implementations SHOULD bound the memory usage when decompressing the
Certificate message.

The implementations MUST limit the size of the resulting decompressed chain to
the specified uncompressed length, and they MUST abort the connection if the
size exceeds that limit.  Implementations MAY impose a lower limit on the chain
size in addition to the 65536 byte limit imposed by TLS framing, in which case
they MUST apply the same limit to the uncompressed chain before starting to
decompress it.

# IANA Considerations

## Update of the TLS ExtensionType Registry

Create an entry, compress_certificates(TBD), in the existing registry for
ExtensionType (defined in [RFC5246]).

## Registry for Compression Algorithms

This document establishes a registry of compression algorithms supported for
compressing the Certificate message, titled "Certificate Compression Algorithm
IDs", under the existing "Transport Layer Security (TLS) Extensions" heading.

The entries in the registry are:

| Algorithm Number | Description              |
|:-----------------|:-------------------------|
| 0                | zlib                     |
| 1                | brotli                   |
| 224 to 255       | Reserved for Private Use |

The values in this registry shall be allocated under "IETF Review" policy for
values strictly smaller than 64, and under "Specification Required" policy
otherwise (see [RFC5226] for the definition of relevant policies).

--- back
