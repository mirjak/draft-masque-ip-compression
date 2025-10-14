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
  [IHL (4)],
  [DSCP (6)],
  [ECN (2)],
  [Omit Length (1)],
  [ID (16)],
  [Reserved (1)],
  [DF (1)],
  [MF (1)],
  [Fragment Offset (13)],
  [TTL (8)],
  [Protocol (8)],
  [Checksum (1)],
  [Source Address (32)],
  [Destination Address (32)]
}
~~~
{: #fig-capsule-ipv4-assign title="IPv4 Compression Assign Capsule Format"}

It contains the following fields:

IHL:
: The value Internet Header Length field of the IP header. The value MUST be between 5 and 15.
  Capules with other values MUST be considered malformed.

DHCP:
: The value of DiffServ Code Point field of the IP header.

ECN:
: The value of ECN codepoint field of the IP header.

Omit Length:
: An indication that the Total length field of the IP header is omitted. If omitted in the payload,
  the receiver need to calulate the Total length after reconstructing the IP header.

ID:
: The value of the Identification of the IP header.

Reserved:
: The value of the Reserved flag of the IP header.

DF:
: The value of the Don't Fragment flag of the IP header.

MF:
: The value of the More Fragments flag of the IP header.

Fragment Offset:
: The value of the Fragment Offset field of the IP header.

TTL:
: The value of the TTL field of the IP header.

Protocol:
: The value of the Protocol field of the IP header

Omit Checksum:
: An indication that the Checksum field of the IP header is omitted. If omitted in the payload,
  the receiver need to calulate the checksum after reconstructing the IP header.

Source Address:
: The IP Source Address of the IP header used in this context. This address MUST be a registered address.
  Capules with other values MUST be considered malformed.

Destination Address:
: The IP Destination Address of the IP header used in this context. This address MUST be a registered address.
  Capules with other values MUST be considered malformed.

~~~
IP6_COMPRESSION_ASSIGN Capsule {
  Type (i) = TBD,
  Length (i),
  Context ID (i),
  [Traffic class (8)],
  [Flow label (20)],
  [Omit Payload Length (1)],
  [Next header (8)],
  [Hop limit (8)],
  [Source Address (128)],
  [Destination Address (128)]
}
~~~
{: #fig-capsule-ipv6-assign title="IPv6 Compression Assign Capsule Format"}

It contains the following fields:

Traffic Class:
: The value of the Traffic Class field of the IP header.

Flow Label:
: The value of the Flow Label field of the IP header.

Omit Payload Length:
: An indication that the Payload length field of the IP header is omitted. If omitted in the payload,
  the receiver need to calulate the Total length after reconstructing the IP header.

Next Header:
: The value of the Next Header field of the IP header.

Hop Limit:
: The value of the Hop Limit field of the IP header.

Source Address:
: The IP Source Address of the IP header used in this context. This address MUST be a registered address.
  Capules with other values MUST be considered malformed.

Destination Address:
: The IP Destination Address of the IP header used in this context. This address MUST be a registered address.
  Capules with other values MUST be considered malformed.

When the indicated Context ID is used all IP header fields that are provided with a static value
in the capsule as well as the version field MUST be omitted in the HTTP datagram payload and the receiver of the
datagram with the indicated Context ID MUST reconstruct the IP accordingly before forwarding
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

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
