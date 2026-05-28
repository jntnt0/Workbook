I used the local KB as the source base and Cisco’s current Nexus NX-OS VXLAN BGP EVPN guide for the exact EVPN syntax: nv overlay evpn, evpn vni <vni> l2, rd auto, route-target both auto, host-reachability protocol bgp, address-family l2vpn evpn, send-community extended, and BGP-based ingress replication.  

VXLAN_BGP_EVPN_L2VNI_Control_Plane.md

VXLAN_BGP_EVPN_L2VNI_Control_Plane

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Data Center VXLAN EVPN and NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and EVPN consistency checks to `NS-XO_combined_epubs.md` |
| `CiscoPress_combined_part1.md` | DCCOR Ch 3, VXLAN MPBGP EVPN Control Plane | Establishes that BGP EVPN distributes VTEP information and remote MAC reachability through BGP instead of pure data-plane flood-and-learn |
| `CiscoPress_combined_part2.md` | L2VPN EVPN service deployment, EVPN route-type sections | Supports EVPN Route Type 2 MAC/IP advertisement and Route Type 3 inclusive multicast route verification logic |
| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Supports checks for NVE source loopback, L2VNI replication method, `advertise l2vpn evpn` on older NX-OS versions, `suppress-arp` constraints, and L3VNI separation |
| `Cisco Nexus 9000 Series NX-OS VXLAN Configuration Guide, Release 10.6(x)` | Configure VXLAN BGP EVPN | Provides current NX-OS syntax for `nv overlay evpn`, `evpn vni <vni> l2`, `rd auto`, `route-target both auto`, `interface nve1`, `host-reachability protocol bgp`, `ingress-replication protocol bgp`, and BGP `address-family l2vpn evpn` |
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VXLAN data plane | VXLAN still carries encapsulated Ethernet frames between VTEPs using outer IP/UDP transport |
| EVPN control plane | MP-BGP EVPN distributes endpoint and VTEP reachability so the fabric does not rely only on multicast flood-and-learn |
| L2VNI | A Layer 2 bridge-domain carried across the VXLAN overlay |
| MAC-VRF | The EVPN control-plane instance for an L2VNI |
| Route Distinguisher | Makes EVPN routes unique per originating VTEP and service |
| Route Target | Controls which EVPN routes are imported and exported into the matching L2VNI |
| Route Type 2 | MAC or MAC/IP advertisement used to tell remote VTEPs where endpoints live |
| Route Type 3 | Inclusive multicast route used to build the EVPN BUM replication list for a VNI |
| BGP ingress replication | BGP provides the remote VTEP list for BUM traffic, avoiding multicast underlay requirements |
| NVE host reachability | `host-reachability protocol bgp` tells NVE to use EVPN control-plane learning |
| Route reflector role | Spines may reflect L2VPN EVPN routes, but the deeper scaleout design belongs in the route-reflector note |
| What this note is not | This note does not build L3VNI routing, ARP suppression, multisite, tenant routed multicast, or distributed anycast gateway beyond what is needed for L2VNI EVPN control-plane learning |
| Related labs | `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final`; `vxlan-bgp-evpn-arp-suppression-final` |
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm underlay reachability before EVPN | Leaf VTEPs | `ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>` | Every VTEP loopback can reach every remote VTEP loopback |
| 2 | Confirm BGP update-source loopback reachability | Leaves and spines | `ping <BGP_NEIGHBOR_LOOPBACK> source <LOCAL_BGP_LOOPBACK>` | EVPN BGP peers can establish using loopback-sourced sessions |
| 3 | Confirm VLAN-to-L2VNI mapping already exists | Leaf VTEPs | `show vxlan` | VLAN maps to the expected L2VNI |
| 4 | Confirm VXLAN features are enabled | Leaf VTEPs | `show feature | include nv`; `show feature | include vn-segment` | VXLAN and VLAN-to-VNI features are enabled |
| 5 | Enable BGP | Leaves and spines | `feature bgp` | NX-OS accepts BGP configuration |
| 6 | Enable VXLAN EVPN control plane | Leaf VTEPs | `nv overlay evpn` | Device enables EVPN as the VXLAN overlay control plane |
| 7 | Enter EVPN configuration mode | Leaf VTEPs | `evpn` | EVPN MAC-VRF configuration mode opens |
| 8 | Create the L2VNI EVPN instance | Leaf VTEPs | `vni <L2VNI> l2` | L2VNI is associated with an EVPN MAC-VRF |
| 9 | Configure automatic RD for the L2VNI | Leaf VTEPs | `rd auto` | Each VTEP derives a unique RD for the MAC-VRF |
| 10 | Configure automatic RT import/export for iBGP EVPN fabrics | Leaf VTEPs | `route-target both auto` | L2VNI routes are imported and exported using auto-derived RTs |
| 11 | Use manual RTs if this is eBGP or asymmetric VNI design | Leaf VTEPs | `route-target both <ASN_OR_IP>:<VALUE>` | RTs match across all VTEPs that must share the same L2VNI |
| 12 | Configure BGP process and router ID | Leaves and spines | `router bgp <ASN>` then `router-id <ROUTER_ID>` | BGP process has a stable router ID |
| 13 | Configure EVPN BGP neighbor | Leaf VTEPs | `neighbor <SPINE_OR_RR_LOOPBACK> remote-as <ASN>` | BGP neighbor is defined |
| 14 | Source BGP from the intended loopback | Leaf VTEPs | `neighbor <SPINE_OR_RR_LOOPBACK> update-source loopback<LOOPBACK_ID>` | BGP uses loopback sourcing for overlay resilience |
| 15 | Activate L2VPN EVPN address family toward the neighbor | Leaf VTEPs | `neighbor <SPINE_OR_RR_LOOPBACK>` then `address-family l2vpn evpn` | Neighbor participates in EVPN route exchange |
| 16 | Send extended communities to EVPN neighbor | Leaf VTEPs | `send-community extended` | EVPN RT and extended community attributes are carried |
| 17 | Configure spine or RR EVPN neighbor back to each leaf | Spine or RR | `neighbor <LEAF_LOOPBACK> remote-as <ASN>` | Spine or RR has the leaf as a BGP neighbor |
| 18 | Source spine or RR BGP from loopback | Spine or RR | `neighbor <LEAF_LOOPBACK> update-source loopback<LOOPBACK_ID>` | Spine or RR uses loopback-sourced BGP |
| 19 | Activate L2VPN EVPN on spine or RR neighbor | Spine or RR | `neighbor <LEAF_LOOPBACK>` then `address-family l2vpn evpn` | Leaf receives EVPN routes through the spine or RR |
| 20 | Configure route reflection in iBGP fabrics | Spine or RR | `route-reflector-client` | Spine reflects EVPN routes between leaf VTEPs |
| 21 | Send extended communities from spine or RR | Spine or RR | `send-community extended` | Reflected EVPN routes retain RT and EVPN attributes |
| 22 | Configure NVE to use BGP host reachability | Leaf VTEPs | `interface nve1` then `host-reachability protocol bgp` | NVE learns remote VTEPs and endpoints from EVPN |
| 23 | Add the L2VNI under NVE | Leaf VTEPs | `member vni <L2VNI>` | NVE has the L2VNI attached |
| 24 | Configure BGP ingress replication for the L2VNI | Leaf VTEPs | `ingress-replication protocol bgp` | BGP EVPN builds the remote VTEP replication list |
| 25 | Enable NVE | Leaf VTEPs | `no shutdown` | NVE interface is administratively enabled |
| 26 | Verify EVPN BGP neighbor state | Leaves and spines | `show bgp l2vpn evpn summary` | EVPN peers are established |
| 27 | Verify L2VNI mode | Leaf VTEPs | `show nve vni <L2VNI> detail` | VNI is up, type L2, and control-plane mode is used |
| 28 | Verify NVE peers are control-plane learned | Leaf VTEPs | `show nve peers` | Remote VTEPs appear with LearnType `CP` |
| 29 | Verify EVPN Route Type 3 exists for VNI membership | Leaf VTEPs | `show bgp l2vpn evpn route-type imet` | Inclusive multicast routes exist for participating VTEPs |
| 30 | Generate endpoint traffic to create local MAC learning | Hosts | `ping <REMOTE_HOST_IP>` | Host traffic triggers local MAC learning and EVPN advertisement |
| 31 | Verify EVPN Route Type 2 MAC advertisement | Leaf VTEPs | `show bgp l2vpn evpn route-type macip` | MAC or MAC/IP routes are present for local and remote endpoints |
| 32 | Verify local and remote MAC installation | Leaf VTEPs | `show mac address-table dynamic vlan <VLAN_ID>` | Local MACs appear on access ports and remote MACs appear through NVE |
| 33 | Verify host reachability across the L2VNI | Hosts | `ping <REMOTE_HOST_IP>` | Hosts in the same L2VNI can communicate through EVPN-controlled VXLAN |
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Skeleton
# LEAF VTEP BASELINE
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
  neighbor <SPINE_OR_RR_LOOPBACK>
    remote-as <ASN>
    update-source loopback<BGPD_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
interface nve1
  source-interface loopback<VTEP_LOOPBACK_ID>
  host-reachability protocol bgp
  member vni <L2VNI>
    ingress-replication protocol bgp
  no shutdown
end
# SPINE OR ROUTE-REFLECTOR BASELINE FOR iBGP EVPN
conf t
feature bgp
router bgp <ASN>
  router-id <SPINE_ROUTER_ID>
  neighbor <LEAF1_BGP_LOOPBACK>
    remote-as <ASN>
    update-source loopback<BGPD_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
  neighbor <LEAF2_BGP_LOOPBACK>
    remote-as <ASN>
    update-source loopback<BGPD_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
      route-reflector-client
end
# LEAF WITH TWO SPINE ROUTE REFLECTORS
conf t
router bgp <ASN>
  router-id <LEAF_ROUTER_ID>
  neighbor <SPINE1_LOOPBACK>
    remote-as <ASN>
    update-source loopback<BGPD_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
  neighbor <SPINE2_LOOPBACK>
    remote-as <ASN>
    update-source loopback<BGPD_LOOPBACK_ID>
    address-family l2vpn evpn
      send-community extended
end
# EVPN L2VNI WITH MANUAL ROUTE TARGETS
# Use this when auto RTs are not appropriate, especially eBGP or asymmetric VNI designs.
conf t
evpn
  vni <L2VNI> l2
    rd auto
    route-target import <RT_VALUE>
    route-target export <RT_VALUE>
end
# OLDER NX-OS CHECK
# Some older NX-OS versions may require:
# advertise l2vpn evpn
# Confirm platform and version behavior before adding it.
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Verification_Commands
# Underlay and loopback reachability
show ip route <REMOTE_VTEP_IP>
ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>
ping <BGP_NEIGHBOR_LOOPBACK> source <LOCAL_BGP_LOOPBACK>
# Feature and EVPN enablement
show feature | include bgp
show feature | include nv
show feature | include vn-segment
show running-config | include "nv overlay evpn"
# VLAN and L2VNI mapping
show vlan id <VLAN_ID>
show vxlan
show running-config vlan <VLAN_ID>
# EVPN MAC-VRF configuration
show running-config evpn
show running-config evpn | section "vni <L2VNI>"
# BGP EVPN configuration and neighbor state
show running-config bgp
show bgp l2vpn evpn summary
show bgp l2vpn evpn neighbors
show bgp l2vpn evpn neighbors <NEIGHBOR_IP>
# EVPN route checks
show bgp l2vpn evpn
show bgp l2vpn evpn route-type macip
show bgp l2vpn evpn route-type imet
show bgp l2vpn evpn vni-id <L2VNI>
# NVE state
show running-config interface nve1
show nve interface
show nve vni
show nve vni <L2VNI> detail
show nve peers
show nve peers detail
# MAC learning and forwarding
show mac address-table dynamic vlan <VLAN_ID>
show l2route evpn mac all
show l2route evpn mac-ip all
# Endpoint tests
ping <REMOTE_HOST_IP>
arp -a
# vPC VTEP checks, only if vPC VTEPs are used
show vpc brief
show vpc consistency-parameters global
show port-channel summary
show nve interface
show nve peers
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Rollback
# REMOVE L2VNI BGP INGRESS REPLICATION FROM NVE
conf t
interface nve1
  member vni <L2VNI>
    no ingress-replication protocol bgp
end
# REMOVE L2VNI FROM NVE
conf t
interface nve1
  no member vni <L2VNI>
end
# REMOVE HOST REACHABILITY PROTOCOL FROM NVE
conf t
interface nve1
  no host-reachability protocol bgp
end
# REMOVE EVPN MAC-VRF INSTANCE FOR ONE L2VNI
conf t
evpn
  no vni <L2VNI> l2
end
# REMOVE VLAN-TO-VNI MAPPING BUT KEEP VLAN
conf t
vlan <VLAN_ID>
  no vn-segment <L2VNI>
end
# REMOVE EVPN BGP ADDRESS FAMILY FROM ONE NEIGHBOR
conf t
router bgp <ASN>
  neighbor <NEIGHBOR_LOOPBACK>
    no address-family l2vpn evpn
end
# REMOVE ONE BGP NEIGHBOR
conf t
router bgp <ASN>
  no neighbor <NEIGHBOR_LOOPBACK>
end
# REMOVE ROUTE-REFLECTOR ROLE FROM ONE NEIGHBOR
conf t
router bgp <ASN>
  neighbor <LEAF_LOOPBACK>
    address-family l2vpn evpn
      no route-reflector-client
end
# DISABLE VXLAN EVPN CONTROL PLANE
conf t
no nv overlay evpn
end
# REMOVE BGP FEATURE ONLY IF NO OTHER BGP SERVICES EXIST
conf t
no router bgp <ASN>
no feature bgp
end
# REMOVE VXLAN FEATURES ONLY IF NO OTHER VXLAN SERVICES EXIST
conf t
no feature nv overlay
no feature vn-segment-vlan-based
end
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| EVPN BGP neighbor is idle | Loopback reachability missing, wrong remote AS, wrong update-source, or BGP feature missing | `show bgp l2vpn evpn summary`; `ping <NEIGHBOR_LOOPBACK> source <LOCAL_LOOPBACK>` | Fix underlay route, neighbor AS, and `update-source` |
| EVPN BGP neighbor is established but no routes are exchanged | L2VPN EVPN AF not activated or extended communities not sent | `show running-config bgp`; `show bgp l2vpn evpn neighbors <NEIGHBOR_IP>` | Add `address-family l2vpn evpn` and `send-community extended` |
| L2VNI does not appear in EVPN | Missing `evpn vni <L2VNI> l2` or missing VLAN-to-VNI mapping | `show running-config evpn`; `show vxlan` | Configure `evpn vni <L2VNI> l2`, `rd auto`, `route-target both auto`, and VLAN `vn-segment` |
| `show nve vni` shows data-plane mode instead of control-plane mode | NVE missing `host-reachability protocol bgp` | `show running-config interface nve1` | Add `host-reachability protocol bgp` under `interface nve1` |
| NVE peers show `DP` instead of `CP` | L2VNI is using flood-and-learn or EVPN is not controlling that VNI | `show nve peers`; `show nve vni <L2VNI> detail` | Replace flood-and-learn replication with BGP EVPN ingress replication |
| NVE peer is missing even though BGP is up | Route Type 3 is missing, RT mismatch, or L2VNI not active on remote VTEP | `show bgp l2vpn evpn route-type imet`; `show running-config evpn` | Fix RT import/export and confirm remote VTEP has the same L2VNI active |
| Remote MAC is not learned through EVPN | No local MAC on remote leaf, Route Type 2 missing, or RT mismatch | `show bgp l2vpn evpn route-type macip`; `show mac address-table dynamic vlan <VLAN_ID>` | Generate endpoint traffic, fix host VLAN, and align RTs |
| Local MAC is missing | Host-facing VLAN or trunk problem | `show interface <HOST_PORT> switchport`; `show vlan id <VLAN_ID>` | Correct access VLAN or trunk allowed VLAN configuration |
| Remote MAC exists in BGP but is not installed in MAC table | L2RIB or NVE programming issue, VNI down, or wrong VLAN/VNI mapping | `show l2route evpn mac all`; `show nve vni <L2VNI> detail`; `show vxlan` | Fix VNI state and VLAN-to-VNI mapping, then clear or re-advertise host state if needed |
| Spine receives EVPN routes but leaves do not learn each other | Spine is not acting as route reflector or extended communities are not preserved | `show running-config bgp`; `show bgp l2vpn evpn summary` | Add `route-reflector-client` and `send-community extended` under leaf neighbors |
| eBGP EVPN routes are not retained on spine | Spine has no local matching VNI and is not retaining RTs | `show running-config bgp`; `show bgp l2vpn evpn` | Add `retain route-target all` for eBGP EVPN spine designs |
| eBGP EVPN route advertisement fails due to peer AS checks | Leaf AS appears in path and spine blocks advertisement | `show bgp l2vpn evpn neighbors <NEIGHBOR_IP>` | Use `disable-peer-as-check` where the design requires it |
| Auto RT does not import routes in multi-AS fabric | Auto-derived RTs do not match across AS boundaries | `show bgp l2vpn evpn route-type macip detail`; `show running-config evpn` | Replace auto RT with matching manual import/export RTs |
| ARP suppression breaks or hides a basic L2VNI test | ARP suppression was configured before EVPN MAC/IP learning and anycast gateway were validated | `show running-config interface nve1`; `show nve vni` | Remove `suppress-arp` from this note and configure it only in the ARP suppression note |
| Underlay multicast is missing | Not a fault for BGP EVPN ingress replication | `show running-config interface nve1` | If using `ingress-replication protocol bgp`, do not troubleshoot PIM unless another VNI uses multicast |
| Older NX-OS does not advertise EVPN as expected | Some older versions may need `advertise l2vpn evpn` | `show version`; `show running-config bgp` | Confirm release behavior and add `advertise l2vpn evpn` only if required |
##### Source_Basis
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Mental_Model
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Configuration_Checklist
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Skeleton
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Verification_Commands
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Rollback
# VXLAN_BGP_EVPN_L2VNI_Control_Plane_Failure_Checks
# index of each title throughout note, not in table format
