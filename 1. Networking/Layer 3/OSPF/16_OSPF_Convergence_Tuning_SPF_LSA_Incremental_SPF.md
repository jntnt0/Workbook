OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF.md

OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and IOS XE OSPF material as project source files |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF | Supports OSPF LSDB, SPF, LSA behavior, baseline verification, and OSPF convergence mental model |
| `All_combined_part3.md` | OSPF incremental SPF references | Supports the distinction between full SPF recalculation and incremental SPF recalculation |
| `Ios-xs_combined_epubs.md` | Configure Other OSPF Parameters | Supports `timers throttle spf <spf-delay> <spf-holdtime> <spf-wait>` syntax and route calculation timer tuning |
| `Ios-xs_combined_epubs.md` | OSPF Incremental SPF Support | Supports `ispf` syntax and the model that incremental SPF recomputes only the affected part of the SPF tree |
| `Ios-xs_combined_epubs.md` | OSPFv3 timer sections | Supports equivalent timer concepts for SPF throttling, LSA throttling, and LSA arrival timing |
| `FTD_combined_epubs.md` | OSPF route calculation timers | Supports `timers lsa arrival`, `timers pacing flood`, `timers pacing lsa-group`, `timers pacing retransmission`, `timers throttle lsa`, and `timers throttle spf` timer definitions |
| Related labs | `ospf-incremental-spf-final`, `ospf-spf-throttling-final`, `ospf-lsa-throttling-final` | Main labs for incremental SPF, SPF throttling, and LSA throttling behavior |
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPF convergence | OSPF reacts to topology change by flooding LSAs, updating the LSDB, running SPF, and installing routes into the RIB |
| LSA generation | Local topology changes cause the router to originate updated LSAs |
| LSA flooding | LSAs are flooded to neighbors so every router in the area can update its LSDB |
| LSA arrival | Controls how quickly the router accepts another instance of the same LSA from a neighbor |
| SPF calculation | Router computes the shortest-path tree from the LSDB and selects best paths |
| Full SPF | Recalculates the complete shortest-path tree after a topology change |
| Incremental SPF | Recalculates only the affected portion of the SPF tree when possible, reducing CPU impact in larger topologies |
| SPF throttling | Controls when SPF runs after topology changes. It prevents repeated SPF recalculation during instability |
| LSA throttling | Controls how frequently a router generates new LSAs during instability |
| LSA pacing | Controls how LSAs are grouped, flooded, refreshed, or retransmitted |
| Fast convergence tradeoff | Lower timer values react faster but can increase CPU and LSA churn |
| Stability tradeoff | Higher timer values protect the router during flaps but delay convergence |
| First number | Initial delay before first action, such as SPF or LSA generation |
| Second number | Hold time between the first and second action during continued instability |
| Third number | Maximum wait time during persistent instability |
| Good lab method | Baseline first, change one timer group at a time, verify SPF/LSA behavior, then test reachability |
| Bad production method | Randomly lowering SPF and LSA timers everywhere without measuring CPU, LSDB churn, and adjacency stability |
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the convergence tuning goal | Design step | `Goal: faster convergence / reduced churn / lab comparison` | Timer changes have a clear reason |
| 2 | Identify routers where tuning will be applied | Design step | `<router-list>` | Scope is known before modifying OSPF |
| 3 | Confirm OSPF baseline neighbor state | All target routers | `show ip ospf neighbor` | Required neighbors are `FULL` |
| 4 | Confirm OSPF interface participation | All target routers | `show ip ospf interface brief` | Expected interfaces participate in OSPF |
| 5 | Record baseline OSPF process timers | All target routers | `show ip ospf` | Current SPF, LSA, and pacing timers are known |
| 6 | Record baseline LSDB state | All target routers | `show ip ospf database` | LSDB is stable before tuning |
| 7 | Record baseline OSPF routes | All target routers | `show ip route ospf` | OSPF route table is known before changes |
| 8 | Record baseline CPU if available | All target routers | `show processes cpu sorted` | Router CPU baseline is known before tuning |
| 9 | Enter global configuration mode | All target routers | `configure terminal` | Router enters global configuration mode |
| 10 | Enter OSPF process | All target routers | `router ospf <process-id>` | OSPF router configuration mode opens |
| 11 | Enable incremental SPF | All target routers where supported | `ispf` | Router uses incremental SPF when applicable |
| 12 | Configure SPF throttling for balanced lab behavior | All target routers | `timers throttle spf <spf-start-ms> <spf-hold-ms> <spf-max-wait-ms>` | OSPF dynamically spaces SPF calculations during instability |
| 13 | Configure example SPF throttling values if the lab requires concrete values | All target routers | `timers throttle spf 200 1000 10000` | SPF first run waits 200 ms, hold begins at 1000 ms, max wait is 10000 ms |
| 14 | Configure LSA generation throttling | All target routers | `timers throttle lsa <lsa-start-ms> <lsa-hold-ms> <lsa-max-wait-ms>` | Router dynamically spaces self-originated LSA generation |
| 15 | Configure example LSA throttle values if the lab requires concrete values | All target routers | `timers throttle lsa 0 5000 5000` | First LSA generation is immediate, repeat LSA generation is rate-limited |
| 16 | Configure minimum LSA arrival timer only if needed | All target routers | `timers lsa arrival <milliseconds>` | Router ignores repeated instances of the same LSA arriving too quickly |
| 17 | Configure example LSA arrival value if the lab requires concrete values | All target routers | `timers lsa arrival 1000` | Same LSA instance is accepted no more often than once per second |
| 18 | Tune LSA flood pacing only if flooding pressure is the issue | All target routers | `timers pacing flood <milliseconds>` | LSAs in the flooding queue are paced between updates |
| 19 | Tune LSA retransmission pacing only if retransmission pressure is the issue | All target routers | `timers pacing retransmission <milliseconds>` | LSA retransmission pacing is controlled |
| 20 | Tune LSA group pacing only if refresh/checksum/aging grouping is the issue | All target routers | `timers pacing lsa-group <seconds>` | LSAs are grouped for refresh, checksum, or aging at the configured interval |
| 21 | Exit configuration mode | All target routers | `end` | Router returns to privileged EXEC mode |
| 22 | Save configuration | All target routers | `write memory` | Configuration is saved |
| 23 | Verify OSPF process timer configuration | All target routers | `show running-config | section router ospf` | `ispf`, SPF throttle, LSA throttle, and pacing commands are present |
| 24 | Verify OSPF process reports timer behavior | All target routers | `show ip ospf` | SPF delay, LSA interval, and LSA arrival values reflect the intended settings |
| 25 | Verify neighbors stayed stable after tuning | All target routers | `show ip ospf neighbor` | Neighbors remain `FULL` |
| 26 | Trigger a controlled topology change in the lab | Lab router | `shutdown` then `no shutdown` on a test OSPF interface | OSPF topology changes in a controlled way |
| 27 | Watch SPF recalculation behavior | Lab router | `debug ip ospf spf` | SPF scheduling and recalculation are visible |
| 28 | Watch LSA generation behavior | Lab router | `debug ip ospf lsa-generation` | LSA origination and throttle behavior are visible |
| 29 | Verify LSDB reconverges | All target routers | `show ip ospf database` | LSDB updates and stabilizes after the change |
| 30 | Verify route table reconverges | All target routers | `show ip route ospf` | Expected OSPF routes return or change as designed |
| 31 | Verify reachability after convergence | Test routers | `ping <remote-loopback-ip>` | Remote OSPF destination responds |
| 32 | Verify forwarding path after convergence | Test routers | `traceroute <remote-loopback-ip>` | Path follows the expected post-convergence route |
| 33 | Check CPU after repeated events | All target routers | `show processes cpu sorted` | CPU impact is acceptable |
| 34 | Check logs for OSPF instability | All target routers | `show logging | include OSPF|SPF|LSA|ADJCHG` | No unexpected adjacency churn or repeated SPF storms |
| 35 | Disable debug after testing | Lab routers | `undebug all` | Debugging is stopped |
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Skeleton
! =========================================================
! OSPF convergence tuning baseline
! Use the same policy intentionally across routers unless the lab asks otherwise.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 ispf
 timers throttle spf <SPF_START_MS> <SPF_HOLD_MS> <SPF_MAX_WAIT_MS>
 timers throttle lsa <LSA_START_MS> <LSA_HOLD_MS> <LSA_MAX_WAIT_MS>
 timers lsa arrival <LSA_ARRIVAL_MS>
end
write memory
! =========================================================
! Balanced lab example
! First SPF after 200 ms.
! Second SPF after 1000 ms.
! Maximum SPF wait during instability is 10000 ms.
! First LSA is immediate.
! Repeated LSA generation is rate-limited.
! =========================================================
configure terminal
router ospf 1
 ispf
 timers throttle spf 200 1000 10000
 timers throttle lsa 0 5000 5000
 timers lsa arrival 1000
end
write memory
! =========================================================
! Optional LSA pacing knobs
! Change only when the lab specifically tests flooding or retransmission pressure.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 timers pacing flood <FLOOD_PACING_MS>
 timers pacing retransmission <RETRANSMISSION_PACING_MS>
 timers pacing lsa-group <LSA_GROUP_SECONDS>
end
write memory
! =========================================================
! Lab test trigger
! Use only on a test link.
! =========================================================
configure terminal
interface <TEST_OSPF_INTERFACE>
 shutdown
end
! Wait briefly, then restore.
configure terminal
interface <TEST_OSPF_INTERFACE>
 no shutdown
end
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Verification_Commands
show running-config | section router ospf
show ip ospf
show ip ospf | include SPF|LSA|Incremental|Initial|Minimum|Pacing|Throttle
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf database router
show ip route ospf
show ip route <remote-prefix>
show processes cpu sorted
show logging | include OSPF|SPF|LSA|ADJCHG
debug ip ospf spf
debug ip ospf lsa-generation
show debugging
undebug all
ping <remote-loopback-ip>
traceroute <remote-loopback-ip>
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Rollback
! =========================================================
! Disable incremental SPF
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no ispf
end
write memory
! =========================================================
! Remove SPF throttle tuning
! Router returns to platform default SPF throttle values.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no timers throttle spf
end
write memory
! =========================================================
! Remove LSA throttle tuning
! Router returns to platform default LSA throttle values.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no timers throttle lsa
end
write memory
! =========================================================
! Remove LSA arrival tuning
! Router returns to platform default LSA arrival behavior.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no timers lsa arrival
end
write memory
! =========================================================
! Remove optional LSA pacing flood tuning
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no timers pacing flood
end
write memory
! =========================================================
! Remove optional LSA pacing retransmission tuning
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no timers pacing retransmission
end
write memory
! =========================================================
! Remove optional LSA group pacing tuning
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no timers pacing lsa-group
end
write memory
! =========================================================
! Stop all debugging
! =========================================================
undebug all
! =========================================================
! Lab-only OSPF reset
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Command `ispf` is rejected | Platform or image does not support incremental SPF | `router ospf <process-id>` then `?` | Use supported IOS XE image or omit incremental SPF from that lab |
| SPF still runs frequently | SPF throttle values are too aggressive | `show ip ospf`, `debug ip ospf spf` | Increase SPF hold and max-wait values |
| Convergence is too slow | SPF start or max-wait value is too high | `show ip ospf`, `show logging | include SPF` | Lower SPF start and hold values carefully |
| LSAs are generated too often during flaps | LSA throttle values are too low | `debug ip ospf lsa-generation`, `show ip ospf` | Increase LSA hold and max interval |
| LSDB updates are delayed too much | LSA throttle or LSA arrival values are too conservative | `show ip ospf database`, `show ip route ospf` | Reduce LSA throttle or arrival values carefully |
| Neighbor drops during convergence test | Link flap or unrelated OSPF mismatch, not SPF tuning itself | `show ip ospf neighbor detail`, `show logging | include ADJCHG` | Check interface state, timers, MTU, authentication, and passive status |
| CPU spikes during repeated topology changes | Timer values are too aggressive or topology is too large | `show processes cpu sorted`, `debug ip ospf spf` | Increase SPF and LSA throttle values, reduce topology churn, or improve area design |
| Route table does not update after change | LSDB did not receive the new LSA or SPF did not complete | `show ip ospf database`, `show ip route <prefix>` | Verify LSA flooding, SPF logs, and neighbor state |
| LSDB updates but RIB does not change | Route type, AD, filtering, or better route source wins | `show ip route <prefix>`, `show ip ospf database` | Check route class, administrative distance, distribute lists, and competing routes |
| LSA arrival drops repeated LSAs | Arrival timer is rejecting repeated LSA instances too quickly | `show ip ospf`, `show logging | include LSA` | Tune `timers lsa arrival <milliseconds>` appropriately |
| LSA pacing change has no obvious effect | Wrong knob for the issue being tested | `show ip ospf`, `debug ip ospf lsa-generation` | Use SPF throttle for SPF scheduling and LSA throttle for self-originated LSA generation |
| SPF throttle change has no effect on LSA flood volume | SPF throttling does not control LSA origination | `debug ip ospf spf`, `debug ip ospf lsa-generation` | Tune `timers throttle lsa` if LSA generation is the issue |
| LSA throttle change has no effect on route calculation timing | LSA throttling does not directly control SPF scheduling | `debug ip ospf spf` | Tune `timers throttle spf` for SPF scheduling |
| Incremental SPF does not show obvious behavior in small lab | Topology change requires full SPF or topology is too small to notice benefit | `debug ip ospf spf`, `show processes cpu sorted` | Test with a larger topology or accept that benefit is scale-dependent |
| Timers are inconsistent across routers | Some routers use different convergence policy | `show running-config | section router ospf` on all routers | Standardize timers where consistent behavior is required |
| OSPF routes flap repeatedly | Physical instability, aggressive timers, or excessive LSA churn | `show logging | include OSPF|LINEPROTO|ADJCHG`, `show interfaces <interface-id>` | Fix physical or interface instability before tuning OSPF timers |
| Debug output overloads router | Debug left running during instability | `show debugging` | Run debug only in lab, use conditions when possible, then `undebug all` |
| After rollback, behavior still looks delayed | OSPF has not reconverged or default timers still impose delay | `show ip ospf`, `show ip ospf neighbor` | Wait for convergence or clear OSPF only in a lab window |
##### Source_Basis
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Mental_Model
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Configuration_Checklist
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Skeleton
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Verification_Commands
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Rollback
# OSPF_Convergence_Tuning_SPF_LSA_Incremental_SPF_Failure_Checks
# index of each title throughout note, not in table format

