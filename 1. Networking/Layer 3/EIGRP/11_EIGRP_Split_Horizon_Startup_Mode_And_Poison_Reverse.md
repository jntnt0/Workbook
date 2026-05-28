EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse.md

EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP loop prevention, DUAL behavior, split horizon, and convergence behavior to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports poison reverse behavior, poison updates, and DUAL route propagation behavior |
| `All_combined_part3.md` | Book 3, Chapter 13: Running EIGRP over Various Switched WAN Technologies | Supports EIGRP split-horizon behavior on hub-and-spoke switched WAN designs |
| `All_combined_part3.md` | Table 13-11: Configuring EIGRP Split Horizon | Supports classic interface syntax: `no ip split-horizon eigrp <as-number>` and `ip split-horizon eigrp <as-number>` |
| `All_combined_part3.md` | EIGRP Split Horizon Warning | Confirms changing EIGRP split horizon resets adjacencies on that interface |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Split Horizon | Supports the rule that EIGRP split horizon prevents routes learned inbound on an interface from being advertised back out that same interface |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 3-18: Configuration to Disable Split Horizon | Supports classic syntax and named-mode syntax: `no split-horizon` under `af-interface` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 4-26: Verifying Whether Split Horizon Is Enabled for EIGRP on an Interface | Supports verification with `show ip eigrp interfaces detail <interface>` |
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Split horizon | EIGRP does not advertise a route back out the same interface on which it was learned |
| EIGRP-specific split horizon | Controlled with `ip split-horizon eigrp <AS>` or `no ip split-horizon eigrp <AS>` |
| Generic IP split horizon | Controlled with `ip split-horizon`; this is not the same as EIGRP split horizon |
| Hub-and-spoke issue | On a multipoint hub interface, routes learned from one spoke must sometimes be advertised back out the same hub interface to other spokes |
| Correct disable location | Disable EIGRP split horizon on the hub multipoint interface, not on every spoke |
| Wrong disable location | Disabling split horizon on spokes usually increases route churn and memory use without fixing the real hub advertisement problem |
| Startup mode | Initial EIGRP adjacency synchronization behavior where routes are exchanged and loop-prevention rules are applied during startup convergence |
| Poison reverse | EIGRP may advertise a route back toward the successor with an infinite metric to prevent loops |
| Poison update | An EIGRP update that marks a route unreachable, commonly by setting the metric to infinity |
| DUAL loop prevention | Split horizon, poison reverse, feasible successors, and query/reply logic all protect against loops |
| Verification truth source | Use `show ip eigrp interfaces detail`, not only `show ip interface`, to verify EIGRP-specific split horizon state |
| Adjacency reset warning | Changing EIGRP split horizon on an interface resets EIGRP neighbors reachable over that interface |
| Operational rule | Do not disable split horizon just because you can; only do it when the topology requires same-interface re-advertisement |
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable before changing split horizon | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm EIGRP process state | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Identify whether the topology is hub-and-spoke or full mesh | All routers | Topology review | Hub, spokes, and shared multipoint interface are clearly identified |
| 5 | Identify the hub multipoint interface | Hub router | `show ip eigrp neighbors` | Multiple spokes are reachable over the same hub interface |
| 6 | Confirm the missing route symptom before changing anything | Spoke routers | `show ip route eigrp` | Spokes are missing other spoke prefixes but still know hub prefixes |
| 7 | Confirm route exists on the hub | Hub router | `show ip route <SPOKE_PREFIX>` | Hub has the spoke route that should be advertised to other spokes |
| 8 | Confirm route was learned inbound on the same hub interface used toward other spokes | Hub router | `show ip route <SPOKE_PREFIX>` | Route next hop and outgoing interface point to the hub multipoint interface |
| 9 | Verify current EIGRP split-horizon state on the hub interface | Hub router | `show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>` | Output shows `Split-horizon is enabled` before the fix |
| 10 | Enter global configuration mode | Hub router | `configure terminal` | Router enters global configuration mode |
| 11 | Enter the hub multipoint interface | Hub router | `interface <HUB_MULTIPOINT_INTERFACE>` | Router enters interface configuration mode |
| 12 | Disable EIGRP split horizon in classic mode | Hub router | `no ip split-horizon eigrp <AS_NUMBER>` | EIGRP can advertise routes back out the same hub interface |
| 13 | Use named-mode syntax instead if the router uses named EIGRP | Hub router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `af-interface <HUB_MULTIPOINT_INTERFACE>` then `no split-horizon` | Named-mode EIGRP split horizon is disabled on the hub interface |
| 14 | Exit configuration mode | Hub router | `end` | Router returns to privileged EXEC mode |
| 15 | Expect neighbor resynchronization after the change | Hub router | `show logging | include EIGRP|DUAL|split horizon|NBRCHANGE` | Logs may show EIGRP neighbors resync because split horizon changed |
| 16 | Verify EIGRP split horizon is disabled on the hub interface | Hub router | `show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>` | Output shows `Split-horizon is disabled` |
| 17 | Verify hub neighbors recover | Hub router | `show ip eigrp neighbors` | Spoke neighbors return with stable uptime and `Q Cnt` of `0` |
| 18 | Verify spoke-to-spoke routes are now learned | Spoke routers | `show ip route eigrp` | Spokes now learn other spoke prefixes through EIGRP |
| 19 | Verify the exact route on a spoke | Spoke router | `show ip route <REMOTE_SPOKE_PREFIX>` | Route points toward the hub as the next hop |
| 20 | Verify the EIGRP topology entry on the spoke | Spoke router | `show ip eigrp topology <REMOTE_SPOKE_PREFIX>/<LENGTH>` | Prefix is passive and has a successor through the hub |
| 21 | Test spoke-to-spoke reachability | Spoke router | `ping <REMOTE_SPOKE_IP>` | Ping succeeds |
| 22 | Verify spoke-to-spoke forwarding path | Spoke router | `traceroute <REMOTE_SPOKE_IP>` | Path goes through the hub unless a shortcut mechanism exists |
| 23 | Confirm split horizon remains enabled on spoke interfaces unless design explicitly requires otherwise | Spoke routers | `show ip eigrp interfaces detail <SPOKE_INTERFACE>` | Spoke interface should normally show `Split-horizon is enabled` |
| 24 | Observe startup route exchange only in a lab if needed | Lab routers | `debug eigrp packets` | EIGRP update/query/reply behavior is visible during adjacency startup |
| 25 | Observe DUAL state transitions only in a lab if needed | Lab routers | `debug eigrp fsm` | DUAL passive/active and route calculation events are visible |
| 26 | Clear or flap an EIGRP adjacency only in a lab to observe startup behavior | Lab router | `clear ip eigrp neighbors` | Neighbor adjacency restarts and initial updates are exchanged |
| 27 | Check for poison reverse or poison update behavior during convergence | Lab router | `show logging | include DUAL|EIGRP|poison|unreachable` | Logs/debug may show route poison or DUAL route transition behavior |
| 28 | Disable debugging immediately after observation | Lab routers | `undebug all` | All debugging is stopped |
| 29 | Verify no route remains active after convergence | All affected routers | `show ip eigrp topology active` | No active routes remain |
| 30 | Verify final route table stability | All affected routers | `show ip route eigrp` | Expected EIGRP routes remain installed |
| 31 | Save the configuration if the split-horizon change is correct | Hub router | `write memory` | Configuration is saved |
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Skeleton
! Baseline checks
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
! Confirm hub-and-spoke symptom
show ip route <REMOTE_SPOKE_PREFIX>
show ip eigrp topology <REMOTE_SPOKE_PREFIX>/<LENGTH>
show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>
! Classic mode: disable EIGRP split horizon on the hub multipoint interface only
configure terminal
 interface <HUB_MULTIPOINT_INTERFACE>
  no ip split-horizon eigrp <AS_NUMBER>
 end
! Named mode: disable EIGRP split horizon on the hub multipoint interface only
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <HUB_MULTIPOINT_INTERFACE>
    no split-horizon
   exit-af-interface
  exit-address-family
 end
! Verify split-horizon state and neighbor recovery
show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>
show ip eigrp neighbors
show logging | include EIGRP|DUAL|split horizon|NBRCHANGE
show ip route eigrp
show ip route <REMOTE_SPOKE_PREFIX>
show ip eigrp topology <REMOTE_SPOKE_PREFIX>/<LENGTH>
! Spoke-to-spoke testing
ping <REMOTE_SPOKE_IP>
traceroute <REMOTE_SPOKE_IP>
! Lab-only startup and poison behavior observation
debug eigrp packets
debug eigrp fsm
clear ip eigrp neighbors
show ip eigrp neighbors
show ip eigrp topology active
show logging | include DUAL|EIGRP|poison|unreachable
undebug all
! Final checks
show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>
show ip eigrp neighbors
show ip eigrp topology active
show ip route eigrp
ping <REMOTE_SPOKE_IP>
traceroute <REMOTE_SPOKE_IP>
! Save
write memory
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before EIGRP split-horizon changes | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency health | Expected neighbors are present with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP AS, router ID, network statements, passive interfaces, K-values, and AD values | EIGRP process matches intended design |
| `show ip route eigrp` | Confirms EIGRP-learned routes | Spokes learn other spoke prefixes after hub split horizon is disabled |
| `show ip route <REMOTE_SPOKE_PREFIX>` | Confirms exact RIB result | Route points toward the expected hub next hop |
| `show ip eigrp topology <REMOTE_SPOKE_PREFIX>/<LENGTH>` | Confirms EIGRP topology state | Prefix is passive and has a valid successor |
| `show ip eigrp topology active` | Confirms no DUAL computation is stuck | No active routes remain after convergence |
| `show ip eigrp interfaces detail <INTERFACE>` | Confirms EIGRP-specific split-horizon state | Hub multipoint interface shows `Split-horizon is disabled` if intentionally changed |
| `show ip interface <INTERFACE>` | Shows generic interface behavior | May still show generic split horizon enabled even when EIGRP-specific split horizon is disabled |
| `show running-config interface <INTERFACE>` | Confirms classic interface command | Shows `no ip split-horizon eigrp <AS_NUMBER>` on the hub interface |
| `show running-config | section router eigrp` | Confirms named-mode command placement | Shows `af-interface <INTERFACE>` and `no split-horizon` under the correct address family |
| `show logging | include EIGRP|DUAL|split horizon|NBRCHANGE` | Checks adjacency reset and DUAL events | One-time resync may appear after the change; repeated resets are not healthy |
| `debug eigrp packets` | Lab-only observation of EIGRP packet exchange | Updates, queries, replies, and convergence packets can be observed |
| `debug eigrp fsm` | Lab-only observation of DUAL state transitions | DUAL route state behavior is visible |
| `undebug all` | Stops debugging | All debugging is disabled |
| `ping <REMOTE_SPOKE_IP>` | Confirms spoke-to-spoke reachability | Ping succeeds |
| `traceroute <REMOTE_SPOKE_IP>` | Confirms forwarding path | Path follows the expected hub path unless shortcut behavior exists |
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Rollback
! Classic mode: re-enable EIGRP split horizon on the interface
configure terminal
 interface <HUB_MULTIPOINT_INTERFACE>
  ip split-horizon eigrp <AS_NUMBER>
 end
! Named mode: re-enable EIGRP split horizon on the interface
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <HUB_MULTIPOINT_INTERFACE>
    split-horizon
   exit-af-interface
  exit-address-family
 end
! Stop any active debugging
undebug all
! Verify rollback
show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>
show ip eigrp neighbors
show logging | include EIGRP|DUAL|split horizon|NBRCHANGE
show ip route eigrp
show ip route <REMOTE_SPOKE_PREFIX>
show ip eigrp topology active
ping <REMOTE_SPOKE_IP>
traceroute <REMOTE_SPOKE_IP>
! Expected result:
! EIGRP split horizon is enabled again on the interface.
! Hub-and-spoke spoke-to-spoke routes may disappear if the design depends on same-interface re-advertisement.
! Neighbor adjacencies may resync after the setting change.
write memory
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Spokes cannot learn each other’s routes | Split horizon is enabled on the hub multipoint interface | `show ip eigrp interfaces detail <HUB_MULTIPOINT_INTERFACE>` | Disable EIGRP split horizon on the hub multipoint interface |
| Hub has all routes but spokes have only hub routes | Hub is not re-advertising spoke-learned routes back out the same interface | Hub `show ip route eigrp`; spoke `show ip route eigrp` | Configure `no ip split-horizon eigrp <AS>` on the hub interface |
| `show ip interface` still says split horizon is enabled | Command shows generic IP split horizon, not EIGRP-specific state | `show ip eigrp interfaces detail <INTERFACE>` | Use EIGRP interface detail output as the source of truth |
| EIGRP neighbors reset after split-horizon change | Expected behavior when EIGRP split horizon changes | `show logging | include NBRCHANGE|split horizon` | Wait for reconvergence and verify neighbors return |
| Neighbors do not recover after the change | Interface, tunnel, authentication, AS, or network statement issue | `show ip eigrp neighbors`, `show ip protocols`, `show ip interface brief` | Fix the adjacency problem first |
| Split horizon was disabled but spoke routes are still missing | Wrong interface was changed | `show ip eigrp neighbors` and `show running-config interface <INTERFACE>` | Apply the command on the hub interface that has multiple spoke neighbors |
| Split horizon was disabled on spokes but problem remains | Wrong device was changed | `show ip eigrp interfaces detail <SPOKE_INTERFACE>` | Re-enable split horizon on spokes unless the design explicitly requires otherwise |
| EIGRP memory or update traffic increases | Split horizon disabled too broadly | `show ip eigrp interfaces detail`, `show process cpu`, `show ip eigrp traffic` | Disable split horizon only where same-interface re-advertisement is required |
| Route appears in topology but not in RIB | Another protocol or route has better administrative distance | `show ip route <PREFIX>` | Confirm RIB winner before blaming split horizon |
| Route goes active during testing | Failure removed successor and no feasible successor exists | `show ip eigrp topology active` | Let DUAL finish or fix missing query replies |
| SIA or repeated DUAL events appear | Query/reply problem or unstable neighbor after topology change | `show logging | include SIA|DUAL|EIGRP` | Fix link stability and reduce query scope with summarization or stub where appropriate |
| Poison reverse is not visible in normal show commands | Poison reverse is packet behavior, not a persistent config line | `debug eigrp packets` in lab only | Use lab debug or packet capture if observation is required |
| Startup mode behavior is unclear | Initial update exchange happens quickly and is usually not visible after convergence | `debug eigrp packets` and `debug eigrp fsm` in lab only | Clear or flap adjacency in a lab, then observe startup exchange |
| Debug output overwhelms the router | Debugging left enabled | `show debugging` | Run `undebug all` immediately |
| Spoke-to-spoke ping still fails after routes appear | Data-plane issue, return path issue, ACL, tunnel/NBMA problem, or CEF issue | `show ip route <SOURCE_PREFIX>` and `traceroute <REMOTE_SPOKE_IP>` | Fix forwarding path separately from EIGRP route advertisement |
| Named-mode command does not work | Command placed outside `af-interface` | `show running-config | section router eigrp` | Configure `no split-horizon` under the correct address-family `af-interface` |
| Classic command does not work | Command placed under router mode instead of interface mode | `show running-config interface <INTERFACE>` | Configure `no ip split-horizon eigrp <AS>` under the hub interface |
# Index
# Source_Basis
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Mental_Model
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Configuration_Checklist
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Skeleton
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Verification_Commands
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Rollback
# EIGRP_Split_Horizon_Startup_Mode_And_Poison_Reverse_Failure_Checks
