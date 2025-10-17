---
title: "Extension to Connect-IP for static IP header compression"
abbrev: "Connect-IP Compression"
category: info

docname: draft-kuehlewind-masque-ip-compression-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Multiplexed Application Substrate over QUIC Encryption"
keyword:
 - connect-ip
venue:
  group: "Multiplexed Application Substrate over QUIC Encryption"
  type: "Working Group"
  mail: "masque@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/masque/"
  github: "mirjak/draft-masque-ip-compression"
  latest: "https://mirjak.github.io/draft-masque-ip-compression/draft-kuehlewind-masque-ip-compression.html"

author:
-
    fullname: Mirja Kühlewind
    organization: Ericsson
    email: mirja.kuehlewind@ericsson.com
-
    fullname: Marcus Ihlar
    organization: Ericsson
    email: marcus.ihlar@ericsson.com
-
    fullname: Magnus Westerlund
    organization: Ericsson
    email: magnus.westerlund@ericsson.com

normative:

informative:


--- abstract

This document specifies an extension for IP header compression when using Connect-IP proxying.


--- middle

# Introduction

This document specifies an extension for IP header compression when using Connect-IP
{{!CONNECT-IP=RFC9484}} proxying.
It specifies a way to provides static information for IP header fields
that are then left out when using the indicated a context ID.

This document defines four new capsules to assign new context IDs and provide static information
in IPv4 and IPv6 headers, acknowlegde the use of such a context ID, or cancel its use.
The capsules MUST only be used with Connect-IP {{CONNECT-IP}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## IP4_COMPRESSION_ASSIGN and IP6_COMPRESSION_ASSIGN capsules {#capsule-assign}

The IPv4 Compression Assign capsule (capsule type TBD) is used to register a
Context ID and provides static values of the IP header fields that will be removed from the
HTTP Datagram Payload when using this Conext ID. It has the following format:

~~~
IP4_COMPRESSION_ASSIGN Capsule {
  Type (i) = TBD,
  Length (i),
  Context ID (i),
  IHL (1),
  DSCP (1),
  ECN (1),
  ID (1),
  Total Length (1),
  Reserved (1),
  DF (1),
  MF (1),
  Fragment Offset (1),
  TTL (1),
  Protocol (1),
  Checksum (1),
  Source Address (1),
  Destination Address (1),
  Source Port (1),
  Destination Port (1),
  IPv4 Header (20),
  [Transport Ports (16)]
}
~~~
{: #fig-capsule-ipv4-assign title="IPv4 Compression Assign Capsule Format"}

It contains the following fields:

IHL:
: This flag indicates if Internet Header Length field of the IP header is static.

DHCP:
: This flag indicates if DiffServ Code Point field of the IP header is static.

ECN:
: This flag indicates if ECN codepoint field of the IP header is static.

Total Length:
: This flag indicates if Total length field of the IP header is static.

ID:
: This flag indicates if the Identification field of the IP header is static.

Reserved:
: This flag indicates if the Reserved flag of the IP header is static.

DF:
: This flag indicates if the Don't Fragment flag of the IP header is static.

MF:
: This flag indicates if the More Fragments flag of the IP header is static.

Fragment Offset:
: This flag indicates if the Fragment Offset field of the IP header is static.

TTL:
: This flag indicates if the TTL field of the IP header is static.

Protocol:
: This flag indicates if the Protocol field of the IP header  is static.

Checksum:
: This flag indicates if  Checksum field of the IP header is static.

Source Address:
: This flag indicates if the IP Source Address of the IP header is static.

Destination Address:
: This flag indicates if the IP Destination Address of the IP header is static.

Source Port:
: This flag indicates if the Source Port of the header that follows the IP header is static.

Destination Port:
: This flag indicates if the Destination Port of the header that follows the IP header is static.

IPv4 Header:
: This field contains the first 20 bytes of the IPv4 header. Fields that are not indicated as static will be omitted
  in the compression and therefore their value is not relevant.  

Transport Ports:
: If either the Source Port or Destination Port Flag is set, this field has to be present and contains
  one byte for the source port followed by one byte for the destionation port.

~~~
IP6_COMPRESSION_ASSIGN Capsule {
  Type (i) = TBD,
  Length (i),
  Context ID (i),
  Traffic class (1),
  Flow label (1),
  Payload Length (1),
  Next header (1),
  Hop limit (1),
  Source Address (1),
  Destination Address (1),
  Source Port (1),
  Destination Port (1),
  IPv6 Header (40),
  [Transport Ports (16)]
}
~~~
{: #fig-capsule-ipv6-assign title="IPv6 Compression Assign Capsule Format"}

It contains the following fields:

Traffic Class:
: This flag indicates if the Traffic Class field of the IP header is static.

Flow Label:
: This flag indicates if the Flow Label field of the IP header is static.

Payload Length:
: This flag indicates if the Payload length field of the IP header is static.

Next Header:
: This flag indicates if The value of the Next Header field of the IP header is static.

Hop Limit:
: This flag indicates if the Hop Limit field of the IP header is static.

Source Address:
: This flag indicates if the IP Source Address of the IP header is static.

Destination Address:
: This flag indicates if the IP Destination Address of the IP header is static.

Source Port:
: This flag indicates if the Source Port of the header that follows the IP header is static.

Destination Port:
: This flag indicates if the Destination Port of the header that follows the IP header is static.

IPv6 Header:
: This field contains the first 40 bytes of the IPv6 header. Fields that are not indicated as static will be omitted
  in the compression and therefore their value is not relevant.  

Transport Ports:
: If either the Source Port or Destination Port Flag is set, this field has to be present and contains
  one byte for the source port followed by one byte for the destionation port.

When the indicated Context ID is used all IP header fields that indicated as static
as well as the version field MUST be omitted in the HTTP datagram payload and the receiver of the
datagram with the indicated Context ID MUST reconstruct the IP header accordingly before forwarding
the payload.

When an endpoint receives a IP4_COMPRESSION_ASSIGN or IP6_COMPRESSION_ASSIGN
capsule, it MUST either accept or reject the corresponding registration:

* if it accepts the registration, the receiver MUST return a COMPRESSION_ACK
  capsule with the Context ID set to the one from the received
  IP4_COMPRESSION_ASSIGN or IP6_COMPRESSION_ASSIGN capsule back to its peer

* if it rejects the registration, the receiver MUST respond by sending a
  COMPRESSION_CLOSE capsule with the Context ID set to the one from the
  received IP4_COMPRESSION_ASSIGN or IP6_COMPRESSION_ASSIGN capsule.

As mandated in {{Section 4 of !CONNECT-UDP=RFC9298}}, clients can only allocate even
Context IDs, while proxies can only allocate odd ones. This makes the
registration capsules from this document unambiguous. For example, if a
client receives a COMPRESSION_ASSIGN capsule with an even Context ID, that
has to be an echo of a capsule that the client initially sent, indicating
that the proxy accepted the registration. Since the value 0 was reserved by
unextended connect-udp, the Context ID value of COMPRESSION_ASSIGN can never
be zero.

Endpoints MUST NOT send two COMPRESSION_ASSIGN capsules with the same
Context ID. If a recipient detects a repeated Context ID, it MUST treat the
capsule as malformed. Receipt of a malformed capsule MUST be treated as an
error processing the Capsule Protocol, as defined in {{Section 3.3 of
!HTTP-DGRAM=RFC9297}}.

Endpoints MAY pre-emptively use Context IDs not yet acknowledged by the peer
via COMPRESSION_ACK, knowing that those HTTP Datagrams can be dropped if
they arrive before the corresponding IP4_COMPRESSION_ASSIGN or IP6_COMPRESSION_ASSIGN capsule,
or if the peer rejects the registration.

## IP_COMPRESSION_ACK capsule {#capsule-ack}

The IP Compression Acknowledgment capsule (capsule type TBD) confirms
registration of a context ID that was received in an IP4_COMPRESSION_ASSIGN
or IP6_COMPRESSION_ASSIGN capsule.

~~~
IP_COMPRESSION_ACK Capsule {
  Type (i) = TBD,
  Length (i),
  Context ID (i),
}
~~~
{: #fig-capsule-ack title="IP Compression Acknowledgment Capsule Format"}

An endpoint can only send a COMPRESSION_ACK capsule if it received a
IP4_COMPRESSION_ASSIGN or IP6_COMPRESSION_ASSIGN capsule with the same Context ID.
If an endpoint receives COMPRESSION_ACK capsule for a context ID it did not attempt
to register in an IP4_COMPRESSION_ASSIGN or IP6_COMPRESSION_ASSIGN capsule, that
capsule is considered malformed.

## IP_COMPRESSION_CLOSE capsule {#capsule-close}

The IP Compression Close capsule (capsule type TDB) serves two purposes. It
can be sent as a direct response to a received IP4_COMPRESSION_ASSIGN or
IP6_COMPRESSION_ASSIGN capsule capsule, to indicate that the registration
was rejected. It can also be sent later to indicate the closure of a
previously assigned registration.

~~~
IP_COMPRESSION_CLOSE Capsule {
  Type (i) = TBD,
  Length (i),
  Context ID (i),
}
~~~
{: #fig-capsule-close title="IP Compression Close Capsule Format"}

Once an endpoint has either sent or received a IP_COMPRESSION_CLOSE for a given
Context ID, it MUST NOT send any further datagrams with that Context ID.
Since the value 0 was reserved by unextended connect-udp, the Context ID
value of IP_COMPRESSION_CLOSE can never be zero.

# Security Considerations

TODO Security


# IANA Considerations

# IANA Considerations

This document will request IANA to register the following new items to the
"HTTP Capsule Types" registry maintained at
<[](https://www.iana.org/assignments/masque)>:

| Value |      Capsule Type      |
|:------|:-----------------------|
|  TBD  | IP4_COMPRESSION_ASSIGN |
|  TBD  | IP6_COMPRESSION_ASSIGN |
|  TBD  | IP_COMPRESSION_ACK     |
|  TBD  | IP_COMPRESSION_CLOSE   |
{: #iana-capsules-table title="New Capsules"}

All of these new entries use the following values for these fields:

Status:
: provisional (permanent if this document is approved)

Reference:
: This document

Change Controller:
: IETF

Contact:
: MASQUE Working Group <masque@ietf.org>

Notes:
: None
{: spacing="compact"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
