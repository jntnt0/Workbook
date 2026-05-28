MST_PVST_RSTP_Interoperability.md
# MST_PVST_RSTP_Interoperability

# MST_PVST_RSTP_Interoperability_Index

- MST_PVST_RSTP_Interoperability_Mental_Model
- MST_PVST_RSTP_Interoperability_Configuration_Checklist
- MST_PVST_RSTP_Interoperability_Skeleton
- MST_PVST_RSTP_Interoperability_Verification_Commands
- MST_PVST_RSTP_Interoperability_Rollback
- MST_PVST_RSTP_Interoperability_Failure_Checks

# MST_PVST_RSTP_Interoperability_Mental_Model

| Concept | Operational Meaning |
|---|---|
| MST region boundary | A port becomes a boundary when it connects to PVST, Rapid PVST, classic STP, or a different MST region |
| PVST simulation | MST translates its IST/CIST information into PVST style BPDUs toward PVST or Rapid PVST neighbors |
| Rapid PVST neighbor | Rapid PVST is handled as a PVST style neighbor at the MST boundary |
| IST to PVST translation | MST does not extend every MSTI into the PVST domain; it represents the region through the IST/CIST boundary behavior |
| VLAN 1 importance | MST maps the PVST BPDU from VLAN 1 into the IST on the boundary |
| MSTI containment | MST1, MST2, and other MST instances stay inside the MST region |
| One region view | The PVST side sees the MST region like a single bridge, not as every internal MST switch |
| Safe design 1 | Make the MST region the root for all VLANs on the PVST side |
| Safe design 2 | Make the MST region the root for no VLANs on the PVST side |
| Unsafe mixed root design | Do not make MST root for some PVST VLANs and non-root for other PVST VLANs on the same boundary |
| PVST simulation check | If MST receives better PVST BPDUs for some VLANs but not consistently, the boundary can be blocked |
| Root-inconsistent result | A failed PVST simulation check can place the boundary port into root-inconsistent state |
| Boundary forwarding limitation | The MST boundary has one CIST view; it cannot independently forward one PVST VLAN and block another like native PVST can |
| Load sharing caution | PVST side can do per-VLAN tuning internally, but MST boundary behavior must stay consistent |
| Trunk dependency | VLANs must exist and be allowed on the MST to PVST trunk before interoperability matters |
| Verification rule | Always check MST configuration, boundary status, VLAN 1, root placement per VLAN, inconsistent ports, and trunk forwarding |

# MST_PVST_RSTP_Interoperability_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the physical topology before configuring interoperability | MST-SW1/PVST-SW1 | `show cdp neighbors` | MST and PVST neighbors are visible on the expected ports |
| 2 | Confirm interswitch links are physically up | MST-SW1/PVST-SW1 | `show interfaces status` | Boundary interfaces show `connected` |
| 3 | Confirm current STP mode on all switches | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree summary` | Current STP mode is documented before changes |
| 4 | Confirm existing VLAN database | MST-SW1/MST-SW2/PVST-SW1 | `show vlan brief` | VLANs are documented before changes |
| 5 | Confirm existing trunk state | MST-SW1/PVST-SW1 | `show interfaces trunk` | Current trunk status and allowed VLANs are known |
| 6 | Confirm no existing inconsistent ports | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree inconsistentports` | No inconsistent ports exist before configuration |
| 7 | Enter configuration mode on first MST switch | MST-SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 8 | Create VLAN 1 if platform requires visibility in the lab baseline | MST-SW1 | `vlan 1` | VLAN 1 exists as the default VLAN |
| 9 | Create VLAN 10 | MST-SW1 | `vlan 10` | VLAN 10 exists locally |
| 10 | Name VLAN 10 | MST-SW1 | `name USERS` | VLAN 10 has a readable name |
| 11 | Create VLAN 20 | MST-SW1 | `vlan 20` | VLAN 20 exists locally |
| 12 | Name VLAN 20 | MST-SW1 | `name SERVERS` | VLAN 20 has a readable name |
| 13 | Enable MST mode | MST-SW1 | `spanning-tree mode mst` | MST-SW1 runs MST |
| 14 | Enter MST configuration mode | MST-SW1 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 15 | Set MST region name | MST-SW1 | `name CAMPUS_MST` | Region name is configured |
| 16 | Set MST revision | MST-SW1 | `revision 10` | Revision is configured |
| 17 | Map VLANs 10 and 20 to MST instance 1 | MST-SW1 | `instance 1 vlan 10,20` | VLANs 10 and 20 map to MST1 inside the region |
| 18 | Exit MST configuration mode | MST-SW1 | `exit` | MST region configuration is applied |
| 19 | Make MST-SW1 the root for MST0 | MST-SW1 | `spanning-tree mst 0 root primary` | MST-SW1 is preferred CIST/IST root |
| 20 | Make MST-SW1 the root for MST1 | MST-SW1 | `spanning-tree mst 1 root primary` | MST-SW1 is preferred MST1 root inside the region |
| 21 | Configure boundary trunk toward PVST switch | MST-SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 22 | Force trunk mode on MST boundary port | MST-SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 23 | Carry VLAN 1, VLAN 10, and VLAN 20 across boundary | MST-SW1 | `switchport trunk allowed vlan 1,10,20` | VLAN 1, 10, and 20 are allowed on the boundary trunk |
| 24 | Bring up MST boundary trunk | MST-SW1 | `no shutdown` | Interface is administratively up |
| 25 | Exit MST-SW1 configuration mode | MST-SW1 | `end` | Prompt returns to privileged EXEC mode |
| 26 | Enter configuration mode on second MST region member | MST-SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 27 | Create VLAN 10 | MST-SW2 | `vlan 10` | VLAN 10 exists locally |
| 28 | Create VLAN 20 | MST-SW2 | `vlan 20` | VLAN 20 exists locally |
| 29 | Enable MST mode | MST-SW2 | `spanning-tree mode mst` | MST-SW2 runs MST |
| 30 | Enter MST configuration mode | MST-SW2 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 31 | Set same MST region name | MST-SW2 | `name CAMPUS_MST` | Region name matches MST-SW1 |
| 32 | Set same MST revision | MST-SW2 | `revision 10` | Revision matches MST-SW1 |
| 33 | Set same VLAN to instance mapping | MST-SW2 | `instance 1 vlan 10,20` | Mapping matches MST-SW1 |
| 34 | Exit MST configuration mode | MST-SW2 | `exit` | MST region configuration is applied |
| 35 | Make MST-SW2 the secondary root for MST0 | MST-SW2 | `spanning-tree mst 0 root secondary` | MST-SW2 is backup CIST/IST root |
| 36 | Make MST-SW2 the secondary root for MST1 | MST-SW2 | `spanning-tree mst 1 root secondary` | MST-SW2 is backup MST1 root |
| 37 | Exit MST-SW2 configuration mode | MST-SW2 | `end` | Prompt returns to privileged EXEC mode |
| 38 | Enter configuration mode on PVST or Rapid PVST switch | PVST-SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 39 | Select Rapid PVST mode if testing Rapid PVST interoperability | PVST-SW1 | `spanning-tree mode rapid-pvst` | PVST-SW1 runs Rapid PVST+ |
| 40 | Select classic PVST mode if testing PVST interoperability instead | PVST-SW1 | `spanning-tree mode pvst` | PVST-SW1 runs PVST+ |
| 41 | Create VLAN 10 on PVST side | PVST-SW1 | `vlan 10` | VLAN 10 exists locally |
| 42 | Name VLAN 10 on PVST side | PVST-SW1 | `name USERS` | VLAN 10 has a readable name |
| 43 | Create VLAN 20 on PVST side | PVST-SW1 | `vlan 20` | VLAN 20 exists locally |
| 44 | Name VLAN 20 on PVST side | PVST-SW1 | `name SERVERS` | VLAN 20 has a readable name |
| 45 | Configure boundary trunk toward MST region | PVST-SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 46 | Force trunk mode on PVST boundary port | PVST-SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 47 | Carry VLAN 1, VLAN 10, and VLAN 20 across boundary | PVST-SW1 | `switchport trunk allowed vlan 1,10,20` | VLAN 1, 10, and 20 are allowed on the boundary trunk |
| 48 | Bring up PVST boundary trunk | PVST-SW1 | `no shutdown` | Interface is administratively up |
| 49 | Make PVST bridge priority worse than MST region for all carried VLANs | PVST-SW1 | `spanning-tree vlan 1,10,20 priority 32768` | PVST side should not become root if MST-root design is used |
| 50 | Exit PVST-SW1 configuration mode | PVST-SW1 | `end` | Prompt returns to privileged EXEC mode |
| 51 | Verify MST mode on MST switches | MST-SW1/MST-SW2 | `show spanning-tree summary` | MST switches show MST mode |
| 52 | Verify PVST or Rapid PVST mode on PVST switch | PVST-SW1 | `show spanning-tree summary` | PVST switch shows PVST or Rapid PVST mode |
| 53 | Verify MST region consistency | MST-SW1/MST-SW2 | `show spanning-tree mst configuration` | Name, revision, and mapping match |
| 54 | Verify MST boundary port toward PVST | MST-SW1 | `show spanning-tree mst interface GigabitEthernet0/1` | Port toward PVST switch shows boundary behavior |
| 55 | Verify MST0/CIST root placement | MST-SW1/MST-SW2 | `show spanning-tree mst 0` | MST-SW1 is root for MST0 |
| 56 | Verify MST1 root placement | MST-SW1/MST-SW2 | `show spanning-tree mst 1` | MST-SW1 is root for MST1 |
| 57 | Verify PVST side sees MST region as root for VLAN 1 | PVST-SW1 | `show spanning-tree vlan 1` | Root ID points toward the MST boundary |
| 58 | Verify PVST side sees MST region as root for VLAN 10 | PVST-SW1 | `show spanning-tree vlan 10` | Root ID points toward the MST boundary |
| 59 | Verify PVST side sees MST region as root for VLAN 20 | PVST-SW1 | `show spanning-tree vlan 20` | Root ID points toward the MST boundary |
| 60 | Verify boundary trunk allows all required VLANs | MST-SW1/PVST-SW1 | `show interfaces trunk` | VLAN 1, 10, and 20 are allowed and active |
| 61 | Verify no PVST simulation inconsistency exists | MST-SW1/PVST-SW1 | `show spanning-tree inconsistentports` | No ports are listed |
| 62 | Verify logs do not show PVST simulation failure | MST-SW1/PVST-SW1 | `show logging | include PVST|SIM|SPANTREE|inconsistent|ROOT` | No active PVST simulation or root-inconsistent errors appear |
| 63 | Verify same-VLAN traffic across boundary | HOSTS | `ping <remote-same-vlan-host-ip>` | Traffic works if VLAN is intended to span the boundary |
| 64 | Save configuration | MST-SW1/MST-SW2/PVST-SW1 | `write memory` | MST and PVST interoperability configuration survives reload |

# MST_PVST_RSTP_Interoperability_Skeleton

MST region switch, preferred CIST and MSTI root:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name CAMPUS_MST
 revision 10
 instance 1 vlan 10,20
 exit
!
spanning-tree mst 0 root primary
spanning-tree mst 1 root primary
!
interface GigabitEthernet0/1
 description BOUNDARY_TO_PVST_SWITCH
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20
 no shutdown
!
end
write memory

Second MST region member:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name CAMPUS_MST
 revision 10
 instance 1 vlan 10,20
 exit
!
spanning-tree mst 0 root secondary
spanning-tree mst 1 root secondary
!
end
write memory

Rapid PVST boundary switch:

configure terminal
!
spanning-tree mode rapid-pvst
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
interface GigabitEthernet0/1
 description BOUNDARY_TO_MST_REGION
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20
 no shutdown
!
spanning-tree vlan 1,10,20 priority 32768
!
end
write memory

Classic PVST boundary switch alternative:

configure terminal
!
spanning-tree mode pvst
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
interface GigabitEthernet0/1
 description BOUNDARY_TO_MST_REGION
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20
 no shutdown
!
spanning-tree vlan 1,10,20 priority 32768
!
end
write memory

Safe design 1, MST region root for all PVST VLANs:

MST side:

configure terminal
!
spanning-tree mst 0 priority 4096
!
end
write memory

PVST side:

configure terminal
!
spanning-tree vlan 1,10,20 priority 32768
!
end
write memory

Safe design 2, MST region root for no PVST VLANs:

PVST side:

configure terminal
!
spanning-tree vlan 1,10,20 priority 4096
!
end
write memory

MST side:

configure terminal
!
spanning-tree mst 0 priority 32768
!
end
write memory

Do not intentionally build this mixed-root boundary design:

VLAN 10 root on MST side
VLAN 20 root on PVST side
Same MST to PVST boundary trunk carrying both VLANs

That design can fail PVST simulation checks and place the boundary into root-inconsistent state.

Boundary verification:

show spanning-tree summary
show spanning-tree mst configuration
show spanning-tree mst
show spanning-tree mst 0
show spanning-tree mst 1
show spanning-tree mst interface GigabitEthernet0/1
show spanning-tree vlan 1
show spanning-tree vlan 10
show spanning-tree vlan 20
show interfaces trunk
show spanning-tree inconsistentports
show logging | include PVST|SIM|SPANTREE|inconsistent|ROOT

# MST_PVST_RSTP_Interoperability_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm MST mode | MST-SW1/MST-SW2 | `show spanning-tree summary` | MST switches show MST mode |
| Confirm PVST or Rapid PVST mode | PVST-SW1 | `show spanning-tree summary` | PVST switch shows PVST or Rapid PVST mode |
| Confirm MST region identity | MST-SW1/MST-SW2 | `show spanning-tree mst configuration` | Name, revision, and VLAN mapping match inside MST region |
| Confirm MST topology | MST-SW1/MST-SW2 | `show spanning-tree mst` | MST0 and MST1 are visible with roles and states |
| Confirm MST0 root | MST-SW1 | `show spanning-tree mst 0` | MST-SW1 is root for MST0 in the MST-root design |
| Confirm MST1 root | MST-SW1 | `show spanning-tree mst 1` | MST-SW1 is root for MST1 inside the MST region |
| Confirm MST to PVST boundary | MST-SW1 | `show spanning-tree mst interface GigabitEthernet0/1` | Interface toward PVST switch is shown as boundary |
| Confirm boundary trunk state | MST-SW1/PVST-SW1 | `show interfaces trunk` | Boundary link is trunking |
| Confirm VLAN 1 carried on boundary | MST-SW1/PVST-SW1 | `show interfaces trunk` | VLAN 1 is allowed and active on the boundary |
| Confirm VLAN 10 carried on boundary | MST-SW1/PVST-SW1 | `show interfaces trunk` | VLAN 10 is allowed and active on the boundary |
| Confirm VLAN 20 carried on boundary | MST-SW1/PVST-SW1 | `show interfaces trunk` | VLAN 20 is allowed and active on the boundary |
| Confirm PVST VLAN 1 root | PVST-SW1 | `show spanning-tree vlan 1` | Root ID matches the intended MST or PVST side design |
| Confirm PVST VLAN 10 root | PVST-SW1 | `show spanning-tree vlan 10` | Root ID matches the intended MST or PVST side design |
| Confirm PVST VLAN 20 root | PVST-SW1 | `show spanning-tree vlan 20` | Root ID matches the intended MST or PVST side design |
| Confirm root placement is consistent across all PVST VLANs | PVST-SW1 | `show spanning-tree vlan 1,10,20` | MST is root for all carried VLANs or for none of them |
| Confirm no inconsistent ports | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree inconsistentports` | No root-inconsistent or other inconsistent ports are listed |
| Confirm no PVST simulation failure logs | MST-SW1/PVST-SW1 | `show logging | include PVST|SIM|SPANTREE|inconsistent|ROOT` | No active PVST simulation failure is logged |
| Confirm no MAC flapping | MST-SW1/MST-SW2/PVST-SW1 | `show logging | include MACFLAP|LOOP` | No loop or MAC flap symptoms appear |
| Confirm VLAN 10 MAC learning | MST-SW1/PVST-SW1 | `show mac address-table dynamic vlan 10` | MAC addresses are stable on expected ports |
| Confirm VLAN 20 MAC learning | MST-SW1/PVST-SW1 | `show mac address-table dynamic vlan 20` | MAC addresses are stable on expected ports |
| Confirm VLAN 10 traffic | HOSTS | `ping <remote-vlan-10-host-ip>` | Same-VLAN traffic works if VLAN 10 spans the boundary |
| Confirm VLAN 20 traffic | HOSTS | `ping <remote-vlan-20-host-ip>` | Same-VLAN traffic works if VLAN 20 spans the boundary |
| Confirm MST running config | MST-SW1/MST-SW2 | `show running-config | section spanning-tree mst configuration` | MST region configuration is present |
| Confirm PVST running config | PVST-SW1 | `show running-config | include spanning-tree mode|spanning-tree vlan` | PVST mode and VLAN priorities are present |
| Confirm saved config | MST-SW1/MST-SW2/PVST-SW1 | `show startup-config | include spanning-tree` | Startup config contains intended STP settings |

# MST_PVST_RSTP_Interoperability_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current STP state before rollback | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree summary` | Current STP modes are documented |
| 2 | Record current MST region configuration | MST-SW1/MST-SW2 | `show spanning-tree mst configuration` | Region name, revision, and mapping are documented |
| 3 | Record current MST boundary state | MST-SW1 | `show spanning-tree mst interface GigabitEthernet0/1` | Boundary behavior is documented |
| 4 | Record PVST root state | PVST-SW1 | `show spanning-tree vlan 1,10,20` | Per-VLAN root state is documented |
| 5 | Record inconsistent ports before rollback | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree inconsistentports` | Any active inconsistent state is documented |
| 6 | If rollback target is all Rapid PVST, enter config mode on MST-SW1 | MST-SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 7 | Change MST-SW1 to Rapid PVST mode | MST-SW1 | `spanning-tree mode rapid-pvst` | MST-SW1 leaves MST mode |
| 8 | Reset VLAN priorities on MST-SW1 if needed | MST-SW1 | `spanning-tree vlan 1,10,20 priority 32768` | VLAN priorities return to default baseline |
| 9 | Exit MST-SW1 configuration mode | MST-SW1 | `end` | Prompt returns to privileged EXEC mode |
| 10 | Change MST-SW2 to Rapid PVST mode if removing MST region | MST-SW2 | `configure terminal` then `spanning-tree mode rapid-pvst` then `end` | MST-SW2 leaves MST mode |
| 11 | Reset PVST switch priority if test priorities were changed | PVST-SW1 | `configure terminal` then `spanning-tree vlan 1,10,20 priority 32768` then `end` | PVST switch returns to default priority baseline |
| 12 | If rollback target is all MST, enter MST config on PVST-SW1 | PVST-SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 13 | Change PVST-SW1 to MST mode | PVST-SW1 | `spanning-tree mode mst` | PVST-SW1 joins MST operation |
| 14 | Configure matching MST region on PVST-SW1 | PVST-SW1 | `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 15 | Set matching MST region name | PVST-SW1 | `name CAMPUS_MST` | Name matches MST region |
| 16 | Set matching MST revision | PVST-SW1 | `revision 10` | Revision matches MST region |
| 17 | Set matching VLAN mapping | PVST-SW1 | `instance 1 vlan 10,20` | VLAN mapping matches MST region |
| 18 | Exit MST configuration mode | PVST-SW1 | `exit` | MST region config is applied |
| 19 | Exit PVST-SW1 configuration mode | PVST-SW1 | `end` | Prompt returns to privileged EXEC mode |
| 20 | Verify rollback STP mode | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree summary` | STP mode matches rollback target |
| 21 | Verify boundary status after rollback | MST-SW1/PVST-SW1 | `show spanning-tree mst interface GigabitEthernet0/1` | Boundary is gone if all switches are same MST region |
| 22 | Verify PVST state after rollback if still PVST | PVST-SW1 | `show spanning-tree vlan 1,10,20` | Per-VLAN STP state is stable |
| 23 | Verify no inconsistent ports remain | MST-SW1/MST-SW2/PVST-SW1 | `show spanning-tree inconsistentports` | No ports are listed |
| 24 | Verify trunk state after rollback | MST-SW1/PVST-SW1 | `show interfaces trunk` | Boundary or former boundary trunk is still operational if needed |
| 25 | Verify host reachability | HOSTS | `ping <remote-same-vlan-host-ip>` | Traffic works if VLANs remain in service |
| 26 | Save rollback | MST-SW1/MST-SW2/PVST-SW1 | `write memory` | Startup configuration reflects rollback target |

# MST_PVST_RSTP_Interoperability_Failure_Checks

| Symptom                                                      | Likely Cause                                                                     | Device           | Command                                                           | Fix                                                                              |                                                  |                                      |                                                                    |       |                                                 |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------- | ---------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------ | ------------------------------------------------------------------ | ----- | ----------------------------------------------- |
| Boundary port goes root-inconsistent                         | PVST simulation check failed because root placement is inconsistent across VLANs | MST-SW1          | `show spanning-tree inconsistentports`                            | Make MST root for all PVST VLANs or root for none of them                        |                                                  |                                      |                                                                    |       |                                                 |
| MST is root for VLAN 10 but not VLAN 20                      | Unsafe mixed-root design on the same boundary                                    | PVST-SW1         | `show spanning-tree vlan 10` and `show spanning-tree vlan 20`     | Align root placement consistently for all carried VLANs                          |                                                  |                                      |                                                                    |       |                                                 |
| PVST simulation failure appears in logs                      | MST received better PVST BPDU for only some VLANs                                | MST-SW1          | `show logging                                                     | include PVST                                                                     | SIM                                              | SPANTREE                             | inconsistent                                                       | ROOT` | Correct PVST bridge priorities or MST0 priority |
| MST boundary not detected                                    | Link is not trunking or neighbor is not sending PVST BPDUs                       | MST-SW1          | `show spanning-tree mst interface GigabitEthernet0/1`             | Fix trunking and verify PVST neighbor STP mode                                   |                                                  |                                      |                                                                    |       |                                                 |
| Boundary expected but port shows internal                    | Neighbor is actually in the same MST region                                      | MST-SW1          | `show spanning-tree mst configuration`                            | Change region identity if a boundary is intended                                 |                                                  |                                      |                                                                    |       |                                                 |
| Internal expected but port shows boundary                    | MST name, revision, or mapping mismatch                                          | MST switches     | `show spanning-tree mst configuration`                            | Make MST region identity identical on both sides                                 |                                                  |                                      |                                                                    |       |                                                 |
| VLAN 1 missing from boundary trunk                           | PVST simulation cannot rely on normal VLAN 1 BPDU handling                       | MST-SW1/PVST-SW1 | `show interfaces trunk`                                           | Allow VLAN 1 unless the design has been specifically validated without it        |                                                  |                                      |                                                                    |       |                                                 |
| VLAN 10 does not cross boundary                              | VLAN 10 missing from trunk or local VLAN database                                | MST-SW1/PVST-SW1 | `show interfaces trunk` and `show vlan brief`                     | Create VLAN 10 and allow it on trunk                                             |                                                  |                                      |                                                                    |       |                                                 |
| VLAN 20 does not cross boundary                              | VLAN 20 missing from trunk or local VLAN database                                | MST-SW1/PVST-SW1 | `show interfaces trunk` and `show vlan brief`                     | Create VLAN 20 and allow it on trunk                                             |                                                  |                                      |                                                                    |       |                                                 |
| PVST side chooses itself as root when MST should be root     | PVST bridge priority is lower than MST CIST priority                             | PVST-SW1         | `show spanning-tree vlan 1,10,20`                                 | Raise PVST priority or lower MST0 priority                                       |                                                  |                                      |                                                                    |       |                                                 |
| MST side becomes root when PVST should be root               | MST0 priority is too low                                                         | MST-SW1          | `show spanning-tree mst 0`                                        | Raise MST0 priority or lower PVST priorities consistently                        |                                                  |                                      |                                                                    |       |                                                 |
| MSTI root does not match expected traffic path               | MST1 is internal and does not extend into PVST side                              | MST-SW1          | `show spanning-tree mst 1`                                        | Tune MST internally, then separately verify boundary CIST behavior               |                                                  |                                      |                                                                    |       |                                                 |
| PVST per-VLAN load sharing fails at boundary                 | MST boundary cannot represent independent PVST topologies per VLAN               | PVST-SW1         | `show spanning-tree vlan 10` and `show spanning-tree vlan 20`     | Keep root placement consistent across boundary VLANs                             |                                                  |                                      |                                                                    |       |                                                 |
| Boundary port blocks all VLANs unexpectedly                  | CIST boundary behavior is blocking the port                                      | MST-SW1/PVST-SW1 | `show spanning-tree mst 0` and `show spanning-tree vlan 1`        | Correct CIST root placement and trunk topology                                   |                                                  |                                      |                                                                    |       |                                                 |
| Inconsistent state does not clear                            | Superior or inconsistent PVST BPDUs continue arriving                            | MST-SW1          | `show spanning-tree inconsistentports`                            | Fix PVST priorities and verify all carried VLANs have consistent root design     |                                                  |                                      |                                                                    |       |                                                 |
| MAC addresses flap                                           | Layer 2 loop or unstable STP boundary                                            | MST-SW1/PVST-SW1 | `show logging                                                     | include MACFLAP                                                                  | LOOP                                             | SPANTREE`                            | Keep boundary blocked until STP root and trunk state are corrected |       |                                                 |
| Host traffic fails after inconsistency clears                | VLAN, trunk, or MAC relearning issue remains                                     | MST-SW1/PVST-SW1 | `show interfaces trunk`, `show mac address-table dynamic vlan 10` | Fix VLAN allowance and generate traffic to relearn MACs                          |                                                  |                                      |                                                                    |       |                                                 |
| Rapid PVST switch behaves like PVST at boundary              | Expected behavior                                                                | PVST-SW1         | `show spanning-tree summary`                                      | Treat Rapid PVST as PVST style interoperability at the MST boundary              |                                                  |                                      |                                                                    |       |                                                 |
| MST configuration looks right but boundary behavior is wrong | MST0/CIST root placement was ignored                                             | MST-SW1          | `show spanning-tree mst 0`                                        | Fix MST0 priority first                                                          |                                                  |                                      |                                                                    |       |                                                 |
| Revision number mismatch creates boundary                    | Revision is part of region identity                                              | MST switches     | `show spanning-tree mst configuration`                            | Match revision if same MST region is intended                                    |                                                  |                                      |                                                                    |       |                                                 |
| VLAN mapping mismatch creates boundary                       | VLAN to instance mapping differs                                                 | MST switches     | `show spanning-tree mst configuration`                            | Match mappings if same MST region is intended                                    |                                                  |                                      |                                                                    |       |                                                 |
| Trunk allowed list overwrote production VLANs                | Used replacement syntax instead of add/remove                                    | MST-SW1/PVST-SW1 | `show running-config interface GigabitEthernet0/1`                | Restore full allowed list or use `switchport trunk allowed vlan add <vlan-list>` |                                                  |                                      |                                                                    |       |                                                 |
| Native VLAN mismatch appears                                 | Boundary trunk native VLAN differs                                               | MST-SW1/PVST-SW1 | `show logging                                                     | include NATIVE                                                                   | native`                                          | Match native VLAN on both trunk ends |                                                                    |       |                                                 |
| Config disappears after reload                               | Configuration was not saved                                                      | MST-SW1/PVST-SW1 | `show startup-config                                              | include spanning-tree`                                                           | Reapply intended settings and run `write memory` |                                      |                                                                    |       |                                                 |