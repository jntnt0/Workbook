I used the project source first, then Cisco IOS XE docs for the exact OSPFv2 LFA and remote-LFA command forms: fast-reroute per-prefix enable prefix-priority, prefix-priority, fast-reroute per-prefix tie-break, fast-reroute keep-all-paths, ip ospf fast-reroute per-prefix candidate disable, fast-reroute per-prefix remote-lfa ... tunnel mpls-ldp, and show ip ospf fast-reroute remote-lfa tunnels.  

OSPF_Fast_Reroute_LFA_And_Remote_LFA.md

OSPF_Fast_Reroute_LFA_And_Remote_LFA

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and IOS XE OSPF material as project source files |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF | Supports OSPF baseline adjacency, LSDB, SPF, route calculation, and convergence mental model |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | OSPF path selection and convergence sections | Supports OSPF SPF, loop-free path calculation, route-type verification, and troubleshooting route installation |
| `Cisco IOS XE IP Routing Configuration Guide` | OSPFv2 Loop-Free Alternate Fast Reroute | Provides local LFA commands: `fast-reroute per-prefix enable prefix-priority`, `prefix-priority`, `fast-reroute per-prefix tie-break`, `fast-reroute keep-all-paths`, `ip ospf fast-reroute per-prefix candidate disable`, `show ip ospf fast-reroute`, `show ip ospf rib`, and `debug ip ospf fast-reroute` |
| `Cisco IOS XE IP Routing Configuration Guide` | OSPF IPv4 Remote Loop-Free Alternate IP Fast Reroute | Provides remote LFA commands: `fast-reroute per-prefix remote-lfa [area <area-id>] tunnel mpls-ldp`, `fast-reroute per-prefix remote-lfa [area <area-id>] maximum-cost <distance>`, and `show ip ospf fast-reroute remote-lfa tunnels` |
| `CiscoPress_combined_part2.md` | Segment Routing / fast reroute verification references | Supports repair-path verification style with `show ip route repair-paths <prefix> <mask>` and the PLR repair-path mental model |
| Related labs | `ospf-fast-reroute-final`, `ospf-fast-reroute-Qq1bt2-final`, `ospf-remote-fast-reroute-final` | Main labs for OSPF local LFA, repair-path selection, and remote LFA tunneling behavior |
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IP Fast Reroute | Precomputes a backup forwarding path before failure occurs |
| OSPF LFA | OSPF computes a loop-free alternate next hop for each protected prefix |
| Per-prefix protection | Repair path is calculated per destination prefix, not just per interface |
| PLR | Point of Local Repair. The router adjacent to the failed primary path that locally redirects traffic to a repair path |
| Primary path | Current best OSPF next hop installed for the prefix |
| Repair path | Precomputed alternate next hop used immediately after primary next-hop or link failure |
| Local LFA | Repair next hop is a directly connected neighbor that satisfies loop-free conditions |
| Remote LFA | Repair endpoint is not directly connected. PLR tunnels traffic to a remote repair node, commonly using MPLS-LDP |
| Remote LFA tunnel | Automatically created tunnel to the remote LFA tailend so traffic can bypass the failed component |
| Tailend router | Remote repair node where the tunnel terminates |
| Loop-free condition | Repair path must not send traffic back toward the failed primary path in a way that creates a loop |
| Node protection | Repair path bypasses the primary next-hop router, not just the failed link |
| Link protection | Repair path bypasses the protected link |
| Broadcast-interface protection | On shared media, repair path should avoid the same broadcast segment when the goal is true link protection |
| Downstream path | Repair neighbor has a lower metric to the destination than the PLR, reducing loop risk |
| Tie-breaker | Policy used when multiple repair candidates exist |
| Prefix priority | Controls whether all prefixes are protected or only selected high-priority prefixes |
| Candidate disable | Prevents an interface from being used as a repair next-hop candidate |
| Keep-all-paths | Stores all considered repair candidates for troubleshooting, but increases memory use |
| Coverage limit | Classic LFA does not protect every topology. Rings, squares, and sparse topologies may need remote LFA or segment-routing TI-LFA |
| BFD relationship | BFD speeds failure detection. LFA speeds forwarding repair after failure detection. They solve different parts of convergence |
| Restriction | OSPFv2 LFA and remote LFA are not supported on routers acting as virtual-link headends |
| Remote LFA requirement | Remote LFA requires MPLS support and tunnel termination candidates with /32 addresses advertised in the OSPF area |
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify prefixes that need fast protection | Design step | `<protected-prefix-list>` | Critical loopbacks, services, or transport destinations are known |
| 2 | Identify PLR routers | Design step | `PLR routers: <router-list>` | Routers that must compute repair paths are known |
| 3 | Identify protected areas | Design step | `Area <area-id>` | OSPF area scope for LFA or remote LFA is known |
| 4 | Confirm the router is not a virtual-link headend | PLR routers | `show ip ospf virtual-links` | Router is not relying on a virtual link as a headend |
| 5 | Confirm OSPF baseline is stable | PLR routers | `show ip ospf neighbor` | Required neighbors are `FULL` |
| 6 | Confirm OSPF route table before FRR | PLR routers | `show ip route ospf` | Primary OSPF paths are known |
| 7 | Confirm target prefix route before FRR | PLR routers | `show ip route <protected-prefix>` | Current primary next hop and outgoing interface are known |
| 8 | Confirm LSDB visibility before FRR | PLR routers | `show ip ospf database` | PLR has a complete enough topology view to compute repair candidates |
| 9 | Confirm current fast-reroute state | PLR routers | `show ip ospf fast-reroute` | Current LFA configuration is known |
| 10 | Enter global configuration mode | PLR routers | `configure terminal` | Router enters global configuration mode |
| 11 | Enter the OSPF process | PLR routers | `router ospf <process-id>` | OSPF router configuration mode opens |
| 12 | Enable per-prefix local LFA for all eligible prefixes | PLR routers | `fast-reroute per-prefix enable prefix-priority low` | Router computes repair paths for all eligible prefixes |
| 13 | Enable per-prefix local LFA only for high-priority prefixes if required | PLR routers | `fast-reroute per-prefix enable prefix-priority high` | Router protects only prefixes classified as high priority |
| 14 | Create a prefix list for high-priority protected prefixes | PLR routers | `ip prefix-list <PREFIX_LIST_NAME> seq 5 permit <prefix>/<length>` | Critical prefixes are matched for priority classification |
| 15 | Create a route map for high-priority prefixes | PLR routers | `route-map <ROUTE_MAP_NAME> permit 10` then `match ip address prefix-list <PREFIX_LIST_NAME>` | Route map matches prefixes that should receive high-priority repair |
| 16 | Attach high-priority prefix policy to OSPF | PLR routers | `prefix-priority high route-map <ROUTE_MAP_NAME>` | Selected prefixes are classified as high priority |
| 17 | Prefer node-protecting repair paths if next-hop node failure must be protected | PLR routers | `fast-reroute per-prefix tie-break node-protecting index <index>` | Node-protecting candidates are preferred according to index |
| 18 | Require node protection if the lab demands only node-protecting repairs | PLR routers | `fast-reroute per-prefix tie-break node-protecting required index <index>` | Non-node-protecting candidates are rejected |
| 19 | Prefer interface-disjoint repairs for point-to-point link protection | PLR routers | `fast-reroute per-prefix tie-break interface-disjoint index <index>` | Repair path avoids the protected primary interface when possible |
| 20 | Prefer broadcast-interface-disjoint repairs on shared media | PLR routers | `fast-reroute per-prefix tie-break broadcast-interface-disjoint index <index>` | Repair path avoids the protected broadcast segment when possible |
| 21 | Prefer downstream repair paths to reduce loop risk | PLR routers | `fast-reroute per-prefix tie-break downstream index <index>` | Downstream candidates are preferred |
| 22 | Prefer SRLG-disjoint repair paths if SRLGs are modeled | PLR routers | `fast-reroute per-prefix tie-break srlg index <index>` | Repair candidates avoid same SRLG when possible |
| 23 | Keep all candidate repair paths only for lab troubleshooting | PLR routers | `fast-reroute keep-all-paths` | Router stores considered repair candidates for review |
| 24 | Exclude a bad interface from repair candidacy | PLR routers | `interface <interface-id>` then `ip ospf fast-reroute per-prefix candidate disable` | Interface is not used as a repair next-hop candidate |
| 25 | Exit configuration mode | PLR routers | `end` | Router returns to privileged EXEC mode |
| 26 | Save configuration | PLR routers | `write memory` | Configuration is saved |
| 27 | Verify local LFA configuration | PLR routers | `show ip ospf fast-reroute` | Per-prefix LFA and tie-break policy are visible |
| 28 | Verify OSPF RIB repair information | PLR routers | `show ip ospf rib <protected-prefix>` | OSPF RIB shows primary path and repair information if supported |
| 29 | Verify routing table repair path | PLR routers | `show ip route repair-paths <protected-prefix> <mask>` | Prefix shows a repair path when coverage exists |
| 30 | Verify CEF programmed repair behavior | PLR routers | `show ip cef <protected-prefix> detail` | CEF shows primary and backup forwarding details where supported |
| 31 | Confirm normal traffic path before failure | Test router | `traceroute <protected-destination>` | Traffic follows the primary OSPF path |
| 32 | Trigger controlled failure in the lab | PLR-facing router | `interface <protected-link>` then `shutdown` | Primary next hop or link fails |
| 33 | Confirm traffic repairs quickly | Test router | `ping <protected-destination> repeat <count>` | Packet loss is minimal during failure |
| 34 | Confirm post-failure route converges | PLR routers | `show ip route <protected-prefix>` | Route eventually reflects post-convergence best path |
| 35 | Restore failed link | PLR-facing router | `interface <protected-link>` then `no shutdown` | Primary link returns |
| 36 | Confirm OSPF returns to stable state | All involved routers | `show ip ospf neighbor` | Neighbors return to `FULL` |
| 37 | Confirm remote LFA prerequisites before enabling remote LFA | PLR and candidate tailend routers | `show ip route <candidate-tailend-loopback>` | Candidate tailend loopback is reachable as a /32 in the area |
| 38 | Confirm MPLS is available for remote LFA | PLR and core routers | `show mpls interfaces` and `show mpls ldp neighbor` | MPLS forwarding and LDP adjacencies are operational |
| 39 | Configure targeted LDP acceptance on possible remote LFA tailends | Candidate tailend routers | `mpls ldp discovery targeted-hello accept` | Tailend can accept targeted LDP sessions from PLR |
| 40 | Enable remote LFA tunnel repair for a specific area | PLR routers | `router ospf <process-id>` then `fast-reroute per-prefix remote-lfa area <area-id> tunnel mpls-ldp` | PLR can build MPLS-LDP remote LFA tunnels for the area |
| 41 | Set remote LFA maximum tunnel endpoint cost if required | PLR routers | `fast-reroute per-prefix remote-lfa area <area-id> maximum-cost <distance>` | Candidate remote LFA tailends are limited by maximum cost |
| 42 | Verify remote LFA tunnel creation | PLR routers | `show ip ospf fast-reroute remote-lfa tunnels` | MPLS-Remote-Lfa tunnel information appears |
| 43 | Verify remote LFA protected prefixes | PLR routers | `show ip ospf fast-reroute remote-lfa tunnels` | Output lists protected prefixes and tunnel tailend information |
| 44 | Verify repair-path programming after remote LFA | PLR routers | `show ip route repair-paths <protected-prefix> <mask>` | Prefix shows a remote LFA repair path when coverage exists |
| 45 | Confirm remote LFA failover in lab | Test router | `ping <protected-destination> repeat <count>` during controlled failure | Traffic survives or loses minimal packets during repair |
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Skeleton
! =========================================================
! Local OSPFv2 LFA FRR for all eligible prefixes
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 fast-reroute per-prefix enable prefix-priority low
end
write memory
! =========================================================
! Local OSPFv2 LFA FRR for selected high-priority prefixes only
! =========================================================
configure terminal
ip prefix-list <PREFIX_LIST_NAME> seq 5 permit <PROTECTED_PREFIX>/<PREFIX_LENGTH>
route-map <ROUTE_MAP_NAME> permit 10
 match ip address prefix-list <PREFIX_LIST_NAME>
exit
router ospf <PROCESS_ID>
 prefix-priority high route-map <ROUTE_MAP_NAME>
 fast-reroute per-prefix enable prefix-priority high
end
write memory
! =========================================================
! Repair-path tie-break policy
! Lower index values are evaluated with higher preference.
! Use required only when nonmatching candidates must be rejected.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 fast-reroute per-prefix tie-break node-protecting index <INDEX>
 fast-reroute per-prefix tie-break interface-disjoint index <INDEX>
 fast-reroute per-prefix tie-break broadcast-interface-disjoint index <INDEX>
 fast-reroute per-prefix tie-break downstream index <INDEX>
 fast-reroute per-prefix tie-break srlg index <INDEX>
end
write memory
! =========================================================
! Strict node-protecting example
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 fast-reroute per-prefix enable prefix-priority low
 fast-reroute per-prefix tie-break node-protecting required index 10
end
write memory
! =========================================================
! Keep all considered repair paths for lab troubleshooting
! Use sparingly because it increases memory consumption.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 fast-reroute keep-all-paths
end
write memory
! =========================================================
! Prevent one interface from being used as a repair next-hop candidate
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ip ospf fast-reroute per-prefix candidate disable
end
write memory
! =========================================================
! Remote LFA using MPLS-LDP
! MPLS must already be working in the core.
! Candidate tailend routers must advertise reachable /32 loopbacks.
! =========================================================
! Candidate remote LFA tailend:
configure terminal
mpls ldp discovery targeted-hello accept
end
write memory
! PLR:
configure terminal
router ospf <PROCESS_ID>
 fast-reroute per-prefix enable prefix-priority low
 fast-reroute per-prefix remote-lfa area <AREA_ID> tunnel mpls-ldp
 fast-reroute per-prefix remote-lfa area <AREA_ID> maximum-cost <DISTANCE>
end
write memory
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Verification_Commands
show running-config | section router ospf
show running-config interface <interface-id>
show ip ospf
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf neighbor
show ip ospf database
show ip ospf rib
show ip ospf rib <protected-prefix>
show ip ospf fast-reroute
show ip ospf fast-reroute remote-lfa tunnels
show ip route <protected-prefix>
show ip route repair-paths <protected-prefix> <mask>
show ip cef <protected-prefix> detail
show ip prefix-list
show route-map
show mpls interfaces
show mpls forwarding-table
show mpls ldp neighbor
show mpls ldp discovery
show mpls ldp bindings
ping <protected-destination> repeat <count>
traceroute <protected-destination>
debug ip ospf fast-reroute
show debugging
undebug all
show logging | include OSPF|FRR|LFA|ADJCHG|MPLS|LDP
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Rollback
! =========================================================
! Remove local per-prefix LFA
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no fast-reroute per-prefix enable prefix-priority low
 no fast-reroute per-prefix enable prefix-priority high
end
write memory
! =========================================================
! Remove prefix priority policy
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no prefix-priority high route-map <ROUTE_MAP_NAME>
end
write memory
! =========================================================
! Remove repair-path tie-breakers
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no fast-reroute per-prefix tie-break node-protecting index <INDEX>
 no fast-reroute per-prefix tie-break interface-disjoint index <INDEX>
 no fast-reroute per-prefix tie-break broadcast-interface-disjoint index <INDEX>
 no fast-reroute per-prefix tie-break downstream index <INDEX>
 no fast-reroute per-prefix tie-break srlg index <INDEX>
end
write memory
! =========================================================
! Remove keep-all-paths
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no fast-reroute keep-all-paths
end
write memory
! =========================================================
! Re-allow an interface as a repair candidate
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf fast-reroute per-prefix candidate disable
end
write memory
! =========================================================
! Remove remote LFA
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no fast-reroute per-prefix remote-lfa area <AREA_ID> tunnel mpls-ldp
 no fast-reroute per-prefix remote-lfa area <AREA_ID> maximum-cost <DISTANCE>
end
write memory
! =========================================================
! Remove targeted LDP acceptance from tailend candidate
! Only do this if no other service depends on targeted LDP.
! =========================================================
configure terminal
no mpls ldp discovery targeted-hello accept
end
write memory
! =========================================================
! Remove route map and prefix list after detaching OSPF policy
! =========================================================
configure terminal
no route-map <ROUTE_MAP_NAME>
no ip prefix-list <PREFIX_LIST_NAME>
end
write memory
! =========================================================
! Stop debug
! =========================================================
undebug all
! =========================================================
! Lab-only OSPF reset
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| No repair path appears | Topology has no valid local LFA candidate | `show ip route repair-paths <prefix> <mask>` | Use remote LFA, redesign topology, or adjust costs if appropriate |
| No repair path appears | LFA not enabled on the PLR | `show running-config | section router ospf` | Configure `fast-reroute per-prefix enable prefix-priority low` |
| No repair path appears for selected prefixes | Prefix priority is high but prefix does not match route map | `show route-map`, `show ip prefix-list`, `show ip ospf fast-reroute` | Correct prefix-list or route-map match logic |
| All prefixes are protected when only selected prefixes were expected | `prefix-priority low` was used | `show running-config | section router ospf` | Use `prefix-priority high` with a matching route map |
| Repair candidate is not the expected next hop | Default tie-break order chose a different candidate | `show ip ospf fast-reroute`, `show ip route repair-paths <prefix> <mask>` | Configure explicit tie-breakers |
| Node protection is missing | Repair candidate bypasses link but not next-hop router | `show ip route repair-paths <prefix> <mask>` | Add `fast-reroute per-prefix tie-break node-protecting required index <index>` if strict node protection is required |
| Link protection is weak on broadcast segment | Repair path uses same broadcast network with a different next hop | `show ip route repair-paths <prefix> <mask>` | Use `broadcast-interface-disjoint` tie-breaker |
| Repair path loops during failure | Candidate is not safely downstream for the failure case | `show ip ospf fast-reroute`, `traceroute <destination>` | Prefer or require `downstream` tie-breaker |
| Bad interface is selected as repair candidate | Interface was not excluded | `show running-config interface <interface-id>` | Configure `ip ospf fast-reroute per-prefix candidate disable` |
| `fast-reroute keep-all-paths` causes memory pressure | All candidate paths are being retained | `show memory`, `show ip ospf fast-reroute` | Remove `fast-reroute keep-all-paths` after lab analysis |
| FRR command is rejected | Platform or forwarding plane does not support OSPF LFA | `router ospf <process-id>` then `?` | Use a supported IOS XE image or document feature limitation |
| LFA unsupported because router is a virtual-link headend | Feature restriction | `show ip ospf virtual-links` | Do not use LFA on that router, or redesign without virtual-link headend dependency |
| Remote LFA tunnel does not appear | MPLS is not working | `show mpls interfaces`, `show mpls ldp neighbor` | Fix MPLS and LDP before remote LFA troubleshooting |
| Remote LFA tunnel does not appear | Tailend lacks /32 reachable address in the area | `show ip route <tailend-loopback>` | Advertise a stable /32 loopback into the remote LFA area |
| Remote LFA tunnel does not appear | Tailend does not accept targeted LDP | `show mpls ldp discovery`, `show mpls ldp neighbor` | Configure `mpls ldp discovery targeted-hello accept` on candidate tailend |
| Remote LFA tunnel does not appear | Remote LFA not enabled for correct area | `show running-config | section router ospf` | Use `fast-reroute per-prefix remote-lfa area <area-id> tunnel mpls-ldp` |
| Remote LFA candidate excluded by maximum cost | `maximum-cost` value is too low | `show ip ospf fast-reroute remote-lfa tunnels` | Raise or remove `maximum-cost` if the candidate is valid |
| Remote LFA is configured but no protection improves | Topology still lacks a valid PQ or repair endpoint | `show ip ospf fast-reroute remote-lfa tunnels`, `show ip route repair-paths <prefix> <mask>` | Adjust topology, MPLS reachability, or consider TI-LFA with segment routing |
| Prefix has multiple primary ECMP paths and incomplete repair coverage | Not every primary path has a valid repair path | `show ip route repair-paths <prefix> <mask>` | Review ECMP topology and repair candidates |
| Controlled failure still causes high packet loss | Failure detection is slow | `show ip ospf neighbor`, `show bfd neighbors` | Add or tune BFD or OSPF fast hellos separately |
| Controlled failure still causes high packet loss | Repair path was not programmed in forwarding plane | `show ip cef <prefix> detail`, `show ip route repair-paths <prefix> <mask>` | Verify platform support and CEF programming |
| OSPF reconverges but traffic takes poor path | FRR is temporary repair, then normal SPF chooses post-convergence path | `traceroute <destination>`, `show ip route <prefix>` | Tune OSPF cost or redesign topology |
| Route exists but ping fails during test | Return path missing or ACL/data-plane issue | `show ip route <source-prefix>` on return-side routers | Verify reverse routing and filtering |
| Debug output is too noisy | Debug left enabled during convergence events | `show debugging` | Use debug only in lab and stop with `undebug all` |
##### Source_Basis
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Mental_Model
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Configuration_Checklist
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Skeleton
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Verification_Commands
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Rollback
# OSPF_Fast_Reroute_LFA_And_Remote_LFA_Failure_Checks
# index of each title throughout note, not in table format

