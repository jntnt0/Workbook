

BGP_Aggregation_And_AS_SET.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP attributes, route aggregation, route maps, AS path behavior, and policy control |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP aggregation configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24548-24548 | BGP aggregation syntax | Supports `aggregate-address <network> <subnet-mask> [summary-only] [as-set]` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24616-24630 | Basic aggregate-address examples | Supports creating IPv4 BGP aggregate prefixes under BGP or address-family configuration |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24694-24725 | `summary-only` behavior | Confirms aggregate routes are advertised with component routes by default, and `summary-only` suppresses component prefixes |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24814-24878 | Atomic aggregate | Confirms route aggregation can lose AS path, MED, and community information, and atomic aggregate signals loss of path information |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24898-24916 | `as-set` behavior | Confirms `as-set` preserves AS path information from component routes inside the AS_SET portion of AS_PATH |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25010-25081 | IPv6 aggregate-address | Supports IPv6 aggregation syntax under IPv6 address family |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28865-28868 | Aggregation troubleshooting | Confirms `aggregate-address` creates a summary route and locally originated aggregate routes show next hop `0.0.0.0` |
| `iosxe_combined_pdfs_.md` lines 33961-33977 | IOS XE aggregate-address syntax | Supports `aggregate-address <address-mask>` and optional `aggregate-address <address-mask> advertise-map <map-name>` |
# BGP_Aggregation_And_AS_SET_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP aggregation | Creates a shorter summary prefix from more-specific component prefixes |
| Component route | More-specific BGP route that falls inside the aggregate range |
| Aggregate route | The summarized prefix created by `aggregate-address` |
| Aggregate trigger | Aggregate is created when at least one viable component route exists in the BGP table |
| `aggregate-address` | BGP command used to create a summary prefix |
| Default aggregate behavior | Advertises the aggregate and still advertises component routes |
| `summary-only` | Advertises the aggregate and suppresses the component routes |
| Atomic aggregate | Attribute that warns path information was lost during aggregation |
| AS_SET | AS path segment that preserves ASNs from component routes inside braces/brackets |
| `as-set` | Preserves AS path information from component routes when generating the aggregate |
| AS_SET path length | AS_SET counts as one AS hop even if it contains multiple ASNs |
| Loop prevention | Preserved ASNs in AS_SET help downstream routers avoid accepting routes that contain their own AS |
| Aggregate next hop | Locally generated aggregate usually shows next hop `0.0.0.0` in the BGP table |
| Discard behavior | Router may install a discard route for the aggregate to prevent routing loops |
| Summary blackhole risk | Aggregate can attract traffic for sub-prefixes that do not actually exist |
| Prefix reachability requirement | Aggregation does not create reachability for missing subnets; it only advertises a summarized control-plane route |
| IPv4 aggregation | Uses `aggregate-address <network> <subnet-mask>` |
| IPv6 aggregation | Uses `aggregate-address <prefix>/<prefix-length>` under IPv6 address family |
| Blunt rule | Use `summary-only` to reduce advertisements. Use `as-set` when the AS path of component routes matters. Without `as-set`, you may hide important AS path information |
# BGP_Aggregation_And_AS_SET_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Relevant neighbors show a number under `State/PfxRcd` |
| 2 | Identify aggregate prefix | Notes | `<aggregate-prefix> <aggregate-mask>` | Summary network and mask are known |
| 3 | Identify component prefixes | Notes | `<component-prefix-list>` | More-specific prefixes inside the aggregate are known |
| 4 | Confirm component routes exist in BGP | Router | `show bgp ipv4 unicast <component-prefix>` | At least one component route exists in the BGP table |
| 5 | Confirm component routes are valid | Router | `show bgp ipv4 unicast <component-prefix>` | Component route is valid and usable for aggregation |
| 6 | Confirm component routes are in RIB if locally originated by `network` | Router | `show ip route <component-prefix> <component-mask>` | Exact component route exists if originated with `network` |
| 7 | Confirm current advertised routes | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Baseline route advertisement is known |
| 8 | Decide whether components should still be advertised | Notes | `aggregate-only` or `aggregate-plus-components` | `summary-only` decision is clear |
| 9 | Use plain aggregate when components should remain visible | Notes | `aggregate-address <prefix> <mask>` | Aggregate and components will be advertised |
| 10 | Use `summary-only` when components should be suppressed | Notes | `aggregate-address <prefix> <mask> summary-only` | Only aggregate is advertised to peers |
| 11 | Decide whether AS path preservation is required | Notes | `as-set` or `no as-set` | AS_SET decision is clear |
| 12 | Use `as-set` when component ASNs must be preserved | Notes | `aggregate-address <prefix> <mask> as-set` | Aggregate carries AS_SET path information |
| 13 | Avoid `as-set` only when loss of AS path detail is acceptable | Notes | `atomic aggregate expected` | Aggregate may show atomic aggregate behavior |
| 14 | Review current BGP config | Router | `show running-config | section router bgp` | Existing aggregate, neighbor, AFI, and policy config are visible |
| 15 | Review current BGP table | Router | `show bgp ipv4 unicast` | Existing component and aggregate routes are visible |
| 16 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 17 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 18 | Enter IPv4 unicast address family if using explicit AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 19 | Configure basic IPv4 aggregate | Router | `aggregate-address <aggregate-prefix> <aggregate-mask>` | Aggregate route is generated when component route exists |
| 20 | Configure IPv4 aggregate with component suppression | Router | `aggregate-address <aggregate-prefix> <aggregate-mask> summary-only` | Aggregate is advertised and components are suppressed |
| 21 | Configure IPv4 aggregate with AS_SET | Router | `aggregate-address <aggregate-prefix> <aggregate-mask> as-set` | Aggregate preserves AS path information from component routes |
| 22 | Configure IPv4 aggregate with AS_SET and component suppression | Router | `aggregate-address <aggregate-prefix> <aggregate-mask> as-set summary-only` | Aggregate is advertised, components are suppressed, AS_SET is preserved |
| 23 | Configure IPv6 aggregate if needed | Router | `address-family ipv6 unicast` then `aggregate-address <ipv6-prefix>/<prefix-length> summary-only` | IPv6 aggregate is created under IPv6 AFI |
| 24 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 25 | Save configuration | Router | `copy running-config startup-config` | Aggregation config is saved |
| 26 | Soft clear outbound after changing aggregation behavior | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Neighbor receives updated aggregate/component advertisement state |
| 27 | Verify aggregate exists locally | Router | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate appears in local BGP table |
| 28 | Verify aggregate next hop | Router | `show bgp ipv4 unicast <aggregate-prefix>` | Locally originated aggregate shows next hop `0.0.0.0` |
| 29 | Verify aggregate origin code | Router | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate usually shows origin code `i` |
| 30 | Verify component routes locally | Router | `show bgp ipv4 unicast <component-prefix>` | Component prefixes still exist locally if not removed from BGP |
| 31 | Verify advertised aggregate | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Aggregate is advertised to peer |
| 32 | Verify `summary-only` suppression | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Component routes inside aggregate are not advertised |
| 33 | Verify aggregate plus components when no `summary-only` is used | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Aggregate and component routes are advertised |
| 34 | Verify AS_SET behavior | Router | `show bgp ipv4 unicast <aggregate-prefix>` | AS_PATH includes AS_SET information from component routes |
| 35 | Verify atomic aggregate behavior | Router | `show bgp ipv4 unicast <aggregate-prefix>` | Atomic aggregate appears if path information was lost |
| 36 | Verify peer receives aggregate | Peer Router | `show bgp ipv4 unicast <aggregate-prefix>` | Peer sees the aggregate prefix |
| 37 | Verify peer component visibility | Peer Router | `show bgp ipv4 unicast <component-prefix>` | Components are present or absent according to `summary-only` design |
| 38 | Verify peer AS_PATH | Peer Router | `show bgp ipv4 unicast <aggregate-prefix>` | Peer sees expected AS_PATH or AS_SET behavior |
| 39 | Verify route install on peer | Peer Router | `show ip route <aggregate-prefix>` | Aggregate installs if selected and next hop is reachable |
| 40 | Verify forwarding path | Peer Router | `show ip cef <aggregate-prefix>` | FIB points toward expected next hop |
| 41 | Test traffic inside aggregate range | Test Host / Router | `ping <reachable-host-inside-aggregate>` | Reachable component subnets work |
| 42 | Test traffic to unused subnet inside aggregate range | Test Host / Router | `ping <unused-host-inside-aggregate>` | Traffic may be discarded due to summary/discard behavior |
| 43 | Verify no unwanted blackhole | Router / Peer | `traceroute <destination-inside-aggregate>` | Traffic path matches intended design |
# BGP_Aggregation_And_AS_SET_Basic_Aggregate_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  aggregate-address <aggregate-prefix> <aggregate-mask>
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Aggregation_And_AS_SET_Summary_Only_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  aggregate-address <aggregate-prefix> <aggregate-mask> summary-only
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Aggregation_And_AS_SET_AS_SET_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  aggregate-address <aggregate-prefix> <aggregate-mask> as-set
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Aggregation_And_AS_SET_AS_SET_Summary_Only_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  aggregate-address <aggregate-prefix> <aggregate-mask> as-set summary-only
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Aggregation_And_AS_SET_With_Component_Origination_Skeleton
configure terminal
ip route <component-prefix-1> <component-mask-1> <next-hop-or-null0>
ip route <component-prefix-2> <component-mask-2> <next-hop-or-null0>
router bgp <local-asn>
 address-family ipv4 unicast
  network <component-prefix-1> mask <component-mask-1>
  network <component-prefix-2> mask <component-mask-2>
  aggregate-address <aggregate-prefix> <aggregate-mask> as-set summary-only
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Aggregation_And_AS_SET_IPv6_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv6 unicast
  aggregate-address <ipv6-aggregate-prefix>/<prefix-length> summary-only
 exit-address-family
end
clear bgp ipv6 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Aggregation_And_AS_SET_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <aggregate-prefix>
show bgp ipv4 unicast <component-prefix>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show ip route <aggregate-prefix>
show ip cef <aggregate-prefix>
show running-config | section router bgp
show logging | include BGP|ADJCHANGE
# BGP_Aggregation_And_AS_SET_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP neighbor state | `show bgp ipv4 unicast summary` | Neighbor is established |
| Verify component prefix in BGP | `show bgp ipv4 unicast <component-prefix>` | Component route exists and is valid |
| Verify aggregate in BGP | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate route appears |
| Verify local aggregate next hop | `show bgp ipv4 unicast <aggregate-prefix>` | Locally generated aggregate shows next hop `0.0.0.0` |
| Verify aggregate origin | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate origin code is usually `i` |
| Verify AS_SET | `show bgp ipv4 unicast <aggregate-prefix>` | AS_PATH includes AS_SET information when `as-set` is used |
| Verify atomic aggregate | `show bgp ipv4 unicast <aggregate-prefix>` | Atomic aggregate appears when path information was lost |
| Verify advertised routes | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Aggregate is advertised |
| Verify component suppression | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Components are absent when `summary-only` is used |
| Verify aggregate plus components | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Components remain visible when `summary-only` is not used |
| Verify peer received aggregate | `show bgp ipv4 unicast <aggregate-prefix>` on peer | Peer sees the aggregate |
| Verify peer component visibility | `show bgp ipv4 unicast <component-prefix>` on peer | Peer sees or does not see component routes based on `summary-only` |
| Verify peer AS_PATH | `show bgp ipv4 unicast <aggregate-prefix>` on peer | Peer sees expected AS_PATH or AS_SET |
| Verify route install | `show ip route <aggregate-prefix>` | Aggregate installs if selected and next hop is reachable |
| Verify forwarding | `show ip cef <aggregate-prefix>` | FIB points toward expected next hop/interface |
| Verify BGP config | `show running-config | section router bgp` | Aggregate command appears under correct BGP AFI |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected BGP reset occurred |
# BGP_Aggregation_And_AS_SET_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current aggregate config | Router | `show running-config | section router bgp` | Aggregate command and options are visible |
| 2 | Identify advertised routes | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Current aggregate and component advertisement state is known |
| 3 | Identify aggregate BGP entry | Router | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate path details are visible |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 7 | Remove basic aggregate | Router | `no aggregate-address <aggregate-prefix> <aggregate-mask>` | Basic aggregate is removed |
| 8 | Remove summary-only aggregate | Router | `no aggregate-address <aggregate-prefix> <aggregate-mask> summary-only` | Summary-only aggregate is removed if exact syntax is required |
| 9 | Remove AS_SET aggregate | Router | `no aggregate-address <aggregate-prefix> <aggregate-mask> as-set` | AS_SET aggregate is removed if exact syntax is required |
| 10 | Remove AS_SET summary-only aggregate | Router | `no aggregate-address <aggregate-prefix> <aggregate-mask> as-set summary-only` | AS_SET summary-only aggregate is removed |
| 11 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 12 | Remove lab-only component network statement | Router | `address-family ipv4 unicast` then `no network <component-prefix> mask <component-mask>` | Component is no longer locally originated if lab-only |
| 13 | Remove lab-only static component route | Router | `no ip route <component-prefix> <component-mask> <next-hop-or-null0>` | Static component route is removed if lab-only |
| 14 | Soft clear outbound | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Peer receives updated advertisement state |
| 15 | Verify aggregate removed | Router | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate is absent |
| 16 | Verify advertised-routes rollback | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Aggregate no longer appears, or components reappear if still originated |
| 17 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Aggregation_And_AS_SET_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Aggregate route does not appear | `show bgp ipv4 unicast <component-prefix>` | No viable component route exists in BGP |
| Component route exists in RIB but not BGP | `show running-config | section router bgp` | Missing `network`, redistribution, or inbound BGP route |
| Aggregate appears but components also advertise | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Missing `summary-only` |
| Components disappear unexpectedly | `show running-config | section router bgp` | `summary-only` is configured |
| AS path detail is missing on aggregate | `show bgp ipv4 unicast <aggregate-prefix>` | Aggregate was configured without `as-set` |
| Atomic aggregate appears | `show bgp ipv4 unicast <aggregate-prefix>` | Path attribute information was lost during aggregation |
| Downstream AS rejects aggregate | Downstream `show bgp ipv4 unicast <aggregate-prefix>` | AS_SET contains downstream AS, triggering AS loop prevention |
| Aggregate causes blackhole | `traceroute <destination-inside-aggregate>` | Aggregate covers addresses that do not have real component reachability |
| Peer receives aggregate but cannot forward | Peer `show ip route <aggregate-prefix>` and `show ip cef <aggregate-prefix>` | Next hop unreachable or route not selected |
| Peer receives components despite summary design | Local `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Summary-only applied on wrong router, wrong AFI, or not refreshed outbound |
| Outbound change does not show on peer | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound route refresh was not run |
| Aggregate in wrong address family | `show running-config | section router bgp` | Aggregate configured outside the intended AFI |
| IPv6 aggregate does not appear | `show bgp ipv6 unicast <ipv6-aggregate-prefix>` | IPv6 component route missing or aggregate configured under wrong AFI |
| Route-map/policy hides aggregate | `show running-config | section route-map` | Outbound policy blocks the aggregate |
| Route-map/policy hides components | `show running-config | section route-map` | Outbound policy blocks components independently of `summary-only` |
| Aggregate selected but not in RIB | `show bgp ipv4 unicast <aggregate-prefix>` and `show ip route <aggregate-prefix>` | RIB failure or another route source wins |
| Component route flap still visible to peer | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Components are still advertised because `summary-only` is absent |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-aggregate-as-set-final` | `BGP_Aggregation_And_AS_SET.md` | Primary lab for BGP aggregate creation and AS_SET behavior |
| `bgp-aggregation-as-set-final` | `BGP_Aggregation_And_AS_SET.md` | Duplicate or variant aggregation lab; attach here instead of making a separate mechanism note |
# Index
# Source_Basis
# BGP_Aggregation_And_AS_SET_Mental_Model
# BGP_Aggregation_And_AS_SET_Configuration_Checklist
# BGP_Aggregation_And_AS_SET_Basic_Aggregate_Skeleton
# BGP_Aggregation_And_AS_SET_Summary_Only_Skeleton
# BGP_Aggregation_And_AS_SET_AS_SET_Skeleton
# BGP_Aggregation_And_AS_SET_AS_SET_Summary_Only_Skeleton
# BGP_Aggregation_And_AS_SET_With_Component_Origination_Skeleton
# BGP_Aggregation_And_AS_SET_IPv6_Skeleton
# BGP_Aggregation_And_AS_SET_Verification_Skeleton
# BGP_Aggregation_And_AS_SET_Verification_Commands
# BGP_Aggregation_And_AS_SET_Rollback
# BGP_Aggregation_And_AS_SET_Failure_Checks
# Attached_Labs

Blunt rule: aggregate-address reduces control-plane noise, but it can also hide reachability detail. summary-only hides the components. as-set preserves AS path loop-prevention information. If you advertise a broad aggregate without real component reachability, you built a blackhole.
