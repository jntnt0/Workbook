VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability.md

VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Data Center VXLAN EVPN and NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and EVPN consistency checks to `NS-XO_combined_epubs.md` |
| `CiscoPress_combined_part1.md` | DCCOR Ch 3, VXLAN Overview, VXLAN Tunnel Endpoint, VXLAN Control Plane, VXLAN Configurations and Verifications | Establishes the underlay vs overlay mental model, VTEP role, loopback sourced VXLAN encapsulation, PIM/OSPF underlay examples, and `show nve` verification |
| `NS-OS_combined_PDFs.md` | VXLAN Flood-and-Learn article, Configure and Verify sections | Provides NX-OS syntax for `feature nv overlay`, `feature vn-segment-vlan-based`, loopback VTEP source, PIM sparse-mode, OSPF loopback advertisement, `show nve interface`, `show ip mroute`, and `show nve peers` |
| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Provides operational checks for NVE source loopbacks, default VRF underlay uplinks, PIM sparse-mode on L3 uplinks when multicast is used, and L2VNI replication requirements |
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Underlay | The routed IP fabric that carries outer VXLAN packets between VTEP loopbacks |
| Overlay | The tenant/service network carried inside VXLAN after NVE encapsulation |
| Spine role | Routes underlay traffic only unless intentionally configured as a VTEP, route reflector, or service node |
| Leaf role | Acts as the VTEP edge where local VLANs, endpoints, and NVE encapsulation live |
| VTEP identity | The loopback sourced by `interface nve1`; remote VTEPs must be able to route to this /32 |
| Transport failure rule | If VTEP loopbacks cannot ping each other through the underlay, VXLAN cannot work correctly |
| Multicast flood-and-learn dependency | If multicast replication is used, PIM sparse-mode and RP reachability must work in the underlay before NVE peer discovery is trusted |
| EVPN dependency | If EVPN is used, BGP EVPN can only work after loopback reachability and underlay routing are stable |
| Related labs | `vxlan-one-spine-two-leafs-final`; `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final`; `vxlan-multicast-flood-and-learn-final` |
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm node roles before configuring overlay services | All nodes | `show hostname` | Spines and leaves are clearly identified before assigning loopbacks, routed links, and NVE roles |
| 2 | Confirm routed leaf-spine interfaces are not Layer 2 switchports | All fabric links | `show interface status`; `show running-config interface <interface>` | Leaf-spine links are planned as routed point-to-point interfaces |
| 3 | Enable the underlay routing feature | All fabric nodes | `conf t` then `feature ospf` | NX-OS accepts OSPF configuration under routed interfaces and loopbacks |
| 4 | Enable multicast routing only if the lab uses multicast flood-and-learn | All fabric nodes | `conf t` then `feature pim` | PIM commands are available on routed interfaces and loopbacks |
| 5 | Configure a stable underlay router-ID loopback | All fabric nodes | `interface loopback0` then `ip address <ROUTER_ID>/32` | Each fabric node has a unique /32 loopback for routing process identity |
| 6 | Advertise the router-ID loopback into the underlay IGP | All fabric nodes | `interface loopback0` then `ip router ospf <PROCESS_ID> area 0.0.0.0` | Router-ID loopbacks appear in the underlay routing table across the fabric |
| 7 | Configure a dedicated VTEP source loopback on leaves | Leaf VTEPs | `interface loopback1` then `ip address <VTEP_SOURCE_IP>/32` | Each leaf VTEP has a stable /32 source address for NVE encapsulation |
| 8 | Advertise the VTEP source loopback into the underlay IGP | Leaf VTEPs | `interface loopback1` then `ip router ospf <PROCESS_ID> area 0.0.0.0` | Every leaf can route to every remote VTEP source loopback |
| 9 | Enable PIM on VTEP loopbacks when multicast replication is used | Leaf VTEPs | `interface loopback1` then `ip pim sparse-mode` | VTEP loopbacks can source and receive multicast control/data-plane state for flood-and-learn labs |
| 10 | Configure routed point-to-point leaf-spine links | All fabric links | `interface <interface>` then `no switchport` then `ip address <P2P_IP>/<PREFIX>` | Leaf-spine interfaces become Layer 3 links with assigned point-to-point IPs |
| 11 | Advertise each routed fabric link into OSPF | All fabric links | `interface <interface>` then `ip router ospf <PROCESS_ID> area 0.0.0.0` | OSPF adjacencies form between directly connected leaves and spines |
| 12 | Enable PIM on routed fabric links when multicast replication is used | All fabric links | `interface <interface>` then `ip pim sparse-mode` | PIM neighbors form across leaf-spine routed links |
| 13 | Configure the multicast RP if using multicast flood-and-learn | Spine or RP node | `ip pim rp-address <RP_LOOPBACK_IP> group-list 224.0.0.0/4` | Leaves resolve a valid RP for VXLAN multicast groups |
| 14 | Enable VXLAN/NVE features on VTEP leaves only | Leaf VTEPs | `feature nv overlay`; `feature vn-segment-vlan-based` | Leaf switches can build NVE interfaces and bind VLANs to VNIs in later notes |
| 15 | Create the NVE interface and bind it to the VTEP loopback | Leaf VTEPs | `interface nve1` then `source-interface loopback1` then `no shutdown` | NVE interface has a configured VTEP source and is administratively enabled |
| 16 | Verify underlay route reachability to every VTEP source loopback | Leaf VTEPs | `show ip route <remote-vtep-loopback>` | Each remote VTEP /32 is reachable through the routed underlay |
| 17 | Test VTEP-to-VTEP loopback reachability | Leaf VTEPs | `ping <remote-vtep-loopback> source <local-vtep-loopback>` | Ping succeeds before any VLAN-to-VNI, EVPN, or flood-and-learn service is blamed |
| 18 | Verify OSPF neighbor state | All fabric nodes | `show ip ospf neighbors` | Leaf-spine adjacencies are FULL |
| 19 | Verify PIM neighbor state if multicast replication is used | All fabric nodes | `show ip pim neighbor` | PIM neighbors are present on routed fabric links |
| 20 | Verify RP state if multicast replication is used | Leaf VTEPs | `show ip pim rp` | Leaf VTEPs identify the expected RP for multicast VXLAN groups |
| 21 | Verify NVE source status | Leaf VTEPs | `show nve interface` | `nve1` is up or ready, with the expected source-interface and VTEP IP |
| 22 | Stop before configuring tenant VNIs | All nodes | `show running-config interface nve1` | This note proves transport and VTEP readiness only; VLAN-to-VNI and EVPN service configuration belongs in later mechanism notes |
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Skeleton
# SPINE UNDERLAY SKELETON
conf t
hostname <SPINE_NAME>
feature ospf
feature pim
interface loopback0
  description UNDERLAY_ROUTER_ID
  ip address <SPINE_ROUTER_ID>/32
  ip router ospf <PROCESS_ID> area 0.0.0.0
  ip pim sparse-mode
  no shutdown
interface <SPINE_TO_LEAF_INTERFACE>
  description TO_<LEAF_NAME>
  no switchport
  ip address <SPINE_P2P_IP>/<PREFIX>
  ip router ospf <PROCESS_ID> area 0.0.0.0
  ip pim sparse-mode
  no shutdown
router ospf <PROCESS_ID>
  router-id <SPINE_ROUTER_ID>
ip pim rp-address <RP_LOOPBACK_IP> group-list 224.0.0.0/4
end
# LEAF VTEP UNDERLAY SKELETON
conf t
hostname <LEAF_NAME>
feature ospf
feature pim
feature nv overlay
feature vn-segment-vlan-based
interface loopback0
  description UNDERLAY_ROUTER_ID
  ip address <LEAF_ROUTER_ID>/32
  ip router ospf <PROCESS_ID> area 0.0.0.0
  ip pim sparse-mode
  no shutdown
interface loopback1
  description VTEP_NVE_SOURCE
  ip address <LEAF_VTEP_IP>/32
  ip router ospf <PROCESS_ID> area 0.0.0.0
  ip pim sparse-mode
  no shutdown
interface <LEAF_TO_SPINE_INTERFACE>
  description TO_<SPINE_NAME>
  no switchport
  ip address <LEAF_P2P_IP>/<PREFIX>
  ip router ospf <PROCESS_ID> area 0.0.0.0
  ip pim sparse-mode
  no shutdown
router ospf <PROCESS_ID>
  router-id <LEAF_ROUTER_ID>
interface nve1
  description VXLAN_NVE_INTERFACE
  source-interface loopback1
  no shutdown
end
# IF THE LAB DOES NOT USE MULTICAST FLOOD-AND-LEARN
# Omit:
# feature pim
# ip pim rp-address <RP_LOOPBACK_IP> group-list 224.0.0.0/4
# ip pim sparse-mode
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Verification_Commands
# Node role and interface state
show hostname
show ip interface brief
show interface status
show running-config interface <interface>
# Underlay routing
show ip route
show ip route <remote-vtep-loopback>
show ip ospf neighbors
show ip ospf interface brief
show running-config ospf
# VTEP loopback reachability
ping <remote-vtep-loopback> source <local-vtep-loopback>
traceroute <remote-vtep-loopback> source <local-vtep-loopback>
# Multicast underlay, only for multicast flood-and-learn
show ip pim neighbor
show ip pim rp
show ip mroute
show ip mroute <vxlan-mcast-group>
# NVE readiness
show nve interface
show running-config interface nve1
show running-config interface loopback1
# Later service notes use these after VLAN/VNI members exist
show nve vni
show nve peers
show vxlan
show mac address-table dynamic
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Rollback
# LEAF VTEP NVE ROLLBACK
conf t
interface nve1
  shutdown
  no source-interface loopback1
exit
no interface nve1
end
# LEAF VTEP LOOPBACK ROLLBACK
conf t
interface loopback1
  no ip pim sparse-mode
  no ip router ospf <PROCESS_ID> area 0.0.0.0
  no ip address <LEAF_VTEP_IP>/32
exit
no interface loopback1
end
# UNDERLAY LINK ROLLBACK
conf t
interface <LEAF_SPINE_INTERFACE>
  shutdown
  no ip pim sparse-mode
  no ip router ospf <PROCESS_ID> area 0.0.0.0
  no ip address <P2P_IP>/<PREFIX>
exit
end
# MULTICAST ROLLBACK, ONLY IF THIS DEVICE NO LONGER USES PIM
conf t
no ip pim rp-address <RP_LOOPBACK_IP> group-list 224.0.0.0/4
no feature pim
end
# VXLAN FEATURE ROLLBACK, ONLY IF NO OTHER VXLAN SERVICES EXIST
conf t
no feature nv overlay
no feature vn-segment-vlan-based
end
# OSPF ROLLBACK, ONLY IF THIS DEVICE NO LONGER USES OSPF
conf t
no router ospf <PROCESS_ID>
no feature ospf
end
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Remote VTEP loopback does not ping | VTEP loopback not advertised into the underlay | `show ip route <remote-vtep-loopback>` | Add `ip router ospf <PROCESS_ID> area 0.0.0.0` under the VTEP source loopback |
| OSPF neighbor missing | Routed fabric link is down, still a switchport, wrong IP mask, or OSPF not enabled on the interface | `show ip ospf neighbors`; `show running-config interface <interface>` | Use `no switchport`, correct the IP addressing, enable OSPF on the link, and `no shutdown` |
| ECMP missing across spines | Only one routed path is advertised or one adjacency is down | `show ip route <remote-vtep-loopback>`; `show ip ospf neighbors` | Restore all leaf-spine adjacencies and confirm equal-cost underlay routing |
| `show nve interface` has no expected source | NVE source-interface missing or wrong loopback selected | `show running-config interface nve1` | Configure `source-interface loopback1` using the intended VTEP loopback |
| NVE source loopback reachable locally but not remotely | Loopback exists but is not in the underlay routing process | `show running-config interface loopback1`; `show ip route <leaf-vtep-ip>` from remote leaf | Advertise the VTEP loopback into OSPF or the chosen underlay IGP |
| PIM neighbors missing | PIM not enabled on routed leaf-spine links or `feature pim` missing | `show ip pim neighbor`; `show running-config interface <interface>` | Enable `feature pim` and `ip pim sparse-mode` on required routed links |
| RP not shown on leaf | RP address not configured or RP loopback not reachable | `show ip pim rp`; `show ip route <rp-loopback>` | Configure `ip pim rp-address <RP_LOOPBACK_IP>` and advertise the RP loopback |
| Multicast mroute missing for VXLAN group | No VNI member exists yet, no receiver joined, PIM broken, or RP broken | `show ip mroute <vxlan-mcast-group>`; `show ip pim rp` | Fix PIM/RP first, then verify VNI multicast group membership in the later VXLAN service note |
| Spine has no VXLAN/NVE state | Spine is not a VTEP | `show nve interface` | Expected in a pure leaf-spine transport design; check routing and multicast on spines instead |
| vPC VTEP pair has inconsistent NVE state | vPC pair does not share the required anycast secondary VTEP IP or NVE consistency is broken | `show vpc`; `show vpc consistency-parameters global`; `show nve interface` | Align vPC peer NVE source, secondary loopback, peer-link, and vPC consistency parameters |
| VXLAN service still fails after loopbacks ping | Underlay is working, issue is now in VLAN/VNI, NVE member, EVPN, ARP suppression, or host-facing VLAN config | `show nve vni`; `show vxlan`; `show bgp l2vpn evpn summary` | Move to the specific VXLAN service mechanism note instead of changing the underlay blindly |
##### Source_Basis
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Mental_Model
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Configuration_Checklist
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Skeleton
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Verification_Commands
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Rollback
# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability_Failure_Checks
# index of each title throughout note, not in table format

