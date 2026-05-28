EIGRP_DUAL_FSM_Query_Reply_And_SIA.md

EIGRP_DUAL_FSM_Query_Reply_And_SIA

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `All_combined_part3.md`, Book 3: EIGRP for IP | Points DUAL, feasible successors, convergence, query behavior, and troubleshooting to the main EIGRP source |
| `All_combined_part3.md` | Book 3, Chapter 1: DUAL, The Heart of EIGRP | Supports DUAL as the core EIGRP algorithm using hello, RTP, neighbor table, and topology table |
| `All_combined_part3.md` | Book 3, Chapter 1: Diffusing Computation | Supports active route state, reply-status table, query generation, reply collection, and route recomputation |
| `All_combined_part3.md` | Book 3, Chapter 1: Receiving a Query Packet and Responding to It | Supports query response logic for missing routes, already-active routes, successor-originated queries, nonsuccessor queries, and recursive query propagation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 3: DUAL and Stuck in Active | Supports passive/active route behavior, query/reply flow, SIA query, SIA reply, and active timer behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 3-3: `show ip eigrp topology active` | Supports identifying active routes and remaining replies |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 3-4: Configuring SIA Timers | Supports `timers active-time` in classic mode and named mode |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Summarization and Stub Routing | Supports summarization and stub routing as query-boundary mechanisms that reduce SIA risk |
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DUAL | The EIGRP algorithm that maintains loop-free routing and recomputes routes during topology changes |
| Passive route | Stable EIGRP topology state; no recomputation is happening for that prefix |
| Active route | EIGRP lost a usable route and is waiting for query replies before choosing a new path |
| Query | Sent when a router has no immediately usable feasible successor and must ask neighbors for a path |
| Reply | Sent in response to a query; DUAL cannot finish until required replies are received |
| Reply-status table | Per-prefix tracking table showing which neighbors still owe replies |
| Feasible successor | Prevents the route from going active because EIGRP already has a loop-free backup |
| Local computation | EIGRP switches to a feasible successor without querying the rest of the domain |
| Diffusing computation | EIGRP marks the route active, sends queries, waits for replies, then installs the new best path |
| Query boundary | A place where a query stops because the router can answer without propagating the query farther |
| Stub router | Reduces query scope by telling neighbors not to query it for routes it should never transit |
| Summary route | Reduces query scope because specific component routes are hidden behind the summary |
| Stuck in active | A route remains active too long because a required query reply is missing or delayed |
| SIA query | Sent halfway through the active timer to check whether a neighbor is still working on the query |
| SIA reply | Response to the SIA query proving the neighbor is still alive and processing |
| Active timer | Default is 3 minutes; controls how long EIGRP waits before declaring SIA |
| Core operational rule | Active routes are not normal. Find the neighbor with the remaining reply status and walk the query chain downstream |
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational before analyzing DUAL behavior | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm EIGRP process settings | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, AD values, and active timer are shown |
| 4 | Confirm the topology table is stable before testing | All routers | `show ip eigrp topology` | Expected prefixes show `P` for passive |
| 5 | Confirm no routes are currently active | All routers | `show ip eigrp topology active` | No active prefixes are listed |
| 6 | Pick a target prefix for DUAL testing | Test router | `show ip route eigrp` | Target EIGRP-learned route is present |
| 7 | Inspect the target prefix before failure | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Route is passive, has a successor, and shows FD/RD values |
| 8 | Determine whether a feasible successor exists before failure | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Backup path appears if it is a feasible successor |
| 9 | Display non-feasible paths if normal topology output does not show backups | Test router | `show ip eigrp topology all-links` | All known paths appear, including paths that failed feasibility condition |
| 10 | Record the current RIB winner | Test router | `show ip route <PREFIX>` | Installed route points to the current EIGRP successor |
| 11 | Configure classic-mode active timer only if the lab explicitly requires timer testing | Classic EIGRP router | `router eigrp <AS_NUMBER>` then `timers active-time <MINUTES>` | Active timer changes from default value |
| 12 | Configure named-mode active timer only if the lab explicitly requires timer testing | Named-mode EIGRP router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `topology base` then `timers active-time <MINUTES>` | Active timer changes under topology base |
| 13 | Verify active timer value | All changed routers | `show ip protocols | include Active` | Active timer shows intended value |
| 14 | Create a controlled failure only in a lab | Router on successor path | `interface <SUCCESSOR_PATH_INTERFACE>` then `shutdown` | Successor path is removed from the topology |
| 15 | Check whether EIGRP performs local computation | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | If a feasible successor existed, route should remain passive and switch to backup |
| 16 | Check whether route goes active | Test router | `show ip eigrp topology active` | Prefix appears active only if no usable feasible successor was available |
| 17 | Identify which neighbor still owes a reply | Test router | `show ip eigrp topology active` | Remaining reply neighbor is marked with reply status |
| 18 | Walk to the downstream neighbor that owes the reply | Reply-status neighbor | `show ip eigrp topology active` | Shows whether that router is waiting on another downstream neighbor |
| 19 | Continue walking the reply chain until the query boundary or broken segment is found | Downstream routers | `show ip eigrp topology active` | The last router either has no active route, has a missing neighbor, or is waiting on another device |
| 20 | Verify neighbor queue health during query behavior | Routers in query path | `show ip eigrp neighbors` | `Q Cnt` should not stay elevated |
| 21 | Check logs for DUAL neighbor and SIA events | Routers in query path | `show logging | include DUAL|EIGRP|SIA|active` | Logs reveal neighbor resets, SIA query/reply events, or route activity |
| 22 | Use packet debugging only in a lab if command output is not enough | Lab router only | `debug eigrp packets` | Query, reply, SIA query, and SIA reply packet activity can be observed |
| 23 | Use FSM debugging only in a lab if DUAL state transitions need to be observed | Lab router only | `debug eigrp fsm` | DUAL active/passive transitions can be observed |
| 24 | Disable debugging immediately after capture | Lab router only | `undebug all` | Debugging is stopped |
| 25 | Restore the failed interface | Router on successor path | `interface <SUCCESSOR_PATH_INTERFACE>` then `no shutdown` | Interface returns to `up/up` |
| 26 | Verify neighbors recover cleanly | All affected routers | `show ip eigrp neighbors` | Expected neighbors return with stable uptime and `Q Cnt` of `0` |
| 27 | Verify the target prefix returns to passive state | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Prefix shows `State is Passive` |
| 28 | Verify no active prefixes remain | All affected routers | `show ip eigrp topology active` | No active prefixes remain |
| 29 | Verify the RIB has a valid EIGRP route | Test router | `show ip route <PREFIX>` | Route points to expected successor after convergence |
| 30 | Verify data-plane reachability | Test router/host | `ping <REMOTE_IP>` | Ping succeeds |
| 31 | Verify forwarding path | Test router/host | `traceroute <REMOTE_IP>` | Path follows expected post-convergence route |
| 32 | Add query-boundary design only if this is a scale or SIA-prevention lab | Edge/stub routers | `eigrp stub connected summary` | Neighbors do not query the stub for transit routes |
| 33 | Add summarization only if this is a hierarchy or query-scope lab | Summarizing interface | `ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>` | Specific routes are hidden behind a summary, reducing query scope |
| 34 | Verify stub recognition if stub was configured | Upstream router | `show ip eigrp neighbors detail` | Neighbor is identified as stub |
| 35 | Verify summary route behavior if summarization was configured | Upstream/downstream routers | `show ip route eigrp` | Summary appears and component specifics are hidden where intended |
| 36 | Save only intentional timer, stub, or summarization changes | Changed routers | `write memory` | Working configuration is saved |
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Skeleton
! Baseline health
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip eigrp topology
show ip eigrp topology active
! Pick a prefix
show ip route eigrp
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip eigrp topology all-links
show ip route <PREFIX>
! Optional classic-mode active timer test
configure terminal
 router eigrp <AS_NUMBER>
  timers active-time <MINUTES>
 end
! Optional named-mode active timer test
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    timers active-time <MINUTES>
   exit-af-topology
  exit-address-family
 end
! Verify active timer
show ip protocols | include Active
! Controlled lab failure
configure terminal
 interface <SUCCESSOR_PATH_INTERFACE>
  shutdown
 end
! DUAL query and active-state inspection
show ip eigrp topology active
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip eigrp neighbors
show logging | include DUAL|EIGRP|SIA|active
! Lab-only debugging
debug eigrp packets
debug eigrp fsm
undebug all
! Restore failure
configure terminal
 interface <SUCCESSOR_PATH_INTERFACE>
  no shutdown
 end
! Query-boundary prevention examples
configure terminal
 router eigrp <AS_NUMBER>
  eigrp stub connected summary
 end
configure terminal
 interface <SUMMARY_INTERFACE>
  ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>
 end
! Final verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip eigrp topology active
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip route <PREFIX>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save only intentional changes
write memory
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before DUAL testing | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms neighbor stability | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process settings | Correct AS, RID, K-values, passive interfaces, network statements, AD values, and active timer |
| `show ip protocols | include Active` | Confirms active timer | Shows configured active timer, usually default `3 min` unless changed |
| `show ip eigrp topology` | Confirms stable topology | Prefixes normally show passive state |
| `show ip eigrp topology active` | Shows active routes and outstanding replies | Healthy steady state shows no active prefixes |
| `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Checks one prefix in detail | Shows passive/active state, successor count, FD/RD, next hops, and reply status if active |
| `show ip eigrp topology all-links` | Shows all known paths | Reveals paths that do not appear in normal topology output |
| `show ip route <PREFIX>` | Confirms actual RIB winner | Route points to expected successor next hop |
| `show ip route eigrp` | Lists installed EIGRP routes | Internal routes appear as `D`; external routes appear as `D EX` |
| `show ip eigrp neighbors detail` | Checks neighbor detail and stub status | Stub neighbors are identified when configured |
| `show logging | include DUAL|EIGRP|SIA|active` | Checks convergence and SIA-related events | No recurring SIA or neighbor reset messages |
| `debug eigrp packets` | Lab-only packet visibility | Shows update, query, reply, SIA query, and SIA reply activity |
| `debug eigrp fsm` | Lab-only DUAL state visibility | Shows DUAL state transitions |
| `undebug all` | Stops debugging | All debugging is disabled |
| `ping <REMOTE_IP>` | Confirms reachability | Ping succeeds |
| `traceroute <REMOTE_IP>` | Confirms forwarding path | Path follows expected EIGRP route |
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Rollback
! Restore failed interface used for testing
configure terminal
 interface <SUCCESSOR_PATH_INTERFACE>
  no shutdown
 end
! Roll back classic-mode active timer to default behavior
configure terminal
 router eigrp <AS_NUMBER>
  no timers active-time
 end
! Roll back named-mode active timer to default behavior
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no timers active-time
   exit-af-topology
  exit-address-family
 end
! Remove classic-mode stub if it was used only for testing
configure terminal
 router eigrp <AS_NUMBER>
  no eigrp stub
 end
! Remove classic-mode summary if it was used only for testing
configure terminal
 interface <SUMMARY_INTERFACE>
  no ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>
 end
! Stop any active debugging
undebug all
! Verify rollback
show ip interface brief
show ip eigrp neighbors
show ip protocols | include Active
show ip eigrp topology active
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip route <PREFIX>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
write memory
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Prefix shows active | No feasible successor exists and DUAL is querying | `show ip eigrp topology active` | Identify remaining replies and walk the query chain downstream |
| Prefix stays active too long | Neighbor has not replied to query | `show ip eigrp topology active` | Find neighbor marked with reply status and troubleshoot that neighbor or link |
| SIA messages appear in logs | Delayed or missing query replies | `show logging | include SIA|DUAL|EIGRP` | Fix packet loss, slow link, overloaded router, or oversized query domain |
| Neighbor resets during convergence | SIA condition or unreliable adjacency | `show ip eigrp neighbors` and `show logging` | Stabilize the link and reduce query scope |
| `Q Cnt` stays above zero | EIGRP packets are queued or neighbor is slow to respond | `show ip eigrp neighbors` | Check CPU, link loss, MTU, congestion, and adjacency health |
| Active route points to one remaining neighbor | Query chain continues downstream | `show ip eigrp topology active` on that neighbor | Continue tracing until the broken segment or query boundary is found |
| Downstream router has no active route | Query may not have reached it or packet was dropped | `show ip eigrp topology active` and `show logging` | Check link, ACL, congestion, or adjacency between upstream and downstream routers |
| Route goes active after every failure | No feasible successor exists | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Add redundancy or adjust topology/metrics so feasible successors exist |
| Query spreads too far | No query boundaries | `show ip eigrp topology active` | Use summarization and stub routing where design permits |
| Spoke routers are queried unnecessarily | Stub routing missing on non-transit routers | `show ip eigrp neighbors detail` | Configure `eigrp stub connected summary` on spokes where appropriate |
| Core routers process too many specific-route queries | Summarization missing | `show ip route eigrp` | Add hierarchical EIGRP summarization at proper boundaries |
| Active timer is not what expected | Timer configured differently in classic or named mode | `show ip protocols | include Active` | Correct `timers active-time` under the proper EIGRP hierarchy |
| Named-mode timer command does not apply | Command placed outside `topology base` | `show running-config | section router eigrp` | Configure `timers active-time` under address-family `topology base` |
| Classic-mode timer command does not apply | Command placed under wrong mode | `show running-config | section router eigrp` | Configure `timers active-time` directly under `router eigrp <AS_NUMBER>` |
| Debug output overwhelms router | Debugging used outside a controlled lab | `show debugging` | Run `undebug all`; do not leave EIGRP debugging enabled |
| Ping fails after convergence | RIB recovered but reverse path is missing | `show ip route <SOURCE_PREFIX>` on remote router | Restore reverse EIGRP route or fix filtering/summarization |
| Route is passive but wrong path is used | DUAL completed, but the selected successor is not the intended design path | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` and `show ip route <PREFIX>` | Review FD/RD, metrics, and route policy |
# Index
# Source_Basis
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Mental_Model
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Configuration_Checklist
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Skeleton
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Verification_Commands
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Rollback
# EIGRP_DUAL_FSM_Query_Reply_And_SIA_Failure_Checks
