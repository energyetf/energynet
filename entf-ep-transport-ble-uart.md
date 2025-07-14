```
+-----------------------------------------------------------------------------+
|                             The Energy Protocol                             |
|                                                                             |
|                          EP over BLE/UART Binding                           |
+-----------------------------------------------------------------------------+

Initiated: February 2023
Public: May 2025

Abstract

  This document describes how the Energy Protocol (EP) can be carried
  over a Bluetooth Low Energy (BLE) UART-like service. It references
  entf-ep-framing-transport for message framing and TLS 1.3 details,
  and entf-ep-protocol for the EP data model.

Table of Contents

  1. Introduction
  2. BLE/UART Overview
  3. Security and TLS
  4. Framing Over BLE/UART
  5. Example Session
  6. References
  Acknowledgments
  Contact Information

1. Introduction

  The Energy Protocol (EP) is transport-agnostic.  This document explains
  how EP frames are exchanged over BLE using a UART-like service. Since BLE
  has data size limitations, messages may be fragmented, then reassembled
  at the receiving side.

2. BLE/UART Overview

  Typically, a BLE-UART service uses two GATT characteristics: one for
  transmitting data, another for receiving data. Devices can use BLE
  notifications or indications to push data in near-real time.

  One well-established such service is the Nordic UART Service (NUS),
  which is widely supported by BLE modules and stacks.  This definition is
  what is selected for this transport binding.

  The upstream device should activate a BLE beacon when it is ready to
  accept connections. The downstream device scans for beacons,
  then connects to the upstream device when in reasonable range.  The
  beacon data typically includes the device mac address and uses the EP
  specific manufacturer ID 0xDCDC. After a device is connected, the
  upstream device should deactivate the beacon to indicate it is no
  longer free for connections (unless it is capable to connect to and
  transfer energy with multiple devices).

  Typically, the upstream device is a stationary system and the downstream
  system a mobile device, such as an electric vehicle or other portable
  storage device, which may have an automated electric coupling mechanism.

3. Security and TLS

  A TLS handshake can be negotiated since the BLE/UART service works
  as a serial port, streaming data asynchronously over an established
  connection.  See entf-ep-framing-transport for more details.

4. Framing Over BLE/UART

  Because BLE packets may arrive fragmented it is up to the receiver
  to reassemble them and handle the full message once all fragments arrive.

5. Example Session
  The following is an example of an EP over BLE/UART session.

  1. The downstream device scans for BLE beacons that are within range.
  2. The downstream device connects to an upstream device via its BT stack.
  3. The devices perform TLS handshake (possibly validating certificates).
  4. Transmission of initial EP messages.
  5. Possible automatic electric coupling is performed.
  6. Transmission of EP messages for energy exchange.
  7. Devices disconnect when the session is over.

Acknowledgments

  The EnergyNet Task Force wants to extend its heartfelt thanks to everyone
  who is playing a part in making this protocol a success. Your contributions
  - whether large or small - are truly appreciated.

Contact Information

EnergyNet Task Force
Email: [info@energynettaskforce.org]

+-----------------------------------------------------------------------------+
|                                                                             |
|                    Join the free & open energy movement!                    |
|                                                                             |
+-----------------------------------------------------------------------------+
