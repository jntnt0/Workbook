OSPF_Virtual_Link_Discontiguous_Backbone_Repair.md

OSPF_Virtual_Link_Discontiguous_Backbone_Repair

# Source_Basis

| Source | Relevant Section | What It Supports |

|---|---|---|

| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 as the main project OSPF source |

| `All_combined_part3.md` | Book 6, Chapter 8, Lab 8-9: Virtual Links and GRE Tunnels | Supports virtual-link repair for areas without direct Area 0 attachment, `area <transit-area> virtual-link <remote-router-id>`, and `show ip ospf virtual-links` verification |

| `All_combined_part3.md` | Book 6, Chapter 8, Virtual link task notes | Supports the requirements that virtual links terminate on ABRs, use router IDs, require a non-stub transit area, and restore backbone connectivity logically |

| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | OSPF Areas / Virtual Links | Supports Area 0 backbone rules, disconnected backbone repair logic, transit-area behavior, and interarea route restoration |

| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | OSPF Troubleshooting | Supports failure checks for missing virtual link, wrong transit area, wrong router ID, stub transit area, and broken interarea routing |

| Related labs | `ospf-virtual-link-discontigous-area-final`, `ospf-virtual-link-three-areas-final` | Main labs for repairing disconnected Area 0 or disconnected nonbackbone reachability using OSPF virtual links |

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Mental_Model

| Concept | Operational Meaning |

|---|---|

| Area 0 backbone rule | All nonbackbone areas must connect to Area 0 physically or logically |

| Discontiguous backbone | Area 0 is split or a nonbackbone area is separated from Area 0 |

| Virtual link | Logical OSPF link that extends Area 0 through a transit area |

| Transit area | The nonbackbone area used to carry the virtual link between ABRs |

| Transit area requirement | Transit area must be a normal area, not stub, totally stub, NSSA, or totally NSSA |

| Endpoint requirement | Virtual link must terminate on ABRs |

| Router ID targeting | Virtual link command uses the remote endpoint router ID, not the remote interface IP |

| Bidirectional configuration | Both ABRs must configure a matching virtual link using the same transit area and each other’s router ID |

| Backbone extension | After the virtual link comes up, the remote endpoint logically participates in Area 0 |

| Logical point-to-point link | The virtual link behaves like a point-to-point unnumbered OSPF interface |

| Unicast OSPF packets | OSPF packets across the virtual link are tunneled as unicast between endpoints, not sent as multicast hellos on the transit area |

| Cost inheritance | Virtual-link cost is inherited from the shortest OSPF path through the transit area |

| Route restoration goal | Successful repair restores Type 3 interarea LSA exchange and `O IA` routes across the OSPF domain |

| Design warning | Virtual links are repair tools, not clean long-term design. The cleaner design is contiguous Area 0 |

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Identify the disconnected backbone or isolated area | Design step | `Disconnected area: <area-id>` | Area needing logical Area 0 repair is known |

| 2 | Identify the transit area | Design step | `Transit area: <area-id>` | Nonbackbone area between the two virtual-link endpoints is known |

| 3 | Confirm transit area is a normal area | ABRs | `show ip ospf` | Transit area is not stub, totally stub, NSSA, or totally NSSA |

| 4 | Identify the two virtual-link endpoint routers | Design step | `Endpoint A: <router-name>, Endpoint B: <router-name>` | Correct ABRs are selected |

| 5 | Confirm endpoint A is an ABR | Endpoint A | `show ip ospf` | Router participates in the transit area and Area 0 or becomes logically connected to Area 0 |

| 6 | Confirm endpoint B is an ABR | Endpoint B | `show ip ospf` | Router participates in the transit area and disconnected area or Area 0 |

| 7 | Confirm OSPF adjacencies inside the transit area | Both endpoints | `show ip ospf neighbor` | Transit-area neighbor path is healthy |

| 8 | Confirm endpoint router IDs | Both endpoints | `show ip ospf | include ID` | Stable router IDs are known |

| 9 | Confirm router IDs are manually configured | Both endpoints | `show running-config | section router ospf` | `router-id <router-id>` is present or router ID selection is understood |

| 10 | Confirm endpoint reachability through transit area | Both endpoints | `show ip route <remote-router-id>` | Each endpoint can reach the other endpoint router ID through the transit area |

| 11 | Confirm interarea routes are missing before repair | Affected routers | `show ip route ospf` | Expected `O IA` routes are absent or incomplete |

| 12 | Enter global configuration mode | Endpoint A | `configure terminal` | Router enters global configuration mode |

| 13 | Enter OSPF process | Endpoint A | `router ospf <process-id>` | OSPF router configuration mode opens |

| 14 | Configure virtual link toward endpoint B | Endpoint A | `area <transit-area-id> virtual-link <endpoint-b-router-id>` | Endpoint A attempts to build virtual link to endpoint B |

| 15 | Exit configuration mode | Endpoint A | `end` | Router returns to privileged EXEC mode |

| 16 | Enter global configuration mode | Endpoint B | `configure terminal` | Router enters global configuration mode |

| 17 | Enter OSPF process | Endpoint B | `router ospf <process-id>` | OSPF router configuration mode opens |

| 18 | Configure virtual link toward endpoint A | Endpoint B | `area <transit-area-id> virtual-link <endpoint-a-router-id>` | Endpoint B attempts to build virtual link to endpoint A |

| 19 | Exit configuration mode | Endpoint B | `end` | Router returns to privileged EXEC mode |

| 20 | Save configuration | Both endpoints | `write memory` | Configuration is saved |

| 21 | Verify virtual-link state | Both endpoints | `show ip ospf virtual-links` | Virtual link is `up` and adjacency state is `FULL` |

| 22 | Verify virtual-link transit area | Both endpoints | `show ip ospf virtual-links` | Output shows the expected transit area |

| 23 | Verify virtual-link cost | Both endpoints | `show ip ospf virtual-links` | Cost is present and below unusable values |

| 24 | Verify virtual-link neighbor appears | Both endpoints | `show ip ospf neighbor` | Neighbor appears over `OSPF_VL` or virtual-link interface |

| 25 | Verify Area 0 logical restoration | Both endpoints | `show ip ospf` | Endpoint participates in backbone behavior as expected |

| 26 | Verify summary LSAs are restored | Affected routers | `show ip ospf database summary` | Type 3 summary LSAs from remote areas appear |

| 27 | Verify interarea routes return | Affected routers | `show ip route ospf` | Expected remote-area routes appear as `O IA` |

| 28 | Verify end-to-end reachability | Affected routers | `ping <remote-area-loopback-ip>` | Remote area destination responds |

| 29 | Verify forwarding path | Affected routers | `traceroute <remote-area-loopback-ip>` | Path follows expected ABR and transit-area repair path |

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Skeleton

  

! =========================================================

! Endpoint A

! Uses transit area and remote endpoint B router ID.

! =========================================================

  

configure terminal

  

router ospf <PROCESS_ID>

 router-id <ENDPOINT_A_ROUTER_ID>

 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID>

end

  

write memory

  

  

! =========================================================

! Endpoint B

! Uses the same transit area and remote endpoint A router ID.

! =========================================================

  

configure terminal

  

router ospf <PROCESS_ID>

 router-id <ENDPOINT_B_ROUTER_ID>

 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID>

end

  

write memory

  

  

! =========================================================

! Example

! R1 router ID 0.0.0.1

! R2 router ID 0.0.0.2

! Transit area 1

! =========================================================

  

! On R1:

configure terminal

router ospf 1

 area 1 virtual-link 0.0.0.2

end

write memory

  

! On R2:

configure terminal

router ospf 1

 area 1 virtual-link 0.0.0.1

end

write memory

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Verification_Commands

  

show ip ospf

show ip ospf | include ID|Area|backbone

show running-config | section router ospf

show ip ospf interface brief

show ip ospf neighbor

show ip ospf neighbor detail

show ip ospf virtual-links

show ip ospf database

show ip ospf database router

show ip ospf database summary

show ip route ospf

show ip route <remote-router-id>

show ip route <remote-area-prefix>

ping <remote-router-id>

ping <remote-area-loopback-ip>

traceroute <remote-area-loopback-ip>

show logging | include OSPF|virtual-link|mismatched area|must be virtual-link

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Rollback

  

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

! Remove mistaken virtual link using wrong router ID

! =========================================================

  

configure terminal

router ospf <PROCESS_ID>

 no area <TRANSIT_AREA_ID> virtual-link <WRONG_REMOTE_ROUTER_ID>

end

write memory

  

  

! =========================================================

! Clear OSPF only in a lab or maintenance window

! This disrupts adjacencies.

! =========================================================

  

clear ip ospf process

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Failure_Checks

| Symptom | Likely Cause | Check | Corrective Action |

|---|---|---|---|

| Virtual link stays down | Configured on only one endpoint | `show running-config | section router ospf` | Configure matching `area <transit-area> virtual-link <remote-router-id>` on both endpoints |

| Virtual link stays down | Wrong remote router ID used | `show ip ospf | include ID`, `show running-config | section router ospf` | Replace interface IP or wrong RID with the remote endpoint router ID |

| Virtual link stays down | Transit area mismatch | `show running-config | section router ospf` on both endpoints | Use the same transit area ID on both endpoints |

| Virtual link stays down | Endpoint is not an ABR | `show ip ospf` | Ensure each endpoint participates in the transit area and either Area 0 or the disconnected area role required |

| Virtual link stays down | Transit area is stub or NSSA style | `show ip ospf` | Convert transit area to normal area or choose another normal transit area |

| Virtual link stays down | Remote endpoint router ID is unreachable through transit area | `show ip route <remote-router-id>` | Restore OSPF reachability through the transit area |

| Console shows `must be virtual-link but not found` | Backbone packet received across area path with missing virtual link | `show logging | include must be virtual-link` | Configure the required virtual link on both ABRs |

| Console shows mismatched area ID | Area 0 relationship is expected but virtual link is missing or wrong | `show logging | include mismatched area` | Verify area placement and virtual-link configuration |

| Virtual-link adjacency stuck below FULL | Authentication, timer, MTU, or reachability mismatch | `show ip ospf virtual-links`, `show ip ospf neighbor detail` | Match virtual-link parameters and fix transit-area reachability |

| Interarea routes still missing after virtual link is FULL | Area 0 still discontiguous elsewhere or another ABR issue exists | `show ip ospf database summary`, `show ip route ospf` | Check all ABRs and Area 0 continuity |

| Type 3 LSAs still missing | ABR is not generating or receiving summaries | `show ip ospf database summary` | Verify ABR role, Area 0 attachment, and filtering policy |

| Virtual link comes up but traffic fails | Route exists one way only or return path is missing | `show ip route <remote-prefix>`, `traceroute <remote-area-loopback-ip>` | Verify routes in both directions |

| Virtual-link cost is unexpectedly high | Transit-area path cost is high | `show ip ospf virtual-links`, `show ip route <remote-router-id>` | Tune transit-area interface costs or redesign Area 0 |

| Virtual link breaks after router reload | Router ID changed | `show ip ospf | include ID` | Configure stable `router-id` on both endpoints |

| Virtual link breaks after area-type change | Transit area was converted to stub, totally stub, NSSA, or totally NSSA | `show ip ospf` | Restore transit area to normal area |

| Wrong area was repaired | Virtual link configured across the wrong transit area | `show ip ospf virtual-links`, `show ip ospf database summary` | Remove wrong virtual link and configure through the correct transit area |

| `O IA` routes appear but path is ugly | Virtual link repairs control plane, not necessarily optimal forwarding | `traceroute <remote-area-loopback-ip>` | Accept as temporary repair or redesign with contiguous Area 0 |

##### Source_Basis

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Mental_Model

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Configuration_Checklist

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Skeleton

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Verification_Commands

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Rollback

# OSPF_Virtual_Link_Discontiguous_Backbone_Repair_Failure_Checks

# index of each title throughout note, not in table format