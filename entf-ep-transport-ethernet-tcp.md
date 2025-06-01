```
+-----------------------------------------------------------------------------+
|                             The Energy Protocol                             |
| 						                                                      |
|                        EP over Ethernet/TCP Binding                         |
+-----------------------------------------------------------------------------+

Initiated: February 2023
Public: May 2025

Abstract

  This document describes how to run the Energy Protocol (EP) over
  Ethernet/TCP.  It references entf-ep-framing-transport for the mandatory TLS
  1.3 encryption and message framing details, and entf-ep-messages for the EP
  data model.

Table of Contents

  1. Introduction
  2. Connection Setup
  3. Message Framing on TCP
  4. Example Session
  Acknowledgments
  Contact Information

1. Introduction

  The Energy Protocol (EP) can run over any reliable byte-stream transport.
  Ethernet/TCP is among the most common for IP-based networks.  This document
  covers how devices establish TCP connections for EP.

2. Connection Setup

  Devices are required to maintain connections to their upstream devices, all
  the way to the energy router device that manages the coordination of energy
  in the local EnergyNet.  For simple systems with only a few devices that
  do not have to coordinate more than the transfers between each device, no
  router is required and the devices can make direct connections in
  device-to-device mode.

  The default listening port for the Energy Protocol is 0xDCDC (56540 in
  decimal).  It is mandatory for devices in the EnergyNet to listen on this
  port and to accept connections from other devices.

  Downstream devices are typically initiating the connection.  When two or
  more energy routers are connecting, typically one is configured as a
  server and the other(s) as client.

  Peer-to-peer connections are also possible, where both devices are
  configured as both servers and clients.  When connections are established,
  the session messages are exchanged to ensure there is not already any
  other connection made.

  It is of course important that devices that connect via EP also have a
  matching physical connection, typically a DC cable that can handle the
  capacity configured in the device, to conduct the actual energy exchange.

3. Message Framing on TCP

  TCP includes framing and reassembly of messages, so EP messages can be
  sent as-is according to the specification in entf-ep-framing-transport.
  No additional framing needs to be applied.

4. Example Session

  1. Device A connects to Device B on port 0xDCDC.
  2. Device B accepts the connection and presents its certificate.
  3. TLS handshake completes by Device A, offering its own certificate.
  4. Device A and B exchanges EP messages and energy accordingly.
  5. Connection is closed on error or when device shuts down.

Acknowledgments

  The EnergyNet Task Force wants to extend its heartfelt thanks to everyone
  who is playing a part in making this protocol a success. Your contributions
  — whether large or small — are truly appreciated.

Contact Information

EnergyNet Task Force
Email: [info@energynettaskforce.org]

+-----------------------------------------------------------------------------+
|                                                                             |
|                    Join the free & open energy movement!                    |
|                                                                             |
+-----------------------------------------------------------------------------+
