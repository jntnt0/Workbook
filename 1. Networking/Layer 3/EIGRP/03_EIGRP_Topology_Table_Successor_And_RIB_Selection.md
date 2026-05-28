EIGRP_Topology_Table_Successor_And_RIB_Selection.md

EIGRP_Topology_Table_Successor_And_RIB_Selection

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `All_combined_part3.md`, Book 3: EIGRP for IP | Points EIGRP concepts, DUAL, topology table, feasible successors, metrics, and path selection to the main EIGRP source |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports EIGRP topology table, neighbor-learned routes, metrics, and successor logic |
| `All_combined_part3.md` | Book 3, Chapter 2: Advanced EIGRP Concepts, Data Structures and Algorithms | Supports DUAL, feasible distance, reported distance, feasible successors, and loop-free path selection |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Terminology | Defines successor route, successor, feasible distance, reported distance, feasibility condition, and feasible successor |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Topology Table | Supports `show ip eigrp topology`, `show ip eigrp topology all-links`, and the distinction between successor, feasible successor, and non-feasible backup paths |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Load Balancing | Supports `maximum-paths` and `variance` for equal-cost and unequal-cost EIGRP RIB installation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Troubleshooting Feasible Successors | Supports comparing FD, RD, successor count, all-links output, and RIB installation behavior |
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Topology table | EIGRP’s route-decision database; it can know paths that are not installed in the routing table |
| Routing table | The router’s actual forwarding decision table; only selected routes are installed here |
| Prefix state `P` | Passive means the route is stable and not currently recomputing |
| Prefix state `A` | Active means EIGRP is querying and recomputing a route |
| Successor route | The lowest-metric EIGRP path to a destination |
| Successor | The next-hop router used by the successor route |
| Feasible distance | The local total metric to reach the destination |
| Reported distance | The neighbor’s advertised distance to the destination |
| Feasibility condition | A backup path is loop-free only if its RD is lower than the successor FD |
| Feasible successor | A backup path that passed the feasibility condition |
| `show ip eigrp topology` | Shows successors and feasible successors by default |
| `show ip eigrp topology all-links` | Shows all known paths, including paths that failed the feasibility condition |
| RIB winner | The route actually installed in `show ip route`; EIGRP can lose to a better administrative distance route |
| ECMP | Equal-cost EIGRP successors can both be installed if `maximum-paths` allows it |
| Unequal-cost load balancing | Feasible successors can be installed if `variance` allows them |
| Dangerous shortcut | Do not assume a path is broken just because it is missing from `show ip route`; it may be in the topology table but not selected |
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are operational before analyzing path selection | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbor adjacencies are stable | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm the EIGRP process is advertising and receiving expected networks | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, and metric weights are shown |
| 4 | Confirm EIGRP has learned remote prefixes | All routers | `show ip eigrp topology` | Expected prefixes appear in the EIGRP topology table |
| 5 | Pick one remote prefix for path-selection analysis | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Output shows route state, successor count, FD, next hops, and composite metric values |
| 6 | Confirm the prefix is stable before interpreting the path | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Prefix state shows `Passive` |
| 7 | Identify the successor count for the prefix | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Output shows `1 Successor(s)` or multiple successors if ECMP exists |
| 8 | Identify the installed successor next hop | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Lowest-FD path is listed as the successor path |
| 9 | Read the FD/RD pair for each displayed path | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Composite metric appears as `(FD/RD)` for each path |
| 10 | Confirm whether displayed backup paths are feasible successors | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Backup path RD is lower than successor FD |
| 11 | Display paths that failed the feasibility condition | Test router | `show ip eigrp topology all-links` | All known paths appear, including paths not shown in normal topology output |
| 12 | Compare normal topology output against all-links output | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` then `show ip eigrp topology all-links` | Paths missing from normal output failed feasibility condition or are not eligible backups |
| 13 | Confirm the successor entered the routing table | Test router | `show ip route <PREFIX>` | RIB shows the expected EIGRP next hop and outgoing interface |
| 14 | Confirm all EIGRP-installed routes | Test router | `show ip route eigrp` | EIGRP internal routes appear as `D`; external routes appear as `D EX` |
| 15 | Check whether EIGRP lost RIB ownership to another protocol | Test router | `show ip route <PREFIX>` | If another protocol owns the route, compare administrative distance and metric |
| 16 | Confirm EIGRP administrative distance values | Test router | `show ip protocols` | Internal EIGRP AD is typically `90`; external EIGRP AD is typically `170` |
| 17 | Confirm metric inputs for the selected path | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Minimum bandwidth, total delay, reliability, load, MTU, and hop count are visible |
| 18 | Confirm interface metric values if path selection looks wrong | Test router | `show interfaces <INTERFACE>` | Bandwidth and delay match the intended lab design |
| 19 | Optionally tune interface delay to influence EIGRP path selection in a lab | Router interface | `delay <TENS_OF_MICROSECONDS>` | EIGRP recalculates metric using the changed delay value |
| 20 | Avoid changing interface bandwidth unless the lab specifically requires it | Router interface | `bandwidth <KBPS>` | Bandwidth value changes only if intentionally used for metric simulation |
| 21 | Verify topology recalculation after metric tuning | Test router | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | FD and path ordering reflect the intended metric change |
| 22 | Verify the routing table changed after metric tuning | Test router | `show ip route <PREFIX>` | Installed next hop matches the intended successor |
| 23 | Configure ECMP path count in classic mode if equal-cost successors should install | Classic EIGRP router | `router eigrp <AS_NUMBER>` then `maximum-paths <NUMBER>` | Multiple equal-cost EIGRP paths can be installed up to the configured limit |
| 24 | Configure ECMP path count in named mode if equal-cost successors should install | Named-mode EIGRP router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `topology base` then `maximum-paths <NUMBER>` | Multiple equal-cost EIGRP paths can be installed up to the configured limit |
| 25 | Verify equal-cost EIGRP RIB installation | Test router | `show ip route <PREFIX>` | Multiple equal-cost EIGRP next hops appear for the same prefix |
| 26 | Configure variance in classic mode if feasible successors should be installed for unequal-cost load balancing | Classic EIGRP router | `router eigrp <AS_NUMBER>` then `variance <MULTIPLIER>` | Eligible feasible successors below the variance threshold can enter the RIB |
| 27 | Configure variance in named mode if feasible successors should be installed for unequal-cost load balancing | Named-mode EIGRP router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `topology base` then `variance <MULTIPLIER>` | Eligible feasible successors below the variance threshold can enter the RIB |
| 28 | Verify unequal-cost EIGRP RIB installation | Test router | `show ip route <PREFIX>` | Multiple EIGRP next hops appear with different metrics only if feasible and within variance |
| 29 | Confirm forwarding path after final selection | Test router | `traceroute <REMOTE_IP>` | Traffic follows the expected successor or load-balanced path |
| 30 | Confirm data-plane reachability | Test router | `ping <REMOTE_IP>` | Ping succeeds across the selected EIGRP path |
| 31 | Save the working configuration | All changed routers | `write memory` | Configuration is saved |
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Skeleton
! Baseline checks
show ip interface brief
show ip eigrp neighbors
show ip protocols
! Topology table inspection
show ip eigrp topology
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip eigrp topology all-links
! RIB comparison
show ip route <PREFIX>
show ip route eigrp
show ip protocols
! Metric input inspection
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show interfaces <INTERFACE>
! Optional lab-only metric influence
configure terminal
 interface <INTERFACE>
  delay <TENS_OF_MICROSECONDS>
 end
! Optional classic-mode ECMP control
configure terminal
 router eigrp <AS_NUMBER>
  maximum-paths <NUMBER>
 end
! Optional named-mode ECMP control
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    maximum-paths <NUMBER>
   exit-af-topology
  exit-address-family
 end
! Optional classic-mode unequal-cost load balancing
configure terminal
 router eigrp <AS_NUMBER>
  variance <MULTIPLIER>
 end
! Optional named-mode unequal-cost load balancing
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    variance <MULTIPLIER>
   exit-af-topology
  exit-address-family
 end
! Final verification
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip route <PREFIX>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before EIGRP path analysis | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms adjacency stability | Expected neighbors are present with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP AS, router ID, network statements, passive interfaces, metric weights, and AD | Values match the intended lab design |
| `show ip eigrp topology` | Displays EIGRP successors and feasible successors | Prefixes appear in passive state with successor count and FD |
| `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Displays detailed path-selection data for one prefix | Shows state, successor count, FD, next hops, composite metric, RD, and metric vectors |
| `show ip eigrp topology all-links` | Displays all known paths, including paths that failed feasibility condition | Additional non-feasible paths may appear compared to normal topology output |
| `show ip route <PREFIX>` | Confirms the actual RIB winner | Installed route points to expected next hop and outgoing interface |
| `show ip route eigrp` | Displays all EIGRP-installed routes | Internal routes appear as `D`; external routes appear as `D EX` |
| `show interfaces <INTERFACE>` | Confirms bandwidth and delay values affecting metric calculation | Bandwidth and delay match the intended lab values |
| `show running-config interface <INTERFACE>` | Confirms configured interface metric overrides | Intended `delay` or `bandwidth` commands are present if configured |
| `show running-config | section router eigrp` | Confirms EIGRP selection-related settings | Shows `maximum-paths`, `variance`, address-family, and topology base settings where applicable |
| `ping <REMOTE_IP>` | Confirms end-to-end reachability | Ping succeeds |
| `traceroute <REMOTE_IP>` | Confirms forwarding path | Path follows the expected successor or load-balanced next hops |
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Rollback
! Roll back lab-only interface delay tuning
configure terminal
 interface <INTERFACE>
  no delay
 end
! Roll back lab-only interface bandwidth tuning if configured
configure terminal
 interface <INTERFACE>
  no bandwidth
 end
! Roll back classic-mode ECMP change
configure terminal
 router eigrp <AS_NUMBER>
  no maximum-paths
 end
! Roll back named-mode ECMP change
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no maximum-paths
   exit-af-topology
  exit-address-family
 end
! Roll back classic-mode variance
configure terminal
 router eigrp <AS_NUMBER>
  no variance
 end
! Roll back named-mode variance
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no variance
   exit-af-topology
  exit-address-family
 end
! Verify rollback
show running-config | section router eigrp
show running-config interface <INTERFACE>
show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>
show ip route <PREFIX>
write memory
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Prefix missing from topology table | Network is not being advertised into EIGRP | `show ip protocols` and `show ip eigrp topology` | Add or correct the `network` statement on the originating router |
| Prefix appears in topology table but not routing table | Another protocol has better administrative distance | `show ip route <PREFIX>` | Confirm RIB winner; adjust design only if EIGRP should own the route |
| Prefix appears in topology table but only one path is shown | Other paths failed feasibility condition | `show ip eigrp topology all-links` | Compare RD of backup path to FD of successor |
| Backup path does not become feasible successor | Backup path RD is not lower than successor FD | `show ip eigrp topology all-links` | Redesign path metrics or topology; do not force unsafe paths |
| Expected equal-cost routes do not both install | Metrics are not actually equal | `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Compare FD values and correct interface delay/bandwidth if lab design expects ECMP |
| Equal-cost routes exist but not all install | `maximum-paths` is too low | `show running-config | section router eigrp` | Increase `maximum-paths` under classic process or named-mode `topology base` |
| Feasible successor exists but does not install in RIB | Variance is not configured or too low | `show running-config | section router eigrp` | Configure appropriate `variance <MULTIPLIER>` |
| Variance configured but backup path still does not install | Path is not a feasible successor | `show ip eigrp topology all-links` | Only feasible successors can be installed with variance |
| Route is stuck active | No feasible successor and query process has not completed | `show ip eigrp topology active` | Fix missing query replies, broken neighbors, or excessive query scope |
| Path selection changes unexpectedly | Interface delay or bandwidth was changed | `show running-config interface <INTERFACE>` | Remove unintended `delay` or `bandwidth` changes |
| Neighbor resets after metric changes | K values were changed inconsistently | `show ip protocols | include K` | Restore matching K values across EIGRP neighbors |
| Traffic follows unexpected path even though topology looks correct | RIB has a different winner or load balancing is active | `show ip route <PREFIX>` | Verify actual installed next hop, AD, metric, and ECMP behavior |
| Ping fails but EIGRP route exists | Return route missing | `show ip route <SOURCE_PREFIX>` on the remote side | Ensure reverse path exists through EIGRP or another valid route |
| Traceroute does not match expected successor | Per-packet or per-destination load balancing may be occurring | `show ip route <PREFIX>` | Confirm multiple next hops and expected platform forwarding behavior |
# Index
# Source_Basis
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Mental_Model
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Configuration_Checklist
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Skeleton
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Verification_Commands
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Rollback
# EIGRP_Topology_Table_Successor_And_RIB_Selection_Failure_Checks
