```
+-----------------------------------------------------------------------------+
|                             The Energy Protocol                             |
| 						                                                      |
|                                   Overview                                  |
+-----------------------------------------------------------------------------+

Published: April 2025
Initiated: February 2023
Status: Public Draft
Revision: 00

Abstract

  This document provides a high-level overview of the Energy Protocol (EP)
  specification, explaining its purpose, design goals, and modular structure.
  
Copyright Notice

  Copyright (c) 2025 Energy Engineering Task Force (EnergyETF)
  and the persons identified as the document authors.
  All rights reserved.

Table of Contents

  1. Introduction
  2. Goals and Rationale
  3. Modular Structure
     3.1. EP Framing and Transport
     3.2. EP Messages
     3.3. EP over Ethernet/TCP
     3.4. EP over BLE/UART
  4. Example Usage
     4.1 Residential Energy Communities (EnergyNet)
     4.2 EV Charge Stations
  5. References
  Acknowledgments
  Contact Information

1. Introduction

  The Energy Protocol (EP) is an open, interoperable framework for
  exchanging energy-related data such as supply capacity, demand needs,
  and storage status. EP fosters near real-time coordination of energy
  flows in both small (e.g., home battery) and large-scale (e.g.,
  interconnected energy community grids, EV charge stations) deployments.

  This specification is split into multiple documents to acheive a more
  digestible and modular structure. The separation of concerns allows for:

  - A separation of data structures from framing and transport details.
  - Flexibility to add e.g. new transport bindings, without changing core
    message definitions.

2. Goals and Rationale

  The overarching purpose of the EP is to offer an alternative to all legacy
  protocols that halter electrification of society for the benefit of
  liberating the world from fossil based energy systems.
    
  The EP is created to be a universal protocol for energy data exchange,
  enabling devices to communicate seamlessly across different manufacturers,
  contexts and technologies. The protocol specification is by design:

  a) Interoperable: Ensure devices from any vendor can exchange EP messages
        to coordinate energy transfers.
  b) Constraint friendly: Supporting embedded systems with limited resources.
  c) Secure: Encryption to protect against eavesdropping or tampering.
  d) Distinct: Use explicit value types and identifiers for message properties.
  e) Extensible: Provide extension possibilities and a mechanism for
        backward compatibility that may survive for an extended time period.
  f) Open: Be open and free to use, without restrictions or costs.

3. Modular Structure

  There are other drafts that separately define different aspects of the
  Energy Protocol, namely messages, framing and transport requirements, and
  specific bindings for different transports.  These is what is included in
  the initial structure:

3.1. EP Framing and Transport
   (draft-eetf-ep-framing-transport) 

   Specifies mandatory TLS 1.3 usage, message serialization and 
   framing. Also outlines general transport guidelines (throughput,
   recovery, safety).

3.2. EP Messages
   (draft-eetf-ep-messages)

   Defines the core EP data model, including fields for supply, demand,
   and storage. Describes all message payload structures in detail.

3.3. EP over Ethernet/TCP
   (draft-eetf-ep-ethernet-tcp)

   A specific binding detailing how connections are established and EP
   messages transmit over TCP, including port usage, etc.

3.4. EP over BLE/UART
   (draft-eetf-ep-ble-uart)

   A specific binding for Bluetooth Low Energy (BLE) UART-like
   connections. Covers fragmentation, how to connect and transport
   EP frames in small BLE packets.

4. Example Usage

  Here are two scenarios that use the same EP messaging protocol for the energy
  transfer coordination between each device. In many cases, there is the need
  for an adapter to support devices that are not yet EP capable. Such adapters
  can be simple devices that translates the EP messages to the legacy protocol
  and vice versa. They can normally be built using a small microcontroller with
  limited processing power.

4.1 Residential Energy Communities

  A residential energy community might consist of several buildings with
  solar panels and energy storage systems. Each building has an energy router
  that connect these local systems to neighbouring buildings and optionally the
  central grid. The buildings can coordinate energy surplus or demand between
  themselves on a neighbourhood energy bus. EP over Ethernet/TCP is chosen as
  the transport since all systems are connected via a wired network.

  This is further described in the [draft-eetf-energynet] document.

4.2 EV Charge Stations

  To transfer energy to (and eventually, from) electric vehicles (EVs), a
  charger can be used either with a plug-in cable or an automatic docking
  system. A group of chargers can be connected into a small energy network
  which unlocks desired funcionality such as operational resilience, peak
  shaving, load balancing by hooking up to energy storage such as battery
  storage and H2-based long-term storage systems. Local energy production
  types such as solar and wind can also be integrated.

  An adapter is used to interface between EP and the EV charging system
  (typically CCS).  For EVs with an automated docking device, the
  communication between the EV and the charger is typically wireless, using
  e.g. EP over BLE/UART.

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
