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
area: WIT
workgroup: MASQUE
keyword:
 - connect-ip
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

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

This document specifies an extension for IP header compression when using Connect-IP proxying.
It specifies a new capsule that provides static information for IP header fields
that are then left out when using the indicated a context ID.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
