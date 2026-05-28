
VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout.md

VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Data Center VXLAN EVPN and NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and consistency checks to `NS-XO_combined_epubs.md` |
| `CiscoPress_combined_part1.md` | DCCOR Ch 3, VXLAN MPBGP EVPN Control Plane | Supports the model that EVPN distributes VTEP and MAC reachability through BGP and that route reflectors reduce the need for full-mesh BGP sessions between VTEPs |
| `CiscoPress_combined_part1.md` | BGP Route Reflection | Supports the route-reflector mental model, route-reflector clients, iBGP split-horizon exception, redundancy, cluster behavior, and physical/logical topology alignment |
| `CiscoPress_combined_part2.md` | L2VPN EVPN service verification sections | Supports EVPN verification using the L2VPN EVPN address family and EVPN route-type visibility |
| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Supports operational checks for NVE source loopbacks, L2VNI replication method, `advertise l2vpn evpn` on older NX-OS versions, vPC consistency, and EVPN/VXLAN fabric sanity checks |
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Full-mesh EVPN problem | Without route reflectors, every VTEP leaf needs EVPN BGP sessions to every other VTEP leaf |
| Route reflector | A spine or dedicated control-plane node that receives EVPN routes from leaf clients and reflects them to other leaf clients |
| Route-reflector client | A leaf VTEP configured under the spine's BGP EVPN address family with `route-reflector-client` |
| Spine role | In this note, the spine is a BGP EVPN route reflector, not a VTEP, unless the lab explicitly makes it one |
| Leaf role | Leaves are VTEPs that originate and consume EVPN routes for local L2VNIs and endpoints |
| EVPN address family | `address-family l2vpn evpn` carries MAC/IP, IMET, Ethernet segment, and other EVPN route types |
| Extended communities | EVPN depends on route targets and EVPN attributes, so `send-community extended` is mandatory on EVPN peers |
| Route Type 2 | MAC/IP route that tells remote VTEPs where endpoints live |
| Route Type 3 | Inclusive multicast Ethernet tag route that tells VTEPs which remote VTEPs participate in a VNI for BUM replication |
| Scaleout rule | Adding a new leaf should require BGP peering to the route reflectors, not a full mesh to every existing leaf |
| Redundancy rule | Use at least two route reflectors when possible so one spine/control-plane node is not a single point of failure |
| Related labs | `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final` |
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm leaf-spine underlay is stable | All fabric nodes | `show ip route`; `show ip ospf neighbors` | All routed fabric links are up and underlay adjacencies are stable |
| 2 | Confirm loopback reachability between leaves and spines | Leaves and spines | `ping <REMOTE_LOOPBACK> source <LOCAL_LOOPBACK>` | BGP loopbacks and VTEP loopbacks are reachable across the underlay |
| 3 | Identify route-reflector nodes | Design worksheet | `Spine1 = RR`; `Spine2 = RR` | Spines or dedicated nodes are assigned as EVPN route reflectors |
| 4 | Identify route-reflector clients | Design worksheet | `Leaf1-Leaf4 = RR clients` | Leaf VTEPs are assigned as clients of both route reflectors |
| 5 | Confirm spines are not being configured as VTEPs unless required | Spine RRs | `show running-config interface nve1` | Pure RR spines do not need NVE, VLAN-to-VNI mapping, or host-facing VLANs |
| 6 | Enable BGP on route reflectors | Spine RRs | `feature bgp` | NX-OS accepts BGP configuration |
| 7 | Enable BGP on leaf VTEPs | Leaf VTEPs | `feature bgp` | Leaf VTEPs can build EVPN BGP sessions |
| 8 | Enable VXLAN EVPN overlay on leaf VTEPs | Leaf VTEPs | `nv overlay evpn` | Leaf VTEPs can use EVPN as the VXLAN control plane |
| 9 | Configure BGP process on each route reflector | Spine RRs | `router bgp <ASN>` then `router-id <SPINE_ROUTER_ID>` | RR has a stable BGP process and router ID |
| 10 | Configure BGP process on each leaf | Leaf VTEPs | `router bgp <ASN>` then `router-id <LEAF_ROUTER_ID>` | Leaf has a stable BGP process and router ID |
| 11 | Configure each leaf as a BGP neighbor on RR1 | RR1 | `neighbor <LEAF_LOOPBACK>` then `remote-as <ASN>` | RR1 has every leaf defined as an iBGP neighbor |
| 12 | Source RR1 BGP sessions from its loopback | RR1 | `neighbor <LEAF_LOOPBACK>` then `update-source loopback<RR_LOOPBACK_ID>` | RR1 uses a stable loopback source for EVPN peering |
| 13 | Activate L2VPN EVPN address family toward each leaf on RR1 | RR1 | `neighbor <LEAF_LOOPBACK>` then `address-family l2vpn evpn` | RR1 can exchange EVPN routes with the leaf |
| 14 | Enable extended community sending toward each leaf on RR1 | RR1 | `send-community extended` | EVPN route targets and extended attributes are preserved |
| 15 | Mark each leaf as a route-reflector client on RR1 | RR1 | `route-reflector-client` | RR1 reflects EVPN routes between leaf clients |
| 16 | Repeat route-reflector client configuration on RR2 | RR2 | `neighbor <LEAF_LOOPBACK>` then `address-family l2vpn evpn` then `route-reflector-client` | RR2 provides redundant EVPN route reflection |
| 17 | Configure each leaf neighbor to RR1 | Leaf VTEPs | `neighbor <RR1_LOOPBACK>` then `remote-as <ASN>` | Leaf has an iBGP EVPN neighbor to RR1 |
| 18 | Source leaf BGP session to RR1 from loopback | Leaf VTEPs | `neighbor <RR1_LOOPBACK>` then `update-source loopback<LEAF_BGP_LOOPBACK_ID>` | Leaf uses stable loopback sourcing for RR1 |
| 19 | Activate L2VPN EVPN toward RR1 | Leaf VTEPs | `neighbor <RR1_LOOPBACK>` then `address-family l2vpn evpn` | Leaf can exchange EVPN routes with RR1 |
| 20 | Send extended communities toward RR1 | Leaf VTEPs | `send-community extended` | Leaf sends EVPN route targets and attributes to RR1 |
| 21 | Repeat leaf neighbor configuration toward RR2 | Leaf VTEPs | `neighbor <RR2_LOOPBACK>` then `address-family l2vpn evpn` | Leaf has redundant EVPN peering to RR2 |
| 22 | Configure EVPN L2VNI instance on each participating leaf | Leaf VTEPs | `evpn` then `vni <L2VNI> l2` | Leaf has an EVPN MAC-VRF for the L2VNI |
| 23 | Configure RD for each L2VNI | Leaf VTEPs | `rd auto` | Each VTEP originates unique EVPN routes for the L2VNI |
| 24 | Configure RT import/export for each L2VNI | Leaf VTEPs | `route-target both auto` | Leaves sharing the same L2VNI import each other's EVPN routes |
| 25 | Confirm VLAN-to-VNI mapping exists on leaves that host the VLAN | Leaf VTEPs | `show vxlan` | VLAN maps to the intended L2VNI |
| 26 | Configure NVE for BGP host reachability | Leaf VTEPs | `interface nve1` then `host-reachability protocol bgp` | NVE uses EVPN control-plane learning |
| 27 | Configure L2VNI BGP ingress replication | Leaf VTEPs | `member vni <L2VNI>` then `ingress-replication protocol bgp` | EVPN builds the remote VTEP replication list |
| 28 | Verify EVPN BGP sessions to both route reflectors | Leaf VTEPs | `show bgp l2vpn evpn summary` | Leaf shows established sessions to RR1 and RR2 |
| 29 | Verify route reflectors see all leaf clients | Spine RRs | `show bgp l2vpn evpn summary` | Each RR shows established EVPN sessions to all leaves |
| 30 | Verify EVPN IMET routes are reflected | Leaf VTEPs | `show bgp l2vpn evpn route-type imet` | Leaf learns Type 3 routes from other leaves through the RRs |
| 31 | Generate endpoint traffic behind leaves | Hosts | `ping <REMOTE_HOST_IP>` | Local MACs are learned and advertised through EVPN |
| 32 | Verify MAC/IP routes are reflected | Leaf VTEPs | `show bgp l2vpn evpn route-type macip` | Leaf learns Type 2 routes for remote endpoints through the RRs |
| 33 | Verify NVE peers are control-plane learned | Leaf VTEPs | `show nve peers` | Remote VTEPs appear with LearnType `CP` |
| 34 | Verify endpoint forwarding across scaled fabric | Hosts | `ping <REMOTE_HOST_IP>` | Hosts behind different leaves in the same L2VNI can communicate |
| 35 | Add a new leaf using only RR peering | New Leaf | Configure EVPN BGP neighbors to RR1 and RR2 only | New leaf joins the EVPN fabric without full-mesh peering to every existing leaf |
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Skeleton
# RR1 SPINE, PURE EVPN ROUTE REFLECTOR
conf t
feature bgp
router bgp <ASN>
  router-id <RR1_ROUTER_ID>
  neighbor <LEAF1_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF2_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF3_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF4_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
end
# RR2 SPINE, PURE EVPN ROUTE REFLECTOR
conf t
feature bgp
router bgp <ASN>
  router-id <RR2_ROUTER_ID>
  neighbor <LEAF1_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF2_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF3_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF4_LOOPBACK>
    remote-as <ASN>
    update-source loopback<RR_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
end
# LEAF VTEP CLIENT
conf t
feature bgp
feature nv overlay
feature vn-segment-vlan-based
nv overlay evpn
vlan <VLAN_ID>
  name <SEGMENT_NAME>
  vn-segment <L2VNI>
evpn
  vni <L2VNI> l2
    rd auto
    route-target both auto
router bgp <ASN>
  router-id <LEAF_ROUTER_ID>
  neighbor <RR1_LOOPBACK>
    remote-as <ASN>
    update-source loopback<LEAF_BGP_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
  neighbor <RR2_LOOPBACK>
    remote-as <ASN>
    update-source loopback<LEAF_BGP_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
interface nve1
  source-interface loopback<VTEP_LOOPBACK_ID>
  host-reachability protocol bgp
  member vni <L2VNI>
    ingress-replication protocol bgp
  no shutdown
end
# ADDING A NEW LEAF TO THE FABRIC
# 1. Give the new leaf underlay loopback reachability.
# 2. Configure the leaf as an EVPN neighbor on RR1 and RR2.
# 3. Configure RR1 and RR2 as EVPN neighbors on the new leaf.
# 4. Add only the L2VNIs that the new leaf actually hosts.
# 5. Do not build leaf-to-leaf EVPN sessions.
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Verification_Commands
# Underlay reachability
show ip route <REMOTE_LOOPBACK>
ping <REMOTE_LOOPBACK> source <LOCAL_LOOPBACK>
traceroute <REMOTE_LOOPBACK> source <LOCAL_LOOPBACK>
# BGP feature and process
show feature | include bgp
show running-config bgp
# EVPN BGP summary
show bgp l2vpn evpn summary
show bgp l2vpn evpn neighbors
show bgp l2vpn evpn neighbors <NEIGHBOR_IP>
# Route-reflector client configuration
show running-config bgp | section "address-family l2vpn evpn"
show bgp l2vpn evpn summary
# EVPN route visibility
show bgp l2vpn evpn
show bgp l2vpn evpn route-type imet
show bgp l2vpn evpn route-type macip
show bgp l2vpn evpn vni-id <L2VNI>
# Leaf EVPN MAC-VRF configuration
show running-config evpn
show running-config evpn | section "vni <L2VNI>"
# NVE state on leaves
show nve interface
show nve vni
show nve vni <L2VNI> detail
show nve peers
show nve peers detail
# VLAN/VNI state on leaves
show vxlan
show vlan id <VLAN_ID>
show running-config vlan <VLAN_ID>
# MAC and L2 route programming
show mac address-table dynamic vlan <VLAN_ID>
show l2route evpn mac all
show l2route evpn mac-ip all
# Endpoint test
ping <REMOTE_HOST_IP>
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Rollback
# REMOVE ONE LEAF AS RR CLIENT FROM RR1 OR RR2
conf t
router bgp <ASN>
  neighbor <LEAF_LOOPBACK>
    address-family l2vpn evpn
      no route-reflector-client
end
# REMOVE ONE LEAF EVPN NEIGHBOR FROM RR
conf t
router bgp <ASN>
  no neighbor <LEAF_LOOPBACK>
end
# REMOVE ONE RR NEIGHBOR FROM LEAF
conf t
router bgp <ASN>
  no neighbor <RR_LOOPBACK>
end
# REMOVE L2VPN EVPN ADDRESS FAMILY FROM ONE NEIGHBOR
conf t
router bgp <ASN>
  neighbor <NEIGHBOR_LOOPBACK>
    no address-family l2vpn evpn
end
# REMOVE ONE L2VNI FROM LEAF NVE
conf t
interface nve1
  no member vni <L2VNI>
end
# REMOVE EVPN MAC-VRF FOR ONE L2VNI
conf t
evpn
  no vni <L2VNI> l2
end
# REMOVE VLAN-TO-VNI MAPPING
conf t
vlan <VLAN_ID>
  no vn-segment <L2VNI>
end
# SHUT DOWN NVE ON A LEAF WITHOUT REMOVING ALL CONFIGURATION
conf t
interface nve1
  shutdown
end
# REMOVE BGP FROM A PURE RR ONLY IF NO OTHER BGP SERVICES EXIST
conf t
no router bgp <ASN>
no feature bgp
end
# REMOVE VXLAN EVPN FROM A LEAF ONLY IF NO OTHER VXLAN EVPN SERVICES EXIST
conf t
no nv overlay evpn
no feature nv overlay
no feature vn-segment-vlan-based
end
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Leaf EVPN session to RR is idle | Loopback reachability failure, wrong remote AS, wrong update-source, or BGP feature missing | `show bgp l2vpn evpn summary`; `ping <RR_LOOPBACK> source <LEAF_LOOPBACK>` | Fix underlay reachability, AS number, and `update-source` |
| RR sees only some leaves | Missing neighbor statements or underlay reachability to missing leaf loopbacks | `show running-config bgp`; `show bgp l2vpn evpn summary` | Add missing leaf neighbors and restore loopback reachability |
| Leaves establish to RR but do not learn each other's EVPN routes | `route-reflector-client` missing on RR | `show running-config bgp | section "address-family l2vpn evpn"` | Add `route-reflector-client` under each leaf neighbor on the RR |
| EVPN routes arrive without usable RT behavior | Extended communities not being sent | `show bgp l2vpn evpn neighbors <NEIGHBOR_IP>` | Add `send-community extended` under the EVPN address family |
| Leaf learns Type 3 from one RR only | Second RR session is down or missing address-family activation | `show bgp l2vpn evpn summary`; `show bgp l2vpn evpn route-type imet` | Fix the second RR neighbor and activate `address-family l2vpn evpn` |
| NVE peers are missing even though BGP is established | Type 3 IMET routes not reflected or L2VNI not active on remote leaves | `show bgp l2vpn evpn route-type imet`; `show nve vni <L2VNI> detail` | Fix RR reflection, RT import/export, and remote L2VNI configuration |
| Remote MAC routes are missing | Remote leaf has no local MAC, RT mismatch, or Type 2 route is not reflected | `show bgp l2vpn evpn route-type macip`; `show mac address-table dynamic vlan <VLAN_ID>` | Generate endpoint traffic, fix VLAN attachment, and align RTs |
| Remote MAC routes exist in BGP but not in MAC table | NVE VNI down, VLAN/VNI mismatch, or L2RIB programming issue | `show nve vni <L2VNI> detail`; `show vxlan`; `show l2route evpn mac all` | Fix VNI state and VLAN-to-VNI mapping |
| Spine has no NVE peers | Expected for a pure route-reflector spine | `show running-config interface nve1`; `show bgp l2vpn evpn summary` | Do not troubleshoot NVE on pure RR spines; verify BGP EVPN routes instead |
| Leaf-to-leaf BGP sessions are absent | Expected in RR design | `show bgp l2vpn evpn summary` | Leaf should peer to RRs, not every other leaf |
| Adding a new leaf requires many sessions | Design drift toward full mesh | `show running-config bgp` on leaves | Peer the new leaf only to RR1 and RR2, then add it as an RR client |
| One RR failure causes EVPN outage | Leaves are peered to only one RR | `show bgp l2vpn evpn summary` | Peer every leaf to at least two route reflectors |
| Routes loop or are unexpectedly rejected between RRs | RR cluster-ID or redundant RR design problem | `show bgp l2vpn evpn route-type macip detail`; `show running-config bgp` | Review RR cluster design and avoid unnecessary RR-to-RR reflection unless required |
| eBGP EVPN scaleout fails to retain remote VNIs | RR/spine does not retain unmatched RTs | `show bgp l2vpn evpn`; `show running-config bgp` | Use `retain route-target all` only in the EVPN design that requires it |
| eBGP EVPN routes fail due to AS-path peer checks | Peer AS appears in the path and is blocked | `show bgp l2vpn evpn neighbors <NEIGHBOR_IP>` | Use `disable-peer-as-check` only when the eBGP EVPN design requires it |
| ARP suppression is blamed for RR issue | Wrong troubleshooting layer | `show bgp l2vpn evpn summary`; `show bgp l2vpn evpn route-type imet` | Prove EVPN session and route reflection first; troubleshoot ARP suppression in its own note |
##### Source_Basis
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Mental_Model
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Configuration_Checklist
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Skeleton
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Verification_Commands
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Rollback
# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout_Failure_Checks
# index of each title throughout note, not in table format
