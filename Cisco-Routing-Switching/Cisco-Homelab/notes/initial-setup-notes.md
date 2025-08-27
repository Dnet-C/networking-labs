# Cisco Home Lab – Initial Configuration

This lab documents my **initial configuration** to validate all hardware shortly after receiving it.  
The goal was to ensure that each device was operational, capable of passing traffic,  
and to test my proficiency in building a network **from scratch**.

I used my laptop to console into and configure each of the following devices:

- TP-Link AX6600 Wi-Fi 6 Router  
- Cisco ASA 8520 Firewall  
- Two Cisco 2911 ISR Routers  
- Cisco Catalyst 3650 (Layer 3 Switch)  
- Cisco Catalyst 2960 (Layer 2 Switch)  

---

## Cisco ASA 8520 – Initial Config

```cisco
hostname LAB-ASA
ip domain-name lab.local
username admin privilege 15 secret ***********
crypto key-generate rsa modulus 2048
ip ssh version 2

interface G0/0
 nameif outside
 security-level 0
 ip address 192.168.0.246 255.255.255.0
 no shutdown 

interface G0/1
 nameif inside
 security-level 100
 ip address 10.0.0.1 255.255.255.252
 no shutdown

route outside 0.0.0.0 0.0.0.0 192.168.0.1

ssh 192.168.0.0 255.255.255.0 outside
telnet 192.168.0.0 255.255.255.0 outside
aaa authentication ssh console LOCAL
aaa authentication telnet console LOCAL

✅ Result: Able to SSH into ASA from external network (Putty).
⚠️ Issue: Telnet failed even though allowed.
💡 Best practice: Removed Telnet for security.

### Cisco ASA 8520

```cisco
no telnet 192.168.0.0 255.255.255.0 outside
no aaa authentication telnet console LOCAL


#### Cisco 2911 Router (R1

```cisco
hostname LAB-R1

interface G0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface G0/1
 ip address 10.0.0.5 255.255.255.252
 no shutdown

interface G0/2
 no shutdown
 interface G0/2.1
  encapsulation dot1q 10
  ip address 10.10.10.2 255.255.255.0
 interface G0/2.2
  encapsulation dot1q 20
  ip address 10.10.20.2 255.255.255.0

interface Loopback0
 ip address 1.1.1.1 255.255.255.255

router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.4 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.255 area 0
 network 10.10.10.0 0.0.0.255 area 10
 network 10.10.20.0 0.0.0.255 area 10

ip route 0.0.0.0 0.0.0.0 10.0.0.1

###### Cisco 2911 Router (R2)

```cisco
hostname LAB-R2

interface G0/0
 ip address 10.0.0.6 255.255.255.252
 no shutdown

interface G0/1
 no shutdown
 interface G0/1.1
  encapsulation dot1q 10
  ip address 10.10.10.3 255.255.255.0
 interface G0/1.2
  encapsulation dot1q 20
  ip address 10.10.20.3 255.255.255.0

interface Loopback0
 ip address 2.2.2.2 255.255.255.255

router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.0.4 0.0.0.3 area 0
 network 10.10.10.0 0.0.0.255 area 20
 network 10.10.20.0 0.0.0.255 area 20

ip route 0.0.0.0 0.0.0.0 10.0.0.5

####### Cisco catalyst 3650 - L3 Switch (SW2)

```cisco
hostname LAB-SW2
ip routing

vtp domain lab
vtp mode server

vlan 10
 name Ubuntu
vlan 20
 name CentOS

interface G0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface G0/4
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 standby 1 ip 10.10.10.10
 no shutdown

interface Vlan20
 ip address 10.10.20.1 255.255.255.0
 standby 1 ip 10.10.20.10
 no shutdown

ip dhcp excluded-address 10.10.10.1 10.10.10.10
ip dhcp excluded-address 10.10.20.1 10.10.20.10

ip dhcp pool VLAN10
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server 8.8.8.8

ip dhcp pool VLAN20
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 8.8.8.8

######## Cisco Catalyst 2960 - L2 Switch (SW1)

```cisco
hostname LAB-SW1

interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface FastEthernet0/4
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface range G0/10 – 19
 switchport access vlan 10

interface range G0/20 – 29
 switchport access vlan 20

vtp domain lab
vtp mode client


# Testing & Validation

Connectivity Tests (ASA → Routers/Subinterfaces)

All pings successful ✅

ASA → R1 (10.0.0.2)

ASA → R2 (10.0.0.6)

ASA → R1 VLAN10 (10.10.10.2)

ASA → R1 VLAN20 (10.10.20.2)

ASA → R2 VLAN10 (10.10.10.3)

ASA → R2 VLAN20 (10.10.20.3)


DHCP Tests

✅ DHCP leases correctly issued on VLAN10 and VLAN20 from Catalyst 3650

⚠️ Initial DHCP failed (misconfigured default-router with subnet mask). Fixed by correcting config.

Excluded addresses reserved for future HSRP high availability lab.


## Key Takeaways

- Successfully brought all equipment online and verified connectivity.

- Practiced configuring ASA firewall management (SSH, routing, access).

- Built and tested OSPF adjacency between R1 & R2.

- Implemented VLANs, trunking, and inter-VLAN routing on Catalyst 3650.

- Deployed DHCP services for multiple VLANs.

- Corrected misconfiguration and documented troubleshooting process.

💡 Next Step: Extend this lab with HSRP & high availability testing.
