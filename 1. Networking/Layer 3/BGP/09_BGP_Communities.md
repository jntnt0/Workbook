BGP_Communities.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP communities, route maps, policy control, AS path filters, and scalable BGP policy |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical source for IOS XE BGP community configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25833-25845 | BGP community definition and enablement | Supports community as optional transitive attribute, `ip bgp-community new-format`, and `neighbor send-community` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25869-25886 | Well-known communities and `no-advertise` | Supports `set community no-advertise` and the rule that routes are not advertised to any BGP peer |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25926-25930 | `no-export` community | Supports `set community no-export` and the rule that routes are not advertised outside the AS or confederation boundary |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25988-25992 | `local-AS` community | Supports `set community local-AS` and the rule that routes stay inside the local AS or confederation sub-AS scope |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26055-26096 | Community matching | Supports `show bgp ipv4 unicast community`, `ip community-list`, and `match community` route-map logic |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26140-26184 | Private community setting | Supports `set community <community> [additive]`, overwrite behavior, and additive preservation |
| `iosxe_combined_pdfs_.md` lines 33735-33739 | Community list syntax | Supports `ip community-list <list> permit <community>` |
| `iosxe_combined_pdfs_.md` lines 33829-33829 | Send community syntax | Supports `neighbor <ip-or-peer-group> send-community` |
| `iosxe_combined_pdfs_.md` lines 34293-34293 | Set community syntax | Supports `set community <community-number> [additive] [well-known-community]` |
# BGP_Communities_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP community | Route tag carried as a BGP attribute |
| Optional transitive attribute | Community can pass between ASNs if policies allow it and communities are sent |
| Private community | Operator-defined tag, usually written as `<asn>:<value>` |
| Well-known community | Standard predefined community with special propagation behavior |
| `send-community` | Required because IOS XE does not advertise communities to neighbors by default |
| `ip bgp-community new-format` | Displays communities in readable `<asn>:<value>` format |
| Route-map set logic | Used to attach communities to routes |
| Route-map match logic | Used to act on routes that already carry a community |
| `additive` | Preserves existing communities when adding a new one |
| No `additive` | Replaces existing communities with the new community set |
| `no-advertise` | Route is not advertised to any BGP peer |
| `no-export` | Route is not advertised outside the local AS or confederation boundary |
| `local-AS` | Route is not advertised outside the local AS or confederation sub-AS scope |
| Community list | ACL-like object used to match communities |
| Standard community list | Matches specific communities or well-known community names |
| Expanded community list | Uses regex-style matching for community patterns |
| Inbound community tagging | Local router tags received routes before storing/using/advertising them |
| Outbound community tagging | Local router tags routes before sending them to a neighbor |
| Policy chain | Tag routes first, then match those tags elsewhere |
| Blunt rule | Communities do nothing by themselves. They are labels. The network changes only when a route map, neighbor policy, or provider policy acts on those labels |
# BGP_Communities_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Neighbor shows a number under `State/PfxRcd` |
| 2 | Confirm target prefix exists | Router | `show bgp ipv4 unicast <prefix>` | Prefix appears in BGP table |
| 3 | Confirm current communities on prefix | Router | `show bgp ipv4 unicast <prefix>` | Existing community values are visible if present |
| 4 | Confirm current display format | Router | `show running-config | include bgp-community` | Community display format is known |
| 5 | Confirm community policy goal | Notes | `tag`, `match`, `block-advertisement`, `limit-export`, or `provider-policy` | Community purpose is clear |
| 6 | Confirm community value | Notes | `<asn>:<value>` or well-known community | Community value is known |
| 7 | Confirm whether existing communities must be preserved | Notes | `additive` or `overwrite` | Route-map set behavior is chosen deliberately |
| 8 | Confirm policy direction | Notes | `in` or `out` | Route map direction is known |
| 9 | Confirm neighbor or peer group | Notes | `<neighbor-ip>` or `<peer-group-name>` | Policy attachment point is known |
| 10 | Review current BGP config | Router | `show running-config | section router bgp` | Existing neighbor, AFI, and policy configuration is visible |
| 11 | Review current route maps | Router | `show running-config | section route-map` | Existing route-map logic is visible |
| 12 | Review current prefix lists | Router | `show running-config | include ip prefix-list` | Existing prefix matching logic is visible |
| 13 | Review current community lists | Router | `show running-config | include ip community-list` | Existing community match logic is visible |
| 14 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 15 | Enable readable community display | Router | `ip bgp-community new-format` | Communities display as `<asn>:<value>` |
| 16 | Create prefix list for target route | Router | `ip prefix-list <pl-name> permit <prefix>/<length>` | Prefix list matches intended route |
| 17 | Create route map to set community | Router | `route-map <set-rm-name> permit 10` | Route-map sequence exists |
| 18 | Match target prefix | Router | `match ip address prefix-list <pl-name>` | Route map matches intended prefix |
| 19 | Set private community and overwrite existing communities | Router | `set community <asn>:<value>` | Matching route receives only the configured community |
| 20 | Set private community and preserve existing communities | Router | `set community <asn>:<value> additive` | Matching route keeps existing communities and adds new community |
| 21 | Set multiple private communities | Router | `set community <asn>:<value-1> <asn>:<value-2> additive` | Matching route receives multiple communities |
| 22 | Set `no-advertise` community | Router | `set community no-advertise` | Matching route will not be advertised to any BGP peer |
| 23 | Set `no-export` community | Router | `set community no-export` | Matching route will not be advertised outside AS/confed boundary |
| 24 | Set `local-AS` community | Router | `set community local-AS` | Matching route is limited to local AS/sub-AS scope |
| 25 | Add permissive route-map sequence | Router | `route-map <set-rm-name> permit 20` | Unmatched routes are not accidentally denied |
| 26 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 27 | Enter IPv4 unicast address family | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 28 | Apply inbound community tagging policy | Router | `neighbor <neighbor-ip> route-map <set-rm-name> in` | Received matching routes are tagged |
| 29 | Apply outbound community tagging policy | Router | `neighbor <neighbor-ip> route-map <set-rm-name> out` | Advertised matching routes are tagged |
| 30 | Enable standard community sending to neighbor | Router | `neighbor <neighbor-ip> send-community` | Standard communities are sent to neighbor |
| 31 | Enable explicit standard community sending | Router | `neighbor <neighbor-ip> send-community standard` | Standard communities are sent to neighbor |
| 32 | Enable standard and extended community sending if needed | Router | `neighbor <neighbor-ip> send-community both` | Standard and extended communities are sent |
| 33 | Enable community sending for peer group if used | Router | `neighbor <peer-group-name> send-community` | Peer group members receive communities |
| 34 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 35 | Soft clear inbound after inbound policy change | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are reprocessed |
| 36 | Soft clear outbound after outbound policy change | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound routes are re-advertised |
| 37 | Verify route-map hit count | Router | `show route-map <set-rm-name>` | Route-map counters increment |
| 38 | Verify community on local route | Router | `show bgp ipv4 unicast <prefix>` | Prefix shows expected community |
| 39 | Verify advertised community | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Prefix is advertised with expected community if permitted |
| 40 | Verify neighbor received community | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer sees expected community |
| 41 | Create standard community list for matching | Router | `ip community-list standard <comm-list-name> permit <asn>:<value>` | Community list matches target private community |
| 42 | Create standard community list for well-known community | Router | `ip community-list standard <comm-list-name> permit no-advertise` | Community list matches well-known community |
| 43 | Create expanded community list if regex matching is needed | Router | `ip community-list expanded <comm-list-name> permit <regex>` | Expanded community pattern is defined |
| 44 | Create route map to match community | Router | `route-map <match-rm-name> permit 10` | Route-map sequence exists |
| 45 | Match community list | Router | `match community <comm-list-name>` | Route map matches routes carrying the community |
| 46 | Match exact community set if required | Router | `match community <comm-list-name> exact-match` | Route must match exact community set |
| 47 | Set action after community match | Router | `set local-preference <value>` or `set weight <value>` | Community match drives routing policy |
| 48 | Add permissive fallback sequence | Router | `route-map <match-rm-name> permit 20` | Unmatched routes are not accidentally denied |
| 49 | Apply community match route map | Router | `neighbor <neighbor-ip> route-map <match-rm-name> in` | Routes are acted on based on community |
| 50 | Verify community-list match | Router | `show ip community-list <comm-list-name>` | Community list exists |
| 51 | Verify routes by community | Router | `show bgp ipv4 unicast community <community>` | Routes carrying that community are displayed |
| 52 | Verify no-advertise behavior | Router | `show bgp ipv4 unicast community no-advertise` | Routes with `no-advertise` appear locally |
| 53 | Verify no route advertisement for no-advertise | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | `no-advertise` route is absent from advertisements |
| 54 | Verify no-export behavior | External Peer | `show bgp ipv4 unicast <prefix>` | External peer outside boundary does not receive route |
| 55 | Verify local-AS behavior | Peer Router | `show bgp ipv4 unicast community local-AS` | Local-AS community is visible within allowed scope |
| 56 | Verify route install after community policy | Router | `show ip route <prefix>` | Route installs if selected and valid |
| 57 | Verify best path reason if community action changed attributes | Router | `show bgp ipv4 unicast <prefix> best-path-reason` | Best-path reason reflects set attributes if supported |
| 58 | Save configuration | Router | `copy running-config startup-config` | Community policy is saved |
# BGP_Communities_Send_Community_Skeleton
configure terminal
ip bgp-community new-format
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> send-community
 exit-address-family
end
copy running-config startup-config
# BGP_Communities_Set_Private_Community_Inbound_Skeleton
configure terminal
ip bgp-community new-format
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <set-rm-name> permit 10
 match ip address prefix-list <pl-name>
 set community <asn>:<value> additive
route-map <set-rm-name> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <set-rm-name> in
  neighbor <neighbor-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Communities_Set_Private_Community_Outbound_Skeleton
configure terminal
ip bgp-community new-format
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <set-rm-name> permit 10
 match ip address prefix-list <pl-name>
 set community <asn>:<value> additive
route-map <set-rm-name> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <set-rm-name> out
  neighbor <neighbor-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Communities_No_Advertise_Skeleton
configure terminal
ip bgp-community new-format
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <no-advertise-rm> permit 10
 match ip address prefix-list <pl-name>
 set community no-advertise
route-map <no-advertise-rm> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <no-advertise-rm> in
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Communities_No_Export_Skeleton
configure terminal
ip bgp-community new-format
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <no-export-rm> permit 10
 match ip address prefix-list <pl-name>
 set community no-export
route-map <no-export-rm> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <no-export-rm> in
  neighbor <neighbor-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Communities_Local_AS_Skeleton
configure terminal
ip bgp-community new-format
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <local-as-rm> permit 10
 match ip address prefix-list <pl-name>
 set community local-AS
route-map <local-as-rm> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <local-as-rm> in
  neighbor <neighbor-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Communities_Match_Community_Skeleton
configure terminal
ip bgp-community new-format
ip community-list standard <comm-list-name> permit <asn>:<value>
route-map <match-rm-name> permit 10
 match community <comm-list-name>
 set local-preference <value>
route-map <match-rm-name> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <match-rm-name> in
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Communities_Expanded_Community_List_Skeleton
configure terminal
ip bgp-community new-format
ip community-list expanded <comm-list-name> permit <regex>
route-map <match-rm-name> permit 10
 match community <comm-list-name>
 set weight <value>
route-map <match-rm-name> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <match-rm-name> in
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Communities_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast community <asn>:<value>
show bgp ipv4 unicast community no-advertise
show bgp ipv4 unicast community no-export
show bgp ipv4 unicast community local-AS
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show route-map <rm-name>
show ip community-list <comm-list-name>
show running-config | section router bgp
show running-config | section route-map
show running-config | include ip community-list|bgp-community
# BGP_Communities_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP session | `show bgp ipv4 unicast summary` | Neighbor is established |
| Verify community display format | `show running-config | include bgp-community` | `ip bgp-community new-format` is configured if desired |
| Verify target prefix | `show bgp ipv4 unicast <prefix>` | Prefix appears in BGP table |
| Verify community on prefix | `show bgp ipv4 unicast <prefix>` | Prefix shows expected community |
| Verify routes by private community | `show bgp ipv4 unicast community <asn>:<value>` | Routes carrying that community appear |
| Verify no-advertise routes | `show bgp ipv4 unicast community no-advertise` | Locally tagged no-advertise routes appear |
| Verify no-export routes | `show bgp ipv4 unicast community no-export` | Locally tagged no-export routes appear |
| Verify local-AS routes | `show bgp ipv4 unicast community local-AS` | Locally tagged local-AS routes appear |
| Verify BGP config | `show running-config | section router bgp` | Route-map, send-community, and AFI config are visible |
| Verify route-map config | `show running-config | section route-map` | Match and set community logic is correct |
| Verify route-map hits | `show route-map <rm-name>` | Route-map counters increment |
| Verify community list config | `show running-config | include ip community-list` | Community list exists with correct match |
| Verify community list object | `show ip community-list <comm-list-name>` | Community list is visible |
| Verify advertised routes | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Routes are advertised or suppressed as expected |
| Verify peer received community | `show bgp ipv4 unicast <prefix>` on peer | Peer sees community only if `send-community` is configured |
| Verify no-advertise suppression | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Prefix with no-advertise is not advertised to any peer |
| Verify no-export suppression | External peer `show bgp ipv4 unicast <prefix>` | Prefix is not received outside AS/confed boundary |
| Verify local-AS suppression | Peer `show bgp ipv4 unicast <prefix>` | Prefix stays inside expected local AS/sub-AS scope |
| Verify route install | `show ip route <prefix>` | Route installs if selected and valid |
| Verify best path if community changed attributes | `show bgp ipv4 unicast <prefix> best-path-reason` | Best-path reason reflects policy result if supported |
# BGP_Communities_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify BGP community config | Router | `show running-config | section router bgp` | Neighbor `send-community` and route-map attachment are visible |
| 2 | Identify route maps | Router | `show running-config | section route-map` | Community set/match route maps are visible |
| 3 | Identify community lists | Router | `show running-config | include ip community-list` | Community lists are visible |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 7 | Remove inbound route-map | Router | `no neighbor <neighbor-ip> route-map <rm-name> in` | Inbound community policy is detached |
| 8 | Remove outbound route-map | Router | `no neighbor <neighbor-ip> route-map <rm-name> out` | Outbound community policy is detached |
| 9 | Remove send-community only if lab-only | Router | `no neighbor <neighbor-ip> send-community` | Communities are no longer sent to peer |
| 10 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 11 | Remove route-map sequence | Router | `no route-map <rm-name> permit <seq>` | Route-map sequence is removed |
| 12 | Remove fallback route-map sequence if lab-only | Router | `no route-map <rm-name> permit 20` | Fallback sequence is removed |
| 13 | Remove standard community list | Router | `no ip community-list standard <comm-list-name>` | Standard community list is removed |
| 14 | Remove expanded community list | Router | `no ip community-list expanded <comm-list-name>` | Expanded community list is removed |
| 15 | Remove prefix list if lab-only | Router | `no ip prefix-list <pl-name>` | Prefix list is removed |
| 16 | Remove readable community format only if lab-only | Router | `no ip bgp-community new-format` | Community display returns to default decimal format |
| 17 | Soft clear inbound if inbound policy changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are reprocessed |
| 18 | Soft clear outbound if outbound policy changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound routes are re-advertised |
| 19 | Verify community rollback | Router | `show bgp ipv4 unicast <prefix>` | Prefix no longer shows removed community policy |
| 20 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Communities_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Peer does not see community | `show running-config | section router bgp` | Missing `neighbor <ip> send-community` |
| Community is visible locally but not remotely | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Community was not sent or route was not advertised |
| Existing communities disappear | `show bgp ipv4 unicast <prefix>` | `set community` used without `additive` |
| New community does not appear | `show route-map <rm-name>` | Route map not hit, wrong prefix list, wrong direction, or no soft clear |
| Route disappears after applying route map | `show route-map <rm-name>` | Missing permissive fallback sequence caused unmatched routes to be denied |
| `no-advertise` route is not advertised anywhere | `show bgp ipv4 unicast community no-advertise` | Expected behavior |
| Route with `no-advertise` should have gone to peer | `show bgp ipv4 unicast <prefix>` | Wrong community was set |
| `no-export` still appears inside AS | `show bgp ipv4 unicast community no-export` | Expected behavior; no-export blocks advertisement outside AS/confed boundary |
| `local-AS` blocks more than expected | `show bgp ipv4 unicast community local-AS` | local-AS scope is narrower than no-export, especially with confederations |
| Community list does not match | `show running-config | include ip community-list` | Wrong community value, wrong list type, or exact-match mismatch |
| Multiple communities on one list line fail to match | `show ip community-list <comm-list-name>` | All communities on that single statement must exist on the route |
| Expanded community list fails | `show running-config | include ip community-list expanded` | Regex pattern is wrong |
| Policy attached but no behavior change | `show running-config | section router bgp` | Route map applied to wrong neighbor, wrong AFI, or wrong direction |
| Community appears in decimal instead of `<asn>:<value>` | `show running-config | include bgp-community` | Missing `ip bgp-community new-format` |
| Inbound change does not take effect | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Routes were not refreshed inbound |
| Outbound change does not take effect | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Routes were not refreshed outbound |
| Community policy changes best path unexpectedly | `show bgp ipv4 unicast <prefix> best-path-reason` | Community match set weight/local-preference/MED or another attribute |
| Provider community does nothing | Provider documentation and `show bgp ipv4 unicast <prefix>` | Community value is not one the upstream provider honors |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used instead of soft policy refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-community-no-advertise-final` | `BGP_Communities.md` | Primary lab for well-known `no-advertise` community and advertisement suppression behavior |
| `bgp-community-local-as-final` | `BGP_Communities.md` | Primary lab for local-AS community scope and local/confederation-style advertisement control |
# Index
# Source_Basis
# BGP_Communities_Mental_Model
# BGP_Communities_Configuration_Checklist
# BGP_Communities_Send_Community_Skeleton
# BGP_Communities_Set_Private_Community_Inbound_Skeleton
# BGP_Communities_Set_Private_Community_Outbound_Skeleton
# BGP_Communities_No_Advertise_Skeleton
# BGP_Communities_No_Export_Skeleton
# BGP_Communities_Local_AS_Skeleton
# BGP_Communities_Match_Community_Skeleton
# BGP_Communities_Expanded_Community_List_Skeleton
# BGP_Communities_Verification_Skeleton
# BGP_Communities_Verification_Commands
# BGP_Communities_Rollback
# BGP_Communities_Failure_Checks
# Attached_Labs

Blunt rule: community policy has two separate jobs: tag routes and act on tagged routes. If you tag routes but forget send-community, peers will not see the tag. If you set a community without additive, you may wipe existing tags and break upstream policy.
