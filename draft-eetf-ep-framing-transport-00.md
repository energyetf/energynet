```
+-----------------------------------------------------------------------------+
|                             The Energy Protocol                             |
| 						                                                      |
|                            Framing and Transport                            |
+-----------------------------------------------------------------------------+

Published: April 2025
Initiated: February 2023
Status: Public Draft
Revision: 00

Abstract

  This document includes two closely related aspects of the Energy
  Protocol (EP): framing and transport. By unifying these two topics, EP
  provides a single reference for how to reliably carry its messages over
  multiple underlying mediums, with mandatory encryption and clear message
  serialization. Specific bindings (e.g., Ethernet/TCP, BLE-UART) are defined
  in separate documents.

Copyright Notice

  Copyright (c) 2025 Energy Engineering Task Force (EnergyETF)
  and the persons identified as the document authors.
  All rights reserved.

Table of Contents

  1. Introduction
  2. Mandatory Encryption
  3. Serialization
  4. Message Framing
     4.1. Example of a Framed EP Message
  5. General Transport Requirements
     5.1. Throughput and Frequency
     5.2. Secure Channel
  6. Potential Transports
  Appendix A. Acknowledgments
  Authors' Addresses

1. Introduction

  The Energy Protocol (EP) is a multi-layered system for exchanging
  energy-related data such as supply capability, demand, or storage
  state. A device implementing EP publishes or subscribes to these
  parameters at a recommended rate of at least 10 messages per second.

  This document combines two fundamental aspects:
  - Framing: Defining how EP messages are delimited and serialized
    within a data stream.
  - Transport: Explaining minimal requirements (e.g., bandwidth,
    encryption) and how EP can sit atop various links (TCP, BLE, etc.).

  The core message definitions (fields for supply, demand, storage)
  are defined in [draft-eetf-ep-messages]. Specific transport bindings
  (Ethernet/TCP, BLE-UART, etc.) appear in separate drafts.

2. Mandatory Encryption

  EP implementations MUST use TLS 1.3 according to RFC8446 and BCP195 to
  protect data streams. This ensures confidentiality and integrity,
  preventing unauthorized eavesdropping or message tampering. Fallback to
  TLS 1.2 or older protocols is not permitted, and not required for backward
  compatibility since this is a new protocol specification.

  It is optional for devices to validate certificates of their peers, but
  it's recommended to establish either a shared authority or at minimum a
  local trusted device list. A distributed PKI to manage device certificates
  is not in the scope for this document, instead it will be described
  separately. Devices could also perform revocation checks on remote
  certificates if feasible, but not mandatory.

3. Serialization

  To support high frequency transmission and quick de/serialization
  even under constrained circumstances such as limited processing
  power, EP message payloads are serialized using CBOR (Concise Binary
  Object Representation) as defined in IETF RFC 8949.  CBOR is a binary
  encoding of JSON-like data structures, with a compact representation
  and a flexible and clear schema.  Devices may choose to specify frame
  headers to implement additional serialization of payloads.  There is
  however no registry of frame headers at the moment.

4. Message Framing

  Since most transports are stream-based, EP messages must be easy to
  transmit with a clear format and possible to defragment. The approach for
  the Energy Protocol is to use a CBOR array, containing a header map,
  a message type id and the payload length, followed by the payload data.

  The frame start is an unconventional CBOR 4-byte size array of size 4.
  This is done to introduce a "magic sequence" that an implementation could
  search for in a recovery attempt scenario (in case of e.g. missing bytes).

  The frame header map is a key:value pair with a unsigned integer as key.
  Keys with ID 0x00 to 0xFF are reserved.  If no headers are included, null
  (0xF6) should be supplied rather than an empty map.

  The message type is a 4-byte unsigned integer, which is used to specify the
  type ID of the message being sent. This is mandatory. ID 0x00000000 to
  0x00ffffff are reserved.

  The payload length specifies how many consecutive bytes to read before
  attempting deserialization.

  For detailed description of the core messages, see [draft-eetf-ep-messages].

4.1. Example of a Framed EP Message

  A single message may appear as:

    9A 00 00 00 04                      EP Frame start (array(4))
    F6                                  Empty header (null)
    1A DC DC F0 0D                      Message type (uint32)
    18 98                               Payload length 152 (byte)
    A8 .. .. .. ..                      Payload data (CBOR map(8))

  After each message, the stream should restart with a 9A 00 00 00 04 byte
  sequence, indicating start of the next frame.  Anything else means corrupt
  or malformed data has arrived and if recovery cannot be performed, the
  connection should be dropped and renegotiated.

5. General Transport Requirements

5.1. Throughput and Frequency

  EP mandates the transmission of energy coordination messages between
  1 and 100 Hz on a dedicated isolated data connection. The underlying
  transport must provide sufficient bandwidth and low latency for that
  rate. The size of the core EP messages are in the sub-kilobyte range.
  
  If there is an ongoing energy transfer and there are no messages coming
  through at a frequency of at least 5 Hz (every 200ms), the connection should
  be considered lost or unstable, dropped and renegotiated instantly, along
  with a controlled shutdown of the energy transfer.  If there is no ongoing
  energy transfer, a device can choose to wait for a longer time before
  renegotiating the connection, typically a few seconds.

  All messages relevant to the device type (e.g. supply, demand, storage)
  must be transmitted constantly at the minimum frequency, regardless of any
  change in the parameters.

5.2. Secure Channel

  If the device is an adapter for another protocol, it is important to
  ensure the other protocol - if possible - also uses encryption with a
  security level as close to or higher than TLS 1.3.  This is however
  outside the scope of EP but worth considering in the design of the adapter.

6. Potential Transports

  Commonly used transports in energy devices may include:
   - TCP (see [draft-eetf-transport-ethernet-tcp])
   - BLE or similar wireless links (see [draft-eetf-transport-ble-uart])
   - MQTT or other broker-based messaging architectures
   - CAN-bus or other field-bus, possibly with fragmentation and a
     secure gateway

Acknowledgments

  The EnergyETF EP working group want to extend its heartfelt thanks
  to everyone who is playing a part in making this protocol a
  success. Your contributions — whether large or small — are
  truly appreciated.

Contact Information

EnergyETF EP Working Group
Email: [ep@energyetf.org]

+-----------------------------------------------------------------------------+
|                                                                             |
|                    Join the free & open energy movement!                    |
|                                                                             |
+-----------------------------------------------------------------------------+
