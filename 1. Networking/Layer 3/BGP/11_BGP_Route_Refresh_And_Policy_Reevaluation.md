
BGP_Route_Refresh_And_Policy_Reevaluation.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP soft reset, route refresh, policy control, and scalable BGP behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP clear, route refresh, route-map, and neighbor verification syntax |
| `All_combined_part3.md` lines 63521-63539 | BGP soft reconfiguration overview | Supports soft clear as a non-disruptive alternative to hard BGP session reset after policy changes |
| `All_combined_part3.md` lines 63539-63567 | Inbound soft reconfiguration | Supports `neighbor <address-or-peer-group> soft-reconfiguration inbound` and explains memory cost |
| `All_combined_part3.md` lines 63569-63597 | Route refresh feature | Supports route refresh as replacement for soft reconfiguration and verifies capability with neighbor output |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25823-25827 | Hard reset, soft reset, and route refresh | Supports `clear ip bgp`, `clear bgp <afi> <safi> <neighbor|*> soft [in|out]`, inbound/outbound policy refresh, and route refresh per address family |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26635-26637 | Route refresh command reference | Supports `clear bgp <afi> <safi> {<ip-address>|*} soft [in|out]` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22958-22967 | Adj-RIB-Out verification | Supports `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 29258-29331 | Routes vs advertised-routes troubleshooting | Supports checking received usable routes after filters and advertised routes before assuming a peer-side issue |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 30395-30415 | Soft outbound refresh after policy change | Supports `clear bgp ipv4 unicast * soft out` followed by advertised-routes verification |
| `iosxe_combined_pdfs_.md` lines 33573-33581 | IOS XE soft outbound clear | Supports `clear ip bgp {*|address|peer-group-name} soft out` when route refresh is supported |
# BGP_Route_Refresh_And_Policy_Reevaluation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Hard clear | Tears down the BGP TCP session and forces full neighbor re-establishment |
| Soft clear | Reprocesses BGP policy without resetting the BGP session |
| Route refresh | Capability where one BGP router asks the peer to resend routes so inbound policy can be re-evaluated |
| Per-AFI route refresh | Route refresh is negotiated per address family |
| Inbound policy change | Policy affecting routes received from a neighbor |
| Outbound policy change | Policy affecting routes advertised to a neighbor |
| `soft in` | Reprocesses routes received from the neighbor |
| `soft out` | Rebuilds and resends routes advertised to the neighbor |
| `clear bgp ipv4 unicast <neighbor> soft in` | Preferred modern command after inbound IPv4 unicast policy changes |
| `clear bgp ipv4 unicast <neighbor> soft out` | Preferred modern command after outbound IPv4 unicast policy changes |
| `clear bgp ipv4 unicast * soft out` | Refreshes outbound advertisements to all IPv4 unicast peers |
| Inbound soft reconfiguration | Legacy method that stores unmodified received routes locally for later reprocessing |
| `soft-reconfiguration inbound` | Enables local storage of received routes but increases memory usage |
| Route refresh vs soft reconfiguration | Route refresh is preferred when supported because it avoids storing every received route locally |
| `advertised-routes` | Shows what this router is sending to a neighbor |
| `routes` | Shows routes accepted from the neighbor after inbound policy processing |
| `received-routes` | Shows raw received routes only when inbound soft reconfiguration is enabled |
| Adj-RIB-Out | Per-neighbor outbound route table after outbound policy |
| Loc-RIB | Local BGP table after inbound policy and best-path processing |
| Blunt rule | After changing BGP policy, do not hard reset the peer unless you actually need to. Use the correct soft refresh direction |
# BGP_Route_Refresh_And_Policy_Reevaluation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Neighbor shows a number under `State/PfxRcd` |
| 2 | Confirm target neighbor | Router / Notes | `<neighbor-ip>` | Peer affected by policy change is known |
| 3 | Confirm address family | Router / Notes | `ipv4 unicast` | Refresh target AFI/SAFI is known |
| 4 | Identify policy type | Router / Notes | `inbound` or `outbound` | Refresh direction is known before running clear command |
| 5 | Identify policy object being changed | Router / Notes | `route-map`, `prefix-list`, `community-list`, `as-path ACL`, `filter-list`, or `distribute-list` | Exact policy object is known |
| 6 | Review BGP neighbor config | Router | `show running-config | section router bgp` | Neighbor policy attachments are visible |
| 7 | Review route maps | Router | `show running-config | section route-map` | Route-map match and set logic is visible |
| 8 | Review prefix lists | Router | `show ip prefix-list` | Prefix-list entries and counters are visible |
| 9 | Review community lists if used | Router | `show running-config | include ip community-list` | Community match logic is visible |
| 10 | Review AS path lists if used | Router | `show running-config | include ip as-path|as-path access-list` | AS path match logic is visible |
| 11 | Verify route refresh capability | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> | include Route refresh|route refresh` | Neighbor advertises and receives route refresh capability |
| 12 | Verify current BGP table for target prefix | Router | `show bgp ipv4 unicast <prefix>` | Current attributes and best path are visible |
| 13 | Verify current route-map counters | Router | `show route-map <route-map-name>` | Baseline hit counters are known |
| 14 | Verify current advertised routes before outbound change | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Baseline outbound advertisements are known |
| 15 | Verify current accepted routes before inbound change | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> routes` | Baseline accepted inbound routes are known |
| 16 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 17 | Modify prefix list if needed | Router | `ip prefix-list <pl-name> permit <prefix>/<length>` | Prefix-list policy is updated |
| 18 | Modify route map if needed | Router | `route-map <rm-name> permit <seq>` | Route-map sequence is entered |
| 19 | Match prefix list in route map | Router | `match ip address prefix-list <pl-name>` | Route map matches intended prefixes |
| 20 | Set BGP attribute if needed | Router | `set local-preference <value>` or `set weight <value>` or `set metric <value>` | Matching routes receive intended attribute |
| 21 | Add permissive fallback if route map should not filter unmatched routes | Router | `route-map <rm-name> permit <fallback-seq>` | Unmatched routes are not accidentally denied |
| 22 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 23 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 24 | Attach route map inbound if changing received routes | Router | `neighbor <neighbor-ip> route-map <rm-name> in` | Inbound policy applies to routes received from peer |
| 25 | Attach route map outbound if changing advertised routes | Router | `neighbor <neighbor-ip> route-map <rm-name> out` | Outbound policy applies to routes sent to peer |
| 26 | Enable inbound soft reconfiguration only if route refresh is not available or raw received-routes visibility is required | Router | `neighbor <neighbor-ip> soft-reconfiguration inbound` | Router stores received routes locally, increasing memory use |
| 27 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 28 | Save policy config | Router | `copy running-config startup-config` | Policy configuration is saved |
| 29 | Refresh inbound policy for one peer | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes are re-requested or reprocessed without BGP session reset |
| 30 | Refresh outbound policy for one peer | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound Adj-RIB-Out is rebuilt and updates are sent |
| 31 | Refresh inbound policy for all IPv4 unicast peers if needed | Router | `clear bgp ipv4 unicast * soft in` | All peers refresh inbound IPv4 unicast routes |
| 32 | Refresh outbound policy for all IPv4 unicast peers if needed | Router | `clear bgp ipv4 unicast * soft out` | All peers receive updated outbound advertisements |
| 33 | Use legacy soft clear only if required by platform syntax | Router | `clear ip bgp <neighbor-ip> soft in` or `clear ip bgp <neighbor-ip> soft out` | Legacy command performs soft policy refresh |
| 34 | Avoid hard clear unless session reset is required | Router | `clear bgp ipv4 unicast <neighbor-ip>` | Hard reset is used only intentionally |
| 35 | Verify BGP session stayed up | Router | `show bgp ipv4 unicast summary` | Neighbor remains established and uptime does not reset after soft clear |
| 36 | Verify inbound policy result | Router | `show bgp ipv4 unicast <prefix>` | Prefix attributes or filtering reflect inbound policy |
| 37 | Verify outbound policy result | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Advertised routes reflect outbound policy |
| 38 | Verify route-map counters after refresh | Router | `show route-map <rm-name>` | Route-map counters increment after policy refresh |
| 39 | Verify peer sees updated advertisement | Peer Router | `show bgp ipv4 unicast <prefix>` | Peer receives updated route, attribute, or withdrawal |
| 40 | Verify route install after policy change | Router | `show ip route <prefix>` | Desired BGP path installs if valid and preferred |
| 41 | Verify best-path reason if supported | Router | `show bgp ipv4 unicast <prefix> best-path-reason` | Best path reflects updated policy if relevant |
| 42 | Verify logs for no reset | Router | `show logging | include BGP|ADJCHANGE|refresh` | Logs show policy refresh behavior without unexpected neighbor reset |
# BGP_Route_Refresh_And_Policy_Reevaluation_Inbound_Soft_Refresh_Skeleton
clear bgp ipv4 unicast <neighbor-ip> soft in
# BGP_Route_Refresh_And_Policy_Reevaluation_Outbound_Soft_Refresh_Skeleton
clear bgp ipv4 unicast <neighbor-ip> soft out
# BGP_Route_Refresh_And_Policy_Reevaluation_All_Peers_Outbound_Skeleton
clear bgp ipv4 unicast * soft out
# BGP_Route_Refresh_And_Policy_Reevaluation_All_Peers_Inbound_Skeleton
clear bgp ipv4 unicast * soft in
# BGP_Route_Refresh_And_Policy_Reevaluation_Inbound_Policy_Change_Skeleton
configure terminal
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set local-preference <value>
route-map <rm-name> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <rm-name> in
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_Route_Refresh_And_Policy_Reevaluation_Outbound_Policy_Change_Skeleton
configure terminal
ip prefix-list <pl-name> permit <prefix>/<length>
route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set community <asn>:<value> additive
route-map <rm-name> permit 20
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <rm-name> out
  neighbor <neighbor-ip> send-community
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_Route_Refresh_And_Policy_Reevaluation_Soft_Reconfiguration_Inbound_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> soft-reconfiguration inbound
 exit-address-family
end
copy running-config startup-config
# BGP_Route_Refresh_And_Policy_Reevaluation_Legacy_Clear_Syntax_Skeleton
clear ip bgp <neighbor-ip> soft in
clear ip bgp <neighbor-ip> soft out
clear ip bgp * soft out
# BGP_Route_Refresh_And_Policy_Reevaluation_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast neighbors <neighbor-ip> | include Route refresh|route refresh
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast neighbors <neighbor-ip> routes
show bgp ipv4 unicast neighbors <neighbor-ip> received-routes
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show route-map <rm-name>
show ip prefix-list <pl-name>
show running-config | section router bgp
show running-config | section route-map
show logging | include BGP|ADJCHANGE|refresh
# BGP_Route_Refresh_And_Policy_Reevaluation_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify neighbor state | `show bgp ipv4 unicast summary` | Neighbor remains established |
| Verify route refresh support | `show bgp ipv4 unicast neighbors <neighbor-ip> | include Route refresh|route refresh` | Neighbor shows route refresh capability advertised/received |
| Verify full neighbor detail | `show bgp ipv4 unicast neighbors <neighbor-ip>` | Capabilities, AFI state, policy, and counters are visible |
| Verify target prefix | `show bgp ipv4 unicast <prefix>` | Prefix attributes reflect current policy |
| Verify inbound accepted routes | `show bgp ipv4 unicast neighbors <neighbor-ip> routes` | Routes accepted after inbound policy are visible |
| Verify raw received routes if soft reconfiguration inbound is enabled | `show bgp ipv4 unicast neighbors <neighbor-ip> received-routes` | Pre-policy received routes are visible |
| Verify outbound advertisements | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Routes sent to neighbor reflect outbound policy |
| Verify route-map config | `show running-config | section route-map` | Match and set logic is correct |
| Verify route-map hits | `show route-map <rm-name>` | Counters increment after soft refresh |
| Verify prefix-list config | `show ip prefix-list <pl-name>` | Prefix-list entries match intended routes |
| Verify BGP policy attachment | `show running-config | section router bgp` | Neighbor route-map, filter, prefix-list, or soft-reconfiguration config is attached in correct AFI and direction |
| Verify session uptime did not reset | `show bgp ipv4 unicast summary` | Neighbor uptime stays continuous after soft clear |
| Verify peer received outbound update | `show bgp ipv4 unicast <prefix>` on peer | Peer sees changed route, attribute, or withdrawal |
| Verify RIB install | `show ip route <prefix>` | Route installs if selected and valid |
| Verify best-path result | `show bgp ipv4 unicast <prefix> best-path-reason` | Best path reflects updated policy if supported |
| Verify logs | `show logging | include BGP|ADJCHANGE|refresh` | No unexpected hard reset occurred |
# BGP_Route_Refresh_And_Policy_Reevaluation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify BGP policy attachment | Router | `show running-config | section router bgp` | Neighbor policy and direction are visible |
| 2 | Identify route maps | Router | `show running-config | section route-map` | Route-map sequences are visible |
| 3 | Identify prefix lists | Router | `show ip prefix-list` | Prefix-list entries are visible |
| 4 | Identify community or AS path lists if used | Router | `show running-config | include community-list|as-path` | Supporting policy objects are visible |
| 5 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 6 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 7 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 8 | Remove inbound route map | Router | `no neighbor <neighbor-ip> route-map <rm-name> in` | Inbound policy is detached |
| 9 | Remove outbound route map | Router | `no neighbor <neighbor-ip> route-map <rm-name> out` | Outbound policy is detached |
| 10 | Remove inbound soft reconfiguration if lab-only | Router | `no neighbor <neighbor-ip> soft-reconfiguration inbound` | Router stops storing raw received routes |
| 11 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 12 | Remove route-map sequence if lab-only | Router | `no route-map <rm-name> permit <seq>` | Route-map sequence is removed |
| 13 | Remove fallback route-map sequence if lab-only | Router | `no route-map <rm-name> permit <fallback-seq>` | Fallback sequence is removed |
| 14 | Remove prefix list if lab-only | Router | `no ip prefix-list <pl-name>` | Prefix-list is removed |
| 15 | Remove community list if lab-only | Router | `no ip community-list standard <comm-list-name>` | Community list is removed |
| 16 | Remove AS path list if lab-only | Router | `no ip as-path access-list <list-number-or-name>` | AS path list is removed |
| 17 | Soft refresh inbound after inbound rollback | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound policy rollback takes effect without reset |
| 18 | Soft refresh outbound after outbound rollback | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound policy rollback takes effect without reset |
| 19 | Verify rollback result | Router | `show bgp ipv4 unicast <prefix>` | Prefix attributes return to expected state |
| 20 | Verify advertised routes after rollback | Router | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Outbound advertisements match rollback design |
| 21 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Route_Refresh_And_Policy_Reevaluation_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Policy change has no effect | `show running-config | section router bgp` | Route map attached to wrong neighbor, wrong AFI, or wrong direction |
| Inbound change has no effect | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound routes were not refreshed |
| Outbound change has no effect | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound Adj-RIB-Out was not rebuilt |
| Route-map counters stay zero | `show route-map <rm-name>` | Prefix list does not match, route map is not attached, or wrong direction is used |
| Prefix-list counters stay zero | `show ip prefix-list <pl-name>` | Prefix or prefix length is wrong |
| Route disappears after policy change | `show route-map <rm-name>` | Route-map lacks a permissive fallback sequence, so unmatched routes are denied |
| Neighbor reset unexpectedly | `show logging | include BGP|ADJCHANGE` | Hard clear used or config change forced a reset |
| `received-routes` command fails or shows nothing useful | `show running-config | section router bgp` | `soft-reconfiguration inbound` is not enabled |
| Memory usage climbs after enabling soft reconfiguration | `show processes memory` or platform memory check | Inbound soft reconfiguration stores received routes locally |
| Route refresh not supported by peer | `show bgp ipv4 unicast neighbors <neighbor-ip> | include Route refresh` | Peer did not negotiate route refresh capability |
| Soft inbound refresh does not retrieve filtered routes | `show bgp ipv4 unicast neighbors <neighbor-ip> received-routes` | Route refresh unsupported and soft-reconfiguration inbound not enabled |
| Peer does not see outbound attribute change | Peer `show bgp ipv4 unicast <prefix>` | Missing `soft out`, wrong outbound route map, or route not advertised |
| Local router does not see inbound attribute change | `show bgp ipv4 unicast <prefix>` | Missing `soft in`, wrong inbound route map, or peer did not resend route |
| Advertised-routes still shows old result | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Outbound refresh not run or route-map condition still unchanged |
| Route exists in BGP but not RIB | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Policy changed BGP attributes, but next-hop/RIB best path still prevents install |
| `clear bgp * soft out` affects too much | `show logging | include BGP` | Used all-peer refresh when only one neighbor needed refresh |
| Hard clear caused outage | `clear bgp ipv4 unicast <neighbor-ip>` | Used hard reset when soft refresh was enough |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-route-refresh-final` | `BGP_Route_Refresh_And_Policy_Reevaluation.md` | Primary lab for route refresh, soft inbound/outbound policy re-evaluation, advertised-routes verification, and avoiding hard BGP resets |
# Index
# Source_Basis
# BGP_Route_Refresh_And_Policy_Reevaluation_Mental_Model
# BGP_Route_Refresh_And_Policy_Reevaluation_Configuration_Checklist
# BGP_Route_Refresh_And_Policy_Reevaluation_Inbound_Soft_Refresh_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Outbound_Soft_Refresh_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_All_Peers_Outbound_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_All_Peers_Inbound_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Inbound_Policy_Change_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Outbound_Policy_Change_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Soft_Reconfiguration_Inbound_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Legacy_Clear_Syntax_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Verification_Skeleton
# BGP_Route_Refresh_And_Policy_Reevaluation_Verification_Commands
# BGP_Route_Refresh_And_Policy_Reevaluation_Rollback
# BGP_Route_Refresh_And_Policy_Reevaluation_Failure_Checks
# Attached_Labs

Blunt rule: route refresh is the clean way to make BGP policy changes take effect without bouncing the session. Use soft in after inbound policy changes, soft out after outbound policy changes, and do not hard clear a BGP neighbor unless you are willing to reset the session.