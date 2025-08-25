# 🔹 Lab Objective

The objective of this lab is to design, configure, and test a multi-device enterprise-style network using both home and enterprise-grade networking equipment. This topology integrates a **TP-Link AX6600 Wi-Fi 6 Tri-Band Router** with a **Cisco ASA 8520 firewall**, **two Cisco 2911 ISR routers**, a **Cisco Catalyst 3650 Layer 3 switch**, and a **Cisco Catalyst 2960 Layer 2 switch**.  

The lab is intended to simulate real-world enterprise networking scenarios with the following goals:  

1. **Firewall Integration** – Connect the ASA firewall to the home router for perimeter defense and controlled access into the internal Cisco lab environment.  
2. **Static IP Assignments** – Configure all routers, switches, and firewall interfaces with static addressing for predictable routing and connectivity.  
3. **Dynamic Routing with OSPF** – Implement OSPF across routers to provide dynamic route exchange and redundancy in the internal network.  
4. **VLANs and Trunking** – Configure VLANs on the Catalyst switches with IEEE 802.1Q trunking between switches and routers for segmented broadcast domains.  
5. **Inter-VLAN Routing** – Enable IP routing on the Catalyst 3650 (Layer 3) switch to allow communication between VLANs.  
6. **VTP Server Role** – Configure the Catalyst 3650 as a VTP server for VLAN propagation across the switching environment.  
7. **DHCP Services** – Deploy DHCP pools on the Catalyst 3650 to dynamically assign IP addresses to hosts within VLANs.  
8. **Remote Management** – Securely configure **SSH** and **Telnet** access on all devices to allow remote management from a host within the 192.168.0.0/24 subnet connected to the TP-Link router.  
9. **End-to-End Connectivity Testing** – Validate connectivity using ping, traceroute, SSH, and Telnet across all segments of the lab to ensure proper routing, switching, firewall access, and management reachability.  

---

This lab provides hands-on experience in **firewall configuration, routing protocols, VLAN management, DHCP services, and secure remote device administration**.
