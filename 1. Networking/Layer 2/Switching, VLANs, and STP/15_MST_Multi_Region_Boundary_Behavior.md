MST_Multi_Region_Boundary_Behavior.md
# MST_Multi_Region_Boundary_Behavior

# MST_Multi_Region_Boundary_Behavior_Index

- MST_Multi_Region_Boundary_Behavior_Mental_Model
- MST_Multi_Region_Boundary_Behavior_Configuration_Checklist
- MST_Multi_Region_Boundary_Behavior_Skeleton
- MST_Multi_Region_Boundary_Behavior_Verification_Commands
- MST_Multi_Region_Boundary_Behavior_Rollback
- MST_Multi_Region_Boundary_Behavior_Failure_Checks

# MST_Multi_Region_Boundary_Behavior_Mental_Model

| Concept | Operational Meaning |
|---|---|
| MST region | A group of MST switches with the same region name, revision number, and VLAN-to-instance mapping |
| Region boundary | A link where the neighboring switch does not share the same MST region identity |
| Boundary trigger | Region boundary forms when MST name, revision, or VLAN mapping differs |
| Internal port | A port connected to another switch in the same MST region |
| Boundary port | A port connected to a different MST region, PVST/RPVST domain, or non-MST STP domain |
| Virtual switch behavior | An MST region appears like one logical bridge to switches outside the region |
| MSTI containment | MST instances are internal to a region and do not extend transparently across region boundaries |
| CIST | Common and Internal Spanning Tree is the STP view used across region boundaries |
| IST | MST instance 0 inside a region; it carries internal region control information |
| MST0 importance | MST0/IST/CIST behavior controls how the region connects to other regions |
| Per-instance root | Each region can have different roots for MST1, MST2, and other MSTIs internally |
| Boundary simplification | Outside a region, the internal MSTI details are hidden behind the CIST view |
| Cross-region VLAN traffic | VLANs can still cross the trunk, but their MSTI topology does not span both regions as one shared MSTI |
| Same VLAN, different region | VLAN 10 can exist on both sides, but the MST calculation is separated at the boundary |
| Region mismatch symptom | A port expected to be internal shows as boundary in MST output |
| Design rule | Make region boundaries intentional; accidental region mismatch is a classic MST failure |
| Verification rule | Always verify region config, port boundary status, instance roots, trunk VLANs, and final forwarding state |

# MST_Multi_Region_Boundary_Behavior_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm physical switch adjacency | SW1/SW2/SW3/SW4 | `show cdp neighbors` | Neighbor relationships match the intended multi-region topology |
| 2 | Confirm interswitch links are physically up | SW1/SW2/SW3/SW4 | `show interfaces status` | Interswitch interfaces show `connected` |
| 3 | Confirm trunk links are operating | SW1/SW2/SW3/SW4 | `show interfaces trunk` | Interswitch links show `trunking` where expected |
| 4 | Confirm current STP mode before MST changes | SW1/SW2/SW3/SW4 | `show spanning-tree summary` | Existing STP mode is documented |
| 5 | Confirm existing VLAN database | SW1/SW2/SW3/SW4 | `show vlan brief` | Existing VLANs are documented before changes |
| 6 | Confirm current MST configuration if any | SW1/SW2/SW3/SW4 | `show spanning-tree mst configuration` | Existing region name, revision, and mappings are known |
| 7 | Enter configuration mode on Region A root | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 8 | Create VLAN 10 in Region A | SW1 | `vlan 10` | VLAN 10 exists locally |
| 9 | Name VLAN 10 | SW1 | `name USERS_A` | VLAN has a readable name |
| 10 | Create VLAN 20 in Region A | SW1 | `vlan 20` | VLAN 20 exists locally |
| 11 | Name VLAN 20 | SW1 | `name SERVERS_A` | VLAN has a readable name |
| 12 | Enable MST mode | SW1 | `spanning-tree mode mst` | SW1 runs MST |
| 13 | Enter MST configuration submode | SW1 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 14 | Set Region A name | SW1 | `name REGION_A` | Region A name is configured |
| 15 | Set Region A revision | SW1 | `revision 10` | Region A revision is configured |
| 16 | Map VLANs 10 and 20 to MST1 | SW1 | `instance 1 vlan 10,20` | VLANs 10 and 20 map to MST1 |
| 17 | Exit MST configuration submode | SW1 | `exit` | MST configuration is applied |
| 18 | Configure SW1 as Region A MST0 root | SW1 | `spanning-tree mst 0 root primary` | SW1 becomes preferred CIST/IST root in Region A |
| 19 | Configure SW1 as Region A MST1 root | SW1 | `spanning-tree mst 1 root primary` | SW1 becomes preferred MST1 root in Region A |
| 20 | Configure trunk toward SW2 | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 21 | Force trunk mode toward SW2 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 22 | Allow VLANs 10 and 20 toward SW2 | SW1 | `switchport trunk allowed vlan 10,20` | Required VLANs are allowed on trunk |
| 23 | Bring up trunk toward SW2 | SW1 | `no shutdown` | Interface is administratively up |
| 24 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 25 | Enter configuration mode on Region A member | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 26 | Create VLAN 10 | SW2 | `vlan 10` | VLAN 10 exists locally |
| 27 | Create VLAN 20 | SW2 | `vlan 20` | VLAN 20 exists locally |
| 28 | Enable MST mode | SW2 | `spanning-tree mode mst` | SW2 runs MST |
| 29 | Enter MST configuration submode | SW2 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 30 | Set identical Region A name | SW2 | `name REGION_A` | Name matches SW1 |
| 31 | Set identical Region A revision | SW2 | `revision 10` | Revision matches SW1 |
| 32 | Map VLANs 10 and 20 to MST1 | SW2 | `instance 1 vlan 10,20` | Mapping matches SW1 |
| 33 | Exit MST configuration submode | SW2 | `exit` | MST configuration is applied |
| 34 | Configure SW2 as Region A backup root for MST0 | SW2 | `spanning-tree mst 0 root secondary` | SW2 is backup root in Region A |
| 35 | Configure SW2 as Region A backup root for MST1 | SW2 | `spanning-tree mst 1 root secondary` | SW2 is backup MST1 root in Region A |
| 36 | Configure internal trunk toward SW1 | SW2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 37 | Force trunk mode toward SW1 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 38 | Allow VLANs 10 and 20 toward SW1 | SW2 | `switchport trunk allowed vlan 10,20` | Required VLANs are allowed |
| 39 | Bring up trunk toward SW1 | SW2 | `no shutdown` | Interface is administratively up |
| 40 | Configure boundary trunk toward Region B | SW2 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 41 | Force trunk mode toward Region B | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 42 | Allow VLANs 10 and 20 across the boundary | SW2 | `switchport trunk allowed vlan 10,20` | VLANs can cross the region boundary |
| 43 | Bring up boundary trunk | SW2 | `no shutdown` | Boundary trunk is administratively up |
| 44 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 45 | Enter configuration mode on Region B root | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 46 | Create VLAN 10 in Region B | SW3 | `vlan 10` | VLAN 10 exists locally |
| 47 | Name VLAN 10 | SW3 | `name USERS_B` | VLAN has a readable name |
| 48 | Create VLAN 20 in Region B | SW3 | `vlan 20` | VLAN 20 exists locally |
| 49 | Name VLAN 20 | SW3 | `name SERVERS_B` | VLAN has a readable name |
| 50 | Enable MST mode | SW3 | `spanning-tree mode mst` | SW3 runs MST |
| 51 | Enter MST configuration submode | SW3 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 52 | Set Region B name | SW3 | `name REGION_B` | Region B name differs from Region A |
| 53 | Set Region B revision | SW3 | `revision 20` | Region B revision differs from Region A |
| 54 | Map VLANs 10 and 20 to MST1 | SW3 | `instance 1 vlan 10,20` | VLANs 10 and 20 map to MST1 inside Region B |
| 55 | Exit MST configuration submode | SW3 | `exit` | MST configuration is applied |
| 56 | Configure SW3 as Region B MST0 root | SW3 | `spanning-tree mst 0 root primary` | SW3 becomes preferred CIST/IST root inside Region B |
| 57 | Configure SW3 as Region B MST1 root | SW3 | `spanning-tree mst 1 root primary` | SW3 becomes preferred MST1 root inside Region B |
| 58 | Configure boundary trunk toward Region A | SW3 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 59 | Force trunk mode toward Region A | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 60 | Allow VLANs 10 and 20 across the boundary | SW3 | `switchport trunk allowed vlan 10,20` | VLANs can cross the region boundary |
| 61 | Bring up boundary trunk | SW3 | `no shutdown` | Boundary trunk is administratively up |
| 62 | Configure internal trunk toward SW4 | SW3 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 63 | Force trunk mode toward SW4 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 64 | Allow VLANs 10 and 20 toward SW4 | SW3 | `switchport trunk allowed vlan 10,20` | Required VLANs are allowed |
| 65 | Bring up trunk toward SW4 | SW3 | `no shutdown` | Interface is administratively up |
| 66 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 67 | Enter configuration mode on Region B member | SW4 | `configure terminal` | Prompt changes to global configuration mode |
| 68 | Create VLAN 10 | SW4 | `vlan 10` | VLAN 10 exists locally |
| 69 | Create VLAN 20 | SW4 | `vlan 20` | VLAN 20 exists locally |
| 70 | Enable MST mode | SW4 | `spanning-tree mode mst` | SW4 runs MST |
| 71 | Enter MST configuration submode | SW4 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 72 | Set identical Region B name | SW4 | `name REGION_B` | Name matches SW3 |
| 73 | Set identical Region B revision | SW4 | `revision 20` | Revision matches SW3 |
| 74 | Map VLANs 10 and 20 to MST1 | SW4 | `instance 1 vlan 10,20` | Mapping matches SW3 |
| 75 | Exit MST configuration submode | SW4 | `exit` | MST configuration is applied |
| 76 | Configure SW4 as Region B backup root for MST0 | SW4 | `spanning-tree mst 0 root secondary` | SW4 is backup root in Region B |
| 77 | Configure SW4 as Region B backup root for MST1 | SW4 | `spanning-tree mst 1 root secondary` | SW4 is backup MST1 root in Region B |
| 78 | Configure internal trunk toward SW3 | SW4 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 79 | Force trunk mode toward SW3 | SW4 | `switchport mode trunk` | Interface is administratively trunking |
| 80 | Allow VLANs 10 and 20 toward SW3 | SW4 | `switchport trunk allowed vlan 10,20` | Required VLANs are allowed |
| 81 | Bring up trunk toward SW3 | SW4 | `no shutdown` | Interface is administratively up |
| 82 | Exit SW4 configuration mode | SW4 | `end` | Prompt returns to privileged EXEC mode |
| 83 | Verify all switches are running MST | SW1/SW2/SW3/SW4 | `show spanning-tree summary` | STP mode shows MST |
| 84 | Verify Region A identity | SW1/SW2 | `show spanning-tree mst configuration` | SW1 and SW2 show `REGION_A`, revision 10, and matching mappings |
| 85 | Verify Region B identity | SW3/SW4 | `show spanning-tree mst configuration` | SW3 and SW4 show `REGION_B`, revision 20, and matching mappings |
| 86 | Verify Region A internal port status | SW1/SW2 | `show spanning-tree mst interface GigabitEthernet0/1` | Link between SW1 and SW2 is internal, not boundary |
| 87 | Verify Region B internal port status | SW3/SW4 | `show spanning-tree mst interface GigabitEthernet0/2` | Link between SW3 and SW4 is internal, not boundary |
| 88 | Verify SW2 sees SW3 link as boundary | SW2 | `show spanning-tree mst interface GigabitEthernet0/2` | Link toward SW3 is boundary |
| 89 | Verify SW3 sees SW2 link as boundary | SW3 | `show spanning-tree mst interface GigabitEthernet0/1` | Link toward SW2 is boundary |
| 90 | Verify MST0/CIST behavior across boundary | SW2/SW3 | `show spanning-tree mst 0` | Boundary link participates in CIST behavior |
| 91 | Verify MST1 remains region-local | SW1/SW2/SW3/SW4 | `show spanning-tree mst 1` | Each region calculates MST1 internally |
| 92 | Verify trunks carry VLANs across boundary | SW2/SW3 | `show interfaces trunk` | VLANs 10 and 20 are allowed and active on the boundary trunk |
| 93 | Verify no inconsistent ports exist | SW1/SW2/SW3/SW4 | `show spanning-tree inconsistentports` | No ports are listed |
| 94 | Verify host reachability across the boundary if intended | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works if VLANs are carried and STP permits forwarding |
| 95 | Save configuration | SW1/SW2/SW3/SW4 | `write memory` | MST region and boundary configuration survives reload |

# MST_Multi_Region_Boundary_Behavior_Skeleton

Region A, SW1 root:

configure terminal
!
vlan 10
 name USERS_A
vlan 20
 name SERVERS_A
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name REGION_A
 revision 10
 instance 1 vlan 10,20
 exit
!
spanning-tree mst 0 root primary
spanning-tree mst 1 root primary
!
interface GigabitEthernet0/1
 description REGION_A_INTERNAL_TO_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

Region A, SW2 member and boundary switch:

configure terminal
!
vlan 10
 name USERS_A
vlan 20
 name SERVERS_A
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name REGION_A
 revision 10
 instance 1 vlan 10,20
 exit
!
spanning-tree mst 0 root secondary
spanning-tree mst 1 root secondary
!
interface GigabitEthernet0/1
 description REGION_A_INTERNAL_TO_SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
interface GigabitEthernet0/2
 description BOUNDARY_TO_REGION_B_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

Region B, SW3 root and boundary switch:

configure terminal
!
vlan 10
 name USERS_B
vlan 20
 name SERVERS_B
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name REGION_B
 revision 20
 instance 1 vlan 10,20
 exit
!
spanning-tree mst 0 root primary
spanning-tree mst 1 root primary
!
interface GigabitEthernet0/1
 description BOUNDARY_TO_REGION_A_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
interface GigabitEthernet0/2
 description REGION_B_INTERNAL_TO_SW4
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

Region B, SW4 member:

configure terminal
!
vlan 10
 name USERS_B
vlan 20
 name SERVERS_B
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name REGION_B
 revision 20
 instance 1 vlan 10,20
 exit
!
spanning-tree mst 0 root secondary
spanning-tree mst 1 root secondary
!
interface GigabitEthernet0/1
 description REGION_B_INTERNAL_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

Boundary verification:

show spanning-tree mst configuration
show spanning-tree mst
show spanning-tree mst 0
show spanning-tree mst 1
show spanning-tree mst interface GigabitEthernet0/1
show spanning-tree mst interface GigabitEthernet0/2
show interfaces trunk
show spanning-tree inconsistentports

Intentional boundary creation methods:

Method 1, different region name:

spanning-tree mst configuration
 name REGION_A
 revision 10
 instance 1 vlan 10,20
 exit

spanning-tree mst configuration
 name REGION_B
 revision 10
 instance 1 vlan 10,20
 exit

Method 2, different revision:

spanning-tree mst configuration
 name CAMPUS
 revision 10
 instance 1 vlan 10,20
 exit

spanning-tree mst configuration
 name CAMPUS
 revision 20
 instance 1 vlan 10,20
 exit

Method 3, different mapping:

spanning-tree mst configuration
 name CAMPUS
 revision 10
 instance 1 vlan 10,20
 exit

spanning-tree mst configuration
 name CAMPUS
 revision 10
 instance 1 vlan 10
 instance 2 vlan 20
 exit

# MST_Multi_Region_Boundary_Behavior_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm MST mode | SW1/SW2/SW3/SW4 | `show spanning-tree summary` | Switches show MST mode |
| Confirm VLAN database | SW1/SW2/SW3/SW4 | `show vlan brief` | VLANs 10 and 20 exist and are active |
| Confirm trunk state | SW1/SW2/SW3/SW4 | `show interfaces trunk` | Interswitch links show trunking |
| Confirm VLANs cross trunks | SW1/SW2/SW3/SW4 | `show interfaces trunk` | VLANs 10 and 20 are allowed and active |
| Confirm Region A config | SW1/SW2 | `show spanning-tree mst configuration` | Name, revision, and VLAN mapping match inside Region A |
| Confirm Region B config | SW3/SW4 | `show spanning-tree mst configuration` | Name, revision, and VLAN mapping match inside Region B |
| Confirm Region A differs from Region B | SW2/SW3 | `show spanning-tree mst configuration` | Name, revision, or mapping differs between regions |
| Confirm Region A internal link | SW1/SW2 | `show spanning-tree mst interface GigabitEthernet0/1` | SW1 to SW2 link is internal |
| Confirm Region B internal link | SW3/SW4 | `show spanning-tree mst interface GigabitEthernet0/2` | SW3 to SW4 link is internal |
| Confirm SW2 boundary port | SW2 | `show spanning-tree mst interface GigabitEthernet0/2` | Link toward SW3 is boundary |
| Confirm SW3 boundary port | SW3 | `show spanning-tree mst interface GigabitEthernet0/1` | Link toward SW2 is boundary |
| Confirm MST0 across boundary | SW2/SW3 | `show spanning-tree mst 0` | CIST behavior is visible on the boundary |
| Confirm MST1 inside Region A | SW1/SW2 | `show spanning-tree mst 1` | Region A calculates MST1 internally |
| Confirm MST1 inside Region B | SW3/SW4 | `show spanning-tree mst 1` | Region B calculates MST1 internally |
| Confirm Region A MST0 root | SW1/SW2 | `show spanning-tree mst 0` | Region A MST0 root matches intended design |
| Confirm Region A MST1 root | SW1/SW2 | `show spanning-tree mst 1` | Region A MST1 root matches intended design |
| Confirm Region B MST0 root | SW3/SW4 | `show spanning-tree mst 0` | Region B MST0 root matches intended design |
| Confirm Region B MST1 root | SW3/SW4 | `show spanning-tree mst 1` | Region B MST1 root matches intended design |
| Confirm no inconsistent ports | SW1/SW2/SW3/SW4 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop symptoms | SW1/SW2/SW3/SW4 | `show logging | include SPANTREE|MST|MACFLAP|LOOP` | No loop, MAC flap, or MST mismatch issue is active |
| Confirm MAC learning in VLAN 10 | SW1/SW2/SW3/SW4 | `show mac address-table dynamic vlan 10` | MAC addresses are stable on expected ports |
| Confirm MAC learning in VLAN 20 | SW1/SW2/SW3/SW4 | `show mac address-table dynamic vlan 20` | MAC addresses are stable on expected ports |
| Confirm host reachability in VLAN 10 | HOSTS | `ping <remote-vlan-10-host-ip>` | Same-VLAN traffic works if the design allows VLAN 10 across the boundary |
| Confirm host reachability in VLAN 20 | HOSTS | `ping <remote-vlan-20-host-ip>` | Same-VLAN traffic works if the design allows VLAN 20 across the boundary |
| Confirm running MST config | SW1/SW2/SW3/SW4 | `show running-config | section spanning-tree mst configuration` | Region name, revision, and mappings are present |
| Confirm saved MST config | SW1/SW2/SW3/SW4 | `show startup-config | section spanning-tree mst configuration` | Startup config contains intended MST region settings |

# MST_Multi_Region_Boundary_Behavior_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current MST configuration before rollback | SW1/SW2/SW3/SW4 | `show spanning-tree mst configuration` | Region name, revision, and mapping are documented |
| 2 | Record current MST topology before rollback | SW1/SW2/SW3/SW4 | `show spanning-tree mst` | Root, roles, states, and boundary behavior are documented |
| 3 | Record trunk state before rollback | SW1/SW2/SW3/SW4 | `show interfaces trunk` | Current trunk and VLAN forwarding state is documented |
| 4 | Decide whether rollback means one shared region or no MST | Operator | `N/A` | Rollback target is clear before changes |
| 5 | If merging regions, enter MST config on SW3 | SW3 | `configure terminal` then `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 6 | Change SW3 to Region A name | SW3 | `name REGION_A` | SW3 region name matches Region A |
| 7 | Change SW3 to Region A revision | SW3 | `revision 10` | SW3 revision matches Region A |
| 8 | Change SW3 mapping to Region A mapping | SW3 | `instance 1 vlan 10,20` | SW3 VLAN mapping matches Region A |
| 9 | Exit SW3 MST configuration mode | SW3 | `exit` | MST region change is applied |
| 10 | Exit SW3 global configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 11 | If merging regions, enter MST config on SW4 | SW4 | `configure terminal` then `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 12 | Change SW4 to Region A name | SW4 | `name REGION_A` | SW4 region name matches Region A |
| 13 | Change SW4 to Region A revision | SW4 | `revision 10` | SW4 revision matches Region A |
| 14 | Change SW4 mapping to Region A mapping | SW4 | `instance 1 vlan 10,20` | SW4 VLAN mapping matches Region A |
| 15 | Exit SW4 MST configuration mode | SW4 | `exit` | MST region change is applied |
| 16 | Exit SW4 global configuration mode | SW4 | `end` | Prompt returns to privileged EXEC mode |
| 17 | Verify former boundary became internal after region merge | SW2/SW3 | `show spanning-tree mst interface GigabitEthernet0/2` | SW2 to SW3 link no longer shows as boundary |
| 18 | Verify unified region configuration | SW1/SW2/SW3/SW4 | `show spanning-tree mst configuration` | All switches share the same name, revision, and mapping |
| 19 | If removing MST entirely, change STP mode to Rapid PVST+ | SW1/SW2/SW3/SW4 | `configure terminal` then `spanning-tree mode rapid-pvst` then `end` | Switches leave MST mode and run Rapid PVST+ |
| 20 | If removing MST lab VLANs, remove VLAN 10 | SW1/SW2/SW3/SW4 | `configure terminal` then `no vlan 10` then `end` | VLAN 10 is removed locally |
| 21 | If removing MST lab VLANs, remove VLAN 20 | SW1/SW2/SW3/SW4 | `configure terminal` then `no vlan 20` then `end` | VLAN 20 is removed locally |
| 22 | Verify final STP mode | SW1/SW2/SW3/SW4 | `show spanning-tree summary` | STP mode matches rollback target |
| 23 | Verify final boundary state | SW1/SW2/SW3/SW4 | `show spanning-tree mst` | Boundary or internal behavior matches rollback target |
| 24 | Verify no inconsistent ports remain | SW1/SW2/SW3/SW4 | `show spanning-tree inconsistentports` | No ports are listed |
| 25 | Verify trunk state after rollback | SW1/SW2/SW3/SW4 | `show interfaces trunk` | Remaining trunks operate as intended |
| 26 | Verify host reachability | HOSTS | `ping <remote-same-vlan-host-ip>` | Traffic works if VLANs remain in service |
| 27 | Save rollback | SW1/SW2/SW3/SW4 | `write memory` | Startup configuration reflects rollback |

# MST_Multi_Region_Boundary_Behavior_Failure_Checks

| Symptom                                                   | Likely Cause                                                  | Device                 | Command                                                              | Fix                                                                              |                                                  |                                |      |                                                             |
| --------------------------------------------------------- | ------------------------------------------------------------- | ---------------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------ | ---- | ----------------------------------------------------------- |
| Port unexpectedly shows boundary                          | Region name, revision, or VLAN mapping mismatch               | SW1/SW2/SW3/SW4        | `show spanning-tree mst configuration`                               | Make region identity identical if the link should be internal                    |                                                  |                                |      |                                                             |
| Port unexpectedly shows internal                          | Both switches share the same MST region identity              | SW2/SW3                | `show spanning-tree mst configuration`                               | Change name, revision, or mapping if an intentional boundary is required         |                                                  |                                |      |                                                             |
| VLAN mapping looks correct but boundary remains           | Region name or revision differs                               | SW2/SW3                | `show spanning-tree mst configuration`                               | Match name and revision if same region is intended                               |                                                  |                                |      |                                                             |
| Name and revision match but boundary remains              | VLAN-to-instance mapping differs                              | SW2/SW3                | `show spanning-tree mst configuration`                               | Correct instance VLAN mappings on both sides                                     |                                                  |                                |      |                                                             |
| Boundary link is down                                     | Physical or administrative link issue                         | SW2/SW3                | `show interfaces status`                                             | Fix cabling, CML link, or apply `no shutdown`                                    |                                                  |                                |      |                                                             |
| Boundary trunk is not trunking                            | Interface is access mode or negotiation failed                | SW2/SW3                | `show interfaces trunk`                                              | Configure `switchport mode trunk` on both sides                                  |                                                  |                                |      |                                                             |
| VLANs do not cross the boundary                           | VLANs are not allowed on the boundary trunk                   | SW2/SW3                | `show interfaces trunk`                                              | Add required VLANs to the allowed list                                           |                                                  |                                |      |                                                             |
| VLANs are allowed but inactive                            | VLANs do not exist locally                                    | SW2/SW3                | `show vlan brief`                                                    | Create VLANs on both sides                                                       |                                                  |                                |      |                                                             |
| MST1 root appears different across regions                | MSTI roots are region-local                                   | SW1/SW2/SW3/SW4        | `show spanning-tree mst 1`                                           | This is normal unless a single shared region was intended                        |                                                  |                                |      |                                                             |
| MSTI traffic path is not what was expected across regions | MST instances do not extend as one instance across boundaries | SW1/SW2/SW3/SW4        | `show spanning-tree mst 0` and `show spanning-tree mst 1`            | Validate CIST boundary behavior and local MSTI roots separately                  |                                                  |                                |      |                                                             |
| Same VLAN cannot pass between regions                     | STP boundary state, trunk allowance, or VLAN database issue   | SW2/SW3                | `show interfaces trunk`, `show vlan brief`, `show spanning-tree mst` | Fix trunk, VLAN, or CIST forwarding state                                        |                                                  |                                |      |                                                             |
| MAC addresses flap at boundary                            | Loop, mismatched trunking, or unsafe redundant path           | SW1/SW2/SW3/SW4        | `show logging                                                        | include MACFLAP                                                                  | LOOP                                             | SPANTREE                       | MST` | Verify boundary ports, blocked ports, and trunk consistency |
| Inconsistent ports appear                                 | STP protection or BPDU issue at boundary                      | SW1/SW2/SW3/SW4        | `show spanning-tree inconsistentports`                               | Inspect guard features, Bridge Assurance, BPDU flow, and region config           |                                                  |                                |      |                                                             |
| Root bridge is not as expected in MST0                    | MST0/CIST priority design is wrong                            | SW1/SW2/SW3/SW4        | `show spanning-tree mst 0`                                           | Configure `spanning-tree mst 0 root primary` or explicit priority                |                                                  |                                |      |                                                             |
| MST1 root is not as expected inside a region              | MST1 priority design is wrong                                 | Region member switches | `show spanning-tree mst 1`                                           | Configure intended region-local MST1 root                                        |                                                  |                                |      |                                                             |
| Trunk allowed list overwrote other VLANs                  | Used full replacement instead of add/remove                   | SW2/SW3                | `show running-config interface <interface>`                          | Restore full allowed list or use `switchport trunk allowed vlan add <vlan-list>` |                                                  |                                |      |                                                             |
| Native VLAN mismatch appears                              | Native VLAN differs across the boundary trunk                 | SW2/SW3                | `show logging                                                        | include NATIVE                                                                   | native`                                          | Match native VLAN on both ends |      |                                                             |
| Host traffic works inside a region but not across regions | Boundary trunk or CIST state is blocking                      | SW2/SW3                | `show spanning-tree mst 0` and `show interfaces trunk`               | Verify CIST forwarding on boundary and VLAN trunk allowance                      |                                                  |                                |      |                                                             |
| Region merge does not work after changing only one switch | All switches in the target region were not updated            | SW1/SW2/SW3/SW4        | `show spanning-tree mst configuration`                               | Update every switch to the same name, revision, and mapping                      |                                                  |                                |      |                                                             |
| Region split happens after a minor edit                   | Revision or mapping changed on one switch only                | SW1/SW2/SW3/SW4        | `show spanning-tree mst configuration`                               | Restore exact region configuration match                                         |                                                  |                                |      |                                                             |
| MST mode is not active                                    | Switch still runs PVST or Rapid PVST                          | SW1/SW2/SW3/SW4        | `show spanning-tree summary`                                         | Configure `spanning-tree mode mst`                                               |                                                  |                                |      |                                                             |
| Config disappears after reload                            | Configuration was not saved                                   | SW1/SW2/SW3/SW4        | `show startup-config                                                 | section spanning-tree mst configuration`                                         | Reapply MST configuration and run `write memory` |                                |      |                                                             |