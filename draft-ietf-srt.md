---
title: The SRT Protocol
abbrev: SRT
docname: draft-ietf-sharabayko-srt-latest
category: std

ipr: trust200902
area: opsarea
workgroup: mops
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: "M.P. Sharabayko"
    name: "Maxim Sharabayko"
    organization: "Haivision"
    email: maxsharabayko@haivision.com

normative:
  RFC2119:

informative:
  RFC8174:


--- abstract

TODO Abstract

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Packet Structure

SRT packets are transmitted in UDP packets.

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
|            SrcPort            |            DstPort            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ UDP Header
|              Len              |            ChkSum             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
|                                                               |
+                          SRT Packet                           +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

The structure of the SRT packet is shown in Figure 2.

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|F|        (Field meaning depends on the packet type)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          (Field meaning depends on the packet type)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Destination Socket ID                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+    (Packet contents: depends on the packet type)              +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #srtpacket title="SRT packet structure"}

F (1 bit): 
: Packet Type Flag. The control packet must set this flag as "1".
  The data packet must set this flag as "0".

Timestamp (32 bits):
: The time stamp of the packet in microseconds.
  The value is relative to the time the SRT connection was established.
  This is relative time value starting from when the connection is established.
  Depending on the transmission mode, the field stores the packet sent time or
  the packet origin time.

Destination Socket ID (32 bits):
: A fixed-width field providing the socket ID to which a packet should be dispatched.
  The field may have the special value "0" when the packet is a connection request.


## Data Packets

The structure of the SRT data packet is the following.

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|                    Packet Sequence Number                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| PP|O|KK |R|                   Message Number                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Destination Socket ID                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                              Data                             +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #srtdatapacket title="data packet structure"}

Packet Sequence Number (31 bits):
: The sequential number of the data packet.

PP (2 bits):
: Packet Position. This field indicates the position of data packet in the message.
  The value "10b" means the first packet of the message, "00b" - a packet in the middle,
  "01b" is the last packet. If a single data packet forms the whole message,
  the value is "11b".

O (1 bit):
: Order Flag. Indicates whether the message should be delivered by the receiver
  in order (1) or not (0).
  Note. In file transfer mode this a message with O=0 that is sent later 
  (but reassembled before an earlier message which may be incomplete due to packet loss)
  is allowed to be delivered immediately, without waiting for the earlier message to be completed.
  In Live Transmission Mode the only valid value is "1".

KK (2 bits):
: Encryption Flag. The flag bits indicate whether or not data is encrypted.
  The value "00b" means data is not encrypted, "01b" - data is encrypted with even key, 
  "11b" - data is encrypted with odd key.

R (1 bit):
: Retransmitted Packet Flag. This flag is clear when a packet is transmitted the very first time.
  The flag set to "1" means the packet is retransmitted.

Message Number (26 bits):
: The sequential number of the message formed by consecutive data packets (see PP field).

Data (variable length):
: The payload of the data packet. The length of the data is the remaining length of the UDP packet.


## Control Packets

SRT control packet has the following structure.

~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|1|         Control Type        |            Subtype            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   Type-specific Information                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Destination Socket ID                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                   Control Information Field                   +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #controlpacket title="control packet structure"}

Type (15 bits):
: Control Packet Type. The use of these bits is determined
  by the control packet type definition.

  - "0": Handshake
  - "1": Keep Alive
  - "2": ACK
  - "3": NAK (Loss Report)
  - "4": Congestion Warning
  - "5": Shutdown
  - "6": ACKACK
  - "7": Drop Request
  - "8": Peer Error

Subtype (16 bits):
: This field specifies additional subtype of specific packets.

Type-specific Information (32 bits):
: The use of this field is defined by the particular control
  packet type. Handshake packets don't use this field.

Control Information Field (variable length):
: The use of this field is defined by the Type field of the control packet.


The types of SRT control packets are shown in Table 1.

| ----------------- | ------- | ------- | -------------------------- |
| Packet Type       | Type    | Subtype | Section                    |
| ----------------- | :-----: | :-----: | -------------------------- |
| HANDSHAKE         |  0x0000 |   0x0   | N/A                        |
| KEEPALIVE         |  0x0001 |   0x0   | N/A                        |
| ACK               |  0x0002 |   0x0   | N/A                        |
| NAK (Loss Report) |  0x0003 |   0x0   | N/A                        |
| Congestion Warning|  0x0004 |   0x0   | N/A                        |
| SHUTDOWN          |  0x0005 |   0x0   | N/A                        |
| ACKACK            |  0x0006 |   0x0   | N/A                        |
| DROPREQ           |  0x0007 |   0x0   | N/A                        |
| PEER_ERROR        |  0x0008 |   0x0   | N/A                        |
| Extension         |  0x7FFF |    -    | N/A                        |
| SRT_HSREQ         |  0x7FFF |   0x1   | N/A                        |
| SRT_HSRSP         |  0x7FFF |   0x2   | N/A                        |
| SRT_KMREQ         |  0x7FFF |   0x3   | N/A                        |
| SRT_KMRSP         |  0x7FFF |   0x4   | N/A                        |
| ----------------- | ------- | ------- | -------------------------- |
{: #srt-ctrl-pkt-type-table title="SRT Control Packet Types"}


### Handshake



### Keep Alive



### ACK (Acknowledgement)



### NAK (Loss Report)



### Congestion Warning

### Shutdown



### ACKACK


### Drop Request

### Peer Error


# SRT Data Transmission and Control

TODO: Priority 2.

## SRT Buffer Latency

## Timestamp Based Packet Delivery

## Packet Acknowledgement (ACKs)

## Packet Retransmission (NAKs)

## Packet Acknowledgment in SRT

## Bidirectional Transmission Queues

## ACKs, ACKACKs & Round Trip Time

## Loss List


# Encryption

TODO: Priority 3.

## Overview

## Definitions

## Encryption Process Walkthrough

## Messages

## Parameters

## Security Issues

## Implementation Notes


# Security Considerations

TODO Security

# IANA Considerations

TODO IANA

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

--- back

# Packet Sequence List coding {#packet-seq-list-coding}

