---
layout: lab
title: Inter-VLAN L3 
description: The next step from VLANs — inter-VLAN routing using a Layer 3 switch. A Cisco Packet Tracer lab simulating a real office network with IT and HR departments on separate VLANs
diagram: /InterVLAN-L3/diagram.png

concepts:
  - Layer 3 Switching
  - SVI Configurations
  - Inter-VLAN Routing
  - DHCP Pooling
  - dot1q Trunking

objective: >
  The objective of this lab is to develop an understanding of inter-VLAN routing using a Layer 3 switch.
  While a router-only approach was available, the Layer 3 switch was chosen as the more viable option due to its scalability.
  The lab simulates a real office scenario with IT and HR departments on separate VLANs communicating through a Layer 3 switch.

hardware:
  - "1x Cisco 3560 24PS (Layer 3 Switch — D1)"
  - "1x Cisco Catalyst 2960 (Layer 2 Switch — SW1)"
  - "7x End Devices (PCs & Laptops)"
  - "1x Cisco 2911 Router"

questions:
  - q: Why couldn't I use one switch for the setup?
    a: >
      For a lab, a single switch architecture would be fine. But the aim here was also to simulate a real enterprise network.
      In a real-world enterprise, each floor or wing would ideally get its own access switch. Since this lab deals with two departments (IT and HR), two switches better reflect that reality.

  - q: Why do I need to configure VLANs as interfaces?
    a: >
      The logical SVI interfaces inside the Layer 3 switch act as the default gateways — not physical ports.
      Without the VLAN interfaces, D1 has no Layer 3 presence to route data between VLANs at the network layer.

  - q: What is stopping the Layer 3 switch from handing out the incorrect DHCP pool?
    a: >
      The DHCP pool name is just a label for identification — it's not what the switch uses to assign addresses.
      The switch assigns DHCP info based on which SVI the traffic arrives on, so it always gives the correct pool.

  - q: Why do I need trunk ports between the switches and the Layer 3 switch?
    a: >
      The L3 switch needs to know which traffic belongs to which VLAN. If access ports were used, VLAN tags would be stripped,
      leaving the L3 switch unable to assign the right IP addresses to devices.

  - q: What is dot1q and why did I have to specify it?
    a: >
      dot1q (802.1Q) is the standard protocol that defines how VLAN tags are added to Ethernet frames.
      The 3560 supports both dot1q and the older ISL protocol, so you have to explicitly specify which one to use before enabling trunking.

steps:
  - title: Enable access ports on S1 & S2
    desc: Set the host-facing ports to access mode on both Layer 2 switches.
    code: |
      S1> enable
      S1# configure terminal
      S1(config)# interface range FastEthernet 0/1 - 3
      S1(config-if-range)# switchport mode access
      S1(config-if-range)# exit
      S1# write memory

  - title: Create VLANs and assign ports
    desc: Enable VLANs 5 (IT) and 10 (HR) and assign switch ports to each VLAN.
    code: |
      S1(config)# vlan 5
      S1(config-vlan)# name IT
      S1(config-vlan)# exit
      S1(config)# interface range FastEthernet 0/1 - 3
      S1(config-if-range)# switchport access vlan 5

  - title: Enable trunk ports on S1 & S2 toward D1
    desc: Set the uplink ports to the Layer 3 switch as trunk ports to carry tagged traffic.
    code: |
      S1(config)# interface Gig0/1
      S1(config-if)# switchport mode trunk

  - title: Enable trunk ports on D1 (Layer 3 switch)
    desc: >
      The 3560 supports multiple trunking protocols so you must specify dot1q before enabling trunk mode.
    code: |
      D1(config)# interface FastEthernet0/1
      D1(config-if)# switchport trunk encapsulation dot1q
      D1(config-if)# switchport mode trunk

  - title: Create SVI interfaces as default gateways
    desc: Configure virtual VLAN interfaces on D1 to act as Layer 3 gateways for each VLAN.
    code: |
      D1(config)# interface vlan 5
      D1(config-if)# ip address 192.168.9.1 255.255.255.0
      D1(config-if)# no shutdown
      D1(config-if)# exit
      D1(config)# interface vlan 10
      D1(config-if)# ip address 192.168.13.1 255.255.255.0
      D1(config-if)# no shutdown

  - title: Configure DHCP pools on D1
    desc: Set up DHCP pools for each VLAN so devices receive IP addresses automatically.
    code: |
      D1(config)# ip dhcp pool VLAN5
      D1(dhcp-config)# network 192.168.9.0 255.255.255.0
      D1(dhcp-config)# default-router 192.168.9.1
      D1(dhcp-config)# exit

      D1(config)# ip dhcp pool VLAN10
      D1(dhcp-config)# network 192.168.13.0 255.255.255.0
      D1(dhcp-config)# default-router 192.168.13.1
      D1(dhcp-config)# exit

      D1(config)# ip routing
      D1# write memory

  - title: Set PCs to DHCP and verify connectivity
    desc: Switch each PC's IP configuration to DHCP, then ping across VLANs to confirm inter-VLAN routing works.
    code: |
      ! From a PC in VLAN 5, ping a device in VLAN 10
      C:\> ping 192.168.13.x

learnings:
  - Layer 3 switches can enable inter-VLAN routing, allowing devices on different VLANs to communicate.
  - DHCP pooling automates IP assignment without manual configuration per device.
  - dot1q is the standard VLAN trunking protocol — older Cisco switches may support ISL too.
  - Without SVIs configured, a Layer 3 switch behaves just like a Layer 2 switch with no routing capability.
---
