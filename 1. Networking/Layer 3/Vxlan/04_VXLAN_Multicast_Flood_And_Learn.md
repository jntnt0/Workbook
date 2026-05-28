VXLAN_Multicast_Flood_And_Learn.md

VXLAN_Multicast_Flood_And_Learn

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Data Center VXLAN EVPN and NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN theory to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and consistency checks to `NS-XO_combined_epubs.md` |
| `CiscoPress_combined_part1.md` | DCCOR Ch 3, VXLAN Control Plane, VXLAN Flood and Learn Multicast-Based Control Plane, VXLAN Configurations and Verifications | Supports BUM traffic handling, VTEP discovery, multicast group mapping per VNI, PIM joins, NVE verification, and data-plane MAC learning |
| `NS-OS_combined_PDFs.md` | VXLAN Flood-and-Learn article, Configure and Verify sections | Provides NX-OS syntax for `feature vn-segment-vlan-based`, `feature nv overlay`, `feature pim`, VLAN-to-VNI mapping, `member vni <vni> mcast-group <group>`, PIM sparse-mode, RP configuration, `show nve interface`, `show nve peers`, `show nve vni`, and `show ip mroute` |
| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Supports operational checks that each L2VNI needs `mcast-group` or another replication method, PIM sparse-mode is required on L3 uplinks when multicast is used, NVE must be up, and vPC VTEP peers must have consistent VLAN/VNI/NVE configuration |
# VXLAN_Multicast_Flood_And_Learn_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Flood-and-learn | Remote VTEPs and remote MACs are learned from data-plane traffic, not from BGP EVPN |
| BUM traffic | Broadcast, unknown unicast, and multicast traffic must be replicated to all VTEPs in the same L2VNI |
| Multicast replication | Each L2VNI is mapped to a multicast group; VTEPs join that group so BUM traffic reaches all participating VTEPs |
| VTEP discovery | Initial ARP or unknown traffic is VXLAN encapsulated toward the multicast group; remote VTEPs learn from the packet source |
| MAC learning | Local MACs are learned on access/trunk ports; remote MACs are learned behind `nve1` after traffic flows |
| Underlay requirement | PIM sparse-mode, RP reachability, and unicast routing to VTEP loopbacks must work before flood-and-learn works |
| NVE peer timing | `show nve peers` can be empty before traffic; peers appear after endpoint traffic triggers learning |
| Multicast group scope | Same L2VNI must use the same multicast group on all participating VTEPs |
| No EVPN control plane | There is no BGP EVPN MAC/IP advertisement in this note |
| Related labs | `vxlan-multicast-flood-and-learn-final` |
# VXLAN_Multicast_Flood_And_Learn_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the routed underlay is already stable | All fabric nodes | `show ip route`; `show ip ospf neighbors` | Leaf-spine routing is working and OSPF adjacencies are FULL |
| 2 | Confirm VTEP loopback reachability | Leaf VTEPs | `ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>` | Each VTEP can reach every remote VTEP source loopback |
| 3 | Enable OSPF if the underlay uses OSPF | All fabric nodes | `feature ospf` | OSPF configuration is accepted on routed links and loopbacks |
| 4 | Enable PIM for multicast transport | All fabric nodes | `feature pim` | PIM commands are available on underlay links and loopbacks |
| 5 | Configure the underlay router-ID loopback | All fabric nodes | `interface loopback0` then `ip address <ROUTER_ID>/32` | Each device has a stable /32 for routing identity |
| 6 | Advertise the router-ID loopback into OSPF | All fabric nodes | `ip router ospf <PROCESS_ID> area 0.0.0.0` | Loopback routes are visible across the underlay |
| 7 | Enable PIM sparse-mode on the router-ID or VTEP source loopback | All fabric nodes as applicable | `ip pim sparse-mode` | Loopback can participate in multicast control-plane behavior |
| 8 | Configure routed leaf-spine interfaces | All fabric links | `no switchport`; `ip address <P2P_IP>/<PREFIX>` | Fabric links are Layer 3 point-to-point links |
| 9 | Advertise routed fabric links into OSPF | All fabric links | `ip router ospf <PROCESS_ID> area 0.0.0.0` | Leaf-spine OSPF adjacencies form |
| 10 | Enable PIM sparse-mode on routed fabric links | All fabric links | `ip pim sparse-mode` | PIM neighbors can form across the underlay |
| 11 | Configure a multicast RP | Spine or RP node | `ip pim rp-address <RP_IP> group-list 224.0.0.0/4` | VTEPs know which RP handles VXLAN multicast groups |
| 12 | Optionally define SSM range if used in the design | Spine or RP node | `ip pim ssm range 232.0.0.0/8` | SSM range is explicitly defined if the fabric design requires it |
| 13 | Verify PIM neighbor state | All fabric nodes | `show ip pim neighbor` | PIM neighbors exist on routed leaf-spine links |
| 14 | Verify RP visibility from leaves | Leaf VTEPs | `show ip pim rp` | Leaves show the expected RP for the VXLAN multicast group range |
| 15 | Enable VLAN-to-VNI support | Leaf VTEPs | `feature vn-segment-vlan-based` | `vn-segment` is accepted under VLAN configuration mode |
| 16 | Enable VXLAN NVE support | Leaf VTEPs | `feature nv overlay` | `interface nve1` configuration is accepted |
| 17 | Create the local VLAN | Leaf VTEPs | `vlan <VLAN_ID>` | VLAN exists locally on the VTEP |
| 18 | Bind the VLAN to the L2VNI | Leaf VTEPs | `vn-segment <L2VNI>` | Local VLAN maps to the VXLAN segment |
| 19 | Attach host-facing access interface to the VLAN | Leaf VTEPs | `interface <HOST_PORT>` then `switchport mode access` then `switchport access vlan <VLAN_ID>` | Local host is placed in the stretched VLAN |
| 20 | Create or enter the NVE interface | Leaf VTEPs | `interface nve1` | NVE configuration mode opens |
| 21 | Bind NVE to the VTEP loopback | Leaf VTEPs | `source-interface loopback<VTEP_LOOPBACK_ID>` | VXLAN packets source from the expected VTEP loopback |
| 22 | Add the L2VNI with multicast replication | Leaf VTEPs | `member vni <L2VNI> mcast-group <MULTICAST_GROUP>` | L2VNI uses the specified multicast group for BUM replication |
| 23 | Enable NVE | Leaf VTEPs | `no shutdown` | `nve1` is administratively enabled |
| 24 | Verify VLAN-to-VNI mapping | Leaf VTEPs | `show vxlan` | VLAN displays the expected VN-segment |
| 25 | Verify NVE interface state | Leaf VTEPs | `show nve interface` | `nve1` is up and sourced from the correct loopback |
| 26 | Verify L2VNI flood-and-learn state | Leaf VTEPs | `show nve vni <L2VNI> detail` | VNI is up, mode is data-plane, type is L2, and multicast group is correct |
| 27 | Verify multicast route state for the VXLAN group | Leaf VTEPs | `show ip mroute <MULTICAST_GROUP>` | `nve1` and routed underlay interfaces appear in multicast state as traffic or joins occur |
| 28 | Generate host traffic to trigger learning | Hosts | `ping <REMOTE_HOST_IP>` | ARP and ICMP traffic trigger multicast flooding and remote MAC learning |
| 29 | Verify NVE peers after traffic starts | Leaf VTEPs | `show nve peers`; `show nve peers detail` | Remote VTEPs appear with LearnType `DP` |
| 30 | Verify remote MAC learning | Leaf VTEPs | `show mac address-table dynamic vlan <VLAN_ID>` | Remote host MACs appear as overlay entries through `nve1(<REMOTE_VTEP_IP>)` |
# VXLAN_Multicast_Flood_And_Learn_Skeleton
# SPINE OR RP NODE UNDERLAY MULTICAST
conf t
feature ospf
feature pim
interface loopback0
  description UNDERLAY_ROUTER_ID_OR_RP
  ip address <SPINE_ROUTER_ID_OR_RP_IP>/32
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
ip pim rp-address <RP_IP> group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
end
# LEAF VTEP UNDERLAY AND VXLAN FLOOD-AND-LEARN
conf t
feature ospf
feature pim
feature vn-segment-vlan-based
feature nv overlay
interface loopback0
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
vlan <VLAN_ID>
  name <SEGMENT_NAME>
  vn-segment <L2VNI>
interface <HOST_PORT>
  description TO_<LOCAL_HOST>
  switchport
  switchport mode access
  switchport access vlan <VLAN_ID>
  no shutdown
interface nve1
  source-interface loopback0
  member vni <L2VNI> mcast-group <MULTICAST_GROUP>
  no shutdown
end
# MULTIPLE L2VNIs USING MULTICAST FLOOD-AND-LEARN
conf t
vlan 10
  name TENANT_A_WEB
  vn-segment 10010
vlan 20
  name TENANT_A_APP
  vn-segment 10020
interface nve1
  source-interface loopback0
  member vni 10010 mcast-group 230.1.1.1
  member vni 10020 mcast-group 230.1.1.2
  no shutdown
end
# vPC VTEP ADDITIONAL RULES
# If the VTEPs are a vPC pair:
# - VLAN-to-VNI mapping must match on both vPC peers.
# - VNI-to-multicast-group mapping must match on both vPC peers.
# - The NVE source loopback secondary anycast VTEP IP must match on both peers.
# - vPC peer-link and consistency must be clean before blaming VXLAN.
# VXLAN_Multicast_Flood_And_Learn_Verification_Commands
# Underlay routing
show ip route
show ip route <REMOTE_VTEP_IP>
show ip ospf neighbors
show ip ospf interface brief
ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>
# PIM and RP
show ip pim neighbor
show ip pim rp
show ip mroute
show ip mroute <MULTICAST_GROUP>
# VXLAN features
show feature | include nv
show feature | include vn-segment
show feature | include pim
# VLAN-to-VNI mapping
show vlan id <VLAN_ID>
show vxlan
show running-config vlan <VLAN_ID>
# NVE interface and VNI state
show running-config interface nve1
show nve interface
show nve vni
show nve vni <L2VNI> detail
show nve internal vni <L2VNI>
# NVE peers after traffic
show nve peers
show nve peers detail
# MAC learning
show mac address-table dynamic vlan <VLAN_ID>
show mac address-table address <HOST_MAC>
# Host-facing interface
show running-config interface <HOST_PORT>
show interface <HOST_PORT> switchport
show interface trunk
# vPC VTEP checks, only if used
show vpc brief
show vpc consistency-parameters global
show port-channel summary
# VXLAN_Multicast_Flood_And_Learn_Rollback
# REMOVE ONE L2VNI MULTICAST MAPPING FROM NVE
conf t
interface nve1
  no member vni <L2VNI>
end
# REMOVE VLAN-TO-VNI MAPPING BUT KEEP VLAN
conf t
vlan <VLAN_ID>
  no vn-segment <L2VNI>
end
# REMOVE LOCAL VLAN
conf t
no vlan <VLAN_ID>
end
# REMOVE HOST-FACING ACCESS VLAN CONFIGURATION
conf t
interface <HOST_PORT>
  shutdown
  no switchport access vlan <VLAN_ID>
end
# SHUT DOWN NVE WITHOUT REMOVING ALL CONFIGURATION
conf t
interface nve1
  shutdown
end
# REMOVE NVE COMPLETELY
conf t
interface nve1
  shutdown
  no member vni <L2VNI>
  no source-interface loopback<VTEP_LOOPBACK_ID>
exit
no interface nve1
end
# REMOVE PIM FROM ONE UNDERLAY LINK
conf t
interface <UNDERLAY_INTERFACE>
  no ip pim sparse-mode
end
# REMOVE PIM FROM VTEP LOOPBACK
conf t
interface loopback<VTEP_LOOPBACK_ID>
  no ip pim sparse-mode
end
# REMOVE RP CONFIGURATION
conf t
no ip pim rp-address <RP_IP> group-list 224.0.0.0/4
end
# REMOVE MULTICAST FEATURE ONLY IF NO OTHER MULTICAST SERVICES EXIST
conf t
no feature pim
end
# REMOVE VXLAN FEATURES ONLY IF NO OTHER VXLAN SERVICES EXIST
conf t
no feature nv overlay
no feature vn-segment-vlan-based
end
# VXLAN_Multicast_Flood_And_Learn_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `show nve interface` is down | NVE is shut, source loopback is down, or source-interface is wrong | `show nve interface`; `show running-config interface nve1` | Configure correct `source-interface loopback<VTEP_LOOPBACK_ID>` and `no shutdown` |
| VTEP loopbacks cannot ping | Underlay routing failure | `show ip route <REMOTE_VTEP_IP>`; `show ip ospf neighbors` | Fix OSPF, interface addressing, or routed leaf-spine links |
| `show ip pim neighbor` is empty | PIM missing on routed fabric links or `feature pim` not enabled | `show feature | include pim`; `show running-config interface <UNDERLAY_INTERFACE>` | Enable `feature pim` and `ip pim sparse-mode` on routed underlay links |
| `show ip pim rp` does not show the expected RP | RP not configured, RP not reachable, or group-list mismatch | `show ip pim rp`; `show ip route <RP_IP>` | Configure `ip pim rp-address <RP_IP> group-list 224.0.0.0/4` and advertise the RP loopback |
| `show nve vni` does not show the L2VNI | VNI not configured under `interface nve1` | `show running-config interface nve1` | Add `member vni <L2VNI> mcast-group <MULTICAST_GROUP>` |
| L2VNI shows wrong multicast group | VNI-to-group mapping mismatch between VTEPs | `show nve vni <L2VNI> detail` on each leaf | Use the same multicast group for the same L2VNI on all participating VTEPs |
| `show vxlan` does not show the VLAN/VNI pair | VLAN is missing `vn-segment` | `show running-config vlan <VLAN_ID>` | Configure `vn-segment <L2VNI>` under the VLAN |
| `show ip mroute <MULTICAST_GROUP>` lacks NVE or underlay OIL entries | PIM/RP issue, no VNI join, or no traffic yet | `show ip mroute <MULTICAST_GROUP>`; `show nve vni` | Fix PIM/RP, confirm NVE VNI state, then generate host traffic |
| `show nve peers` is empty before traffic | Expected behavior in flood-and-learn before data-plane learning | `show nve peers`; `show mac address-table dynamic vlan <VLAN_ID>` | Generate ARP or ICMP traffic between hosts in the same L2VNI |
| `show nve peers` stays empty after traffic | Multicast flood is not reaching remote VTEPs | `show ip mroute <MULTICAST_GROUP>`; `show ip pim neighbor`; `show nve vni` | Fix PIM adjacency, RP state, VNI multicast group, or VTEP loopback reachability |
| Local MAC appears but remote MAC does not | Remote flood/learn did not complete, host traffic is absent, or remote host port is wrong | `show mac address-table dynamic vlan <VLAN_ID>`; `show nve peers` | Verify remote host VLAN, generate traffic, and fix multicast replication |
| Remote MAC appears as local port instead of overlay | VLAN leak, wrong cabling, or host connected to wrong leaf/interface | `show mac address-table address <HOST_MAC>` | Correct host attachment and VLAN placement |
| vPC VTEP pair has inconsistent behavior | vPC peer VLAN/VNI/multicast/NVE source mismatch | `show vpc brief`; `show vpc consistency-parameters global`; `show nve vni` | Make VLAN-to-VNI, VNI-to-group, and NVE source configuration identical where required |
| MACs age out and NVE peer disappears | Expected data-plane learning behavior when traffic stops | `show nve peers`; `show mac address-table dynamic vlan <VLAN_ID>` | Generate traffic again or move to EVPN control-plane learning if persistent control-plane state is required |
| EVPN commands show nothing | This note does not use BGP EVPN | `show running-config bgp`; `show bgp l2vpn evpn summary` | Do not troubleshoot EVPN for this lab unless the design intentionally uses BGP EVPN |
##### Source_Basis
# VXLAN_Multicast_Flood_And_Learn_Mental_Model
# VXLAN_Multicast_Flood_And_Learn_Configuration_Checklist
# VXLAN_Multicast_Flood_And_Learn_Skeleton
# VXLAN_Multicast_Flood_And_Learn_Verification_Commands
# VXLAN_Multicast_Flood_And_Learn_Rollback
# VXLAN_Multicast_Flood_And_Learn_Failure_Checks
# index of each title throughout note, not in table format

