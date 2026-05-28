
BGP_Route_Dampening.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP route flap dampening, route stability, graded dampening, and route policy behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP dampening configuration and verification syntax |
| `iosxe_combined_pdfs_.md` lines 34161-34177 | IOS XE route dampening CLI | Supports `bgp dampening` and `bgp dampening <half-life> <reuse> <suppress> <max-suppress> [route-map <map>]` |
| `iosxe_combined_pdfs_.md` lines 34203-34209 | IOS XE route dampening verification | Supports `show ip bgp dampened-paths` and `show ip bgp flap-statistics` |
| `All_combined_part3.md` lines 63321-63347 | Route flap dampening goals and penalty model | Supports dampening purpose, route flap history, penalty, suppress limit, and history state |
| `All_combined_part3.md` lines 63347-63393 | Damp state, half-life, reuse limit, and maximum suppress time | Supports how routes are suppressed, decay over time, and become reusable |
| `All_combined_part3.md` lines 63433-63487 | Dampening CLI and parameter warning | Supports `bgp dampening <half-life> <reuse-limit> <suppress-limit> <maximum-suppress-time>` and warns that bad values can make dampening ineffective |
| `All_combined_part3.md` lines 63491-63507 | External-route scope and per-path behavior | Confirms BGP dampening affects only external BGP routes and operates per path |
| `All_combined_part3.md` lines 63515-63525 | Hard clears and flap dampening | Confirms excessive BGP session resets can trigger route flap dampening and that soft clears reduce disruption |
| `All_combined_part3.md` lines 87879-87957 | Graded route flap dampening | Supports route-map based graded dampening, exclusion of critical prefixes, and `set dampening` under route maps |
| `All_combined_part3.md` lines 88031-88047 | Graded dampening behavior by prefix length | Supports different dampening profiles for different prefix lengths and the warning that some operational events can cause multiple flaps |
# BGP_Route_Dampening_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Route flap dampening | Suppresses unstable external BGP paths so repeated flaps do not churn the routing table |
| Flap | A route withdrawal, reannouncement, or significant path change that increases the route penalty |
| Penalty | Numeric score assigned to a route when it flaps |
| Default flap penalty | A normal route flap adds a penalty of 1000 |
| Attribute-change penalty | A path attribute-only change can add a smaller penalty |
| History state | Route has flapped and is being tracked, but is not necessarily suppressed yet |
| Suppress limit | Penalty threshold above which the path is suppressed |
| Damp state | Route is suppressed and not considered for best-path selection |
| Reuse limit | Penalty threshold below which a suppressed route becomes usable again |
| Half-life | Time interval used to decay the penalty |
| Maximum suppress time | Upper bound on how long a route can remain suppressed |
| External route scope | Dampening applies to external BGP routes, not internal BGP prefixes |
| Per-path behavior | One path to a prefix can be dampened while another path to the same prefix remains usable |
| Flat dampening | Same dampening values apply to all eligible prefixes |
| Graded dampening | Different prefixes receive different dampening values through a route map |
| Critical-prefix exclusion | Important infrastructure prefixes can be excluded from dampening with route-map deny logic |
| Prefix-length sensitivity | Shorter aggregates usually deserve less aggressive suppression than longer specifics |
| Bad parameter risk | If maximum possible penalty cannot exceed suppress limit, dampening never actually suppresses routes |
| Soft clear relevance | Hard BGP resets can create churn and may contribute to dampening; soft clear is safer for policy changes |
| Blunt rule | Route dampening is a stability tool, not a default best practice. Use it deliberately, verify parameters, and do not suppress important reachability by accident |
# BGP_Route_Dampening_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | External BGP neighbors are established |
| 2 | Confirm target route source | Router | `show bgp ipv4 unicast <prefix>` | Prefix is learned from eBGP if dampening is expected to apply |
| 3 | Confirm route is not an internal BGP-only path | Router | `show bgp ipv4 unicast <prefix>` | Route is external, not purely iBGP |
| 4 | Confirm operational reason for dampening | Notes | `unstable external route / lab dampening test / graded policy` | Dampening is being used for a real reason |
| 5 | Confirm whether flat or graded dampening is required | Notes | `flat` or `graded` | Dampening design is selected before configuration |
| 6 | Confirm default dampening values if using flat default | Notes | `15 750 2000 60` | Default-style values are understood |
| 7 | Confirm custom half-life | Notes | `<half-life-minutes>` | Penalty decay interval is known |
| 8 | Confirm custom reuse limit | Notes | `<reuse-limit>` | Route reusability threshold is known |
| 9 | Confirm custom suppress limit | Notes | `<suppress-limit>` | Suppression threshold is known |
| 10 | Confirm maximum suppress time | Notes | `<max-suppress-minutes>` | Maximum suppression time is known |
| 11 | Validate dampening math | Notes | `max-penalty = reuse-limit * 2^(max-suppress-time / half-life)` | Max penalty is greater than suppress limit |
| 12 | Reject ineffective parameters | Notes | `max-penalty <= suppress-limit` | Dampening values are corrected before deployment |
| 13 | Identify prefixes to exclude from dampening | Notes | `<critical-prefix-list>` | Critical prefixes are not accidentally suppressed |
| 14 | Identify short dampening profile prefixes | Notes | `<short-damp-prefix-list>` | Shorter aggregates or important ranges have gentle suppression |
| 15 | Identify medium dampening profile prefixes | Notes | `<medium-damp-prefix-list>` | Medium-length prefixes have moderate suppression |
| 16 | Identify long dampening profile prefixes | Notes | `<long-damp-prefix-list>` | Longer specifics can receive stronger suppression |
| 17 | Review current BGP config | Router | `show running-config | section router bgp` | Current BGP and dampening config is visible |
| 18 | Review existing route maps | Router | `show running-config | section route-map` | Existing route-map logic is visible |
| 19 | Review existing prefix lists | Router | `show running-config | include ip prefix-list` | Existing prefix-list logic is visible |
| 20 | Review current flap statistics | Router | `show ip bgp flap-statistics` | Current route flap history is visible |
| 21 | Review current dampened paths | Router | `show ip bgp dampened-paths` | Current suppressed paths are visible |
| 22 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 23 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 24 | Enable default route dampening | Router | `bgp dampening` | BGP route dampening is enabled with default parameters |
| 25 | Enable custom flat dampening | Router | `bgp dampening <half-life> <reuse-limit> <suppress-limit> <max-suppress-time>` | BGP dampening is enabled with custom values |
| 26 | Create prefix list for excluded critical prefixes | Router | `ip prefix-list <exclude-pl> permit <critical-prefix>/<length>` | Critical prefixes are matched for exclusion |
| 27 | Create prefix list for short profile | Router | `ip prefix-list <short-pl> permit 0.0.0.0/0 le <max-length>` | Short-prefix profile is defined |
| 28 | Create prefix list for medium profile | Router | `ip prefix-list <medium-pl> permit 0.0.0.0/0 ge <min-length> le <max-length>` | Medium-prefix profile is defined |
| 29 | Create prefix list for long profile | Router | `ip prefix-list <long-pl> permit 0.0.0.0/0 ge <min-length>` | Long-prefix profile is defined |
| 30 | Create graded dampening route map exclusion for critical prefixes | Router | `route-map <graded-rm> deny 10` then `match ip address prefix-list <exclude-pl>` | Critical prefixes are excluded from dampening |
| 31 | Create short graded dampening sequence | Router | `route-map <graded-rm> permit 20` | Short dampening sequence exists |
| 32 | Match short prefix list | Router | `match ip address prefix-list <short-pl>` | Short profile matches intended prefixes |
| 33 | Set short dampening values | Router | `set dampening <half-life> <reuse-limit> <suppress-limit> <max-suppress-time>` | Short profile receives gentle dampening |
| 34 | Create medium graded dampening sequence | Router | `route-map <graded-rm> permit 30` | Medium dampening sequence exists |
| 35 | Match medium prefix list | Router | `match ip address prefix-list <medium-pl>` | Medium profile matches intended prefixes |
| 36 | Set medium dampening values | Router | `set dampening <half-life> <reuse-limit> <suppress-limit> <max-suppress-time>` | Medium profile receives selected dampening |
| 37 | Create long graded dampening sequence | Router | `route-map <graded-rm> permit 40` | Long dampening sequence exists |
| 38 | Match long prefix list | Router | `match ip address prefix-list <long-pl>` | Long profile matches intended prefixes |
| 39 | Set long dampening values | Router | `set dampening <half-life> <reuse-limit> <suppress-limit> <max-suppress-time>` | Long profile receives stronger dampening |
| 40 | Apply graded dampening route map | Router | `router bgp <local-asn>` then `bgp dampening route-map <graded-rm>` | BGP dampening uses route-map based profiles |
| 41 | Exit configuration mode | Router | `end` | Router returns to privileged EXEC mode |
| 42 | Save configuration | Router | `copy running-config startup-config` | Dampening configuration is saved |
| 43 | Verify dampening config | Router | `show running-config | section router bgp` | `bgp dampening` or route-map dampening appears |
| 44 | Verify route-map config | Router | `show running-config | section route-map` | `set dampening` values and deny exclusions are correct |
| 45 | Verify prefix-list config | Router | `show ip prefix-list` | Prefix lists match intended prefixes |
| 46 | Generate or observe route flap | Peer / Lab | `<withdraw and re-advertise prefix>` | Prefix penalty increases |
| 47 | Verify flap statistics | Router | `show ip bgp flap-statistics` | Flapping prefix appears with penalty/history data |
| 48 | Verify dampened paths | Router | `show ip bgp dampened-paths` | Suppressed prefix appears once penalty exceeds suppress limit |
| 49 | Verify BGP table state | Router | `show bgp ipv4 unicast <prefix>` | Dampened path is not selected for best path |
| 50 | Verify peer advertisement behavior | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Dampened path is not advertised while suppressed |
| 51 | Wait for penalty decay or lab timer | Router / Notes | `<wait until penalty falls below reuse>` | Route should become reusable after decay |
| 52 | Verify route reuse | Router | `show bgp ipv4 unicast <prefix>` | Route becomes usable again when penalty drops below reuse limit |
| 53 | Verify dampened path clears | Router | `show ip bgp dampened-paths` | Prefix disappears after unsuppression |
| 54 | Verify traffic path after reuse | Router | `show ip route <prefix>` and `show ip cef <prefix>` | Route installs and forwards again if best and valid |
| 55 | Avoid hard BGP clears during testing unless intentional | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` or `soft out` | Policy refresh does not create unnecessary flap churn |
# BGP_Route_Dampening_Default_Skeleton
configure terminal
router bgp <local-asn>
 bgp dampening
end
copy running-config startup-config
# BGP_Route_Dampening_Custom_Flat_Skeleton
configure terminal
router bgp <local-asn>
 bgp dampening <half-life> <reuse-limit> <suppress-limit> <max-suppress-time>
end
copy running-config startup-config
# BGP_Route_Dampening_Default_Style_Values_Skeleton
configure terminal
router bgp <local-asn>
 bgp dampening 15 750 2000 60
end
copy running-config startup-config
# BGP_Route_Dampening_Graded_Route_Map_Skeleton
configure terminal
ip prefix-list <exclude-pl> permit <critical-prefix>/<length>
ip prefix-list <short-pl> permit 0.0.0.0/0 le <short-max-length>
ip prefix-list <medium-pl> permit 0.0.0.0/0 ge <medium-min-length> le <medium-max-length>
ip prefix-list <long-pl> permit 0.0.0.0/0 ge <long-min-length>
route-map <graded-rm> deny 10
 match ip address prefix-list <exclude-pl>
route-map <graded-rm> permit 20
 match ip address prefix-list <short-pl>
 set dampening <short-half-life> <short-reuse> <short-suppress> <short-max-suppress>
route-map <graded-rm> permit 30
 match ip address prefix-list <medium-pl>
 set dampening <medium-half-life> <medium-reuse> <medium-suppress> <medium-max-suppress>
route-map <graded-rm> permit 40
 match ip address prefix-list <long-pl>
 set dampening <long-half-life> <long-reuse> <long-suppress> <long-max-suppress>
router bgp <local-asn>
 bgp dampening route-map <graded-rm>
end
copy running-config startup-config
# BGP_Route_Dampening_Graded_Example_Skeleton
configure terminal
ip prefix-list CRITICAL-PREFIXES permit <critical-prefix>/<length>
ip prefix-list SHORT-DAMP permit 0.0.0.0/0 le 21
ip prefix-list MEDIUM-DAMP permit 0.0.0.0/0 ge 22 le 23
ip prefix-list LONG-DAMP permit 0.0.0.0/0 ge 24
route-map GRADED-DAMPENING deny 10
 match ip address prefix-list CRITICAL-PREFIXES
route-map GRADED-DAMPENING permit 20
 match ip address prefix-list SHORT-DAMP
 set dampening 10 1500 3000 30
route-map GRADED-DAMPENING permit 30
 match ip address prefix-list MEDIUM-DAMP
 set dampening 15 750 3000 45
route-map GRADED-DAMPENING permit 40
 match ip address prefix-list LONG-DAMP
 set dampening 30 820 3000 60
router bgp <local-asn>
 bgp dampening route-map GRADED-DAMPENING
end
copy running-config startup-config
# BGP_Route_Dampening_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show ip bgp flap-statistics
show ip bgp flap-statistics <prefix> <mask>
show ip bgp dampened-paths
show ip route <prefix>
show ip cef <prefix>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show running-config | section router bgp
show running-config | section route-map
show ip prefix-list
show logging | include BGP|ADJCHANGE
# BGP_Route_Dampening_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP session state | `show bgp ipv4 unicast summary` | eBGP neighbors are established |
| Verify target prefix | `show bgp ipv4 unicast <prefix>` | Prefix appears as an external BGP path |
| Verify current route install | `show ip route <prefix>` | Route installs if best, valid, and not dampened |
| Verify CEF forwarding | `show ip cef <prefix>` | CEF points to expected next hop/interface |
| Verify dampening config | `show running-config | section router bgp` | `bgp dampening` or `bgp dampening route-map <map>` is configured |
| Verify graded route map | `show running-config | section route-map` | `set dampening` values and deny exclusions are correct |
| Verify prefix lists | `show ip prefix-list` | Prefix lists match excluded, short, medium, and long profiles |
| Verify route flap history | `show ip bgp flap-statistics` | Flapping prefixes and penalty information appear |
| Verify specific flap history | `show ip bgp flap-statistics <prefix> <mask>` | Target prefix flap data is visible |
| Verify dampened paths | `show ip bgp dampened-paths` | Suppressed paths appear with remaining dampening time |
| Verify dampened path not selected | `show bgp ipv4 unicast <prefix>` | Dampened path is not used as best path |
| Verify advertisement suppression | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Dampened prefix is not advertised while suppressed |
| Verify route reuse | `show bgp ipv4 unicast <prefix>` | Prefix becomes usable again after penalty drops below reuse limit |
| Verify no longer dampened | `show ip bgp dampened-paths` | Prefix disappears after unsuppression |
| Verify logs | `show logging | include BGP|ADJCHANGE` | BGP flaps and neighbor events are visible |
# BGP_Route_Dampening_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current dampening config | Router | `show running-config | section router bgp` | Dampening command and parameters are visible |
| 2 | Identify graded route map if used | Router | `show running-config | section route-map` | `set dampening` policy is visible |
| 3 | Identify prefix lists if used | Router | `show ip prefix-list` | Dampening-related prefix lists are visible |
| 4 | Check current dampened paths before rollback | Router | `show ip bgp dampened-paths` | Currently suppressed paths are known |
| 5 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 6 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 7 | Remove flat dampening | Router | `no bgp dampening` | Global flat dampening is disabled |
| 8 | Remove custom flat dampening if required by exact syntax | Router | `no bgp dampening <half-life> <reuse-limit> <suppress-limit> <max-suppress-time>` | Custom dampening command is removed if platform requires exact removal |
| 9 | Remove graded dampening route map | Router | `no bgp dampening route-map <graded-rm>` | Route-map based dampening is disabled |
| 10 | Exit BGP config mode | Router | `end` | Router returns to privileged EXEC mode |
| 11 | Remove graded route-map sequences if lab-only | Router | `configure terminal` then `no route-map <graded-rm> permit <seq>` | Lab-only route map sequences are removed |
| 12 | Remove graded route-map deny sequence if lab-only | Router | `no route-map <graded-rm> deny <seq>` | Critical-prefix exclusion sequence is removed |
| 13 | Remove prefix lists if lab-only | Router | `no ip prefix-list <pl-name>` | Dampening prefix lists are removed |
| 14 | Soft refresh BGP if policy changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` or `soft out` | Policy is refreshed without hard session reset |
| 15 | Verify dampening removed | Router | `show running-config | section router bgp` | Dampening command no longer appears |
| 16 | Verify dampened paths clear over time | Router | `show ip bgp dampened-paths` | Suppressed paths disappear or are no longer suppressed after state clears |
| 17 | Verify route usability | Router | `show bgp ipv4 unicast <prefix>` | Route is eligible for best-path selection again |
| 18 | Verify route install | Router | `show ip route <prefix>` | Route installs if selected and valid |
| 19 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Route_Dampening_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Dampening never suppresses routes | `show ip bgp flap-statistics` and parameter check | Suppress limit is too high or max penalty cannot exceed suppress limit |
| Dampening suppresses too aggressively | `show ip bgp dampened-paths` | Suppress limit too low, max suppress time too high, or prefix profile too strict |
| Route stays suppressed too long | `show ip bgp dampened-paths` | Maximum suppress time or half-life is too aggressive |
| Important prefix disappears | `show ip bgp dampened-paths` | Critical prefix was not excluded from graded dampening |
| iBGP route is not dampened | `show bgp ipv4 unicast <prefix>` | Expected behavior; BGP dampening applies to external BGP routes |
| One path dampened but another path still works | `show bgp ipv4 unicast <prefix>` | Expected per-path dampening behavior |
| Dampened prefix not advertised to peers | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected behavior while route is in Damp state |
| Prefix flaps but only shows history, not dampened | `show ip bgp flap-statistics` | Penalty has not exceeded suppress limit |
| Prefix becomes usable again later | `show ip bgp dampened-paths` | Expected behavior after penalty decays below reuse limit |
| Route-map dampening has no effect | `show running-config | section route-map` | Route-map not attached with `bgp dampening route-map <map>` |
| Route-map counters do not match expected prefixes | `show ip prefix-list` and `show running-config | section route-map` | Prefix-list match logic is wrong |
| Excluded prefix still dampened | `show running-config | section route-map` | Exclusion sequence is missing, ordered wrong, or does not match the prefix |
| All prefixes get same dampening despite graded design | `show running-config | section router bgp` | Flat `bgp dampening` used instead of route-map dampening |
| Lab route never flaps enough to test dampening | `show ip bgp flap-statistics` | Not enough withdraw/reannounce cycles occurred to exceed suppress limit |
| Hard BGP clear triggers unexpected flap churn | `show logging | include BGP|ADJCHANGE` | Hard clear was used instead of soft refresh |
| Neighbor reset causes dampening test confusion | `show bgp ipv4 unicast summary` | Session reset changed route state rather than real prefix instability |
| Route is best in BGP but not in RIB | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | RIB failure, next-hop problem, or better administrative distance route, not necessarily dampening |
| Dampening hides a policy issue | `show bgp ipv4 unicast <prefix>` and route policy review | Route is suppressed, so policy troubleshooting must wait until route is reusable |
| Modern design rejects dampening | Design review | Many networks avoid broad dampening because it can suppress reachability during normal operational events |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-dampening-final` | `BGP_Route_Dampening.md` | Primary lab for route flap penalty, suppress/reuse behavior, dampened-path verification, and dampening rollback |
# Index
# Source_Basis
# BGP_Route_Dampening_Mental_Model
# BGP_Route_Dampening_Configuration_Checklist
# BGP_Route_Dampening_Default_Skeleton
# BGP_Route_Dampening_Custom_Flat_Skeleton
# BGP_Route_Dampening_Default_Style_Values_Skeleton
# BGP_Route_Dampening_Graded_Route_Map_Skeleton
# BGP_Route_Dampening_Graded_Example_Skeleton
# BGP_Route_Dampening_Verification_Skeleton
# BGP_Route_Dampening_Verification_Commands
# BGP_Route_Dampening_Rollback
# BGP_Route_Dampening_Failure_Checks
# Attached_Labs

Blunt rule: BGP dampening can protect the network from unstable external paths, but broad dampening can also hide valid reachability after maintenance, reloads, or normal churn. Treat it like a scalpel, not a default checkbox.