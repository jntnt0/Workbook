OSPF_Graceful_Shutdown_And_Stub_Router.md

OSPF_Graceful_Shutdown_And_Stub_Router

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and IOS XE OSPF material as project source files |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF | Supports OSPF LSDB, router LSA behavior, SPF recalculation, neighbor verification, and route verification |
| `Ios-xs_combined_epubs.md` | Chapter 10, OSPF Stub Router Advertisement | Provides the OSPF stub router advertisement model and the `max-metric router-lsa` graceful shutdown syntax |
| `Ios-xs_combined_epubs.md` | Configure advertisement on startup | Provides `max-metric router-lsa on-startup <seconds>` for startup drain behavior |
| `Ios-xs_combined_epubs.md` | Configure advertisement until routing tables converge | Provides `max-metric router-lsa on-startup wait-for-bgp` for startup drain until BGP convergence or timer expiry |
| `Ios-xs_combined_epubs.md` | Configure advertisement for a graceful shutdown | Provides `max-metric router-lsa` and verification with `show ip ospf` before shutdown or reload |
| `Ios-xs_combined_epubs.md` | Verify advertisement of maximum metric | Provides `show ip ospf` and `show ip ospf database` verification for maximum-metric router LSA state |
| Related labs | `ospf-graceful-shutdown-final`, `ospf-stub-router-final` | Main labs for OSPF graceful drain and OSPF stub router advertisement behavior |
# OSPF_Graceful_Shutdown_And_Stub_Router_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPF stub router advertisement | A router intentionally advertises itself with maximum metric so other routers avoid using it as a transit node |
| Not OSPF stub area | This is not `area <id> stub`. Stub router advertisement changes this router's advertised metrics, not the area type |
| Graceful shutdown | Before maintenance, make the router unattractive for transit traffic while keeping OSPF adjacencies up long enough for the domain to reconverge |
| Transit drain | Other routers recalculate SPF and prefer alternate paths around the draining router |
| Local adjacency | OSPF neighbors can remain `FULL`; the goal is not to kill OSPF immediately |
| Router LSA | `max-metric router-lsa` modifies the local router's self-originated Type 1 router LSA metrics |
| Maximum metric | Transit links are advertised with very high cost so the router is avoided as an intermediate hop |
| Include stub links | `include-stub` also affects stub links, such as directly connected networks, when the platform supports the option |
| Startup drain | `max-metric router-lsa on-startup <seconds>` keeps the router unattractive for a fixed time after boot |
| BGP wait drain | `max-metric router-lsa on-startup wait-for-bgp` keeps OSPF unattractive until BGP converges or the default timer expires |
| Maintenance drain | Plain `max-metric router-lsa` keeps max-metric active until removed or the router is shut down |
| ABR drain | Optional summary LSA max-metric behavior can make an ABR less preferred for interarea transit |
| ASBR drain | Optional external LSA max-metric behavior can make redistributed external routes less preferred through this ASBR |
| Correct verification | Do not just check neighbor state. Check `show ip ospf`, self-originated router LSA, remote routes, and traceroute |
| Safe maintenance sequence | Advertise max metric, verify traffic moves away, then shut down or reload |
| Bad pattern | Shutting down the router first and hoping OSPF reconverges cleanly afterward |
# OSPF_Graceful_Shutdown_And_Stub_Router_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the router that must be drained | Design step | `Drain router: <router-name>` | Correct router is selected for maintenance or startup protection |
| 2 | Confirm alternate paths exist before drain | Neighboring routers | `show ip route <remote-prefix>` and `traceroute <remote-prefix>` | Network has an alternate path that avoids the drain router |
| 3 | Confirm OSPF adjacencies are healthy before changes | Drain router | `show ip ospf neighbor` | Required neighbors are `FULL` |
| 4 | Confirm OSPF interface participation | Drain router | `show ip ospf interface brief` | Expected OSPF interfaces are active in correct areas |
| 5 | Confirm current OSPF process state | Drain router | `show ip ospf` | No existing max-metric state is active unless expected |
| 6 | Confirm current self-originated router LSA before drain | Drain router | `show ip ospf database router self-originate` | Normal interface metrics are visible before max-metric |
| 7 | Confirm remote routing table before drain | Neighboring routers | `show ip route <prefix-through-drain-router>` | Current next hop and metric are known |
| 8 | Confirm remote forwarding path before drain | Neighboring routers | `traceroute <prefix-through-drain-router>` | Current path is known before drain |
| 9 | Enter configuration mode for maintenance drain | Drain router | `configure terminal` | Router enters global configuration mode |
| 10 | Enter OSPF process | Drain router | `router ospf <process-id>` | OSPF router configuration mode opens |
| 11 | Advertise maximum metric for graceful shutdown | Drain router | `max-metric router-lsa` | Router advertises itself as undesirable for transit until command is removed |
| 12 | Optionally include stub links when local connected prefixes should also be unattractive | Drain router | `max-metric router-lsa include-stub` | Stub links are also advertised with maximum metric where supported |
| 13 | Optionally make ABR summary routes less preferred | ABR drain router | `max-metric router-lsa summary-lsa <metric>` | Type 3 summary LSAs are advertised with high metric where supported |
| 14 | Optionally make ASBR external routes less preferred | ASBR drain router | `max-metric router-lsa external-lsa <metric>` | Type 5 external LSAs are advertised with high metric where supported |
| 15 | Exit configuration mode | Drain router | `end` | Router returns to privileged EXEC mode |
| 16 | Save configuration only if the drain state must survive reload | Drain router | `write memory` | Max-metric drain state is saved if intentionally persistent |
| 17 | Verify max-metric state is active | Drain router | `show ip ospf` | Output shows router LSAs are being originated with maximum metric |
| 18 | Verify self-originated router LSA maximum metrics | Drain router | `show ip ospf database router self-originate` | Router LSA shows maximum link costs |
| 19 | Verify neighbors stayed up during drain | Drain router | `show ip ospf neighbor` | Neighbors remain `FULL` |
| 20 | Verify remote routers recalculated away from drain router | Neighboring routers | `show ip route <remote-prefix>` | Next hop changes away from the drain router where alternate path exists |
| 21 | Verify traffic path moved away | Neighboring routers | `traceroute <remote-prefix>` | Path avoids the drain router |
| 22 | Verify traffic still works before shutdown | Neighboring routers | `ping <remote-prefix-test-ip>` | Traffic succeeds through alternate path |
| 23 | Shut down or reload only after drain verification | Drain router | `reload` or controlled interface shutdown | Router is removed after traffic has moved away |
| 24 | Configure startup drain with a fixed timer if the router should not attract traffic immediately after reload | Drain router | `router ospf <process-id>` then `max-metric router-lsa on-startup <seconds>` | Router advertises max metric for the configured startup interval |
| 25 | Configure startup drain until BGP converges if OSPF depends on BGP readiness | Drain router | `router ospf <process-id>` then `max-metric router-lsa on-startup wait-for-bgp` | Router advertises max metric until BGP convergence or default timeout |
| 26 | Verify startup drain condition | Drain router | `show ip ospf` | Output shows startup max-metric condition, state, and remaining time when active |
| 27 | Remove manual graceful-shutdown drain after maintenance | Drain router | `router ospf <process-id>` then `no max-metric router-lsa` | Router returns to normal metric advertisement |
| 28 | Verify normal metric restoration | Drain router | `show ip ospf` and `show ip ospf database router self-originate` | Maximum-metric state is gone and normal link metrics return |
| 29 | Verify remote routers can use the router again | Neighboring routers | `show ip route <remote-prefix>` | Router may again be selected if it has the best OSPF path |
| 30 | Verify final forwarding path and reachability | Neighboring routers | `traceroute <remote-prefix>` and `ping <remote-prefix-test-ip>` | Traffic follows the expected post-maintenance path |
# OSPF_Graceful_Shutdown_And_Stub_Router_Skeleton
! =========================================================
! Graceful maintenance drain
! Keeps OSPF neighbors up while making this router unattractive
! as a transit router.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 max-metric router-lsa
end
write memory
! =========================================================
! Graceful drain including stub links
! Use only when local connected stub prefixes should also become
! unattractive or the router is being fully removed from service.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 max-metric router-lsa include-stub
end
write memory
! =========================================================
! Startup drain for a fixed time after boot
! Timer range is platform dependent, commonly 5 to 86400 seconds.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 max-metric router-lsa on-startup <SECONDS>
end
write memory
! =========================================================
! Startup drain until BGP converges
! Useful when the router must not attract OSPF transit traffic
! before BGP is ready.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 max-metric router-lsa on-startup wait-for-bgp
end
write memory
! =========================================================
! ABR or ASBR drain options
! Use only when supported by the platform and required by the lab.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 max-metric router-lsa summary-lsa <METRIC>
 max-metric router-lsa external-lsa <METRIC>
end
write memory
! =========================================================
! Remove graceful drain and restore normal OSPF metrics
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa
end
write memory
# OSPF_Graceful_Shutdown_And_Stub_Router_Verification_Commands
show running-config | section router ospf
show ip ospf
show ip ospf | include maximum|Maximum|max-metric|Originating router-LSAs|Condition|State|Time remaining
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf database router
show ip ospf database router self-originate
show ip ospf database summary
show ip ospf database external
show ip route ospf
show ip route <remote-prefix>
show ip route <local-connected-prefix>
ping <remote-prefix-test-ip>
traceroute <remote-prefix-test-ip>
show logging | include OSPF|MAX|metric|ADJCHG|SPF
# OSPF_Graceful_Shutdown_And_Stub_Router_Rollback
! =========================================================
! Remove manual graceful shutdown max-metric state
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa
end
write memory
! =========================================================
! Remove startup fixed-timer max-metric state
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa on-startup <SECONDS>
end
write memory
! =========================================================
! Remove startup wait-for-BGP max-metric state
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa on-startup wait-for-bgp
end
write memory
! =========================================================
! Remove ABR summary max-metric behavior
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa summary-lsa <METRIC>
end
write memory
! =========================================================
! Remove ASBR external max-metric behavior
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa external-lsa <METRIC>
end
write memory
! =========================================================
! Simple full cleanup if unsure which option was configured
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no max-metric router-lsa
end
write memory
! =========================================================
! Lab-only OSPF reset
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Graceful_Shutdown_And_Stub_Router_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Traffic still transits the drain router | No alternate path exists | `show ip route <remote-prefix>` and `traceroute <remote-prefix>` on neighbors | Do not proceed with shutdown until an alternate path exists or outage is accepted |
| Traffic still transits the drain router | Alternate path has worse route class | `show ip route <remote-prefix>` | Remember OSPF route class can beat metric. Fix area or route source design |
| Traffic still transits the drain router | Remote routers have not reconverged yet | `show ip ospf database router <drain-router-id>` and `show ip route <prefix>` | Wait for SPF and route update before shutting down |
| Max-metric state not visible | Command not configured under correct OSPF process | `show running-config | section router ospf` | Configure `max-metric router-lsa` under the active OSPF process |
| Max-metric state not visible | Wrong VRF OSPF process was changed | `show ip ospf`, `show ip route vrf <vrf-name> ospf` | Configure the correct OSPF process and VRF |
| Neighbors drop after max-metric | Separate OSPF issue, not normal stub router advertisement behavior | `show ip ospf neighbor detail`, `show logging | include OSPF|ADJCHG` | Check interface status, timers, authentication, MTU, and area mismatch |
| Local connected prefixes become unreachable | `include-stub` made directly connected stub links unattractive | `show ip ospf database router self-originate`, `show ip route <local-prefix>` from remote router | Remove `include-stub` if local prefixes must remain reachable |
| Router still attracts traffic to local services | Plain `max-metric router-lsa` may not affect all stub links | `show ip ospf database router self-originate` | Use `include-stub` only if draining local connected prefixes is intended |
| ABR still attracts interarea traffic | Summary LSAs still have normal metric | `show ip ospf database summary`, `show ip route <interarea-prefix>` | Use summary LSA max-metric option if supported and required |
| ASBR still attracts external traffic | External LSAs still have normal metric | `show ip ospf database external`, `show ip route <external-prefix>` | Use external LSA max-metric option if supported and required |
| Startup drain expires too soon | `on-startup <seconds>` timer is too short | `show ip ospf | include Time remaining|Condition` | Increase startup timer |
| Startup drain never clears when expected | `on-startup wait-for-bgp` is waiting for BGP or timer expiry | `show ip ospf`, `show ip bgp summary` | Fix BGP convergence or use a fixed startup timer |
| Command option is rejected | Platform does not support that option | `router ospf <process-id>` then `max-metric router-lsa ?` | Use only supported options on that IOS XE image |
| Drain state remains after maintenance | Manual `max-metric router-lsa` was saved and not removed | `show running-config | section router ospf` | Remove with `no max-metric router-lsa` |
| Remote routing table still avoids router after rollback | SPF has not reconverged or normal costs are still high | `show ip ospf database router self-originate`, `show ip route <prefix>` | Verify max-metric is removed and normal interface costs are restored |
| Route changes cause unexpected asymmetric traffic | Only one direction was drained or return path differs | `traceroute` in both directions | Verify forward and reverse OSPF paths |
| Packet loss occurs during shutdown | Router was shut down before neighbors recalculated around it | `show ip route <prefix>` on neighbors before shutdown | Reapply drain, verify route movement, then shut down |
| No route remains after drain | The drain router was the only path to the destination | `show ip route <prefix>` on remote routers | Restore the router or add alternate path before maintenance |
| LSDB shows max metric but RIB did not change | Better alternate path does not exist or route source preference still chooses same path | `show ip ospf database router <drain-router-id>`, `show ip route <prefix>` | Fix topology, route class, or metric design |
| Ping fails after drain even though route changed | Return path or data-plane filtering issue | `show ip route <source-prefix>` on return-side routers | Verify reverse routing, ACLs, and interface state |
##### Source_Basis
# OSPF_Graceful_Shutdown_And_Stub_Router_Mental_Model
# OSPF_Graceful_Shutdown_And_Stub_Router_Configuration_Checklist
# OSPF_Graceful_Shutdown_And_Stub_Router_Skeleton
# OSPF_Graceful_Shutdown_And_Stub_Router_Verification_Commands
# OSPF_Graceful_Shutdown_And_Stub_Router_Rollback
# OSPF_Graceful_Shutdown_And_Stub_Router_Failure_Checks
# index of each title throughout note, not in table format

