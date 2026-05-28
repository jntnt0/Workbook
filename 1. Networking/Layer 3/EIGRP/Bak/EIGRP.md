

# EIGRP_Troubleshooting_Workflow_Mental_Model

| Concept | Operational Meaning |

|---|---|

| Troubleshooting order | Work from interface/IP baseline, then EIGRP process, neighbor adjacency, topology table, RIB, policy, redistribution, and data plane |

| Interface baseline | EIGRP cannot work if the interface is down, misaddressed, passive, or not matched by a network statement |

| EIGRP process | Classic mode uses `router eigrp <AS>`; named mode uses `router eigrp <PROCESS_NAME>` plus address-family AS |

| AS number | Must match between routers that should become neighbors |

| Network statement | Enables EIGRP on matching interfaces and advertises those connected networks |

| Passive interface | Advertises a connected network but prevents EIGRP neighbor formation on that interface |

| Neighbor table | First real control-plane checkpoint; no neighbor means no EIGRP route exchange |

| Topology table | EIGRP’s route-decision database; a route may exist here without entering the routing table |

| Routing table | Actual RIB winner used to build forwarding behavior |

| Successor | Current best EIGRP path for a prefix |

| Feasible successor | Loop-free backup path that can be used without going active |

| Feasibility condition | Backup path is feasible only when alternate RD is lower than current successor FD |

| Active route | EIGRP is querying because it lost a path and lacks an immediately usable feasible successor |

| SIA | Stuck in active means query/reply logic failed to complete in time |

| Filtering | Distribute-lists and route-maps can remove routes from topology or RIB visibility |

| Summarization | Can hide component routes and install a local Null0 summary route |

| Stub routing | Reduces query scope but can suppress transit routes if configured on the wrong router |

| Redistribution | Requires seed metrics, route tags, and loop prevention when crossing protocol domains |

| Authentication | Neighbor gate; mismatch prevents adjacency |

| Administrative distance | Decides RIB ownership when EIGRP competes with static, OSPF, RIP, BGP, or another source |

| Data-plane validation | Ping and traceroute prove forwarding; EIGRP control-plane success alone is not enough |

| Hard rule | Do not edit random EIGRP config lines until you know whether the failure is adjacency, route selection, policy, redistribution, or forwarding |

# EIGRP_Troubleshooting_Workflow_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Confirm the reported failure from the source side | Source router or host | `ping <REMOTE_IP>` | Ping fails or succeeds in a way that matches the reported issue |

| 2 | Confirm the path symptom | Source router or host | `traceroute <REMOTE_IP>` | Trace shows where forwarding stops or loops |

| 3 | Confirm local interface state | All involved routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with expected IPv4 addresses |

| 4 | Confirm the destination-side interface state | Destination-side router | `show ip interface brief` | Destination network interface is `up/up` |

| 5 | Confirm directly connected routes exist | All involved routers | `show ip route connected` | Transit and local LAN prefixes exist as connected routes |

| 6 | Confirm basic next-hop reachability on each adjacent link | Neighboring routers | `ping <NEIGHBOR_INTERFACE_IP>` | Directly connected neighbors are reachable |

| 7 | Confirm CEF forwarding is enabled | All involved routers | `show ip cef` | CEF is running and has entries |

| 8 | Confirm the EIGRP configuration mode | All EIGRP routers | `show running-config | section router eigrp` | Classic or named-mode EIGRP configuration is visible |

| 9 | Confirm the EIGRP AS number | All EIGRP routers | `show ip protocols` | Routers that should peer use the same EIGRP AS |

| 10 | Confirm router ID values | All EIGRP routers | `show ip protocols` | Router IDs are stable and unique |

| 11 | Confirm EIGRP network statements match the intended interfaces | All EIGRP routers | `show ip protocols` | Network statements cover the expected EIGRP-facing interfaces |

| 12 | Confirm EIGRP-enabled interfaces | All EIGRP routers | `show ip eigrp interfaces` | Expected EIGRP interfaces appear |

| 13 | Confirm detailed EIGRP interface state | All EIGRP routers | `show ip eigrp interfaces detail` | Timers, split horizon, authentication, and peer counts match the design |

| 14 | Confirm passive-interface behavior | All EIGRP routers | `show ip protocols` | Router-facing links are not passive; LAN-only interfaces may be passive |

| 15 | Confirm neighbor adjacency state | All EIGRP routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |

| 16 | If a neighbor is missing, check local and remote AS mismatch | Both neighbors | `show ip protocols` | AS numbers match between the two routers |

| 17 | If a neighbor is missing, check passive interface | Both neighbors | `show ip protocols` | Neighbor-facing interface is not listed as passive |

| 18 | If a neighbor is missing, check K-value mismatch | Both neighbors | `show ip protocols | include K` | K-values match on both sides |

| 19 | If a neighbor is missing, check authentication state | Both neighbors | `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Authentication method and state match on both sides |

| 20 | If a neighbor is missing, check logs | Both neighbors | `show logging | include EIGRP|DUAL|NBRCHANGE|auth|K-value|hold|retry` | Logs identify adjacency reset reason |

| 21 | Confirm all expected EIGRP routes in the topology table | All involved routers | `show ip eigrp topology` | Expected prefixes appear in passive state |

| 22 | Inspect the specific missing or wrong prefix | Problem router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Output shows successor, FD, RD, and route state if the prefix exists |

| 23 | Show all known EIGRP paths for the prefix | Problem router | `show ip eigrp topology all-links` | Alternate paths appear, including paths that failed feasibility condition |

| 24 | Confirm route is not active | Problem router | `show ip eigrp topology active` | No active route remains for the prefix |

| 25 | If route is active, identify the neighbor still owing a reply | Problem router | `show ip eigrp topology active` | Reply-status neighbor is visible |

| 26 | Walk the active query chain downstream | Downstream routers | `show ip eigrp topology active` | Query chain leads to the router/link causing delay |

| 27 | Confirm the route entered the routing table | Problem router | `show ip route <PREFIX>` | Prefix appears with expected source, AD, metric, next hop, and interface |

| 28 | Confirm EIGRP-installed routes | Problem router | `show ip route eigrp` | Internal routes appear as `D`; external routes appear as `D EX` |

| 29 | If topology has the route but RIB does not, check AD competition | Problem router | `show ip route <PREFIX>` | Another source may own the route due to lower AD |

| 30 | Compare exact route ownership | Problem router | `show ip route <PREFIX>` | Static, connected, OSPF, RIP, BGP, or EIGRP ownership is confirmed |

| 31 | Confirm EIGRP metric path selection | Problem router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Successor has the lowest valid EIGRP metric |

| 32 | Check bandwidth and delay if EIGRP metric looks wrong | Problem router | `show interfaces <INTERFACE>` | Bandwidth and delay match the intended lab design |

| 33 | Check for accidental metric tuning | Problem router | `show running-config interface <INTERFACE>` | No unintended `bandwidth` or `delay` command exists |

| 34 | Check distribute-list filters | Filtering router | `show running-config | section router eigrp` | Any `distribute-list` is identified |

| 35 | Verify distribute-list match object | Filtering router | `show ip prefix-list` or `show access-lists` | Prefix-list or ACL permits and denies match intended routes |

| 36 | Check route-map filters | Filtering router | `show route-map` | Route-map has expected match logic and final permit if needed |

| 37 | Check summary configuration | Summarizing router | `show running-config interface <INTERFACE>` | `ip summary-address eigrp` is present only where intended |

| 38 | Check local Null0 summary route | Summarizing router | `show ip route <SUMMARY_PREFIX>` | Summary route points to `Null0` if summarization is active |

| 39 | Check leaked summary components | Receiving router | `show ip route <LEAKED_PREFIX>` | Leaked specific appears where intended |

| 40 | Check stub status | Upstream router | `show ip eigrp neighbors detail` | Stub neighbors are identified if configured |

| 41 | Confirm stub router is not being used as transit unintentionally | Stub router and upstream router | `show ip route eigrp` | Routes behind a stub are not accidentally suppressed |

| 42 | Check default route behavior | Downstream routers | `show ip route` | Gateway of last resort is set only if intended |

| 43 | Check default candidate behavior | Downstream routers | `show ip route` | Default candidate route appears with `*` if `ip default-network` is used |

| 44 | Check redistribution source | Boundary router | `show ip protocols` | Redistributed protocols, metrics, and route-maps are visible |

| 45 | Verify EIGRP external routes | EIGRP routers | `show ip route eigrp` | Redistributed routes appear as `D EX` |

| 46 | Confirm EIGRP seed metric for redistributed routes | Boundary router | `show running-config | section router eigrp` | Redistribution into EIGRP includes metric values or route-map metric setting |

| 47 | Confirm OSPF redistribution uses `subnets` if involved | Boundary router | `show running-config | section router ospf` | `redistribute eigrp <AS> subnets` appears if EIGRP enters OSPF |

| 48 | Check route tags for loop prevention | Boundary router and downstream routers | `show ip route <PREFIX>` | Redistributed route shows expected tag where platform displays it |

| 49 | Check redistribution route-maps | Boundary router | `show route-map` | Deny return-tag logic and set-tag logic are correct |

| 50 | Check for redistribution loops | Multiple routers | `traceroute <DESTINATION_IP> numeric` | Trace does not repeat the same router sequence |

| 51 | Check split horizon state for hub-and-spoke designs | Hub router | `show ip eigrp interfaces detail <HUB_INTERFACE>` | Split horizon state matches design requirement |

| 52 | Check named-mode hierarchy if commands appear missing | Named-mode routers | `show running-config | section router eigrp` | Settings are under address-family, af-interface, or topology base as required |

| 53 | Check for graceful shutdown versus real instability | Affected routers | `show logging | include Goodbye|PEER|hold|retry|SIA|NBRCHANGE` | Planned teardown differs from hold timer, retry limit, or SIA failure |

| 54 | Check data-plane forwarding entry | Problem router | `show ip cef <PREFIX>` | CEF next hop matches the RIB path |

| 55 | Check exact next-hop reachability | Problem router | `ping <NEXT_HOP_IP>` | Next hop is reachable |

| 56 | Check reverse route from destination side | Destination-side router | `show ip route <SOURCE_PREFIX>` | Return path exists |

| 57 | Test source-specific reachability | Source router | `ping <REMOTE_IP> source <SOURCE_INTERFACE>` | Ping result matches expected routing path |

| 58 | Trace source-specific forwarding path | Source router | `traceroute <REMOTE_IP> source <SOURCE_INTERFACE>` | Trace follows expected EIGRP path |

| 59 | Check logs after changes | All affected routers | `show logging | include EIGRP|DUAL|NBRCHANGE|SIA|auth|K-value` | No recurring instability remains |

| 60 | Save only after control plane and data plane are both fixed | Changed routers | `write memory` | Final working configuration is saved |

# EIGRP_Troubleshooting_Workflow_Skeleton

! 1. Prove the symptom

ping <REMOTE_IP>

traceroute <REMOTE_IP>

  

! 2. Interface and connected-route baseline

show ip interface brief

show ip route connected

show ip cef

  

! 3. EIGRP process and interface participation

show running-config | section router eigrp

show ip protocols

show ip eigrp interfaces

show ip eigrp interfaces detail

  

! 4. Neighbor health

show ip eigrp neighbors

show ip eigrp neighbors detail

show logging | include EIGRP|DUAL|NBRCHANGE|auth|K-value|hold|retry

  

! 5. Topology table inspection

show ip eigrp topology

show ip eigrp topology <PREFIX>/<LENGTH>

show ip eigrp topology all-links

show ip eigrp topology active

  

! 6. RIB selection

show ip route eigrp

show ip route <PREFIX>

show ip protocols

  

! 7. Metric validation

show interfaces <INTERFACE>

show running-config interface <INTERFACE>

show ip eigrp topology <PREFIX>/<LENGTH>

  

! 8. Filtering and policy validation

show running-config | section router eigrp

show ip prefix-list

show access-lists

show route-map

show ip route <FILTERED_PREFIX>

show ip eigrp topology <FILTERED_PREFIX>/<LENGTH>

  

! 9. Summarization validation

show running-config interface <SUMMARY_INTERFACE>

show ip route <SUMMARY_PREFIX>

show ip route <COMPONENT_PREFIX>

show ip eigrp topology <SUMMARY_PREFIX>/<LENGTH>

  

! 10. Stub validation

show ip protocols

show ip eigrp neighbors detail

show ip route eigrp

show ip eigrp topology active

  

! 11. Default route validation

show ip route

show ip route 0.0.0.0

show ip eigrp topology <DEFAULT_OR_CANDIDATE_PREFIX>

  

! 12. Redistribution validation

show ip protocols

show running-config | section router eigrp

show running-config | section router ospf

show running-config | section router rip

show route-map

show ip route eigrp

show ip route ospf

show ip route rip

show ip route <REDISTRIBUTED_PREFIX>

traceroute <DESTINATION_IP> numeric

  

! 13. Split horizon and hub-and-spoke validation

show ip eigrp interfaces detail <HUB_INTERFACE>

show ip route <REMOTE_SPOKE_PREFIX>

show ip eigrp topology <REMOTE_SPOKE_PREFIX>/<LENGTH>

  

! 14. Authentication validation

show ip eigrp interfaces detail <INTERFACE> | include Auth

show running-config | section router eigrp

show logging | include auth|Auth|EIGRP

  

! 15. Data-plane and reverse-path validation

show ip cef <PREFIX>

ping <NEXT_HOP_IP>

ping <REMOTE_IP> source <SOURCE_INTERFACE>

traceroute <REMOTE_IP> source <SOURCE_INTERFACE>

show ip route <SOURCE_PREFIX>

  

! 16. Final stability

show ip eigrp neighbors

show ip eigrp topology active

show logging | include EIGRP|DUAL|NBRCHANGE|SIA|auth|K-value

write memory

# EIGRP_Troubleshooting_Workflow_Verification_Commands

| Command | Purpose | Healthy Output |

|---|---|---|

| `show ip interface brief` | Confirms interface state and addressing | EIGRP-facing interfaces are `up/up` |

| `show ip route connected` | Confirms local connected routes exist | Transit and LAN prefixes appear as connected |

| `show running-config | section router eigrp` | Confirms EIGRP process configuration | Correct classic or named-mode hierarchy appears |

| `show ip protocols` | Confirms EIGRP process details | Correct AS, router ID, network statements, passive interfaces, K-values, AD, and redistribution references |

| `show ip eigrp interfaces` | Confirms active EIGRP interfaces | Expected interfaces appear |

| `show ip eigrp interfaces detail` | Confirms interface-level EIGRP behavior | Timers, split horizon, authentication, peer count, and packet counters match design |

| `show ip eigrp neighbors` | Confirms neighbor adjacency | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |

| `show ip eigrp neighbors detail` | Confirms neighbor capabilities and stub status | Neighbor details match design |

| `show ip eigrp topology` | Confirms EIGRP topology table | Expected prefixes appear in passive state |

| `show ip eigrp topology <PREFIX>/<LENGTH>` | Troubleshoots one prefix | Shows successor, FD, RD, route state, and candidate paths |

| `show ip eigrp topology all-links` | Shows every known EIGRP path | Non-feasible paths are visible for comparison |

| `show ip eigrp topology active` | Finds active or stuck routes | Healthy steady state shows no active routes |

| `show ip route eigrp` | Confirms EIGRP RIB installation | Internal routes show `D`; external routes show `D EX` |

| `show ip route <PREFIX>` | Confirms exact RIB winner | Route source, AD, metric, next hop, interface, and tag match intent |

| `show interfaces <INTERFACE>` | Confirms metric inputs | Bandwidth and delay match design |

| `show running-config interface <INTERFACE>` | Confirms interface-level changes | Intended delay, bandwidth, summary, split horizon, or shutdown state is visible |

| `show ip prefix-list` | Confirms prefix-list policy | Permit and deny entries match intended prefixes |

| `show access-lists` | Confirms ACL policy | ACL wildcard logic matches intended prefixes |

| `show route-map` | Confirms route-map policy | Match, set, permit, deny, and counters match intended policy |

| `show ip cef <PREFIX>` | Confirms data-plane forwarding | CEF next hop matches RIB expectation |

| `show logging | include EIGRP|DUAL|NBRCHANGE|SIA|auth|K-value|hold|retry` | Finds instability reason | No recurring adjacency, SIA, authentication, K-value, hold, or retry issues remain |

| `ping <REMOTE_IP> source <SOURCE_INTERFACE>` | Confirms source-specific reachability | Ping succeeds after fix |

| `traceroute <REMOTE_IP> source <SOURCE_INTERFACE>` | Confirms forwarding path | Trace follows intended EIGRP path |

| `traceroute <DESTINATION_IP> numeric` | Finds loops or wrong redistribution path | Trace reaches destination without repeated router sequence |

# EIGRP_Troubleshooting_Workflow_Rollback

! Roll back accidental interface metric tuning

configure terminal

 interface <INTERFACE>

  no delay

  no bandwidth

 end

  

! Roll back accidental passive-interface setting

configure terminal

 router eigrp <AS_NUMBER>

  no passive-interface <INTERFACE>

 end

  

! Roll back accidental classic EIGRP network statement

configure terminal

 router eigrp <AS_NUMBER>

  no network <NETWORK> <WILDCARD_MASK>

 end

  

! Restore intended classic EIGRP network statement

configure terminal

 router eigrp <AS_NUMBER>

  network <NETWORK> <WILDCARD_MASK>

 end

  

! Roll back distribute-list filter

configure terminal

 router eigrp <AS_NUMBER>

  no distribute-list prefix <LIST_NAME> in

  no distribute-list prefix <LIST_NAME> out

  no distribute-list route-map <ROUTE_MAP_NAME> in

  no distribute-list route-map <ROUTE_MAP_NAME> out

 end

  

! Roll back classic EIGRP summarization

configure terminal

 interface <SUMMARY_INTERFACE>

  no ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>

 end

  

! Roll back classic EIGRP stub if applied to the wrong router

configure terminal

 router eigrp <AS_NUMBER>

  no eigrp stub

 end

  

! Roll back EIGRP redistribution

configure terminal

 router eigrp <AS_NUMBER>

  no redistribute ospf <OSPF_PROCESS_ID>

  no redistribute rip

  no redistribute static

  no redistribute connected

 end

  

! Roll back OSPF redistribution of EIGRP

configure terminal

 router ospf <OSPF_PROCESS_ID>

  no redistribute eigrp <AS_NUMBER>

 end

  

! Roll back RIP redistribution of EIGRP

configure terminal

 router rip

  no redistribute eigrp <AS_NUMBER>

 end

  

! Roll back named-mode topology policy if applicable

configure terminal

 router eigrp <PROCESS_NAME>

  address-family ipv4 unicast autonomous-system <AS_NUMBER>

   topology base

    no redistribute <PROTOCOL>

    no distribute-list prefix <LIST_NAME> in <INTERFACE>

    no distribute-list prefix <LIST_NAME> out <INTERFACE>

   exit-af-topology

  exit-address-family

 end

  

! Roll back named-mode interface settings if applicable

configure terminal

 router eigrp <PROCESS_NAME>

  address-family ipv4 unicast autonomous-system <AS_NUMBER>

   af-interface <INTERFACE>

    no authentication mode

    split-horizon

    next-hop-self

   exit-af-interface

  exit-address-family

 end

  

! Remove temporary policy objects if created only for troubleshooting

configure terminal

 no ip prefix-list <LIST_NAME>

 no route-map <ROUTE_MAP_NAME>

 no access-list <ACL_NUMBER>

end

  

! Stop any active debugging

undebug all

  

! Verify rollback

show running-config | section router eigrp

show ip eigrp neighbors

show ip protocols

show ip eigrp topology active

show ip route eigrp

show ip route <PREFIX>

ping <REMOTE_IP> source <SOURCE_INTERFACE>

traceroute <REMOTE_IP> source <SOURCE_INTERFACE>

  

! Save only if rollback produces the intended stable state

write memory

# EIGRP_Troubleshooting_Workflow_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |

|---|---|---|---|

| No EIGRP neighbors form | Wrong AS number | `show ip protocols` | Match the EIGRP AS on both routers |

| No EIGRP neighbors form | Interface not matched by network statement | `show ip eigrp interfaces` | Add or correct the `network` statement |

| No EIGRP neighbors form | Interface is passive | `show ip protocols` | Configure `no passive-interface <INTERFACE>` |

| No EIGRP neighbors form | K-value mismatch | `show ip protocols | include K` | Restore matching K-values |

| No EIGRP neighbors form | Authentication mismatch | `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Match authentication method and secret/key-chain |

| Neighbor forms then drops | Link instability, hold timer, retry limit, MTU, CPU, or packet loss | `show logging | include EIGRP|NBRCHANGE|hold|retry` | Fix underlying link or platform issue |

| Route missing from topology table | Prefix not advertised into EIGRP | `show ip protocols` and `show ip eigrp topology` | Add network statement, redistribute correctly, or remove filter |

| Route in topology table but not in routing table | Another protocol has better AD | `show ip route <PREFIX>` | Confirm RIB winner and tune AD or policy only if EIGRP should win |

| Route in RIB but traffic fails | Return route missing | Destination-side `show ip route <SOURCE_PREFIX>` | Fix reverse path |

| Route goes active | No feasible successor and DUAL is querying | `show ip eigrp topology active` | Identify outstanding reply neighbor and trace query chain |

| Route stuck active | Missing or delayed query reply | `show ip eigrp topology active` and `show logging | include SIA` | Fix unstable neighbor, packet loss, or excessive query scope |

| Backup path not used immediately | Backup failed feasibility condition | `show ip eigrp topology all-links` | Redesign topology or metrics to create feasible successor |

| Expected route filtered accidentally | Distribute-list or route-map blocks it | `show running-config | section router eigrp`, `show route-map`, `show ip prefix-list` | Correct filter direction, match object, or final permit |

| All routes disappear after filter | Missing final permit | `show ip prefix-list` or `show route-map` | Add final permit for allowed routes |

| Summary works but specific route missing | Component route suppressed by summary | `show running-config interface <INTERFACE>` | Use leak-map or remove summary if specifics are required |

| Traffic to unused summary space blackholes | Local Null0 summary discard route | `show ip route <SUMMARY_PREFIX>` | Tighten summary or add valid component route |

| Stub router suppresses needed routes | Stub configured on a transit router | `show ip eigrp neighbors detail` and `show ip route eigrp` | Remove stub or use leak-map/design correction |

| Default route missing | Default candidate or redistribution not configured | `show ip route` and `show ip protocols` | Correct default injection method |

| External EIGRP route missing | Redistribution missing seed metric | `show running-config | section router eigrp` | Add EIGRP seed metric or route-map `set metric` |

| Redistributed route loops | Mutual redistribution lacks tag filtering | `show route-map` and `traceroute <DESTINATION> numeric` | Set route tags and deny return tags |

| OSPF beats EIGRP external route | OSPF AD 110 beats EIGRP external AD 170 | `show ip route <PREFIX>` | Use filtering, tagging, metric design, or targeted AD control |

| Spokes cannot learn each other’s routes | Split horizon enabled on hub multipoint interface | `show ip eigrp interfaces detail <HUB_INTERFACE>` | Disable EIGRP split horizon on hub interface only |

| Named-mode command does not apply | Command placed in wrong hierarchy | `show running-config | section router eigrp` | Put command under address-family, af-interface, or topology base as required |

| Classic-mode command does not apply | Command placed under wrong mode | `show running-config | section router eigrp` | Put command under router mode or interface mode as required |

| Route appears correct but CEF path is wrong | Data-plane entry stale or different from RIB | `show ip cef <PREFIX>` | Clear route/CEF only in lab or troubleshoot platform forwarding |

| Ping fails with correct forward route | ACL, NAT, source interface, or return path issue | `ping <REMOTE_IP> source <SOURCE_INTERFACE>` and remote `show ip route <SOURCE_PREFIX>` | Fix data-plane policy or reverse route |

| Traceroute loops | Redistribution loop, summary loop, or bad next-hop design | `traceroute <DESTINATION> numeric` and `show ip route <PREFIX>` | Correct route tags, summaries, filters, or next-hop path |

| Debug overwhelms device | Debugging left enabled | `show debugging` | Run `undebug all` |

# Index

# Source_Basis

# EIGRP_Troubleshooting_Workflow_Mental_Model

# EIGRP_Troubleshooting_Workflow_Configuration_Checklist

# EIGRP_Troubleshooting_Workflow_Skeleton

# EIGRP_Troubleshooting_Workflow_Verification_Commands

# EIGRP_Troubleshooting_Workflow_Rollback

# EIGRP_Troubleshooting_Workflow_Failure_Checks