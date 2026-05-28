OSPF_Area_Range_Summarization.md

OSPF_Area_Range_Summarization

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 as the main project OSPF source |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF, area range examples | Supports `area <area-id> range <network> <mask>`, `not-advertise`, Type 3 LSA suppression, and Null0 discard route behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, Summarization of Routes | Supports interarea summarization on ABRs, Type 3 summary LSA reduction, `area range`, optional `cost`, and discard route behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, LSA Type 3 Summary Link | Supports Type 3 LSA verification with `show ip ospf database summary` and ABR advertisement logic |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Troubleshooting OSPF Summarization | Supports failure checks for wrong area, missing component routes, incorrect summary mask, and unexpected Null0 discard routes |
| Related lab | `ospf-area-summarization-final` | Main lab for ABR interarea summarization with `area range` |
# OSPF_Area_Range_Summarization_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPF area summarization | Summarizes intra-area prefixes at an ABR before advertising them into other areas as Type 3 LSAs |
| ABR-only operation | `area range` is configured on the ABR, because the ABR converts local area topology into interarea Type 3 LSAs |
| Source area | The area ID in `area <area-id> range` is the area where the more-specific routes originate |
| Destination areas | The summary is advertised from the ABR into other attached areas, commonly Area 0 or from Area 0 into nonbackbone areas |
| Type 3 LSA reduction | Multiple specific Type 3 LSAs are replaced by one summary Type 3 LSA |
| LSDB scaling | Summarization hides detail from other areas and reduces LSDB size outside the source area |
| SPF containment | Internal changes inside the summarized range are less visible outside the source area |
| Component route requirement | The ABR needs at least one matching component route from the source area before the summary is useful |
| Null0 discard route | IOS installs an automatic discard route to Null0 for the summary to prevent loops toward missing more-specifics |
| Summary metric | By default, the summary metric is derived from component costs. The optional `cost <metric>` keyword makes the summary metric deterministic |
| `not-advertise` | `area range ... not-advertise` suppresses matching Type 3 LSAs instead of advertising a summary |
| Not external summarization | `area range` summarizes internal OSPF routes. External OSPF routes use `summary-address` on the ASBR |
| Addressing dependency | Summarization only works cleanly when prefixes are planned in contiguous blocks |
| Black hole risk | A summary can attract traffic for unused subnets inside the range. The Null0 route drops traffic if no more-specific route exists |
# OSPF_Area_Range_Summarization_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the ABR where summarization must occur | Design step | `ABR: <router-name>` | Correct router is selected for `area range` |
| 2 | Identify the source area containing the component prefixes | Design step | `Source area: <area-id>` | Area ID used in `area <area-id> range` is known |
| 3 | Identify the component prefixes to summarize | Design step | `<prefix-1>, <prefix-2>, <prefix-3>` | Specific routes fit inside one planned summary |
| 4 | Calculate the summary prefix and mask | Design step | `<summary-prefix> <summary-mask>` | Summary covers intended component prefixes only |
| 5 | Confirm the router is an ABR | ABR | `show ip ospf` | Router participates in multiple areas and acts as an ABR |
| 6 | Confirm ABR interface area placement | ABR | `show ip ospf interface brief` | ABR has the source area and at least one destination area |
| 7 | Confirm OSPF adjacencies are healthy before summarizing | ABR and area routers | `show ip ospf neighbor` | Required adjacencies are `FULL` |
| 8 | Confirm component routes exist before summarization | ABR | `show ip route ospf` | More-specific source area routes exist in the ABR routing table |
| 9 | Confirm component LSAs exist in the source area | ABR | `show ip ospf database router` | ABR has source-area topology information for the component prefixes |
| 10 | Confirm current Type 3 LSAs before summarization | Downstream routers | `show ip ospf database summary` | More-specific Type 3 LSAs are visible before summarization |
| 11 | Confirm current interarea routes before summarization | Downstream routers | `show ip route ospf` | More-specific routes appear as `O IA` before summarization |
| 12 | Enter global configuration mode | ABR | `configure terminal` | Router enters global configuration mode |
| 13 | Enter the OSPF process | ABR | `router ospf <process-id>` | OSPF router configuration mode opens |
| 14 | Configure area range summarization | ABR | `area <source-area-id> range <summary-prefix> <summary-mask>` | ABR advertises one Type 3 summary for the range |
| 15 | Optionally configure deterministic summary metric | ABR | `area <source-area-id> range <summary-prefix> <summary-mask> cost <metric>` | Summary Type 3 LSA uses the configured metric |
| 16 | Optionally suppress the summary instead of advertising it | ABR | `area <source-area-id> range <summary-prefix> <summary-mask> not-advertise` | Matching Type 3 LSAs are suppressed |
| 17 | Exit configuration mode | ABR | `end` | Router returns to privileged EXEC mode |
| 18 | Save the configuration | ABR | `write memory` | Configuration is saved |
| 19 | Verify area range is present in running config | ABR | `show running-config | section router ospf` | `area <area-id> range <summary-prefix> <summary-mask>` is present |
| 20 | Verify ABR installed the summary discard route | ABR | `show ip route ospf | include Null0` | Summary route to Null0 appears for loop prevention |
| 21 | Verify specific Type 3 LSAs are reduced downstream | Downstream routers | `show ip ospf database summary` | Component Type 3 LSAs are removed or reduced as expected |
| 22 | Verify summary Type 3 LSA appears downstream | Downstream routers | `show ip ospf database summary <summary-prefix>` | Summary prefix appears as a Type 3 LSA |
| 23 | Verify summary route installs in downstream RIB | Downstream routers | `show ip route <summary-prefix>` | Summary appears as an interarea `O IA` route |
| 24 | Verify component routes remain visible inside the source area | Source area routers | `show ip route <component-prefix>` | More-specific routes remain intra-area inside the source area |
| 25 | Verify the ABR still has more-specific component routes | ABR | `show ip route <component-prefix>` | ABR can forward to real component prefixes |
| 26 | Verify reachability to covered component prefixes | Downstream routers | `ping <component-prefix-test-ip>` | Traffic succeeds for existing more-specific prefixes |
| 27 | Verify unused space inside the summary does not loop | ABR | `show ip route <unused-prefix-inside-summary>` | Traffic matching unused space resolves to the Null0 discard summary |
| 28 | Verify routing table simplification | Downstream routers | `show ip route ospf` | Multiple `O IA` component routes are replaced by one summarized `O IA` route |
# OSPF_Area_Range_Summarization_Skeleton
! =========================================================
! OSPF interarea summarization on an ABR
! The area ID is the source area where the component routes live.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK>
end
write memory
! =========================================================
! OSPF interarea summarization with fixed summary metric
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> cost <SUMMARY_METRIC>
end
write memory
! =========================================================
! Suppress Type 3 LSAs for the range instead of advertising summary
! Use only when the goal is to hide the range completely.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> not-advertise
end
write memory
! =========================================================
! Example
! Area 1 contains 10.1.0.0/24 through 10.1.3.0/24.
! ABR advertises 10.1.0.0/22 into other areas.
! =========================================================
configure terminal
router ospf 1
 area 1 range 10.1.0.0 255.255.252.0
end
write memory
! =========================================================
! Example with deterministic metric
! =========================================================
configure terminal
router ospf 1
 area 1 range 10.1.0.0 255.255.252.0 cost 50
end
write memory
# OSPF_Area_Range_Summarization_Verification_Commands
show ip ospf
show ip ospf interface brief
show ip ospf neighbor
show running-config | section router ospf
show ip ospf database
show ip ospf database router
show ip ospf database summary
show ip ospf database summary <SUMMARY_PREFIX>
show ip route ospf
show ip route <SUMMARY_PREFIX>
show ip route <COMPONENT_PREFIX>
show ip route ospf | include Null0
show ip route <UNUSED_PREFIX_INSIDE_SUMMARY>
ping <COMPONENT_PREFIX_TEST_IP>
ping <SUMMARY_RANGE_TEST_IP>
traceroute <COMPONENT_PREFIX_TEST_IP>
# OSPF_Area_Range_Summarization_Rollback
! =========================================================
! Remove standard area range summarization
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK>
end
write memory
! =========================================================
! Remove area range summarization with configured cost
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> cost <SUMMARY_METRIC>
end
write memory
! =========================================================
! Remove not-advertise area range suppression
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> not-advertise
end
write memory
! =========================================================
! Replace incorrect summary with corrected summary
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <SOURCE_AREA_ID> range <WRONG_SUMMARY_PREFIX> <WRONG_SUMMARY_MASK>
 area <SOURCE_AREA_ID> range <CORRECT_SUMMARY_PREFIX> <CORRECT_SUMMARY_MASK>
end
write memory
! =========================================================
! Lab-only OSPF refresh if output remains stale
! This resets OSPF adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Area_Range_Summarization_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Summary does not appear downstream | Command was applied on a non-ABR | `show ip ospf`, `show ip ospf interface brief` | Configure `area range` on the ABR between the source area and destination area |
| Summary does not appear downstream | Wrong source area used in `area range` | `show ip ospf interface brief`, `show running-config | section router ospf` | Use the area where the component routes originate |
| Summary does not appear downstream | No component route exists inside the range | `show ip route <component-prefix>`, `show ip ospf database router` | Restore or advertise at least one matching component prefix |
| Summary covers too much address space | Summary mask is too broad | `show ip route <unused-prefix-inside-summary>` | Recalculate the summary or split it into smaller summaries |
| Component Type 3 LSAs still appear downstream | Summary does not cover the component prefix or wrong ABR was configured | `show ip ospf database summary` | Correct the summary range on the ABR advertising those prefixes |
| Summary appears but metric is not expected | Default metric derived from component routes | `show ip ospf database summary <summary-prefix>` | Configure `cost <metric>` under the `area range` command |
| Summary route points to Null0 on the ABR | Normal discard route behavior for loop prevention | `show ip route ospf | include Null0` | No fix needed unless there is a specific lab requirement |
| Traffic to unused subnet inside summary drops | Summary attracts traffic for unused space and Null0 discards it | `show ip route <unused-prefix-inside-summary>` | This is expected. Use a narrower summary if blackholing is unacceptable |
| Source area routers still see specifics | Area range affects interarea advertisement only | `show ip route ospf` inside source area | No fix needed. Intra-area LSAs are not summarized inside their own area |
| External OSPF routes are not summarized | Used `area range` for Type 5 external routes | `show ip ospf database external` | Use `summary-address` on the ASBR for external summarization |
| Route disappears completely | `not-advertise` was configured by mistake | `show running-config | section router ospf` | Remove `not-advertise` or replace with normal `area range` |
| Downstream router still prefers a more-specific route | Longer prefix match exists from another source | `show ip route <component-prefix>` | Check alternate ABR, redistribution, static route, or another routing protocol |
| Interarea route appears as `O IA` summary but ping fails | Return path missing or more-specific component missing behind ABR | `show ip route <destination>`, `traceroute <destination>` | Verify reverse routing and component prefix reachability from the ABR |
| Summary does not reduce LSDB much | Prefix plan is not contiguous | `show ip ospf database summary` | Redesign addressing or create multiple smaller summaries |
| Removing summary does not immediately restore expected output | OSPF refresh delay or stale lab state | `show ip ospf database summary`, `show ip route ospf` | Wait for update or use `clear ip ospf process` only in a lab window |
##### Source_Basis
# OSPF_Area_Range_Summarization_Mental_Model
# OSPF_Area_Range_Summarization_Configuration_Checklist
# OSPF_Area_Range_Summarization_Skeleton
# OSPF_Area_Range_Summarization_Verification_Commands
# OSPF_Area_Range_Summarization_Rollback
# OSPF_Area_Range_Summarization_Failure_Checks
# index of each title throughout note, not in table format

