BGP_Multipath_And_Load_Sharing.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP attributes, path selection, multipath, route reflection, confederations, and policy behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP multipath configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26651-26659 | BGP equal-cost multipathing overview | Confirms BGP keeps multiple paths in Loc-RIB and can present more than one path to the RIB for installation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26797-26834 | BGP best-path order | Supports the attributes that must line up before multipath can install extra paths |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 27838-27872 | BGP multipath behavior and requirements | Confirms eBGP multipath, iBGP multipath, eiBGP multipath, same attribute requirements, and `maximum-paths` / `maximum-paths ibgp` syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28088-28094 | Multipath command summary | Supports `maximum-paths <number>` for eBGP and `maximum-paths ibgp <number>` for iBGP |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22803 and 27546 | BGP table status code `m` | Supports reading `m` as a multipath marker in BGP table output |
| `All_combined_part3.md` lines 58495-58503 | Multipath same attribute and same-AS expectations | Supports equal-cost paths, identical AS_PATH requirements, and default eBGP/confederation external behavior |
| `All_combined_part3.md` lines 73947-74063 | eBGP multipath enterprise/provider example | Supports multiple eBGP sessions over multiple links, `maximum-paths 3`, multiple RIB next hops, and memory/scale caution |
# BGP_Multipath_And_Load_Sharing_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP multipath | Allows more than one eligible BGP path to be installed into the routing table |
| ECMP | Equal-cost multipath forwarding across multiple installed next hops |
| Local forwarding feature | BGP multipath affects this router's RIB/FIB installation behavior |
| Not Additional Paths | Multipath does not advertise extra paths to peers; it installs extra paths locally |
| Best path still exists | BGP still selects a best path even when additional eligible paths are installed |
| Only best path advertised | BGP advertises only the best path to peers unless Additional Paths is configured separately |
| eBGP multipath | Installs multiple eligible external BGP paths |
| iBGP multipath | Installs multiple eligible internal BGP paths |
| eiBGP multipath | Platform-dependent design where eBGP and iBGP paths can both be considered |
| `maximum-paths <n>` | Enables eBGP multipath and allows up to `n` eBGP paths into the RIB |
| `maximum-paths ibgp <n>` | Enables iBGP multipath and allows up to `n` iBGP paths into the RIB |
| Same weight | Candidate paths must match weight |
| Same local preference | Candidate paths must match local preference |
| Same AS path length | Candidate paths must usually have the same AS path length |
| Same AS path content | Candidate paths usually need matching AS path content unless relaxation is supported and configured |
| Same origin | Candidate paths must match origin type |
| Same MED | Candidate paths must match MED where MED comparison applies |
| Same advertisement method | Candidate paths should be both eBGP or both iBGP unless eiBGP behavior is explicitly supported |
| iBGP IGP cost requirement | iBGP multipath usually requires matching IGP metric to the BGP next hop |
| RIB install limit | `maximum-paths` controls how many eligible paths can install |
| FIB load sharing | Actual forwarding is usually per-flow CEF hashing, not per-packet load balancing |
| `m` BGP table marker | Indicates a BGP path is used as multipath |
| Same-AS eBGP multipath | Common case where multiple eBGP sessions to the same neighboring AS carry equivalent routes |
| Different-AS eBGP multipath | Requires platform support and usually AS path multipath relaxation |
| Memory/scale caution | More installed paths mean more RIB/FIB state and potentially more memory usage |
| Blunt rule | Multipath does not make unequal BGP paths equal. It only installs additional paths that already match the multipath requirements |
# BGP_Multipath_And_Load_Sharing_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Relevant neighbors show a number under `State/PfxRcd` |
| 2 | Confirm target prefix has multiple BGP paths | Router | `show bgp ipv4 unicast <prefix>` | Prefix shows more than one candidate path |
| 3 | Confirm all candidate next hops are reachable | Router | `show ip route <next-hop-ip>` | Each BGP next hop is reachable |
| 4 | Confirm current best path | Router | `show bgp ipv4 unicast <prefix>` | One path is marked best with `>` |
| 5 | Confirm current route install | Router | `show ip route <prefix>` | Before multipath, usually only one BGP next hop is installed |
| 6 | Confirm current CEF forwarding | Router | `show ip cef <prefix>` | Current forwarding next hop is visible |
| 7 | Identify multipath type | Notes | `eBGP same-AS`, `eBGP different-AS`, or `iBGP` | Multipath design is clear before configuration |
| 8 | Confirm eBGP same-AS case | Router | `show bgp ipv4 unicast <prefix>` | Candidate eBGP paths come from the same neighboring AS |
| 9 | Confirm eBGP different-AS case | Router | `show bgp ipv4 unicast <prefix>` | Candidate eBGP paths come from different neighboring ASNs |
| 10 | Confirm iBGP case | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths are learned through iBGP |
| 11 | Confirm weight match | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths have the same weight |
| 12 | Confirm local preference match | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths have the same local preference |
| 13 | Confirm AS path length match | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths have same AS path length |
| 14 | Confirm AS path content match for normal multipath | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths have same AS path content unless multipath relaxation is used |
| 15 | Confirm origin match | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths have same origin code |
| 16 | Confirm MED match | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths have same MED where MED is compared |
| 17 | Confirm advertisement method match | Router | `show bgp ipv4 unicast <prefix>` | Candidate paths are all eBGP or all iBGP unless eiBGP multipath is intentionally used |
| 18 | Confirm iBGP next-hop IGP cost match | Router | `show ip route <next-hop-ip>` | IGP metric to each iBGP next hop is equal if required |
| 19 | Confirm route policy is not making paths unequal | Router | `show running-config | include route-map|prefix-list|filter-list|distribute-list` | Policy does not set different weight, local preference, MED, or AS path behavior |
| 20 | Review current BGP configuration | Router | `show running-config | section router bgp` | Existing multipath and best-path knobs are visible |
| 21 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 22 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 23 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 24 | Enable eBGP multipath | Router | `maximum-paths <number>` | Router may install up to `<number>` eligible eBGP paths |
| 25 | Enable iBGP multipath | Router | `maximum-paths ibgp <number>` | Router may install up to `<number>` eligible iBGP paths |
| 26 | Enable different-AS eBGP multipath relaxation if supported and required | Router | `bgp bestpath as-path multipath-relax` | Router can consider eBGP paths with different AS path content if other requirements align |
| 27 | Avoid AS path relaxation unless different-AS load sharing is intentional | Notes | `No multipath-relax unless required` | Prevents accidental load sharing across dissimilar external paths |
| 28 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 29 | Save configuration | Router | `copy running-config startup-config` | Multipath configuration is saved |
| 30 | Soft clear inbound if policy changed route attributes | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Routes are reprocessed without hard reset |
| 31 | Verify multipath config | Router | `show running-config | section router bgp` | `maximum-paths` commands appear under correct AFI |
| 32 | Verify BGP table multipath marker | Router | `show bgp ipv4 unicast <prefix>` | Additional installed paths show `m` or multipath status |
| 33 | Verify route table has multiple next hops | Router | `show ip route <prefix>` | Multiple BGP routing descriptor blocks appear |
| 34 | Verify CEF has multiple next hops | Router | `show ip cef <prefix>` | CEF shows multiple forwarding adjacencies |
| 35 | Verify all installed paths are equivalent | Router | `show bgp ipv4 unicast <prefix>` | Installed multipath candidates still match required attributes |
| 36 | Verify same-AS eBGP multipath | Router | `show ip route <prefix>` | Multiple eBGP next hops from same neighboring AS install |
| 37 | Verify different-AS eBGP multipath | Router | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Multiple eBGP paths from different ASNs install only if relaxation is configured and supported |
| 38 | Verify iBGP multipath | Router | `show ip route <prefix>` | Multiple iBGP next hops install if eligible |
| 39 | Test forwarding path repeatedly | Router | `traceroute <destination-ip> source <source-ip>` | Different flows may use different next hops depending on hashing |
| 40 | Verify load sharing from traffic counters | Router | `show interfaces <interface>` | Candidate egress interfaces show traffic movement during test |
| 41 | Verify no unexpected route advertisement change | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Only best path is advertised unless Additional Paths is separately configured |
| 42 | Verify no RIB failure | Router | `show bgp ipv4 unicast <prefix>` | Multipath candidates are not blocked by RIB failure |
| 43 | Verify no next-hop failure | Router | `show ip route <next-hop-ip>` | Every installed BGP next hop remains reachable |
| 44 | Verify logs | Router | `show logging | include BGP|ADJCHANGE` | No unexpected neighbor reset occurred |
# BGP_Multipath_And_Load_Sharing_eBGP_Same_AS_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  maximum-paths <number>
 exit-address-family
end
copy running-config startup-config
# BGP_Multipath_And_Load_Sharing_iBGP_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  maximum-paths ibgp <number>
 exit-address-family
end
copy running-config startup-config
# BGP_Multipath_And_Load_Sharing_eBGP_Different_AS_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  maximum-paths <number>
  bgp bestpath as-path multipath-relax
 exit-address-family
end
copy running-config startup-config
# BGP_Multipath_And_Load_Sharing_eBGP_Multiple_Links_To_Same_AS_Skeleton
configure terminal
router bgp <local-asn>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <peer-link1-ip> remote-as <remote-asn>
 neighbor <peer-link2-ip> remote-as <remote-asn>
 neighbor <peer-link3-ip> remote-as <remote-asn>
 address-family ipv4 unicast
  neighbor <peer-link1-ip> activate
  neighbor <peer-link2-ip> activate
  neighbor <peer-link3-ip> activate
  maximum-paths 3
 exit-address-family
end
copy running-config startup-config
# BGP_Multipath_And_Load_Sharing_iBGP_Loopback_Multipath_Skeleton
configure terminal
router bgp <local-asn>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <ibgp-peer1-loopback> remote-as <local-asn>
 neighbor <ibgp-peer1-loopback> update-source Loopback0
 neighbor <ibgp-peer2-loopback> remote-as <local-asn>
 neighbor <ibgp-peer2-loopback> update-source Loopback0
 address-family ipv4 unicast
  neighbor <ibgp-peer1-loopback> activate
  neighbor <ibgp-peer2-loopback> activate
  maximum-paths ibgp 2
 exit-address-family
end
copy running-config startup-config
# BGP_Multipath_And_Load_Sharing_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show ip route <prefix>
show ip cef <prefix>
show ip route <next-hop-ip>
show running-config | section router bgp
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show interfaces <egress-interface>
show logging | include BGP|ADJCHANGE
# BGP_Multipath_And_Load_Sharing_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP neighbor state | `show bgp ipv4 unicast summary` | All relevant neighbors are established |
| Verify multiple candidate paths | `show bgp ipv4 unicast <prefix>` | Prefix has multiple BGP paths |
| Verify multipath marker | `show bgp ipv4 unicast <prefix>` | Additional installed paths show `m` or multipath status |
| Verify current best path | `show bgp ipv4 unicast <prefix>` | One path remains marked best with `>` |
| Verify weight match | `show bgp ipv4 unicast <prefix>` | Candidate paths have same weight |
| Verify local preference match | `show bgp ipv4 unicast <prefix>` | Candidate paths have same local preference |
| Verify AS path match | `show bgp ipv4 unicast <prefix>` | AS path length/content matches unless relaxation is used |
| Verify origin match | `show bgp ipv4 unicast <prefix>` | Origin code matches across candidates |
| Verify MED match | `show bgp ipv4 unicast <prefix>` | MED matches across candidates where MED comparison applies |
| Verify next-hop reachability | `show ip route <next-hop-ip>` | Each BGP next hop is reachable |
| Verify eBGP multipath config | `show running-config | section router bgp` | `maximum-paths <number>` appears under IPv4 unicast AFI |
| Verify iBGP multipath config | `show running-config | section router bgp` | `maximum-paths ibgp <number>` appears under IPv4 unicast AFI |
| Verify different-AS relaxation if used | `show running-config | section router bgp` | `bgp bestpath as-path multipath-relax` appears only if intended |
| Verify RIB has multiple next hops | `show ip route <prefix>` | Multiple BGP routing descriptor blocks appear |
| Verify FIB has multiple next hops | `show ip cef <prefix>` | Multiple CEF next hops appear |
| Verify egress traffic movement | `show interfaces <egress-interface>` | Counters move on expected load-shared interfaces |
| Verify advertised routes unchanged | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Router still advertises only the best path unless Additional Paths is configured |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected neighbor reset occurred |
# BGP_Multipath_And_Load_Sharing_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current multipath config | Router | `show running-config | section router bgp` | `maximum-paths`, `maximum-paths ibgp`, or multipath-relax config is visible |
| 2 | Identify current installed paths | Router | `show ip route <prefix>` | Current multiple RIB next hops are visible |
| 3 | Identify BGP multipath candidates | Router | `show bgp ipv4 unicast <prefix>` | Multipath paths and best path are visible |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 7 | Remove eBGP multipath | Router | `no maximum-paths <number>` | eBGP multipath limit is removed |
| 8 | Remove iBGP multipath | Router | `no maximum-paths ibgp <number>` | iBGP multipath limit is removed |
| 9 | Remove different-AS multipath relaxation | Router | `no bgp bestpath as-path multipath-relax` | AS path multipath relaxation is removed |
| 10 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 11 | Soft clear inbound if route attributes or policy were changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Routes are reprocessed without hard reset |
| 12 | Verify only one BGP path installs | Router | `show ip route <prefix>` | RIB returns to single best path unless another multipath feature remains |
| 13 | Verify BGP table after rollback | Router | `show bgp ipv4 unicast <prefix>` | Multipath marker is absent from extra paths |
| 14 | Verify CEF after rollback | Router | `show ip cef <prefix>` | FIB uses one selected next hop |
| 15 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Multipath_And_Load_Sharing_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Only one route installs after `maximum-paths` | `show bgp ipv4 unicast <prefix>` | Candidate paths do not match required attributes |
| Second path appears in BGP but not RIB | `show ip route <prefix>` | Path is not eligible for multipath or next hop is unreachable |
| No `m` marker appears | `show bgp ipv4 unicast <prefix>` | Multipath did not install additional paths |
| eBGP same-AS multipath fails | `show bgp ipv4 unicast <prefix>` | Weight, local preference, AS path, origin, MED, or next-hop reachability differs |
| eBGP different-AS multipath fails | `show running-config | section router bgp` | Missing `bgp bestpath as-path multipath-relax` or platform does not support the behavior |
| Different-AS paths still do not install after relaxation | `show bgp ipv4 unicast <prefix>` | Other required attributes still differ |
| iBGP multipath fails | `show bgp ipv4 unicast <prefix>` and `show ip route <next-hop-ip>` | Missing `maximum-paths ibgp`, unequal IGP metric to next hop, or attributes differ |
| iBGP next hops are unreachable | `show ip route <next-hop-ip>` | IGP/static reachability to BGP next hops is missing |
| Route is best but not installed | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | RIB failure or better administrative distance route exists |
| CEF does not show multiple next hops | `show ip cef <prefix>` | RIB did not install multiple next hops, or platform/FIB limitation exists |
| Traffic does not visibly balance evenly | `show interfaces <egress-interface>` | CEF hashing is per-flow; one test flow may use only one path |
| Ping test uses only one path | `show ip cef exact-route <src-ip> <dst-ip>` if supported | Same flow hashes to same next hop |
| Load sharing breaks firewall state | Path review | Return traffic may use a different path across stateful firewalls or NAT devices |
| Multipath changes local forwarding but peer sees one path | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected behavior; BGP multipath is not Additional Paths |
| Peer needs multiple paths but receives one | Peer `show bgp ipv4 unicast <prefix>` | Configure Additional Paths, not local multipath |
| Provider refuses eBGP multipath | Design review | Multipath may increase RIB/FIB and memory load on provider edge |
| AS path relaxation creates risky load sharing | `show bgp ipv4 unicast <prefix>` | Paths from different external ASNs may not be operationally equivalent |
| Attributes differ because policy changed one path | `show running-config | section route-map` | Route map set different weight, local preference, MED, or AS path |
| Outbound policy expected to change multipath | `show running-config | section router bgp` | Multipath is local install behavior; outbound policy affects advertisements |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used instead of soft refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-maximum-paths-ebgp-same-as-final` | `BGP_Multipath_And_Load_Sharing.md` | Primary lab for normal eBGP multipath across multiple equivalent paths from the same neighboring AS |
| `bgp-maximum-paths-ebgp-different-as-final` | `BGP_Multipath_And_Load_Sharing.md` | Primary lab for different-AS eBGP multipath and AS path relaxation behavior |
| `bgp-maximum-paths-ibgp-final` | `BGP_Multipath_And_Load_Sharing.md` | Primary lab for iBGP multipath, next-hop reachability, and equal IGP cost requirements |
# Index
# Source_Basis
# BGP_Multipath_And_Load_Sharing_Mental_Model
# BGP_Multipath_And_Load_Sharing_Configuration_Checklist
# BGP_Multipath_And_Load_Sharing_eBGP_Same_AS_Skeleton
# BGP_Multipath_And_Load_Sharing_iBGP_Skeleton
# BGP_Multipath_And_Load_Sharing_eBGP_Different_AS_Skeleton
# BGP_Multipath_And_Load_Sharing_eBGP_Multiple_Links_To_Same_AS_Skeleton
# BGP_Multipath_And_Load_Sharing_iBGP_Loopback_Multipath_Skeleton
# BGP_Multipath_And_Load_Sharing_Verification_Skeleton
# BGP_Multipath_And_Load_Sharing_Verification_Commands
# BGP_Multipath_And_Load_Sharing_Rollback
# BGP_Multipath_And_Load_Sharing_Failure_Checks
# Attached_Labs

Blunt rule: BGP multipath is local load sharing. It does not advertise multiple paths to other routers. If your goal is to make a peer learn more than one path, that is Additional Paths, not maximum-paths.