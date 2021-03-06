> Read [original](https://tools.ietf.org/html/draft-ietf-quic-invariants-07) / [summary](../summary/draft-ietf-quic-invariants-07.md)

---

# Version-Independent Properties of QUIC

## Abstract

This document defines the properties of the QUIC transport protocol that are expected to remain unchanged over time as new versions of the protocol are developed.

## 1. Introduction

In addition to providing secure, multiplexed transport, QUIC [QUIC-TRANSPORT] includes the ability to negotiate a version.  This allows the protocol to change over time in response to new requirements.  Many characteristics of the protocol will change between versions.

This document describes the subset of QUIC that is intended to remain stable as new versions are developed and deployed.  All of these invariants are IP-version-independent.

The primary goal of this document is to ensure that it is possible to deploy new versions of QUIC.  By documenting the properties that can't change, this document aims to preserve the ability to change any other aspect of the protocol.  Thus, unless specifically described in this document, any aspect of the protocol can change between different versions.

Appendix A is a non-exhaustive list of some incorrect assumptions that might be made based on knowledge of QUIC version 1; these do not apply to every version of QUIC.

## 2. Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

This document uses terms and notational conventions from [QUIC-TRANSPORT].

## 3. An Extremely Abstract Description of QUIC

QUIC is a connection-oriented protocol between two endpoints.  Those endpoints exchange UDP datagrams.  These UDP datagrams contain QUIC packets.  QUIC endpoints use QUIC packets to establish a QUIC connection, which is shared protocol state between those endpoints.

## 4. QUIC Packet Headers

A QUIC packet is the content of the UDP datagrams exchanged by QUIC endpoints.  This document describes the contents of those datagrams.

QUIC defines two types of packet header: long and short.  Packets with long headers are identified by the most significant bit of the first byte being set; packets with a short header have that bit cleared.

Aside from the values described here, the payload of QUIC packets is version-specific and of arbitrary length.

### 4.1. Long Header

Long headers take the form described in Figure 1.  Bits that have version-specific semantics are marked with an X.




```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+
   |1|X X X X X X X|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Version (32)                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | DCID Len (8)  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |               Destination Connection ID (0..2040)           ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | SCID Len (8)  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Source Connection ID (0..2040)              ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |X X X X X X X X X X X X X X X X X X X X X X X X X X X X X X  ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                        Figure 1: QUIC Long Header
```

A QUIC packet with a long header has the high bit of the first byte set to 1.  All other bits in that byte are version specific.

The next four bytes include a 32-bit Version field (see Section 4.4).

The next byte contains the length in bytes of the Destination Connection ID (see Section 4.3) field that follows it.  This length is encoded as an 8-bit unsigned integer.  The Destination Connection ID field follows the DCID Len field and is between 0 and 255 bytes in length.

The next byte contains the length in bytes of the Source Connection ID field that follows it.  This length is encoded as a 8-bit unsigned integer.  The Source Connection ID field follows the SCID Len field and is between 0 and 255 bytes in length.

The remainder of the packet contains version-specific content.

### 4.2. Short Header

Short headers take the form described in Figure 2.  Bits that have version-specific semantics are marked with an X.




```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+
   |0|X X X X X X X|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Destination Connection ID (*)               ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |X X X X X X X X X X X X X X X X X X X X X X X X X X X X X X  ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                        Figure 2: QUIC Short Header
```

A QUIC packet with a short header has the high bit of the first byte set to 0.

A QUIC packet with a short header includes a Destination Connection ID immediately following the first byte.  The short header does not include the Connection ID Lengths, Source Connection ID, or Version fields.  The length of the Destination Connection ID is not specified in packets with a short header and is not constrained by this specification.

The remainder of the packet has version-specific semantics.

### 4.3. Connection ID

A connection ID is an opaque field of arbitrary length.

The primary function of a connection ID is to ensure that changes in addressing at lower protocol layers (UDP, IP, and below) don't cause packets for a QUIC connection to be delivered to the wrong endpoint. The connection ID is used by endpoints and the intermediaries that support them to ensure that each QUIC packet can be delivered to the correct instance of an endpoint.  At the endpoint, the connection ID is used to identify which QUIC connection the packet is intended for.

The connection ID is chosen by each endpoint using version-specific methods.  Packets for the same QUIC connection might use different connection ID values.

### 4.4. Version

QUIC versions are identified with a 32-bit integer, encoded in network byte order.  Version 0 is reserved for version negotiation (see Section 5).  All other version numbers are potentially valid.

The properties described in this document apply to all versions of QUIC.  A protocol that does not conform to the properties described in this document is not QUIC.  Future documents might describe additional properties which apply to a specific QUIC version, or to a range of QUIC versions.

## 5. Version Negotiation

A QUIC endpoint that receives a packet with a long header and a version it either does not understand or does not support might send a Version Negotiation packet in response.  Packets with a short header do not trigger version negotiation.

A Version Negotiation packet sets the high bit of the first byte, and thus it conforms with the format of a packet with a long header as defined in Section 4.1.  A Version Negotiation packet is identifiable as such by the Version field, which is set to 0x00000000.


```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+
   |1|X X X X X X X|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Version (32) = 0                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | DCID Len (8)  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |               Destination Connection ID (0..2040)           ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | SCID Len (8)  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Source Connection ID (0..2040)              ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Supported Version 1 (32)                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                   [Supported Version 2 (32)]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                  ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                   [Supported Version N (32)]                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                   Figure 3: Version Negotiation Packet
```

The Version Negotiation packet contains a list of Supported Version fields, each identifying a version that the endpoint sending the packet supports.  The Supported Version fields follow the Version field.  A Version Negotiation packet contains no other fields.  An endpoint MUST ignore a packet that contains no Supported Version fields, or a truncated Supported Version.

Version Negotiation packets do not use integrity or confidentiality protection.  A specific QUIC version might authenticate the packet as part of its connection establishment process.

An endpoint MUST include the value from the Source Connection ID field of the packet it receives in the Destination Connection ID field.  The value for Source Connection ID MUST be copied from the Destination Connection ID of the received packet, which is initially randomly selected by a client.  Echoing both connection IDs gives clients some assurance that the server received the packet and that the Version Negotiation packet was not generated by an off-path attacker.

An endpoint that receives a Version Negotiation packet might change the version that it decides to use for subsequent packets.  The conditions under which an endpoint changes QUIC version will depend on the version of QUIC that it chooses.

See [QUIC-TRANSPORT] for a more thorough description of how an endpoint that supports QUIC version 1 generates and consumes a Version Negotiation packet.

## 6. Security and Privacy Considerations

It is possible that middleboxes could use traits of a specific version of QUIC and assume that when other versions of QUIC exhibit similar traits the same underlying semantic is being expressed. There are potentially many such traits (see Appendix A).  Some effort has been made to either eliminate or obscure some observable traits in QUIC version 1, but many of these remain.  Other QUIC versions might make different design decisions and so exhibit different traits.

The QUIC version number does not appear in all QUIC packets, which means that reliably extracting information from a flow based on version-specific traits requires that middleboxes retain state for every connection ID they see.

The Version Negotiation packet described in this document is not integrity-protected; it only has modest protection against insertion by off-path attackers.  QUIC versions MUST define a mechanism that authenticates the values it contains.

## 7. IANA Considerations

This document makes no request of IANA.



## 8. References

### 8.1. Normative References

[QUIC-TRANSPORT] Iyengar, J., Ed. and M. Thomson, Ed., "QUIC: A UDP-Based Multiplexed and Secure Transport", draft-ietf-quic-transport-23 (work in progress), September 2019.

[RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997, <https://www.rfc-editor.org/info/rfc2119>.

[RFC8174]  Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017, <https://www.rfc-editor.org/info/rfc8174>.

### 8.2. Informative References

[QUIC-TLS] Thomson, M., Ed. and S. Turner, Ed., "Using Transport Layer Security (TLS) to Secure QUIC", draft-ietf-quic-tls-23 (work in progress), September 2019.

[RFC5116]  McGrew, D., "An Interface and Algorithms for Authenticated Encryption", RFC 5116, DOI 10.17487/RFC5116, January 2008, <https://www.rfc-editor.org/info/rfc5116>.

### 8.3. URIs

[1] https://mailarchive.ietf.org/arch/search/?email_list=quic

[2] https://github.com/quicwg

[3] https://github.com/quicwg/base-drafts/labels/-invariants

## Appendix A. Incorrect Assumptions

There are several traits of QUIC version 1 [QUIC-TRANSPORT] that are not protected from observation, but are nonetheless considered to be changeable when a new version is deployed.

This section lists a sampling of incorrect assumptions that might be made based on knowledge of QUIC version 1.  Some of these statements are not even true for QUIC version 1.  This is not an exhaustive list, it is intended to be illustrative only.

The following statements are NOT guaranteed to be true for every QUIC version:

*  QUIC uses TLS [QUIC-TLS] and some TLS messages are visible on the wire

*  QUIC long headers are only exchanged during connection establishment

*  Every flow on a given 5-tuple will include a connection establishment phase

*  The first packets exchanged on a flow use the long header

*  QUIC forbids acknowledgments of packets that only contain ACK frames, therefore the last packet before a long period of quiescence might be assumed to contain an acknowledgment

*  QUIC uses an AEAD (AEAD_AES_128_GCM [RFC5116]) to protect the packets it exchanges during connection establishment

*  QUIC packet numbers appear after the Version field

*  QUIC packet numbers increase by one for every packet sent

*  QUIC has a minimum size for the first handshake packet sent by a client

*  QUIC stipulates that a client speaks first

*  A QUIC Version Negotiation packet is only sent by a server

*  A QUIC connection ID changes infrequently

*  QUIC endpoints change the version they speak if they are sent a Version Negotiation packet

*  The version field in a QUIC long header is the same in both directions

*  Only one connection at a time is established between any pair of QUIC endpoints

## Author's Address

Martin Thomson Mozilla

Email: mt@lowentropy.net
