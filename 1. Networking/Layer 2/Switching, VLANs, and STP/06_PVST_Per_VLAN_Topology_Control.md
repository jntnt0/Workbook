PVST_Per_VLAN_Topology_Control.md
# PVST_Per_VLAN_Topology_Control

# PVST_Per_VLAN_Topology_Control_Index

- PVST_Per_VLAN_Topology_Control_Mental_Model
- PVST_Per_VLAN_Topology_Control_Configuration_Checklist
- PVST_Per_VLAN_Topology_Control_Skeleton
- PVST_Per_VLAN_Topology_Control_Verification_Commands
- PVST_Per_VLAN_Topology_Control_Rollback
- PVST_Per_VLAN_Topology_Control_Failure_Checks

# PVST_Per_VLAN_Topology_Control_Mental_Model

| Concept | Operational Meaning |
|---|---|
| PVST | Cisco Per-VLAN Spanning Tree runs a separate STP instance for each VLAN |
| Per-VLAN root bridge | Each VLAN can have a different root bridge |
| Per-VLAN blocked port | A link can forward for one VLAN and block for another VLAN |
| Traffic steering | VLAN root placement and per-VLAN cost tuning can steer Layer 2 forwarding paths |
| VLAN load sharing | VLAN 10 can prefer SW1 while VLAN 20 prefers SW2 across the same physical topology |
| PVST scale cost | Every VLAN creates its own STP calculation, so too many VLANs can add control-plane overhead |
| Root primary | `spanning-tree vlan <vlan-id> root primary` makes a switch the preferred root for that VLAN |
| Root secondary | `spanning-tree vlan <vlan-id> root secondary` makes a switch the backup root for that VLAN |
| VLAN-specific cost | `spanning-tree vlan <vlan-id> cost <cost>` changes STP path selection for only that VLAN |
| Global cost risk | `spanning-tree cost <cost>` affects all VLANs on the interface unless the VLAN keyword is used |
| Trunk dependency | PVST only matters across trunks that actually carry the VLANs being tuned |
| Access VLAN dependency | A host VLAN must exist and be assigned to access ports before PVST forwarding can carry host traffic |
| Native VLAN separation | Native VLAN consistency matters on trunks, but PVST topology control is still done per VLAN |
| Root placement first | Pick the root bridge per VLAN before tuning interface costs |
| Verification rule | Always verify each VLAN separately with `show spanning-tree vlan <vlan-id>` |

# PVST_Per_VLAN_Topology_Control_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm physical switch adjacency before STP tuning | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended triangle or redundant topology |
| 2 | Confirm interswitch links are connected | SW1/SW2/SW3 | `show interfaces status` | Interswitch ports show `connected` |
| 3 | Confirm trunks are already formed | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch ports show `trunking` |
| 4 | Confirm current spanning-tree mode | SW1/SW2/SW3 | `show spanning-tree summary` | Current STP mode is known before changes |
| 5 | Enter global configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 6 | Enable classic PVST mode | SW1 | `spanning-tree mode pvst` | SW1 runs PVST |
| 7 | Create VLAN 10 | SW1 | `vlan 10` | VLAN 10 exists locally |
| 8 | Name VLAN 10 | SW1 | `name USERS` | VLAN 10 has a readable name |
| 9 | Create VLAN 20 | SW1 | `vlan 20` | VLAN 20 exists locally |
| 10 | Name VLAN 20 | SW1 | `name SERVERS` | VLAN 20 has a readable name |
| 11 | Configure SW1 as primary root for VLAN 10 | SW1 | `spanning-tree vlan 10 root primary` | SW1 becomes preferred root for VLAN 10 |
| 12 | Configure SW1 as secondary root for VLAN 20 | SW1 | `spanning-tree vlan 20 root secondary` | SW1 becomes backup root for VLAN 20 |
| 13 | Configure trunk toward SW2 to carry both VLANs | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 14 | Force trunking toward SW2 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 15 | Permit VLANs 10 and 20 on the trunk | SW1 | `switchport trunk allowed vlan 10,20` | Trunk is limited to the intended test VLANs |
| 16 | Bring up the trunk interface | SW1 | `no shutdown` | Interface is administratively up |
| 17 | Configure trunk toward SW3 to carry both VLANs | SW1 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 18 | Force trunking toward SW3 | SW1 | `switchport mode trunk` | Interface is administratively trunking |
| 19 | Permit VLANs 10 and 20 on the trunk | SW1 | `switchport trunk allowed vlan 10,20` | Trunk is limited to the intended test VLANs |
| 20 | Bring up the trunk interface | SW1 | `no shutdown` | Interface is administratively up |
| 21 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 22 | Enter global configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 23 | Enable classic PVST mode | SW2 | `spanning-tree mode pvst` | SW2 runs PVST |
| 24 | Create VLAN 10 | SW2 | `vlan 10` | VLAN 10 exists locally |
| 25 | Name VLAN 10 | SW2 | `name USERS` | VLAN 10 has a readable name |
| 26 | Create VLAN 20 | SW2 | `vlan 20` | VLAN 20 exists locally |
| 27 | Name VLAN 20 | SW2 | `name SERVERS` | VLAN 20 has a readable name |
| 28 | Configure SW2 as secondary root for VLAN 10 | SW2 | `spanning-tree vlan 10 root secondary` | SW2 becomes backup root for VLAN 10 |
| 29 | Configure SW2 as primary root for VLAN 20 | SW2 | `spanning-tree vlan 20 root primary` | SW2 becomes preferred root for VLAN 20 |
| 30 | Configure trunk toward SW1 to carry both VLANs | SW2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 31 | Force trunking toward SW1 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 32 | Permit VLANs 10 and 20 on the trunk | SW2 | `switchport trunk allowed vlan 10,20` | Trunk carries both PVST-controlled VLANs |
| 33 | Bring up the trunk interface | SW2 | `no shutdown` | Interface is administratively up |
| 34 | Configure trunk toward SW3 to carry both VLANs | SW2 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 35 | Force trunking toward SW3 | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 36 | Permit VLANs 10 and 20 on the trunk | SW2 | `switchport trunk allowed vlan 10,20` | Trunk carries both PVST-controlled VLANs |
| 37 | Bring up the trunk interface | SW2 | `no shutdown` | Interface is administratively up |
| 38 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 39 | Enter global configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 40 | Enable classic PVST mode | SW3 | `spanning-tree mode pvst` | SW3 runs PVST |
| 41 | Create VLAN 10 | SW3 | `vlan 10` | VLAN 10 exists locally |
| 42 | Name VLAN 10 | SW3 | `name USERS` | VLAN 10 has a readable name |
| 43 | Create VLAN 20 | SW3 | `vlan 20` | VLAN 20 exists locally |
| 44 | Name VLAN 20 | SW3 | `name SERVERS` | VLAN 20 has a readable name |
| 45 | Configure trunk toward SW1 to carry both VLANs | SW3 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 46 | Force trunking toward SW1 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 47 | Permit VLANs 10 and 20 on the trunk | SW3 | `switchport trunk allowed vlan 10,20` | Trunk carries both PVST-controlled VLANs |
| 48 | Bring up the trunk interface | SW3 | `no shutdown` | Interface is administratively up |
| 49 | Configure trunk toward SW2 to carry both VLANs | SW3 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 50 | Force trunking toward SW2 | SW3 | `switchport mode trunk` | Interface is administratively trunking |
| 51 | Permit VLANs 10 and 20 on the trunk | SW3 | `switchport trunk allowed vlan 10,20` | Trunk carries both PVST-controlled VLANs |
| 52 | Bring up the trunk interface | SW3 | `no shutdown` | Interface is administratively up |
| 53 | Optionally steer VLAN 10 to prefer the SW1 path | SW3 | `interface GigabitEthernet0/1` then `spanning-tree vlan 10 cost 4` | VLAN 10 prefers the lower-cost path toward SW1 |
| 54 | Optionally make the alternate VLAN 10 path less preferred | SW3 | `interface GigabitEthernet0/2` then `spanning-tree vlan 10 cost 19` | VLAN 10 avoids the higher-cost path unless needed |
| 55 | Optionally steer VLAN 20 to prefer the SW2 path | SW3 | `interface GigabitEthernet0/2` then `spanning-tree vlan 20 cost 4` | VLAN 20 prefers the lower-cost path toward SW2 |
| 56 | Optionally make the alternate VLAN 20 path less preferred | SW3 | `interface GigabitEthernet0/1` then `spanning-tree vlan 20 cost 19` | VLAN 20 avoids the higher-cost path unless needed |
| 57 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 58 | Verify SW1 is root for VLAN 10 | SW1 | `show spanning-tree vlan 10` | Output shows `This bridge is the root` |
| 59 | Verify SW2 is root for VLAN 20 | SW2 | `show spanning-tree vlan 20` | Output shows `This bridge is the root` |
| 60 | Verify SW3 has different root ports per VLAN if load sharing is intended | SW3 | `show spanning-tree vlan 10` and `show spanning-tree vlan 20` | VLAN 10 and VLAN 20 can use different root ports |
| 61 | Verify VLANs are allowed and forwarding on trunks | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10 and 20 appear as allowed, active, and forwarding where expected |
| 62 | Verify no inconsistent STP ports exist | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 63 | Verify same-VLAN host reachability for VLAN 10 | HOSTS | `ping <remote-vlan-10-host-ip>` | VLAN 10 host traffic works across the PVST topology |
| 64 | Verify same-VLAN host reachability for VLAN 20 | HOSTS | `ping <remote-vlan-20-host-ip>` | VLAN 20 host traffic works across the PVST topology |
| 65 | Save the configuration | SW1/SW2/SW3 | `write memory` | PVST configuration survives reload |

# PVST_Per_VLAN_Topology_Control_Skeleton

SW1, primary root for VLAN 10 and secondary root for VLAN 20:

configure terminal
!
spanning-tree mode pvst
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
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

SW2, secondary root for VLAN 10 and primary root for VLAN 20:

configure terminal
!
spanning-tree mode pvst
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
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

SW3, access or downstream switch with per-VLAN cost steering:

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
 description TRUNK_TO_SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree vlan 10 cost 4
 spanning-tree vlan 20 cost 19
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK_TO_SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 spanning-tree vlan 10 cost 19
 spanning-tree vlan 20 cost 4
 no shutdown
!
end
write memory

Manual priority alternative:

SW1:

configure terminal
spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 8192
end
write memory

SW2:

configure terminal
spanning-tree vlan 10 priority 8192
spanning-tree vlan 20 priority 4096
end
write memory

Add VLANs to an existing trunk without overwriting the current allowed list:

configure terminal
interface GigabitEthernet0/1
 switchport trunk allowed vlan add 10,20
end
write memory

# PVST_Per_VLAN_Topology_Control_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm PVST mode | SW1/SW2/SW3 | `show spanning-tree summary` | Switches show PVST mode |
| Confirm VLAN database | SW1/SW2/SW3 | `show vlan brief` | VLANs 10 and 20 exist and are active |
| Confirm trunk state | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links are trunking |
| Confirm VLANs are allowed on trunks | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10 and 20 are listed as allowed |
| Confirm VLANs are active on trunks | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10 and 20 appear as allowed and active |
| Confirm VLANs are forwarding on trunks | SW1/SW2/SW3 | `show interfaces trunk` | VLANs 10 and 20 appear in forwarding state where expected |
| Confirm VLAN 10 root bridge | SW1 | `show spanning-tree vlan 10` | SW1 shows `This bridge is the root` |
| Confirm VLAN 20 root bridge | SW2 | `show spanning-tree vlan 20` | SW2 shows `This bridge is the root` |
| View all root placement at once | SW1/SW2/SW3 | `show spanning-tree root` | VLAN 10 root is SW1 and VLAN 20 root is SW2 |
| Confirm SW3 VLAN 10 root port | SW3 | `show spanning-tree vlan 10` | Intended interface toward SW1 is root forwarding |
| Confirm SW3 VLAN 20 root port | SW3 | `show spanning-tree vlan 20` | Intended interface toward SW2 is root forwarding |
| Confirm VLAN 10 blocked port | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Redundant port is blocking or alternate for VLAN 10 |
| Confirm VLAN 20 blocked port | SW1/SW2/SW3 | `show spanning-tree vlan 20` | Redundant port is blocking or alternate for VLAN 20 |
| Confirm per-VLAN cost on interface | SW3 | `show spanning-tree interface GigabitEthernet0/1 detail` | Interface shows configured STP cost behavior |
| Confirm running STP config | SW1/SW2/SW3 | `show running-config | include spanning-tree vlan` | Root and cost commands are present |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include SPANTREE|MACFLAP|LOOP` | No MAC flapping or loop messages appear |
| Confirm VLAN 10 MAC learning | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | VLAN 10 MAC addresses are learned on expected ports |
| Confirm VLAN 20 MAC learning | SW1/SW2/SW3 | `show mac address-table dynamic vlan 20` | VLAN 20 MAC addresses are learned on expected ports |
| Confirm VLAN 10 host reachability | HOSTS | `ping <remote-vlan-10-host-ip>` | Same-VLAN hosts in VLAN 10 can communicate |
| Confirm VLAN 20 host reachability | HOSTS | `ping <remote-vlan-20-host-ip>` | Same-VLAN hosts in VLAN 20 can communicate |
| Confirm saved STP config | SW1/SW2/SW3 | `show startup-config | include spanning-tree` | Startup config contains intended PVST settings |

# PVST_Per_VLAN_Topology_Control_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current root and port roles before rollback | SW1/SW2/SW3 | `show spanning-tree root` | Current per-VLAN root state is documented |
| 2 | Record VLAN 10 STP state before rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | VLAN 10 root, roles, and states are documented |
| 3 | Record VLAN 20 STP state before rollback | SW1/SW2/SW3 | `show spanning-tree vlan 20` | VLAN 20 root, roles, and states are documented |
| 4 | Enter configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 5 | Reset VLAN 10 bridge priority on SW1 | SW1 | `spanning-tree vlan 10 priority 32768` | VLAN 10 base bridge priority returns to default |
| 6 | Reset VLAN 20 bridge priority on SW1 | SW1 | `spanning-tree vlan 20 priority 32768` | VLAN 20 base bridge priority returns to default |
| 7 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 8 | Enter configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 9 | Reset VLAN 10 bridge priority on SW2 | SW2 | `spanning-tree vlan 10 priority 32768` | VLAN 10 base bridge priority returns to default |
| 10 | Reset VLAN 20 bridge priority on SW2 | SW2 | `spanning-tree vlan 20 priority 32768` | VLAN 20 base bridge priority returns to default |
| 11 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 12 | Enter configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 13 | Remove VLAN 10 cost tuning from trunk toward SW1 | SW3 | `interface GigabitEthernet0/1` then `no spanning-tree vlan 10 cost` | VLAN 10 interface cost returns to default |
| 14 | Remove VLAN 20 cost tuning from trunk toward SW1 | SW3 | `interface GigabitEthernet0/1` then `no spanning-tree vlan 20 cost` | VLAN 20 interface cost returns to default |
| 15 | Remove VLAN 10 cost tuning from trunk toward SW2 | SW3 | `interface GigabitEthernet0/2` then `no spanning-tree vlan 10 cost` | VLAN 10 interface cost returns to default |
| 16 | Remove VLAN 20 cost tuning from trunk toward SW2 | SW3 | `interface GigabitEthernet0/2` then `no spanning-tree vlan 20 cost` | VLAN 20 interface cost returns to default |
| 17 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 18 | Remove test VLANs if the lab is being fully cleared | SW1/SW2/SW3 | `configure terminal` then `no vlan 10` and `no vlan 20` | Test VLANs are removed locally |
| 19 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |
| 20 | Verify root placement after rollback | SW1/SW2/SW3 | `show spanning-tree root` | Root placement is recalculated from remaining defaults |
| 21 | Verify per-VLAN cost rollback | SW3 | `show running-config interface GigabitEthernet0/1` | VLAN-specific STP cost commands are removed |
| 22 | Verify VLAN cleanup if performed | SW1/SW2/SW3 | `show vlan brief` | VLANs 10 and 20 are gone if intentionally removed |

# PVST_Per_VLAN_Topology_Control_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| VLAN 10 and VLAN 20 use the same root when load sharing was intended | Per-VLAN root placement was not configured correctly | SW1/SW2 | `show spanning-tree root` | Configure separate root primary and secondary roles per VLAN |
| Wrong switch is root for VLAN 10 | Another switch has a lower bridge ID for VLAN 10 | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Lower the intended root priority for VLAN 10 |
| Wrong switch is root for VLAN 20 | Another switch has a lower bridge ID for VLAN 20 | SW1/SW2/SW3 | `show spanning-tree vlan 20` | Lower the intended root priority for VLAN 20 |
| VLAN appears in VLAN database but not trunk forwarding | VLAN is not allowed or STP is blocking the trunk | SW1/SW2/SW3 | `show interfaces trunk` | Add the VLAN to the allowed list and verify STP forwarding |
| VLAN missing from STP output | VLAN does not exist or has no active ports | SW1/SW2/SW3 | `show vlan brief` | Create the VLAN and assign active access or trunk ports |
| Per-VLAN cost does not change root port | Root bridge placement or upstream BPDU cost still wins | SW3 | `show spanning-tree vlan 10` | Verify root placement first, then adjust VLAN-specific cost |
| Cost tuning affects both VLANs unexpectedly | Cost was configured globally without the VLAN keyword | SW3 | `show running-config interface GigabitEthernet0/1` | Remove global cost and use `spanning-tree vlan <vlan-id> cost <cost>` |
| VLAN 10 host traffic fails but VLAN 20 works | VLAN 10 is not allowed, active, or forwarding somewhere on the trunk path | SW1/SW2/SW3 | `show interfaces trunk` | Fix VLAN 10 trunk allowance or STP state |
| VLAN 20 host traffic fails but VLAN 10 works | VLAN 20 is not allowed, active, or forwarding somewhere on the trunk path | SW1/SW2/SW3 | `show interfaces trunk` | Fix VLAN 20 trunk allowance or STP state |
| Both VLANs fail across the same link | Trunk is down or not trunking | SW1/SW2/SW3 | `show interfaces status` and `show interfaces trunk` | Bring the interface up and force trunk mode |
| VLAN root placement changes after reload | Configuration was not saved | SW1/SW2/SW3 | `show startup-config | include spanning-tree` | Reapply intended PVST commands and run `write memory` |
| MAC addresses flap between trunks | Layer 2 loop or STP not blocking a redundant path | SW1/SW2/SW3 | `show logging | include MACFLAP|SPANTREE|LOOP` | Restore STP, verify one root per VLAN, and confirm blocked ports |
| Unexpected blocked port appears | STP selected the path based on bridge ID, cost, port priority, and port number | SW1/SW2/SW3 | `show spanning-tree vlan <vlan-id>` | Adjust root placement first, then cost if the block is not desired |
| Root bridge is correct but traffic path is inefficient | Per-VLAN cost was not tuned for the desired forwarding path | SW3 | `show spanning-tree vlan 10` and `show spanning-tree vlan 20` | Tune VLAN-specific interface costs |
| Trunk allowed list overwrote existing VLANs | `switchport trunk allowed vlan` replaced the old list | SW1/SW2/SW3 | `show running-config interface <interface>` | Restore the full allowed list or use `switchport trunk allowed vlan add <vlan-list>` |
| Native VLAN mismatch appears | Trunk native VLANs differ | SW1/SW2/SW3 | `show logging | include NATIVE|native` | Configure matching native VLANs on both sides |
| Switch shows a different STP mode | Mixed STP modes change behavior and output | SW1/SW2/SW3 | `show spanning-tree summary` | Standardize the lab on `spanning-tree mode pvst` |
| CPU load rises with many VLANs | PVST creates one STP instance per VLAN | SW1/SW2/SW3 | `show spanning-tree summary` | Consider MST for larger VLAN counts |
| Access host cannot reach same VLAN across topology | Access port, VLAN membership, trunk, or STP forwarding is wrong | SW1/SW2/SW3 | `show vlan brief`, `show interfaces trunk`, `show spanning-tree vlan <vlan-id>` | Fix access VLAN assignment, trunk allowance, or PVST state |
