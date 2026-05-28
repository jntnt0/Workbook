BGP_Conditional_Advertisement.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP policy, route maps, attributes, conditional advertisement, and controlled advertisement behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP conditional advertisement syntax and verification |
| `All_combined_part3.md` lines 65763-65805 | Conditional advertisement overview and non-exist-map syntax | Supports the mental model that conditional advertisement withholds selected prefixes until tracked prefixes exist or do not exist |
| `All_combined_part3.md` lines 65801-65805 | Non-exist conditional advertisement syntax | Supports `neighbor <neighbor> advertise-map <advertise-map> non-exist-map <condition-map>` |
| `All_combined_part3.md` lines 65829-65833 | Exist conditional advertisement syntax | Supports `neighbor <neighbor> advertise-map <advertise-map> exist-map <condition-map>` |
| `All_combined_part3.md` lines 65897-65905 | Non-exist-map example | Supports route-map based conditional advertisement for a primary/backup scenario |
| `All_combined_part3.md` lines 66009-66057 | Exist-map example | Supports route-map based advertisement when a trigger prefix exists |
| `All_combined_part3.md` lines 130224-130269 | Conditional advertisement lab explanation | Supports advertise-map, non-exist-map, and route withdrawal behavior when the condition is not met |
| `All_combined_part3.md` lines 130413-130413 | Exist-map lab syntax | Supports `neighbor <neighbor> advertise-map <advertise-map> exist-map <exist-map>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25112-25132 | Conditional route selection and route maps | Supports using route maps for selective route advertisement, filtering, and manipulation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 31001-31520 | Route maps and conditional matching | Supports prefix-list and route-map match logic, route-map sequence behavior, and implicit deny behavior |
| `iosxe_combined_pdfs_.md` lines 33493-33493 | Conditional advertisement neighbor status | Supports verifying whether conditional advertisement is enabled or disabled in neighbor output |
# BGP_Conditional_Advertisement_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Conditional advertisement | BGP advertises selected prefixes to a neighbor only when a separate condition is met |
| Advertise-map | Route map that defines the prefixes that may be advertised conditionally |
| Exist-map | Route map that defines the trigger prefix that must exist before the advertise-map prefixes are advertised |
| Non-exist-map | Route map that defines the trigger prefix that must be absent before the advertise-map prefixes are advertised |
| Trigger prefix | Prefix used as the condition, not necessarily the prefix being advertised |
| Advertised prefix | Prefix sent to the neighbor only when the condition is true |
| Route-map dependency | Conditional advertisement uses route maps for both advertised routes and trigger routes |
| Prefix-list dependency | Prefix lists are commonly used inside the advertise-map and condition-map route maps |
| Prefix must exist in BGP | Conditional advertisement does not create routes; the advertised prefix must already exist in the BGP table |
| Exist-map logic | Advertise the selected prefix when the trigger prefix exists |
| Non-exist-map logic | Advertise the selected prefix when the trigger prefix does not exist |
| Withdrawal behavior | If the condition becomes false, BGP withdraws the conditionally advertised prefix from that neighbor |
| Neighbor-specific policy | Conditional advertisement is applied per neighbor or peer group |
| Outbound-only behavior | Conditional advertisement controls what this router advertises to a neighbor |
| Primary/backup use case | Advertise backup route only when primary trigger route disappears |
| Controlled advertisement use case | Advertise a route only when a dependency route is present |
| Not default-originate | Conditional advertisement is for selected existing BGP prefixes; conditional default-originate is a related but separate default-route mechanism |
| Blunt rule | Conditional advertisement is not magic failover. It is route-map-driven outbound withholding based on whether a trigger prefix exists or does not exist in BGP |
# BGP_Conditional_Advertisement_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor is established | Router | `show bgp ipv4 unicast summary` | Neighbor shows a number under `State/PfxRcd` |
| 2 | Confirm target neighbor | Router / Notes | `<neighbor-ip>` | Neighbor that should receive conditional advertisement is known |
| 3 | Confirm advertised prefix | Router / Notes | `<advertised-prefix>/<length>` | Prefix that may be advertised is known |
| 4 | Confirm trigger prefix | Router / Notes | `<trigger-prefix>/<length>` | Prefix that controls the condition is known |
| 5 | Decide condition type | Router / Notes | `exist-map` or `non-exist-map` | Logic is clear before configuration |
| 6 | Use exist-map when trigger prefix must exist | Router / Notes | `advertise when trigger exists` | Advertisement occurs only when trigger route exists |
| 7 | Use non-exist-map when trigger prefix must disappear | Router / Notes | `advertise when trigger is absent` | Advertisement occurs only when trigger route is absent |
| 8 | Confirm advertised prefix exists in BGP | Router | `show bgp ipv4 unicast <advertised-prefix>` | Prefix exists in local BGP table |
| 9 | Confirm advertised prefix exists in RIB if locally originated by network statement | Router | `show ip route <advertised-prefix> <mask>` | Exact route exists if using BGP `network` statement |
| 10 | Confirm trigger prefix current state | Router | `show bgp ipv4 unicast <trigger-prefix>` | Trigger prefix is present or absent as expected before test |
| 11 | Confirm current advertised routes to neighbor | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Baseline advertisement state is known |
| 12 | Review existing BGP config | Router | `show running-config | section router bgp` | Existing neighbor, AFI, and policy config are visible |
| 13 | Review existing route maps | Router | `show running-config | section route-map` | Existing route maps are visible |
| 14 | Review existing prefix lists | Router | `show running-config | include ip prefix-list` | Existing prefix lists are visible |
| 15 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 16 | Create prefix list for advertised prefix | Router | `ip prefix-list <advertise-pl> permit <advertised-prefix>/<length>` | Advertise prefix list matches only the intended route |
| 17 | Create prefix list for trigger prefix | Router | `ip prefix-list <condition-pl> permit <trigger-prefix>/<length>` | Condition prefix list matches only the trigger route |
| 18 | Create advertise route map | Router | `route-map <advertise-map-name> permit 10` | Advertise-map route map exists |
| 19 | Match advertised prefix in advertise map | Router | `match ip address prefix-list <advertise-pl>` | Advertise map identifies the route that may be sent |
| 20 | Do not add broad fallback to advertise map unless intended | Router / Notes | `No permit any fallback` | Advertise-map does not accidentally advertise unrelated prefixes |
| 21 | Create condition route map | Router | `route-map <condition-map-name> permit 10` | Exist/non-exist condition route map exists |
| 22 | Match trigger prefix in condition map | Router | `match ip address prefix-list <condition-pl>` | Condition map identifies the trigger route |
| 23 | Do not add broad fallback to condition map | Router / Notes | `No permit any fallback` | Condition does not become always true or always false by accident |
| 24 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 25 | Enter IPv4 unicast address family | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 26 | Confirm neighbor is activated | Router | `neighbor <neighbor-ip> activate` | Neighbor exchanges IPv4 unicast NLRI |
| 27 | Configure exist-map conditional advertisement | Router | `neighbor <neighbor-ip> advertise-map <advertise-map-name> exist-map <condition-map-name>` | Advertised prefix is sent only when trigger prefix exists |
| 28 | Configure non-exist-map conditional advertisement | Router | `neighbor <neighbor-ip> advertise-map <advertise-map-name> non-exist-map <condition-map-name>` | Advertised prefix is sent only when trigger prefix is absent |
| 29 | Configure peer-group conditional advertisement if used | Router | `neighbor <peer-group-name> advertise-map <advertise-map-name> exist-map <condition-map-name>` | Peer-group members inherit conditional advertisement |
| 30 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 31 | Save configuration | Router | `copy running-config startup-config` | Conditional advertisement config is saved |
| 32 | Soft clear outbound updates | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Neighbor receives updated advertisement state without hard reset |
| 33 | Verify conditional advertisement config | Router | `show running-config | section router bgp` | Neighbor has advertise-map plus exist-map or non-exist-map |
| 34 | Verify route maps | Router | `show route-map <advertise-map-name>` and `show route-map <condition-map-name>` | Route maps exist and counters are visible |
| 35 | Verify prefix lists | Router | `show ip prefix-list <advertise-pl>` and `show ip prefix-list <condition-pl>` | Prefix lists match intended routes only |
| 36 | Verify advertised prefix is in local BGP | Router | `show bgp ipv4 unicast <advertised-prefix>` | Advertised prefix exists locally |
| 37 | Verify trigger prefix state | Router | `show bgp ipv4 unicast <trigger-prefix>` | Trigger prefix state matches expected condition |
| 38 | Verify neighbor conditional advertisement status | Router | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Neighbor output shows conditional advertisement state if supported |
| 39 | Verify advertised routes when condition is true | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix appears when condition is true |
| 40 | Verify route absent when condition is false | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix is absent or withdrawn when condition is false |
| 41 | Verify peer received route | Peer Router | `show bgp ipv4 unicast <advertised-prefix>` | Peer sees route only when condition is true |
| 42 | Verify peer installed route if valid | Peer Router | `show ip route <advertised-prefix>` | Peer installs route if selected and next hop is reachable |
| 43 | Test exist-map by adding trigger route | Router | `<restore or originate trigger prefix>` | Trigger prefix appears in BGP |
| 44 | Verify exist-map advertisement begins | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix appears |
| 45 | Test exist-map by removing trigger route | Router | `<withdraw trigger prefix>` | Trigger prefix disappears from BGP |
| 46 | Verify exist-map advertisement is withdrawn | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix disappears |
| 47 | Test non-exist-map by removing trigger route | Router | `<withdraw trigger prefix>` | Trigger prefix disappears from BGP |
| 48 | Verify non-exist-map advertisement begins | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix appears |
| 49 | Test non-exist-map by restoring trigger route | Router | `<restore or originate trigger prefix>` | Trigger prefix reappears in BGP |
| 50 | Verify non-exist-map advertisement is withdrawn | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix disappears |
| 51 | Verify route-map counters after test | Router | `show route-map <advertise-map-name>` and `show route-map <condition-map-name>` | Counters reflect matches during testing |
| 52 | Verify BGP logs | Router | `show logging | include BGP|ADJCHANGE` | No unexpected neighbor reset occurred |
# BGP_Conditional_Advertisement_Non_Exist_Map_Skeleton
configure terminal
ip prefix-list <advertise-pl> permit <advertised-prefix>/<length>
ip prefix-list <condition-pl> permit <trigger-prefix>/<length>
route-map <advertise-map-name> permit 10
 match ip address prefix-list <advertise-pl>
route-map <condition-map-name> permit 10
 match ip address prefix-list <condition-pl>
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> advertise-map <advertise-map-name> non-exist-map <condition-map-name>
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Conditional_Advertisement_Exist_Map_Skeleton
configure terminal
ip prefix-list <advertise-pl> permit <advertised-prefix>/<length>
ip prefix-list <condition-pl> permit <trigger-prefix>/<length>
route-map <advertise-map-name> permit 10
 match ip address prefix-list <advertise-pl>
route-map <condition-map-name> permit 10
 match ip address prefix-list <condition-pl>
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> advertise-map <advertise-map-name> exist-map <condition-map-name>
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Conditional_Advertisement_With_Local_Origination_Skeleton
configure terminal
ip route <advertised-prefix> <advertised-mask> <next-hop-or-null0>
ip prefix-list <advertise-pl> permit <advertised-prefix>/<length>
ip prefix-list <condition-pl> permit <trigger-prefix>/<length>
route-map <advertise-map-name> permit 10
 match ip address prefix-list <advertise-pl>
route-map <condition-map-name> permit 10
 match ip address prefix-list <condition-pl>
router bgp <local-asn>
 address-family ipv4 unicast
  network <advertised-prefix> mask <advertised-mask>
  neighbor <neighbor-ip> advertise-map <advertise-map-name> non-exist-map <condition-map-name>
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Conditional_Advertisement_Primary_Backup_Non_Exist_Map_Skeleton
configure terminal
ip prefix-list BACKUP-ADVERTISE permit <backup-prefix>/<length>
ip prefix-list PRIMARY-TRIGGER permit <primary-prefix>/<length>
route-map ADVERTISE-BACKUP permit 10
 match ip address prefix-list BACKUP-ADVERTISE
route-map PRIMARY-EXISTS permit 10
 match ip address prefix-list PRIMARY-TRIGGER
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <backup-neighbor-ip> advertise-map ADVERTISE-BACKUP non-exist-map PRIMARY-EXISTS
 exit-address-family
end
clear bgp ipv4 unicast <backup-neighbor-ip> soft out
copy running-config startup-config
# BGP_Conditional_Advertisement_Dependency_Exist_Map_Skeleton
configure terminal
ip prefix-list DEPENDENT-ADVERTISE permit <advertised-prefix>/<length>
ip prefix-list REQUIRED-TRIGGER permit <trigger-prefix>/<length>
route-map ADVERTISE-DEPENDENT permit 10
 match ip address prefix-list DEPENDENT-ADVERTISE
route-map REQUIRED-EXISTS permit 10
 match ip address prefix-list REQUIRED-TRIGGER
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> advertise-map ADVERTISE-DEPENDENT exist-map REQUIRED-EXISTS
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Conditional_Advertisement_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <advertised-prefix>
show bgp ipv4 unicast <trigger-prefix>
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show ip route <advertised-prefix>
show ip route <trigger-prefix>
show route-map <advertise-map-name>
show route-map <condition-map-name>
show ip prefix-list <advertise-pl>
show ip prefix-list <condition-pl>
show running-config | section router bgp
show running-config | section route-map
show logging | include BGP|ADJCHANGE
# BGP_Conditional_Advertisement_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP session | `show bgp ipv4 unicast summary` | Neighbor is established |
| Verify advertised prefix exists locally | `show bgp ipv4 unicast <advertised-prefix>` | Prefix exists in local BGP table |
| Verify trigger prefix state | `show bgp ipv4 unicast <trigger-prefix>` | Trigger prefix is present or absent depending on test case |
| Verify advertised prefix RIB source | `show ip route <advertised-prefix>` | Prefix exists in RIB if locally originated with `network` |
| Verify trigger prefix RIB state | `show ip route <trigger-prefix>` | Trigger route state is understood |
| Verify conditional advertisement config | `show running-config | section router bgp` | Neighbor has advertise-map plus exist-map or non-exist-map |
| Verify advertise route map | `show route-map <advertise-map-name>` | Advertise-map route map exists and counters are visible |
| Verify condition route map | `show route-map <condition-map-name>` | Exist/non-exist condition map exists and counters are visible |
| Verify advertise prefix list | `show ip prefix-list <advertise-pl>` | Prefix list matches intended advertised prefix only |
| Verify condition prefix list | `show ip prefix-list <condition-pl>` | Prefix list matches intended trigger prefix only |
| Verify neighbor detail | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Conditional advertisement state appears if supported |
| Verify advertised routes | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised prefix appears only when condition is true |
| Verify peer received route | `show bgp ipv4 unicast <advertised-prefix>` on peer | Peer sees advertised prefix only when condition is true |
| Verify route withdrawal | `show bgp ipv4 unicast <advertised-prefix>` on peer | Peer no longer sees prefix after condition becomes false |
| Verify peer RIB install | `show ip route <advertised-prefix>` on peer | Peer installs route if valid and selected |
| Verify outbound refresh | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Policy is refreshed without hard reset |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected session reset occurred |
# BGP_Conditional_Advertisement_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current conditional advertisement config | Router | `show running-config | section router bgp` | Neighbor advertise-map and condition map are visible |
| 2 | Identify advertise and condition route maps | Router | `show running-config | section route-map` | Route maps are visible |
| 3 | Identify prefix lists | Router | `show running-config | include ip prefix-list` | Advertised and trigger prefix lists are visible |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 7 | Remove non-exist conditional advertisement | Router | `no neighbor <neighbor-ip> advertise-map <advertise-map-name> non-exist-map <condition-map-name>` | Non-exist conditional advertisement is removed |
| 8 | Remove exist conditional advertisement | Router | `no neighbor <neighbor-ip> advertise-map <advertise-map-name> exist-map <condition-map-name>` | Exist conditional advertisement is removed |
| 9 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 10 | Remove advertise route map | Router | `no route-map <advertise-map-name> permit 10` | Advertise route-map sequence is removed |
| 11 | Remove condition route map | Router | `no route-map <condition-map-name> permit 10` | Condition route-map sequence is removed |
| 12 | Remove advertise prefix list if lab-only | Router | `no ip prefix-list <advertise-pl>` | Advertise prefix list is removed |
| 13 | Remove condition prefix list if lab-only | Router | `no ip prefix-list <condition-pl>` | Condition prefix list is removed |
| 14 | Remove static advertised route if lab-only | Router | `no ip route <advertised-prefix> <advertised-mask> <next-hop-or-null0>` | Lab-only static route is removed |
| 15 | Soft clear outbound | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Neighbor receives updated advertisement state |
| 16 | Verify conditional advertisement removed | Router | `show running-config | section router bgp` | Advertise-map command no longer appears |
| 17 | Verify advertised-routes behavior | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Prefix advertisement follows normal policy again |
| 18 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Conditional_Advertisement_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Advertised prefix never appears | `show bgp ipv4 unicast <advertised-prefix>` | Advertised prefix does not exist in local BGP table |
| Advertised prefix exists in RIB but not BGP | `show running-config | section router bgp` | Missing `network`, redistribution, or inbound route source into BGP |
| Exist-map never triggers | `show bgp ipv4 unicast <trigger-prefix>` | Trigger prefix does not exist in BGP |
| Non-exist-map never advertises | `show bgp ipv4 unicast <trigger-prefix>` | Trigger prefix still exists in BGP |
| Non-exist-map keeps advertising when it should stop | `show bgp ipv4 unicast <trigger-prefix>` | Trigger prefix did not return to BGP or condition map does not match it |
| Route-map counters stay zero | `show route-map <advertise-map-name>` and `show route-map <condition-map-name>` | Prefix lists do not match, route maps are not referenced, or condition route is absent |
| Prefix-list counters stay zero | `show ip prefix-list <pl-name>` | Prefix or prefix length is wrong |
| Advertisement goes to wrong neighbor | `show running-config | section router bgp` | Conditional advertisement applied to wrong neighbor or peer group |
| Route is advertised unconditionally | `show running-config | section router bgp` | Prefix also allowed by another outbound policy or advertised to a different neighbor without condition |
| Route withdrawn unexpectedly | `show bgp ipv4 unicast <trigger-prefix>` | Trigger condition changed and conditional advertisement withdrew the route |
| Exist-map and non-exist-map logic is reversed | Config review | Used `exist-map` when backup behavior required `non-exist-map`, or reverse |
| Broad route-map makes condition always true | `show running-config | section route-map` | Condition map has a permissive fallback or broad match |
| Broad advertise-map sends too many routes | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertise-map matches more prefixes than intended |
| Route appears on local router but not peer | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Condition false, outbound policy blocks it, next-hop issue, or peer rejects it |
| Peer receives route but does not install | Peer `show bgp ipv4 unicast <advertised-prefix>` and `show ip route <advertised-prefix>` | Next hop unreachable, better route exists, or policy blocks install |
| Soft clear does not update behavior | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Wrong neighbor cleared or wrong policy direction changed |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Used hard reset instead of soft outbound refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-conditional-advertisement-final` | `BGP_Conditional_Advertisement.md` | Primary lab for advertise-map, exist-map, non-exist-map, trigger prefix logic, route withdrawal, and neighbor-specific conditional advertisement |
# Index
# Source_Basis
# BGP_Conditional_Advertisement_Mental_Model
# BGP_Conditional_Advertisement_Configuration_Checklist
# BGP_Conditional_Advertisement_Non_Exist_Map_Skeleton
# BGP_Conditional_Advertisement_Exist_Map_Skeleton
# BGP_Conditional_Advertisement_With_Local_Origination_Skeleton
# BGP_Conditional_Advertisement_Primary_Backup_Non_Exist_Map_Skeleton
# BGP_Conditional_Advertisement_Dependency_Exist_Map_Skeleton
# BGP_Conditional_Advertisement_Verification_Skeleton
# BGP_Conditional_Advertisement_Verification_Commands
# BGP_Conditional_Advertisement_Rollback
# BGP_Conditional_Advertisement_Failure_Checks
# Attached_Labs

Blunt rule: conditional advertisement does not create the route. It only decides whether an existing BGP prefix is advertised to a specific neighbor. If the advertised prefix is not already in BGP, your advertise-map has nothing to send.