
OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA.md

OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 as the main project OSPF source |
| `All_combined_part3.md` | Book 6, Chapter 8, Lab 8-10: OSPF Stub, Totally Stubby, and NSSA Areas | Supports `area <area-id> stub`, `area <area-id> stub no-summary`, `area <area-id> nssa`, `area <area-id> nssa default-information-originate`, and `area <area-id> nssa no-summary` |
| `All_combined_part3.md` | Book 6, Chapter 8, NSSA Type 7 to Type 5 Translation | Supports NSSA external route behavior, Type 7 LSAs, ABR translation to Type 5 LSAs, and `show ip ospf database nssa-external` verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, Stub Areas | Supports stub area rules, Type 4 and Type 5 LSA suppression, and default Type 3 injection by the ABR |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, Totally Stubby Areas | Supports `area stub no-summary`, Type 3, Type 4, and Type 5 suppression, and default route replacement |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, NSSA and Totally NSSA | Supports NSSA redistribution, Type 7 LSAs, Type 7 to Type 5 translation, optional NSSA default route injection, and totally NSSA behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Stub Area Configuration Troubleshooting | Supports failure checks for mismatched area flags, missing default routes, wrong area type, and unexpected route suppression |
| Related labs | `ospf-stub-final`, `ospf-totally-stub-final`, `ospf-nssa-final`, `ospf-totally-nssa-final` | Main labs for stub, totally stub, NSSA, and totally NSSA behavior |
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Normal area | Allows intra-area, interarea, and external OSPF routes. Type 1, Type 2, Type 3, Type 4, and Type 5 LSAs may exist depending on topology |
| Stub area | Blocks Type 4 and Type 5 LSAs from entering the area. Allows Type 3 interarea LSAs. ABR injects a default route |
| Totally stub area | Blocks Type 3, Type 4, and Type 5 LSAs from entering the area. ABR injects a default route. Internal routers mainly see intra-area routes plus default |
| NSSA | Blocks Type 5 LSAs from entering the area, but allows redistribution inside the area as Type 7 LSAs |
| Totally NSSA | Blocks Type 3 and Type 5 LSAs from entering, while still allowing local redistribution as Type 7 LSAs. ABR injects default |
| ABR role | The ABR enforces what LSAs are allowed into the stub or NSSA area |
| Stub area default route | Stub and totally stub areas receive default as a Type 3 summary LSA from the ABR |
| NSSA default route | NSSA default route is not automatic in the same way. Use `area <area-id> nssa default-information-originate` on the ABR when needed |
| Totally NSSA default route | `area <area-id> nssa no-summary` on the ABR suppresses summaries and injects default into the NSSA |
| Area type flag | All routers in the area must agree on the stub or NSSA area type, or adjacencies fail |
| `no-summary` placement | For totally stub and totally NSSA, configure `no-summary` only on the ABR. Internal routers use normal `stub` or `nssa` |
| Type 7 LSA | NSSA external LSA originated by an ASBR inside an NSSA |
| Type 7 to Type 5 translation | NSSA ABR translates Type 7 LSAs into Type 5 LSAs for the rest of the OSPF domain |
| ASBR in stub area | A normal stub or totally stub area cannot contain an ASBR doing redistribution |
| ASBR in NSSA | NSSA and totally NSSA exist specifically to allow redistribution inside an otherwise stub-like area |
| Backbone restriction | Area 0 cannot be configured as stub, totally stub, NSSA, or totally NSSA |
| Virtual link restriction | Stub and NSSA style areas should not be used as virtual-link transit areas |
| Visibility tradeoff | Stub designs reduce LSDB and routing table size, but reduce route visibility and can create suboptimal routing if used carelessly |
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the area type being configured | Design step | `stub`, `totally stub`, `nssa`, or `totally nssa` | Correct area behavior is selected before configuration |
| 2 | Confirm the target area is not Area 0 | Design step | `Target area: <area-id>` | Area ID is nonzero |
| 3 | Identify the ABR for the area | Design step | `ABR: <router-name>` | ABR has one interface in Area 0 and one interface in target area |
| 4 | Identify internal routers in the target area | Design step | `Internal routers: <router-list>` | All routers that need matching area type are known |
| 5 | Confirm whether redistribution is needed inside the area | Design step | `Redistribution required: yes/no` | Stub or totally stub is rejected if local redistribution is required |
| 6 | Confirm OSPF adjacencies are healthy before changing area type | All involved routers | `show ip ospf neighbor` | Existing neighbors are `FULL` before conversion |
| 7 | Confirm current area membership | All involved routers | `show ip ospf interface brief` | Interfaces are in the intended areas |
| 8 | Confirm current route visibility before the change | Internal area routers | `show ip route ospf` | Baseline `O`, `O IA`, `O E1`, `O E2`, `O N1`, or `O N2` routes are known |
| 9 | Confirm current LSDB before the change | Internal area routers | `show ip ospf database` | Existing summary, external, and NSSA external LSAs are known |
| 10 | Enter global configuration mode | All routers in target area | `configure terminal` | Router enters global configuration mode |
| 11 | Enter the OSPF process | All routers in target area | `router ospf <process-id>` | OSPF router configuration mode opens |
| 12 | Configure a standard stub area on internal routers | Internal routers | `area <area-id> stub` | Routers set the stub area flag |
| 13 | Configure a standard stub area on the ABR | ABR | `area <area-id> stub` | ABR blocks Type 4 and Type 5 LSAs into the area and injects default |
| 14 | Configure a totally stub area on internal routers | Internal routers | `area <area-id> stub` | Internal routers match the stub area flag |
| 15 | Configure a totally stub area on the ABR | ABR | `area <area-id> stub no-summary` | ABR blocks Type 3, Type 4, and Type 5 LSAs into the area and injects default |
| 16 | Configure an NSSA on internal routers | Internal routers | `area <area-id> nssa` | Routers set the NSSA area flag |
| 17 | Configure an NSSA on the ABR | ABR | `area <area-id> nssa` | ABR blocks Type 5 LSAs into the area and allows Type 7 translation behavior |
| 18 | Inject a default route into an NSSA when required | ABR | `area <area-id> nssa default-information-originate` | NSSA receives a default route as an NSSA external route |
| 19 | Configure a totally NSSA on internal routers | Internal routers | `area <area-id> nssa` | Internal routers match the NSSA area flag |
| 20 | Configure a totally NSSA on the ABR | ABR | `area <area-id> nssa no-summary` | ABR blocks Type 3 and Type 5 LSAs into the area and injects default |
| 21 | Configure redistribution only inside NSSA or totally NSSA if required | NSSA ASBR | `redistribute <source-protocol> subnets` | Redistributed routes originate as Type 7 LSAs |
| 22 | Exit configuration mode | All involved routers | `end` | Router returns to privileged EXEC mode |
| 23 | Save the configuration | All involved routers | `write memory` | Configuration is saved |
| 24 | Verify area type on the ABR | ABR | `show ip ospf` | Area shows stub or NSSA behavior as expected |
| 25 | Verify area type on internal routers | Internal routers | `show ip ospf` | Internal routers show the correct stub or NSSA area type |
| 26 | Verify adjacency recovery after area type change | All involved routers | `show ip ospf neighbor` | Neighbors return to `FULL` after matching area type flags |
| 27 | Verify standard stub route table behavior | Stub internal routers | `show ip route ospf` | External `O E1` and `O E2` routes are absent, `O IA` routes may remain, default route exists |
| 28 | Verify totally stub route table behavior | Totally stub internal routers | `show ip route ospf` | Most `O IA`, `O E1`, and `O E2` routes are absent, default route exists |
| 29 | Verify NSSA route table behavior | NSSA routers | `show ip route ospf` | Type 5 external routes from outside are absent, local redistributed routes appear as `O N1` or `O N2` |
| 30 | Verify totally NSSA route table behavior | Totally NSSA internal routers | `show ip route ospf` | Interarea and outside external detail is suppressed, default route exists, local NSSA externals may exist |
| 31 | Verify Type 5 LSAs are blocked from stub or NSSA area | Internal routers | `show ip ospf database external` | No unwanted Type 5 LSAs are present inside the area |
| 32 | Verify Type 3 summary behavior | Internal routers | `show ip ospf database summary` | Stub keeps summaries, totally stub and totally NSSA suppress most summaries except default |
| 33 | Verify NSSA Type 7 LSAs if redistribution is used | NSSA routers | `show ip ospf database nssa-external` | Redistributed NSSA prefixes appear as Type 7 LSAs |
| 34 | Verify Type 7 to Type 5 translation outside NSSA | Backbone or normal area routers | `show ip ospf database external` | NSSA redistributed route appears outside NSSA as Type 5 |
| 35 | Verify default route presence | Internal routers | `show ip route 0.0.0.0` | Stub, totally stub, NSSA with default injection, and totally NSSA have expected default route |
| 36 | Verify reachability to external or interarea destinations | Internal routers | `ping <remote-prefix-test-ip>` | Traffic exits area through ABR default or specific interarea route as designed |
| 37 | Verify forwarding path exits through correct ABR | Internal routers | `traceroute <remote-prefix-test-ip>` | Path follows expected ABR |
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Skeleton
! =========================================================
! Standard stub area
! Blocks Type 4 and Type 5 LSAs.
! Allows Type 3 interarea LSAs.
! Configure on every router in the area.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> stub
end
write memory
! =========================================================
! Totally stub area
! Internal routers use normal stub.
! ABR uses stub no-summary.
! Blocks Type 3, Type 4, and Type 5 LSAs into the area.
! =========================================================
! Internal router:
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> stub
end
write memory
! ABR:
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> stub no-summary
end
write memory
! =========================================================
! NSSA
! Blocks Type 5 LSAs from entering.
! Allows redistribution inside the area as Type 7 LSAs.
! Configure nssa on every router in the area.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> nssa
end
write memory
! =========================================================
! NSSA with ABR default route injection
! Use on the NSSA ABR when the NSSA needs a default route.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> nssa default-information-originate
end
write memory
! =========================================================
! NSSA ASBR redistribution example
! Use only inside NSSA or totally NSSA when redistribution is required.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> nssa
 redistribute <SOURCE_PROTOCOL> subnets
end
write memory
! =========================================================
! Totally NSSA
! Internal routers use normal nssa.
! ABR uses nssa no-summary.
! Blocks Type 3 and Type 5 LSAs from entering.
! Allows local redistribution as Type 7.
! =========================================================
! Internal router:
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> nssa
end
write memory
! ABR:
configure terminal
router ospf <PROCESS_ID>
 area <AREA_ID> nssa no-summary
end
write memory
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Verification_Commands
show ip ospf
show ip ospf interface brief
show ip ospf neighbor
show ip protocols
show running-config | section router ospf
show ip route ospf
show ip route 0.0.0.0
show ip ospf database
show ip ospf database summary
show ip ospf database asbr-summary
show ip ospf database external
show ip ospf database nssa-external
show ip ospf database nssa-external <prefix>
show ip route <external-prefix>
show ip route <interarea-prefix>
show ip route <nssa-external-prefix>
ping <remote-prefix-test-ip>
traceroute <remote-prefix-test-ip>
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Rollback
! =========================================================
! Remove standard stub area
! Must be removed from all routers in the area to restore adjacency.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> stub
end
write memory
! =========================================================
! Remove totally stub ABR setting
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> stub no-summary
 area <AREA_ID> stub
end
write memory
! =========================================================
! Convert totally stub back to normal area
! Remove stub from ABR and all internal routers.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> stub
end
write memory
! =========================================================
! Remove NSSA area
! Must be removed from all routers in the area to restore normal area behavior.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> nssa
end
write memory
! =========================================================
! Remove NSSA default injection only
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> nssa default-information-originate
 area <AREA_ID> nssa
end
write memory
! =========================================================
! Remove totally NSSA ABR setting
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> nssa no-summary
 area <AREA_ID> nssa
end
write memory
! =========================================================
! Remove redistribution from NSSA ASBR
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no redistribute <SOURCE_PROTOCOL> subnets
end
write memory
! =========================================================
! Lab-only OSPF reset after area type changes
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| OSPF neighbor drops after area type change | Area type flag mismatch | `show ip ospf neighbor`, `show ip ospf interface brief`, `show ip ospf` | Configure matching `area <area-id> stub` or `area <area-id> nssa` on every router in the area |
| Stub area does not form adjacency | One router is normal area and the other is stub | `show ip ospf` | Apply `area <area-id> stub` to all routers in the area |
| NSSA does not form adjacency | One router is normal or stub while the other is NSSA | `show ip ospf` | Apply `area <area-id> nssa` to all routers in the area |
| Totally stub internal router has `no-summary` configured | Misplaced `no-summary` keyword | `show running-config | section router ospf` | Remove `no-summary` from internal routers. Keep `area <area-id> stub` internally and `area <area-id> stub no-summary` on the ABR |
| Totally NSSA internal router has `no-summary` configured | Misplaced `no-summary` keyword | `show running-config | section router ospf` | Remove `no-summary` from internal routers. Keep `area <area-id> nssa` internally and `area <area-id> nssa no-summary` on the ABR |
| External routes still appear in a stub area | Area is not actually stub on the ABR or router is checking another area | `show ip ospf database external`, `show ip ospf` | Correct area type on ABR and internal routers |
| Type 3 interarea routes still appear in standard stub | Normal behavior | `show ip route ospf`, `show ip ospf database summary` | No fix needed. Standard stub allows Type 3 LSAs |
| Type 3 routes still appear in totally stub | ABR missing `no-summary` | `show running-config | section router ospf` on ABR | Configure `area <area-id> stub no-summary` on ABR |
| Type 3 routes still appear in totally NSSA | ABR missing `no-summary` | `show running-config | section router ospf` on ABR | Configure `area <area-id> nssa no-summary` on ABR |
| Default route missing in stub area | ABR does not have Area 0 attachment or area type is wrong | `show ip ospf`, `show ip route 0.0.0.0` | Restore ABR Area 0 adjacency and correct stub area config |
| Default route missing in NSSA | NSSA default injection is not automatic | `show running-config | section router ospf`, `show ip route 0.0.0.0` | Configure `area <area-id> nssa default-information-originate` on the ABR |
| Default route missing in totally NSSA | ABR missing `no-summary` or no valid backbone attachment | `show ip ospf`, `show running-config | section router ospf` | Configure `area <area-id> nssa no-summary` on ABR and verify Area 0 adjacency |
| Redistribution fails in stub area | Stub and totally stub areas cannot contain an ASBR | `show running-config | section router ospf`, `show ip ospf database external` | Use NSSA or totally NSSA if redistribution is required |
| NSSA redistributed route not visible inside NSSA | Redistribution missing or route source not present | `show ip route`, `show running-config | section router ospf` | Restore source route and configure `redistribute <source-protocol> subnets` |
| NSSA redistributed route not visible outside NSSA | Type 7 to Type 5 translation issue or ABR problem | `show ip ospf database nssa-external`, `show ip ospf database external` | Verify NSSA ABR, P-bit behavior, and Area 0 adjacency |
| Backbone router does not see NSSA external route | ABR not translating Type 7 to Type 5 | `show ip ospf database nssa-external`, `show ip ospf database external` | Verify NSSA ABR role and Type 7 LSA presence |
| Route appears as `O N1` or `O N2` inside NSSA | Normal NSSA external route behavior | `show ip route ospf` | No fix needed unless metric type is wrong |
| Route appears as `O E1` or `O E2` outside NSSA | Normal Type 7 to Type 5 translation behavior | `show ip route ospf`, `show ip ospf database external` | No fix needed |
| Area 0 was configured as stub or NSSA | Invalid design | `show running-config | section router ospf` | Remove stub or NSSA config from Area 0 |
| Virtual link fails through area | Transit area is stub or NSSA style | `show ip ospf virtual-links`, `show ip ospf` | Use a normal transit area or redesign the backbone |
| Traffic exits through wrong ABR | Default-only area hides detailed routes | `traceroute <destination>`, `show ip route 0.0.0.0` | Adjust ABR costs, area type, or avoid totally stub behavior in redundant designs |
##### Source_Basis
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Mental_Model
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Configuration_Checklist
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Skeleton
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Verification_Commands
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Rollback
# OSPF_Stub_Totally_Stub_NSSA_Totally_NSSA_Failure_Checks
# index of each title throughout note, not in table format

