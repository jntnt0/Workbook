
BGP_Route_Leak_Prevention.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP policy, prefix lists, AS path filters, communities, route maps, route control, and BGP design behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP policy, prefix-list, filter-list, route-map, and neighbor syntax |
| `All_combined_part3.md` lines 48250-48252 | BGP route leak definition | Supports route leak as an AS advertising a route to another AS when policy says it should not |
| `All_combined_part3.md` lines 57313-57330 | BGP routing and administrative policy | Supports inbound filtering, accepting routes only from expected upstream/customer sources, maximum-prefix, and advertising only locally intended routes |
| `All_combined_part3.md` lines 73429-73510 | Enterprise multihoming and partial routes | Supports enterprise border behavior, default-only, partial routes from providers, full tables, and provider/customer routing scope |
| `All_combined_part3.md` lines 86747-86880 | Transit, peering, and community design | Supports transit vs peering mental model, no-export, no-advertise, internet community behavior, and policy-based route control |
| `All_combined_part3.md` lines 74729-74851 | Outbound AS path filter examples | Supports `neighbor <ip> filter-list <list> out` and `ip as-path access-list <list> permit ^$` for advertising only local originated routes |
| `All_combined_part3.md` lines 132387-132400 | Inbound AS path filtering | Supports `ip as-path access-list`, denying/allowing AS path patterns, and applying with `neighbor <ip> filter-list <list> in` |
| `All_combined_part3.md` lines 132522-132526 | Outbound AS path leak filtering example | Supports denying/allowing AS path patterns and applying outbound `neighbor <ip> filter-list <list> out` |
| `All_combined_part3.md` lines 132768-132808 | AS path regex validation workflow | Supports testing regex with `show ip bgp regexp`, then applying with `ip as-path access-list` and `neighbor filter-list` |
| `iosxe_combined_pdfs_.md` lines 33643-33647 | IOS XE filter-list syntax | Supports `neighbor <ip-or-peer-group> filter-list <access-list-number-or-name> in|out` |
| `iosxe_combined_pdfs_.md` lines 33691-33695 | IOS XE prefix-list syntax | Supports `ip prefix-list <name> permit|deny <network>/<len> ge <value> le <value>` |
| `iosxe_combined_pdfs_.md` lines 34229-34233 | IOS XE route-map prefix-list matching | Supports `match ip address prefix-list <name>` inside route maps |
# BGP_Route_Leak_Prevention_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Route leak | A route is advertised to a BGP neighbor that should not receive it under the relationship policy |
| Route hijack | Unauthorized origin of someone else's prefix; related, but not the same as a leak |
| Customer route | Route received from or originated for a downstream customer |
| Provider route | Route learned from an upstream transit provider |
| Peer route | Route learned from a lateral settlement-free or private peer |
| Local route | Prefix originated by the local AS |
| Transit | Carrying traffic between two other ASNs |
| Customer-to-provider rule | Advertise local and customer routes to providers |
| Customer-to-peer rule | Advertise local and customer routes to peers |
| Provider-to-customer rule | Advertise local, customer, peer, and provider routes to customers if providing full transit |
| Peer-to-peer rule | Do not advertise routes learned from one peer to another peer |
| Provider-to-provider rule | Do not advertise routes learned from one provider to another provider unless you intentionally sell transit |
| Provider-to-peer rule | Do not advertise provider-learned routes to peers |
| Peer-to-provider rule | Do not advertise peer-learned routes to providers unless intentional |
| Inbound filter | Controls what routes this router accepts from a neighbor |
| Outbound filter | Controls what routes this router advertises to a neighbor |
| Prefix-list | Filters routes by prefix and prefix length |
| AS path ACL | Filters routes by AS_PATH pattern |
| Community tag | Marks route source or policy class so later outbound policy can act on it |
| `no-export` | Community that prevents advertising a route to external peers outside the AS or confederation boundary |
| `no-advertise` | Community that prevents advertising a route to any BGP peer |
| `^$` AS path regex | Matches locally originated BGP routes because the AS path is empty before external advertisement |
| `^<asn>_` AS path regex | Matches routes where the immediate neighboring AS is at the start of the AS_PATH |
| `_<asn>$` AS path regex | Matches routes originated by a specific AS at the end of the AS_PATH |
| Maximum-prefix | Administrative protection so a neighbor cannot flood the router with unexpected route volume |
| RPKI | Validates origin authorization; useful, but not sufficient by itself to prevent route leaks |
| Blunt rule | Route leak prevention is mostly outbound policy discipline. If you do not explicitly define who may receive customer, peer, provider, and local routes, you are trusting luck |
# BGP_Route_Leak_Prevention_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify local AS | Router / Notes | `<local-asn>` | Local BGP AS is known |
| 2 | Identify all BGP neighbors | Router | `show bgp ipv4 unicast summary` | All BGP peers and neighbor ASNs are visible |
| 3 | Classify each neighbor role | Notes | `customer`, `provider`, `peer`, `internal`, or `route-server` | Every neighbor has a policy role |
| 4 | Identify local originated prefixes | Notes | `<local-prefix-list>` | Prefixes owned/originated by local AS are known |
| 5 | Identify customer prefixes | Notes | `<customer-prefix-list>` | Customer prefixes authorized for transit are known |
| 6 | Identify provider-learned route scope | Notes | `default-only`, `partial`, or `full-table` | Expected inbound provider routes are known |
| 7 | Identify peer-learned route scope | Notes | `<peer-and-peer-customer-prefixes>` | Expected peer routes are known |
| 8 | Identify routes that must never leave | Notes | `<internal-only-prefixes>` | Internal, lab, management, and infrastructure routes are known |
| 9 | Confirm whether this AS sells transit | Notes | `transit-provider` or `non-transit-enterprise` | Outbound policy model is clear |
| 10 | Confirm BGP sessions are established | Router | `show bgp ipv4 unicast summary` | Relevant neighbors are established |
| 11 | Baseline received routes from each neighbor | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> routes` | Accepted inbound route set is known |
| 12 | Baseline advertised routes to each neighbor | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Current outbound advertisement set is known |
| 13 | Check for obvious leaked routes already being advertised | Router | `show bgp ipv4 unicast neighbors <provider-or-peer-ip> advertised-routes` | Provider/peer should not receive provider-learned or peer-learned routes |
| 14 | Review current BGP config | Router | `show running-config | section router bgp` | Current neighbor policies are visible |
| 15 | Review prefix lists | Router | `show running-config | include ip prefix-list` | Existing prefix filters are visible |
| 16 | Review AS path filters | Router | `show running-config | include ip as-path|filter-list` | Existing AS path filters are visible |
| 17 | Review route maps | Router | `show running-config | section route-map` | Existing match/set policy is visible |
| 18 | Review community policy | Router | `show running-config | include community|send-community` | Community tagging and sending behavior is visible |
| 19 | Create local-origin prefix list | Router | `ip prefix-list PL-LOCAL-ORIGIN permit <local-prefix>/<length>` | Local originated prefixes are matched |
| 20 | Create customer prefix list | Router | `ip prefix-list PL-CUSTOMER-ROUTES permit <customer-prefix>/<length>` | Authorized customer prefixes are matched |
| 21 | Create combined customer-transit export prefix list | Router | `ip prefix-list PL-EXPORT-TO-PROVIDER-PEER permit <local-prefix>/<length>` and `ip prefix-list PL-EXPORT-TO-PROVIDER-PEER permit <customer-prefix>/<length>` | Only local and customer routes are eligible for provider/peer export |
| 22 | Create deny list for never-export routes | Router | `ip prefix-list PL-NEVER-EXPORT deny <internal-prefix>/<length>` | Internal-only prefixes are blocked |
| 23 | Add final permit only if list is used as a prefilter with careful route-map logic | Router | `ip prefix-list PL-NEVER-EXPORT permit 0.0.0.0/0 le 32` | Non-internal prefixes continue to later policy if intended |
| 24 | Create AS path ACL for local-originated routes | Router | `ip as-path access-list ASPATH-LOCAL permit ^$` | Locally originated BGP routes are matched |
| 25 | Create AS path ACL for customer-originated routes | Router | `ip as-path access-list ASPATH-CUSTOMERS permit ^<customer-asn>_` | Routes learned from customer AS are matched |
| 26 | Create AS path ACL to deny private ASNs if required on public edge | Router | `ip as-path access-list ASPATH-DENY-PRIVATE deny _6451[2-9]_|_64[6-9][0-9][0-9]_|_65[0-4][0-9][0-9]_|_6553[0-4]_` | Private AS path leakage is blocked if regex is correct for platform |
| 27 | Test AS path regex before applying filter | Router | `show ip bgp regexp <regex>` | Regex matches intended routes only |
| 28 | Create community tagging route map for customer routes | Router | `route-map RM-TAG-CUSTOMER-IN permit 10` | Customer inbound tagging route map exists |
| 29 | Match customer prefixes for tagging | Router | `match ip address prefix-list PL-CUSTOMER-ROUTES` | Customer routes are matched |
| 30 | Set customer community | Router | `set community <local-asn>:100 additive` | Customer routes receive customer-source tag |
| 31 | Add permissive tagging fallback | Router | `route-map RM-TAG-CUSTOMER-IN permit 20` | Other accepted routes are not accidentally denied |
| 32 | Create provider inbound tagging route map | Router | `route-map RM-TAG-PROVIDER-IN permit 10` | Provider inbound tagging route map exists |
| 33 | Set provider community | Router | `set community <local-asn>:200 additive` | Provider-learned routes receive provider-source tag |
| 34 | Create peer inbound tagging route map | Router | `route-map RM-TAG-PEER-IN permit 10` | Peer inbound tagging route map exists |
| 35 | Set peer community | Router | `set community <local-asn>:300 additive` | Peer-learned routes receive peer-source tag |
| 36 | Create outbound route map to provider | Router | `route-map RM-OUT-PROVIDER permit 10` | Provider export route map exists |
| 37 | Permit only local and customer prefixes to provider | Router | `match ip address prefix-list PL-EXPORT-TO-PROVIDER-PEER` | Provider receives only authorized local/customer routes |
| 38 | Create provider outbound deny fallback | Router | `route-map RM-OUT-PROVIDER deny 100` | Everything else is denied outbound to provider |
| 39 | Create outbound route map to peer | Router | `route-map RM-OUT-PEER permit 10` | Peer export route map exists |
| 40 | Permit only local and customer prefixes to peer | Router | `match ip address prefix-list PL-EXPORT-TO-PROVIDER-PEER` | Peer receives only authorized local/customer routes |
| 41 | Create peer outbound deny fallback | Router | `route-map RM-OUT-PEER deny 100` | Everything else is denied outbound to peer |
| 42 | Create outbound route map to customer | Router | `route-map RM-OUT-CUSTOMER permit 10` | Customer export route map exists |
| 43 | Permit intended full or partial transit to customer | Router | `match ip address prefix-list <customer-export-policy-prefix-list>` | Customer receives the route set the service contract allows |
| 44 | Add no-export to routes that must stay inside local AS or confederation | Router | `set community no-export additive` | Routes are marked not to leave external boundary |
| 45 | Add no-advertise to routes that should not be advertised to any peer | Router | `set community no-advertise` | Routes are suppressed from all BGP peers |
| 46 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 47 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 48 | Apply customer inbound tagging | Router | `neighbor <customer-ip> route-map RM-TAG-CUSTOMER-IN in` | Customer-learned routes are tagged |
| 49 | Apply provider inbound tagging | Router | `neighbor <provider-ip> route-map RM-TAG-PROVIDER-IN in` | Provider-learned routes are tagged |
| 50 | Apply peer inbound tagging | Router | `neighbor <peer-ip> route-map RM-TAG-PEER-IN in` | Peer-learned routes are tagged |
| 51 | Apply outbound provider leak-prevention route map | Router | `neighbor <provider-ip> route-map RM-OUT-PROVIDER out` | Provider receives only permitted routes |
| 52 | Apply outbound peer leak-prevention route map | Router | `neighbor <peer-ip> route-map RM-OUT-PEER out` | Peer receives only permitted routes |
| 53 | Apply outbound customer policy | Router | `neighbor <customer-ip> route-map RM-OUT-CUSTOMER out` | Customer receives intended transit or partial route set |
| 54 | Apply AS path outbound filter to provider if using filter-list model | Router | `neighbor <provider-ip> filter-list ASPATH-LOCAL-CUSTOMER out` | Provider receives only AS paths allowed by AS path ACL |
| 55 | Apply AS path outbound filter to peer if using filter-list model | Router | `neighbor <peer-ip> filter-list ASPATH-LOCAL-CUSTOMER out` | Peer receives only AS paths allowed by AS path ACL |
| 56 | Enable community sending where community policy must propagate | Router | `neighbor <neighbor-ip> send-community` | Standard communities are advertised |
| 57 | Configure maximum-prefix for provider or peer inbound protection | Router | `neighbor <neighbor-ip> maximum-prefix <max-prefixes> <threshold-percent>` | Session is protected from unexpected route volume |
| 58 | Configure shutdown on maximum-prefix only if operationally intended | Router | `neighbor <neighbor-ip> maximum-prefix <max-prefixes> <threshold-percent> restart <minutes>` | Excessive prefixes trigger controlled session behavior |
| 59 | Exit AFI mode | Router | `exit-address-family` | Router exits address-family mode |
| 60 | Save configuration | Router | `copy running-config startup-config` | Route leak prevention policy is saved |
| 61 | Soft clear inbound after inbound policy changes | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound route tags/filters are reprocessed |
| 62 | Soft clear outbound after outbound policy changes | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound advertisements are refreshed |
| 63 | Verify provider advertisements | Router | `show bgp ipv4 unicast neighbors <provider-ip> advertised-routes` | Provider receives local and customer routes only |
| 64 | Verify peer advertisements | Router | `show bgp ipv4 unicast neighbors <peer-ip> advertised-routes` | Peer receives local and customer routes only |
| 65 | Verify customer advertisements | Router | `show bgp ipv4 unicast neighbors <customer-ip> advertised-routes` | Customer receives intended route set |
| 66 | Verify no provider route leaked to peer | Router | `show bgp ipv4 unicast neighbors <peer-ip> advertised-routes` | Provider-learned prefixes are absent |
| 67 | Verify no peer route leaked to provider | Router | `show bgp ipv4 unicast neighbors <provider-ip> advertised-routes` | Peer-learned prefixes are absent |
| 68 | Verify no provider route leaked to another provider | Router | `show bgp ipv4 unicast neighbors <provider2-ip> advertised-routes` | Provider1-learned prefixes are absent |
| 69 | Verify community tags | Router | `show bgp ipv4 unicast <prefix>` | Route shows expected customer, provider, peer, no-export, or no-advertise community |
| 70 | Verify route-map counters | Router | `show route-map <route-map-name>` | Route-map match counters increment |
| 71 | Verify prefix-list counters | Router | `show ip prefix-list <prefix-list-name>` | Prefix-list counters increment if supported |
| 72 | Verify AS path filter results | Router | `show ip bgp filter-list <aspath-list-name-or-number>` | Filter-list matches intended routes |
| 73 | Verify BGP table for unexpected transit paths | Router | `show bgp ipv4 unicast regexp <regex>` | Suspicious AS paths are identified |
| 74 | Verify logs | Router | `show logging | include BGP|ADJCHANGE|MAXPFX` | No unexpected reset or maximum-prefix breach occurred |
# BGP_Route_Leak_Prevention_Local_Only_Outbound_AS_Path_Filter_Skeleton
configure terminal
ip as-path access-list ASPATH-LOCAL permit ^$
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <provider-or-peer-ip> filter-list ASPATH-LOCAL out
 exit-address-family
end
clear bgp ipv4 unicast <provider-or-peer-ip> soft out
copy running-config startup-config
# BGP_Route_Leak_Prevention_Local_And_Customer_Outbound_Prefix_List_Skeleton
configure terminal
ip prefix-list PL-EXPORT-TO-PROVIDER-PEER permit <local-prefix>/<length>
ip prefix-list PL-EXPORT-TO-PROVIDER-PEER permit <customer-prefix>/<length>
route-map RM-OUT-PROVIDER-PEER permit 10
 match ip address prefix-list PL-EXPORT-TO-PROVIDER-PEER
route-map RM-OUT-PROVIDER-PEER deny 100
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <provider-or-peer-ip> route-map RM-OUT-PROVIDER-PEER out
 exit-address-family
end
clear bgp ipv4 unicast <provider-or-peer-ip> soft out
copy running-config startup-config
# BGP_Route_Leak_Prevention_Customer_Inbound_Tagging_Skeleton
configure terminal
ip bgp-community new-format
ip prefix-list PL-CUSTOMER-ROUTES permit <customer-prefix>/<length>
route-map RM-TAG-CUSTOMER-IN permit 10
 match ip address prefix-list PL-CUSTOMER-ROUTES
 set community <local-asn>:100 additive
route-map RM-TAG-CUSTOMER-IN permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <customer-ip> route-map RM-TAG-CUSTOMER-IN in
  neighbor <customer-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <customer-ip> soft in
copy running-config startup-config
# BGP_Route_Leak_Prevention_Provider_Inbound_Tagging_Skeleton
configure terminal
ip bgp-community new-format
route-map RM-TAG-PROVIDER-IN permit 10
 set community <local-asn>:200 additive
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <provider-ip> route-map RM-TAG-PROVIDER-IN in
 exit-address-family
end
clear bgp ipv4 unicast <provider-ip> soft in
copy running-config startup-config
# BGP_Route_Leak_Prevention_Peer_Inbound_Tagging_Skeleton
configure terminal
ip bgp-community new-format
route-map RM-TAG-PEER-IN permit 10
 set community <local-asn>:300 additive
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <peer-ip> route-map RM-TAG-PEER-IN in
 exit-address-family
end
clear bgp ipv4 unicast <peer-ip> soft in
copy running-config startup-config
# BGP_Route_Leak_Prevention_No_Export_Skeleton
configure terminal
ip prefix-list PL-INTERNAL-ONLY permit <internal-prefix>/<length>
route-map RM-INTERNAL-NO-EXPORT permit 10
 match ip address prefix-list PL-INTERNAL-ONLY
 set community no-export additive
route-map RM-INTERNAL-NO-EXPORT permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map RM-INTERNAL-NO-EXPORT out
  neighbor <neighbor-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Route_Leak_Prevention_No_Advertise_Skeleton
configure terminal
ip prefix-list PL-DO-NOT-ADVERTISE permit <prefix>/<length>
route-map RM-NO-ADVERTISE permit 10
 match ip address prefix-list PL-DO-NOT-ADVERTISE
 set community no-advertise
route-map RM-NO-ADVERTISE permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map RM-NO-ADVERTISE in
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Route_Leak_Prevention_AS_Path_Customer_Cone_Outbound_Skeleton
configure terminal
ip as-path access-list ASPATH-LOCAL-CUSTOMER permit ^$
ip as-path access-list ASPATH-LOCAL-CUSTOMER permit ^<customer-asn>_
ip as-path access-list ASPATH-LOCAL-CUSTOMER permit ^<customer-asn>_<downstream-customer-asn>_
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <provider-or-peer-ip> filter-list ASPATH-LOCAL-CUSTOMER out
 exit-address-family
end
clear bgp ipv4 unicast <provider-or-peer-ip> soft out
copy running-config startup-config
# BGP_Route_Leak_Prevention_Maximum_Prefix_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> maximum-prefix <max-prefixes> <threshold-percent>
 exit-address-family
end
copy running-config startup-config
# BGP_Route_Leak_Prevention_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast neighbors <neighbor-ip> routes
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast regexp <regex>
show ip bgp regexp <regex>
show ip bgp filter-list <aspath-list-name-or-number>
show route-map <route-map-name>
show ip prefix-list <prefix-list-name>
show running-config | section router bgp
show running-config | section route-map
show running-config | include ip prefix-list|ip as-path|community|maximum-prefix
show logging | include BGP|ADJCHANGE|MAXPFX
# BGP_Route_Leak_Prevention_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP sessions | `show bgp ipv4 unicast summary` | Relevant BGP neighbors are established |
| Verify accepted routes from neighbor | `show bgp ipv4 unicast neighbors <neighbor-ip> routes` | Inbound route set matches neighbor role |
| Verify advertised routes to provider | `show bgp ipv4 unicast neighbors <provider-ip> advertised-routes` | Provider receives local and customer routes only |
| Verify advertised routes to peer | `show bgp ipv4 unicast neighbors <peer-ip> advertised-routes` | Peer receives local and customer routes only |
| Verify advertised routes to customer | `show bgp ipv4 unicast neighbors <customer-ip> advertised-routes` | Customer receives intended full, default, or partial route set |
| Verify provider routes are not sent to peer | `show bgp ipv4 unicast neighbors <peer-ip> advertised-routes` | Provider-learned routes are absent |
| Verify peer routes are not sent to provider | `show bgp ipv4 unicast neighbors <provider-ip> advertised-routes` | Peer-learned routes are absent |
| Verify provider routes are not sent to another provider | `show bgp ipv4 unicast neighbors <provider2-ip> advertised-routes` | Provider-learned routes from provider1 are absent |
| Verify local-only AS path regex | `show ip bgp regexp ^$` | Locally originated routes are matched |
| Verify customer AS path regex | `show ip bgp regexp ^<customer-asn>_` | Customer-learned routes are matched |
| Verify filter-list output | `show ip bgp filter-list <aspath-list-name-or-number>` | Filter-list matches intended routes |
| Verify prefix list | `show ip prefix-list <prefix-list-name>` | Prefix-list permits only intended prefixes |
| Verify route-map hits | `show route-map <route-map-name>` | Route-map counters increment |
| Verify community tags | `show bgp ipv4 unicast <prefix>` | Route has expected customer, provider, peer, no-export, or no-advertise community |
| Verify neighbor policy attachment | `show running-config | section router bgp` | Route maps, filter-lists, send-community, and maximum-prefix are applied in correct direction |
| Verify route-map config | `show running-config | section route-map` | Match/set/deny logic is correct |
| Verify policy objects | `show running-config | include ip prefix-list|ip as-path|community|maximum-prefix` | Prefix-list, AS path ACL, community, and maximum-prefix config is visible |
| Verify logs | `show logging | include BGP|ADJCHANGE|MAXPFX` | No unexpected reset or maximum-prefix violation occurred |
# BGP_Route_Leak_Prevention_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current BGP policy attachments | Router | `show running-config | section router bgp` | Neighbor route maps, filter-lists, send-community, and maximum-prefix are visible |
| 2 | Identify route maps | Router | `show running-config | section route-map` | Route-map names and sequence numbers are visible |
| 3 | Identify prefix lists | Router | `show running-config | include ip prefix-list` | Prefix-list names are visible |
| 4 | Identify AS path filters | Router | `show running-config | include ip as-path` | AS path ACL names or numbers are visible |
| 5 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 6 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 7 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 8 | Remove inbound customer tagging route map | Router | `no neighbor <customer-ip> route-map RM-TAG-CUSTOMER-IN in` | Customer inbound tagging is detached |
| 9 | Remove inbound provider tagging route map | Router | `no neighbor <provider-ip> route-map RM-TAG-PROVIDER-IN in` | Provider inbound tagging is detached |
| 10 | Remove inbound peer tagging route map | Router | `no neighbor <peer-ip> route-map RM-TAG-PEER-IN in` | Peer inbound tagging is detached |
| 11 | Remove outbound provider route map | Router | `no neighbor <provider-ip> route-map RM-OUT-PROVIDER out` | Provider outbound route-map is detached |
| 12 | Remove outbound peer route map | Router | `no neighbor <peer-ip> route-map RM-OUT-PEER out` | Peer outbound route-map is detached |
| 13 | Remove outbound customer route map | Router | `no neighbor <customer-ip> route-map RM-OUT-CUSTOMER out` | Customer outbound route-map is detached |
| 14 | Remove outbound filter-list to provider | Router | `no neighbor <provider-ip> filter-list <aspath-list-name-or-number> out` | Provider AS path outbound filter is detached |
| 15 | Remove outbound filter-list to peer | Router | `no neighbor <peer-ip> filter-list <aspath-list-name-or-number> out` | Peer AS path outbound filter is detached |
| 16 | Remove maximum-prefix if lab-only | Router | `no neighbor <neighbor-ip> maximum-prefix <max-prefixes> <threshold-percent>` | Maximum-prefix guard is removed |
| 17 | Remove send-community only if lab-only and unused elsewhere | Router | `no neighbor <neighbor-ip> send-community` | Standard community sending is removed |
| 18 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 19 | Remove route-map sequences if lab-only | Router | `no route-map <route-map-name> permit <seq>` or `no route-map <route-map-name> deny <seq>` | Route-map sequences are removed |
| 20 | Remove prefix lists if lab-only | Router | `no ip prefix-list <prefix-list-name>` | Prefix list is removed |
| 21 | Remove AS path ACL if lab-only | Router | `no ip as-path access-list <aspath-list-name-or-number>` | AS path filter is removed |
| 22 | Soft clear inbound after inbound rollback | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are reprocessed |
| 23 | Soft clear outbound after outbound rollback | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound advertisements are refreshed |
| 24 | Verify rollback advertisement state | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Neighbor receives route set expected after rollback |
| 25 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Route_Leak_Prevention_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Provider receives another provider's routes | `show bgp ipv4 unicast neighbors <provider2-ip> advertised-routes` | Outbound provider policy missing or too broad |
| Peer receives provider-learned routes | `show bgp ipv4 unicast neighbors <peer-ip> advertised-routes` | Peer outbound policy allows noncustomer transit |
| Provider receives peer-learned routes | `show bgp ipv4 unicast neighbors <provider-ip> advertised-routes` | Provider outbound policy allows peer routes |
| Peer receives another peer's routes | `show bgp ipv4 unicast neighbors <peer2-ip> advertised-routes` | Peer-to-peer route leak; outbound peer policy missing |
| Customer does not receive intended transit | `show bgp ipv4 unicast neighbors <customer-ip> advertised-routes` | Customer outbound policy too restrictive |
| Local prefixes not advertised | `show bgp ipv4 unicast <local-prefix>` | Local prefix not originated, not in RIB, or blocked by outbound filter |
| Customer prefixes not advertised to provider | `show bgp ipv4 unicast <customer-prefix>` | Customer route not accepted, not tagged, or export prefix-list missing it |
| Route-map counters stay zero | `show route-map <route-map-name>` | Wrong neighbor, wrong direction, wrong AFI, or match condition wrong |
| Prefix-list counters stay zero | `show ip prefix-list <prefix-list-name>` | Prefix or length is wrong |
| AS path filter matches wrong routes | `show ip bgp regexp <regex>` | Regex is too broad or written for wrong AS path direction |
| `^$` filter blocks customer routes | `show ip bgp filter-list <list>` | Expected behavior; `^$` matches only locally originated routes |
| Private ASNs leak upstream | `show bgp ipv4 unicast neighbors <provider-ip> advertised-routes` | Missing private-AS removal/filtering or customer AS path policy |
| Internal prefixes leak externally | `show bgp ipv4 unicast neighbors <provider-or-peer-ip> advertised-routes` | Never-export prefix-list or no-export policy missing |
| `no-export` route still appears inside local AS | `show bgp ipv4 unicast community no-export` | Expected behavior; no-export blocks external advertisement, not internal use |
| `no-advertise` route disappears from all peers | `show bgp ipv4 unicast community no-advertise` | Expected behavior |
| Communities not seen by peer | Peer `show bgp ipv4 unicast <prefix>` | Missing `neighbor <ip> send-community` |
| Policy works locally but peer still has stale route | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound refresh not performed or peer has stale route |
| Inbound tagging did not appear | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound refresh not performed or route map not attached inbound |
| Maximum-prefix shuts down session | `show logging | include MAXPFX|BGP` | Neighbor sent more prefixes than allowed, or threshold is too low |
| Route leak detected externally but local advertised-routes looks clean | Route collector / peer check | Leak may be elsewhere, stale collector view, or another border router is leaking |
| RPKI Valid route still causes route leak | `show bgp ipv4 unicast <prefix>` | RPKI validates origin only; export policy still leaked route |
| Route leak prevention breaks traffic | `show ip route <destination>` and advertised-routes checks | Policy blocked required customer transit or default route |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used instead of soft route refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-route-leak-final` | `BGP_Route_Leak_Prevention.md` | Primary lab for BGP route leak behavior, customer/provider/peer export policy, prefix-list and AS path filtering, community tagging, and advertised-routes verification |
# Index
# Source_Basis
# BGP_Route_Leak_Prevention_Mental_Model
# BGP_Route_Leak_Prevention_Configuration_Checklist
# BGP_Route_Leak_Prevention_Local_Only_Outbound_AS_Path_Filter_Skeleton
# BGP_Route_Leak_Prevention_Local_And_Customer_Outbound_Prefix_List_Skeleton
# BGP_Route_Leak_Prevention_Customer_Inbound_Tagging_Skeleton
# BGP_Route_Leak_Prevention_Provider_Inbound_Tagging_Skeleton
# BGP_Route_Leak_Prevention_Peer_Inbound_Tagging_Skeleton
# BGP_Route_Leak_Prevention_No_Export_Skeleton
# BGP_Route_Leak_Prevention_No_Advertise_Skeleton
# BGP_Route_Leak_Prevention_AS_Path_Customer_Cone_Outbound_Skeleton
# BGP_Route_Leak_Prevention_Maximum_Prefix_Skeleton
# BGP_Route_Leak_Prevention_Verification_Skeleton
# BGP_Route_Leak_Prevention_Verification_Commands
# BGP_Route_Leak_Prevention_Rollback
# BGP_Route_Leak_Prevention_Failure_Checks
# Attached_Labs

Blunt rule: route leaks are usually outbound policy failures. RPKI will not save you from exporting a valid route to the wrong neighbor. Tag routes by source, filter exports by relationship, and verify with advertised-routes before you trust the edge.