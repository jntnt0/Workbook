



BGP_Troubleshooting_Methodology.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Main conceptual source for BGP attributes, policy, iBGP/eBGP behavior, route reflectors, confederations, communities, MP-BGP, and IPv6 BGP |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Main practical IOS XE source for BGP configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28104-28119 | Troubleshooting BGP chapter overview | Supports troubleshooting neighbor adjacencies, route exchange, route selection, and IPv6 BGP |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28302-28357 | BGP neighbor troubleshooting | Supports `show bgp ipv4 unicast summary`, `show bgp ipv4 unicast neighbors`, Idle/Active interpretation, interface state, L3 reachability, remote-as, update-source, TTL, authentication, peer groups, and timers |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28359-28836 | Neighbor failure causes | Supports interface down, Layer 3 failure, default-route reachability problem, wrong neighbor statement, wrong source IP, TTL expiration, authentication mismatch, peer-group issues, and timer issues |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28838-28857 | BGP route troubleshooting causes | Supports missing/bad `network mask`, next-hop unreachable, iBGP split-horizon, better route source, and route filtering |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28857-28919 | BGP table and routing table checks | Supports `show bgp ipv4 unicast`, status codes, origin codes, RPKI codes, and `show ip route bgp` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28921-29008 | `network mask` troubleshooting | Confirms BGP `network` requires an exact matching route in the local routing table |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 29010-29085 | Next-hop troubleshooting | Supports diagnosing BGP routes visible in BGP but absent from RIB because the next hop is unreachable |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 29355-29452 | Route filtering troubleshooting | Supports checking `show ip protocols`, prefix lists, distribute lists, route maps, filter lists, and neighbor filter details |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 29456-29534 | BGP path-selection troubleshooting | Supports best-path order: weight, local preference, locally originated, AIGP, AS path, origin, MED, eBGP over iBGP, and next-hop prerequisite |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22958-22967 | Adj-RIB-Out verification | Supports `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` |
# BGP_Troubleshooting_Methodology_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP troubleshooting order | Verify transport first, then neighbor state, AFI activation, route existence, policy, next hop, best path, RIB, FIB, and data plane |
| Transport before BGP | BGP cannot work until IP reachability and TCP/179 work |
| Neighbor state before route state | If the neighbor is Idle or Active, route troubleshooting is premature |
| Number in `State/PfxRcd` | BGP session is established and receiving that many prefixes |
| `Idle` | BGP is not successfully forming the session |
| `Active` | BGP is trying TCP but the TCP connection is failing or being reset |
| Established with zero prefixes | Session works, but route exchange, AFI activation, origination, policy, or next-hop behavior is broken |
| AFI activation | Neighbor must be activated under the correct address family before routes are exchanged |
| Route origination | A local route must be injected through exact `network`, redistribution, aggregate, or default-originate |
| Exact `network mask` rule | BGP `network <prefix> mask <mask>` requires that exact prefix and mask in the local RIB |
| BGP table | Shows BGP-learned and locally originated BGP paths |
| Routing table | Shows what actually installed for forwarding |
| FIB/CEF | Shows what the data plane actually forwards |
| Next-hop validity | BGP path cannot become usable if the next hop is unreachable |
| RIB failure | BGP selected or learned a path but the RIB did not install it |
| Better route source | Same prefix may be installed from connected, static, OSPF, EIGRP, or another source instead of BGP |
| Inbound policy | Controls what routes this router accepts from a neighbor |
| Outbound policy | Controls what routes this router advertises to a neighbor |
| Advertised-routes | Proves what this router is actually sending to a neighbor |
| Neighbor routes | Shows what this router accepted from a neighbor after inbound policy |
| Received-routes | Shows raw received routes only when inbound soft reconfiguration is enabled |
| Best-path decision | BGP chooses one best path by ordered attributes after next-hop reachability is satisfied |
| iBGP split-horizon | iBGP-learned routes are not advertised to another iBGP peer unless route reflectors or confederations are used |
| Route reflector issue | Reflected route visibility is separate from next-hop reachability |
| MP-BGP issue | Check the correct AFI, such as `show bgp ipv6 unicast`, not only IPv4 unicast |
| Soft clear | Re-evaluates BGP policy without tearing down the session |
| Hard clear | Resets the BGP session and should not be the first troubleshooting move |
| Blunt rule | Do not chase attributes, communities, or route maps until the neighbor is established, the AFI is active, the route exists, and the next hop is reachable |
# BGP_Troubleshooting_Methodology_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Define the symptom clearly | Notes | `neighbor down / route missing / route not installed / wrong best path / traffic failing` | Troubleshooting target is specific |
| 2 | Identify affected prefix | Notes | `<prefix>/<length>` | Prefix being troubleshot is known |
| 3 | Identify affected neighbor | Notes | `<neighbor-ip>` | Neighbor involved in the issue is known |
| 4 | Identify address family | Notes | `ipv4 unicast / ipv6 unicast / vpnv4 / vrf` | Correct BGP table is selected |
| 5 | Confirm local BGP process | Router | `show running-config | section router bgp` | Local AS, neighbors, AFIs, policy, and features are visible |
| 6 | Verify interface state | Router | `show ip interface brief` | BGP source and path interfaces are up/up |
| 7 | Verify IPv6 interface state if applicable | Router | `show ipv6 interface brief` | IPv6 interfaces are up/up |
| 8 | Verify source interface if update-source is used | Router | `show running-config interface <source-interface>` | Source interface exists and has expected IP |
| 9 | Verify route to neighbor | Router | `show ip route <neighbor-ip>` | Router has a route to the neighbor address |
| 10 | Reject default-route-only neighbor reachability for BGP peering | Router | `show ip route <neighbor-ip>` | Neighbor is reached by a specific route, not only default route |
| 11 | Verify sourced ping to neighbor | Router | `ping <neighbor-ip> source <local-source-ip>` | Ping succeeds from the same source BGP uses |
| 12 | Verify reverse reachability from peer | Peer Router | `ping <local-source-ip> source <peer-source-ip>` | Peer can reach local BGP source |
| 13 | Verify TCP/179 path | Router / Path | `show access-lists` or path firewall check | TCP/179 is not blocked |
| 14 | Check BGP neighbor summary | Router | `show bgp ipv4 unicast summary` | Neighbor state is visible |
| 15 | Interpret established state | Router | `show bgp ipv4 unicast summary` | Number in `State/PfxRcd` means session is established |
| 16 | Interpret Idle or Active | Router | `show bgp ipv4 unicast summary` | Idle or Active means neighbor formation problem |
| 17 | Check detailed neighbor output only after summary | Router | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Remote AS, source, capabilities, timers, and state detail are visible |
| 18 | Verify neighbor remote AS | Router | `show running-config | section router bgp` | `neighbor <ip> remote-as <asn>` matches peer local AS |
| 19 | Verify neighbor source matching | Router | `show running-config | section router bgp` | `update-source` matches peer neighbor statement expectation |
| 20 | Verify eBGP multihop if peer is not directly connected | Router | `show running-config | include ebgp-multihop` | TTL is high enough for non-direct eBGP |
| 21 | Verify authentication if configured | Router | `show running-config | include neighbor .* password` | Password is configured consistently on both peers |
| 22 | Verify timers if session fails after OpenSent/OpenConfirm | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> | include hold time|holdtime|keepalive` | Timer and minimum hold-time behavior is visible |
| 23 | Verify peer-group inheritance | Router | `show running-config | section router bgp` | Neighbor is in correct peer group and no specific neighbor command overrides intended behavior |
| 24 | Verify AFI activation | Router | `show running-config | section address-family` | Neighbor is activated under correct AFI |
| 25 | Verify IPv6 AFI if troubleshooting IPv6 BGP | Router | `show bgp ipv6 unicast summary` | Neighbor appears under IPv6 unicast |
| 26 | Verify BGP route exists in BGP table | Router | `show bgp ipv4 unicast <prefix>` | Prefix appears in BGP table |
| 27 | If route is missing from BGP table, check source/originator | Source Router | `show bgp ipv4 unicast <prefix>` | Source router has the prefix in BGP |
| 28 | Verify exact RIB route for local `network` statement | Source Router | `show ip route <prefix> <mask>` | Exact route exists in local RIB |
| 29 | Verify local BGP `network` statement | Source Router | `show running-config | section router bgp` | Correct `network <prefix> mask <mask>` exists |
| 30 | Verify redistribution if route enters through redistribution | Source Router | `show running-config | include redistribute` | Redistribution source and route map are visible |
| 31 | Verify aggregate if route enters through aggregation | Source Router | `show running-config | include aggregate-address` | Aggregate exists under correct AFI |
| 32 | Verify default route origination if default is missing | Source Router | `show running-config | include default-originate|network 0.0.0.0` | Default advertisement mechanism is visible |
| 33 | Verify route is advertised to the neighbor | Source Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Prefix is present in Adj-RIB-Out |
| 34 | If route is not advertised, inspect outbound policy | Source Router | `show bgp ipv4 unicast neighbors <neighbor-ip> | include prefix|filter|Route map` | Outbound prefix list, filter list, or route map is visible |
| 35 | Verify outbound route-map logic | Source Router | `show running-config | section route-map` | Route map permits or modifies expected prefix |
| 36 | Verify outbound prefix-list logic | Source Router | `show ip prefix-list <prefix-list-name>` | Prefix list permits expected prefix |
| 37 | Verify outbound AS path filter logic | Source Router | `show running-config | include ip as-path|filter-list` | AS path filter permits expected path |
| 38 | Refresh outbound policy after policy change | Source Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound advertisements are rebuilt |
| 39 | Verify route accepted from neighbor | Receiving Router | `show bgp ipv4 unicast neighbors <neighbor-ip> routes` | Prefix is accepted after inbound policy |
| 40 | If route is not accepted, inspect inbound policy | Receiving Router | `show bgp ipv4 unicast neighbors <neighbor-ip> | include prefix|filter|Route map` | Inbound policy attachments are visible |
| 41 | Verify process-wide filters | Receiving Router | `show ip protocols` | BGP distribute lists, filters, route maps, and maximum paths are visible |
| 42 | Verify inbound prefix-list logic | Receiving Router | `show ip prefix-list <prefix-list-name>` | Prefix list permits expected route |
| 43 | Verify inbound route-map logic | Receiving Router | `show route-map <route-map-name>` | Match counters increment and permit sequence is correct |
| 44 | Verify inbound AS path filter logic | Receiving Router | `show ip bgp regexp <regex>` | Regex matches intended routes only |
| 45 | Refresh inbound policy after policy change | Receiving Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are reprocessed |
| 46 | Verify BGP table status codes | Router | `show bgp ipv4 unicast <prefix>` | Prefix status markers show valid, best, RIB failure, damped, suppressed, multipath, backup, or additional-path state |
| 47 | Verify route validity marker | Router | `show bgp ipv4 unicast <prefix>` | `*` means valid |
| 48 | Verify best-path marker | Router | `show bgp ipv4 unicast <prefix>` | `>` means selected best path |
| 49 | Verify next-hop address | Router | `show bgp ipv4 unicast <prefix>` | BGP next hop is identified |
| 50 | Verify route to BGP next hop | Router | `show ip route <bgp-next-hop-ip>` | BGP next hop is reachable |
| 51 | Verify sourced ping to BGP next hop | Router | `ping <bgp-next-hop-ip> source <local-source-ip>` | Next hop responds or path is proven reachable |
| 52 | Fix iBGP next-hop problem where required | Edge Router | `neighbor <ibgp-neighbor-ip> next-hop-self` under AFI | iBGP peer receives reachable next hop |
| 53 | Verify route installs in RIB | Router | `show ip route <prefix>` | Prefix appears as BGP route if selected and valid |
| 54 | Verify BGP-only RIB entries | Router | `show ip route bgp` | BGP-installed routes are visible |
| 55 | If route is best but not installed, check RIB failure | Router | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Better AD route or RIB conflict is identified |
| 56 | Verify CEF/FIB entry | Router | `show ip cef <prefix>` | FIB points to expected next hop/interface |
| 57 | Verify data-plane forwarding | Router | `traceroute <destination-ip> source <source-ip>` | Forwarding path matches expected route |
| 58 | Verify return path | Remote Router | `show ip route <source-ip>` | Remote side can route back to source |
| 59 | If wrong path wins, inspect best-path attributes | Router | `show bgp ipv4 unicast <prefix>` | Weight, local preference, locally originated, AIGP, AS path, origin, MED, and eBGP/iBGP state are visible |
| 60 | Verify best-path reason if supported | Router | `show bgp ipv4 unicast <prefix> best-path-reason` | Router explains why selected path won |
| 61 | Verify route-map attribute changes | Router | `show route-map <route-map-name>` | Set clauses and hit counters explain attribute modification |
| 62 | Verify local preference propagation | iBGP Router | `show bgp ipv4 unicast <prefix>` | Local preference is consistent where intended |
| 63 | Verify weight is local-only | Other Router | `show bgp ipv4 unicast <prefix>` | Other routers do not inherit local weight |
| 64 | Verify AS path prepending externally | External Peer | `show bgp ipv4 unicast <prefix>` | AS path shows expected prepend |
| 65 | Verify community policy | Router | `show bgp ipv4 unicast <prefix>` | Communities are present or absent as expected |
| 66 | Verify send-community if peer should receive tags | Router | `show running-config | include send-community` | Communities are sent to intended peers |
| 67 | Verify route reflector behavior if iBGP route is missing | RR | `show running-config | include route-reflector-client` | RR clients are configured on route reflector only |
| 68 | Verify confederation behavior if used | Router | `show running-config | include bgp confederation` | Confederation identifier and peers are correct |
| 69 | Verify IPv6 BGP table if IPv6 route is missing | Router | `show bgp ipv6 unicast <ipv6-prefix>` | IPv6 NLRI appears under IPv6 AFI |
| 70 | Verify IPv6 route install | Router | `show ipv6 route <ipv6-prefix>` | IPv6 BGP route installs |
| 71 | Verify IPv6 next hop | Router | `show bgp ipv6 unicast <ipv6-prefix>` | Next hop is real IPv6, not invalid IPv4-mapped next hop |
| 72 | Verify RPKI state if origin validation is in use | Router | `show bgp ipv4 unicast <prefix>` | Route is Valid, Invalid, or Not Found |
| 73 | Verify dampening state if route flaps | Router | `show ip bgp dampened-paths` | Damped routes are identified |
| 74 | Verify maximum-prefix events | Router | `show logging | include MAXPFX|BGP` | Prefix-limit events are identified |
| 75 | Verify BGP logs | Router | `show logging | include BGP|ADJCHANGE|NOTIFICATION` | Neighbor resets and notifications are visible |
| 76 | Make the smallest corrective change | Router | `<single targeted fix>` | One variable changes at a time |
| 77 | Use soft clear for policy re-evaluation | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` or `soft out` | Policy updates without hard reset |
| 78 | Avoid hard reset unless required | Router | `clear bgp ipv4 unicast <neighbor-ip>` | Used only when capability/session renegotiation is required |
| 79 | Verify control plane after fix | Router | `show bgp ipv4 unicast summary` and `show bgp ipv4 unicast <prefix>` | Neighbor and route state are correct |
| 80 | Verify data plane after fix | Router | `show ip route <prefix>`, `show ip cef <prefix>`, `traceroute <destination-ip>` | Route installs and forwards correctly |
# BGP_Troubleshooting_Methodology_Neighbor_Startup_Skeleton
show ip interface brief
show running-config | section router bgp
show ip route <neighbor-ip>
ping <neighbor-ip> source <local-source-ip>
show bgp ipv4 unicast summary
show bgp ipv4 unicast neighbors <neighbor-ip>
show logging | include BGP|ADJCHANGE|NOTIFICATION
# BGP_Troubleshooting_Methodology_Neighbor_Fix_Skeleton
configure terminal
router bgp <local-asn>
 neighbor <neighbor-ip> remote-as <remote-asn>
 neighbor <neighbor-ip> update-source <source-interface>
 neighbor <neighbor-ip> ebgp-multihop <ttl>
 neighbor <neighbor-ip> description <description>
 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
 exit-address-family
end
copy running-config startup-config
# BGP_Troubleshooting_Methodology_Route_Missing_From_BGP_Table_Skeleton
show bgp ipv4 unicast <prefix>
show running-config | section router bgp
show ip route <prefix> <mask>
show running-config | include network|redistribute|aggregate-address|default-originate
show route-map
show ip prefix-list
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
# BGP_Troubleshooting_Methodology_Network_Statement_Fix_Skeleton
configure terminal
ip route <prefix> <mask> <next-hop-or-null0>
router bgp <local-asn>
 address-family ipv4 unicast
  network <prefix> mask <mask>
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Troubleshooting_Methodology_Route_Visible_Not_Installed_Skeleton
show bgp ipv4 unicast <prefix>
show ip route <bgp-next-hop-ip>
ping <bgp-next-hop-ip> source <local-source-ip>
show ip route <prefix>
show ip route bgp
show ip cef <prefix>
show bgp ipv4 unicast <prefix> best-path-reason
# BGP_Troubleshooting_Methodology_iBGP_Next_Hop_Self_Fix_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <ibgp-neighbor-ip> next-hop-self
 exit-address-family
end
clear bgp ipv4 unicast <ibgp-neighbor-ip> soft out
copy running-config startup-config
# BGP_Troubleshooting_Methodology_Policy_Filter_Skeleton
show ip protocols
show bgp ipv4 unicast neighbors <neighbor-ip> | include prefix|filter|Route map
show running-config | section router bgp
show running-config | section route-map
show ip prefix-list
show running-config | include ip as-path|filter-list|distribute-list
show route-map <route-map-name>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show bgp ipv4 unicast neighbors <neighbor-ip> routes
# BGP_Troubleshooting_Methodology_Path_Selection_Skeleton
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast <prefix> bestpath
show bgp ipv4 unicast <prefix> best-path-reason
show ip route <bgp-next-hop-ip>
show ip route <prefix>
show ip cef <prefix>
show running-config | section route-map
show route-map <route-map-name>
# BGP_Troubleshooting_Methodology_Soft_Refresh_Skeleton
clear bgp ipv4 unicast <neighbor-ip> soft in
clear bgp ipv4 unicast <neighbor-ip> soft out
# BGP_Troubleshooting_Methodology_IPv6_BGP_Skeleton
show ipv6 interface brief
show bgp ipv6 unicast summary
show bgp ipv6 unicast neighbors <neighbor-ip>
show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>
show ipv6 route <ipv6-prefix>/<prefix-length>
show ipv6 cef <ipv6-prefix>/<prefix-length>
show bgp ipv6 unicast neighbors <neighbor-ip> advertised-routes
# BGP_Troubleshooting_Methodology_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify interface state | `show ip interface brief` | BGP source and path interfaces are up/up |
| Verify IPv6 interface state | `show ipv6 interface brief` | IPv6 interfaces are up/up |
| Verify BGP config | `show running-config | section router bgp` | Local AS, neighbors, AFIs, and policy are visible |
| Verify neighbor route | `show ip route <neighbor-ip>` | Specific route to neighbor exists |
| Verify sourced neighbor reachability | `ping <neighbor-ip> source <local-source-ip>` | BGP neighbor is reachable from configured source |
| Verify BGP summary | `show bgp ipv4 unicast summary` | Neighbor is established if `State/PfxRcd` shows a number |
| Verify neighbor detail | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Remote AS, state, timers, capabilities, source, and policy detail are visible |
| Verify BGP logs | `show logging | include BGP|ADJCHANGE|NOTIFICATION` | Neighbor resets and reasons are visible |
| Verify IPv4 route in BGP | `show bgp ipv4 unicast <prefix>` | Prefix appears in BGP table |
| Verify IPv6 route in BGP | `show bgp ipv6 unicast <ipv6-prefix>/<prefix-length>` | IPv6 prefix appears under IPv6 AFI |
| Verify locally originated route source | `show bgp ipv4 unicast <prefix>` | Locally originated route shows next hop `0.0.0.0` |
| Verify exact route for `network` | `show ip route <prefix> <mask>` | Exact route exists in local RIB |
| Verify IPv6 exact route for `network` | `show ipv6 route <ipv6-prefix>/<prefix-length>` | Exact IPv6 route exists |
| Verify advertised routes | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Prefix is being sent to neighbor |
| Verify accepted routes from neighbor | `show bgp ipv4 unicast neighbors <neighbor-ip> routes` | Prefix is accepted after inbound policy |
| Verify raw received routes if enabled | `show bgp ipv4 unicast neighbors <neighbor-ip> received-routes` | Raw received routes are visible only if soft reconfiguration inbound is enabled |
| Verify process filters | `show ip protocols` | BGP filters, distribute lists, route maps, and maximum paths are visible |
| Verify neighbor filters | `show bgp ipv4 unicast neighbors <neighbor-ip> | include prefix|filter|Route map` | Neighbor-specific prefix list, filter list, and route map attachments are visible |
| Verify route-map logic | `show running-config | section route-map` | Match and set clauses are correct |
| Verify route-map hits | `show route-map <route-map-name>` | Counters increment on expected sequence |
| Verify prefix-list logic | `show ip prefix-list <prefix-list-name>` | Expected prefix is permitted |
| Verify AS path filter | `show ip bgp regexp <regex>` | Regex matches expected AS paths |
| Verify next-hop route | `show ip route <bgp-next-hop-ip>` | BGP next hop is reachable |
| Verify next-hop ping | `ping <bgp-next-hop-ip> source <local-source-ip>` | Next hop is reachable from expected source |
| Verify RIB install | `show ip route <prefix>` | Prefix installs if selected, valid, and not beaten by better source |
| Verify BGP-only routes | `show ip route bgp` | BGP-installed routes are visible |
| Verify FIB install | `show ip cef <prefix>` | FIB points toward expected next hop/interface |
| Verify best-path attributes | `show bgp ipv4 unicast <prefix>` | Weight, local preference, AS path, origin, MED, and next hop are visible |
| Verify best-path reason | `show bgp ipv4 unicast <prefix> best-path-reason` | Router explains why path won if supported |
| Verify IPv6 RIB install | `show ipv6 route <ipv6-prefix>/<prefix-length>` | IPv6 route installs |
| Verify IPv6 FIB install | `show ipv6 cef <ipv6-prefix>/<prefix-length>` | IPv6 forwarding entry is present |
| Verify data-plane path | `traceroute <destination-ip> source <source-ip>` | Traffic follows expected path |
| Verify return path | Remote side `show ip route <source-ip>` | Remote side can return traffic |
# BGP_Troubleshooting_Methodology_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify exact change made during troubleshooting | Notes | `<changed-command-list>` | Change set is known before rollback |
| 2 | Capture current BGP state | Router | `show bgp ipv4 unicast summary` | Neighbor state is documented |
| 3 | Capture current route state | Router | `show bgp ipv4 unicast <prefix>` | Prefix state is documented |
| 4 | Capture current RIB/FIB state | Router | `show ip route <prefix>` and `show ip cef <prefix>` | Install and forwarding state are documented |
| 5 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 6 | Remove temporary neighbor change | Router | `router bgp <local-asn>` then `no neighbor <neighbor-ip> <temporary-command>` | Temporary neighbor command is removed |
| 7 | Remove temporary route-map attachment | Router | `address-family ipv4 unicast` then `no neighbor <neighbor-ip> route-map <rm-name> in|out` | Temporary policy attachment is removed |
| 8 | Remove temporary prefix-list if lab-only | Router | `no ip prefix-list <prefix-list-name>` | Prefix list is removed |
| 9 | Remove temporary route-map sequence if lab-only | Router | `no route-map <route-map-name> permit <seq>` | Route-map sequence is removed |
| 10 | Remove temporary static route used for origination | Router | `no ip route <prefix> <mask> <next-hop-or-null0>` | Temporary RIB route is removed |
| 11 | Remove temporary `network` statement | Router | `router bgp <local-asn>` then `address-family ipv4 unicast` then `no network <prefix> mask <mask>` | Temporary BGP origination is removed |
| 12 | Remove temporary next-hop-self | Router | `no neighbor <ibgp-neighbor-ip> next-hop-self` | Next-hop rewrite is removed if lab-only |
| 13 | Remove temporary eBGP multihop | Router | `no neighbor <neighbor-ip> ebgp-multihop <ttl>` | TTL override is removed if lab-only |
| 14 | Soft refresh inbound if inbound policy changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are reprocessed |
| 15 | Soft refresh outbound if outbound policy changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound advertisements are rebuilt |
| 16 | Verify rollback neighbor state | Router | `show bgp ipv4 unicast summary` | Neighbor state matches intended baseline |
| 17 | Verify rollback route state | Router | `show bgp ipv4 unicast <prefix>` | Prefix state matches intended baseline |
| 18 | Verify rollback forwarding state | Router | `show ip route <prefix>` and `show ip cef <prefix>` | RIB and FIB match intended baseline |
| 19 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Troubleshooting_Methodology_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Neighbor stuck Idle | `show bgp ipv4 unicast summary` | Wrong neighbor IP, wrong remote AS, missing route to peer, source mismatch, authentication issue, or TCP/179 blocked |
| Neighbor stuck Active | `show bgp ipv4 unicast summary` | TCP session cannot complete |
| Neighbor reachable only through default route | `show ip route <neighbor-ip>` | BGP peering should use a specific route, not only default route |
| Ping works but BGP fails | `ping <neighbor-ip> source <local-source-ip>` | Ping used wrong source or BGP source is not reachable |
| eBGP loopback peering fails | `show running-config | section router bgp` | Missing `update-source`, missing `ebgp-multihop`, or missing route to loopback |
| Remote AS mismatch | `show logging | include BGP|NOTIFICATION` | `neighbor <ip> remote-as <asn>` is wrong on one side |
| Authentication mismatch | `show logging | include BGP|NOTIFICATION` | BGP password differs or exists on only one side |
| Hold-time notification | `show logging | include hold|BGP|NOTIFICATION` | Minimum hold time or timer negotiation problem |
| Peer-group neighbor fails | `show running-config | section router bgp` | Neighbor not assigned to peer group or specific neighbor command overrides group behavior |
| Established but zero prefixes | `show bgp ipv4 unicast summary` | AFI not activated, no routes originated, policy blocks routes, or peer has nothing to advertise |
| IPv6 route missing while IPv4 session is up | `show bgp ipv6 unicast summary` | IPv6 AFI not activated |
| Prefix missing from source BGP table | Source `show bgp ipv4 unicast <prefix>` | Route was never originated into BGP |
| `network` statement does nothing | `show ip route <prefix> <mask>` | Exact matching route is not in local RIB |
| Route is advertised but not received | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` and peer `routes` | Peer inbound policy blocks it or peer rejects it |
| Route is received but not advertised onward | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Outbound policy, iBGP split-horizon, route reflector behavior, or best-path rule blocks advertisement |
| iBGP route not passed to another iBGP peer | `show bgp ipv4 unicast neighbors <ibgp-peer> advertised-routes` | Normal iBGP split-horizon, need full mesh, route reflector, or confederation |
| Route appears in BGP but not routing table | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Next hop unreachable, RIB failure, better route source, or route is not best |
| No `>` best marker | `show bgp ipv4 unicast <prefix>` | Path did not win best-path selection or next hop is invalid |
| No `*` valid marker | `show bgp ipv4 unicast <prefix>` | Next hop is unreachable or path is otherwise invalid |
| RIB failure marker appears | `show bgp ipv4 unicast <prefix>` | Another route source with better administrative distance owns the RIB entry |
| Next-hop unreachable | `show ip route <bgp-next-hop-ip>` | Missing IGP/static route or iBGP next-hop-self issue |
| BGP route installs but traffic fails | `show ip cef <prefix>` and return-path check | FIB, return route, ACL, NAT, firewall, or host path is broken |
| Wrong path selected | `show bgp ipv4 unicast <prefix> best-path-reason` | Higher-priority attribute won before the attribute you were looking at |
| Weight fix affects only one router | Other router `show bgp ipv4 unicast <prefix>` | Expected behavior; weight is local-only |
| Local preference does not affect external peer | External peer `show bgp ipv4 unicast <prefix>` | Expected behavior; local preference stays inside AS |
| MED has no effect | `show bgp ipv4 unicast <prefix>` | Earlier attributes already decided path or paths are from different neighboring ASNs |
| Community policy has no effect | `show bgp ipv4 unicast <prefix>` | Missing community, missing `send-community`, wrong route-map direction, or no soft refresh |
| Route-map drops too much | `show route-map <route-map-name>` | Missing permit fallback sequence |
| Prefix-list misses route | `show ip prefix-list <prefix-list-name>` | Prefix length or ge/le logic is wrong |
| AS path regex matches wrong routes | `show ip bgp regexp <regex>` | Regex is too broad or direction assumption is wrong |
| Advertised-routes does not update | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound policy was not refreshed |
| Inbound policy does not update | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound policy was not refreshed |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used instead of soft refresh |
| Troubleshooting starts in the wrong layer | `show bgp ipv4 unicast summary` | Neighbor is not established, so route and path selection checks are premature |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-troubleshooting-startup-final` | `BGP_Troubleshooting_Methodology.md` | Primary startup methodology lab for neighbor state, BGP transport, source IP, remote AS, AFI activation, and initial route exchange |
| `bgp-troubleshooting-final` | `BGP_Troubleshooting_Methodology.md` | Primary end-to-end BGP troubleshooting lab for route visibility, policy, next-hop, path selection, RIB install, and forwarding verification |
| `how-to-read-bgp-table-final` | `BGP_Troubleshooting_Methodology.md` | Cross-reference lab for interpreting BGP table codes, next hop, best path, origin, AS path, and RIB status |
# Index
# Source_Basis
# BGP_Troubleshooting_Methodology_Mental_Model
# BGP_Troubleshooting_Methodology_Configuration_Checklist
# BGP_Troubleshooting_Methodology_Neighbor_Startup_Skeleton
# BGP_Troubleshooting_Methodology_Neighbor_Fix_Skeleton
# BGP_Troubleshooting_Methodology_Route_Missing_From_BGP_Table_Skeleton
# BGP_Troubleshooting_Methodology_Network_Statement_Fix_Skeleton
# BGP_Troubleshooting_Methodology_Route_Visible_Not_Installed_Skeleton
# BGP_Troubleshooting_Methodology_iBGP_Next_Hop_Self_Fix_Skeleton
# BGP_Troubleshooting_Methodology_Policy_Filter_Skeleton
# BGP_Troubleshooting_Methodology_Path_Selection_Skeleton
# BGP_Troubleshooting_Methodology_Soft_Refresh_Skeleton
# BGP_Troubleshooting_Methodology_IPv6_BGP_Skeleton
# BGP_Troubleshooting_Methodology_Verification_Commands
# BGP_Troubleshooting_Methodology_Rollback
# BGP_Troubleshooting_Methodology_Failure_Checks
# Attached_Labs

Blunt rule: BGP troubleshooting has an order. Neighbor first. AFI second. Route existence third. Policy fourth. Next hop fifth. Best path sixth. RIB/FIB seventh. Data plane last. Skip that order and you will waste time staring at attributes while the session, source address, or next hop is broken.