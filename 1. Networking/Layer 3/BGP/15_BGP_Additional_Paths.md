BGP_Additional_Paths.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP path advertisement, iBGP scaling, path diversity, route reflection, and BGP design behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP neighbor, address-family, and verification syntax |
| `CiscoPress_combined_part2.md` lines 10882-10886 | BGP Additional Path implementation steps | Supports the three-step model: enable send/receive, define path selection, advertise additional paths |
| `CiscoPress_combined_part2.md` lines 10897-10901 | IOS XE Additional Paths behavior | Confirms IOS XE configures send/receive per neighbor under address-family and advertises additional paths with `neighbor <ip> advertise additional-paths all` |
| `CiscoPress_combined_part2.md` lines 10920-10929 | IOS XE Additional Paths config example | Supports `neighbor <ip> additional-paths send receive` and `neighbor <ip> advertise additional-paths all` |
| `CiscoPress_combined_part2.md` lines 11277-11277 | BGP table additional-path marker | Supports `add-path` marker in BGP path output |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22804 and 27547 | BGP table status code `a` | Supports `a additional-path` as a BGP table code |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 27838-27872 | BGP multipath behavior | Supports contrast with `maximum-paths`, which installs multiple paths locally rather than advertising additional paths |
# BGP_Additional_Paths_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Additional Paths | BGP feature that allows more than one path for the same prefix to be advertised to a peer |
| Normal BGP advertisement | BGP usually advertises only its best path to a neighbor |
| Path hiding | A route reflector or upstream router may hide alternate paths because it advertises only the best path |
| Add-Path purpose | Preserves path diversity by allowing selected non-best paths to be sent |
| Path ID | Additional Paths adds a path identifier so multiple paths for the same NLRI can be distinguished |
| Send capability | Router is allowed to send additional paths to the neighbor |
| Receive capability | Router is allowed to receive additional paths from the neighbor |
| Send and receive agreement | Both sides must support the capability in the needed direction |
| `additional-paths send receive` | IOS XE per-neighbor address-family command enabling send/receive capability |
| `advertise additional-paths all` | IOS XE per-neighbor address-family command advertising all eligible additional paths |
| Selection model | Router must decide which additional paths are advertised, such as all, best N, or group-best depending on platform support |
| `a` table marker | Indicates an additional path in BGP table output |
| `add-path` detail marker | BGP path detail may mark a path as `add-path` |
| Route reflector use case | Route reflectors can pass more than one path to clients to reduce path hiding |
| PIC/backup path use case | Additional paths can expose backup paths for faster convergence in supported designs |
| Not multipath | Additional Paths advertises extra paths; multipath installs extra local forwarding paths |
| Not load balancing by itself | Receiving extra paths does not guarantee multiple next hops are installed in the RIB/FIB |
| Not policy replacement | Additional paths still depend on BGP policy, next-hop reachability, and best-path rules |
| Blunt rule | Additional Paths is about route visibility between BGP speakers. `maximum-paths` is about local forwarding installation. Do not merge those notes |
# BGP_Additional_Paths_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Neighbor shows a number under `State/PfxRcd` |
| 2 | Confirm target prefix has multiple BGP paths locally | Router | `show bgp ipv4 unicast <prefix>` | Prefix shows more than one path |
| 3 | Confirm current advertised routes to peer | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Baseline shows only the current advertised path |
| 4 | Confirm peer currently sees only one path | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer has one path for the prefix before Add-Path |
| 5 | Confirm this is an advertisement problem, not local multipath | Notes | `Additional Paths, not maximum-paths` | Goal is to send/receive extra paths, not merely install multiple local next hops |
| 6 | Confirm address family | Notes | `ipv4 unicast` | Add-Path AFI/SAFI is known |
| 7 | Confirm path selection goal | Notes | `all`, `best-n`, or `group-best` | Selection model is known before configuration |
| 8 | Confirm IOS XE syntax support | Router | `?` under `router bgp` address-family neighbor context | Platform supports needed Add-Path commands |
| 9 | Confirm neighbor supports Add-Path capability | Router | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Neighbor capability details are visible |
| 10 | Confirm next-hop reachability for all candidate paths | Router | `show ip route <next-hop-ip>` | Candidate next hops are reachable |
| 11 | Confirm route policy does not filter alternate paths | Router | `show running-config | include route-map|prefix-list|filter-list|distribute-list` | Existing policy is identified |
| 12 | Review current BGP config | Router | `show running-config | section router bgp` | Neighbor and address-family config is visible |
| 13 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 14 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 15 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 16 | Enable Add-Path send capability toward neighbor | Router | `neighbor <neighbor-ip> additional-paths send` | Router can send additional paths to peer |
| 17 | Enable Add-Path receive capability from neighbor | Router | `neighbor <neighbor-ip> additional-paths receive` | Router can receive additional paths from peer |
| 18 | Enable Add-Path send and receive together | Router | `neighbor <neighbor-ip> additional-paths send receive` | Router can send and receive additional paths |
| 19 | Advertise all additional paths to neighbor | Router | `neighbor <neighbor-ip> advertise additional-paths all` | Router advertises all eligible additional paths to peer |
| 20 | Apply Add-Path to peer group if used | Router | `neighbor <peer-group-name> additional-paths send receive` | Peer group members inherit Add-Path send/receive capability |
| 21 | Advertise additional paths to peer group if used | Router | `neighbor <peer-group-name> advertise additional-paths all` | Peer group members receive eligible additional paths |
| 22 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 23 | Save configuration | Router | `copy running-config startup-config` | Add-Path configuration is saved |
| 24 | Soft clear outbound after send/advertise change | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound updates are refreshed without hard reset |
| 25 | Soft clear inbound after receive-capability/policy change | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound updates are reprocessed |
| 26 | Verify Add-Path config | Router | `show running-config | section router bgp` | Neighbor has `additional-paths` and `advertise additional-paths` under correct AFI |
| 27 | Verify neighbor capability negotiation | Router | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Neighbor shows Add-Path send/receive capability if supported |
| 28 | Verify local multiple paths | Router | `show bgp ipv4 unicast <prefix>` | Local router has multiple candidate paths |
| 29 | Verify additional-path marker locally | Router | `show bgp ipv4 unicast <prefix>` | Additional paths may show `a` or `add-path` marker |
| 30 | Verify advertised routes after Add-Path | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | More than one path for the prefix is advertised if supported |
| 31 | Verify peer receives multiple paths | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer sees multiple paths for the same prefix |
| 32 | Verify peer additional-path marker | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer output shows additional path marker or path IDs if supported |
| 33 | Verify peer still chooses one best path | Peer Router | `show bgp ipv4 unicast <prefix>` | One path remains best with `>` |
| 34 | Verify peer RIB install behavior | Peer Router | `show ip route <prefix>` | Peer installs only selected path unless multipath/PIC behavior separately installs more |
| 35 | Verify peer FIB behavior | Peer Router | `show ip cef <prefix>` | FIB reflects actual installed forwarding path |
| 36 | Verify no unexpected path leak | Peer Router | `show bgp ipv4 unicast neighbors <downstream-neighbor-ip> advertised-routes` | Additional paths are not leaked farther than intended |
| 37 | Verify route policy still applies | Router | `show route-map <rm-name>` | Policy counters reflect intended matches if policy is attached |
| 38 | Verify BGP logs | Router | `show logging | include BGP|ADJCHANGE` | No unexpected neighbor reset occurred |
# BGP_Additional_Paths_IOS_XE_Send_Receive_All_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> additional-paths send receive
  neighbor <neighbor-ip> advertise additional-paths all
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Additional_Paths_IOS_XE_Send_Only_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> additional-paths send
  neighbor <neighbor-ip> advertise additional-paths all
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Additional_Paths_IOS_XE_Receive_Only_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> additional-paths receive
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Additional_Paths_IOS_XE_Peer_Group_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <peer-group-name> additional-paths send receive
  neighbor <peer-group-name> advertise additional-paths all
 exit-address-family
end
clear bgp ipv4 unicast <peer-group-name> soft out
copy running-config startup-config
# BGP_Additional_Paths_Route_Reflector_Client_Skeleton
configure terminal
router bgp <local-asn>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <rr-client-ip> remote-as <local-asn>
 neighbor <rr-client-ip> update-source Loopback0
 address-family ipv4 unicast
  neighbor <rr-client-ip> activate
  neighbor <rr-client-ip> route-reflector-client
  neighbor <rr-client-ip> additional-paths send receive
  neighbor <rr-client-ip> advertise additional-paths all
 exit-address-family
end
clear bgp ipv4 unicast <rr-client-ip> soft out
copy running-config startup-config
# BGP_Additional_Paths_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show ip route <prefix>
show ip cef <prefix>
show running-config | section router bgp
show logging | include BGP|ADJCHANGE
# BGP_Additional_Paths_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP session | `show bgp ipv4 unicast summary` | Neighbor is established |
| Verify Add-Path config | `show running-config | section router bgp` | Neighbor has `additional-paths send`, `receive`, or `send receive` under correct AFI |
| Verify Add-Path advertisement config | `show running-config | section router bgp` | Neighbor has `advertise additional-paths all` if sending additional paths |
| Verify neighbor capability | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Add-Path send/receive capability appears if negotiated |
| Verify local multiple paths | `show bgp ipv4 unicast <prefix>` | Local router has multiple paths for same prefix |
| Verify additional-path marker | `show bgp ipv4 unicast <prefix>` | Output may show `a` or `add-path` for additional paths |
| Verify path IDs if displayed | `show bgp ipv4 unicast <prefix>` | Additional paths have unique path identifiers where platform displays them |
| Verify outbound advertisements | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | More than one eligible path is advertised to the neighbor |
| Verify peer received multiple paths | `show bgp ipv4 unicast <prefix>` on peer | Peer sees multiple paths for the same prefix |
| Verify peer best path | `show bgp ipv4 unicast <prefix>` on peer | One path still shows best with `>` |
| Verify peer RIB install | `show ip route <prefix>` on peer | Peer installs selected best path unless separate multipath/PIC install behavior is configured |
| Verify peer FIB install | `show ip cef <prefix>` on peer | FIB reflects actual forwarding path |
| Verify advertised routes downstream | `show bgp ipv4 unicast neighbors <downstream-neighbor-ip> advertised-routes` | Additional paths are propagated only where intended |
| Verify route policy hits | `show route-map <rm-name>` | Policy counters increment if route policy is involved |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected hard reset occurred |
# BGP_Additional_Paths_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current Add-Path config | Router | `show running-config | section router bgp` | Additional Paths commands are visible |
| 2 | Identify current advertised routes | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Current multiple-path advertisements are visible |
| 3 | Identify peer received paths | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer path visibility is known before rollback |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 7 | Remove additional path advertisement | Router | `no neighbor <neighbor-ip> advertise additional-paths all` | Router stops advertising all additional paths to neighbor |
| 8 | Remove send/receive capability | Router | `no neighbor <neighbor-ip> additional-paths send receive` | Add-Path send/receive is removed |
| 9 | Remove send-only capability if used | Router | `no neighbor <neighbor-ip> additional-paths send` | Add-Path send is removed |
| 10 | Remove receive-only capability if used | Router | `no neighbor <neighbor-ip> additional-paths receive` | Add-Path receive is removed |
| 11 | Remove peer-group advertisement if used | Router | `no neighbor <peer-group-name> advertise additional-paths all` | Peer group no longer advertises additional paths |
| 12 | Remove peer-group send/receive if used | Router | `no neighbor <peer-group-name> additional-paths send receive` | Peer group Add-Path capability is removed |
| 13 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 14 | Soft clear outbound | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Peer receives updated single-path advertisement behavior |
| 15 | Soft clear inbound if receive behavior changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are reprocessed |
| 16 | Verify config removal | Router | `show running-config | section router bgp` | Add-Path commands no longer appear |
| 17 | Verify advertised routes after rollback | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Only the best path is advertised |
| 18 | Verify peer path visibility after rollback | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer no longer sees extra paths from this router |
| 19 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Additional_Paths_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Peer still receives only one path | `show running-config | section router bgp` | Missing `neighbor <ip> advertise additional-paths all` or missing send capability |
| Local router has only one path | `show bgp ipv4 unicast <prefix>` | There are no extra local paths to advertise |
| Add-Path configured but not negotiated | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Peer does not support capability or direction does not match |
| Router sends but peer does not receive extra paths | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Local advertisement is wrong, peer lacks receive capability, or policy filters paths |
| Peer receives extra paths but installs only one | Peer `show ip route <prefix>` | Expected behavior unless multipath/PIC install behavior is separately configured |
| Extra paths are visible but not forwarded | Peer `show ip cef <prefix>` | Additional Paths gave visibility, not local FIB load sharing |
| Confused with multipath | `show ip route <prefix>` | Need `maximum-paths` for local ECMP installation, not Add-Path alone |
| Multipath configured but peer still sees one path | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Need Additional Paths, not `maximum-paths` |
| Route reflector still hides paths | RR `show running-config | section router bgp` | Add-Path not configured toward clients or not configured in correct AFI |
| Add-Path leaks too many paths | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | `advertise additional-paths all` may be broader than intended |
| Memory/control-plane load increases | `show bgp ipv4 unicast summary` and platform resource checks | More paths are being stored, advertised, and processed |
| Downstream router sees unexpected duplicate prefix paths | Downstream `show bgp ipv4 unicast <prefix>` | Add-Path enabled farther than intended |
| Route policy removes extra paths | `show route-map <rm-name>` | Outbound route map or filter denies alternate paths |
| Additional path marker is missing | `show bgp ipv4 unicast <prefix>` | Path is not being received as Add-Path or platform output differs |
| Session resets after enabling feature | `show logging | include BGP|ADJCHANGE` | Capability change may require renegotiation or hard session reset on some platforms |
| Soft clear does not update advertised paths | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Neighbor capability/session may need reset after capability change |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-additional-paths-final` | `BGP_Additional_Paths.md` | Primary lab for advertising more than one path for the same prefix, verifying Add-Path markers, and separating Add-Path from local multipath |
# Index
# Source_Basis
# BGP_Additional_Paths_Mental_Model
# BGP_Additional_Paths_Configuration_Checklist
# BGP_Additional_Paths_IOS_XE_Send_Receive_All_Skeleton
# BGP_Additional_Paths_IOS_XE_Send_Only_Skeleton
# BGP_Additional_Paths_IOS_XE_Receive_Only_Skeleton
# BGP_Additional_Paths_IOS_XE_Peer_Group_Skeleton
# BGP_Additional_Paths_Route_Reflector_Client_Skeleton
# BGP_Additional_Paths_Verification_Skeleton
# BGP_Additional_Paths_Verification_Commands
# BGP_Additional_Paths_Rollback
# BGP_Additional_Paths_Failure_Checks
# Attached_Labs

Blunt rule: Additional Paths is not load sharing. It gives another BGP speaker visibility into more than one path. Local forwarding still depends on what the receiving router installs in its RIB/FIB, which may require multipath or PIC behavior separately.