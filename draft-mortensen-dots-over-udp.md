---
title: DDoS Open Threat Signaling over UDP
abbrev: DOTS over UDP
docname: draft-mortensen-dots-over-udp-00
date: @DATE@

area: Security
wg: DOTS
kw: Internet-Draft
cat: std

coding: us-ascii
pi:
  toc: yes
  sortrefs: no
  symrefs: yes

author:
      -
        ins: A. Mortensen
        name: Andrew Mortensen
        org: Arbor Networks, Inc.
        street: 2727 S. State St
        city: Ann Arbor, MI
        code: 48104
        country: United States
        email: amortensen@arbor.net

normative:
  RFC0768:
  RFC2119:
  RFC5405:
  RFC6520:
  RFC7030:
  RFC7540:
  I-D.ietf-dots-architecture:
  I-D.ietf-dots-requirements:
  I-D.ietf-tls-tls13:
  I-D.rescorla-tls-dtls13:

informative:
  RFC7252:
  I-D.hamilton-quic-transport-protocol:
  CURVECP:
    target: https://curvecp.org/
    title: "CurveCP: Usable security for the Internet"
    author:
      ins: D. Bernstein
      name: Daniel J. Bernstein
      org: University of Illinois at Chicago
    date: 2014
    format: HTML

--- abstract

This document describes the use of the Distributed-Denial-of-Service (DDoS) Open
Threat Signaling (DOTS) protocol over UDP [RFC0768].

--- middle

Introduction
============

The DOTS protocol described in [I-D.teague-dots-protocol] defines the protocol
message format and message exchanges, but purposely divides the protocol from
the transport in order to make DOTS adaptable to arbitrary transports.

This division is meant to simplify the process of bringing DOTS to modern secure
transports like QUIC [I-D.hamilton-quic-transport-protocol] and the experimental
CurveCP [CURVECP], as well as application-layer protocols like CoAP [RFC7252]
and HTTP/2 [RFC7540].

This document defines the most basic option for a DOTS protocol transport,
implementing the protocol over UDP, using DTLS1.3 [I-D.rescorla-tls-dtls13] to
secure the signaling session.


Terminology
-----------

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{RFC2119}}.

Terms used to define entity relationships, transmitted data, and methods of
communication are drawn from the terminology defined in
[I-D.ietf-dots-requirements].


Architecture
============

The architecture in which the DOTS protocol operates is assumed to be derived
from the architectural components and concepts described in
[I-D.ietf-dots-architecture].


Protocol
========

This document uses the signal channel message exchanges and message
serialization defined in [I-D.teague-dots-protocol], except as specified below.


Provisioning    {#protocol-provisioning}
------------

### DOTS Server Discovery

DOTS server address discovery is implementation-specific, but anticipates
methods such as direct configuration (i.e., DOTS server address is manually
provided to the DOTS client) and DNS SRV records will be the most common.

### Keying Material

A key challenge to deploying DOTS unsurprisingly is provisioning DOTS clients,
including the distribution of keying material to DOTS clients to make possible
the required mutual authentication of DOTS agents. DOTS over UDP relies on
Enrollment over Secure Transport (EST) [RFC7030] to overcome this. EST defines a
method of certificate enrollment by which domains operating DOTS servers may
provision DOTS clients with all necessary cryptographic keying material,
including a private key and certificate with which to authenticate itself.

This document does not specify which EST mechanism the DOTS client uses to
achieve initial enrollment. Each mechanism has certain advantages balanced by
obvious drawbacks. For example, HTTP-based client authentication for initial
enrollment is by far the simplest, allowing DOTS clients to enroll with just a
username and one-time-use password, but is also by far the most easily abused.
Client authentication using a previously installed certificate improves on
HTTP-based client authentication, but requires either a manufacturer-installed
certificate, or yet another mechanism to install the client certificate.
The EST mechanism for initial enrollment is therefore left to the operators of
the DOTS deployment.


Signal Channel
--------------

### Initialization      {#signal-channel-initialization}

[I-D.teague-dots-protocol] defines a DOTS signal channel initialization message
exchange in which the DOTS client initiates contact with the DOTS server by
connecting to the DOTS server on port 4646, but leaves instantiation of the
signal channel's security context up to DOTS transport documents.

DOTS over UDP uses DTLS1.3 [I-D.rescorla-tls-dtls13] to secure the signal
channel and provide the requisite confidentiality, integrity, and authenticity
for messages exchanged between DOTS agents [I-D.ietf-dots-requirements]. Signal
channel initialization assumes the presence of a DOTS client certificate
and DOTS server certificate obtained through EST as described in
{{protocol-provisioning}} above.

As described in [I-D.teague-dots-protocol], the DOTS client begins creation of
the security context for the the signaling session after successfully connecting
to the DOTS server on port 4646. The DOTS over UDP client makes that connection
over UDP.

Once connected, DOTS over UDP clients SHOULD attempt session resumption through
use of pre-shared keys (PSKs) created during previous signaling sessions with
the DOTS server.  Session resumption using PSKs from a previous DTLS session is
described in [I-D.ietf-tls-tls13].

If a DOTS client does not have a PSK from a previous signaling session, or if
the PSK is expired or otherwise invalid, the DOTS client will fallback to a full
DTLS1.3 handshake, using the private key and certificates obtained through EST
at the time of client provisioning. After performing a full DTLS handshake, the
DOTS server SHOULD send a NewSessionTicket message to the client after receiving
the client's Finished message, as described in [I-D.ietf-tls-tls13]. The DOTS
client SHOULD cache the PSK identity and use it for future session resumption.

Once the DTLS handshake is complete, the DOTS signal channel security context is
established, the DOTS over UDP client proceeds to establish the signaling
session by sending a channel initiatlization method, as described in
[I-D.teague-dots-protocol].


### Messaging

Once the signal channel security context is established through DTLS and the
signaling session is active, all messaging proceeds as defined in
[I-D.teague-dots-protocol], relying on DTLS to provide the requisite channel
security.

#### Heartbeats

DOTS over UDP purposely avoids using DTLS heartbeats [RFC6520] to maintain the
signaling session, depending instead on DOTS protocol heartbeats to track signal
session health.

#### DTLS Alerts

DOTS over UDP is subject to any condition triggering DTLS Alert messages as
described in [I-D.rescorla-tls-dtls13]. Should the alert lead to connection
termination, the DOTS over UDP MAY attempt to reestablish the 


Data Channel
------------

DOTS over UDP adopts the data channel as specified in [I-D.teague-dots-protocol]
with modification or extension.

Congestion Control Considerations
=================================

DOTS over UDP is subject to the congestion control discussion in [RFC5405].


Security Considerations
=======================

[I-D.teague-dots-protocol] describes general DOTS protocol security
considerations. Additionally, all security considerations in
[I-D.rescorla-tls-dtls13] apply to DOTS over UDP.

When provisioning DOTS clients using EST, all considerations in [RFC7030] also
apply to DOTS over UDP.


Contributors
============

Nik Teague
Verisign, Inc.
nteague@verisign.com
