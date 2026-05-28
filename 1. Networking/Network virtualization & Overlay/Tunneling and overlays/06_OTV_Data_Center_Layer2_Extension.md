##### index
```
# OTV_Data_Center_Layer2_Extension_Mental_Model
```

# index of each title throughout note, not in table format



Used local KB first. The local material places OTV under DCCOR data center overlay protocols, but the full NX-OS OTV syntax had to be filled from Cisco’s Nexus OTV configuration guide for feature otv, interface overlay, otv control-group, otv data-group, otv join-interface, otv extend-vlan, otv site-vlan, otv site-identifier, and verification commands. Cisco also states OTV uses an overlay control plane to discover remote sites, build adjacencies, and exchange MAC reachability information.  


```
# OTV_Data_Center_Layer2_Extension_Configuration_Checklist
# OTV_Data_Center_Layer2_Extension_Skeleton
# OTV_Data_Center_Layer2_Extension_Verification_Commands
# OTV_Data_Center_Layer2_Extension_Rollback
# OTV_Data_Center_Layer2_Extension_Failure_Checks
```
###### OTV_Data_Center_Layer2_Extension_Mental_Model

| Concept | Operational Meaning |
|---|---|
| OTV | Overlay Transport Virtualization extends selected Layer 2 VLANs across a routed data center interconnect transport |
| Edge device | Nexus device that performs OTV encapsulation, decapsulation, MAC route learning, and overlay forwarding |
| Overlay interface | Logical OTV interface where the overlay control group, data group, join interface, and extended VLANs are configured |
| Join interface | Layer 3 transport-facing interface used to send and receive OTV encapsulated traffic across the routed DCI |
| Internal interface | Site-facing Layer 2 interface carrying VLANs that may be extended by OTV |
| Extended VLAN | VLAN that OTV advertises and forwards across the overlay |
| Site VLAN | Local-only VLAN used by OTV edge devices inside the same site to discover each other and prevent both devices from forwarding the same VLANs incorrectly |
| Site identifier | Site-wide ID that must match on local OTV edge devices and must be unique between different sites |
| MAC routing | OTV exchanges MAC reachability across the overlay instead of blindly stretching a pure bridged domain everywhere |
| OTV control plane | Background IS-IS process discovers remote OTV edge devices and exchanges MAC reachability information |
| Control group | Multicast group used for OTV control-plane neighbor discovery and MAC reachability exchange |
| Data group | SSM multicast group range used for multicast data traffic across the overlay |
| AED | Authoritative Edge Device responsible for forwarding a given extended VLAN in a multihomed site |
| Not a generic tunnel | OTV is not GRE, L2TPv3, or a user-data VPN; it is a data center Layer 2 extension mechanism with MAC routing behavior |
| STP boundary | OTV is designed to avoid extending STP across the overlay like a raw trunk |
| Site VLAN rule | The site VLAN must not be extended through OTV |
| SVI rule | Do not extend a VLAN from the same VDC where that VLAN also has an active SVI |
| Underlay dependency | The join interface and routed DCI must work before OTV adjacency or MAC routes can work |
| Failure boundary | If join-interface reachability, multicast, site VLAN, site ID, or VLAN extension scope is wrong, the overlay fails or forwards incorrectly |
# OTV_Data_Center_Layer2_Extension_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm platform and image support OTV | OTV Edge | `show version` and `show module` | Platform, license, and modules support OTV |
| 2 | Confirm OTV feature state | OTV Edge | `show feature | include otv` | OTV is visible and currently disabled or enabled |
| 3 | Identify the routed DCI join interface | OTV Edge | `show ip interface brief` | Transport-facing Layer 3 interface is known |
| 4 | Configure join interface as Layer 3 transport interface | OTV Edge | `interface <JOIN_INTERFACE>` then `no switchport` then `ip address <JOIN_IP>/<PREFIX>` then `no shutdown` | Join interface is up/up with IP address |
| 5 | Enable IGMPv3 on the join interface | OTV Edge | `interface <JOIN_INTERFACE>` then `ip igmp version 3` | Join interface is ready for OTV multicast behavior |
| 6 | Verify join-interface route to remote site | OTV Edge | `show ip route <REMOTE_JOIN_IP>` | Remote join IP is reachable through routed DCI |
| 7 | Test join-interface reachability | OTV Edge | `ping <REMOTE_JOIN_IP> source <LOCAL_JOIN_INTERFACE>` | Ping succeeds across the DCI |
| 8 | Confirm multicast transport design | DCI Core | `show ip mroute` and `show ip pim neighbor` | Multicast core can support OTV control and data groups |
| 9 | Create extended VLANs locally | OTV Edge | `vlan <EXTENDED_VLAN_RANGE>` | VLANs that will be extended exist locally |
| 10 | Create site VLAN locally | OTV Edge | `vlan <SITE_VLAN>` | Site VLAN exists locally |
| 11 | Ensure site VLAN is active on at least one local edge-facing port | OTV Edge | `show vlan id <SITE_VLAN>` | Site VLAN is active locally |
| 12 | Confirm site VLAN is not in the extended VLAN list | OTV Edge | Design check: `<SITE_VLAN>` not inside `<EXTENDED_VLAN_RANGE>` | Site VLAN will remain local-only |
| 13 | Confirm extended VLANs do not have SVIs in the OTV VDC | OTV Edge | `show ip interface brief | include Vlan` | Extended VLANs are not routed in the same OTV VDC |
| 14 | Enable the OTV feature | OTV Edge | `feature otv` | OTV feature is enabled |
| 15 | Configure OTV site VLAN | OTV Edge | `otv site-vlan <SITE_VLAN>` | Local site VLAN is defined |
| 16 | Configure OTV site identifier | OTV Edge | `otv site-identifier <SITE_ID>` | Site ID is configured |
| 17 | Match site identifier on local edge devices in the same site | OTV Edge Pair | `otv site-identifier <SITE_ID>` | Same site uses the same site identifier |
| 18 | Use unique site identifier between different data center sites | OTV Edge | Design check: Site A ID differs from Site B ID | Remote sites do not share the same site ID |
| 19 | Create overlay interface | OTV Edge | `interface Overlay<OVERLAY_ID>` | Overlay interface exists |
| 20 | Add overlay description | OTV Edge | `description <OTV_DESCRIPTION>` | Overlay purpose is documented |
| 21 | Configure OTV control group | OTV Edge | `otv control-group <ASM_MULTICAST_GROUP>` | Overlay has a control-plane multicast group |
| 22 | Configure OTV data group range | OTV Edge | `otv data-group <SSM_MULTICAST_RANGE>` | Overlay has SSM data multicast group range |
| 23 | Bind overlay to join interface | OTV Edge | `otv join-interface <JOIN_INTERFACE>` | Overlay uses the routed DCI join interface |
| 24 | Configure extended VLAN range | OTV Edge | `otv extend-vlan <EXTENDED_VLAN_RANGE>` | Selected VLANs are advertised and carried by OTV |
| 25 | Bring overlay interface up | OTV Edge | `no shutdown` | Overlay interface attempts to become operational |
| 26 | Repeat equivalent configuration on the remote OTV edge device | Remote OTV Edge | Same commands with local join IP and matching overlay values | Remote edge is configured for the same OTV overlay |
| 27 | Verify OTV feature state | OTV Edge | `show feature | include otv` | OTV is enabled |
| 28 | Verify OTV running configuration | OTV Edge | `show running-configuration otv all` | OTV site, overlay, groups, join interface, and VLANs are present |
| 29 | Verify overlay interface | OTV Edge | `show otv overlay` | Overlay is up and bound to expected join interface |
| 30 | Verify OTV site information | OTV Edge | `show otv site` | Site VLAN and site identifier are correct |
| 31 | Verify OTV adjacency | OTV Edge | `show otv adjacency detail` | Remote OTV edge adjacency is up |
| 32 | Verify extended VLAN state | OTV Edge | `show otv vlan <VLAN_ID> detail` | VLAN is active, extended, and has correct AED state |
| 33 | Verify OTV routes | OTV Edge | `show otv route` | Remote MAC routes are learned across the overlay |
| 34 | Verify local and remote MAC learning | OTV Edge | `show mac address-table vlan <VLAN_ID>` | Local and remote MAC behavior matches design |
| 35 | Test host reachability across extended VLAN | Host A/Host B | `ping <REMOTE_HOST_SAME_VLAN_IP>` | Same-subnet host reachability works across sites |
| 36 | Verify ARP across extended VLAN | Host or OTV Edge | `show ip arp` or host ARP table | Remote host ARP resolves |
| 37 | Verify multicast data group if multicast is expected | OTV Edge | `show otv data-group` | Local and remote data groups are advertised |
| 38 | Check OTV multicast route state | OTV Edge | `show otv mroute vlan <VLAN_ID> startup` | OTV multicast route information exists where expected |
| 39 | Confirm no site VLAN leak | OTV Edge | `show otv vlan <SITE_VLAN> detail` | Site VLAN is not extended |
| 40 | Save working configuration | OTV Edge | `copy running-config startup-config` | OTV configuration persists after reload |
# OTV_Data_Center_Layer2_Extension_Skeleton
! =========================================================
! DESIGN RULES
! =========================================================
! OTV extends selected VLANs across a routed DCI.
! The join interface is Layer 3 and faces the transport network.
! The site VLAN is local-only and must not be extended.
! The site identifier must match inside a site and differ between sites.
! Do not extend a VLAN that also has an SVI in the same OTV VDC.
! Bring the overlay up only after site VLAN and site identifier are configured.
! =========================================================
! SITE A OTV EDGE
! =========================================================
conf t
!
feature otv
!
vlan <SITE_VLAN>
 name OTV_SITE_VLAN_LOCAL_ONLY
!
vlan <EXTENDED_VLAN_RANGE>
 name OTV_EXTENDED_VLANS
!
interface <JOIN_INTERFACE>
 description OTV_JOIN_TO_DCI
 no switchport
 ip address <SITE_A_JOIN_IP>/<PREFIX>
 ip igmp version 3
 no shutdown
!
otv site-vlan <SITE_VLAN>
otv site-identifier <SITE_A_SITE_ID>
!
interface Overlay<OVERLAY_ID>
 description OTV_SITE_A_TO_SITE_B
 otv control-group <CONTROL_GROUP_ASM>
 otv data-group <DATA_GROUP_SSM_RANGE>
 otv join-interface <JOIN_INTERFACE>
 otv extend-vlan <EXTENDED_VLAN_RANGE>
 no shutdown
!
end
copy running-config startup-config
! =========================================================
! SITE B OTV EDGE
! =========================================================
conf t
!
feature otv
!
vlan <SITE_VLAN>
 name OTV_SITE_VLAN_LOCAL_ONLY
!
vlan <EXTENDED_VLAN_RANGE>
 name OTV_EXTENDED_VLANS
!
interface <JOIN_INTERFACE>
 description OTV_JOIN_TO_DCI
 no switchport
 ip address <SITE_B_JOIN_IP>/<PREFIX>
 ip igmp version 3
 no shutdown
!
otv site-vlan <SITE_VLAN>
otv site-identifier <SITE_B_SITE_ID>
!
interface Overlay<OVERLAY_ID>
 description OTV_SITE_B_TO_SITE_A
 otv control-group <CONTROL_GROUP_ASM>
 otv data-group <DATA_GROUP_SSM_RANGE>
 otv join-interface <JOIN_INTERFACE>
 otv extend-vlan <EXTENDED_VLAN_RANGE>
 no shutdown
!
end
copy running-config startup-config
! =========================================================
! MULTIHOMED SITE REMINDER
! =========================================================
! On two local OTV edge devices in the same site:
! - use the same site VLAN
! - use the same site identifier
! - keep the site VLAN active locally
! - do not extend the site VLAN
! - verify AED state per VLAN
! =========================================================
! VERIFICATION
! =========================================================
show feature | include otv
show running-configuration otv all
show ip route <REMOTE_JOIN_IP>
ping <REMOTE_JOIN_IP> source <JOIN_INTERFACE>
show otv overlay
show otv site
show otv adjacency detail
show otv vlan <VLAN_ID> detail
show otv route
show otv data-group
show otv mroute vlan <VLAN_ID> startup
show mac address-table vlan <VLAN_ID>
# OTV_Data_Center_Layer2_Extension_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Confirm OTV feature | OTV Edge | `show feature | include otv` | OTV is enabled |
| Confirm OTV running configuration | OTV Edge | `show running-configuration otv all` | Site VLAN, site ID, overlay, join interface, groups, and extended VLANs are present |
| Confirm join interface state | OTV Edge | `show ip interface brief | include <JOIN_INTERFACE>` | Join interface is up/up |
| Confirm join interface IP and IGMP | OTV Edge | `show running-config interface <JOIN_INTERFACE>` | Interface has IP address and `ip igmp version 3` |
| Confirm route to remote join IP | OTV Edge | `show ip route <REMOTE_JOIN_IP>` | Remote join IP resolves through routed DCI |
| Confirm join reachability | OTV Edge | `ping <REMOTE_JOIN_IP> source <JOIN_INTERFACE>` | Ping succeeds |
| Confirm overlay state | OTV Edge | `show otv overlay` | Overlay is up and associated with expected join interface |
| Confirm overlay details | OTV Edge | `show otv overlay Overlay<OVERLAY_ID>` | Control group, data group, join interface, and state match design |
| Confirm OTV adjacency | OTV Edge | `show otv adjacency` | Remote OTV edge appears as adjacent |
| Confirm OTV adjacency detail | OTV Edge | `show otv adjacency detail` | Neighbor details and state are correct |
| Confirm site state | OTV Edge | `show otv site` | Site VLAN and site identifier are correct |
| Confirm local site IS-IS state | OTV Edge | `show otv isis site database` | Site control-plane information exists |
| Confirm extended VLAN state | OTV Edge | `show otv vlan <VLAN_ID> detail` | VLAN is extended and operational |
| Confirm AED state | OTV Edge | `show otv <OVERLAY_INTERFACE> vlan <VLAN_ID> authoritative` | Correct edge device is authoritative for the VLAN |
| Confirm OTV MAC routes | OTV Edge | `show otv route` | Remote MAC routes are present |
| Confirm specific MAC route | OTV Edge | `show otv route vlan <VLAN_ID> mac-address <MAC_ADDRESS>` | MAC resolves through expected overlay neighbor |
| Confirm multicast data groups | OTV Edge | `show otv data-group` | Data group ranges are advertised |
| Confirm OTV multicast route | OTV Edge | `show otv mroute vlan <VLAN_ID> startup` | OTV multicast route information exists |
| Confirm FIB multicast route | OTV Edge | `show forwarding distribution otv multicast route vlan <VLAN_ID>` | Forwarding entry exists where expected |
| Confirm MAC table behavior | OTV Edge | `show mac address-table vlan <VLAN_ID>` | Local and remote MAC learning matches design |
| Confirm site VLAN is not extended | OTV Edge | `show otv vlan <SITE_VLAN> detail` | Site VLAN is not extended |
| Confirm host reachability | Host A/Host B | `ping <REMOTE_HOST_SAME_VLAN_IP>` | Same-subnet host traffic crosses OTV |
| Confirm host ARP | Host A/Host B | `arp -a` or platform ARP command | Remote host MAC resolves |
| Confirm logs are clean | OTV Edge | `show logging | include OTV|Overlay|ISIS|site|AED` | No active OTV failure messages appear |
# OTV_Data_Center_Layer2_Extension_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop OTV forwarding before removing overlay configuration | OTV Edge | `interface Overlay<OVERLAY_ID>` then `shutdown` | Overlay stops forwarding |
| 2 | Remove extended VLANs from overlay | OTV Edge | `interface Overlay<OVERLAY_ID>` then `no otv extend-vlan <EXTENDED_VLAN_RANGE>` | VLANs are no longer extended by OTV |
| 3 | Remove join interface binding | OTV Edge | `interface Overlay<OVERLAY_ID>` then `no otv join-interface <JOIN_INTERFACE>` | Overlay is detached from transport interface |
| 4 | Remove data group | OTV Edge | `interface Overlay<OVERLAY_ID>` then `no otv data-group <DATA_GROUP_SSM_RANGE>` | OTV data group is removed |
| 5 | Remove control group | OTV Edge | `interface Overlay<OVERLAY_ID>` then `no otv control-group <CONTROL_GROUP_ASM>` | OTV control group is removed |
| 6 | Delete overlay interface | OTV Edge | `no interface Overlay<OVERLAY_ID>` | Overlay interface is removed |
| 7 | Remove site VLAN only if OTV is fully removed | OTV Edge | `no otv site-vlan <SITE_VLAN>` | Site VLAN setting is removed |
| 8 | Remove site identifier only if OTV is fully removed | OTV Edge | `no otv site-identifier <SITE_ID>` | Site identifier is removed |
| 9 | Disable OTV feature only if no overlays remain | OTV Edge | `no feature otv` | OTV feature is disabled |
| 10 | Remove lab-only extended VLANs if not reused | OTV Edge | `no vlan <EXTENDED_VLAN_RANGE>` | Lab VLANs are removed |
| 11 | Remove lab-only site VLAN if not reused | OTV Edge | `no vlan <SITE_VLAN>` | Site VLAN is removed |
| 12 | Restore join interface to baseline role if needed | OTV Edge | `interface <JOIN_INTERFACE>` then baseline interface config | Join interface returns to pre-lab function |
| 13 | Verify OTV overlay removal | OTV Edge | `show otv overlay` | Removed overlay no longer appears |
| 14 | Verify OTV adjacency removal | OTV Edge | `show otv adjacency` | No adjacency remains for removed overlay |
| 15 | Verify host traffic no longer depends on OTV | Host/OTV Edge | `ping <REMOTE_HOST_SAME_VLAN_IP>` and `show otv route` | Extended path is gone as expected |
| 16 | Save rollback state | OTV Edge | `copy running-config startup-config` | Rollback persists after reload |
# OTV_Data_Center_Layer2_Extension_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| `interface overlay` command missing | OTV feature not enabled or unsupported platform | OTV Edge | `show feature | include otv` | Configure `feature otv` or use supported Nexus/feature set |
| Overlay stays down | Site VLAN or site identifier missing | OTV Edge | `show otv site` | Configure `otv site-vlan <SITE_VLAN>` and `otv site-identifier <SITE_ID>` before `no shutdown` |
| Overlay stays down | Missing control group, data group, or join interface | OTV Edge | `show running-configuration otv all` | Configure `otv control-group`, `otv data-group`, and `otv join-interface` |
| OTV adjacency does not form | Join interface cannot reach remote edge | OTV Edge | `ping <REMOTE_JOIN_IP> source <JOIN_INTERFACE>` | Fix routed DCI reachability |
| OTV adjacency does not form | Multicast control group not working in transport | OTV Edge/DCI Core | `show ip mroute` and `show otv adjacency detail` | Fix multicast/PIM/IGMP support in the DCI |
| OTV multicast traffic fails | Data group range missing or multicast transport issue | OTV Edge | `show otv data-group` and `show otv mroute vlan <VLAN_ID> startup` | Configure valid SSM data group and fix multicast core |
| VLAN does not extend | VLAN not configured under `otv extend-vlan` | OTV Edge | `show otv vlan <VLAN_ID> detail` | Add VLAN with `otv extend-vlan add <VLAN_ID>` |
| VLAN does not extend | VLAN does not exist locally | OTV Edge | `show vlan id <VLAN_ID>` | Create the VLAN locally |
| Overlay forwards wrong VLAN scope | Extended VLAN range too broad | OTV Edge | `show running-configuration otv all` | Narrow `otv extend-vlan` to only required VLANs |
| Site VLAN appears in overlay | Site VLAN accidentally included in extended VLAN range | OTV Edge | `show otv vlan <SITE_VLAN> detail` | Remove site VLAN from `otv extend-vlan` immediately |
| Overlay is unstable in multihomed site | Site identifier mismatch between local edge devices | OTV Edge Pair | `show otv site` | Use same site identifier on edge devices in the same site |
| Remote sites behave like same local site | Site identifier reused between different sites | OTV Edge | `show otv site` | Use unique site identifier per data center site |
| VLAN has SVI conflict | Extended VLAN has active SVI in the OTV VDC | OTV Edge | `show ip interface brief | include Vlan<VLAN_ID>` | Move SVI out of OTV VDC or do not extend that VLAN |
| Hosts cannot ARP across sites | OTV route or MAC reachability missing | OTV Edge | `show otv route` and `show mac address-table vlan <VLAN_ID>` | Fix adjacency, VLAN extension, or MAC learning |
| Host ping fails but OTV adjacency is up | Host VLAN, trunk, or access path issue | OTV Edge/Access Switch | `show vlan brief`, `show interface trunk`, `show mac address-table vlan <VLAN_ID>` | Fix local Layer 2 path |
| Wrong AED handles VLAN | Multihomed AED election or site adjacency issue | OTV Edge | `show otv <OVERLAY_INTERFACE> vlan <VLAN_ID> authoritative` | Fix site VLAN reachability and local edge consistency |
| OTV route table empty | No traffic or no MAC learning from local VLANs | OTV Edge | `show otv route` and `show mac address-table vlan <VLAN_ID>` | Generate host traffic and verify local L2 learning |
| Overlay came up before site configuration was correct | Site VLAN or site ID changed while overlay was active | OTV Edge | `show running-configuration otv all` | Shut overlay, correct site values, then `no shutdown` |
| DCI ping works but OTV control fails | Multicast not permitted or unsupported across DCI | DCI Core | `show ip pim neighbor`, `show ip mroute <CONTROL_GROUP>` | Fix multicast routing or use a supported OTV adjacency design |
| Operator treats OTV like a trunk | Wrong mental model | Admin | `show otv route` and `show otv vlan` | Treat OTV as MAC-routed Layer 2 extension, not a raw Layer 2 trunk |


