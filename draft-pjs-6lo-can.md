---
title: "Transmission of IPv6 Packets over Controller Area Network"
abbrev: "6LoCAN"
category: std

docname: draft-pjs-6lo-can-latest
submissiontype: IETF
number:
date: 2026-11-02
consensus: true
v: 3

area: "int"
workgroup: "6lo"

keyword:
 - Controller Area Network
 - CAN
 - ISO 11898
 - ISO-TP
 - 6LowPAN
 - IPv6
 - Adaptation Layer
 - IoT
 - Internet of Things

ipr: trust200902

author:
 -
    fullname: "Alexander Wachter"
    organization: "Graz University of Technology"
    email: alexander@wachter.cloud
 -
    fullname: "Jungsoo Park"
    role: editor
    initials: "J."
    surname: "Park"
    organization: "Electronics and Telecommunications Research Institute"
    abbrev: "ETRI"
    street: "218 Gajeongno, Yuseong-gu"
    city: "Daejeon"
    code: "34129"
    country: "South Korea"
    phone: "+82 42 860 6514"
    email: pjs@etri.re.kr

normative:
  RFC2464:
  RFC4291:
  RFC4944:
  RFC4861:
  RFC4862:
  RFC6282:
  RFC6775:
  RFC7136:
  RFC7217:
  RFC8064:
  RFC8200:
  RFC8505:
  RFC8724:
  I-D.ietf-6lo-schc-15dot4:

informative:
  RFC2119:
  RFC4919:
  RFC8174:
  I-D.wachter-6lo-can:
  ISO11898-1:
    title: "Road vehicles — Controller area network (CAN) Part 1: Data link layer and physical coding sublayer"
    author:
      org: ISO
    date: 2024-05
    seriesinfo:
      "ISO Std": "11898-1:2024"
    target: https://www.iso.org/standard/86384.html
  ISO11898-2:
    title: "Road vehicles — Controller area network (CAN) Part 2: High-speed physical medium attachment (PMA) sublayer"
    author:
      org: ISO
    date: 2024-03
    seriesinfo:
      "ISO Std": "11898-2:2024"
    target: https://www.iso.org/standard/85120.html
  ISO15765-2:
    title: "Road vehicles — Diagnostic communication over Controller Area Network (DoCAN) Part 2: Transport protocol and network layer services"
    author:
      org: ISO
    date: 2024-04
    seriesinfo:
      "ISO Std": "15765-2:2024"
    target: https://www.iso.org/standard/84211.html

--- abstract

Controller Area Network (CAN) is a fieldbus initially designed for automotive applications.
It has since been adopted for use in various industrial applications, such as CANopen and building automation systems.
CAN operates as a multi-master bus with frame identifiers of 11 bits or 29 bits. The CAN standard, defined in the ISO 11898 series, specifies both the physical and data-link layers.
This document describes how to transfer IPv6 packets over CAN using ISO-TP, a dedicated addressing scheme for CAN nodes, and IP header compression (IPHC).

--- middle

# Introduction {#Introduction}

Controller Area Network (CAN) is mostly known for its use in the automotive domain.
However, it is also used in industrial applications as CANopen, building automation and many more.

It is a two-wire wired-AND multi-master bus that uses CSMA/CR in its arbitration field.
CAN uses 11-bit (standard ID) and 29-bit (extended ID) identifiers to identify frames.
The arbitration field also incorporates a Remote Transmission Request (RTR), marking a frame as an RTR-frame.
The maximum payload data size is 8 octets for classical CAN and 64 octets for CAN-FD.

The minimal MTU of IPv6 is 1280 octets, and therefore, a mechanism to support a larger payload is needed.
This document uses a slightly modified version of the ISO-TP protocol ({{ISO15765-2}}) to transfer data up to 4095 octets per packet.
Mapping addresses to identifiers uses an addressing scheme with a 14-bit source address, a 14-bit destination address, and a multicast bit.
This scheme uses extended identifiers only.

To make data transfer more efficient IPHC {{RFC6282}} is used.

Due to the limited address space of 14 bits, random address generation would generate duplicate addresses with an unacceptably high probability.
For this reason, a link-layer duplicate address detection is introduced to resolve address conflicts.

An Ethernet border translator is designed to connect a 6LoCAN bus segment to other networks.


# Applicability {#applicability-sec}

CAN is used in domains other than the automotive one it was designed for. Two properties
recur in those domains: a bus that is already installed and certified as part of the
equipment, and devices that are increasingly expected to be reachable from an IP network
for monitoring, configuration or software update.

Elevators are an example. An elevator installation uses CAN to connect the controller with
the car, the door operator and the landing stations. The bus is specified as part of the
installation and cannot be replaced without re-certifying it, while the same installation
is expected to report its state to a building management system or to a remote maintenance
service. Such connectivity is commonly provided by a gateway that translates a
vendor-specific CAN protocol into an IP protocol, and that translation has to be provided
for each combination of equipment and management system.

Running IPv6 on the existing CAN bus removes that per-combination translation. The
controller, the car and the landing stations become IPv6 nodes that a management system
addresses in the same way as any other node, and the Ethernet Border Translator
(see {{translator-sec}}) is the only element that has to be adapted to the building
network. The same reasoning applies to other installations that use CAN for their internal
communication, such as CANopen machinery and building automation.

6LoCAN assumes a single bus segment with a modest number of nodes that are powered from the
installation rather than battery operated, and traffic that consists mostly of small
messages. It is not intended for installations that need the throughput of Ethernet, nor
for battery-operated nodes that need the duty-cycling support of {{RFC6775}} and
{{RFC8505}}.

# Conventions and Terminology {#Terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in
all capitals, as shown here.

This specification requires readers to be familiar with all the terms and concepts that are discussed in
"IPv6 Stateless Address Autoconfiguration" {{RFC4862}},
"IPv6 over Low-Power Wireless Personal Area Networks (6LoWPANs): Overview, Assumptions, Problem Statement, and Goals" {{RFC4919}},
"Transmission of IPv6 Packets over IEEE 802.15.4 Networks" {{RFC4944}},
"Neighbor Discovery Optimization for IPv6 over Low-Power Wireless Personal Area Networks (6LoWPANs)" {{RFC6775}},
and "SCHC: Generic Framework for Static Context Header Compression and Fragmentation" {{RFC8724}}.

6LoWPAN Node (6LN):
: A 6LoWPAN node is any host or router participating in a LoWPAN. This term is used when referring to situations in which either a host or router can play the role described.

6LoWPAN Router (6LR):
: An intermediate router in the LoWPAN that is able to send and receive Router Advertisements (RAs) and Router Solicitations (RSs), as well as forward and route IPv6 packets. 6LoWPAN Routers are present only in route-over topologies.

6LoWPAN Border Router (6LBR):
: A border router located at the junction of separate 6LoWPAN networks or between a 6LoWPAN network and another IP network. There may be one or more 6LBRs at the 6LoWPAN network boundary. A 6LBR is the responsible authority for IPv6 prefix propagation for the 6LoWPAN network it is serving. An isolated LoWPAN also contains a 6LBR in the network that provides the prefix(es) for the isolated network.

Duplicate Address Detection (DAD):
: A mechanism used in IPv6 networks to ensure that no two devices share the same address, preventing address conflicts.

Static Context Header Compression and fragmentation (SCHC):
: A header compression and fragmentation framework defined in {{RFC8724}}. It relies on a
  context that is known to both endpoints before a packet is sent, and elides the header
  fields whose values can be derived from that context.


# Controller Area Network Overview {#Controller-area-network-overview}

This section provides a brief overview of Controller Area Network (CAN), as specified in {{ISO11898-1}} and {{ISO11898-2}}.
CAN has two wires, CAN High and CAN Low.
When a dominant (0) bit is transmitted, CAN High is driven to a higher voltage than CAN Low,
so that the differential voltage between the two wires is nominally 2V.
Both wires are at approximately the same level when a recessive (1) bit is transmitted, so
that the differential voltage is nominally 0V. {{ISO11898-2}} specifies the voltage levels.
Because of the wired-AND structure, a dominant bit overrides a recessive bit.
The CAN specification does not define any structure for the identifier.

To resolve collisions in the arbitration field, a CAN controller checks for overridden recessive bits.
The sender that was sending the recessive bit then stops the transmission.
Therefore an identifier with all zeros has the highest priority.

CAN controllers are usually able to filter frames by identifiers and only pass frames where the filter matches.
The identifiers can be masked in order to define which bits of the identifier must match and which ones are ignored.

## ISO-TP Overview {#iso-tp-overview}

This section provides information about the use of ISO-TP ({{ISO15765-2}}) in this document.
Parts of ISO-TP are used to provide a reliable way for sending up to 4095 octets as a packet.
It includes a flow-control mechanism for unicast-packets and timeouts.

Multicast packets do not use any flow-control mechanism and are therefore not covered by the ISO-TP standard.
However, the fragmentation and reassembly mechanism is still used for multicast packets.

ISO-TP defines four different types of frames: Single-Frames (SF), First-Frames (FF), Consecutive-Frames (CF), and Flow Control Frames (FC).
Single-Frames are used when the payload data size is small enough to fit into a single CAN frame.
For larger payload data sizes, a First-Frame indicates the start of the message, Consecutive-Frames carry the payload data and Flow Control Frames steer the transmission.
Network address extension and packet size larger than 4095 octets defined by ISO 15765-2 MUST NOT be used for 6LoCAN.
Only CAN-FD Single-Frame packets are useful due to their payload size of up to 64 octets.
Eight octet payload of classical CAN is too small to carry any IPv6 headers.
This applies to the transfer of IPv6 packets only; link-layer duplicate address detection
uses the classical CAN frame format (see {{link-layer-duplicate-address-detection}}).


# Specification of IPv6 over CAN {#Spec-IPv6-over-CAN}

CAN technologies have requirements owing to low power consumption and allowed protocol overhead. 6LoWPAN standards
{{RFC4944}} {{RFC6775}} {{RFC6282}} provide useful functionality
for reducing the overhead of IPv6 over CAN. This functionality consists of link-local IPv6 addresses and stateless IPv6 address autoconfiguration
(see Sections {{addr-conf-sec}} and {{link-local-addr-sec}}), Neighbor Discovery
(see {{nd-sec}}), header compression (see {{hc-sec}}) and fragmentation (see {{FAR-sec}}).

## Protocol Stack {#IPv6-over-CAN-protocol-stack-sec}

{{IPv6-over-CAN-protocol-stack-fig}} illustrates the IPv6-over-CAN protocol stack. Upper-layer protocols can be transport-layer protocols
(e.g., TCP and UDP), application-layer protocols, and other protocols capable of running on top of IPv6.

~~~
+----------------------------------------+
|     Upper-Layer Protocols (CANopen)    |
+----------------------------------------+
|                 IPv6                   |
+----------------------------------------+
|   Adaptation Layer for IPv6 over CAN   |
+----------------------------------------+
|   ISO-TP     |   CAN Data Link Layer   |
+----------------------------------------+
|           CAN Physical Layer           |
+----------------------------------------+
~~~
{: #IPv6-over-CAN-protocol-stack-fig title="Protocol Stack for IPv6 over CAN"}

The Adaptation Layer for IPv6 over CAN supports Neighbor Discovery, stateless address autoconfiguration, header compression, and fragmentation and reassembly, based on 6LoWPAN.
Note that 6LoWPAN header compression {{RFC6282}} does not define header compression for TCP.

## CAN Frame Identifiers {#node-addr-scheme}

This section provides information about the 14-bit node address to CAN identifier mapping.

Because CAN uses identifiers to identify the frame's content, an addressing scheme is introduced to map node addresses to identifiers.
Every node has a unique 14-bit address. This address is assigned either statically or randomly.
The addressing scheme uses the 29-bit extended identifier only. It is a combination of a source address, a destination address, and a multicast bit.
The multicast bit is at the highest bit-position, which causes the multicast traffic to have the lowest priority on the bus.

The address 0x3DFE is reserved for link-layer duplicate address detection, and address 0x3DF0 is reserved for the Ethernet border translator.
Addresses from 0x0100 to 0x3DEF are used as node addresses.
Other addresses (0x0000 to 0x00FF and 0x3DF0 to 0x3FFF) are reserved or used for special purposes.
Note that a lower address number has a higher priority on the bus.

The ranges listed in {{tab-address-ranges}} apply to node addresses, that is, to the source
address of every frame and to the destination address of unicast frames (see {{unicast}}).
They do not apply to the destination address of multicast frames: that field carries the
low-order 14 bits of the multicast group as specified in {{multicast}}, and the multicast bit
distinguishes such a value from a node address.

6LoCAN does not use the 11-bit standard identifiers. They may be used for other purposes.

| Address | Description |
|---------|-------------|
| 0x3DFF - 0x3FFF | Reserved |
| 0x3DFE | LLDAD |
| 0x3DF1 - 0x3DFD | Reserved |
| 0x3DF0 | Ethernet Translator |
| 0x0100 - 0x3DEF | Node addresses |
| 0x0000 - 0x00FF | Reserved |
{: #tab-address-ranges title="Node address ranges"}

~~~
 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|M|     DEST     |     SRC      |
+-+--------------+--------------+
~~~
{: #fig-identifier-layout title="Frame Identifier"}

M:
: Multicast.

DEST:
: Destination Address (14 bits).

SRC:
: Source Address (14 bits).

For example, a destination of 0x3055 and source address of 0x3AAF result in the following identifier:

~~~
 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|0|11000001010101|11101010101111|
+-+--------------+--------------+
~~~
{: #fig-unicast-id title="Unicast frame identifier example"}

A multicast group of 1 and a source address of 0x3AAF result in the following identifier:

~~~
 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|1|00000000000001|11101010101111|
+-+--------------+--------------+
~~~
{: #fig-multicast-id title="Multicast frame identifier example"}

### Unicast Frame Identifier {#unicast}

For unicast packets, the multicast bit is set to zero, and the 14-bit source address is the address of the sender.
The 14-bit destination address of the receiver is discovered by IPv6 NDP defined in {{RFC4861}}.
Every node MUST be able to receive all frames targeting its address as the destination address.

### Multicast Frame Identifier {#multicast}

For multicast packets, the multicast bit is set to one, and the 14-bit source address is the address of the sender.
The 14-bit destination address is the last 14 bits of the multicast group.
Every node MUST be able to receive all frames matching the last 14 bits of all joined multicast groups as the destination address.

### Node Address Generation {#address-generation}

Every node has a 14-bit address. This address MUST be unique within the CAN bus segment.
The address can either be statically defined or assigned randomly.
For the random address assignment, the node tries randomly chosen addresses until the link-layer duplicate address detection succeeds.
The link-layer duplicate address detection prevents nodes from assigning an address already in use.

### Link-Layer Duplicate Address Detection {#link-layer-duplicate-address-detection}

This section provides information about how to perform link-layer duplicate address detection (LLDAD).

LLDAD is introduced to prevent collisions of CAN identifiers and makes it possible to use random address assignment with only 14 bits of address space.
To perform an LLDAD, a LLDAD-request is sent. If there is no DAD-response sent back, the DAD is considered successful.
The node MUST wait for a response for at least 100ms.

LLDAD-requests are remote transmission request (RTR) frames with the desired address as the destination and 14 bits entropy as the source address.
The entropy prevents identifier collisions when nodes are trying to get the same address at the same time.

LLDAD frames MUST use the classical CAN frame format.
The CAN FD frame format does not support remote frames: the RTR bit position is replaced by
the Remote Request Substitution (RRS) bit, which is always transmitted as dominant ({{ISO11898-1}}).
Nodes that exchange data in the CAN FD frame format therefore send and receive LLDAD-requests
and DAD-responses in the classical CAN frame format.
Both frame formats may coexist on the same bus segment.
Consequently, a 6LoCAN node MUST support the classical CAN frame format even when all of its
data traffic uses the CAN FD frame format.

DAD-responses are data-frames sent to the LLDAD address (0x3DFE) with the responder's address as the source address.
Both LLDAD-request and DAD-response have a data length of zero.

The node MUST be configured to receive RTR frames with the desired address as the destination address before the LLDAD-request is sent
and frames with the LLDAD address as long as the LLDAD is in progress. This prevents from assigning the same address to more than one node
when sending the LLDAD-request at the same time.
The ability to receive RTR frames with the desired address as the destination address MUST be kept as long as the node uses the address.
The response to LLDAD-requests that matches the node address MUST be sent before the requesting node stops waiting for the response, which is 100ms.

{{fig-dad-fail}} shows a DAD Fail example where node A performs a LLDAD-request on address 0x3055 where this address is already in use by node B.

~~~
    Node A            Node B
       |--LLDAD-request->|
       |                 |
       |<-LLDAD-response-|

LLDAD-request identifier:
This frame is a Remote Transmission Request (RTR)
 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|0|11000001010101|   entropy    |
+-+--------------+--------------+

DAD-response identifier:
 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|0|11110111111110|11000001010101|
+-+--------------+--------------+
~~~
{: #fig-dad-fail title="DAD Fail example"}

## Stateless Address Autoconfiguration {#addr-conf-sec}

This section defines how to obtain an IPv6 Interface Identifier.

The Interface Identifier for link-local addresses SHOULD be formed by taking the node address
and zero filling it to the left.
The resulting value is an IID in the sense of {{RFC7136}} and carries no embedded meaning
beyond the node address.
For example, an address of hexadecimal value 0x3AAF results in the following IID:

~~~
|0              1|1              3|3              4|4              6|
|0              5|6              1|2              7|8              3|
+----------------+----------------+----------------+----------------+
|0000000000000000|0000000000000000|0000000000000000|0011101010101111|
+----------------+----------------+----------------+----------------+
~~~
{: #fig-generate-iid title="IID from Address 0x3AAF"}

Deriving the IID from the node address allows IPHC {{RFC6282}} to elide the IID of a
link-local address, because the receiver can reconstruct it from the CAN frame identifier.
Any other IID has to be carried in-line, which costs 8 octets in every frame that carries an
IPv6 header. On a link whose frames carry at most 64 octets of payload, this is the
difference between a usefully compressed header and an uncompressed one.

{{RFC8064}} recommends that stable IIDs be generated with the method of {{RFC7217}} rather
than from a link-layer address. The link-local IID defined above is a deliberate deviation
from that recommendation, made for the header compression reason given above; its privacy
properties are discussed in {{security-considerations}}.
For addresses other than link-local addresses, a 6LoCAN node SHOULD generate IIDs as
described in {{RFC7217}}. Such IIDs cannot be elided by IPHC and are carried in-line.

## IPv6 Link-Local Address {#link-local-addr-sec}

The IPv6 link-local address {{RFC4291}} for a 6LoCAN interface is formed by appending the Interface Identifier, as defined above, to the prefix FE80::/64.

~~~
  10 bits            54 bits                  64 bits
+----------+-----------------------+----------------------------+
|1111111010|         (zeros)       |    Interface Identifier    |
+----------+-----------------------+----------------------------+
~~~
{: #fig-link-local-address title="Link-Local address from IID"}

## Neighbor Discovery {#nd-sec}

A 6LoCAN bus segment is a single link. Every node attached to the segment can reach every
other node of that segment directly, and nodes do not forward packets between each other
within a segment.

Neighbor Discovery for IPv6 {{RFC4861}} is used for address resolution and for neighbor
unreachability detection. The Source/Target Link-Layer Address option carries the 14-bit
node address in the form specified in {{unicasting-sec}}.

The Neighbor Discovery optimizations for 6LoWPANs {{RFC6775}} {{RFC8505}} address
route-over, multi-hop topologies in which multicast is expensive or unreliable.
Because a 6LoCAN segment is a single link and CAN provides a multicast mechanism
(see {{multicast}}), these optimizations are not required within a segment.
Connectivity between a 6LoCAN segment and other links is provided by the Ethernet Border
Translator (see {{translator-sec}}) rather than by a 6LBR.

## Header Compression {#hc-sec}

### Traditional 6LoWPAN Header Compression {#hc-subsec-6lowpan}

TBD

### SCHC Header Compression {#hc-subsec-schc}

Refer to {{I-D.ietf-6lo-schc-15dot4}}.

## Maximum Transmission Unit {#mtu-sec}

IPv6 requires every link to have an MTU of 1280 octets or greater {{RFC8200}}.
The subset of ISO-TP used in this document (see {{FAR-sec}}) carries up to 4095 octets
per packet and therefore satisfies this requirement.

The MTU of a 6LoCAN bus segment MUST be at least 1280 octets and MUST NOT exceed
4095 octets. Implementations SHOULD use an MTU of 1280 octets.
All nodes attached to the same bus segment MUST use the same MTU; 6LoCAN provides no
mechanism to negotiate the MTU on a per-node basis.

A larger MTU increases the reassembly buffer that every receiving node has to provide
and the number of CAN frames that a single IPv6 packet occupies.
Because ISO-TP transfers are interleaved with other bus traffic at frame granularity,
a long transfer does not block frames of higher priority, but it does increase the
latency of traffic whose identifiers have a lower priority than the transfer.
An MTU larger than 1280 octets SHOULD therefore be used only where the resulting bus
occupancy is acceptable for the application.

## Fragmentation and Reassembly Considerations {#FAR-sec}

A subset of ISO-TP (ISO 15765-2) is used to fragment and reassemble the packets.
This subset of ISO-TP can send packets with a payload size of up to 4095 octets, enough for IPv6 minimum MTU size of 1280 octets.
ISO-TP is designed for CAN and its small payload data size and therefore preferred over {{RFC4944}} fragmentation.

The 6LoWPAN fragmentation would use more than the half of the available payload for the fragmentation headers.
This fact prevents 6LoWPAN fragmentation from being used for 6LoCAN.

## IPv6 Address Mapping {#addr-map-sec}

### Unicast Address Mapping {#unicasting-sec}

The address resolution procedure for mapping IPv6 non-multicast addresses into CAN Data link layer Addresses (Node addresses) follows the general description in
Sections 4.6.1 and 7.2 of {{RFC4861}}, unless otherwise specified.

The Source/Target Link-Layer Address option has the following form when the addresses are 14-bit CAN Data Link Layer Addresses.

~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      Type     |   Length=1    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               |
+-     Padding (all zeros)     -+
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Rsv| CAN Data Link-Layer Addr. |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #unicasting-fig title="IPv6 Unicast Address Mapping"}

Option fields:

{:indent="4"}
Type:
: 1: This is for the Source Link-Layer Address.
  2: This is for the Target Link-Layer Address.

Length:
: This is the length of this option (including the Type and Length fields) in units of 8 octets. The value of this field is 1 for 14-bit CAN Data Link Layer addresses.

Rsv:
: Reserved. 2 bits. These bits MUST be set to zero by the sender and MUST be ignored by the
  receiver.

CAN Data Link-Layer Address:
: The 14-bit node address of the sender (Source Link-Layer Address) or of the target
  (Target Link-Layer Address).

### Multicast Address Mapping {#multicast-map-sec}

CAN has no link-layer multicast address. An IPv6 multicast address is therefore mapped
directly onto a frame identifier: the multicast bit is set and the destination field carries
the low-order 14 bits of the multicast group, as specified in {{multicast}}.
No link-layer address resolution is performed for a multicast destination, and the
Source/Target Link-Layer Address option is not used for one.

Because the CAN data link layer does not provide multicast, ISO-TP is used for multicast
frames without its flow control mechanism; see {{iso-tp-overview}} and {{FAR-sec}}.

## Frame Format {#sec-frame-format}

This section provides information about data arrangement in the frame data field.

The first byte(s) of the CAN frames always contains the ISO-TP header.
For the first frame, the ISO-TP header is followed by the IPHC and in-line header fields.
The IPHC and in-line might be spread over several frames. The payload data follows directly after.

~~~
+-----------------------------------------+
| ISO-TP Header (1-3 octets)              |
+-----------------------------------------+
| Dispatch + LOWPAN_IPHC (2-3 octets)     |
+-----------------------------------------+
| In-line IPv6 Header Fields (0-38 octets)|
+-----------------------------------------+
| Payload                                 |
+-----------------------------------------+
~~~
{: #fig-frame-format title="6LoCAN Frame Format"}

The in-line IPv6 header fields are the fields that LOWPAN_IPHC {{RFC6282}} could not elide:
traffic class and flow label (4 octets), next header (1 octet), hop limit (1 octet),
source address (16 octets) and destination address (16 octets). This amounts to 38 octets
when none of them can be elided, and to 0 octets when all of them can.

Packets with a destination or source address of the 0x3DF0 (Translator address) carry the Ethernet MAC address inline directly after the ISO-TP Header.
For packets destined for the translator, it is the destination MAC address, and for packets originated by the translator, it is the source MAC address.

~~~
+-----------------------------------------+
| ISO-TP Header (1-3 octets)              |
+-----------------------------------------+
| Ethernet MAC Address (48 bits)          |
+-----------------------------------------+
| Dispatch + LOWPAN_IPHC (2-3 octets)     |
+-----------------------------------------+
| In-line IPv6 Header Fields (0-38 octets)|
+-----------------------------------------+
| Payload                                 |
+-----------------------------------------+
~~~
{: #fig-frame-format-translator title="6LoCAN Translator Frame Format"}


# Ethernet Border Translator {#translator-sec}

This section specifies how 6LoCAN packets are translated to and from Ethernet frames.

The Ethernet Border Translator connects 6LoCAN bus-segments either to other 6LoCAN bus-segments or other technologies.
Ethernet is a widely used technology that provides enough bandwidth to connect several 6LoCAN segments.
A mechanism like the 6LBR is not necessary because there is no routing on 6LoCAN segments.
To provide routing or switching capabilities, the Ethernet Border Translator connects a 6LoCAN network to such devices via Ethernet.

Bus segments MUST NOT have more than one translator. The translator has a fixed node address (0x3DF0) and a range of Ethernet MAC addresses.
Every packet sent to this node address or any multicast address is forwarded to Ethernet.
Every Ethernet frame matching the MAC address range and every multicast Ethernet frame is forwarded to the 6LoCAN bus-segment.

For translating a 6LoCAN packet to an Ethernet frame, the source address is extended with a 34 bits MAC address prefix, and the IPHC compressed headers are decompressed.
The 34 bits prefix MUST be chosen in a way that the resulting 48-bit MAC address forms a locally administered address that is unique on the link.
The destination MAC is carried in-line before the compressed IPv6 header (see {{sec-frame-format}}, {{fig-frame-format-translator}}).
ICMPv6 messages MUST be checked for Link-Layer Address Options (LLAO), and if an LLAO is present, it MUST be changed to the extended link-layer address.
For translating Ethernet frames to 6LoCAN packets, the source MAC address is carried in-line, the destination node address is the last 14 bits of the MAC address, and the IPv6 headers are compressed using IPHC.

The Neighbor Caches of the networking stack must be able to store link-layer addresses with a size of 14 and 48 bits.

For multicast Ethernet frames, the last 14 bits of the multicast group is the destination address, and the multicast bit is set. The destination address MAY also be reconstructed from the destination multicast address.
The destination Ethernet MAC address is formed from the destination IP address as described in {{RFC2464}} section 7.

If the translator includes a network stack, the translator MUST NOT use a MAC address within the ranges used for translation, with the following exception:
The translator MAY use the extended MAC address that corresponds to the translator node address.

# Internet Connectivity Scenarios {#connection-scenario-sec}

This section shows an example of a 6LoCAN segment connected to an Ethernet network and
examples of the address translations specified in {{translator-sec}}.

{{fig-translator-setup}} shows an example setup of a 6LoCAN segment connected to an Ethernet network.

{{fig-unicast-mac-translation}} shows a translation from Ethernet MAC to CAN identifier.
The source (src) MAC address is carried in-line in the CAN frame data.
The 34-bit MAC address prefix for this example is 02:00:5E:10 followed by two zero bits,
written as 02:00:5E:10:00/34.

~~~
|0                            4|4                              9
|0                            7|8                              5
+------------------------------+-------------------------------+
| dest MAC (02:00:5E:10:30:55) |  src MAC (02:00:5E:10:00:FF)  |
+------------------------------+-------------------------------+
                          Ethernet MAC

 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|0|dest (0x3055) | src (0x3DF0) |
+-+--------------+--------------+
            CAN identifier
~~~
{: #fig-unicast-mac-translation title="Example address translation from Ethernet MAC to CAN identifier."}

{{fig-multicast-mac-translation}} shows a translation from a multicast Ethernet MAC to CAN identifier.
The source MAC address is carried in-line in the CAN frame data.

~~~
|0                            4|4                             9|
|0                            7|8                             5|
+------------------------------+-------------------------------+
| dest MAC (33:33:00:01:00:01) |  src MAC (02:00:5E:10:00:FF)  |
+------------------------------+-------------------------------+
                          Ethernet MAC

 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|1|dest (0x0001) | src (0x3DF0) |
+-+--------------+--------------+
            CAN identifier
~~~
{: #fig-multicast-mac-translation title="Example address translation from Ethernet to CAN for multicast frames"}

{{fig-unicast-id-translation}} shows a translation CAN identifier to Ethernet MAC.
The destination (dest) MAC address is carried inline in the CAN frame data.
The 34-bit MAC address prefix for this example is 02:00:5E:10 followed by two zero bits,
written as 02:00:5E:10:00/34.

~~~
 0|0            1|1            2|
 0|1            4|5            8|
+-+--------------+--------------+
|0|dest (0x3DF0) | src (0x3055) |
+-+--------------+--------------+
            CAN identifier
|0                            4|4                             9|
|0                            7|8                             5|
+------------------------------+-------------------------------+
| dest MAC (02:00:5E:10:00:FF) |  src MAC (02:00:5E:10:30:55)  |
+------------------------------+-------------------------------+
                          Ethernet MAC
~~~
{: #fig-unicast-id-translation title="Example address translation from CAN identifier to Ethernet MAC for unicast frames"}

~~~
                                          +-----+  +-----+
                                          |CAN  |  |CAN  |  ...
                                          |node |  |node |
                                          +--+--+  +--+--+
+--------+---+     +---+----------+---+      |        |
|        |   |     |   |Ethernet  |   |      |        |
| Switch |ETH|<--->|ETH|Border    |CAN|------+--------+---- ...
|        |   |     |   |Translator|   |
+--------+---+     +---+----------+---+
~~~
{: #fig-translator-setup title="Example setup with Ethernet Border Translator"}


# IANA Considerations {#iana-considerations}

The IANA considerations for this document have not yet been determined.
A future revision will specify the registry actions required by this
document, or state that this document has no IANA actions.


# Security Considerations {#security-considerations}

This document doesn't provide any security mechanisms.
Traffic on the bus can be intercepted, spoofed, or destroyed.
For confidentiality and integrity, mechanisms like TLS or IPsec need to be applied.

The link-local IID defined in {{addr-conf-sec}} is derived from the node address and therefore
exposes that address in every link-local IPv6 address of the node. Within a bus segment this
discloses nothing new, because the node address appears in the CAN frame identifier of every
frame the node sends or receives. It would become a concern if the node address were also
used to form addresses that are routable beyond the segment: such an address would remain
stable for as long as the node keeps its node address and would allow the node to be tracked
across networks, which is what {{RFC8064}} and {{RFC7217}} are intended to prevent.
For this reason the node address is used only for link-local IIDs.

A 14-bit node address space holds 16384 values and can be scanned exhaustively in a short
time. The size of the address space therefore provides no protection against discovery of
the nodes present on a bus segment.


--- back

# Acknowledgements
{:unnumbered}

This document is based on {{I-D.wachter-6lo-can}}, an earlier Internet-Draft
authored by Alexander Wachter.
