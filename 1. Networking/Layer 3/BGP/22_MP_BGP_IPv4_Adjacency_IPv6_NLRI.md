


MP_BGP_IPv4_Adjacency_IPv6_NLRI.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Main conceptual source for BGP, MP-BGP, IPv6 BGP, address families, attributes, and scalable BGP design |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Main practical source for IOS XE BGP configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22067-22067 | Multiprotocol BGP for IPv6 topic | Identifies MP-BGP for IPv6 as the relevant source section |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22268-22274 | MP-BGP AFI/SAFI and NLRI model | Supports AFI/SAFI separation, MP_REACH_NLRI, MP_UNREACH_NLRI, and MP-BGP carrying non-IPv4 NLRI |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23612-23646 | MP-BGP for IPv6 concepts | Supports IPv6 unicast AFI/SAFI, BGP TCP/179, capability negotiation, and IPv6 address-family activation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23654-23741 | Native IPv6 BGP configuration and verification | Supports `address-family ipv6`, `neighbor <ipv6> activate`, `show bgp ipv6 unicast neighbors`, and IPv6 AF capability verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23880-23943 | IPv4 BGP session carrying IPv6 routes | Supports `neighbor <ipv4> remote-as`, `address-family ipv6 unicast`, `neighbor <ipv4> activate`, and `show bgp ipv6 unicast summary` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23943-24021 | IPv4-mapped IPv6 next-hop problem | Confirms IPv6 routes advertised over an IPv4 BGP session receive `::FFFF:<IPv4>` next hops and do not install in the IPv6 RIB |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24021-24080 | Manual IPv6 next-hop correction | Supports `route-map <name> permit 10`, `set ipv6 next-hop <ipv6-next-hop>`, and outbound neighbor route-map application |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24080-24100 | Verified corrected IPv6 next-hop behavior | Confirms IPv6 BGP routes become valid and installable after setting a usable IPv6 next hop |
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Mental_Model
| Concept | Operational Meaning |
|---|---|
| MP-BGP | BGP extension that carries routing information for multiple address families |
| AFI | Address Family Identifier; identifies the network protocol family, such as IPv4 or IPv6 |
| SAFI | Subsequent Address Family Identifier; identifies the route type, such as unicast |
| IPv6 unicast AFI/SAFI | AFI 2, SAFI 1 |
| NLRI | Network Layer Reachability Information; the prefix information carried by BGP |
| MP_REACH_NLRI | BGP attribute used to advertise reachability for non-classic IPv4 address families |
| MP_UNREACH_NLRI | BGP attribute used to withdraw reachability for MP-BGP address families |
| Transport session | The TCP BGP session used to exchange updates |
| Route family | The NLRI carried inside the BGP update, such as IPv6 unicast |
| IPv4 adjacency carrying IPv6 NLRI | BGP peers form TCP/179 over IPv4 addresses but exchange IPv6 unicast routes under `address-family ipv6 unicast` |
| Neighbor activation | The IPv4 neighbor must be activated under the IPv6 address family before IPv6 routes are exchanged |
| `no bgp default ipv4-unicast` | Prevents automatic IPv4 unicast activation and forces explicit address-family activation |
| IPv6 next-hop problem | IPv6 NLRI over an IPv4 session may receive an IPv4-mapped IPv6 next hop like `::FFFF:10.12.1.2` |
| Invalid forwarding next hop | `::FFFF:<IPv4>` is not a usable IPv6 forwarding next hop for the IPv6 RIB |
| Manual next-hop repair | Use an outbound route map with `set ipv6 next-hop <reachable-ipv6-next-hop>` |
| IPv6 RIB install | IPv6 BGP route installs only after the IPv6 next hop is valid and reachable |
| Local IPv6 origination | Locally originated IPv6 prefixes show next hop `::` and weight `32768` |
| Exact IPv6 network statement | `network <ipv6-prefix>/<prefix-length>` requires the exact IPv6 prefix to exist in the local IPv6 RIB |
| IPv6 forwarding | `ipv6 unicast-routing` must be enabled for the router to forward IPv6 traffic |
| Blunt rule | An IPv4 BGP session can carry IPv6 prefixes, but the IPv6 next hop still has to be a real IPv6 forwarding address. If you see `::FFFF:<IPv4>` as next hop, expect RIB failure until you fix it |
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm lab intent | Notes | `IPv4 BGP adjacency carrying IPv6 unicast NLRI` | You are not building native IPv6 BGP peering |
| 2 | Identify local AS | Router / Notes | `<local-asn>` | Local AS is known |
| 3 | Identify remote AS | Router / Notes | `<remote-asn>` | Remote AS is known |
| 4 | Identify IPv4 BGP neighbor address | Router / Notes | `<neighbor-ipv4>` | TCP BGP session endpoint is known |
| 5 | Identify local IPv4 BGP source | Router / Notes | `<local-ipv4-source>` | Local source for BGP session is known |
| 6 | Identify IPv6 next hop to advertise | Router / Notes | `<local-ipv6-next-hop>` | Real IPv6 next-hop address is known |
| 7 | Identify peer IPv6 next hop | Router / Notes | `<peer-ipv6-next-hop>` | Peer has a real IPv6 address to use as next hop |
| 8 | Identify IPv6 prefixes to advertise | Router / Notes | `<ipv6-prefix>/<prefix-length>` | IPv6 NLRI list is known |
| 9 | Enable IPv6 forwarding | Router | `ipv6 unicast-routing` | Router can forward IPv6 traffic |
| 10 | Verify IPv4 interface state | Router | `show ip interface brief` | IPv4 BGP path interfaces are up/up |
| 11 | Verify IPv6 interface state | Router | `show ipv6 interface brief` | IPv6 interfaces and next-hop interfaces are up/up |
| 12 | Verify IPv4 reachability to BGP peer | Router | `ping <neighbor-ipv4> source <local-ipv4-source>` | IPv4 BGP neighbor is reachable |
| 13 | Verify IPv6 next-hop reachability on local link | Router | `ping <peer-ipv6-next-hop> source <local-ipv6-next-hop>` | Peer IPv6 next hop is reachable |
| 14 | Verify TCP/179 is not blocked | Router / Path | `show access-lists` or path ACL check | BGP TCP session is not filtered |
| 15 | Verify exact local IPv6 route for network statement | Router | `show ipv6 route <ipv6-prefix>/<prefix-length>` | Exact IPv6 prefix exists if using `network` |
| 16 | Create static IPv6 route if exact prefix is needed for lab origination | Router | `ipv6 route <ipv6-prefix>/<prefix-length> <next-hop-or-null0>` | Exact IPv6 route exists in RIB |
| 17 | Review existing BGP config | Router | `show running-config | section router bgp` | Existing neighbors, AFIs, and policy are visible |
| 18 | Review existing route maps | Router | `show running-config | section route-map` | Existing next-hop policy is visible |
| 19 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 20 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 21 | Set deterministic BGP router ID | Router | `bgp router-id <router-id>` | Router ID is stable |
| 22 | Enable neighbor logging | Router | `bgp log-neighbor-changes` | BGP neighbor changes are logged |
| 23 | Disable automatic IPv4 unicast activation | Router | `no bgp default ipv4-unicast` | Neighbors must be activated explicitly under AFIs |
| 24 | Configure IPv4 BGP neighbor | Router | `neighbor <neighbor-ipv4> remote-as <remote-asn>` | IPv4 TCP BGP neighbor is defined |
| 25 | Add neighbor description | Router | `neighbor <neighbor-ipv4> description <description>` | Peer purpose is documented |
| 26 | Configure update-source if using loopback IPv4 peering | Router | `neighbor <neighbor-ipv4> update-source Loopback0` | BGP session sources from loopback |
| 27 | Configure eBGP multihop if IPv4 eBGP peer is not directly connected | Router | `neighbor <neighbor-ipv4> ebgp-multihop <ttl>` | Non-direct IPv4 eBGP session can form |
| 28 | Enter IPv6 unicast address family | Router | `address-family ipv6 unicast` | Router enters MP-BGP IPv6 unicast AFI |
| 29 | Activate IPv4 neighbor under IPv6 AFI | Router | `neighbor <neighbor-ipv4> activate` | IPv4 BGP session can carry IPv6 unicast NLRI |
| 30 | Originate IPv6 prefix with network statement | Router | `network <ipv6-prefix>/<prefix-length>` | Exact IPv6 RIB prefix is injected into BGP |
| 31 | Redistribute connected IPv6 routes only if lab design requires it | Router | `redistribute connected` | Connected IPv6 prefixes enter BGP |
| 32 | Configure IPv6 aggregate if lab requires summary | Router | `aggregate-address <ipv6-prefix>/<prefix-length> summary-only` | IPv6 BGP aggregate is created |
| 33 | Exit BGP address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 34 | Create outbound route map to set IPv6 next hop | Router | `route-map <rm-set-ipv6-nh> permit 10` | Route map exists |
| 35 | Set real IPv6 next hop | Router | `set ipv6 next-hop <local-ipv6-next-hop>` | BGP updates advertise reachable IPv6 next hop |
| 36 | Reenter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 37 | Enter IPv6 unicast AFI again | Router | `address-family ipv6 unicast` | Router enters IPv6 unicast AFI |
| 38 | Apply route map outbound to IPv4 neighbor | Router | `neighbor <neighbor-ipv4> route-map <rm-set-ipv6-nh> out` | IPv6 NLRI sent to IPv4 neighbor carries valid IPv6 next hop |
| 39 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 40 | Save configuration | Router | `copy running-config startup-config` | MP-BGP config is saved |
| 41 | Soft clear IPv6 unicast outbound updates | Router | `clear bgp ipv6 unicast <neighbor-ipv4> soft out` | Peer receives updated IPv6 next-hop attribute |
| 42 | Verify IPv4 BGP session carrying IPv6 AFI | Router | `show bgp ipv6 unicast summary` | IPv4 neighbor appears with IPv6 prefix count |
| 43 | Verify IPv6 AF capability negotiation | Router | `show bgp ipv6 unicast neighbors <neighbor-ipv4>` | IPv6 Unicast capability is advertised and received |
| 44 | Verify local IPv6 BGP table | Router | `show bgp ipv6 unicast` | IPv6 prefixes appear in BGP table |
| 45 | Check for bad IPv4-mapped next hop | Router | `show bgp ipv6 unicast` | `::FFFF:<neighbor-ipv4>` should not remain after route-map fix |
| 46 | Verify corrected IPv6 next hop | Router | `show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>` | Next hop is a real IPv6 address |
| 47 | Verify IPv6 route installs in RIB | Router | `show ipv6 route bgp` | BGP-learned IPv6 routes appear as `B` routes |
| 48 | Verify specific IPv6 route install | Router | `show ipv6 route <ipv6-prefix>/<prefix-length>` | IPv6 route resolves through valid next hop |
| 49 | Verify IPv6 CEF forwarding | Router | `show ipv6 cef <ipv6-prefix>/<prefix-length>` | FIB points to expected IPv6 next hop/interface |
| 50 | Verify advertised IPv6 routes | Router | `show bgp ipv6 unicast neighbors <neighbor-ipv4> advertised-routes` | Expected IPv6 prefixes are advertised |
| 51 | Verify peer received IPv6 routes | Peer Router | `show bgp ipv6 unicast` | Peer sees advertised IPv6 prefixes |
| 52 | Verify peer installed IPv6 routes | Peer Router | `show ipv6 route bgp` | Peer installs IPv6 BGP routes after next-hop correction |
| 53 | Test IPv6 connectivity | Router / Host | `ping <remote-ipv6-destination> source <local-ipv6-source>` | IPv6 traffic succeeds |
| 54 | Test IPv6 forwarding path | Router | `traceroute <remote-ipv6-destination> source <local-ipv6-source>` | Path follows expected BGP next hop |
| 55 | Verify logs | Router | `show logging | include BGP|ADJCHANGE` | No unexpected neighbor reset occurred |
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Basic_Skeleton
configure terminal
ipv6 unicast-routing
router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <neighbor-ipv4> remote-as <remote-asn>
 neighbor <neighbor-ipv4> description <description>
 address-family ipv6 unicast
  neighbor <neighbor-ipv4> activate
  network <ipv6-prefix>/<prefix-length>
 exit-address-family
end
copy running-config startup-config
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_With_Manual_IPv6_Next_Hop_Skeleton
configure terminal
ipv6 unicast-routing
route-map <rm-set-ipv6-nh> permit 10
 set ipv6 next-hop <local-ipv6-next-hop>
router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <neighbor-ipv4> remote-as <remote-asn>
 neighbor <neighbor-ipv4> description <description>
 address-family ipv6 unicast
  neighbor <neighbor-ipv4> activate
  neighbor <neighbor-ipv4> route-map <rm-set-ipv6-nh> out
  network <ipv6-prefix>/<prefix-length>
 exit-address-family
end
clear bgp ipv6 unicast <neighbor-ipv4> soft out
copy running-config startup-config
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Loopback_IPv4_Peering_Skeleton
configure terminal
ipv6 unicast-routing
route-map <rm-set-ipv6-nh> permit 10
 set ipv6 next-hop <local-ipv6-next-hop>
router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <neighbor-ipv4-loopback> remote-as <remote-asn>
 neighbor <neighbor-ipv4-loopback> update-source Loopback0
 neighbor <neighbor-ipv4-loopback> ebgp-multihop <ttl>
 address-family ipv6 unicast
  neighbor <neighbor-ipv4-loopback> activate
  neighbor <neighbor-ipv4-loopback> route-map <rm-set-ipv6-nh> out
  network <ipv6-prefix>/<prefix-length>
 exit-address-family
end
clear bgp ipv6 unicast <neighbor-ipv4-loopback> soft out
copy running-config startup-config
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Redistribute_Connected_Skeleton
configure terminal
ipv6 unicast-routing
route-map <rm-set-ipv6-nh> permit 10
 set ipv6 next-hop <local-ipv6-next-hop>
router bgp <local-asn>
 bgp router-id <router-id>
 no bgp default ipv4-unicast
 neighbor <neighbor-ipv4> remote-as <remote-asn>
 address-family ipv6 unicast
  neighbor <neighbor-ipv4> activate
  neighbor <neighbor-ipv4> route-map <rm-set-ipv6-nh> out
  redistribute connected
 exit-address-family
end
clear bgp ipv6 unicast <neighbor-ipv4> soft out
copy running-config startup-config
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Aggregate_Summary_Skeleton
configure terminal
ipv6 unicast-routing
route-map <rm-set-ipv6-nh> permit 10
 set ipv6 next-hop <local-ipv6-next-hop>
router bgp <local-asn>
 bgp router-id <router-id>
 no bgp default ipv4-unicast
 neighbor <neighbor-ipv4> remote-as <remote-asn>
 address-family ipv6 unicast
  neighbor <neighbor-ipv4> activate
  neighbor <neighbor-ipv4> route-map <rm-set-ipv6-nh> out
  network <component-ipv6-prefix-1>/<prefix-length-1>
  network <component-ipv6-prefix-2>/<prefix-length-2>
  aggregate-address <aggregate-ipv6-prefix>/<aggregate-prefix-length> summary-only
 exit-address-family
end
clear bgp ipv6 unicast <neighbor-ipv4> soft out
copy running-config startup-config
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Static_Route_For_Network_Origination_Skeleton
configure terminal
ipv6 unicast-routing
ipv6 route <ipv6-prefix>/<prefix-length> <next-hop-or-null0>
router bgp <local-asn>
 address-family ipv6 unicast
  network <ipv6-prefix>/<prefix-length>
 exit-address-family
end
copy running-config startup-config
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Verification_Skeleton
show ip interface brief
show ipv6 interface brief
show bgp ipv4 unicast summary
show bgp ipv6 unicast summary
show bgp ipv6 unicast neighbors <neighbor-ipv4>
show bgp ipv6 unicast
show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>
show ipv6 route bgp
show ipv6 route <ipv6-prefix>/<prefix-length>
show ipv6 cef <ipv6-prefix>/<prefix-length>
show bgp ipv6 unicast neighbors <neighbor-ipv4> advertised-routes
show running-config | section router bgp
show running-config | section route-map
ping <remote-ipv6-destination> source <local-ipv6-source>
traceroute <remote-ipv6-destination> source <local-ipv6-source>
show logging | include BGP|ADJCHANGE
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify IPv4 interface state | `show ip interface brief` | IPv4 BGP transport interfaces are up/up |
| Verify IPv6 interface state | `show ipv6 interface brief` | IPv6 forwarding interfaces are up/up |
| Verify IPv4 BGP transport session | `show bgp ipv4 unicast summary` | IPv4 BGP session may be established if IPv4 AFI is active |
| Verify IPv6 AFI session over IPv4 neighbor | `show bgp ipv6 unicast summary` | IPv4 neighbor appears under IPv6 unicast AFI |
| Verify IPv6 capability negotiation | `show bgp ipv6 unicast neighbors <neighbor-ipv4>` | IPv6 Unicast capability is advertised and received |
| Verify IPv6 neighbor prefix count | `show bgp ipv6 unicast summary` | `State/PfxRcd` shows received IPv6 prefix count |
| Verify IPv6 BGP table | `show bgp ipv6 unicast` | IPv6 prefixes appear in BGP table |
| Verify specific IPv6 prefix | `show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>` | Path attributes, AS path, and next hop are visible |
| Detect bad IPv4-mapped next hop | `show bgp ipv6 unicast` | `::FFFF:<IPv4>` means IPv6 next hop is not valid for forwarding |
| Verify corrected IPv6 next hop | `show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>` | Next hop is a real reachable IPv6 address |
| Verify local IPv6 origination | `show bgp ipv6 unicast <local-ipv6-prefix>/<length>` | Locally originated route shows next hop `::` and high local weight |
| Verify exact IPv6 route for network statement | `show ipv6 route <ipv6-prefix>/<prefix-length>` | Exact route exists in IPv6 RIB |
| Verify IPv6 BGP RIB install | `show ipv6 route bgp` | BGP-learned IPv6 prefixes appear |
| Verify specific IPv6 route install | `show ipv6 route <ipv6-prefix>/<prefix-length>` | Route installs through expected next hop/interface |
| Verify IPv6 CEF | `show ipv6 cef <ipv6-prefix>/<prefix-length>` | FIB has usable forwarding entry |
| Verify outbound route map | `show running-config | section route-map` | Route map contains `set ipv6 next-hop <ipv6-next-hop>` |
| Verify route map attached under IPv6 AFI | `show running-config | section router bgp` | Neighbor route-map is applied under `address-family ipv6 unicast` |
| Verify advertised IPv6 routes | `show bgp ipv6 unicast neighbors <neighbor-ipv4> advertised-routes` | Expected IPv6 prefixes are advertised to IPv4 neighbor |
| Verify peer BGP table | `show bgp ipv6 unicast` on peer | Peer sees IPv6 prefixes |
| Verify peer IPv6 RIB | `show ipv6 route bgp` on peer | Peer installs IPv6 BGP routes |
| Verify IPv6 data-plane reachability | `ping <remote-ipv6-destination> source <local-ipv6-source>` | IPv6 ping succeeds |
| Verify IPv6 forwarding path | `traceroute <remote-ipv6-destination> source <local-ipv6-source>` | IPv6 path follows expected BGP next hop |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected BGP reset occurred |
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify MP-BGP IPv6 config | Router | `show running-config | section router bgp` | IPv4 neighbor, IPv6 AFI activation, network statements, and route maps are visible |
| 2 | Identify IPv6 next-hop route map | Router | `show running-config | section route-map` | `set ipv6 next-hop` policy is visible |
| 3 | Identify advertised IPv6 routes | Router | `show bgp ipv6 unicast neighbors <neighbor-ipv4> advertised-routes` | Current IPv6 advertisements are known |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 6 | Enter IPv6 unicast AFI | Router | `address-family ipv6 unicast` | Router enters IPv6 unicast AFI |
| 7 | Remove outbound IPv6 next-hop route map | Router | `no neighbor <neighbor-ipv4> route-map <rm-set-ipv6-nh> out` | Neighbor no longer has next-hop policy attached |
| 8 | Remove IPv6 network statement if lab-only | Router | `no network <ipv6-prefix>/<prefix-length>` | IPv6 prefix is no longer originated |
| 9 | Remove IPv6 aggregate if lab-only | Router | `no aggregate-address <ipv6-prefix>/<prefix-length> summary-only` | IPv6 aggregate is removed |
| 10 | Deactivate IPv4 neighbor under IPv6 AFI | Router | `no neighbor <neighbor-ipv4> activate` | IPv4 neighbor no longer exchanges IPv6 unicast NLRI |
| 11 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 12 | Remove IPv4 neighbor if full lab rollback is intended | Router | `no neighbor <neighbor-ipv4> remote-as <remote-asn>` | BGP neighbor is removed |
| 13 | Remove route map sequence if lab-only | Router | `no route-map <rm-set-ipv6-nh> permit 10` | IPv6 next-hop route map is removed |
| 14 | Remove lab-only static IPv6 route | Router | `no ipv6 route <ipv6-prefix>/<prefix-length> <next-hop-or-null0>` | Static route used for origination is removed |
| 15 | Soft clear IPv6 outbound updates | Router | `clear bgp ipv6 unicast <neighbor-ipv4> soft out` | Peer receives updated IPv6 advertisement state |
| 16 | Verify IPv6 AFI removal | Router | `show bgp ipv6 unicast summary` | Neighbor no longer appears under IPv6 AFI if deactivated |
| 17 | Verify route withdrawal | Peer Router | `show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>` | Peer no longer sees withdrawn IPv6 prefix |
| 18 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| IPv4 BGP session is up but no IPv6 prefixes exchange | `show bgp ipv6 unicast summary` | Neighbor not activated under `address-family ipv6 unicast` |
| Neighbor does not show under IPv6 unicast summary | `show running-config | section router bgp` | Missing `neighbor <ipv4> activate` under IPv6 AFI |
| IPv6 route appears in BGP but not IPv6 RIB | `show bgp ipv6 unicast <prefix>` and `show ipv6 route <prefix>` | IPv6 next hop is invalid or unreachable |
| BGP next hop shows `::FFFF:<IPv4>` | `show bgp ipv6 unicast` | IPv6 NLRI was carried over IPv4 session without manual IPv6 next-hop policy |
| IPv6 ping fails with no route | `show ipv6 route <destination-prefix>` | IPv6 BGP route did not install |
| IPv6 route installs after route map | `show bgp ipv6 unicast <prefix>` | Expected behavior after `set ipv6 next-hop` fixes the forwarding next hop |
| Route map has no effect | `show running-config | section router bgp` | Route map attached under wrong AFI, wrong neighbor, or wrong direction |
| Route map exists but next hop still wrong | `show running-config | section route-map` | Missing `set ipv6 next-hop`, wrong IPv6 address, or outbound refresh not run |
| IPv6 next hop is set but unreachable | `show ipv6 route <ipv6-next-hop>` | IPv6 adjacency, connected route, static route, or IGP reachability is missing |
| IPv6 `network` command does not originate route | `show ipv6 route <ipv6-prefix>/<prefix-length>` | Exact IPv6 prefix is not in local IPv6 RIB |
| Redistribute connected advertises too much | `show bgp ipv6 unicast` | Redistribution was used without filtering |
| IPv6 aggregate appears but components disappear | `show bgp ipv6 unicast` | `summary-only` suppresses component routes |
| IPv6 AF capability not negotiated | `show bgp ipv6 unicast neighbors <neighbor-ipv4>` | Peer does not support or is not configured for IPv6 unicast AFI |
| IPv4 session fails before MP-BGP matters | `show bgp ipv6 unicast summary` and IPv4 ping | IPv4 transport reachability, TCP/179, `remote-as`, or update-source is wrong |
| Router ID missing in IPv6-only design | `show running-config | section router bgp` | BGP router ID must be set manually when no IPv4 address exists |
| `show ip bgp` looks empty and causes confusion | `show bgp ipv6 unicast` | Wrong command family checked; IPv6 NLRI is under IPv6 unicast AFI |
| Peer receives route but does not install | Peer `show bgp ipv6 unicast <prefix>` and `show ipv6 route <prefix>` | Peer has bad next hop, policy issue, or better route |
| Data plane fails while control plane looks fixed | `show ipv6 cef <prefix>` and return-path check | Forward path, return path, ACL, or host IPv6 gateway is broken |
| Soft clear did not update next hop | `clear bgp ipv6 unicast <neighbor-ipv4> soft out` | Wrong AFI was cleared or wrong neighbor was refreshed |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used instead of soft outbound refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `multiprotocol-bgp-ipv4-adjacency-ipv6-prefixes-final` | `MP_BGP_IPv4_Adjacency_IPv6_NLRI.md` | Primary lab for IPv4 BGP adjacency carrying IPv6 unicast NLRI, IPv6 AFI activation, IPv4-mapped IPv6 next-hop failure, and manual IPv6 next-hop repair |
# Index
# Source_Basis
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Mental_Model
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Configuration_Checklist
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Basic_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_With_Manual_IPv6_Next_Hop_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Loopback_IPv4_Peering_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Redistribute_Connected_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Aggregate_Summary_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Static_Route_For_Network_Origination_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Verification_Skeleton
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Verification_Commands
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Rollback
# MP_BGP_IPv4_Adjacency_IPv6_NLRI_Failure_Checks
# Attached_Labs

Blunt rule: the IPv4 BGP session is only the transport. It does not magically create an IPv6 forwarding next hop. For this lab, the key failure is ::FFFF:<IPv4> as the IPv6 next hop. Fix it with an outbound route map using set ipv6 next-hop <real-ipv6-next-hop>, then verify with show bgp ipv6 unicast and show ipv6 route bgp.