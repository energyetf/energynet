```
+-----------------------------------------------------------------------------+
|                             The Energy Protocol                             |
| 						                                                      |
|                            Data Model & Messages                            |
+-----------------------------------------------------------------------------+

Published: April 2025
Initiated: February 2023
Status: Public Draft
Revision: 00

Abstract

  This document describes the Energy Protocol (EP) data model, including
  message structures for energy supply, demand, and storage parameters.
  It also standardizes how devices express values like power, voltage,
  current, energy, price, etc in a value type explicit manner.

Copyright Notice

  Copyright (c) 2025 Energy Engineering Task Force (EnergyETF)
  and the persons identified as the document authors.
  All rights reserved.

Table of Contents

  1. Introduction
  2. Terminology
  3. Data Model
     3.1. Value Type Explicit Fields
     3.1.1. Bounds
     3.1.2. Price
     3.1.3. Source Mix
     3.1.4. Energy Mix
     3.1.5. Isolation
  4. Messages
     4.1. Ping
     4.1.1. Ping binary example
     4.2. Session Parameters
     4.2.1. Session Parameters binary example
     4.3. Soft Disconnect
     4.3.1. Soft Disconnect binary example
     4.4. Supply Parameters
     4.4.1. Supply Parameters binary example
     4.5. Demand Parameters
     4.5.1. Demand Parameters binary example
     4.6. Storage Parameters
     4.6.1. Storage Parameters binary example
  5. Extensibility
  6. Security Considerations
  7. References
  Appendix A. Acknowledgments
  Authors' Addresses

1. Introduction

  The Energy Protocol (EP) enables devices to share structured information
  about energy supply, demand, and storage. This document focuses on the
  messages and their content: the message type identifiers, what fields
  exist and how values are specified.

  The messages specified in this document are mostly specific to 
  electric energy.

  Some examples are expressed in corresponding JSON notation for readability.

2. Terminology

  - Supply: A device’s capability to provide energy or power.
  - Demand: A device’s requirements for energy or power.
  - Storage: A device’s buffering capability (e.g., battery).

3. Data Model

3.1. Value Type Explicit Fields

  EP uses contractually defined value types in the message definitions to
  avoid misunderstandings, whereas the actual binary data representation
  can differ (as long as the data type can represent the value type).
  
  Properties thus appear in explicitly typed subfields, for instance
  maxpower = { "watts": 3501.4 } rather than, as sometimes used, property
  naming: { "maxpower_w": 3501.4 }.
  
  This is a table of the value types in use and which group of CBOR data
  type is expected.

  Value type    ID      Data type       Description
  
  Text          0x00    text            Text (UTF-8 encoded)
  Flag          0x01    bool            Boolean flag (true/false)
  Amount        0x02    numeric         Numeric amount (any number)
  Timestamp     0x03    text            Date, time and TZ (RFC 7049)
  Binary        0x04    binary          Raw binary data (byte array)
  Currency      0x05    text            Currency code (ISO 4217)
  Duration      0x06    numeric         Duration (milliseconds)

  Voltage       0x10    numeric         Voltage (Volts)
  Current       0x11    numeric         Current (Amperes)
  Power         0x12    numeric         Power (Watts)
  Energy        0x13    numeric         Energy (Watt-hours)
  Percentage    0x14    numeric         Percentage value (Percent)
  Resistance    0x15    numeric         Resistance (Ohms)

  The value type specific values are structured as a map with the
  type ID as key and the value as the data.  For example, a voltage of
  824V can freely be represented as either a float { 0x10: 824.0 }
  or a ushort { 0x10: 824 }.  The receiver must be able to handle
  both data type representations, as mandated by CBOR.

  Arrays and key-value pairs are used to represent composite data
  structures.  In the protocol there are currently a few defined complex
  value types, as follows in the subchapters.

  Example binaries:
  A1 00 6B 31 32 33 34 35 36 37 38 39 61 62     "0123456789ab"
  A1 01 F4                                      false
  A1 02 F9 52 00                                48.0
  A1 03 6A 32 30 32 35 2D 30 32 2D 30 31        "2025-02-01"
  A1 04 44 01 02 03 04                          0x01 0x02 0x03 0x04
  A1 05 63 53 45 4B                             "SEK"
  A1 06 19 03 E8                                1000

  A1 10 19 02 E8                                744V
  A1 11 F9 5C 92                                292.5A
  A1 12 FA 47 43 50 00                          50000.0W
  A1 13 1A 00 BD 83 BD                          12420029Wh
  A1 14 FB 40 58 FF FE 5C 91 D1 4E              99.9999%

3.1.1. Bounds (ID 0x20)
  The Bounds value type is a structured array of two values (minimum
  and maximum).  They are required to have the same value type.

  Example binary: (200V minimum, 1000V maximum)
  A1 18 20                                  Bounds
  82                                        Array of two values
  A1 10 18 C8                               Minimum value: 200V
  A1 10 19 03 E8                            Maximum value: 1000V  
  
3.1.2. Price Forecast (ID 0x30)
  The price forecast type is defined as an array of price information arrays,
  with a timestamp (when the price starts), an amount and a currency. Each
  individual price is valid until obsoleted by another value that has a more
  recent timestamp.

  This structure is typically used in the context of energy pricing.
  
  Pos  Name            Value type          Description
  0    timestamp       Timestamp           Time from when it is valid
  1    amount          Amount              Price amount
  2    currency        Currency            Currency code (ISO 4217)

  Example binary: (42.50 SEK, valid from 2025-02-01 12:00)
  A1 18 30                                  Price Forecast
  81                                        Array with one price
  83                                        Price structure (array(3))
     A1 03 70 [..long text..]               Timestamp ("2025-02-01 12:00")
     A1 02 F9 51 50                         Amount (42.50)
     A1 05 63 53 45 4B                      Currency ("SEK")

3.1.3. Source Mix (ID 0x40)
  This value type describes a power generation mix.  It is defined
  as an array of source identifiers and shares in percentage.  The
  total share sum must be 100%.

  The source identifiers are currently defined as follows:

    ID    Source        Typical carbon intensity (CO2eq/kWh)
    0x00  Unknown       1000g
    0x01  Wind          35g
    0x02  Solar         50g
    0x03  Hydro         50g
    0x04  Nuclear       50g
    0x05  Gas           500g
    0x06  Oil           800g
    0x07  Coal          1000g
    0x08  LocalWind     30g
    0x09  LocalSolar    40g

  Example binary:
  A1 18 40                                  Source mix
  82                                        Two power sources
     A1 01 A1 14 18 46                      Wind: 70%
     A1 09 A1 14 18 1E                      Local Solar: 30%

3.1.4. Energy Mix (ID 0x41)
  This value type is similar to the source mix, but describes the
  mix in terms of energy amount per source instead of share per source.

  It is represented by an array of attributed energy entries.

  Example binary:
  A1 18 41                                  Energy mix
  83                                        Three energy sources
     A1 01 A1 13 19 18 38                   Wind: 6.2kWh
     A1 03 A1 13 19 CB 20                   Hydro: 52kWh
     A1 09 A1 13 19 09 56                   Local Solar: 2390Wh

3.1.5. Isolation (ID 0x50)
  This is a value type that describes the electric isolation of
  a device.  It is typically provided by an IMD (Isolation monitoring
  device).  The value is defined as an array with a number value
  representing the isolation state and the resistance between PE and
  the poles.
  
  Isolation states      Code
  Unknown               0x00
  Ok                    0x01
  Warning               0x02
  Fault                 0x03

  Example binary:
  A1 18 50                                  Isolation
  83                                        Array with three values
     01                                     Isolation state: Ok
     A1 15 19 03 E8                         Negative resistance: 1kOhms
     A1 15 19 03 E8                         Positive resistance: 1kOhms

4. Messages

  There are three main messages in the energy protocol: Supply, Demand,
  and Storage. A device may publish any combination of these depending
  on its capabilities.  Apart from this there are three messages that 
  relate to the connection management, Ping, Session and Disconnect.
  
  Message types follow, including the message type and payload
  definitions.
  
4.1. Ping (0xffffffff)
  This message type is used for keep-alive purposes, with no payload.

4.1.1. Ping binary example
  Header:
    9A 00 00 00 04                                  EP Frame start
    F6                         		                No extra headers
    1A FF FF FF FF                                  Message type
    01                                              Payload length 1 byte

  Payload:
    F6                                              Payload data (null)

4.2. Session Parameters (0xbaba5e55)
  This message type is used to define a device session.  It includes the
  device's identity, device type, version, a human-readable name, tenant,
  service provider and an ephemeral session id that can be used for
  correlation across systems.

  This should be considered the device handshake, and before any other
  messages are sent, this should be transmitted from both the devices
  acting as server and client.  If both devices are connected to an
  access provider service, the ephemeral session id can be used as the
  access token that the devices can use to validate the other side with
  the service.

  When receiving this message, identity, tenant and service provider id
  should be validated against the certificate used to establish the
  connection.

  Payload map definition:

  ID    Name            Value type          Description
  0x00  identity        Text                Device identity
  0x01  type            Text                Device type
  0x02  version         Text                Device version <major>.<minor>
  0x03  name            Text                Device name (human readable)
  0x04  tenant          Text                Device tenant
  0x05  provider        Text                Tenant's service provider
  0x06  session         Text                Ephemeral session id

4.2.1. Session Parameters binary example
  Header:
    9A 00 00 00 04                                  EP Frame start
    F6                         		                No extra headers
    1A BA BA 5E 55                                  Message type
    18 58                                           Payload length 88 bytes

  Payload:
    A7                                              Session (7 props)
    00 A1 00                                        Device Id (Text)
    6C 30 31 32 33 34 35 36 37 38 39 61 62          "0123456789ab"
    01 A1 00                                        Device Type (Text)
    67 76 65 68 69 63 6C 65                         "vehicle"
    02 A1 00                                        Device Version (Text)
    63 31 2E 30                                     "1.0"
    03 A1 00                                        Name (Text)
    6C 54 65 73 74 20 56 65 68 69 63 6C 65          "Test Vehicle"
    04 A1 00                                        Tenant (Text)
    64 64 65 6D 6F                                  "demo"
    05 A1 00                                        Service Provider (Text)
    64 64 65 6D 6F                                  "demo"
    06 A1 00                                        Type 00 = Text
    78 24 64 30 36 30 64 65 64 62 2D 36 61 30 [..]  "d060dedb-6a0[..]"

4.3. Soft Disconnect (0xbabadead)
  This message type is used to inform before a device performs an
  intentional disconnect with the other device.  It will indicate a
  reason why the disconnection is being made and a flag whether a
  reconnection attempt should be made (if the remote device is the
  one initiating the connection), for instance if there is a software
  update that requires a restart, reconnection should be enabled.  The
  reason can be omitted using a null payload.

  Payload map definition:

  ID    Name            Value type          Description
  0x00  reconnect       Flag                Reconnect flag
  0x01  reason          Text                Reason for disconnect

4.3.1. Soft Disconnect binary example
  Header:
    9A 00 00 00 04                                  EP Frame start
    F6                         		                No extra headers
    1A BA BA DE AD                                  Message type
    0F                                              Payload length 15 bytes

  Payload:
    A2                                              Disconnect (2 props)
    00 A1 01                                        Reconnect (Flag)
    F4                                              false
    01 A1 00                                        Reason (Text)
    66 6E 6F 72 6D 61 6C                            "normal"

4.4 Supply Parameters (0xdcdcf00d)
  Message used by a device that provides energy.

  Payload map definition:

  ID    Name            Value type          Description
  0x00  voltage-limit   Bounds(Voltage)     Voltage limit (min-max)
  0x01  current-limit   Bounds(Current)     Current limit (min-max)
  0x02  power-limit     Power               Power limit (max)
  0x03  power-mix       SourceMix           Power source mix
  0x04  energy-prices   Array(Price)        Energy price forecast
  0x05  voltage         Voltage             Current measured voltage
  0x06  current         Current             Current measured current
  0x07  isolation       IsolationState      Isolation status of device

4.4.1. Supply Parameters binary example
  Header:
    9A 00 00 00 04                                  EP Frame start
    F6                         		                No extra headers
    1A DC DC F0 0D                                  Message type
    18 96                                           Payload length 150 bytes (TBD)

  Payload:
    A8                                              Supply (8 props)
    00 A1 18 20 82                                  Voltage Limits (Bounds)
       A1 10 F9 5B 54                               Minimum: 234.5V
       A1 10 F9 5B C8                               Maximum: 249.0V
    01 A1 18 20 82                                  Current Limits (Bounds)
       A1 11 F9 55 90                               Minimum: 89.0A
       A1 11 F9 58 CC                               Maximum: 153.5A
    02 A1 12                                        Power Limit (Power)
       FA 47 43 50 00                               50.0kW
    ... (TBD)

4.5 Demand Parameters (0xdcdcfeed)
  Message used by a device that wants to make energy transfer demands.
  Current can also be negative, meaning the device requests energy to
  be shed from it (reverse demand).

  Payload map definition:

  ID    Name            Value type          Description
  0x00  voltage         Voltage             Voltage requirement
  0x01  current         Current             Current requirement
  0x02  voltage-limit   Voltage             Max voltage capability
  0x03  current-limit   Current             Max current capability
  0x04  power-limit     Power               Max power capability
  0x05  duration        Duration            Estimated remaining time

4.5.1. Demand Parameters binary example
  Header:
    9A 00 00 00 04                                  EP Frame start
    F6                         		                No extra headers
    1A DC DC FE ED                                  Message type
    18 70                                           Payload length 112 bytes (TBD)

  Payload:
    A6                                              Demand (6 props)
    ... (TBD)

4.6 Storage Parameters (0xdcdcba77)
  Message used by a device with a battery or reservoir.

  Payload map definition:

  ID    Name            Value type          Description
  0x00  soc             Percentage          Current state of charge
  0x01  soc-target      Percentage          Target state of charge
  0x02  soc-target-time Duration            Time left to reach target
  0x03  capacity        Energy              Total capacity of storage
  0x04  energy-mix      EnergyMix           Currently stored energy mix

4.6.1. Storage Parameters binary example
  Header:
    9A 00 00 00 04                                  EP Frame start
    F6                         		                No extra headers
    1A DC DC FE ED                                  Message type
    18 70                                           Payload length 112 bytes (TBD)

  Payload:
    A5                                              Storage (5 props)
    ... (TBD)

5. Extensibility
  Future versions or extensions may add:

  - New messages, e.g messages to support thermal energy transfers.
  - More data, e.g adding time based pricing in the supply message.
  - New value types, e.g. to support new data types or units.
  - New extra headers in the framing, e.g. to add other serialization options.
  - EP Versioning (use a version property in the extra header map).

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
