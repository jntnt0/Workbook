STP_8021D_Root_Election_Port_Roles.md
# STP_8021D_Root_Election_Port_Roles

# STP_8021D_Root_Election_Port_Roles_Index

- STP_8021D_Root_Election_Port_Roles_Mental_Model
- STP_8021D_Root_Election_Port_Roles_Configuration_Checklist
- STP_8021D_Root_Election_Port_Roles_Skeleton
- STP_8021D_Root_Election_Port_Roles_Verification_Commands
- STP_8021D_Root_Election_Port_Roles_Rollback
- STP_8021D_Root_Election_Port_Roles_Failure_Checks

# STP_8021D_Root_Election_Port_Roles_Mental_Model

| Concept | Operational Meaning |
|---|---|
| STP purpose | STP prevents Layer 2 loops by allowing only one active logical path through a bridged topology |
| 802.1D | Classic STP uses root election, root ports, designated ports, and blocking ports to build a loop-free tree |
| PVST+ | Cisco PVST+ runs a separate 802.1D-style spanning-tree instance per VLAN |
| Root bridge | The root bridge is the reference point for the entire STP calculation in a VLAN |
| Bridge ID | The bridge ID is made from bridge priority, VLAN system ID extension, and switch MAC address |
| Lowest bridge ID wins | The switch with the lowest bridge ID becomes root for that VLAN |
| Root priority | Lower priority is better; default base priority is 32768 before the VLAN system ID extension is added |
| Root primary macro | `spanning-tree vlan <vlan-id> root primary` lowers the bridge priority so the switch should become root |
| Root secondary macro | `spanning-tree vlan <vlan-id> root secondary` sets a backup root priority behind the primary root |
| Root port | Every non-root switch selects one best port toward the root bridge |
| Designated port | Each Layer 2 segment has one designated port that forwards toward that segment |
| Blocking port | A redundant port that is not forwarding user traffic because STP blocked it to prevent a loop |
| Root bridge ports | All ports on the root bridge are designated ports, assuming they are participating and forwarding |
| Root path cost | A switch chooses its root port based on the lowest accumulated cost toward the root |
| Port priority | Port priority is a tie-breaker when path cost and upstream bridge ID are equal |
| Port roles are calculated | You do not directly configure a port as root, designated, or blocking; you influence the calculation with bridge priority, port cost, and port priority |
| VLAN-specific behavior | Root election and port roles must be checked per VLAN, not just globally |
| STP first rule | Pick the root bridge intentionally before troubleshooting blocked links |

# STP_8021D_Root_Election_Port_Roles_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the Layer 2 topology and target VLAN before changing STP | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended topology |
| 2 | Confirm trunk links are up between switches | SW1/SW2/SW3 | `show interfaces trunk` | Interswitch links show `trunking` |
| 3 | Confirm the target VLAN exists on every participating switch | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| 4 | Confirm current STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | Current STP mode is known before changes |
| 5 | Enter global configuration mode on the intended primary root | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 6 | Use classic PVST mode for 802.1D-style per-VLAN STP behavior | SW1 | `spanning-tree mode pvst` | SW1 runs PVST+ |
| 7 | Create the target VLAN if missing | SW1 | `vlan 10` | VLAN 10 exists locally |
| 8 | Name the VLAN | SW1 | `name USERS` | VLAN 10 has a readable label |
| 9 | Make SW1 the primary root for VLAN 10 | SW1 | `spanning-tree vlan 10 root primary` | SW1 bridge priority is lowered for VLAN 10 |
| 10 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 11 | Enter global configuration mode on the intended secondary root | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 12 | Use classic PVST mode for 802.1D-style per-VLAN STP behavior | SW2 | `spanning-tree mode pvst` | SW2 runs PVST+ |
| 13 | Create the target VLAN if missing | SW2 | `vlan 10` | VLAN 10 exists locally |
| 14 | Name the VLAN | SW2 | `name USERS` | VLAN 10 has a readable label |
| 15 | Make SW2 the secondary root for VLAN 10 | SW2 | `spanning-tree vlan 10 root secondary` | SW2 bridge priority is lower than default but higher than the primary root |
| 16 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 17 | Enter global configuration mode on the downstream switch | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 18 | Use classic PVST mode for 802.1D-style per-VLAN STP behavior | SW3 | `spanning-tree mode pvst` | SW3 runs PVST+ |
| 19 | Create the target VLAN if missing | SW3 | `vlan 10` | VLAN 10 exists locally |
| 20 | Name the VLAN | SW3 | `name USERS` | VLAN 10 has a readable label |
| 21 | Exit configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 22 | Verify SW1 became the root bridge for VLAN 10 | SW1 | `show spanning-tree vlan 10` | Output shows `This bridge is the root` |
| 23 | Verify SW2 sees SW1 as root and itself as backup candidate | SW2 | `show spanning-tree vlan 10` | Root ID points to SW1 and Bridge ID shows secondary-root priority |
| 24 | Verify SW3 selected one root port toward SW1 | SW3 | `show spanning-tree vlan 10` | Exactly one interface shows `Root FWD` |
| 25 | Verify root bridge ports are designated | SW1 | `show spanning-tree vlan 10` | Participating ports on SW1 show `Desg FWD` |
| 26 | Verify each shared segment has one designated port | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Each interswitch segment has one forwarding designated port |
| 27 | Verify redundant paths are blocked where needed | SW2/SW3 | `show spanning-tree vlan 10` | One redundant port shows blocking or alternate behavior depending platform/STP display |
| 28 | Optionally steer SW3 root-port choice with STP cost | SW3 | `configure terminal` then `interface GigabitEthernet0/1` then `spanning-tree vlan 10 cost 4` | Interface cost for VLAN 10 is explicitly set |
| 29 | Optionally make the less-preferred path more expensive | SW3 | `interface GigabitEthernet0/2` then `spanning-tree vlan 10 cost 19` | SW3 prefers the lower-cost path toward the root |
| 30 | Exit configuration mode after cost tuning | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 31 | Verify the intended root port after cost tuning | SW3 | `show spanning-tree vlan 10` | Intended interface shows `Root FWD` |
| 32 | Optionally tune upstream port priority for a parallel-link tie-breaker | SW1 | `configure terminal` then `interface GigabitEthernet0/2` then `spanning-tree vlan 10 port-priority 64` | Upstream port becomes preferred when bridge ID and cost tie |
| 33 | Exit configuration mode after port-priority tuning | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 34 | Verify port priority is reflected in STP output | SW1 | `show spanning-tree vlan 10` | Interface shows lower `Prio.Nbr` value |
| 35 | Verify no unexpected STP inconsistency exists | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 36 | Verify no MAC flapping symptoms exist | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | No loop or MAC flap messages appear |
| 37 | Save configuration on all switches | SW1/SW2/SW3 | `write memory` | STP configuration survives reload |

# STP_8021D_Root_Election_Port_Roles_Skeleton

Primary root switch:

configure terminal
!
spanning-tree mode pvst
!
vlan 10
 name USERS
!
spanning-tree vlan 10 root primary
!
end
write memory

Secondary root switch:

configure terminal
!
spanning-tree mode pvst
!
vlan 10
 name USERS
!
spanning-tree vlan 10 root secondary
!
end
write memory

Downstream switch:

configure terminal
!
spanning-tree mode pvst
!
vlan 10
 name USERS
!
end
write memory

Optional direct bridge-priority method instead of root macro:

Primary root:

configure terminal
spanning-tree vlan 10 priority 4096
end
write memory

Secondary root:

configure terminal
spanning-tree vlan 10 priority 8192
end
write memory

Optional root-port steering with path cost:

configure terminal
!
interface GigabitEthernet0/1
 description PREFERRED_PATH_TO_ROOT
 spanning-tree vlan 10 cost 4
!
interface GigabitEthernet0/2
 description BACKUP_PATH_TO_ROOT
 spanning-tree vlan 10 cost 19
!
end
write memory

Optional parallel-link tie-breaker with port priority:

configure terminal
!
interface GigabitEthernet0/2
 spanning-tree vlan 10 port-priority 64
!
end
write memory

Basic trunk baseline for each interswitch link if needed:

configure terminal
!
interface GigabitEthernet0/1
 description TRUNK_TO_NEIGHBOR
 switchport mode trunk
 switchport trunk allowed vlan add 10
 no shutdown
!
end
write memory

# STP_8021D_Root_Election_Port_Roles_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | Switches show PVST mode for classic per-VLAN STP behavior |
| Confirm VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists on every participating switch |
| Confirm trunks are carrying the VLAN | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 10 is allowed, active, and forwarding where expected |
| Confirm root bridge | SW1 | `show spanning-tree vlan 10` | Output shows `This bridge is the root` |
| Confirm root ID from non-root switch | SW2/SW3 | `show spanning-tree vlan 10` | Root ID matches SW1 bridge ID |
| Confirm primary root priority | SW1 | `show spanning-tree vlan 10 | include Bridge ID|priority|This bridge` | SW1 has the lowest bridge priority for VLAN 10 |
| Confirm secondary root priority | SW2 | `show spanning-tree vlan 10 | include Bridge ID|priority` | SW2 priority is lower than default but higher than SW1 |
| Confirm downstream root port | SW3 | `show spanning-tree vlan 10` | Exactly one interface shows `Root FWD` |
| Confirm designated ports | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Segment-facing forwarding ports show `Desg FWD` |
| Confirm blocked redundant port | SW2/SW3 | `show spanning-tree vlan 10` | Redundant path shows blocking or alternate state |
| Confirm path cost to root | SW2/SW3 | `show spanning-tree vlan 10` | Root path cost matches the intended topology |
| Confirm interface cost | SW3 | `show spanning-tree interface GigabitEthernet0/1 detail` | Interface cost matches configured value |
| Confirm port priority | SW1 | `show spanning-tree vlan 10` | Tuned interface shows the expected `Prio.Nbr` value |
| Confirm STP state by interface | SW1/SW2/SW3 | `show spanning-tree interface GigabitEthernet0/1` | Interface role and state match design |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include MACFLAP|SPANTREE|LOOP` | No MAC flapping or STP loop messages appear |
| Confirm MAC learning is stable | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MACs do not constantly move between ports |
| Confirm host reachability after STP settles | H1/H2 | `ping <same-vlan-remote-host>` | Same-VLAN hosts can reach each other through the forwarding path |
| Confirm saved configuration | SW1/SW2/SW3 | `show startup-config | include spanning-tree` | Startup config contains intended STP settings |

# STP_8021D_Root_Election_Port_Roles_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Document current STP state before rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Current root bridge, root ports, and blocked ports are recorded |
| 2 | Enter configuration mode on the primary root | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 3 | Remove root primary macro effect by resetting VLAN priority | SW1 | `no spanning-tree vlan 10 priority` | VLAN 10 priority returns to default behavior |
| 4 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 5 | Enter configuration mode on the secondary root | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 6 | Remove root secondary macro effect by resetting VLAN priority | SW2 | `no spanning-tree vlan 10 priority` | VLAN 10 priority returns to default behavior |
| 7 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 8 | Remove interface cost tuning if configured | SW3 | `configure terminal` then `interface GigabitEthernet0/1` then `no spanning-tree vlan 10 cost` | Interface returns to default STP cost |
| 9 | Remove backup-path cost tuning if configured | SW3 | `interface GigabitEthernet0/2` then `no spanning-tree vlan 10 cost` | Interface returns to default STP cost |
| 10 | Exit configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 11 | Remove port-priority tuning if configured | SW1 | `configure terminal` then `interface GigabitEthernet0/2` then `no spanning-tree vlan 10 port-priority` | Interface returns to default STP port priority |
| 12 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 13 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |
| 14 | Verify root election after rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root bridge is recalculated based on remaining default values |
| 15 | Verify port roles after rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root, designated, and blocking ports reflect default STP calculation |

# STP_8021D_Root_Election_Port_Roles_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Wrong switch becomes root | Lower bridge ID exists on another switch | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Configure intended root with lower priority or `spanning-tree vlan 10 root primary` |
| Backup root is not preferred after primary fails | Secondary root priority was not configured or is too high | SW2 | `show spanning-tree vlan 10` | Configure `spanning-tree vlan 10 root secondary` or an explicit lower priority |
| Port roles do not match expected topology | Path cost calculation favors another link | SW2/SW3 | `show spanning-tree vlan 10` | Adjust `spanning-tree vlan 10 cost <cost>` on the intended interfaces |
| Root port is not the intended uplink | Lower accumulated path cost exists through another path | SW3 | `show spanning-tree vlan 10` | Lower cost on the preferred uplink or raise cost on the backup uplink |
| Parallel links choose the wrong forwarding port | Port priority and port number tie-breaker favors another port | Upstream SW | `show spanning-tree vlan 10` | Tune upstream `spanning-tree vlan 10 port-priority <priority>` |
| VLAN does not appear in STP output | VLAN is missing or inactive | SW1/SW2/SW3 | `show vlan brief` | Create VLAN 10 and ensure it is active |
| VLAN does not cross trunk | VLAN is not allowed on trunk | SW1/SW2/SW3 | `show interfaces trunk` | Add VLAN 10 to the trunk allowed list |
| Trunk is up but STP blocks expected forwarding path | STP selected a loop-free blocked port | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Validate whether the block is expected; adjust root placement or cost only if design requires it |
| Host traffic fails after STP change | Access VLAN, trunk VLAN, or STP forwarding path is broken | SW1/SW2/SW3 | `show vlan brief`, `show interfaces trunk`, `show spanning-tree vlan 10` | Fix VLAN membership, trunk allowance, or STP root/cost design |
| All ports forward and loop forms | STP disabled or bypassed somewhere | SW1/SW2/SW3 | `show spanning-tree summary` | Re-enable STP and remove unsafe Layer 2 bridging |
| MAC addresses flap between ports | Layer 2 loop or unstable redundant path | SW1/SW2/SW3 | `show logging | include MACFLAP` | Check STP state, remove loop, and verify blocked port exists |
| CPU spikes or switch becomes unstable | Broadcast storm from Layer 2 loop | SW1/SW2/SW3 | `show processes cpu sorted` | Restore STP protection and break the physical loop |
| Interface is not participating in STP | Port is routed, down, or VLAN not present | SW1/SW2/SW3 | `show interfaces switchport` | Make it a Layer 2 port, bring it up, and ensure VLAN exists |
| STP mode differs across switches | Mixed STP modes can change behavior and timers | SW1/SW2/SW3 | `show spanning-tree summary` | Standardize STP mode for the lab |
| Root bridge changed after reload | Config was not saved | SW1/SW2/SW3 | `show startup-config | include spanning-tree` | Reapply STP priority and run `write memory` |
| Cost tuning affects all VLANs unexpectedly | Cost was configured without VLAN keyword | SW1/SW2/SW3 | `show running-config interface <interface>` | Use `spanning-tree vlan 10 cost <cost>` for VLAN-specific tuning |
| Port-priority tuning affects all VLANs unexpectedly | Port priority was configured without VLAN keyword | SW1/SW2/SW3 | `show running-config interface <interface>` | Use `spanning-tree vlan 10 port-priority <priority>` for VLAN-specific tuning |
| STP output shows unexpected root path cost | Interface speeds or costs differ from assumption | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Validate link speed and configure explicit STP costs if needed |
| Port stuck in blocking unexpectedly | It may be the correct redundant blocked port | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Compare root, cost, sender bridge ID, port priority, and port number before changing anything |
