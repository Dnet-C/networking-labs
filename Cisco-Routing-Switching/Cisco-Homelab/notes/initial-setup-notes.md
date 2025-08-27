Initial Lab Configuration – ASA + Routing + Switching

This lab represents my initial configuration after receiving the equipment. The primary goal was to verify that all devices were operational, able to pass traffic, and that I could successfully build a functioning network from scratch.

I used my laptop to console into and configure each of the following devices:

TP-Link AX6600 Wi-Fi 6 Router

Cisco ASA 8520

Two Cisco 2911 ISR Routers

Cisco Catalyst 3650 (Layer 3 Switch)

Cisco Catalyst 2960 (Layer 2 Switch)

Cisco ASA 8520 Configuration
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


✅ Result: SSH worked successfully from an external PC using PuTTY.
⚠️ Issue: Telnet access failed despite explicit configuration.
🔐 Best Practice: Telnet should be disabled for security.

no telnet 192.168.0.0 255.255.255.0 outside
no aaa authentication telnet console LOCAL

TP-Link AX6600 Wi-Fi 6 Router
Accessed web console from a host on 192.168.0.0/24 and reserved 192.168.0.246/24 using the ASA MAC Address.

Cisco 2911 Router 1 (R1)
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

interface loopback0
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

Cisco 2911 Router 2 (R2)
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

interface loopback0
  ip address 2.2.2.2 255.255.255.255

router ospf 1
  router-id 2.2.2.2
  network 2.2.2.2 0.0.0.0 area 0
  network 10.0.0.4 0.0.0.3 area 0
  network 10.10.10.0 0.0.0.255 area 20
  network 10.10.20.0 0.0.0.255 area 20

ip route 0.0.0.0 0.0.0.0 10.0.0.5

Cisco Catalyst 3650 (SW2 – L3)
hostname Lab-SW2
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

Cisco Catalyst 2960 (SW1 – L2)
hostname Lab-SW1

interface Fa0/1
  switchport trunk encapsulation dot1q
  switchport mode trunk

interface Fa0/4
  switchport trunk encapsulation dot1q
  switchport mode trunk

interface range G0/10 – 19
  switchport access vlan 10

interface range G0/20 – 29
  switchport access vlan 20

vtp domain lab
vtp mode client


Testing Steps

Connectivity from ASA (192.168.0.246):

✅ SSH to ASA from external host

✅ Pings successful to:

R1 (10.0.0.2)

R2 (10.0.0.6)

VLAN 10 (10.10.10.2, 10.10.10.3)

VLAN 20 (10.10.20.2, 10.10.20.3)


DHCP Tests (Client connections):

SW1 (2960 – L2):

G0/25 (VLAN20): received 10.10.20.11 / 24, GW 10.10.20.10

G0/13 (VLAN10): received 10.10.10.11 / 24, GW 10.10.10.10


SW2 (3650 – L3):

G0/17 (VLAN10): received 10.10.10.11 / 24, GW 10.10.10.10

G0/23 (VLAN20): received 10.10.20.11 / 24, GW 10.10.20.10


⚠️ Issue: DHCP initially failed because default-router was incorrectly configured with a subnet mask. After correction, DHCP worked as expected.

💡 Future Planning: DHCP exclusions account for IPs reserved for HSRP and high availability in future labs.
