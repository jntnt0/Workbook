MST_Region_IST_MSTI_Core.md
# MST_Region_IST_MSTI_Core

# MST_Region_IST_MSTI_Core_Index

- MST_Region_IST_MSTI_Core_Mental_Model
- MST_Region_IST_MSTI_Core_Configuration_Checklist
- MST_Region_IST_MSTI_Core_Skeleton
- MST_Region_IST_MSTI_Core_Verification_Commands
- MST_Region_IST_MSTI_Core_Rollback
- MST_Region_IST_MSTI_Core_Failure_Checks

# MST_Region_IST_MSTI_Core_Mental_Model

| Concept | Operational Meaning |
|---|---|
| MST | Multiple Spanning Tree maps many VLANs into fewer STP instances |
| MST region | A group of switches with the same MST region name, revision number, and VLAN-to-instance mapping |
| Region identity | MST region membership depends on exact match of name, revision, and VLAN mapping |
| Revision number | The revision number is just part of the region identifier; it does not automatically mean newer or better |
| MST0 | Instance 0 is always present and acts as the IST inside the MST region |
| IST | Internal Spanning Tree connects all MST switches inside the region |
| CIST | Common and Internal Spanning Tree represents the MST region toward the outside STP world |
| MSTI | MST instance used to carry one or more VLANs with its own logical STP topology |
| VLAN mapping | VLANs are assigned to MST instances under MST configuration mode |
| Default VLAN mapping | VLANs not explicitly mapped to another MST instance remain mapped to MST0 |
| BPDU efficiency | MST carries instance information inside MST BPDUs instead of sending one BPDU per VLAN like PVST |
| Internal boundary | Ports between switches with matching MST config are internal region ports |
| Boundary port | A port facing a different MST region, PVST, RSTP, or classic STP domain |
| Root per instance | MST0, MST1, MST2, and other instances can have separate root bridges |
| Instance priority | `spanning-tree mst <instance> root primary` or explicit priority controls root placement per MST instance |
| Trunk dependency | MST calculations only matter for VLANs that exist and are carried on the trunk path |
| Consistency first | Before troubleshooting blocked ports, verify the MST region config matches on all switches |
| Operational rule | Configure identical region name, revision, and mappings everywhere before tuning cost or root placement |

# MST_Region_IST_MSTI_Core_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm physical switch adjacency | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended Layer 2 topology |
| 2 | Confirm interswitch links are physically up | SW1/SW2/SW3 | `show interfaces status` | Interswitch interfaces show `connected` |
| 3 | Confirm trunk links are operating before MST region testing | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links show `trunking` |
| 4 | Confirm current STP mode before migration | SW1/SW2/SW3 | `show spanning-tree summary` | Existing STP mode is known |
| 5 | Confirm existing VLAN database | SW1/SW2/SW3 | `show vlan brief` | Existing VLANs are documented before MST changes |
| 6 | Confirm current STP state before migration | SW1/SW2/SW3 | `show spanning-tree` | Existing root bridges, port roles, and blocked ports are known |
| 7 | Enter global configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 8 | Create VLAN 10 | SW1 | `vlan 10` | VLAN 10 exists locally |
| 9 | Name VLAN 10 | SW1 | `name USERS` | VLAN 10 has a readable name |
| 10 | Create VLAN 20 | SW1 | `vlan 20` | VLAN 20 exists locally |
| 11 | Name VLAN 20 | SW1 | `name VOICE` | VLAN 20 has a readable name |
| 12 | Create VLAN 30 | SW1 | `vlan 30` | VLAN 30 exists locally |
| 13 | Name VLAN 30 | SW1 | `name SERVERS` | VLAN 30 has a readable name |
| 14 | Create VLAN 40 | SW1 | `vlan 40` | VLAN 40 exists locally |
| 15 | Name VLAN 40 | SW1 | `name GUEST` | VLAN 40 has a readable name |
| 16 | Enable MST mode | SW1 | `spanning-tree mode mst` | SW1 runs MST |
| 17 | Enter MST configuration submode | SW1 | `spanning-tree mst configuration` | Prompt changes to `config-mst` mode |
| 18 | Set the MST region name | SW1 | `name CAMPUS_MST` | MST region name is configured |
| 19 | Set the MST revision number | SW1 | `revision 10` | MST revision is configured |
| 20 | Map VLANs 10 and 20 to MST instance 1 | SW1 | `instance 1 vlan 10,20` | VLANs 10 and 20 map to MST1 |
| 21 | Map VLANs 30 and 40 to MST instance 2 | SW1 | `instance 2 vlan 30,40` | VLANs 30 and 40 map to MST2 |
| 22 | Exit MST configuration submode | SW1 | `exit` | MST configuration is applied locally |
| 23 | Configure SW1 as root for MST0 | SW1 | `spanning-tree mst 0 root primary` | SW1 becomes preferred IST/CIST root inside the region |
| 24 | Configure SW1 as root for MST1 | SW1 | `spanning-tree mst 1 root primary` | SW1 becomes preferred root for MST1 |
| 25 | Configure SW1 as secondary root for MST2 | SW1 | `spanning-tree mst 2 root secondary` | SW1 becomes backup root for MST2 |
| 26 | Configure trunk toward SW2 | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 27 | Force trunking toward SW2 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 28 | Allow MST VLANs on trunk toward SW2 | SW1 | `switchport trunk allowed vlan 10,20,30,40` | Required VLANs are allowed on the trunk |
| 29 | Bring up trunk toward SW2 | SW1 | `no shutdown` | Interface is administratively up |
| 30 | Configure trunk toward SW3 | SW1 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 31 | Force trunking toward SW3 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 32 | Allow MST VLANs on trunk toward SW3 | SW1 | `switchport trunk allowed vlan 10,20,30,40` | Required VLANs are allowed on the trunk |
| 33 | Bring up trunk toward SW3 | SW1 | `no shutdown` | Interface is administratively up |
| 34 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 35 | Enter global configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 36 | Create VLAN 10 | SW2 | `vlan 10` | VLAN 10 exists locally |
| 37 | Create VLAN 20 | SW2 | `vlan 20` | VLAN 20 exists locally |
| 38 | Create VLAN 30 | SW2 | `vlan 30` | VLAN 30 exists locally |
| 39 | Create VLAN 40 | SW2 | `vlan 40` | VLAN 40 exists locally |
| 40 | Enable MST mode | SW2 | `spanning-tree mode mst` | SW2 runs MST |
| 41 | Enter MST configuration submode | SW2 | `spanning-tree mst configuration` | Prompt changes to `config-mst` mode |
| 42 | Set the same MST region name | SW2 | `name CAMPUS_MST` | Region name matches SW1 |
| 43 | Set the same MST revision number | SW2 | `revision 10` | Revision matches SW1 |
| 44 | Map VLANs 10 and 20 to MST instance 1 | SW2 | `instance 1 vlan 10,20` | VLAN-to-instance mapping matches SW1 |
| 45 | Map VLANs 30 and 40 to MST instance 2 | SW2 | `instance 2 vlan 30,40` | VLAN-to-instance mapping matches SW1 |
| 46 | Exit MST configuration submode | SW2 | `exit` | MST configuration is applied locally |
| 47 | Configure SW2 as secondary root for MST0 | SW2 | `spanning-tree mst 0 root secondary` | SW2 becomes backup root for MST0 |
| 48 | Configure SW2 as secondary root for MST1 | SW2 | `spanning-tree mst 1 root secondary` | SW2 becomes backup root for MST1 |
| 49 | Configure SW2 as primary root for MST2 | SW2 | `spanning-tree mst 2 root primary` | SW2 becomes preferred root for MST2 |
| 50 | Configure trunk toward SW1 | SW2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 51 | Force trunking toward SW1 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 52 | Allow MST VLANs on trunk toward SW1 | SW2 | `switchport trunk allowed vlan 10,20,30,40` | Required VLANs are allowed on the trunk |
| 53 | Bring up trunk toward SW1 | SW2 | `no shutdown` | Interface is administratively up |
| 54 | Configure trunk toward SW3 | SW2 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 55 | Force trunking toward SW3 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 56 | Allow MST VLANs on trunk toward SW3 | SW2 | `switchport trunk allowed vlan 10,20,30,40` | Required VLANs are allowed on the trunk |
| 57 | Bring up trunk toward SW3 | SW2 | `no shutdown` | Interface is administratively up |
| 58 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 59 | Enter global configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 60 | Create VLAN 10 | SW3 | `vlan 10` | VLAN 10 exists locally |
| 61 | Create VLAN 20 | SW3 | `vlan 20` | VLAN 20 exists locally |
| 62 | Create VLAN 30 | SW3 | `vlan 30` | VLAN 30 exists locally |
| 63 | Create VLAN 40 | SW3 | `vlan 40` | VLAN 40 exists locally |
| 64 | Enable MST mode | SW3 | `spanning-tree mode mst` | SW3 runs MST |
| 65 | Enter MST configuration submode | SW3 | `spanning-tree mst configuration` | Prompt changes to `config-mst` mode |
| 66 | Set the same MST region name | SW3 | `name CAMPUS_MST` | Region name matches SW1 and SW2 |
| 67 | Set the same MST revision number | SW3 | `revision 10` | Revision matches SW1 and SW2 |
| 68 | Map VLANs 10 and 20 to MST instance 1 | SW3 | `instance 1 vlan 10,20` | VLAN-to-instance mapping matches the region |
| 69 | Map VLANs 30 and 40 to MST instance 2 | SW3 | `instance 2 vlan 30,40` | VLAN-to-instance mapping matches the region |
| 70 | Exit MST configuration submode | SW3 | `exit` | MST configuration is applied locally |
| 71 | Configure trunk toward SW1 | SW3 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 72 | Force trunking toward SW1 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 73 | Allow MST VLANs on trunk toward SW1 | SW3 | `switchport trunk allowed vlan 10,20,30,40` | Required VLANs are allowed on the trunk |
| 74 | Bring up trunk toward SW1 | SW3 | `no shutdown` | Interface is administratively up |
| 75 | Configure trunk toward SW2 | SW3 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 76 | Force trunking toward SW2 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 77 | Allow MST VLANs on trunk toward SW2 | SW3 | `switchport trunk allowed vlan 10,20,30,40` | Required VLANs are allowed on the trunk |
| 78 | Bring up trunk toward SW2 | SW3 | `no shutdown` | Interface is administratively up |
| 79 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 80 | Verify MST mode on every switch | SW1/SW2/SW3 | `show spanning-tree summary` | Switches show MST mode |
| 81 | Verify MST region configuration on every switch | SW1/SW2/SW3 | `show spanning-tree mst configuration` | Name, revision, and VLAN mappings match |
| 82 | Verify MST0, MST1, and MST2 topology | SW1/SW2/SW3 | `show spanning-tree mst` | MST instances, mapped VLANs, roles, and states are visible |
| 83 | Verify SW1 is root for MST0 | SW1 | `show spanning-tree mst 0` | SW1 shows itself as root for MST0 |
| 84 | Verify SW1 is root for MST1 | SW1 | `show spanning-tree mst 1` | SW1 shows itself as root for MST1 |
| 85 | Verify SW2 is root for MST2 | SW2 | `show spanning-tree mst 2` | SW2 shows itself as root for MST2 |
| 86 | Verify internal MST region ports | SW1/SW2/SW3 | `show spanning-tree mst interface GigabitEthernet0/1` | Boundary field shows internal when the neighbor matches the region |
| 87 | Verify trunks carry the mapped VLANs | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10, 20, 30, and 40 are allowed and active |
| 88 | Verify no inconsistent STP ports exist | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 89 | Verify host reachability per mapped VLAN | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works across the MST region |
| 90 | Save configuration | SW1/SW2/SW3 | `write memory` | MST region configuration survives reload |

# MST_Region_IST_MSTI_Core_Skeleton

SW1, MST root for MST0 and MST1, backup root for MST2:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name VOICE
vlan 30
 name SERVERS
vlan 40
 name GUEST
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name CAMPUS_MST
 revision 10
 instance 1 vlan 10,20
 instance 2 vlan 30,40
 exit
!
spanning-tree mst 0 root primary
spanning-tree mst 1 root primary
spanning-tree mst 2 root secondary
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
!
end
write memory

SW2, backup root for MST0 and MST1, root for MST2:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name VOICE
vlan 30
 name SERVERS
vlan 40
 name GUEST
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name CAMPUS_MST
 revision 10
 instance 1 vlan 10,20
 instance 2 vlan 30,40
 exit
!
spanning-tree mst 0 root secondary
spanning-tree mst 1 root secondary
spanning-tree mst 2 root primary
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
!
end
write memory

SW3, MST region member:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name VOICE
vlan 30
 name SERVERS
vlan 40
 name GUEST
!
spanning-tree mode mst
!
spanning-tree mst configuration
 name CAMPUS_MST
 revision 10
 instance 1 vlan 10,20
 instance 2 vlan 30,40
 exit
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
!
end
write memory

Manual MST priority alternative:

SW1:

configure terminal
!
spanning-tree mst 0 priority 4096
spanning-tree mst 1 priority 4096
spanning-tree mst 2 priority 8192
!
end
write memory

SW2:

configure terminal
!
spanning-tree mst 0 priority 8192
spanning-tree mst 1 priority 8192
spanning-tree mst 2 priority 4096
!
end
write memory

Optional MST path-cost tuning per instance:

configure terminal
!
interface GigabitEthernet0/1
 description PREFERRED_PATH_FOR_MST1
 spanning-tree mst 1 cost 4
 spanning-tree mst 2 cost 19
!
interface GigabitEthernet0/2
 description PREFERRED_PATH_FOR_MST2
 spanning-tree mst 1 cost 19
 spanning-tree mst 2 cost 4
!
end
write memory

# MST_Region_IST_MSTI_Core_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | Switches show MST mode |
| Confirm VLAN database | SW1/SW2/SW3 | `show vlan brief` | VLANs 10, 20, 30, and 40 exist and are active |
| Confirm trunk state | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links show trunking |
| Confirm mapped VLANs cross trunks | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10, 20, 30, and 40 are allowed and active |
| Confirm MST region name, revision, and mappings | SW1/SW2/SW3 | `show spanning-tree mst configuration` | Name, revision, and VLAN-to-instance mappings match on all switches |
| Confirm MST topology summary | SW1/SW2/SW3 | `show spanning-tree mst` | MST0, MST1, and MST2 are visible with mapped VLANs |
| Confirm MST0 root | SW1 | `show spanning-tree mst 0` | SW1 is root for MST0 |
| Confirm MST1 root | SW1 | `show spanning-tree mst 1` | SW1 is root for MST1 |
| Confirm MST2 root | SW2 | `show spanning-tree mst 2` | SW2 is root for MST2 |
| Confirm SW3 root ports per instance | SW3 | `show spanning-tree mst` | SW3 shows root ports toward the correct MST roots |
| Confirm mapped VLANs in MST1 | SW1/SW2/SW3 | `show spanning-tree mst 1` | VLANs 10 and 20 are mapped to MST1 |
| Confirm mapped VLANs in MST2 | SW1/SW2/SW3 | `show spanning-tree mst 2` | VLANs 30 and 40 are mapped to MST2 |
| Confirm unmapped VLANs remain in MST0 | SW1/SW2/SW3 | `show spanning-tree mst 0` | VLANs not mapped to MST1 or MST2 remain in MST0 |
| Confirm interface MST details | SW1/SW2/SW3 | `show spanning-tree mst interface GigabitEthernet0/1` | Instance roles, states, costs, and mapped VLANs are visible |
| Confirm internal region boundary status | SW1/SW2/SW3 | `show spanning-tree mst interface GigabitEthernet0/1` | Boundary shows internal on matching-region links |
| Confirm BPDU counters on interswitch links | SW1/SW2/SW3 | `show spanning-tree mst interface GigabitEthernet0/1 | include Bpdus|Boundary|Role|Sts` | BPDUs are sent and received and boundary is internal |
| Confirm configured MST root commands | SW1/SW2 | `show running-config | include spanning-tree mst` | Root or priority commands are present |
| Confirm MST configuration in running config | SW1/SW2/SW3 | `show running-config | section spanning-tree mst configuration` | MST name, revision, and instance mappings are present |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include SPANTREE|MACFLAP|LOOP|MST` | No loop, MAC flap, or MST mismatch symptoms appear |
| Confirm MAC learning for MST1 VLAN | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | VLAN 10 MAC addresses are stable on expected ports |
| Confirm MAC learning for MST2 VLAN | SW1/SW2/SW3 | `show mac address-table dynamic vlan 30` | VLAN 30 MAC addresses are stable on expected ports |
| Confirm host reachability in VLAN 10 | HOSTS | `ping <remote-vlan-10-host-ip>` | Same-VLAN traffic works across MST1 topology |
| Confirm host reachability in VLAN 30 | HOSTS | `ping <remote-vlan-30-host-ip>` | Same-VLAN traffic works across MST2 topology |
| Confirm saved MST configuration | SW1/SW2/SW3 | `show startup-config | section spanning-tree mst configuration` | Startup config contains intended MST region settings |

# MST_Region_IST_MSTI_Core_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current MST region state before rollback | SW1/SW2/SW3 | `show spanning-tree mst configuration` | Name, revision, and mappings are documented |
| 2 | Record current MST topology before rollback | SW1/SW2/SW3 | `show spanning-tree mst` | Root, roles, states, and mapped VLANs are documented |
| 3 | Record trunk state before rollback | SW1/SW2/SW3 | `show interfaces trunk` | Current trunk and VLAN forwarding state is documented |
| 4 | Enter global configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 5 | Remove MST1 root or priority control on SW1 | SW1 | `no spanning-tree mst 1 priority` | MST1 priority returns to default behavior if supported |
| 6 | Remove MST2 root or priority control on SW1 | SW1 | `no spanning-tree mst 2 priority` | MST2 priority returns to default behavior if supported |
| 7 | Reset MST priorities manually if no-form is not supported | SW1 | `spanning-tree mst 0 priority 32768` | MST0 base priority returns to default |
| 8 | Reset MST1 priority manually | SW1 | `spanning-tree mst 1 priority 32768` | MST1 base priority returns to default |
| 9 | Reset MST2 priority manually | SW1 | `spanning-tree mst 2 priority 32768` | MST2 base priority returns to default |
| 10 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 11 | Enter global configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 12 | Reset MST0 priority manually | SW2 | `spanning-tree mst 0 priority 32768` | MST0 base priority returns to default |
| 13 | Reset MST1 priority manually | SW2 | `spanning-tree mst 1 priority 32768` | MST1 base priority returns to default |
| 14 | Reset MST2 priority manually | SW2 | `spanning-tree mst 2 priority 32768` | MST2 base priority returns to default |
| 15 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 16 | Remove MST region mappings if fully decommissioning MST lab | SW1/SW2/SW3 | `configure terminal` then `spanning-tree mst configuration` | Prompt changes to MST configuration mode |
| 17 | Remove MST1 VLAN mapping | SW1/SW2/SW3 | `no instance 1 vlan 10,20` | VLANs 10 and 20 return to MST0 mapping |
| 18 | Remove MST2 VLAN mapping | SW1/SW2/SW3 | `no instance 2 vlan 30,40` | VLANs 30 and 40 return to MST0 mapping |
| 19 | Exit MST configuration mode | SW1/SW2/SW3 | `exit` | MST mapping changes are applied |
| 20 | Optionally return the switch to Rapid PVST+ | SW1/SW2/SW3 | `spanning-tree mode rapid-pvst` | Switch exits MST mode and runs Rapid PVST+ |
| 21 | Exit configuration mode | SW1/SW2/SW3 | `end` | Prompt returns to privileged EXEC mode |
| 22 | Remove test VLANs if the lab is being fully cleared | SW1/SW2/SW3 | `configure terminal` then `no vlan 10,20,30,40` then `end` | Test VLANs are removed locally if supported by platform syntax |
| 23 | If comma removal is unsupported, remove VLANs individually | SW1/SW2/SW3 | `configure terminal` then `no vlan 10` then `no vlan 20` then `no vlan 30` then `no vlan 40` then `end` | Test VLANs are removed locally |
| 24 | Verify rollback STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode matches rollback target |
| 25 | Verify MST mappings after rollback | SW1/SW2/SW3 | `show spanning-tree mst configuration` | MST mappings are removed or changed as intended |
| 26 | Verify no inconsistent ports remain | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 27 | Verify trunk state after rollback | SW1/SW2/SW3 | `show interfaces trunk` | Remaining trunks operate as intended |
| 28 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |

# MST_Region_IST_MSTI_Core_Failure_Checks

| Symptom                                                   | Likely Cause                                                | Device      | Command                                                                | Fix                                                                                  |                                           |          |      |                                                |
| --------------------------------------------------------- | ----------------------------------------------------------- | ----------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------- | -------- | ---- | ---------------------------------------------- |
| Switches are not in the same MST region                   | Region name, revision, or VLAN mapping mismatch             | SW1/SW2/SW3 | `show spanning-tree mst configuration`                                 | Make name, revision, and mappings identical on all region members                    |                                           |          |      |                                                |
| Port shows boundary instead of internal                   | Neighbor is outside the region or MST config does not match | SW1/SW2     | `show spanning-tree mst interface GigabitEthernet0/1`                  | Correct MST region config on both ends                                               |                                           |          |      |                                                |
| VLANs map to MST0 unexpectedly                            | VLANs were not mapped to an MSTI                            | SW1/SW2/SW3 | `show spanning-tree mst configuration`                                 | Add `instance <id> vlan <vlan-list>` under MST config mode                           |                                           |          |      |                                                |
| VLAN mapping matches but traffic fails                    | VLAN does not exist locally                                 | SW1/SW2/SW3 | `show vlan brief`                                                      | Create the VLAN locally or propagate it with the intended VLAN database method       |                                           |          |      |                                                |
| VLAN mapping matches but traffic still fails              | VLAN is not allowed on trunk                                | SW1/SW2/SW3 | `show interfaces trunk`                                                | Add the VLAN to the trunk allowed list                                               |                                           |          |      |                                                |
| MST root is wrong for MST1                                | Instance priority or root macro is missing or wrong         | SW1/SW2/SW3 | `show spanning-tree mst 1`                                             | Configure intended root with `spanning-tree mst 1 root primary` or explicit priority |                                           |          |      |                                                |
| MST root is wrong for MST2                                | Instance priority or root macro is missing or wrong         | SW1/SW2/SW3 | `show spanning-tree mst 2`                                             | Configure intended root with `spanning-tree mst 2 root primary` or explicit priority |                                           |          |      |                                                |
| MST0 root is wrong                                        | IST/CIST root placement was not set                         | SW1/SW2/SW3 | `show spanning-tree mst 0`                                             | Configure intended MST0 root and secondary root                                      |                                           |          |      |                                                |
| VLAN 10 and VLAN 20 share the same topology               | Both VLANs are mapped to the same MSTI                      | SW1/SW2/SW3 | `show spanning-tree mst configuration`                                 | This is expected for VLANs in the same MST instance                                  |                                           |          |      |                                                |
| VLAN 10 and VLAN 30 should use different paths but do not | MST roots or costs are not different between MST1 and MST2  | SW1/SW2/SW3 | `show spanning-tree mst`                                               | Set different roots or tune instance-specific costs                                  |                                           |          |      |                                                |
| Cost tuning affects wrong traffic                         | VLAN is mapped to a different MST instance than assumed     | SW1/SW2/SW3 | `show spanning-tree mst configuration`                                 | Tune `spanning-tree mst <correct-instance> cost <cost>`                              |                                           |          |      |                                                |
| Interface cost tuning has no effect                       | Root placement or another lower-cost path still wins        | SW3         | `show spanning-tree mst <instance>`                                    | Verify root bridge first, then tune MST instance cost                                |                                           |          |      |                                                |
| Instance priority is rejected                             | Priority value is not a multiple of 4096                    | SW1/SW2     | `spanning-tree mst <instance> priority <value>`                        | Use 0, 4096, 8192, up to 61440                                                       |                                           |          |      |                                                |
| Trunk shows VLAN allowed but not forwarding               | STP is blocking that instance or VLAN is not active         | SW1/SW2/SW3 | `show interfaces trunk` and `show spanning-tree mst`                   | Verify MST instance state, trunk allowance, and VLAN existence                       |                                           |          |      |                                                |
| Host traffic fails in VLAN 10 only                        | MST1 path, VLAN 10 access ports, or trunk allowance issue   | SW1/SW2/SW3 | `show vlan brief`, `show interfaces trunk`, `show spanning-tree mst 1` | Fix access VLAN, trunk, or MST1 state                                                |                                           |          |      |                                                |
| Host traffic fails in VLAN 30 only                        | MST2 path, VLAN 30 access ports, or trunk allowance issue   | SW1/SW2/SW3 | `show vlan brief`, `show interfaces trunk`, `show spanning-tree mst 2` | Fix access VLAN, trunk, or MST2 state                                                |                                           |          |      |                                                |
| MAC addresses flap                                        | Layer 2 loop or mismatched region behavior                  | SW1/SW2/SW3 | `show logging                                                          | include MACFLAP                                                                      | LOOP                                      | SPANTREE | MST` | Verify region match, trunks, and blocked ports |
| Inconsistent ports appear                                 | STP protection, region mismatch, or BPDU issue              | SW1/SW2/SW3 | `show spanning-tree inconsistentports`                                 | Inspect guard features, BPDU flow, and MST boundary state                            |                                           |          |      |                                                |
| Region mismatch after a harmless-looking edit             | Revision number changed on only one switch                  | SW1/SW2/SW3 | `show spanning-tree mst configuration`                                 | Match revision across all switches in the region                                     |                                           |          |      |                                                |
| Region mismatch after VLAN remapping                      | Instance mapping changed on only one switch                 | SW1/SW2/SW3 | `show spanning-tree mst configuration`                                 | Apply identical mappings across all region switches                                  |                                           |          |      |                                                |
| MST mode not active                                       | Switch still runs PVST or Rapid PVST                        | SW1/SW2/SW3 | `show spanning-tree summary`                                           | Configure `spanning-tree mode mst`                                                   |                                           |          |      |                                                |
| Config missing after reload                               | MST configuration was not saved                             | SW1/SW2/SW3 | `show startup-config                                                   | section spanning-tree mst configuration`                                             | Reapply MST config and run `write memory` |          |      |                                                |