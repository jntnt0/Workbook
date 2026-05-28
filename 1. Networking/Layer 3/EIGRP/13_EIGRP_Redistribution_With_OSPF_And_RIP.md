EIGRP_Redistribution_With_OSPF_And_RIP.md

EIGRP_Redistribution_With_OSPF_And_RIP

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP, Chapter 9: Integrating EIGRP with Other Routing Protocols | Points EIGRP redistribution, seed metrics, route tagging, and loop prevention to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 9: Integrating EIGRP with Other Enterprise Routing Protocols | Supports EIGRP redistribution with OSPF and RIP |
| `All_combined_part3.md` | Chapter 9 redistribution examples | Supports `redistribute ospf`, `redistribute eigrp`, `redistribute rip`, `metric`, `subnets`, and `route-map` syntax |
| `All_combined_part3.md` | Mutual redistribution loop examples | Supports route-tagging loop prevention using `match tag` and `set tag` |
| `All_combined_part3.md` | RIP and EIGRP mutual redistribution example | Supports `redistribute eigrp 100 metric 1` and `redistribute rip metric 128 2000 255 1 1500` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Redistribution | Supports seed metric requirements, external route behavior, route-map tagging, and redistribution verification |
# EIGRP_Redistribution_With_OSPF_And_RIP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Redistribution | Takes routes from one routing domain and injects them into another routing domain |
| Boundary router | Router running both routing protocols and performing redistribution |
| One-way redistribution | Routes move from protocol A into protocol B only |
| Mutual redistribution | Routes move both directions between protocols |
| Seed metric | Required because each protocol uses a different metric system |
| EIGRP seed metric | Bandwidth, delay, reliability, load, and MTU must be supplied when redistributing OSPF or RIP into EIGRP |
| OSPF seed metric | OSPF external metric is supplied with `metric`; default external type is usually E2 unless changed |
| RIP seed metric | RIP needs a hop-count metric from 1 to 15 |
| OSPF `subnets` keyword | Required when redistributing classless subnetted routes into OSPF |
| EIGRP external route | Redistributed routes appear as `D EX` and normally use AD 170 |
| OSPF external route | Redistributed routes appear as `O E1` or `O E2` |
| RIP redistributed route | Redistributed routes appear as `R` with AD 120 |
| Route tag | Metadata attached to redistributed routes to identify their origin |
| Loop prevention | Deny routes from being redistributed back into the protocol they originally came from |
| Common tag convention | EIGRP routes tag `90`, OSPF routes tag `110`, RIP routes tag `120` |
| Dangerous shortcut | Mutual redistribution without filtering or tagging can create routing loops even when every router has a route |
| Control-plane success | Routes appear in tables |
| Data-plane success | Ping and traceroute prove traffic is not looping or blackholing |
# EIGRP_Redistribution_With_OSPF_And_RIP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are operational before redistribution | All routers | `show ip interface brief` | Protocol-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable | EIGRP routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm OSPF neighbors are stable | OSPF routers | `show ip ospf neighbor` | Expected neighbors are `FULL` or correct state for network type |
| 4 | Confirm RIP is active where expected | RIP routers | `show ip protocols` | RIP version, networks, timers, and passive interfaces are shown |
| 5 | Record baseline EIGRP routes | Boundary and EIGRP routers | `show ip route eigrp` | Existing EIGRP routes are documented before redistribution |
| 6 | Record baseline OSPF routes | Boundary and OSPF routers | `show ip route ospf` | Existing OSPF routes are documented before redistribution |
| 7 | Record baseline RIP routes | Boundary and RIP routers | `show ip route rip` | Existing RIP routes are documented before redistribution |
| 8 | Identify the EIGRP AS number | Boundary router | `show ip protocols` | EIGRP AS number is confirmed |
| 9 | Identify the OSPF process ID | Boundary router | `show ip protocols` | OSPF process ID is confirmed |
| 10 | Confirm RIP version 2 and no auto-summary before RIP redistribution | RIP routers | `show running-config | section router rip` | `version 2` and `no auto-summary` are present |
| 11 | Create EIGRP-to-OSPF route-map deny statement to prevent OSPF-originated routes from returning to OSPF | Boundary router | `route-map EIGRP_TO_OSPF deny 10` then `match tag 110` | Routes tagged as OSPF-originated are denied |
| 12 | Create EIGRP-to-OSPF route-map permit statement and tag EIGRP-originated routes | Boundary router | `route-map EIGRP_TO_OSPF permit 90` then `set tag 90` | EIGRP routes redistributed into OSPF receive tag `90` |
| 13 | Apply EIGRP into OSPF redistribution | Boundary router | `router ospf <OSPF_PROCESS_ID>` then `redistribute eigrp <EIGRP_AS> subnets route-map EIGRP_TO_OSPF` | EIGRP routes are injected into OSPF as external routes |
| 14 | Create OSPF-to-EIGRP route-map deny statement to prevent EIGRP-originated routes from returning to EIGRP | Boundary router | `route-map OSPF_TO_EIGRP deny 10` then `match tag 90` | Routes tagged as EIGRP-originated are denied |
| 15 | Create OSPF-to-EIGRP route-map permit statement and tag OSPF-originated routes | Boundary router | `route-map OSPF_TO_EIGRP permit 90` then `set tag 110` | OSPF routes redistributed into EIGRP receive tag `110` |
| 16 | Apply OSPF into EIGRP redistribution with seed metric | Boundary router | `router eigrp <EIGRP_AS>` then `redistribute ospf <OSPF_PROCESS_ID> metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU> route-map OSPF_TO_EIGRP` | OSPF routes enter EIGRP as `D EX` |
| 17 | Verify EIGRP-to-OSPF route-map logic | Boundary router | `show route-map EIGRP_TO_OSPF` | Deny sequence matches tag `110`; permit sequence sets tag `90` |
| 18 | Verify OSPF-to-EIGRP route-map logic | Boundary router | `show route-map OSPF_TO_EIGRP` | Deny sequence matches tag `90`; permit sequence sets tag `110` |
| 19 | Verify OSPF external routes after EIGRP redistribution | OSPF routers | `show ip route ospf` | EIGRP-originated routes appear as `O E2` or `O E1` |
| 20 | Verify OSPF external LSAs | OSPF routers | `show ip ospf database external` | Redistributed EIGRP prefixes appear as Type 5 external LSAs |
| 21 | Verify EIGRP external routes after OSPF redistribution | EIGRP routers | `show ip route eigrp` | OSPF-originated routes appear as `D EX` |
| 22 | Verify EIGRP topology entries for redistributed routes | EIGRP routers | `show ip eigrp topology` | Redistributed prefixes appear in the EIGRP topology table |
| 23 | Create EIGRP-to-RIP route-map deny statement to prevent RIP-originated routes from returning to RIP | Boundary router | `route-map EIGRP_TO_RIP deny 10` then `match tag 120` | Routes tagged as RIP-originated are denied |
| 24 | Create EIGRP-to-RIP route-map permit statement and tag EIGRP-originated routes | Boundary router | `route-map EIGRP_TO_RIP permit 90` then `set tag 90` | EIGRP routes redistributed into RIP receive tag `90` |
| 25 | Apply EIGRP into RIP redistribution with hop-count metric | Boundary router | `router rip` then `redistribute eigrp <EIGRP_AS> metric <RIP_HOP_COUNT> route-map EIGRP_TO_RIP` | EIGRP routes are injected into RIP with valid hop count |
| 26 | Create RIP-to-EIGRP route-map deny statement to prevent EIGRP-originated routes from returning to EIGRP | Boundary router | `route-map RIP_TO_EIGRP deny 10` then `match tag 90` | Routes tagged as EIGRP-originated are denied |
| 27 | Create RIP-to-EIGRP route-map permit statement and tag RIP-originated routes | Boundary router | `route-map RIP_TO_EIGRP permit 90` then `set tag 120` | RIP routes redistributed into EIGRP receive tag `120` |
| 28 | Apply RIP into EIGRP redistribution with seed metric | Boundary router | `router eigrp <EIGRP_AS>` then `redistribute rip metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU> route-map RIP_TO_EIGRP` | RIP routes enter EIGRP as `D EX` |
| 29 | Verify EIGRP-to-RIP route-map logic | Boundary router | `show route-map EIGRP_TO_RIP` | Deny sequence matches tag `120`; permit sequence sets tag `90` |
| 30 | Verify RIP-to-EIGRP route-map logic | Boundary router | `show route-map RIP_TO_EIGRP` | Deny sequence matches tag `90`; permit sequence sets tag `120` |
| 31 | Verify RIP-learned redistributed routes | RIP routers | `show ip route rip` | EIGRP-originated routes appear as `R` |
| 32 | Verify RIP database if supported | RIP routers | `show ip rip database` | Redistributed routes appear with expected metric |
| 33 | Verify EIGRP external routes learned from RIP | EIGRP routers | `show ip route eigrp` | RIP-originated routes appear as `D EX` |
| 34 | Verify protocol-level redistribution configuration | Boundary router | `show ip protocols` | Redistributed protocols, metrics, and route-maps are listed |
| 35 | Clear route table in lab after redistribution if stale routes remain | Lab routers | `clear ip route *` | Route tables rebuild from current protocol state |
| 36 | Clear EIGRP neighbors only in a lab if EIGRP external routes do not refresh | Lab EIGRP routers | `clear ip eigrp neighbors` | EIGRP adjacencies reset and routes are relearned |
| 37 | Clear OSPF process only in a lab if OSPF external LSAs do not refresh | Lab OSPF routers | `clear ip ospf process` | OSPF process resets after confirmation |
| 38 | Verify route tags on redistributed routes | Boundary and downstream routers | `show ip route <PREFIX>` | Redistributed routes show expected route tag where platform displays it |
| 39 | Verify no route feedback loop exists | Multiple routers | `traceroute <REMOTE_PREFIX_IP> numeric` | Trace reaches destination without repeating the same router sequence |
| 40 | Verify end-to-end reachability across redistributed domains | Edge routers or hosts | `ping <REMOTE_PREFIX_IP>` | Ping succeeds across EIGRP, OSPF, and RIP domains |
| 41 | Verify return path from remote domain | Remote router | `show ip route <SOURCE_PREFIX>` | Return route exists through the intended redistributed path |
| 42 | Save the configuration after stable control-plane and data-plane verification | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Redistribution_With_OSPF_And_RIP_Skeleton
! Baseline verification
show ip interface brief
show ip protocols
show ip eigrp neighbors
show ip ospf neighbor
show ip route eigrp
show ip route ospf
show ip route rip
! EIGRP into OSPF with route tagging
configure terminal
 route-map EIGRP_TO_OSPF deny 10
  match tag 110
 route-map EIGRP_TO_OSPF permit 90
  set tag 90
 router ospf <OSPF_PROCESS_ID>
  redistribute eigrp <EIGRP_AS> subnets route-map EIGRP_TO_OSPF
end
! OSPF into EIGRP with seed metric and route tagging
configure terminal
 route-map OSPF_TO_EIGRP deny 10
  match tag 90
 route-map OSPF_TO_EIGRP permit 90
  set tag 110
 router eigrp <EIGRP_AS>
  redistribute ospf <OSPF_PROCESS_ID> metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU> route-map OSPF_TO_EIGRP
end
! Example EIGRP seed metric for lab use
! metric 100000 10 255 1 1500
! EIGRP into RIP with route tagging
configure terminal
 route-map EIGRP_TO_RIP deny 10
  match tag 120
 route-map EIGRP_TO_RIP permit 90
  set tag 90
 router rip
  version 2
  no auto-summary
  redistribute eigrp <EIGRP_AS> metric <RIP_HOP_COUNT> route-map EIGRP_TO_RIP
end
! RIP into EIGRP with seed metric and route tagging
configure terminal
 route-map RIP_TO_EIGRP deny 10
  match tag 90
 route-map RIP_TO_EIGRP permit 90
  set tag 120
 router eigrp <EIGRP_AS>
  redistribute rip metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU> route-map RIP_TO_EIGRP
end
! Example RIP to EIGRP seed metric from lab pattern
! metric 128 2000 255 1 1500
! Verification
show running-config | section router eigrp
show running-config | section router ospf
show running-config | section router rip
show route-map
show ip protocols
show ip route eigrp
show ip route ospf
show ip route rip
show ip ospf database external
show ip eigrp topology
show ip rip database
show ip route <PREFIX>
ping <REMOTE_PREFIX_IP>
traceroute <REMOTE_PREFIX_IP> numeric
! Lab-only convergence refresh
clear ip route *
clear ip eigrp neighbors
clear ip ospf process
! Save
write memory
# EIGRP_Redistribution_With_OSPF_And_RIP_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before redistribution | Routing interfaces are `up/up` |
| `show ip protocols` | Confirms protocol state and redistribution policy | Shows EIGRP, OSPF, RIP, redistribution statements, metrics, and route-map references |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip ospf neighbor` | Confirms OSPF adjacency health | Expected neighbors are `FULL` where required |
| `show running-config | section router eigrp` | Confirms EIGRP redistribution | Shows `redistribute ospf` and `redistribute rip` with metrics and route-maps |
| `show running-config | section router ospf` | Confirms OSPF redistribution | Shows `redistribute eigrp <AS> subnets route-map <NAME>` |
| `show running-config | section router rip` | Confirms RIP redistribution | Shows `version 2`, `no auto-summary`, and `redistribute eigrp <AS> metric <HOPS>` |
| `show route-map` | Confirms tag filtering and tag setting | Deny sequences match return tags; permit sequences set origin tags |
| `show ip route eigrp` | Confirms EIGRP route installation | Redistributed routes appear as `D EX` |
| `show ip route ospf` | Confirms OSPF route installation | Redistributed routes appear as `O E1` or `O E2` |
| `show ip route rip` | Confirms RIP route installation | Redistributed routes appear as `R` |
| `show ip route <PREFIX>` | Confirms exact RIB winner and route tag | Route points to expected next hop and shows expected source protocol |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Confirms EIGRP topology state for redistributed route | Prefix appears as passive with valid successor |
| `show ip ospf database external` | Confirms OSPF external LSA generation | Redistributed prefixes appear as external LSAs |
| `show ip rip database` | Confirms RIP database state if supported | Redistributed prefixes appear with valid hop count |
| `traceroute <REMOTE_PREFIX_IP> numeric` | Confirms no forwarding loop exists | Trace reaches destination without repeating router sequence |
| `ping <REMOTE_PREFIX_IP>` | Confirms end-to-end data-plane reachability | Ping succeeds |
| `show logging | include EIGRP|OSPF|RIP|DUAL|ADJCHG` | Checks convergence and adjacency events | No repeated instability after redistribution settles |
# EIGRP_Redistribution_With_OSPF_And_RIP_Rollback
! Remove EIGRP into OSPF redistribution
configure terminal
 router ospf <OSPF_PROCESS_ID>
  no redistribute eigrp <EIGRP_AS>
end
! Remove OSPF into EIGRP redistribution
configure terminal
 router eigrp <EIGRP_AS>
  no redistribute ospf <OSPF_PROCESS_ID>
end
! Remove EIGRP into RIP redistribution
configure terminal
 router rip
  no redistribute eigrp <EIGRP_AS>
end
! Remove RIP into EIGRP redistribution
configure terminal
 router eigrp <EIGRP_AS>
  no redistribute rip
end
! Remove route-maps if created only for this redistribution design
configure terminal
 no route-map EIGRP_TO_OSPF
 no route-map OSPF_TO_EIGRP
 no route-map EIGRP_TO_RIP
 no route-map RIP_TO_EIGRP
end
! Remove optional prefix-lists if used only for this redistribution design
configure terminal
 no ip prefix-list <PREFIX_LIST_NAME>
end
! Lab-only convergence refresh
clear ip route *
clear ip eigrp neighbors
clear ip ospf process
! Verify rollback
show running-config | section router eigrp
show running-config | section router ospf
show running-config | section router rip
show route-map
show ip protocols
show ip route eigrp
show ip route ospf
show ip route rip
show ip ospf database external
show ip eigrp topology
ping <REMOTE_PREFIX_IP>
traceroute <REMOTE_PREFIX_IP> numeric
! Expected result:
! Redistributed routes disappear from the opposite routing domains.
! Native protocol routes remain.
! No mutual redistribution route feedback remains.
write memory
# EIGRP_Redistribution_With_OSPF_And_RIP_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| OSPF routes do not appear in EIGRP | Missing EIGRP seed metric | `show running-config | section router eigrp` | Add `metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU>` to `redistribute ospf` |
| RIP routes do not appear in EIGRP | Missing EIGRP seed metric | `show running-config | section router eigrp` | Add `metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU>` to `redistribute rip` |
| EIGRP routes do not appear in OSPF | Missing `subnets` keyword | `show running-config | section router ospf` | Add `subnets` to `redistribute eigrp <AS> subnets` |
| EIGRP routes do not appear in RIP | RIP metric missing or invalid | `show running-config | section router rip` | Add `metric <1-15>` to `redistribute eigrp` |
| RIP routes disappear or summarize incorrectly | RIP auto-summary enabled | `show running-config | section router rip` | Configure `version 2` and `no auto-summary` |
| Routes appear but ping fails | Return path missing | Remote `show ip route <SOURCE_PREFIX>` | Ensure reverse route exists through redistribution or native routing |
| Routes appear but traceroute loops | Mutual redistribution feedback loop | `traceroute <DESTINATION> numeric` | Add route tags and deny routes from returning to their source domain |
| EIGRP external routes have unexpected AD | Redistributed EIGRP routes are external | `show ip route eigrp` | Recognize `D EX` normally uses AD 170 |
| OSPF external routes are not preferred as expected | E1 versus E2 behavior or metric issue | `show ip route ospf` and `show ip ospf database external` | Set appropriate external metric or `metric-type type-1` if required |
| Route-map has no effect | Route-map not attached to redistribution | `show ip protocols` and `show running-config | section router` | Attach route-map to the relevant `redistribute` command |
| Route-map blocks everything | Missing final permit sequence | `show route-map <NAME>` | Add permit sequence with intended `set tag` behavior |
| Route-map does not block looped routes | Wrong tag matched | `show ip route <PREFIX>` and `show route-map <NAME>` | Match the origin tag of the opposite protocol |
| Tags are not visible in normal output | Platform does not show tag in brief route output | `show ip route <PREFIX>` | Use exact route output and protocol database commands |
| OSPF external LSA exists but route not in RIB | Better route exists or forwarding address issue | `show ip route <PREFIX>` and `show ip ospf database external <PREFIX>` | Check AD, metric, forwarding address, and reachability |
| EIGRP topology has route but RIB does not | Another protocol wins RIB selection | `show ip route <PREFIX>` | Compare administrative distance and metric |
| RIP route metric reaches 16 | RIP hop count exceeded | `show ip route rip` and `show ip rip database` | Reduce RIP hop count or redesign redistribution boundary |
| Multiple boundary routers cause inconsistent paths | No tag-based loop control across all boundaries | `show route-map` on every boundary router | Standardize route tags and deny rules on every redistribution point |
| Boundary router redistributes connected routes unintentionally | Broad redistribution policy | `show ip protocols` | Use route-maps, prefix-lists, or protocol-specific redistribution only |
| Clear commands disrupt the lab | `clear ip ospf process` or neighbor clears are intrusive | `show logging` | Use only in lab or maintenance window |
| Data plane fails even though control plane looks right | ACL, NAT, interface, or CEF issue outside redistribution | `traceroute`, `show ip route`, interface counters | Troubleshoot forwarding separately after route tables are correct |
# Index
# Source_Basis
# EIGRP_Redistribution_With_OSPF_And_RIP_Mental_Model
# EIGRP_Redistribution_With_OSPF_And_RIP_Configuration_Checklist
# EIGRP_Redistribution_With_OSPF_And_RIP_Skeleton
# EIGRP_Redistribution_With_OSPF_And_RIP_Verification_Commands
# EIGRP_Redistribution_With_OSPF_And_RIP_Rollback
# EIGRP_Redistribution_With_OSPF_And_RIP_Failure_Checks