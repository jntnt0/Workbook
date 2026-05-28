EIGRP_Graceful_Shutdown_And_Neighbor_Teardown.md

EIGRP_Graceful_Shutdown_And_Neighbor_Teardown

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP DUAL behavior, K-values, neighbor relationships, convergence, and troubleshooting to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports EIGRP hello packets, K-values, DUAL, neighbor table behavior, topology table behavior, and route convergence |
| `All_combined_part3.md` | Book 3, Chapter 2: Advanced EIGRP Concepts, Data Structures and Algorithms | Supports active/passive state behavior, query/reply convergence, and successor recalculation after route loss |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Troubleshooting | Supports neighbor-down causes including interface shutdown, K-value mismatch, route loss, and adjacency verification |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | EIGRP Goodbye Message | Supports goodbye message behavior when an EIGRP process is shut down and the neighbor log message `Interface Goodbye received` |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | Mismatched K Values | Supports the caveat that older or nonsupporting peers may interpret peer-termination/goodbye behavior as a K-value mismatch |
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Graceful shutdown | EIGRP intentionally tells neighbors that the adjacency is being torn down instead of silently disappearing |
| Goodbye message | EIGRP hello-style notification sent when an EIGRP process or EIGRP-enabled interface is shut down |
| Peer termination | Neighbor teardown event triggered by a received EIGRP goodbye/termination message |
| Interface Goodbye received | Log reason showing the neighbor went down because the peer intentionally shut down EIGRP on that path |
| Hold timer expiration | Ungraceful failure behavior where the neighbor disappears only after hellos are missed long enough |
| K-value 255 behavior | Goodbye/peer-termination signaling can be interpreted by older peers as a K-value mismatch |
| Planned teardown | Admin removes EIGRP from a link, shuts an interface, removes a network statement, makes an interface passive, or removes the EIGRP process |
| Unplanned teardown | Interface failure, packet loss, CPU issue, authentication mismatch, K-value mismatch, or reachability problem |
| DUAL reaction | Neighbor loss triggers route recalculation; if a feasible successor exists, failover is local and fast |
| Active route risk | If no feasible successor exists, EIGRP may go active and query neighbors |
| Maintenance workflow | Verify baseline, remove EIGRP intentionally, confirm goodbye/neighbor down reason, verify alternate route, then restore and validate |
| Operational warning | Do not confuse a clean `Interface Goodbye received` during a planned change with random EIGRP instability |
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are operational before teardown testing | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable before planned shutdown | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm EIGRP process state before teardown | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Record baseline EIGRP routes | All affected routers | `show ip route eigrp` | Current EIGRP-learned routes are documented |
| 5 | Record the exact route that should change during teardown | Test router | `show ip route <PREFIX>` | Prefix points through the neighbor that will be torn down |
| 6 | Record topology state before teardown | Test router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Prefix is passive and shows current successor and possible feasible successor |
| 7 | Confirm whether a feasible successor exists before teardown | Test router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Backup path appears if EIGRP can fail over locally |
| 8 | Confirm no routes are active before maintenance | All affected routers | `show ip eigrp topology active` | No active prefixes are listed |
| 9 | Enable neighbor change logging if not already enabled | All affected routers | `router eigrp <AS_NUMBER>` then `eigrp log-neighbor-changes` | Neighbor-up/down events are logged |
| 10 | Confirm logging is active before teardown | All affected routers | `show running-config | section router eigrp` | `eigrp log-neighbor-changes` appears if manually configured |
| 11 | Choose the teardown method | Maintenance router | Planning step | Pick interface shutdown, EIGRP process shutdown/removal, network statement removal, passive-interface, or named-mode shutdown |
| 12 | Enter global configuration mode for interface shutdown test | Maintenance router | `configure terminal` | Router enters global configuration mode |
| 13 | Shut the EIGRP-facing interface for a clean interface-based teardown test | Maintenance router | `interface <EIGRP_INTERFACE>` then `shutdown` | Interface goes administratively down and EIGRP neighbor drops |
| 14 | Verify neighbor teardown from the remote side | Remote neighbor | `show ip eigrp neighbors` | Neighbor entry for the maintenance router is removed |
| 15 | Verify goodbye/peer-termination logging from the remote side | Remote neighbor | `show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value` | Log shows `Interface Goodbye received`, peer termination, or related neighbor-down reason |
| 16 | Verify the local interface is down if interface shutdown was used | Maintenance router | `show ip interface brief` | Selected interface shows `administratively down/down` |
| 17 | Verify the affected route reconverges | Remote neighbor | `show ip route <PREFIX>` | Route is removed or replaced by an alternate path |
| 18 | Verify DUAL did not remain stuck active | All affected routers | `show ip eigrp topology active` | No route remains active after convergence |
| 19 | Restore the interface after interface-based test | Maintenance router | `interface <EIGRP_INTERFACE>` then `no shutdown` | Interface returns to `up/up` |
| 20 | Verify neighbor reforms after restoration | Both routers | `show ip eigrp neighbors` | Neighbor returns with stable uptime and `Q Cnt` of `0` |
| 21 | Enter classic EIGRP mode for process-removal teardown test | Maintenance router | `router eigrp <AS_NUMBER>` | Router enters EIGRP router configuration mode |
| 22 | Remove a specific network statement to tear down EIGRP only on matched interfaces | Maintenance router | `no network <NETWORK> <WILDCARD_MASK>` | Interfaces matching that network no longer run EIGRP |
| 23 | Verify EIGRP interface removal | Maintenance router | `show ip eigrp interfaces` | Removed interface no longer appears as an EIGRP interface |
| 24 | Verify remote neighbor teardown after network statement removal | Remote neighbor | `show ip eigrp neighbors` | Neighbor adjacency over that link is removed |
| 25 | Restore the network statement after the test | Maintenance router | `network <NETWORK> <WILDCARD_MASK>` | Interface is added back to EIGRP |
| 26 | Verify neighbor reforms after restoring the network statement | Both routers | `show ip eigrp neighbors` | Neighbor returns with `Q Cnt` of `0` |
| 27 | Use passive-interface if the goal is to stop adjacency while still advertising the connected network | Maintenance router | `router eigrp <AS_NUMBER>` then `passive-interface <EIGRP_INTERFACE>` | Interface stops sending hellos and neighbor drops |
| 28 | Verify passive-interface behavior | Maintenance router | `show ip protocols` | Interface is listed as passive |
| 29 | Restore adjacency after passive-interface test | Maintenance router | `router eigrp <AS_NUMBER>` then `no passive-interface <EIGRP_INTERFACE>` | Interface resumes EIGRP hellos |
| 30 | Remove the entire classic EIGRP process only if full process teardown is intended | Maintenance router | `no router eigrp <AS_NUMBER>` | EIGRP process is removed and all EIGRP adjacencies on that process drop |
| 31 | Verify full process teardown | Maintenance router | `show running-config | section router eigrp` | Classic EIGRP process is absent |
| 32 | Restore classic EIGRP process after full teardown test | Maintenance router | `router eigrp <AS_NUMBER>` plus original `network`, `router-id`, and passive settings | EIGRP process is rebuilt |
| 33 | Use named-mode address-family shutdown if supported and full named EIGRP AF shutdown is intended | Maintenance router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `shutdown` | Named EIGRP address family stops participating |
| 34 | Verify named-mode address-family shutdown | Maintenance router | `show running-config | section router eigrp` | Address family shows shutdown state if supported |
| 35 | Restore named-mode address-family after shutdown test | Maintenance router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `no shutdown` | Named EIGRP address family resumes operation |
| 36 | Verify neighbors after named-mode restore | All affected routers | `show ip eigrp neighbors` | Expected neighbors return |
| 37 | Verify routes after restore | All affected routers | `show ip route eigrp` | Expected EIGRP routes return |
| 38 | Verify topology stability after restore | All affected routers | `show ip eigrp topology` | Expected prefixes are passive with valid successors |
| 39 | Check logs for planned vs unplanned neighbor-down reasons | All affected routers | `show logging | include EIGRP|DUAL|Goodbye|PEER|K-value|hold|retry|SIA` | Planned teardown shows goodbye/peer-termination; unplanned failures show hold timer, retry limit, SIA, auth, or real K-value mismatch |
| 40 | Test end-to-end reachability after restore | Edge router or host | `ping <REMOTE_IP>` | Ping succeeds |
| 41 | Verify forwarding path after restore | Edge router or host | `traceroute <REMOTE_IP>` | Path follows expected EIGRP next hops |
| 42 | Save the restored stable configuration | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip route <PREFIX>
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology active
show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value
! Optional: ensure neighbor changes are logged
configure terminal
 router eigrp <AS_NUMBER>
  eigrp log-neighbor-changes
 end
! Method 1: interface-based graceful teardown
configure terminal
 interface <EIGRP_INTERFACE>
  shutdown
 end
show ip interface brief
show ip eigrp neighbors
show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value
show ip route <PREFIX>
show ip eigrp topology active
! Restore interface
configure terminal
 interface <EIGRP_INTERFACE>
  no shutdown
 end
show ip eigrp neighbors
show ip route eigrp
show ip eigrp topology active
! Method 2: remove EIGRP from a specific interface by removing network statement
configure terminal
 router eigrp <AS_NUMBER>
  no network <NETWORK> <WILDCARD_MASK>
 end
show ip eigrp interfaces
show ip eigrp neighbors
show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value
! Restore network statement
configure terminal
 router eigrp <AS_NUMBER>
  network <NETWORK> <WILDCARD_MASK>
 end
show ip eigrp neighbors
show ip route eigrp
! Method 3: stop neighbor formation but keep local network advertisement
configure terminal
 router eigrp <AS_NUMBER>
  passive-interface <EIGRP_INTERFACE>
 end
show ip protocols
show ip eigrp neighbors
! Restore neighbor formation
configure terminal
 router eigrp <AS_NUMBER>
  no passive-interface <EIGRP_INTERFACE>
 end
show ip eigrp neighbors
! Method 4: full classic EIGRP process teardown
configure terminal
 no router eigrp <AS_NUMBER>
end
show running-config | section router eigrp
show ip eigrp neighbors
show ip route eigrp
! Restore classic EIGRP process from documented baseline
configure terminal
 router eigrp <AS_NUMBER>
  eigrp router-id <ROUTER_ID>
  no auto-summary
  network <NETWORK_1> <WILDCARD_1>
  network <NETWORK_2> <WILDCARD_2>
  passive-interface default
  no passive-interface <EIGRP_INTERFACE>
 end
! Method 5: named-mode address-family shutdown if supported
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   shutdown
  exit-address-family
 end
show running-config | section router eigrp
show ip eigrp neighbors
show ip route eigrp
! Restore named-mode address-family
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   no shutdown
  exit-address-family
 end
! Final verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
show ip eigrp topology active
show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value|hold|retry|SIA
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before and after teardown | Interfaces are `up/up` before maintenance and restored to `up/up` afterward |
| `show ip eigrp neighbors` | Confirms neighbor state | Neighbor disappears during planned teardown and returns after restore |
| `show ip protocols` | Confirms process, network, passive-interface, K-values, AD, and timer state | Values match the intended lab baseline |
| `show ip eigrp interfaces` | Confirms which interfaces are running EIGRP | Removed or passive interfaces are reflected correctly |
| `show ip eigrp topology` | Confirms topology table after convergence | Expected prefixes are present and passive |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Confirms route recalculation for one prefix | Successor changes or route removal matches expected teardown behavior |
| `show ip eigrp topology active` | Confirms DUAL did not get stuck after teardown | No active routes remain after convergence |
| `show ip route eigrp` | Confirms EIGRP route installation | Routes disappear during teardown if no alternate path exists and return after restore |
| `show ip route <PREFIX>` | Confirms exact RIB result for affected route | Route points to alternate path, is removed, or returns as expected |
| `show running-config | section router eigrp` | Confirms process, passive-interface, network statements, and named-mode shutdown state | Config matches intended teardown or restored baseline |
| `show running-config interface <EIGRP_INTERFACE>` | Confirms interface shutdown state | Interface shows `shutdown` during test and no shutdown after restore |
| `show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value` | Confirms graceful shutdown/goodbye behavior | Planned teardown shows goodbye or peer-termination style neighbor-down reason |
| `show logging | include hold|retry|SIA|auth|K-value` | Checks for unplanned instability causes | No recurring hold timer, retry limit, SIA, authentication, or real K-value mismatch messages |
| `debug eigrp packets` | Lab-only packet observation | Goodbye/termination behavior may be visible during planned shutdown |
| `debug eigrp fsm` | Lab-only DUAL behavior observation | DUAL recalculation can be observed during neighbor loss |
| `undebug all` | Stops debugging | All debugging is disabled |
| `ping <REMOTE_IP>` | Confirms restored reachability | Ping succeeds after restore |
| `traceroute <REMOTE_IP>` | Confirms restored forwarding path | Path follows expected EIGRP next hops |
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Rollback
! Restore an interface shut down during maintenance
configure terminal
 interface <EIGRP_INTERFACE>
  no shutdown
 end
! Restore a removed classic EIGRP network statement
configure terminal
 router eigrp <AS_NUMBER>
  network <NETWORK> <WILDCARD_MASK>
 end
! Restore neighbor formation after passive-interface test
configure terminal
 router eigrp <AS_NUMBER>
  no passive-interface <EIGRP_INTERFACE>
 end
! Restore full classic EIGRP process if it was removed
configure terminal
 router eigrp <AS_NUMBER>
  eigrp router-id <ROUTER_ID>
  no auto-summary
  network <NETWORK_1> <WILDCARD_1>
  network <NETWORK_2> <WILDCARD_2>
  passive-interface default
  no passive-interface <EIGRP_INTERFACE>
 end
! Restore named-mode address family if it was shut down
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   no shutdown
  exit-address-family
 end
! Stop any lab debugging
undebug all
! Verify rollback
show ip interface brief
show running-config | section router eigrp
show running-config interface <EIGRP_INTERFACE>
show ip eigrp interfaces
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
show ip eigrp topology active
show logging | include EIGRP|DUAL|Goodbye|PEER|NBRCHANGE|K-value|hold|retry|SIA
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Expected result:
! EIGRP interface or process is restored.
! Neighbor adjacency returns.
! EIGRP routes return.
! No active routes remain.
! End-to-end reachability succeeds.
write memory
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Neighbor drops with `Interface Goodbye received` during planned change | Normal graceful shutdown behavior | `show logging | include Goodbye|NBRCHANGE` | No fix needed if the teardown was intentional |
| Neighbor drops with `PEER-TERMINATION received` during planned change | Peer intentionally terminated EIGRP adjacency | `show logging | include PEER|NBRCHANGE` | No fix needed if the teardown was intentional |
| Neighbor drops with `K-value mismatch` during planned shutdown | Older or nonsupporting peer interpreted goodbye behavior as K-value mismatch | `show logging | include K-value|Goodbye` | Confirm whether this happened during process/interface shutdown; verify K-values if not planned |
| Neighbor drops with `K-value mismatch` outside maintenance | Real K-value mismatch | `show ip protocols | include K` | Match K-values on both neighbors |
| Neighbor does not return after interface restore | Interface still down or wrong cable/link state | `show ip interface brief` | Restore physical/CML link and configure `no shutdown` |
| Neighbor does not return after network statement restore | Network statement does not match interface IP | `show ip protocols` and `show ip eigrp interfaces` | Correct `network <NETWORK> <WILDCARD_MASK>` |
| Neighbor does not return after passive-interface rollback | Interface still passive | `show ip protocols` | Configure `no passive-interface <EIGRP_INTERFACE>` |
| Neighbor does not return after process restore | Missing router ID, network statement, authentication, or passive-interface setting | `show running-config | section router eigrp` | Rebuild the original EIGRP baseline |
| Named-mode EIGRP does not return | Address-family still shutdown or config placed in wrong hierarchy | `show running-config | section router eigrp` | Configure `no shutdown` under the correct address family |
| Routes disappear and do not return | EIGRP adjacency did not reform or route source missing | `show ip eigrp neighbors` and `show ip route eigrp` | Restore neighbor first, then verify advertised networks |
| Route goes active after teardown | No feasible successor exists and DUAL is querying | `show ip eigrp topology active` | Wait for convergence; troubleshoot missing replies if active state persists |
| SIA appears after teardown | Query reply did not return before active timer expired | `show logging | include SIA|DUAL` | Fix unstable neighbor, packet loss, or excessive query scope |
| Traffic fails during planned teardown | No alternate path exists | `show ip route <PREFIX>` | Confirm redundancy before maintenance or accept expected outage |
| Traffic fails after restore | Return path missing or route not installed | Remote `show ip route <SOURCE_PREFIX>` | Fix reverse route or missing advertisement |
| Logs show hold timer expired instead of goodbye | Peer disappeared without graceful signaling or packets were lost | `show logging | include hold|NBRCHANGE` | Check interface failure, packet loss, CPU, congestion, or one-way link |
| Logs show retry limit exceeded | EIGRP reliable packet delivery failed | `show logging | include retry|EIGRP` | Check link drops, MTU, congestion, CPU, or unstable neighbor |
| Authentication errors appear after restore | Auth config missing or mismatched | `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Match authentication method and secret/key-chain on both sides |
| Debug output overwhelms the router | Debugging left enabled | `show debugging` | Run `undebug all` |
| Neighbor flaps repeatedly after restore | Real instability, not graceful shutdown | `show logging | include EIGRP|DUAL|NBRCHANGE|hold|retry|SIA` | Troubleshoot link health, timers, CPU, MTU, authentication, K-values, and packet loss |
# Index
# Source_Basis
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Mental_Model
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Configuration_Checklist
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Skeleton
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Verification_Commands
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Rollback
# EIGRP_Graceful_Shutdown_And_Neighbor_Teardown_Failure_Checks
