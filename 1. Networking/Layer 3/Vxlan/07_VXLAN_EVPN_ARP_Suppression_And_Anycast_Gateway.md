Cisco’s Nexus VXLAN guide states that ARP suppression is supported only when the VTEP hosts the distributed anycast gateway for that VNI, and that all VTEPs must use the same anycast gateway MAC for proper operation. It also states that ARP suppression must be consistent across the fabric for a given VNI.  

VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway.md

VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Data Center VXLAN EVPN and NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and consistency checks to `NS-XO_combined_epubs.md` |
| `CiscoPress_combined_part1.md` | DCCOR Ch 3, VXLAN MPBGP EVPN Control Plane and VXLAN EVPN design discussion | Supports the control-plane model where EVPN distributes host MAC/IP reachability and reduces flooding |
| `CiscoPress_combined_part2.md` | Anycast gateway discussion | Supports the mental model that fabric edge switches use the same gateway IP and MAC for a VLAN so endpoints keep a stable default gateway during mobility |
| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Supports checks that `suppress-arp` should only be used on L2VNIs where the extended VLAN SVI has `fabric forwarding mode anycast-gateway`, and that NVE, vPC, L2VNI, and replication consistency must be verified |
| `Cisco Nexus 9000 Series NX-OS VXLAN Configuration Guide, Release 10.6(x)` | Configure SVI for host-facing VXLAN routing and Configure ARP suppression | Provides current NX-OS syntax for `fabric forwarding anycast-gateway-mac`, `fabric forwarding mode anycast-gateway`, `global suppress-arp`, per-VNI `suppress-arp`, `suppress-arp disable`, and ARP suppression limitations |
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Distributed anycast gateway | Every participating VTEP hosts the same default gateway IP and virtual gateway MAC for the same stretched VLAN |
| Gateway IP consistency | The SVI IP address for a given VLAN/subnet must match on every VTEP that hosts that VLAN as an anycast gateway |
| Gateway MAC consistency | The global `fabric forwarding anycast-gateway-mac` value must match across the fabric |
| ARP suppression | The VTEP answers ARP locally when it has the target IP-to-MAC binding from EVPN instead of flooding ARP across the VXLAN fabric |
| EVPN dependency | ARP suppression depends on EVPN MAC/IP Route Type 2 learning; without MAC/IP reachability, the VTEP cannot safely suppress ARP |
| Silent host behavior | If the ARP suppression cache lacks the requested entry, ARP still floods so silent or unknown hosts can be discovered |
| L2VNI dependency | `suppress-arp` belongs under the L2VNI member in `interface nve1`, not under the VLAN or SVI |
| Anycast gateway dependency | Do not enable `suppress-arp` for an L2VNI unless the associated SVI is configured with `fabric forwarding mode anycast-gateway` |
| Fabric consistency rule | For a given L2VNI, either all participating VTEPs use ARP suppression or none of them do |
| What this note assumes | Underlay reachability, BGP EVPN sessions, L2VNI mapping, NVE host reachability, and BGP ingress replication are already working |
| What this note is not | This note does not build the full underlay, EVPN route-reflector fabric, L3VNI route leaking, multisite, or tenant routed multicast |
| Related labs | `vxlan-bgp-evpn-arp-suppression-final` |
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm underlay VTEP reachability is already working | Leaf VTEPs | `ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>` | Remote VTEP loopbacks are reachable before ARP suppression is touched |
| 2 | Confirm EVPN BGP control plane is already established | Leaf VTEPs | `show bgp l2vpn evpn summary` | EVPN neighbors are established |
| 3 | Confirm the L2VNI already exists and is control-plane learned | Leaf VTEPs | `show nve vni <L2VNI> detail` | L2VNI is up and uses control-plane mode |
| 4 | Confirm NVE uses BGP host reachability | Leaf VTEPs | `show running-config interface nve1` | `host-reachability protocol bgp` is configured |
| 5 | Confirm remote VTEPs are control-plane learned | Leaf VTEPs | `show nve peers` | Remote VTEPs appear with LearnType `CP` |
| 6 | Confirm VLAN-to-VNI mapping is correct | Leaf VTEPs | `show vxlan` | VLAN maps to the intended L2VNI |
| 7 | Enable SVI support if not already enabled | Leaf VTEPs | `feature interface-vlan` | NX-OS accepts `interface vlan <VLAN_ID>` |
| 8 | Confirm VXLAN EVPN feature state | Leaf VTEPs | `show feature | include bgp`; `show feature | include nv`; `show running-config | include "nv overlay evpn"` | BGP, NVE, and EVPN overlay control plane are active |
| 9 | Configure the global anycast gateway MAC | All participating VTEPs | `fabric forwarding anycast-gateway-mac <ANYCAST_GATEWAY_MAC>` | Every VTEP uses the same distributed gateway MAC |
| 10 | Confirm the tenant VRF exists if the gateway SVI belongs to a tenant VRF | Leaf VTEPs | `show vrf <TENANT_VRF>` | Tenant VRF exists before binding the SVI |
| 11 | Confirm L3VNI exists if inter-VNI routing is part of this lab | Leaf VTEPs | `show nve vni` | Tenant L3VNI exists and is associated to the VRF if routed service is required |
| 12 | Create or confirm the VLAN | Leaf VTEPs | `vlan <VLAN_ID>` | VLAN exists locally on each VTEP that hosts the segment |
| 13 | Map the VLAN to the L2VNI | Leaf VTEPs | `vn-segment <L2VNI>` | VLAN is bound to the correct VXLAN L2VNI |
| 14 | Create or enter the SVI for the stretched VLAN | Leaf VTEPs | `interface vlan <VLAN_ID>` | SVI configuration mode opens |
| 15 | Bind the SVI to the tenant VRF if used | Leaf VTEPs | `vrf member <TENANT_VRF>` | SVI belongs to the correct tenant routing table |
| 16 | Configure the anycast gateway IP address | Leaf VTEPs | `ip address <ANYCAST_GATEWAY_IP>/<PREFIX>` | SVI has the same default gateway IP on every participating VTEP |
| 17 | Disable redirects on the anycast SVI | Leaf VTEPs | `no ip redirects` | SVI avoids sending redirects in the overlay gateway role |
| 18 | Enable distributed anycast gateway on the SVI | Leaf VTEPs | `fabric forwarding mode anycast-gateway` | SVI acts as the distributed gateway for the VLAN |
| 19 | Enable the SVI | Leaf VTEPs | `no shutdown` | SVI is administratively up |
| 20 | Confirm host-facing ports carry the VLAN | Leaf VTEPs | `show interface <HOST_PORT> switchport`; `show interface trunk` | Local endpoints are attached to the correct VLAN |
| 21 | Enter NVE interface configuration | Leaf VTEPs | `interface nve1` | NVE configuration mode opens |
| 22 | Confirm L2VNI is configured under NVE | Leaf VTEPs | `show running-config interface nve1` | `member vni <L2VNI>` exists |
| 23 | Enable ARP suppression globally for all eligible L2VNIs if the design uses global suppression | Leaf VTEPs | `global suppress-arp` | All eligible L2VNIs under NVE use ARP suppression unless disabled per VNI |
| 24 | Enable ARP suppression only for the target L2VNI if the design uses per-VNI suppression | Leaf VTEPs | `member vni <L2VNI>` then `suppress-arp` | Only the selected L2VNI suppresses ARP |
| 25 | Disable ARP suppression for an exception VNI if global suppression is enabled | Leaf VTEPs | `member vni <L2VNI>` then `suppress-arp disable` | The exception L2VNI does not use ARP suppression |
| 26 | Keep ARP suppression consistent for the same L2VNI across all VTEPs | All participating VTEPs | `show running-config interface nve1` | Every VTEP has matching ARP suppression behavior for the same L2VNI |
| 27 | Generate endpoint ARP and traffic | Hosts | `ping <REMOTE_HOST_IP>` | Hosts learn gateway and remote endpoint reachability |
| 28 | Verify EVPN MAC/IP advertisements exist | Leaf VTEPs | `show bgp l2vpn evpn route-type macip` | Local and remote MAC/IP bindings appear in EVPN |
| 29 | Verify ARP suppression flag on the VNI | Leaf VTEPs | `show nve vni <L2VNI> detail` | L2VNI shows suppress ARP state |
| 30 | Verify local ARP and remote ARP learning | Leaf VTEPs | `show ip arp vrf <TENANT_VRF>` | Local and remote endpoint ARP entries appear as expected |
| 31 | Verify host database entries | Leaf VTEPs | `show fabric forwarding ip local-host-db vrf <TENANT_VRF>` | Local hosts appear in the fabric forwarding host database |
| 32 | Verify remote MAC/IP programming | Leaf VTEPs | `show l2route evpn mac-ip all` | Remote MAC/IP entries are learned from EVPN |
| 33 | Verify endpoint default gateway behavior | Hosts | `arp -a`; `ping <ANYCAST_GATEWAY_IP>` | Hosts resolve the shared anycast gateway MAC and can ping the gateway |
| 34 | Verify same-subnet host reachability | Hosts | `ping <REMOTE_HOST_IP>` | Hosts in the same VLAN/L2VNI can communicate across the fabric |
| 35 | If inter-VNI routing is enabled, verify routed reachability through the distributed gateway | Hosts | `ping <REMOTE_SUBNET_HOST_IP>` | Hosts in different VNIs route through the local leaf anycast gateway |
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Skeleton
# BASE FEATURE CHECK ON EACH LEAF VTEP
conf t
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay
nv overlay evpn
fabric forwarding anycast-gateway-mac <ANYCAST_GATEWAY_MAC>
end
# VLAN TO L2VNI MAPPING
conf t
vlan <VLAN_ID>
  name <SEGMENT_NAME>
  vn-segment <L2VNI>
end
# ANYCAST GATEWAY SVI
conf t
interface vlan <VLAN_ID>
  description ANYCAST_GATEWAY_FOR_<SEGMENT_NAME>
  no shutdown
  vrf member <TENANT_VRF>
  ip address <ANYCAST_GATEWAY_IP>/<PREFIX>
  no ip redirects
  fabric forwarding mode anycast-gateway
end
# HOST-FACING ACCESS PORT
conf t
interface <HOST_PORT>
  description TO_<HOST_OR_SERVER>
  switchport
  switchport mode access
  switchport access vlan <VLAN_ID>
  no shutdown
end
# NVE L2VNI WITH PER-VNI ARP SUPPRESSION
conf t
interface nve1
  source-interface loopback<VTEP_LOOPBACK_ID>
  host-reachability protocol bgp
  member vni <L2VNI>
    ingress-replication protocol bgp
    suppress-arp
  no shutdown
end
# NVE GLOBAL ARP SUPPRESSION OPTION
conf t
interface nve1
  global suppress-arp
end
# PER-VNI EXCEPTION WHEN GLOBAL ARP SUPPRESSION IS ENABLED
conf t
interface nve1
  member vni <L2VNI>
    suppress-arp disable
end
# OPTIONAL IPV6 ND SUPPRESSION
# Use only when suppress-arp is enabled, the associated SVI is up,
# and the SVI has both IPv4 and IPv6 addressing as required by the design.
conf t
interface nve1
  suppress nd
end
# OPTIONAL TCAM CARVE FOR PLATFORMS THAT REQUIRE IT
# Save and reload if the platform prompts for reload.
conf t
hardware access-list tcam region arp-ether 256 double-wide
end
copy running-config startup-config
reload
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Verification_Commands
# Baseline VXLAN EVPN state
show bgp l2vpn evpn summary
show bgp l2vpn evpn route-type imet
show bgp l2vpn evpn route-type macip
show nve interface
show nve peers
show nve vni
show nve vni <L2VNI> detail
# Anycast gateway configuration
show running-config | include anycast-gateway-mac
show running-config interface vlan <VLAN_ID>
show ip interface brief vrf all
show vrf <TENANT_VRF>
# VLAN and VNI mapping
show vlan id <VLAN_ID>
show vxlan
show running-config vlan <VLAN_ID>
# NVE ARP suppression configuration
show running-config interface nve1
show nve vni <L2VNI> detail
# ARP and MAC/IP learning
show ip arp vrf <TENANT_VRF>
show mac address-table dynamic vlan <VLAN_ID>
show l2route evpn mac all
show l2route evpn mac-ip all
show fabric forwarding ip local-host-db vrf <TENANT_VRF>
# Endpoint verification
ping <ANYCAST_GATEWAY_IP>
ping <REMOTE_HOST_IP>
ping <REMOTE_SUBNET_HOST_IP>
arp -a
# vPC VTEP checks, only if vPC VTEPs are used
show vpc brief
show vpc consistency-parameters global
show port-channel summary
show running-config interface nve1
show running-config interface vlan <VLAN_ID>
# Optional IPv6 ND suppression verification
show running-config nv overlay
show ipv6 nd suppression-cache detail
show ipv6 nd suppression-cache local
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Rollback
# REMOVE PER-VNI ARP SUPPRESSION
conf t
interface nve1
  member vni <L2VNI>
    no suppress-arp
end
# DISABLE ARP SUPPRESSION FOR ONE VNI WHEN GLOBAL SUPPRESSION IS ENABLED
conf t
interface nve1
  member vni <L2VNI>
    suppress-arp disable
end
# REMOVE GLOBAL ARP SUPPRESSION
conf t
interface nve1
  no global suppress-arp
end
# REMOVE OPTIONAL ND SUPPRESSION
conf t
interface nve1
  no suppress nd
end
# REMOVE ANYCAST GATEWAY MODE FROM SVI
conf t
interface vlan <VLAN_ID>
  no fabric forwarding mode anycast-gateway
end
# SHUT DOWN THE ANYCAST GATEWAY SVI WITHOUT DELETING IT
conf t
interface vlan <VLAN_ID>
  shutdown
end
# REMOVE SVI IP ADDRESS
conf t
interface vlan <VLAN_ID>
  no ip address <ANYCAST_GATEWAY_IP>/<PREFIX>
end
# REMOVE SVI FROM TENANT VRF
conf t
interface vlan <VLAN_ID>
  no vrf member <TENANT_VRF>
end
# REMOVE THE SVI COMPLETELY
conf t
no interface vlan <VLAN_ID>
end
# REMOVE VLAN-TO-VNI MAPPING BUT KEEP VLAN
conf t
vlan <VLAN_ID>
  no vn-segment <L2VNI>
end
# REMOVE GLOBAL ANYCAST GATEWAY MAC ONLY IF NO VXLAN ANYCAST GATEWAY SVIs REMAIN
conf t
no fabric forwarding anycast-gateway-mac <ANYCAST_GATEWAY_MAC>
end
# REMOVE INTERFACE VLAN FEATURE ONLY IF NO SVIs ARE USED ANYWHERE
conf t
no feature interface-vlan
end
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `suppress-arp` is configured but ARP still floods | Missing EVPN MAC/IP entry, silent host, or ARP suppression cache miss | `show bgp l2vpn evpn route-type macip`; `show l2route evpn mac-ip all` | Generate host traffic, verify EVPN MAC/IP route learning, and confirm local ARP is populated |
| ARP suppression breaks connectivity | `suppress-arp` enabled without a valid anycast gateway SVI | `show running-config interface vlan <VLAN_ID>`; `show nve vni <L2VNI> detail` | Configure `fabric forwarding mode anycast-gateway` on the associated SVI or remove `suppress-arp` |
| Hosts resolve different gateway MACs on different leaves | Anycast gateway MAC mismatch | `show running-config | include anycast-gateway-mac` | Configure the same `fabric forwarding anycast-gateway-mac <MAC>` on all participating VTEPs |
| Host cannot ping its default gateway | SVI is down, wrong VLAN, wrong VRF, missing anycast gateway mode, or host-facing port is wrong | `show ip interface brief vrf all`; `show interface <HOST_PORT> switchport`; `show vlan id <VLAN_ID>` | Bring up SVI, correct VLAN attachment, assign VRF, and configure anycast gateway mode |
| Host can ping gateway but not remote same-subnet host | L2VNI, EVPN, or MAC/IP route issue | `show nve peers`; `show nve vni <L2VNI> detail`; `show bgp l2vpn evpn route-type macip` | Fix EVPN L2VNI control plane and verify MAC/IP routes |
| Host can reach same subnet but not another subnet | L3VNI, VRF routing, or route leaking issue | `show vrf <TENANT_VRF>`; `show nve vni`; `show ip route vrf <TENANT_VRF>` | Troubleshoot the L3VNI and tenant routing note, not ARP suppression |
| `show nve vni` does not show suppress ARP flag | `suppress-arp` missing or global suppression not configured | `show running-config interface nve1` | Add `suppress-arp` under the L2VNI or configure `global suppress-arp` |
| Some VTEPs suppress ARP and others do not | Fabric-wide ARP suppression mismatch for the same L2VNI | `show running-config interface nve1` on every VTEP | Make ARP suppression consistent across all VTEPs participating in that VNI |
| EVPN MAC routes exist but MAC/IP routes are missing | Host IP binding is not learned or not advertised | `show bgp l2vpn evpn route-type macip`; `show ip arp vrf <TENANT_VRF>` | Generate ARP/ICMP from hosts and verify ARP adjacency learning |
| Remote MAC/IP exists in BGP but not installed locally | L2VNI down, RT mismatch, or L2RIB programming issue | `show l2route evpn mac-ip all`; `show nve vni <L2VNI> detail`; `show running-config evpn` | Fix L2VNI state and route-target import/export |
| `fabric forwarding mode anycast-gateway` is rejected | Missing `feature interface-vlan`, unsupported platform, or wrong interface mode | `show feature | include interface-vlan`; `show version` | Enable `feature interface-vlan` and confirm platform support |
| SVI loses IP configuration after `vrf member` | NX-OS clears L3 config when changing VRF membership | `show running-config interface vlan <VLAN_ID>` | Configure `vrf member` first, then reapply IP address and anycast gateway mode |
| vPC pair behaves inconsistently | vPC peer VLAN/VNI/NVE/SVI/anycast config mismatch | `show vpc consistency-parameters global`; `show running-config interface nve1`; `show running-config interface vlan <VLAN_ID>` | Align VLAN-to-VNI, NVE member, SVI IP, anycast gateway MAC, and ARP suppression on both vPC peers |
| `show nve peers` has no control-plane peers | EVPN BGP or IMET route issue | `show bgp l2vpn evpn summary`; `show bgp l2vpn evpn route-type imet` | Fix EVPN BGP neighbor state, route reflection, and L2VNI RTs |
| ND suppression causes IPv6 host drops | ND suppression enabled without required SVI and IPv6 conditions | `show running-config nv overlay`; `show ipv6 nd suppression-cache detail` | Remove `suppress nd` unless ARP suppression, SVI state, and IPv4/IPv6 requirements are satisfied |
| ARP suppression command appears correct but platform still fails | Required TCAM region is missing on platforms that need it | `show hardware access-list tcam region` | Configure `hardware access-list tcam region arp-ether 256 double-wide`, save, and reload if required |
| ARP suppression blamed before EVPN is working | Wrong troubleshooting layer | `show bgp l2vpn evpn summary`; `show nve peers`; `show bgp l2vpn evpn route-type macip` | Prove EVPN L2VNI control plane first, then troubleshoot ARP suppression |
##### Source_Basis
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Mental_Model
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Configuration_Checklist
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Skeleton
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Verification_Commands
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Rollback
# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway_Failure_Checks
# index of each title throughout note, not in table format

