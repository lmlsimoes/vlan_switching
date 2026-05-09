VLAN Local Significance & Translation with SR Linux (Containerlab Lab)
Overview

This lab demonstrates how VLANs are locally significant and how Nokia SR Linux can translate VLANs using MAC‑VRFs, while still maintaining strict traffic isolation — even when end hosts use overlapping IPv4/IPv6 subnets.
The lab is built using containerlab, Linux network namespaces, and a Nokia 7220 IXR‑D3L running SR Linux.
A key objective of this lab is to show that:

Even if Client 1 and Client 2 use the same IP subnet, traffic remains isolated because:

VLANs are translated locally on the switch
Each client is bound to a different MAC‑VRF
Linux namespaces provide independent L3 stacks



Topology

Components


Nutanix host (Linux container)

Connected to the leaf switch on port e1‑1
Carries:

VLAN 101 → Client 1
VLAN 102 → Client 2


Uses network namespaces to simulate multiple isolated clients



Nokia 7220 IXR‑D3L (SR Linux)

Acts as the leaf switch
Translates VLANs using MAC‑VRFs
Enforces traffic separation at Layer‑2



Firewall A

Connected on port e1‑5
Uses VLAN 10
Represents Client 1 security domain



Firewall B

Connected on port e1‑6
Uses VLAN 10
Represents Client 2 security domain




VLAN & MAC‑VRF Design
Client 1

Nutanix side: VLAN 101
Firewall side: VLAN 10
SR Linux MAC‑VRF: l2_cliente_a
Interfaces:

ethernet-1/1.101
ethernet-1/5.10



Client 2

Nutanix side: VLAN 102
Firewall side: VLAN 10
SR Linux MAC‑VRF: l2_cliente_b
Interfaces:

ethernet-1/1.102
ethernet-1/6.10



✅ This clearly shows VLANs are not globally significant — the same VLAN ID (10) is reused on different ports but remains isolated through separate MAC‑VRFs.

IP Addressing Model
Both clients intentionally use the same IP subnet to prove isolation:

IPv4 (example): 10.10.10.0/24
IPv6 (derived): 2002::10:10:10:X/96

Despite identical addressing:

Client 1 cannot reach Client 2
Client 2 cannot reach Client 1

This isolation is enforced by:

VLAN separation
MAC‑VRFs on the switch
Independent Linux network namespaces


Linux Network Namespaces
On the Nutanix host, each client is represented by a network namespace:

ns101 → Client 1 on VLAN 101
ns102 → Client 2 on VLAN 102

Each namespace has:

Its own VLAN sub‑interface
Its own MAC address
Its own IPv4/IPv6 addresses
Its own routing table

Key Point

Namespaces isolate Layer‑3 completely.
Even if two namespaces use the same subnet, they behave like separate hosts.


Namespace Creation Script
The file create_vlan_namespace.sh is invoked by containerlab and performs the following per client:

Creates a network namespace (ns<VLANID>)
Creates a VLAN sub‑interface on the parent NIC
Moves the interface into the namespace
Assigns:

IPv4 address
Derived IPv6 address (2002::/96)


Sets a deterministic, locally administered MAC address
Installs supernet routes:

IPv4: 10.0.0.0/8
IPv6: 2002::/16



Deterministic MAC Logic
MAC addresses are derived using:
02:<vlan_high>:<vlan_low>:<ipv4_octet_2>:<ipv4_octet_3>:<ipv4_octet_4>

This ensures:

No MAC collisions
Stable behavior across reboots
Compatibility with EVPN / MAC learning
Easy troubleshooting


Validation & Expected Results
From Client 1 (ns101)

✅ Can ping Firewall A
❌ Cannot ping Firewall B
❌ Cannot ping Client 2

From Client 2 (ns102)

✅ Can ping Firewall B
❌ Cannot ping Firewall A
❌ Cannot ping Client 1

This clearly proves:

VLAN translation works
MAC‑VRFs enforce isolation
Overlapping IP addressing is safe when properly segmented


Files Included


lab_vlan_switching_1_leaf.clab.yml
Containerlab topology definition


leaf_switch.flat.txt
SR Linux startup configuration (interfaces, VLANs, MAC‑VRFs)


create_vlan_namespace.sh
Deterministic namespace + VLAN + IP + MAC setup script



Key Takeaways

✅ VLANs are locally significant, not end‑to‑end
✅ SR Linux MAC‑VRFs provide clean L2 service separation
✅ VLAN ID reuse is safe and scalable
✅ Identical IP subnets can coexist without conflict
✅ This model aligns with EVPN and modern fabric designs