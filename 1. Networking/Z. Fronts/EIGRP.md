Here’s the proper note order. Do not make one note per lab. That fragments the learning. Make mechanism notes, then attach the labs as reps under the correct mechanism.

Source basis: _KB_INDEX.md points EIGRP depth to All_combined_part3.md, Book 3: EIGRP for IP, plus the ENARSI EIGRP chapters for modern configuration, named mode, filtering, summarization, redistribution, and troubleshooting. I’d only use outside Cisco docs for FRR, OTP, graceful shutdown, and startup-mode split-horizon behavior where the project source is thinner.

01_EIGRP_Foundation_Classic_Mode.md

Mental model: EIGRP is first an interface, AS number, and neighbor-adjacency problem. If the correct interfaces are not participating and neighbors are not forming, routes are irrelevant.

Verification anchor:  
show ip eigrp interfaces  
show ip eigrp neighbors  
show ip protocols  
show ip route eigrp

Related labs:

- eigrp-two-routers-final
- eigrp-basic-final
- eigrp-four-routers-final

02_EIGRP_Named_Mode_Address_Families.md

Mental model: Named mode does not change EIGRP logic. It changes configuration structure. The knobs move under address-family, af-interface, and topology base.

Verification anchor:  
show running-config | section router eigrp  
show ip eigrp neighbors  
show ip protocols

Related labs:

- eigrp-named-mode-final

03_EIGRP_Topology_Table_Successor_And_RIB_Selection.md

Mental model: The EIGRP topology table is not the routing table. EIGRP may know multiple candidates, but only the best valid route becomes the successor and only the winning administrative-distance route enters the RIB.

Verification anchor:  
show ip eigrp topology  
show ip route eigrp  
show ip route <prefix>

Related labs:

- eigrp-basic-final
- eigrp-four-routers-final

04_EIGRP_Feasible_Successor_And_Feasibility_Condition.md

Mental model: A backup path is not automatically safe. A feasible successor must pass the feasibility condition: the neighbor’s reported distance must be lower than the current successor’s feasible distance.

Verification anchor:  
show ip eigrp topology  
show ip eigrp topology <prefix>  
show ip route <prefix>

Related labs:

- eigrp-feasible-successor-final
- eigrp-feasible-successor-rule-final

05_EIGRP_DUAL_FSM_Query_Reply_And_SIA.md

Mental model: Passive means stable. Active means EIGRP lost its safe path and is running DUAL query logic. Query boundaries, stubs, summaries, and filters control how far the blast radius spreads.

Verification anchor:  
show ip eigrp topology  
show ip eigrp topology active  
show ip eigrp events  
show ip eigrp neighbors

Related labs:

- eigrp-dual-fsm-final
- eigrp-troubleshooting-final

06_EIGRP_Default_Route_Injection_Default_Network.md

Mental model: A default route is reachability policy. ip default-network marks a candidate default network for EIGRP advertisement, but you still verify whether downstream routers actually install a gateway of last resort.

Verification anchor:  
show ip route  
show ip protocols  
show ip eigrp topology  
show running-config | include default-network

Related labs:

- eigrp-default-network-final

07_EIGRP_Filtering_With_Distribute_Lists.md

Mental model: Filtering is route control. Inbound filtering stops routes before they enter local EIGRP decision-making. Outbound filtering stops known routes from being advertised to neighbors.

Verification anchor:  
show ip protocols  
show access-lists  
show ip route eigrp  
show ip eigrp topology

Related labs:

- eigrp-distribute-list-filtering-final

08_EIGRP_Filtering_With_Route_Maps.md

Mental model: Route-map filtering is policy filtering. Use it when plain ACL or prefix filtering is too crude and you need match logic such as prefix-list, tag, interface, or next-hop.

Verification anchor:  
show route-map  
show ip prefix-list  
show ip protocols  
show ip route eigrp

Related labs:

- eigrp-route-map-filtering-final

09_EIGRP_Summarization_And_Summary_Leak_Map.md

Mental model: Summarization hides specifics and creates a local Null0 discard route for loop prevention. A leak-map is a deliberate exception that allows selected specifics to escape the summary.

Verification anchor:  
show ip route eigrp  
show ip eigrp topology  
show running-config | include summary-address|leak-map  
show ip route <specific-prefix>

Related labs:

- eigrp-summary-leak-map-final

10_EIGRP_Stub_And_Stub_Leak_Map.md

Mental model: Stub makes a router a query boundary. A stub router should not be treated as transit. Stub options determine what it may advertise, and a leak-map allows controlled exceptions.

Verification anchor:  
show ip protocols  
show ip eigrp neighbors detail  
show ip route eigrp  
show ip eigrp topology active

Related labs:

- eigrp-stub-final
- eigrp-stub-leak-map-final

11_EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse.md

Mental model: Split horizon prevents a route from being advertised back out the interface it was learned on. Poison reverse is used in special EIGRP behaviors such as startup exchange, topology changes, and queries. In hub-and-spoke designs, default split horizon can block spoke reachability if the hub learns and must re-advertise routes over the same interface. Cisco describes split horizon as preventing reverse advertisement and notes split horizon or poison reverse behavior during startup mode, topology changes, and queries.  

Verification anchor:  
show ip eigrp interfaces detail  
show running-config interface <interface>  
show ip route eigrp  
show ip eigrp topology

Related labs:

- eigrp-split-horizon-startup-mode-final

12_EIGRP_Authentication_HMAC_SHA_256.md

Mental model: Authentication is an adjacency gate. If authentication mode, key, password, or interface scope does not match, neighbors do not form. Named mode makes HMAC-SHA-256 the cleaner modern option.

Verification anchor:  
show ip eigrp neighbors  
show running-config | section router eigrp  
show running-config interface <interface>  
debug eigrp packets

Related labs:

- eigrp-sha-authentication-final

13_EIGRP_Redistribution_With_OSPF_And_RIP.md

Mental model: Redistribution is not just “make routes appear.” It creates a border between routing domains. Seed metrics, AD, tags, external route type, and loop prevention decide whether the design is safe.

Verification anchor:  
show ip protocols  
show ip route eigrp  
show ip route ospf  
show ip route rip  
show ip eigrp topology  
show route-map

Related labs:

- eigrp-ospf-final
- rip-eigrp-final

14_EIGRP_External_Route_Path_Selection.md

Mental model: External EIGRP routes are not normal internal EIGRP routes with a different label. They carry external attributes and use AD 170 by default, so RIB ownership, tags, seed metrics, and competing protocols matter.

Verification anchor:  
show ip route eigrp  
show ip eigrp topology <prefix>  
show ip protocols  
show route-map

Related labs:

- eigrp-external-path-selection-final

15_EIGRP_Graceful_Shutdown_And_Neighbor_Teardown.md

Mental model: Graceful shutdown is intentional neighbor teardown signaling. Do not misread it as random instability. Cisco’s troubleshooting docs identify goodbye behavior and intentional K-value signaling during EIGRP graceful shutdown events.  

Verification anchor:  
show ip eigrp neighbors  
show logging | include EIGRP|DUAL|Goodbye  
show ip protocols  
show running-config | section router eigrp

Related labs:

- eigrp-graceful-shutdown-final

16_EIGRP_LFA_Fast_Reroute_Per_Prefix.md

Mental model: EIGRP FRR precomputes repair paths before failure. The goal is to shift traffic to a backup path faster than normal convergence. Cisco describes EIGRP LFA FRR as precomputing repair paths and installing backup routes into the RIB for fast transition after a failure.  

Verification anchor:  
show ip eigrp topology frr  
show ip eigrp topology  
show ip route <prefix>  
show running-config | include fast-reroute

Related labs:

- eigrp-fast-reroute-final

17_EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint.md

Mental model: Disjointness is failure-domain selection, not ordinary metric tuning. Interface-disjoint tries to avoid the protected outgoing interface. SRLG-disjoint tries to avoid the same shared-risk group. Cisco documents EIGRP FRR tie-breakers including interface-disjoint and SRLG-disjoint.  

Verification anchor:  
show ip eigrp topology frr  
show running-config | include fast-reroute|tie-break|srlg  
show ip route <prefix>

Related labs:

- eigrp-fast-reroute-interface-disjoint-final
- eigrp-fast-reroute-srlg-disjoint-final

18_EIGRP_Over_The_Top_OTP.md

Mental model: OTP separates the EIGRP control plane from the WAN underlay. EIGRP carries routing information between sites, while LISP handles data-plane encapsulation across the transport. Cisco describes EIGRP OTP as enabling a single end-to-end EIGRP routing domain over public or private WAN while using EIGRP control plane and LISP data plane behavior.  

Verification anchor:  
show ip eigrp neighbors  
show ip route eigrp  
show ip lisp  
show running-config | section router eigrp

Related labs:

- eigrp-over-the-top-final

19_EIGRP_Troubleshooting_Workflow.md

Mental model: Troubleshoot EIGRP in order: physical/interface, IP reachability, AS/network participation, neighbor adjacency, timers/K-values/authentication, topology table, RIB winner, filters, summarization, redistribution, then data-plane forwarding. Cisco’s EIGRP troubleshooting guidance emphasizes checking adjacency state, common neighbor failures, active routes, and stuck-in-active behavior before chasing random config lines.  

Verification anchor:  
show ip interface brief  
show ip eigrp interfaces  
show ip eigrp neighbors  
show ip protocols  
show ip eigrp topology  
show ip route  
show logging | include EIGRP|DUAL|SIA

Related labs:

- eigrp-troubleshooting-final

Final order is:

1. 01_EIGRP_Foundation_Classic_Mode.md
2. 02_EIGRP_Named_Mode_Address_Families.md
3. 03_EIGRP_Topology_Table_Successor_And_RIB_Selection.md
4. 04_EIGRP_Feasible_Successor_And_Feasibility_Condition.md
5. 05_EIGRP_DUAL_FSM_Query_Reply_And_SIA.md
6. 06_EIGRP_Default_Route_Injection_Default_Network.md
7. 07_EIGRP_Filtering_With_Distribute_Lists.md
8. 08_EIGRP_Filtering_With_Route_Maps.md
9. 09_EIGRP_Summarization_And_Summary_Leak_Map.md
10. 10_EIGRP_Stub_And_Stub_Leak_Map.md
11. 11_EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse.md
12. 12_EIGRP_Authentication_HMAC_SHA_256.md
13. 13_EIGRP_Redistribution_With_OSPF_And_RIP.md
14. 14_EIGRP_External_Route_Path_Selection.md
15. 15_EIGRP_Graceful_Shutdown_And_Neighbor_Teardown.md
16. 16_EIGRP_LFA_Fast_Reroute_Per_Prefix.md
17. 17_EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint.md
18. 18_EIGRP_Over_The_Top_OTP.md
19. 19_EIGRP_Troubleshooting_Workflow.md