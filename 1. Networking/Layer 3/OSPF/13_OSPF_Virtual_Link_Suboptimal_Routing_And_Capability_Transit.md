OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit.md

OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and IOS XE OSPF material as project source files |
| `All_combined_part3.md` | Book 6, Chapter 8, Virtual Links and OSPF area behavior | Supports virtual-link control-plane repair, ABR behavior, Area 0 continuity, and interarea route restoration |
| `All_combined_part3.md` | Book 6, Chapter 8, Stub area verification output | Shows `show ip ospf` output including `Supports area transit capability` and `Number of areas transit capable` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Virtual Links | Supports virtual links as temporary repair for nonadjacent Area 0 and verification with `show ip ospf virtual-links` |
| `Ios-xs_combined_epubs.md` | OSPF Configuration Guide, OSPF Area Transit Capability | Supports the feature model that nonbackbone areas can act as transit areas, that the feature is enabled by default, and that `no capability transit` disables it under `router ospf` |
| Related labs | `ospf-virtual-link-suboptimal-routing-final`, `ospf-capability-transit-final` | Main labs for seeing virtual-link repair create functional but suboptimal routing, then validating OSPF area transit capability behavior |
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Virtual-link repair | A virtual link repairs OSPF backbone continuity by logically extending Area 0 through a transit area |
| Control-plane success | A virtual link can restore LSDB exchange and `O IA` routes even if the data path is not optimal |
| Data-plane problem | Traffic may follow the logical Area 0 repair path instead of the physically shortest path |
| Suboptimal routing | Routing is technically working, but packets take a longer or less desirable path |
| Transit area | The nonbackbone area used to carry the virtual link between ABRs |
| Normal Area 0 rule | By default OSPF interarea transit is expected to move through Area 0 |
| Area transit capability | Enhancement that allows a nonbackbone area to act as a transit area when it provides a better path |
| Enabled by default | IOS XE OSPF area transit capability is enabled by default on supported platforms |
| Disable command | `no capability transit` disables OSPF area transit capability under the OSPF process |
| Re-enable model | Re-enable by removing the disabling command or using `capability transit` if supported by the platform |
| ABR decision point | The ABR must be able to recognize and use the shorter nonbackbone transit path |
| Verification target | Do not stop at `show ip ospf virtual-links`. Also verify `show ip route`, `show ip ospf route`, and `traceroute` |
| Virtual link caveat | Virtual links are temporary repair tools, not clean long-term design |
| Clean design | A contiguous Area 0 or redesigned area layout is better than depending on virtual links and transit capability behavior |
| Failure boundary | If the virtual link is down, fix backbone continuity first. If the virtual link is up but the path is ugly, troubleshoot path selection and transit capability |
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the disconnected or repaired backbone design | Design step | `Area 0 repair path: <area-id>` | The virtual-link repair topology is understood |
| 2 | Identify the virtual-link endpoints | Design step | `Endpoint A: <router-name>, Endpoint B: <router-name>` | Correct ABRs are selected |
| 3 | Identify the transit area | Design step | `Transit area: <area-id>` | Nonbackbone transit area is known |
| 4 | Identify the expected optimal forwarding path | Design step | `Expected path: <router-list>` | Data-plane expectation is documented before testing |
| 5 | Confirm the transit area is normal | Virtual-link endpoints | `show ip ospf` | Transit area is not stub, totally stub, NSSA, or totally NSSA |
| 6 | Confirm endpoint router IDs | Virtual-link endpoints | `show ip ospf | include ID` | Each endpoint router ID is known |
| 7 | Confirm OSPF adjacencies in the transit area | Virtual-link endpoints | `show ip ospf neighbor` | Transit-area adjacencies are `FULL` |
| 8 | Confirm reachability to remote endpoint router ID | Virtual-link endpoints | `show ip route <remote-router-id>` | Remote router ID is reachable through the transit area |
| 9 | Confirm current virtual-link state | Virtual-link endpoints | `show ip ospf virtual-links` | Virtual link state is known before capability testing |
| 10 | Configure virtual link if backbone repair is not already present | Endpoint A | `router ospf <process-id>` then `area <transit-area-id> virtual-link <endpoint-b-router-id>` | Endpoint A attempts virtual-link adjacency to endpoint B |
| 11 | Configure matching virtual link | Endpoint B | `router ospf <process-id>` then `area <transit-area-id> virtual-link <endpoint-a-router-id>` | Endpoint B attempts virtual-link adjacency to endpoint A |
| 12 | Verify virtual-link repair is working | Both endpoints | `show ip ospf virtual-links` | Virtual link is `up` and state is `FULL` |
| 13 | Verify Type 3 LSAs are restored | Affected routers | `show ip ospf database summary` | Interarea summary LSAs are present |
| 14 | Verify interarea routes are installed | Affected routers | `show ip route ospf` | Expected routes appear as `O IA` |
| 15 | Record the current forwarding path | Test router | `traceroute <remote-prefix-test-ip>` | Actual path is captured before transit capability change |
| 16 | Confirm whether the path is suboptimal | Test router | `show ip route <remote-prefix>` and `traceroute <remote-prefix-test-ip>` | Installed next hop and hop sequence are compared against expected path |
| 17 | Check OSPF route calculation view | Test router | `show ip ospf route` | OSPF internal route calculation shows selected path and cost |
| 18 | Confirm area transit capability status | ABRs | `show ip ospf | include transit|Transit|Supports area transit` | Router shows whether area transit capability is supported or active |
| 19 | Enter configuration mode if disabling transit capability for lab comparison | ABR | `configure terminal` | Router enters global configuration mode |
| 20 | Enter OSPF process | ABR | `router ospf <process-id>` | OSPF router configuration mode opens |
| 21 | Disable area transit capability for comparison testing | ABR | `no capability transit` | OSPF area transit capability is disabled for the process |
| 22 | Exit configuration mode | ABR | `end` | Router returns to privileged EXEC mode |
| 23 | Save the configuration | ABR | `write memory` | Configuration is saved |
| 24 | Recheck OSPF process capability state | ABR | `show ip ospf | include transit|Transit|Supports area transit` | Output reflects changed transit capability behavior |
| 25 | Recheck OSPF route selection | Test router | `show ip ospf route` | OSPF path calculation reflects current capability behavior |
| 26 | Recheck the installed route | Test router | `show ip route <remote-prefix>` | Next hop and metric reflect current OSPF decision |
| 27 | Recheck forwarding path | Test router | `traceroute <remote-prefix-test-ip>` | Path shows whether traffic follows virtual-link repair or shorter transit-area path |
| 28 | Re-enable area transit capability if the lab goal is optimal transit behavior | ABR | `router ospf <process-id>` then `capability transit` | OSPF area transit capability is restored on supported platforms |
| 29 | Alternative re-enable method if command form differs | ABR | `router ospf <process-id>` then `no no capability transit` is not valid; remove the disabling line or use platform-supported `capability transit` | Running config no longer disables transit capability |
| 30 | Verify final virtual-link state | Both endpoints | `show ip ospf virtual-links` | Virtual link remains `up` and `FULL` |
| 31 | Verify final interarea route table | Affected routers | `show ip route ospf` | Required `O IA` routes remain installed |
| 32 | Verify final forwarding path | Test router | `traceroute <remote-prefix-test-ip>` | Path matches the intended lab result |
| 33 | Verify final reachability | Test router | `ping <remote-prefix-test-ip>` | Destination responds |
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Skeleton
! =========================================================
! Baseline virtual-link repair
! Configure on both ABRs using the transit area and remote router ID.
! =========================================================
! Endpoint A:
configure terminal
router ospf <PROCESS_ID>
 router-id <ENDPOINT_A_ROUTER_ID>
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID>
end
write memory
! Endpoint B:
configure terminal
router ospf <PROCESS_ID>
 router-id <ENDPOINT_B_ROUTER_ID>
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID>
end
write memory
! =========================================================
! Disable OSPF area transit capability
! Use for lab comparison when validating suboptimal routing behavior.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no capability transit
end
write memory
! =========================================================
! Re-enable OSPF area transit capability
! Area transit capability is normally enabled by default.
! Use the positive command if supported by the platform.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 capability transit
end
write memory
! =========================================================
! Example
! R2 RID 2.2.2.2
! R4 RID 4.4.4.4
! Transit area 1
! =========================================================
! On R2:
configure terminal
router ospf 1
 area 1 virtual-link 4.4.4.4
end
write memory
! On R4:
configure terminal
router ospf 1
 area 1 virtual-link 2.2.2.2
end
write memory
! Lab comparison:
configure terminal
router ospf 1
 no capability transit
end
write memory
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Verification_Commands
show ip ospf
show ip ospf | include ID|Area|transit|Transit|Supports area transit|Number of areas transit
show running-config | section router ospf
show ip ospf interface brief
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf virtual-links
show ip ospf database
show ip ospf database router
show ip ospf database summary
show ip ospf route
show ip route ospf
show ip route <remote-prefix>
show ip route <remote-router-id>
ping <remote-prefix-test-ip>
traceroute <remote-prefix-test-ip>
show logging | include OSPF|virtual-link|transit|mismatch|ADJCHG
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Rollback
! =========================================================
! Remove virtual link from endpoint A
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID>
end
write memory
! =========================================================
! Remove virtual link from endpoint B
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID>
end
write memory
! =========================================================
! Re-enable OSPF area transit capability after lab disablement
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 capability transit
end
write memory
! =========================================================
! Disable OSPF area transit capability again if restoring a comparison state
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no capability transit
end
write memory
! =========================================================
! Remove the OSPF process for full lab reset
! =========================================================
configure terminal
no router ospf <PROCESS_ID>
end
write memory
! =========================================================
! Lab-only OSPF reset
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Virtual link is down | Wrong remote router ID | `show ip ospf | include ID`, `show running-config | section router ospf` | Use the remote endpoint router ID, not the interface IP |
| Virtual link is down | Wrong transit area ID | `show running-config | section router ospf`, `show ip ospf` | Configure the shared transit area, not the disconnected area |
| Virtual link is down | Transit area is stub or NSSA | `show ip ospf` | Convert transit area to normal area or choose a different normal transit area |
| Virtual link is down | Remote endpoint router ID is unreachable | `show ip route <remote-router-id>` | Restore transit-area reachability |
| Interarea routes are missing | Backbone continuity is still broken | `show ip ospf virtual-links`, `show ip ospf database summary` | Fix virtual-link state before troubleshooting path optimization |
| Route exists but path is ugly | Virtual link repaired control plane but data plane follows a suboptimal route | `show ip route <remote-prefix>`, `traceroute <remote-prefix-test-ip>` | Check OSPF costs and area transit capability |
| Path changes after `no capability transit` | Area transit capability was allowing a shorter nonbackbone transit path | `show ip ospf route`, `traceroute <remote-prefix-test-ip>` | This is expected in the lab. Re-enable capability if the shorter path is desired |
| Path does not change after disabling capability transit | Topology has no better nonbackbone transit path or platform behavior differs | `show ip ospf route`, `show ip route <remote-prefix>` | Verify the topology actually contains an alternate transit-area path |
| `no capability transit` not accepted | Platform or image does not support the command | `show version`, `router ospf ?` | Use the platform-supported feature set or document the limitation |
| `capability transit` not accepted | Platform uses default behavior without positive command support | `show running-config | section router ospf` | Remove `no capability transit` if present or document default behavior |
| ABR does not show transit-capable area count | Feature not supported, disabled, or no qualifying area exists | `show ip ospf | include transit|Transit` | Confirm IOS XE support and area design |
| `show ip ospf virtual-links` is full but `O IA` routes are missing | Type 3 filtering, summarization suppression, or area issue exists | `show ip ospf database summary`, `show running-config | section router ospf` | Check ABR Type 3 filtering, area range `not-advertise`, and area type |
| Traffic follows longer path despite better physical link | OSPF cost on shorter path is higher | `show ip ospf interface <interface-id> | include Cost`, `show ip ospf route` | Tune interface cost carefully |
| Traffic follows default instead of detailed route | Stub, totally stub, or totally NSSA area hides detail | `show ip ospf`, `show ip route 0.0.0.0` | Use a normal area or adjust area type if route visibility is required |
| Route installed through wrong ABR | ABR cost or summary/default metric is influencing path | `show ip route <remote-prefix>`, `show ip ospf database summary <prefix>` | Adjust ABR interface cost, summary cost, or default cost |
| Route exists but ping fails | Return path missing or ACL/data-plane issue | `show ip route <source-prefix>` on return-side routers | Verify reverse route and filtering |
| Traceroute does not match OSPF route | CEF recursion or another route source is winning | `show ip route <remote-prefix>` | Confirm the installed route source and next hop on every hop |
| After rollback, output still looks stale | OSPF has not refreshed or lab state is stuck | `show ip ospf route`, `show ip ospf database summary` | Wait for reconvergence or use `clear ip ospf process` only in a lab window |
| Long-term design depends on virtual links | Temporary repair was treated as permanent design | Design review | Redesign Area 0 to be contiguous |
##### Source_Basis
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Mental_Model
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Configuration_Checklist
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Skeleton
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Verification_Commands
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Rollback
# OSPF_Virtual_Link_Suboptimal_Routing_And_Capability_Transit_Failure_Checks
# index of each title throughout note, not in table format
