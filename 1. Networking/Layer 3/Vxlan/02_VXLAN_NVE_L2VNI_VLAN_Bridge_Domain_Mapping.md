
VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping.md

VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Data Center VXLAN EVPN and NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and EVPN consistency checks to `NS-XO_combined_epubs.md` |
| `CiscoPress_combined_part1.md` | DCCOR Ch 3, VXLAN Overview, VXLAN Tunnel Endpoint, Virtual Network Identifier, VXLAN Configurations and Verifications | Establishes VLAN-to-VNI mapping, VTEP function, NVE source-interface behavior, `vlan <id>`, `vn-segment <vni>`, `member vni`, `show nve vni`, and `show vxlan` verification |
| `NS-OS_combined_PDFs.md` | VXLAN Flood-and-Learn article, Configure and Verify sections | Provides NX-OS examples for `feature vn-segment-vlan-based`, `feature nv overlay`, VLAN-to-VNID mapping, `interface nve1`, `source-interface loopback0`, `member vni <vni> mcast-group <group>`, `show nve interface`, `show nve vni`, `show nve peers`, and `show ip mroute` |
| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Provides operational checks for L2VNI replication method, NVE source loopback, vPC VTEP consistency, VLAN-to-VN-segment consistency, and correct use of `suppress-arp` only on eligible L2VNIs |
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Local VLAN | The access-side Layer 2 segment where hosts, trunks, or server-facing ports live |
| L2VNI | The overlay bridge-domain identifier carried in the VXLAN header |
| VLAN-to-VNI mapping | `vlan <VLAN_ID>` plus `vn-segment <L2VNI>` tells the VTEP which overlay segment represents that local VLAN |
| NVE interface | `interface nve1` is the logical VXLAN tunnel interface that performs encapsulation and decapsulation |
| NVE source-interface | The loopback used as the VTEP source IP; remote VTEPs must be able to route to it |
| NVE member VNI | `member vni <L2VNI>` activates that VNI under the NVE interface |
| Replication method | Each L2VNI needs a BUM replication method: multicast group, static ingress replication, or EVPN/BGP ingress replication |
| Bridge-domain behavior | Hosts in the same VLAN/L2VNI are bridged across VTEPs as if they share one Layer 2 segment |
| What this note is not | This note does not build BGP EVPN, L3VNI routing, anycast gateway, or ARP suppression logic |
| Related labs | `vxlan-one-spine-two-leafs-final`; `vxlan-unicast-final`; `vxlan-multicast-flood-and-learn-final`; `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final`; `vxlan-bgp-evpn-arp-suppression-final` |
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm underlay and VTEP loopback reachability are already working | Leaf VTEPs | `ping <remote-vtep-loopback> source <local-vtep-loopback>` | Remote VTEP loopbacks are reachable before VLAN/VNI mapping is configured |
| 2 | Confirm the local VTEP source loopback exists | Leaf VTEPs | `show running-config interface loopback<VTEP_LOOPBACK_ID>` | Loopback has the expected /32 VTEP source IP |
| 3 | Enable VXLAN VLAN-to-VNI feature support | Leaf VTEPs | `feature vn-segment-vlan-based` | NX-OS accepts `vn-segment` under VLAN configuration mode |
| 4 | Enable the VXLAN NVE overlay feature | Leaf VTEPs | `feature nv overlay` | NX-OS accepts `interface nve1` and NVE member commands |
| 5 | Define the VLAN-to-L2VNI plan before configuring devices | Design worksheet | `VLAN <VLAN_ID> = L2VNI <L2VNI>` | Every extended VLAN has one planned L2VNI, and the same tenant segment uses the same L2VNI on every participating VTEP |
| 6 | Create the local VLAN | Leaf VTEPs | `vlan <VLAN_ID>` | VLAN exists in the local VLAN database |
| 7 | Optionally name the VLAN for operational clarity | Leaf VTEPs | `name <TENANT_OR_SEGMENT_NAME>` | `show vlan id <VLAN_ID>` displays a meaningful name |
| 8 | Bind the VLAN to the L2VNI | Leaf VTEPs | `vn-segment <L2VNI>` | VLAN is mapped to the overlay L2VNI |
| 9 | Repeat VLAN-to-VNI mapping for every local stretched Layer 2 segment | Leaf VTEPs | `vlan <VLAN_ID>` then `vn-segment <L2VNI>` | Each local bridge-domain has a unique intended L2VNI |
| 10 | Confirm the access or trunk port carries the local VLAN | Leaf VTEPs | `show running-config interface <host-facing-interface>` | Host-facing interface is access VLAN `<VLAN_ID>` or trunk allows VLAN `<VLAN_ID>` |
| 11 | Create or enter the NVE interface | Leaf VTEPs | `interface nve1` | NVE configuration mode opens |
| 12 | Bind NVE to the VTEP loopback | Leaf VTEPs | `source-interface loopback<VTEP_LOOPBACK_ID>` | NVE uses the correct local VTEP source address |
| 13 | Add the L2VNI to the NVE interface using multicast replication when flood-and-learn is used | Leaf VTEPs | `member vni <L2VNI> mcast-group <MULTICAST_GROUP>` | L2VNI is active under NVE with a multicast group for BUM replication |
| 14 | Add the L2VNI to the NVE interface using EVPN/BGP ingress replication when the EVPN lab uses ingress replication | Leaf VTEPs | `member vni <L2VNI>` then `ingress-replication protocol bgp` | L2VNI is active under NVE with BGP signaled replication |
| 15 | Do not configure `suppress-arp` in this baseline note | Leaf VTEPs | No command | ARP suppression stays reserved for the dedicated ARP suppression and anycast gateway mechanism note |
| 16 | Enable the NVE interface | Leaf VTEPs | `no shutdown` | NVE interface is administratively up |
| 17 | Verify VLAN-to-VNI mapping | Leaf VTEPs | `show vxlan` | VLAN shows the expected VN-segment value |
| 18 | Verify L2VNI state under NVE | Leaf VTEPs | `show nve vni` | VNI appears as `Up`, type `L2`, and references the expected bridge-domain or VLAN |
| 19 | Verify NVE source state | Leaf VTEPs | `show nve interface` | NVE interface is up and sourced from the expected loopback |
| 20 | Verify remote VTEP peer state after traffic or EVPN signaling exists | Leaf VTEPs | `show nve peers` | Remote VTEP peer appears with the expected learn type, such as `DP` for flood-and-learn or `CP` for EVPN |
| 21 | Verify local MAC learning in the mapped VLAN | Leaf VTEPs | `show mac address-table dynamic vlan <VLAN_ID>` | Local host MACs appear on local access/trunk ports |
| 22 | Verify overlay MAC learning after endpoint traffic | Leaf VTEPs | `show mac address-table dynamic vlan <VLAN_ID>` | Remote host MACs appear as overlay entries after data-plane learning or EVPN control-plane learning |
| 23 | Verify endpoint reachability inside the same L2VNI | Hosts | `ping <remote-host-ip>` | Hosts in the same VLAN/L2VNI can reach each other across the VXLAN fabric |
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Skeleton
# BASE L2VNI MAPPING ON EACH LEAF VTEP
conf t
feature vn-segment-vlan-based
feature nv overlay
vlan <VLAN_ID>
  name <TENANT_OR_SEGMENT_NAME>
  vn-segment <L2VNI>
interface <HOST_FACING_INTERFACE>
  description TO_<HOST_OR_SERVER>
  switchport
  switchport mode access
  switchport access vlan <VLAN_ID>
  no shutdown
interface nve1
  source-interface loopback<VTEP_LOOPBACK_ID>
  no shutdown
end
# OPTION A: MULTICAST FLOOD-AND-LEARN L2VNI MEMBER
conf t
interface nve1
  member vni <L2VNI> mcast-group <MULTICAST_GROUP>
end
# OPTION B: EVPN/BGP INGRESS REPLICATION L2VNI MEMBER
conf t
interface nve1
  member vni <L2VNI>
    ingress-replication protocol bgp
end
# MULTIPLE VLAN/VNI EXAMPLE
conf t
vlan 10
  name TENANT_A_WEB
  vn-segment 160010
vlan 20
  name TENANT_A_APP
  vn-segment 160020
interface nve1
  source-interface loopback1
  member vni 160010 mcast-group 231.1.1.1
  member vni 160020 mcast-group 231.1.1.2
  no shutdown
end
# vPC VTEP NOTE
# If this is a vPC VTEP pair:
# - VLAN-to-VNI mapping must match on both vPC peers.
# - NVE source loopback secondary anycast VTEP IP must match on both vPC peers.
# - NVE member VNI configuration must match on both vPC peers.
# - Check vPC consistency before blaming VXLAN.
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Verification_Commands
# Feature state
show feature | include nv
show feature | include vn-segment
# VLAN and VNI mapping
show vlan id <VLAN_ID>
show vxlan
show running-config vlan <VLAN_ID>
# Host-facing VLAN attachment
show running-config interface <HOST_FACING_INTERFACE>
show interface <HOST_FACING_INTERFACE> switchport
show interface trunk
# NVE interface state
show nve interface
show running-config interface nve1
# NVE VNI state
show nve vni
show nve vni <L2VNI>
# NVE peer state
show nve peers
show nve peers detail
# MAC learning
show mac address-table dynamic vlan <VLAN_ID>
show mac address-table address <HOST_MAC>
# Multicast replication checks, only if using mcast-group
show ip pim neighbor
show ip pim rp
show ip mroute <MULTICAST_GROUP>
# EVPN replication checks, only if using BGP EVPN ingress replication
show bgp l2vpn evpn summary
show bgp l2vpn evpn
show nve peers
# vPC VTEP consistency checks, only if using vPC VTEPs
show vpc
show vpc consistency-parameters global
show port-channel summary
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Rollback
# REMOVE A SINGLE L2VNI FROM NVE
conf t
interface nve1
  no member vni <L2VNI>
end
# REMOVE VLAN-TO-VNI MAPPING BUT KEEP THE VLAN
conf t
vlan <VLAN_ID>
  no vn-segment <L2VNI>
end
# REMOVE THE LOCAL VLAN ENTIRELY
conf t
no vlan <VLAN_ID>
end
# REMOVE HOST-FACING VLAN ATTACHMENT
conf t
interface <HOST_FACING_INTERFACE>
  shutdown
  no switchport access vlan <VLAN_ID>
end
# SHUT DOWN NVE WITHOUT REMOVING CONFIGURATION
conf t
interface nve1
  shutdown
end
# REMOVE NVE INTERFACE COMPLETELY
conf t
interface nve1
  shutdown
  no member vni <L2VNI>
  no source-interface loopback<VTEP_LOOPBACK_ID>
exit
no interface nve1
end
# REMOVE VXLAN FEATURES ONLY IF NO OTHER VXLAN SERVICES EXIST
conf t
no feature nv overlay
no feature vn-segment-vlan-based
end
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `vn-segment` command is rejected under VLAN config | VXLAN VLAN segment feature is not enabled | `show feature | include vn-segment` | Enable `feature vn-segment-vlan-based` |
| `interface nve1` or NVE commands are rejected | NVE overlay feature is not enabled | `show feature | include nv` | Enable `feature nv overlay` |
| `show vxlan` does not show the VLAN/VNI pair | VLAN exists but no `vn-segment` is configured, or wrong VLAN was configured | `show running-config vlan <VLAN_ID>` | Add `vn-segment <L2VNI>` under the correct VLAN |
| `show nve vni` does not show the VNI | VNI is mapped under VLAN but not added as an NVE member | `show running-config interface nve1` | Add `member vni <L2VNI>` with the correct replication method |
| L2VNI shows `Down` | NVE source-interface is missing, NVE is shutdown, underlay is broken, or replication method is missing | `show nve interface`; `show nve vni` | Configure `source-interface`, `no shutdown`, underlay reachability, and a valid replication method |
| NVE interface is down | Source loopback is down or not configured | `show running-config interface nve1`; `show ip interface brief vrf all` | Restore the VTEP source loopback and bind it under `interface nve1` |
| Remote host cannot ping across same L2VNI | VLAN/VNI mismatch between VTEPs | `show vxlan` on both leaves | Use the same L2VNI for the same stretched segment across all participating leaves |
| Local host MAC appears, but remote MAC never appears | No remote VTEP peer, no BUM replication, no EVPN signaling, or no traffic | `show nve peers`; `show nve vni`; `show mac address-table dynamic vlan <VLAN_ID>` | Fix replication or EVPN control plane, then generate host traffic |
| `show nve peers` is empty in flood-and-learn | No endpoint traffic yet, multicast underlay broken, or VNI multicast group mismatch | `show ip mroute <MULTICAST_GROUP>`; `show nve vni` | Verify multicast group consistency and PIM/RP state, then generate traffic |
| `show nve peers` is empty in EVPN mode | BGP EVPN not established or L2VNI not advertised | `show bgp l2vpn evpn summary`; `show nve vni` | Fix BGP EVPN peering and L2VNI EVPN configuration in the EVPN mechanism note |
| vPC peer VLAN/VNI inconsistency | vPC peers have different VLAN-to-VNI mappings or NVE member configuration | `show vpc consistency-parameters global` | Make VLAN-to-VNI and NVE member VNI configuration identical on both vPC peers |
| Hosts in the same local VLAN cannot reach each other before VXLAN | Host-facing VLAN or trunk problem, not VXLAN | `show interface <interface> switchport`; `show vlan id <VLAN_ID>` | Fix local access/trunk VLAN attachment before troubleshooting overlay |
| ARP suppression breaks connectivity | `suppress-arp` was configured before anycast gateway and EVPN MAC/IP learning were ready | `show running-config interface nve1`; `show nve vni` | Remove `suppress-arp` here and handle it only in the ARP suppression mechanism note |
##### Source_Basis
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Mental_Model
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Configuration_Checklist
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Skeleton
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Verification_Commands
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Rollback
# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping_Failure_Checks
# index of each title throughout note, not in table format
