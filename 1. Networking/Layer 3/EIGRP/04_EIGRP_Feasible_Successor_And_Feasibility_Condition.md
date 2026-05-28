EIGRP_Feasible_Successor_And_Feasibility_Condition.md

EIGRP_Feasible_Successor_And_Feasibility_Condition

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `All_combined_part3.md`, Book 3: EIGRP for IP | Points EIGRP concepts, DUAL, feasible successors, metrics, convergence, and troubleshooting to the main EIGRP source |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports reported distance, feasible distance, feasibility condition, and feasible successor logic |
| `All_combined_part3.md` | Book 3, Chapter 2: Advanced EIGRP Concepts, Data Structures and Algorithms | Supports DUAL behavior when a feasible successor exists or does not exist |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Terminology | Defines successor, feasible distance, reported distance, feasibility condition, and feasible successor |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Topology Table | Supports `show ip eigrp topology` and `show ip eigrp topology all-links` behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Troubleshooting Feasible Successors | Supports comparing FD and RD, identifying non-feasible paths, and validating feasible successor behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Load Balancing | Supports the rule that unequal-cost load balancing can only use feasible successors |
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Successor | The current best EIGRP next hop for a prefix |
| Feasible distance | The local router’s total calculated metric to reach the destination through a neighbor |
| Reported distance | The neighbor’s advertised distance to the destination |
| Feasibility condition | A backup path is loop-free only if its RD is lower than the current successor FD |
| Feasible successor | A backup path that passes the feasibility condition |
| Non-feasible path | A known path that fails the feasibility condition and cannot be used as an immediate loop-free backup |
| DUAL benefit | If the successor fails and a feasible successor exists, EIGRP can switch locally without going active |
| Active route | If no feasible successor exists after successor loss, EIGRP must query neighbors and recompute the route |
| `show ip eigrp topology` | Shows successors and feasible successors by default |
| `show ip eigrp topology all-links` | Shows all known paths, including paths that failed the feasibility condition |
| Variance dependency | Unequal-cost load balancing can install feasible successors only; variance cannot install a path that failed feasibility condition |
| Core rule | You do not configure a feasible successor directly. You create conditions where a backup path’s RD is lower than the successor FD |
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm the EIGRP process is correctly configured | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Identify the destination prefix to analyze | Test router | `show ip route eigrp` | Target remote EIGRP prefix appears as `D` or `D EX` |
| 5 | Display the EIGRP topology entry for the target prefix | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Output shows route state, successor count, FD, next hops, and metric values |
| 6 | Confirm the route is stable before judging feasibility | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Prefix state shows `Passive` |
| 7 | Identify the current successor path | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Best path shows the lowest FD and is counted as the successor |
| 8 | Record the successor feasible distance | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Successor path shows the local FD value |
| 9 | Record each alternate path reported distance | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Alternate path output shows RD as the second value in the metric pair |
| 10 | Apply the feasibility condition test | Test router | Manual comparison: `alternate RD < successor FD` | If true, the alternate path is a feasible successor |
| 11 | Display all known paths, including non-feasible paths | Test router | `show ip eigrp topology all-links` | Paths hidden from normal topology output appear if they failed feasibility condition |
| 12 | Compare normal topology output against all-links output | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` and `show ip eigrp topology all-links` | Normal output shows successors and feasible successors; all-links shows every known path |
| 13 | Confirm whether a backup path is actually feasible | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Backup path appears in normal topology output only if it is a feasible successor |
| 14 | Confirm the current successor is installed in the RIB | Test router | `show ip route <PREFIX>` | Routing table points to the expected successor next hop |
| 15 | Confirm the feasible successor is not installed by default | Test router | `show ip route <PREFIX>` | Only the best successor appears unless ECMP or variance is configured |
| 16 | Optionally tune interface delay to create a lab condition where a backup path becomes feasible | Lab router interface | `delay <TENS_OF_MICROSECONDS>` | EIGRP metric changes after topology recalculation |
| 17 | Verify FD/RD values after delay tuning | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Backup path now passes or fails feasibility condition according to RD and FD |
| 18 | Optionally verify non-feasible backup behavior before failure | Test router | `show ip eigrp topology all-links` | Backup path may be visible only in all-links output if it fails feasibility condition |
| 19 | Test local failover behavior when a feasible successor exists | Lab router/interface | `shutdown` on the current successor-facing interface | EIGRP promotes the feasible successor without a full query process |
| 20 | Verify the feasible successor became the new successor | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Former feasible successor becomes the successor path |
| 21 | Verify the new successor entered the RIB | Test router | `show ip route <PREFIX>` | Route now points to the backup next hop |
| 22 | Restore the failed successor-facing interface | Lab router/interface | `no shutdown` | Original link returns to `up/up` |
| 23 | Confirm EIGRP reconverges cleanly after restoration | Test router | `show ip eigrp neighbors` | Neighbor adjacency returns and `Q Cnt` remains `0` |
| 24 | Confirm final topology state | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Prefix returns to passive state with expected successor and feasible successor status |
| 25 | Confirm data-plane reachability | Test router/host | `ping <REMOTE_IP>` | Ping succeeds |
| 26 | Confirm forwarding path | Test router/host | `traceroute <REMOTE_IP>` | Path follows the expected current successor |
| 27 | Save the working lab configuration only if the metric changes are intentional | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Skeleton
! Baseline checks
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
! Pick one prefix and inspect normal topology output
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
! Inspect every known path, including paths that failed feasibility condition
show ip eigrp topology all-links
! Feasibility condition:
! Alternate path is a feasible successor only if:
! alternate RD < current successor FD
! Confirm current RIB winner
show ip route <PREFIX>
! Optional lab-only metric tuning to influence feasible successor behavior
configure terminal
 interface <INTERFACE>
  delay <TENS_OF_MICROSECONDS>
 end
! Recheck topology after metric change
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip eigrp topology all-links
show ip route <PREFIX>
! Optional failover test
configure terminal
 interface <CURRENT_SUCCESSOR_FACING_INTERFACE>
  shutdown
 end
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip route <PREFIX>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Restore interface
configure terminal
 interface <CURRENT_SUCCESSOR_FACING_INTERFACE>
  no shutdown
 end
! Final checks
show ip interface brief
show ip eigrp neighbors
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip route <PREFIX>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save only if intentional
write memory
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before EIGRP path testing | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency stability | Expected neighbors are present with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP AS, router ID, network statements, passive interfaces, K-values, and AD | Values match the intended lab design |
| `show ip route eigrp` | Lists EIGRP-installed routes | Internal routes appear as `D`; external routes appear as `D EX` |
| `show ip eigrp topology` | Displays successors and feasible successors | Expected prefixes appear in passive state |
| `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Shows detailed FD/RD and successor status for one prefix | Output shows successor count, FD, next hops, and metric pairs |
| `show ip eigrp topology all-links` | Displays all known EIGRP paths, including non-feasible paths | More paths may appear than in normal topology output |
| `show ip route <PREFIX>` | Confirms the actual installed route | RIB points to the current successor next hop |
| `show running-config interface <INTERFACE>` | Confirms lab metric tuning | Intended `delay` value is present only if deliberately configured |
| `show interfaces <INTERFACE>` | Confirms interface bandwidth and delay values used in metric calculation | Bandwidth and delay match the intended lab design |
| `ping <REMOTE_IP>` | Confirms data-plane reachability | Ping succeeds |
| `traceroute <REMOTE_IP>` | Confirms forwarding path | Path follows the expected successor |
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Rollback
! Remove lab-only delay tuning
configure terminal
 interface <INTERFACE>
  no delay
 end
! Remove lab-only bandwidth tuning if it was configured
configure terminal
 interface <INTERFACE>
  no bandwidth
 end
! Restore a shutdown interface used for failover testing
configure terminal
 interface <CURRENT_SUCCESSOR_FACING_INTERFACE>
  no shutdown
 end
! Verify rollback
show ip interface brief
show running-config interface <INTERFACE>
show ip eigrp neighbors
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip eigrp topology all-links
show ip route <PREFIX>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
write memory
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Expected backup path does not appear in normal topology output | Backup path failed feasibility condition | `show ip eigrp topology all-links` | Compare alternate RD against successor FD |
| Backup path appears in all-links only | RD is greater than or equal to successor FD | `show ip eigrp topology all-links` | Accept that it is not loop-free by EIGRP’s rule, or redesign metrics/topology |
| Backup path appears as feasible successor but not in routing table | Normal behavior without ECMP or variance | `show ip route <PREFIX>` | No fix needed unless load balancing is required |
| Feasible successor is not used until failure | Normal EIGRP behavior | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | No fix needed; feasible successor is a backup path |
| Route goes active after successor failure | No feasible successor existed | `show ip eigrp topology active` | Reduce query scope with summarization/stub design or adjust topology |
| EIGRP does not fail over quickly | Backup path was not a feasible successor | `show ip eigrp topology all-links` | Ensure a valid feasible successor exists before testing failure |
| Variance does not install backup path | Backup path is not a feasible successor | `show ip eigrp topology all-links` | Variance cannot override feasibility condition |
| Expected feasible successor disappeared after metric tuning | RD/FD relationship changed | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Recalculate and correct delay/bandwidth values |
| Multiple paths exist but only one is visible | Normal topology command hides non-feasible paths | `show ip eigrp topology all-links` | Use all-links for full path visibility |
| Prefix is missing entirely from topology table | Route is not being advertised into EIGRP | `show ip protocols` and `show ip eigrp topology` | Fix missing `network` statement, passive interface, or redistribution |
| Neighbor relationship drops during testing | Interface shutdown hit the only transit path or unstable link | `show ip eigrp neighbors` | Restore interface and validate physical/logical topology |
| Ping fails after failover | Return path missing or ACL/security policy blocks traffic | `show ip route <SOURCE_PREFIX>` on remote router | Add or restore reverse route through EIGRP |
| RIB still points to old path after failure | Interface did not actually go down or route is from another protocol | `show ip interface brief` and `show ip route <PREFIX>` | Confirm interface state and RIB owner |
# Index
# Source_Basis
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Mental_Model
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Configuration_Checklist
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Skeleton
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Verification_Commands
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Rollback
# EIGRP_Feasible_Successor_And_Feasibility_Condition_Failure_Checks
