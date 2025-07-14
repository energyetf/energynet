```
+-----------------------------------------------------------------------------+
|                                The EnergyNet                                |
|                                                                             |
|                                  Overview                                   |
+-----------------------------------------------------------------------------+

Initiated: February 2023
Public: April 2025
Revised: May 2025

Abstract

  This document provides a high-level overview of EnergyNet, a decentralized
  network system based on interconnected energy routers that manage devices
  supporting the Energy Protocol (EP).
  
Table of Contents

  1. Introduction
  2. Energy Routing
     2.1. System architecture
     2.2. Functional logic
     2.3. Network architecture
     2.4. Network policy levels
  Acknowledgments
  Contact Information

1. Introduction
  EnergyNet is a decentralized energy network system based on
  interconnected energy routers that manage devices supporting the Energy
  Protocol (EP). The goal of EnergyNet is to enable efficient and
  flexible energy routing between devices, allowing for easy integration of
  e.g. renewable energy sources, energy storage systems, and electric
  vehicles. With a flexible and resilient network architecture, EnergyNet
  can quickly adapt to the changing energy landscape, enabling the integration
  of new devices and technologies as they become available.
  
  This document provides an overview of the EnergyNet system, including its
  architecture, functional logic, network architecture, and policy levels. It
  is intended for developers, engineers, and researchers interested in
  understanding the EnergyNet system and its potential applications in the
  emerging open and free energy society.
  
  The EnergyNet Task Force welcomes feedback and contributions from the
  community to help shape the future of the EnergyNet system and its
  applications. We are also committed to ensuring that the EnergyNet system
  is open and accessible to everyone, and that it is developed in a
  transparent and collaborative manner.

2. Energy Routing
  To establish a decentralized energy network, EnergyNet utilizes a type
  of device called an energy router (ER).  The router is responsible for
  coordinating energy transfers between connected devices, as well as the
  communication with other energy routers in the network.  It also acts
  as a local management server, gathering metrics for all the connected
  devices and reporting it to a management system (typically a cloud
  service).  To remote routers, the router itself can be seen as a device
  that represents an entire local energy network (LEN).  
  
2.1. System architecture
  Each router has a number of ports, which (via power converters) can
  connect to e.g. a solar PV array, a battery storage system (BESS), an
  AC utility grid or another energy router.  Each port can support AC or
  DC energy links, be uni- or bi-directional, support different voltages
  simultaneously and is galvanically isolated from the other ports.

  Internally, the ports are connected to a DC bus backplane, which is where
  balancing of energy transfers takes place, matching supply and demand
  parameters of the connected devices with policies applied on top.
  
  Here is an example of a typical router with four ports and the two
  blocks of software that manages energy routing and the connected devices:

              |  ^       |  ^
              v  |       v  |
          +----------+----------+----------+ +------------+
          |          |          |          | |            |
          |  Port 1  |  Port 2  |  ..  ..  | |  EP Ports  |
          |          |          |          | | Controller |
          +----------+----------+----------+ |            |
          |                                | +------------+
          |           Backplane            |
          |                                | +------------+
          +----------+----------+----------+ |   Energy   |
          |          |          |          | |   Router   |
          |  Port 3  |  Port 4  |  ..  ..  | | Operating  |
          |          |          |          | |   System   |
          +----------+----------+----------+ +------------+
              ^  |       ^  |
              |  v	     |  v

  The EP Ports Controller is responsible for managing the energy transfers
  between the ports and the connected devices via the backplane and the
  power converters. It is responsible for the encrypted EP message
  communications. It also manages the connections with the remote routers
  outside of the network, including validating certificates.  It runs on top
  of the Energy Router Operating System (EROS), which is responsible
  for applying policies and monitoring the system, connecting to the cloud
  services used for device management (configuration, PKI, OTA, etc), live
  energy transfer reporting and access validation.

  As an example, the router may be in this state during runtime:

              | 420V   540V ^
              v  30A    10A |
          +----------+----------+----------+ +------------+
          |   (DC)   |   (DC)   |          | |            |
          |  Port 1  |  Port 2  |  ..  ..  | |  EP Ports  |
          |   (DC)   |   (DC)   |          | | Controller |
          +----------+----------+----------+ |            |
          |                                | +------------+
          |           Backplane            |
          |                                | +------------+
          +----------+----------+----------+ |   Energy   |
          |   (DC)   |          |          | |   Router   |
          |  Port 3  |  Port 4  |  ..  ..  | | Operating  |
          |   (AC)   |          |          | |   System   |
          +----------+----------+----------+ +------------+
              | 400V     Idle
              v  10A

  Explained: Port 2 poses a demand of 10A 540VDC (5.4kW) and Port 3 a demand of
  10A 400VAC (6.9kW), leading to the router posing a total demand of 12.6kW
  from Port 1 (30A420VDC).  The 0.3kW difference is related to conversion and
  transmission losses.
  
  As a second example, in case Port 2 has supply capacity, Port 1 and 2 can
  share the increased load on Port 3:

              | 420V     | 570V
              v  20A     v  10A
          +----------+----------+----------+ +------------+
          |   (DC)   |   (DC)   |          | |            |
          |  Port 1  |  Port 2  |  ..  ..  | |  EP Ports  |
          |   (DC)   |   (DC)   |          | | Controller |
          +----------+----------+----------+ |            |
          |                                | +------------+
          |           Backplane            |
          |                                | +------------+
          +----------+----------+----------+ |   Energy   |
          |   (DC)   |          |          | |   Router   |
          |  Port 3  |  Port 4  |  ..  ..  | | Operating  |
          |   (AC)   |          |          | |   System   |
          +----------+----------+----------+ +------------+
              | 400V     Idle
              v  20A

2.2. Functional logic
  Regardless of the number of ports, a router can be said to have four logical
  sides, with the following functionality.  All sides can, as mentioned in the
  system architecture section, be either AC or DC, uni- or bi-directional
  ports, all running at different voltages.

                                Ports on the A side
                               are for general energy
                             use (such as home & office
                                    appliances)

      Ports on the D side         +-------------+
     are for transferring         |      A      |      Ports on the B side 
    energy to or from local       |             |      are for transferring
     energy resources (such       | D         B |     energy to or from the
    as photovoltaic and wind      |             |    traditional utility grid
   energy production, storage,    |      C      |
  or electric vehicle chargers)   +-------------+

                                 Ports on the C side
                                are for transferring
                               energy to or from other
                                energy routers (such
                                 as in a residential
                                  energy community)


  2.3. Network architecture
  The network architecture is based on a mesh topology, where each energy
  router can communicate with multiple other routers.  This allows for
  redundancy and resilience in the network, as well  efficient energy routing.
  The routers can dynamically adjust their connections based on the current
  energy supply and demand, as well as the availability of other routers in
  the network. The network can also be extended by adding new routers, which
  can automatically join the network and start communicating with other
  routers.

  Due to the nature of DC circuitry, routers can also be connected in a bus
  topology, where multiple routers are connected to a single DC bus. One
  router is dynamically elected as a bus driver to coordinate transfers
  between routers on the bus. This allows for simplified wiring and
  installation. The bus topology can be used in combination with the mesh
  topology, where some routers are connected in a bus and others are
  connected in a mesh. This allows for flexibility in the network design
  and growth.
  
  Example network architecture in bus topology:

                +---------+                 +---------+
                |    A    |                 |    A    |
                | D     B |                 | D     B |
                |    C    |                 |    C    |
                +---------+                 +---------+
                     ^                           ^
                     |                           |
             +-------+---------------------------+----+
             |                                        |
             |  +---------+                           |
             |  |    A    |                           |
             |  | D     B |                           |
             |  |    C    |                           |
             |  +---------+                           |
             |       ^                                |
             |       |                                |
             +-------+--------------------------------+

2.4. Network policy levels
  The routers and their energy networks (local and remote) require policies to
  guide their operation.  These policies can be segmented into three levels:
  
  Device level: Policies that are specific to a single device, such as supply
    and demand parameters, configuration of temperature derating hysteresis or
    state of charge bounds.  These policies are enforced in near real-time to
    balance the connections and make sure resources are not overloaded.  They
    can be set by the device or via the router and enforced by the device.

  Local network level: Policies that apply to a local energy
    network (LEN), such as maximum power transfer or minimum state of charge
    for the entire network.  It can also include prioritization between
    devices, such as prioritizing energy transfers to/from a specific device
    or type of devices.  These policies are enforced in near real-time by the
    router or the management systems.

  Remote network level: Policies that apply to remote energy network
    connections, such as pioritization of transfers or pricing to the
    connected remote networks.  It can also include access policies, to
    enable routers to have different policies depending on the tenant/owner
    of the remote networks.  This typically does not required real-time
    enforcement, but rather a periodic update of the policies.  These policies
    are enforced by the routers and the management systems.
 
  All policies are stored locally on the router and devices and are valid
  until updated by the management system or router.

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
