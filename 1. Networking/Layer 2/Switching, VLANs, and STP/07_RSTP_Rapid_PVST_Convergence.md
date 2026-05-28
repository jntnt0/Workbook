RSTP_Rapid_PVST_Convergence.md
# RSTP_Rapid_PVST_Convergence

# RSTP_Rapid_PVST_Convergence_Index

- RSTP_Rapid_PVST_Convergence_Mental_Model
- RSTP_Rapid_PVST_Convergence_Configuration_Checklist
- RSTP_Rapid_PVST_Convergence_Skeleton
- RSTP_Rapid_PVST_Convergence_Verification_Commands
- RSTP_Rapid_PVST_Convergence_Rollback
- RSTP_Rapid_PVST_Convergence_Failure_Checks

# RSTP_Rapid_PVST_Convergence_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Rapid PVST+ | Cisco per-VLAN implementation of RSTP, giving each VLAN its own rapid spanning-tree instance |
| RSTP purpose | RSTP keeps Layer 2 loop prevention but improves convergence compared to classic 802.1D STP |
| Per-VLAN convergence | Each VLAN converges independently, so VLAN 10 and VLAN 20 can have different roots and port roles |
| Root bridge | The root bridge is still the reference point for the spanning-tree calculation |
| Root port | Each non-root switch selects one best port toward the root bridge per VLAN |
| Designated port | Each segment has one forwarding designated port per VLAN |
| Alternate port | A backup path toward the root bridge that can rapidly move to forwarding if the active root path fails |
| Backup port | A redundant port toward the same shared segment, rarely seen in modern switched full-duplex designs |
| Discarding state | RSTP combines disabled, blocking, and listening behavior into discarding |
| Learning state | The port learns MAC addresses but does not forward user traffic yet |
| Forwarding state | The port forwards user traffic and learns MAC addresses |
| Edge port | A host-facing PortFast port that can move directly to forwarding because it should not create a loop |
| Non-edge port | A port that receives BPDUs and participates in normal RSTP synchronization |
| Point-to-point link | A full-duplex switch-to-switch link that supports rapid proposal/agreement convergence |
| Proposal/agreement | RSTP handshake used between switches to move safe links to forwarding quickly |
| 802.1D fallback | If a neighbor does not participate in RSTP, the port can fall back to slower 802.1D behavior |
| PortFast role | PortFast makes access ports edge ports, reducing host startup delay and suppressing unnecessary topology churn |
| BPDU Guard role | BPDU Guard protects PortFast access ports by err-disabling the port if a BPDU is received |
| Convergence verification | Do not assume rapid convergence; verify port roles, port states, topology changes, and host reachability |
| Root placement first | Rapid convergence does not fix bad root placement; root bridge design still comes first |

# RSTP_Rapid_PVST_Convergence_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm physical switch adjacency before changing STP mode | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended topology |
| 2 | Confirm interswitch interfaces are up | SW1/SW2/SW3 | `show interfaces status` | Switch-to-switch interfaces show `connected` |
| 3 | Confirm trunk links exist before enabling per-VLAN convergence | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links show `trunking` |
| 4 | Confirm current spanning-tree mode | SW1/SW2/SW3 | `show spanning-tree summary` | Current STP mode is documented before migration |
| 5 | Confirm target VLANs exist | SW1/SW2/SW3 | `show vlan brief` | VLANs 10 and 20 exist and are active |
| 6 | Record current STP root state before migration | SW1/SW2/SW3 | `show spanning-tree root` | Current root bridge state is documented |
| 7 | Enter global configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 8 | Enable Rapid PVST+ mode | SW1 | `spanning-tree mode rapid-pvst` | SW1 uses Rapid PVST+ |
| 9 | Create VLAN 10 if missing | SW1 | `vlan 10` | VLAN 10 exists locally |
| 10 | Name VLAN 10 | SW1 | `name USERS` | VLAN 10 has a readable name |
| 11 | Create VLAN 20 if missing | SW1 | `vlan 20` | VLAN 20 exists locally |
| 12 | Name VLAN 20 | SW1 | `name SERVERS` | VLAN 20 has a readable name |
| 13 | Make SW1 primary root for VLAN 10 | SW1 | `spanning-tree vlan 10 root primary` | SW1 becomes preferred root for VLAN 10 |
| 14 | Make SW1 secondary root for VLAN 20 | SW1 | `spanning-tree vlan 20 root secondary` | SW1 becomes backup root for VLAN 20 |
| 15 | Enable PortFast globally on access ports | SW1 | `spanning-tree portfast default` | Access ports become RSTP edge ports by default |
| 16 | Enable BPDU Guard globally for PortFast ports | SW1 | `spanning-tree portfast bpduguard default` | Edge ports are protected if BPDUs are received |
| 17 | Configure trunk toward SW2 | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 18 | Force trunk mode toward SW2 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 19 | Permit target VLANs toward SW2 | SW1 | `switchport trunk allowed vlan 10,20` | VLANs 10 and 20 are carried on the trunk |
| 20 | Ensure trunk is not treated as PortFast edge | SW1 | `spanning-tree portfast disable` | Switch-to-switch trunk remains non-edge |
| 21 | Optionally force RSTP point-to-point link type | SW1 | `spanning-tree link-type point-to-point` | RSTP can use rapid proposal/agreement on this full-duplex link |
| 22 | Bring up trunk toward SW2 | SW1 | `no shutdown` | Interface is administratively up |
| 23 | Configure trunk toward SW3 | SW1 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 24 | Force trunk mode toward SW3 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 25 | Permit target VLANs toward SW3 | SW1 | `switchport trunk allowed vlan 10,20` | VLANs 10 and 20 are carried on the trunk |
| 26 | Ensure trunk is not treated as PortFast edge | SW1 | `spanning-tree portfast disable` | Switch-to-switch trunk remains non-edge |
| 27 | Optionally force RSTP point-to-point link type | SW1 | `spanning-tree link-type point-to-point` | RSTP can use rapid proposal/agreement on this full-duplex link |
| 28 | Bring up trunk toward SW3 | SW1 | `no shutdown` | Interface is administratively up |
| 29 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 30 | Enter global configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 31 | Enable Rapid PVST+ mode | SW2 | `spanning-tree mode rapid-pvst` | SW2 uses Rapid PVST+ |
| 32 | Create VLAN 10 if missing | SW2 | `vlan 10` | VLAN 10 exists locally |
| 33 | Create VLAN 20 if missing | SW2 | `vlan 20` | VLAN 20 exists locally |
| 34 | Make SW2 secondary root for VLAN 10 | SW2 | `spanning-tree vlan 10 root secondary` | SW2 becomes backup root for VLAN 10 |
| 35 | Make SW2 primary root for VLAN 20 | SW2 | `spanning-tree vlan 20 root primary` | SW2 becomes preferred root for VLAN 20 |
| 36 | Enable PortFast globally on access ports | SW2 | `spanning-tree portfast default` | Access ports become RSTP edge ports by default |
| 37 | Enable BPDU Guard globally for PortFast ports | SW2 | `spanning-tree portfast bpduguard default` | Edge ports are protected if BPDUs are received |
| 38 | Configure trunk toward SW1 | SW2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 39 | Force trunk mode toward SW1 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 40 | Permit target VLANs toward SW1 | SW2 | `switchport trunk allowed vlan 10,20` | VLANs 10 and 20 are carried on the trunk |
| 41 | Ensure trunk is not treated as PortFast edge | SW2 | `spanning-tree portfast disable` | Switch-to-switch trunk remains non-edge |
| 42 | Optionally force RSTP point-to-point link type | SW2 | `spanning-tree link-type point-to-point` | RSTP can use rapid proposal/agreement on this full-duplex link |
| 43 | Bring up trunk toward SW1 | SW2 | `no shutdown` | Interface is administratively up |
| 44 | Configure trunk toward SW3 | SW2 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 45 | Force trunk mode toward SW3 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 46 | Permit target VLANs toward SW3 | SW2 | `switchport trunk allowed vlan 10,20` | VLANs 10 and 20 are carried on the trunk |
| 47 | Ensure trunk is not treated as PortFast edge | SW2 | `spanning-tree portfast disable` | Switch-to-switch trunk remains non-edge |
| 48 | Optionally force RSTP point-to-point link type | SW2 | `spanning-tree link-type point-to-point` | RSTP can use rapid proposal/agreement on this full-duplex link |
| 49 | Bring up trunk toward SW3 | SW2 | `no shutdown` | Interface is administratively up |
| 50 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 51 | Enter global configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 52 | Enable Rapid PVST+ mode | SW3 | `spanning-tree mode rapid-pvst` | SW3 uses Rapid PVST+ |
| 53 | Create VLAN 10 if missing | SW3 | `vlan 10` | VLAN 10 exists locally |
| 54 | Create VLAN 20 if missing | SW3 | `vlan 20` | VLAN 20 exists locally |
| 55 | Enable PortFast globally on access ports | SW3 | `spanning-tree portfast default` | Access ports become RSTP edge ports by default |
| 56 | Enable BPDU Guard globally for PortFast ports | SW3 | `spanning-tree portfast bpduguard default` | Edge ports are protected if BPDUs are received |
| 57 | Configure trunk toward SW1 | SW3 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 58 | Force trunk mode toward SW1 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 59 | Permit target VLANs toward SW1 | SW3 | `switchport trunk allowed vlan 10,20` | VLANs 10 and 20 are carried on the trunk |
| 60 | Ensure trunk is not treated as PortFast edge | SW3 | `spanning-tree portfast disable` | Switch-to-switch trunk remains non-edge |
| 61 | Optionally force RSTP point-to-point link type | SW3 | `spanning-tree link-type point-to-point` | RSTP can use rapid proposal/agreement on this full-duplex link |
| 62 | Bring up trunk toward SW1 | SW3 | `no shutdown` | Interface is administratively up |
| 63 | Configure trunk toward SW2 | SW3 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 64 | Force trunk mode toward SW2 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 65 | Permit target VLANs toward SW2 | SW3 | `switchport trunk allowed vlan 10,20` | VLANs 10 and 20 are carried on the trunk |
| 66 | Ensure trunk is not treated as PortFast edge | SW3 | `spanning-tree portfast disable` | Switch-to-switch trunk remains non-edge |
| 67 | Optionally force RSTP point-to-point link type | SW3 | `spanning-tree link-type point-to-point` | RSTP can use rapid proposal/agreement on this full-duplex link |
| 68 | Bring up trunk toward SW2 | SW3 | `no shutdown` | Interface is administratively up |
| 69 | Configure host-facing access port for VLAN 10 | SW3 | `interface GigabitEthernet0/3` | Prompt changes to interface configuration mode |
| 70 | Force access mode on host-facing port | SW3 | `switchport mode access` | Port is not trunking |
| 71 | Assign host-facing port to VLAN 10 | SW3 | `switchport access vlan 10` | Host traffic enters VLAN 10 |
| 72 | Enable PortFast on the host-facing port | SW3 | `spanning-tree portfast` | Port becomes an RSTP edge port |
| 73 | Enable BPDU Guard on the host-facing port | SW3 | `spanning-tree bpduguard enable` | Port is protected from accidental switch attachment |
| 74 | Bring up the host-facing port | SW3 | `no shutdown` | Interface is administratively up |
| 75 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 76 | Verify Rapid PVST+ mode on all switches | SW1/SW2/SW3 | `show spanning-tree summary` | Switches show Rapid PVST+ mode |
| 77 | Verify VLAN 10 root placement | SW1/SW2/SW3 | `show spanning-tree vlan 10` | SW1 is root for VLAN 10 |
| 78 | Verify VLAN 20 root placement | SW1/SW2/SW3 | `show spanning-tree vlan 20` | SW2 is root for VLAN 20 |
| 79 | Verify RSTP port types and roles | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Trunks show P2p and access edge ports show Edge |
| 80 | Verify interface-level RSTP details | SW1/SW2/SW3 | `show spanning-tree interface GigabitEthernet0/1 detail` | Link type, BPDU counters, and port state are visible |
| 81 | Test rapid failover by shutting the active root-facing link | SW3 | `configure terminal` then `interface GigabitEthernet0/1` then `shutdown` | Alternate path should move to forwarding rapidly |
| 82 | Verify alternate path moved to forwarding | SW3 | `show spanning-tree vlan 10` | Backup path becomes root forwarding or designated forwarding as expected |
| 83 | Restore the failed link | SW3 | `configure terminal` then `interface GigabitEthernet0/1` then `no shutdown` | Link returns to service and RSTP reconverges |
| 84 | Verify no inconsistent ports exist | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 85 | Verify no err-disabled access ports exist | SW1/SW2/SW3 | `show interfaces status err-disabled` | No required access ports are err-disabled |
| 86 | Verify host reachability after convergence | HOSTS | `ping <same-vlan-remote-host-ip>` | Same-VLAN traffic works after topology change |
| 87 | Save configuration | SW1/SW2/SW3 | `write memory` | Rapid PVST+ configuration survives reload |

# RSTP_Rapid_PVST_Convergence_Skeleton

SW1, Rapid PVST+ root for VLAN 10 and backup root for VLAN 20:

configure terminal
!
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root secondary
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree portfast disable
 spanning-tree link-type point-to-point
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree portfast disable
 spanning-tree link-type point-to-point
 no shutdown
!
end
write memory

SW2, Rapid PVST+ backup root for VLAN 10 and root for VLAN 20:

configure terminal
!
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
spanning-tree vlan 10 root secondary
spanning-tree vlan 20 root primary
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree portfast disable
 spanning-tree link-type point-to-point
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree portfast disable
 spanning-tree link-type point-to-point
 no shutdown
!
end
write memory

SW3, Rapid PVST+ downstream switch:

configure terminal
!
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree portfast disable
 spanning-tree link-type point-to-point
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree portfast disable
 spanning-tree link-type point-to-point
 no shutdown
!
interface GigabitEthernet0/3
 description H1_ACCESS_VLAN_10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
!
end
write memory

Optional interface cost steering for predictable failover:

configure terminal
!
interface GigabitEthernet0/1
 description PREFERRED_PATH_TO_VLAN_10_ROOT
 spanning-tree vlan 10 cost 4
 spanning-tree vlan 20 cost 19
!
interface GigabitEthernet0/2
 description PREFERRED_PATH_TO_VLAN_20_ROOT
 spanning-tree vlan 10 cost 19
 spanning-tree vlan 20 cost 4
!
end
write memory

Rapid convergence test:

configure terminal
interface GigabitEthernet0/1
 shutdown
end

show spanning-tree vlan 10
show spanning-tree interface GigabitEthernet0/2 detail
ping <same-vlan-remote-host-ip>

configure terminal
interface GigabitEthernet0/1
 no shutdown
end

# RSTP_Rapid_PVST_Convergence_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm Rapid PVST+ mode | SW1/SW2/SW3 | `show spanning-tree summary` | Switches show Rapid PVST+ mode |
| Confirm VLAN database | SW1/SW2/SW3 | `show vlan brief` | VLANs 10 and 20 exist and are active |
| Confirm trunk operation | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links show trunking |
| Confirm VLANs are allowed on trunks | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10 and 20 are allowed and active |
| Confirm VLAN 10 root bridge | SW1/SW2/SW3 | `show spanning-tree vlan 10` | SW1 is root for VLAN 10 |
| Confirm VLAN 20 root bridge | SW1/SW2/SW3 | `show spanning-tree vlan 20` | SW2 is root for VLAN 20 |
| Confirm root summary | SW1/SW2/SW3 | `show spanning-tree root` | Root placement matches the per-VLAN design |
| Confirm port roles for VLAN 10 | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root, designated, and alternate roles match design |
| Confirm port roles for VLAN 20 | SW1/SW2/SW3 | `show spanning-tree vlan 20` | Root, designated, and alternate roles match design |
| Confirm RSTP link type on trunk | SW1/SW2/SW3 | `show spanning-tree interface GigabitEthernet0/1 detail` | Link type shows point-to-point where expected |
| Confirm access edge behavior | SW3 | `show spanning-tree interface GigabitEthernet0/3 detail` | PortFast or edge behavior is shown for the host port |
| Confirm BPDU Guard state | SW3 | `show spanning-tree interface GigabitEthernet0/3 detail` | BPDU Guard state is visible if supported in output |
| Confirm access port is not err-disabled | SW3 | `show interfaces status err-disabled` | Host-facing port is not err-disabled |
| Confirm BPDU counters on trunk | SW1/SW2/SW3 | `show spanning-tree interface GigabitEthernet0/1 detail` | BPDU sent and received counters increment on switch-to-switch links |
| Confirm topology change count | SW1/SW2/SW3 | `show spanning-tree vlan 10 detail | include topology change|from|occurred` | Topology change information is visible and not constantly increasing |
| Confirm rapid failover path | SW3 | `show spanning-tree vlan 10` | Alternate path becomes forwarding after active path failure |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include SPANTREE|MACFLAP|LOOP` | No MAC flap or loop messages appear |
| Confirm MAC learning stability | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MAC addresses do not constantly move between trunks |
| Confirm host reachability | HOSTS | `ping <same-vlan-remote-host-ip>` | Same-VLAN traffic works after convergence |
| Confirm final STP config | SW1/SW2/SW3 | `show running-config | include spanning-tree` | Rapid PVST+, root, PortFast, and BPDU Guard commands are present |
| Confirm saved STP config | SW1/SW2/SW3 | `show startup-config | include spanning-tree` | Startup configuration contains intended STP settings |

# RSTP_Rapid_PVST_Convergence_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record Rapid PVST+ state before rollback | SW1/SW2/SW3 | `show spanning-tree summary` | Current STP mode and counters are documented |
| 2 | Record current root placement before rollback | SW1/SW2/SW3 | `show spanning-tree root` | Current per-VLAN root state is documented |
| 3 | Enter configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 4 | Remove global PortFast default if not wanted | SW1 | `no spanning-tree portfast default` | Access ports no longer inherit PortFast globally |
| 5 | Remove global BPDU Guard default if not wanted | SW1 | `no spanning-tree portfast bpduguard default` | PortFast ports no longer inherit BPDU Guard globally |
| 6 | Reset VLAN 10 root priority on SW1 | SW1 | `spanning-tree vlan 10 priority 32768` | VLAN 10 base priority returns to default |
| 7 | Reset VLAN 20 root priority on SW1 | SW1 | `spanning-tree vlan 20 priority 32768` | VLAN 20 base priority returns to default |
| 8 | Remove trunk link-type tuning toward SW2 | SW1 | `interface GigabitEthernet0/1` then `no spanning-tree link-type` | Interface returns to default link-type detection |
| 9 | Remove trunk link-type tuning toward SW3 | SW1 | `interface GigabitEthernet0/2` then `no spanning-tree link-type` | Interface returns to default link-type detection |
| 10 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 11 | Enter configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 12 | Remove global PortFast default if not wanted | SW2 | `no spanning-tree portfast default` | Access ports no longer inherit PortFast globally |
| 13 | Remove global BPDU Guard default if not wanted | SW2 | `no spanning-tree portfast bpduguard default` | PortFast ports no longer inherit BPDU Guard globally |
| 14 | Reset VLAN 10 root priority on SW2 | SW2 | `spanning-tree vlan 10 priority 32768` | VLAN 10 base priority returns to default |
| 15 | Reset VLAN 20 root priority on SW2 | SW2 | `spanning-tree vlan 20 priority 32768` | VLAN 20 base priority returns to default |
| 16 | Remove trunk link-type tuning toward SW1 | SW2 | `interface GigabitEthernet0/1` then `no spanning-tree link-type` | Interface returns to default link-type detection |
| 17 | Remove trunk link-type tuning toward SW3 | SW2 | `interface GigabitEthernet0/2` then `no spanning-tree link-type` | Interface returns to default link-type detection |
| 18 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Enter configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 20 | Remove global PortFast default if not wanted | SW3 | `no spanning-tree portfast default` | Access ports no longer inherit PortFast globally |
| 21 | Remove global BPDU Guard default if not wanted | SW3 | `no spanning-tree portfast bpduguard default` | PortFast ports no longer inherit BPDU Guard globally |
| 22 | Remove trunk link-type tuning toward SW1 | SW3 | `interface GigabitEthernet0/1` then `no spanning-tree link-type` | Interface returns to default link-type detection |
| 23 | Remove trunk link-type tuning toward SW2 | SW3 | `interface GigabitEthernet0/2` then `no spanning-tree link-type` | Interface returns to default link-type detection |
| 24 | Remove explicit host-port PortFast and BPDU Guard if needed | SW3 | `interface GigabitEthernet0/3` then `no spanning-tree portfast` then `no spanning-tree bpduguard enable` | Interface no longer has explicit edge protection settings |
| 25 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 26 | Optionally return all switches to classic PVST mode | SW1/SW2/SW3 | `configure terminal` then `spanning-tree mode pvst` then `end` | Switches return to classic PVST mode |
| 27 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |
| 28 | Verify STP mode after rollback | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode matches rollback target |
| 29 | Verify root placement after rollback | SW1/SW2/SW3 | `show spanning-tree root` | Root placement reflects remaining priority settings |
| 30 | Verify no inconsistent ports after rollback | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |

# RSTP_Rapid_PVST_Convergence_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Switch still converges slowly | Neighbor is not running RSTP or link is not point-to-point | SW1/SW2/SW3 | `show spanning-tree interface <interface> detail` | Run Rapid PVST+ on both sides and use full-duplex point-to-point links |
| Port does not show edge behavior | PortFast is missing or BPDU was received | Access SW | `show spanning-tree interface <interface> detail` | Configure `spanning-tree portfast` only on true host-facing ports |
| Access port becomes err-disabled | BPDU Guard received a BPDU on a PortFast port | Access SW | `show interfaces status err-disabled` | Remove the unauthorized switch, then shut/no shut the port |
| Switch-to-switch trunk is accidentally edge | PortFast was enabled on a trunk | SW1/SW2/SW3 | `show spanning-tree interface <interface> detail` | Remove PortFast with `spanning-tree portfast disable` |
| Root bridge is wrong | Root priority not configured or wrong switch has lower bridge ID | SW1/SW2/SW3 | `show spanning-tree root` | Configure intended root primary and secondary per VLAN |
| VLAN missing from Rapid PVST output | VLAN does not exist or has no active ports | SW1/SW2/SW3 | `show vlan brief` | Create the VLAN and activate access or trunk ports |
| VLAN not crossing trunk | VLAN not allowed on trunk | SW1/SW2/SW3 | `show interfaces trunk` | Add the VLAN to the trunk allowed list |
| Alternate port does not become forwarding after failure | Root placement, cost, or port role does not produce the expected backup path | SW3 | `show spanning-tree vlan <vlan-id>` | Verify root, root path cost, alternate role, and trunk state |
| Rapid convergence test fails | The failed link was not the active root path | SW3 | `show spanning-tree vlan <vlan-id>` | Identify the current root port before shutting a link |
| Port falls back to 802.1D behavior | Far end is not RSTP compatible or handshake fails | SW1/SW2/SW3 | `show spanning-tree interface <interface> detail` | Standardize STP mode and verify BPDUs are exchanged |
| Topology change count keeps increasing | Unstable link, host-facing BPDU issue, or loop condition | SW1/SW2/SW3 | `show spanning-tree vlan <vlan-id> detail` | Find the source interface and fix flapping or incorrect edge classification |
| MAC addresses flap | Layer 2 loop or unstable STP state | SW1/SW2/SW3 | `show logging | include MACFLAP|SPANTREE|LOOP` | Verify root placement, blocked ports, and remove unsafe redundant forwarding |
| Host has delay after link up | Access port is not PortFast edge | Access SW | `show spanning-tree interface <interface> detail` | Configure `spanning-tree portfast` on true host ports |
| DHCP or PXE times out on access host | Host port waits through STP state transitions | Access SW | `show spanning-tree interface <interface> detail` | Enable PortFast on the access port |
| BPDU Guard does not trigger on host port | BPDU Guard not enabled globally or locally | Access SW | `show running-config | include bpduguard` | Configure `spanning-tree portfast bpduguard default` or interface-level BPDU Guard |
| Trunk does not show point-to-point | Duplex mismatch or link-type not detected | SW1/SW2/SW3 | `show interfaces <interface> status` and `show spanning-tree interface <interface> detail` | Fix duplex or configure `spanning-tree link-type point-to-point` |
| VLAN 10 converges but VLAN 20 does not | Per-VLAN root, trunk, or STP state differs | SW1/SW2/SW3 | `show spanning-tree vlan 10` and `show spanning-tree vlan 20` | Troubleshoot each VLAN separately |
| STP mode changed but behavior is inconsistent | Not all switches were migrated to Rapid PVST+ | SW1/SW2/SW3 | `show spanning-tree summary` | Standardize `spanning-tree mode rapid-pvst` across the lab |
| Existing VLANs stopped crossing trunk | Allowed VLAN list was overwritten | SW1/SW2/SW3 | `show running-config interface <interface>` | Restore the full allowed list or use `switchport trunk allowed vlan add <vlan-list>` |
| Native VLAN mismatch appears | Native VLAN differs across trunk ends | SW1/SW2/SW3 | `show logging | include NATIVE|native` | Match native VLAN on both ends |
| Config disappears after reload | Configuration was not saved | SW1/SW2/SW3 | `show startup-config | include spanning-tree` | Reapply settings and run `write memory` |