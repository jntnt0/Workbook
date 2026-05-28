STP_Root_Bridge_Placement.md
# STP_Root_Bridge_Placement

# STP_Root_Bridge_Placement_Index

- STP_Root_Bridge_Placement_Mental_Model
- STP_Root_Bridge_Placement_Configuration_Checklist
- STP_Root_Bridge_Placement_Skeleton
- STP_Root_Bridge_Placement_Verification_Commands
- STP_Root_Bridge_Placement_Rollback
- STP_Root_Bridge_Placement_Failure_Checks

# STP_Root_Bridge_Placement_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Root bridge placement | The STP root bridge should be chosen deliberately, not left to the lowest MAC address |
| Root bridge role | The root bridge is the Layer 2 reference point used by all switches to calculate root ports, designated ports, and blocked ports |
| Lowest bridge ID wins | The switch with the lowest bridge ID becomes root for that VLAN |
| Bridge ID | Bridge ID is made from priority, VLAN system ID extension, and switch MAC address |
| Priority control | Lower priority is better; Cisco priorities are configured in increments of 4096 |
| Default priority | Default base priority is 32768 before the VLAN system ID extension is added |
| Primary root | The intended root bridge should have the lowest priority for the VLAN |
| Secondary root | A backup root bridge should have the next-lowest priority so it takes over if the primary fails |
| Core or distribution placement | The root should usually sit at the stable aggregation point, not on an access switch |
| Per-VLAN root control | PVST and Rapid PVST calculate root bridge placement per VLAN |
| VLAN load sharing | Different VLANs can intentionally use different roots, but this must be planned and verified |
| Root macro | `spanning-tree vlan <vlan-id> root primary` and `root secondary` calculate priority values automatically |
| Explicit priority | `spanning-tree vlan <vlan-id> priority <value>` gives deterministic manual control |
| Diameter keyword | The optional `diameter` keyword can tune STP timers from the root, but it should not be used casually |
| Root placement first | Root placement should be fixed before tuning path cost, port priority, or troubleshooting blocked ports |
| Access switch risk | If an access switch becomes root, the forwarding tree can become inefficient or unstable |
| STP guard separation | Root Guard protects the root placement later, but root placement itself is controlled with bridge priority |

# STP_Root_Bridge_Placement_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the Layer 2 topology before changing root placement | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended design |
| 2 | Confirm interswitch links are trunking | SW1/SW2/SW3 | `show interfaces trunk` | Trunk links are up and carrying the target VLANs |
| 3 | Confirm target VLANs exist on all participating switches | SW1/SW2/SW3 | `show vlan brief` | VLANs 10 and 20 exist and are active |
| 4 | Confirm current STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before changes |
| 5 | Record current root bridge state for all target VLANs | SW1/SW2/SW3 | `show spanning-tree root` | Current root bridge, cost, and root port information is documented |
| 6 | Record detailed STP state for VLAN 10 | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Current root, bridge ID, port roles, and port states are known |
| 7 | Record detailed STP state for VLAN 20 | SW1/SW2/SW3 | `show spanning-tree vlan 20` | Current root, bridge ID, port roles, and port states are known |
| 8 | Enter global configuration mode on the intended primary root | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 9 | Set STP mode if the lab requires classic PVST behavior | SW1 | `spanning-tree mode pvst` | SW1 runs PVST for per-VLAN STP |
| 10 | Ensure VLAN 10 exists | SW1 | `vlan 10` | VLAN 10 exists locally |
| 11 | Name VLAN 10 | SW1 | `name USERS` | VLAN 10 has a readable name |
| 12 | Ensure VLAN 20 exists | SW1 | `vlan 20` | VLAN 20 exists locally |
| 13 | Name VLAN 20 | SW1 | `name SERVERS` | VLAN 20 has a readable name |
| 14 | Configure SW1 as primary root for VLAN 10 | SW1 | `spanning-tree vlan 10 root primary` | SW1 priority is lowered so it should become root for VLAN 10 |
| 15 | Configure SW1 as primary root for VLAN 20 if using a single-root design | SW1 | `spanning-tree vlan 20 root primary` | SW1 priority is lowered so it should become root for VLAN 20 |
| 16 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 17 | Enter global configuration mode on the intended secondary root | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 18 | Set STP mode if the lab requires classic PVST behavior | SW2 | `spanning-tree mode pvst` | SW2 runs PVST for per-VLAN STP |
| 19 | Ensure VLAN 10 exists | SW2 | `vlan 10` | VLAN 10 exists locally |
| 20 | Name VLAN 10 | SW2 | `name USERS` | VLAN 10 has a readable name |
| 21 | Ensure VLAN 20 exists | SW2 | `vlan 20` | VLAN 20 exists locally |
| 22 | Name VLAN 20 | SW2 | `name SERVERS` | VLAN 20 has a readable name |
| 23 | Configure SW2 as secondary root for VLAN 10 | SW2 | `spanning-tree vlan 10 root secondary` | SW2 priority is set as backup root for VLAN 10 |
| 24 | Configure SW2 as secondary root for VLAN 20 if using a single-root design | SW2 | `spanning-tree vlan 20 root secondary` | SW2 priority is set as backup root for VLAN 20 |
| 25 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 26 | Enter global configuration mode on the access switch | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 27 | Set STP mode if the lab requires classic PVST behavior | SW3 | `spanning-tree mode pvst` | SW3 runs PVST for per-VLAN STP |
| 28 | Ensure VLAN 10 exists | SW3 | `vlan 10` | VLAN 10 exists locally |
| 29 | Ensure VLAN 20 exists | SW3 | `vlan 20` | VLAN 20 exists locally |
| 30 | Keep access switch at default STP priority unless design requires otherwise | SW3 | `end` | Access switch is not made root |
| 31 | Verify SW1 is root for VLAN 10 | SW1 | `show spanning-tree vlan 10` | Output shows `This bridge is the root` |
| 32 | Verify SW1 is root for VLAN 20 | SW1 | `show spanning-tree vlan 20` | Output shows `This bridge is the root` |
| 33 | Verify SW2 sees SW1 as root for VLAN 10 | SW2 | `show spanning-tree vlan 10` | Root ID points to SW1 and SW2 has one root port |
| 34 | Verify SW2 sees SW1 as root for VLAN 20 | SW2 | `show spanning-tree vlan 20` | Root ID points to SW1 and SW2 has one root port |
| 35 | Verify SW3 selects a root port toward the intended root path | SW3 | `show spanning-tree vlan 10` | One interface shows `Root FWD` toward the best path to SW1 |
| 36 | Verify the root summary across VLANs | SW1/SW2/SW3 | `show spanning-tree root` | VLANs 10 and 20 show SW1 as root |
| 37 | Verify no inconsistent STP ports exist | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 38 | Verify host traffic still forwards after root placement | HOSTS | `ping <same-vlan-remote-host>` | Same-VLAN reachability still works |
| 39 | Save the configuration | SW1/SW2/SW3 | `write memory` | STP root placement survives reload |

# STP_Root_Bridge_Placement_Skeleton

Single-root design:

SW1 primary root for VLANs 10 and 20:

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
spanning-tree vlan 20 root primary
!
end
write memory

SW2 secondary root for VLANs 10 and 20:

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
spanning-tree vlan 20 root secondary
!
end
write memory

SW3 access switch:

configure terminal
!
spanning-tree mode pvst
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
end
write memory

Manual priority method:

SW1 primary root:

configure terminal
!
spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 4096
!
end
write memory

SW2 secondary root:

configure terminal
!
spanning-tree vlan 10 priority 8192
spanning-tree vlan 20 priority 8192
!
end
write memory

Optional VLAN load-sharing design:

SW1 primary for VLAN 10 and secondary for VLAN 20:

configure terminal
!
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root secondary
!
end
write memory

SW2 primary for VLAN 20 and secondary for VLAN 10:

configure terminal
!
spanning-tree vlan 20 root primary
spanning-tree vlan 10 root secondary
!
end
write memory

Optional root macro with diameter:

configure terminal
!
spanning-tree vlan 10 root primary diameter 4
!
end
write memory

Use the diameter option only when the Layer 2 diameter is known and timer tuning is actually intended.

# STP_Root_Bridge_Placement_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | Switches run the intended STP mode |
| Confirm VLANs exist | SW1/SW2/SW3 | `show vlan brief` | Target VLANs exist and are active |
| Confirm trunks carry VLANs | SW1/SW2/SW3 | `show interfaces trunk` | Target VLANs are allowed, active, and forwarding where expected |
| View root bridge summary | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge placement matches the design |
| Confirm SW1 is root for VLAN 10 | SW1 | `show spanning-tree vlan 10` | Output shows `This bridge is the root` |
| Confirm SW1 is root for VLAN 20 | SW1 | `show spanning-tree vlan 20` | Output shows `This bridge is the root` |
| Confirm bridge priority on primary root | SW1 | `show spanning-tree vlan 10 | include Bridge ID|priority|This bridge` | SW1 has the lowest bridge priority for VLAN 10 |
| Confirm bridge priority on secondary root | SW2 | `show spanning-tree vlan 10 | include Root ID|Bridge ID|priority` | SW2 has backup priority and sees SW1 as root |
| Confirm access switch does not become root | SW3 | `show spanning-tree vlan 10 | include Root ID|Bridge ID|This bridge` | SW3 does not show `This bridge is the root` |
| Confirm root port selection | SW2/SW3 | `show spanning-tree vlan 10` | Non-root switches have one root port |
| Confirm root bridge ports are designated | SW1 | `show spanning-tree vlan 10` | Participating SW1 ports show `Desg FWD` |
| Confirm redundant paths are blocked correctly | SW2/SW3 | `show spanning-tree vlan 10` | Redundant links have expected forwarding or blocking states |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include SPANTREE|MACFLAP|LOOP` | No STP loop or MAC flapping messages appear |
| Confirm MAC learning is stable | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MAC addresses are not bouncing between ports |
| Confirm host reachability | HOSTS | `ping <same-vlan-remote-host>` | Same-VLAN traffic works after STP reconverges |
| Confirm configured STP priority | SW1/SW2 | `show running-config | include spanning-tree vlan` | Root primary, root secondary, or explicit priority commands are present |
| Confirm saved STP priority | SW1/SW2 | `show startup-config | include spanning-tree vlan` | Startup configuration includes intended STP root placement |

# STP_Root_Bridge_Placement_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current root placement before rollback | SW1/SW2/SW3 | `show spanning-tree root` | Current root placement is documented |
| 2 | Enter configuration mode on the primary root | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 3 | Remove explicit priority for VLAN 10 if configured | SW1 | `no spanning-tree vlan 10 priority` | VLAN 10 priority returns to default behavior |
| 4 | Remove explicit priority for VLAN 20 if configured | SW1 | `no spanning-tree vlan 20 priority` | VLAN 20 priority returns to default behavior |
| 5 | If the platform does not accept removal, reset VLAN 10 priority manually | SW1 | `spanning-tree vlan 10 priority 32768` | VLAN 10 base priority returns to default |
| 6 | If the platform does not accept removal, reset VLAN 20 priority manually | SW1 | `spanning-tree vlan 20 priority 32768` | VLAN 20 base priority returns to default |
| 7 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 8 | Enter configuration mode on the secondary root | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 9 | Remove explicit priority for VLAN 10 if configured | SW2 | `no spanning-tree vlan 10 priority` | VLAN 10 priority returns to default behavior |
| 10 | Remove explicit priority for VLAN 20 if configured | SW2 | `no spanning-tree vlan 20 priority` | VLAN 20 priority returns to default behavior |
| 11 | If the platform does not accept removal, reset VLAN 10 priority manually | SW2 | `spanning-tree vlan 10 priority 32768` | VLAN 10 base priority returns to default |
| 12 | If the platform does not accept removal, reset VLAN 20 priority manually | SW2 | `spanning-tree vlan 20 priority 32768` | VLAN 20 base priority returns to default |
| 13 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 14 | Save rollback | SW1/SW2 | `write memory` | Startup configuration reflects rollback |
| 15 | Verify root election after rollback | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge is recalculated based on remaining bridge IDs |
| 16 | Verify detailed STP state after rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root, bridge ID, and port roles reflect rollback state |

# STP_Root_Bridge_Placement_Failure_Checks

| Symptom                                                         | Likely Cause                                                                | Device      | Command                                                  | Fix                                                                          |                                               |                                                                  |
| --------------------------------------------------------------- | --------------------------------------------------------------------------- | ----------- | -------------------------------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------- | ---------------------------------------------------------------- |
| Wrong switch becomes root                                       | Another switch has a lower bridge ID                                        | SW1/SW2/SW3 | `show spanning-tree vlan 10`                             | Lower the intended root priority or raise the unintended root priority       |                                               |                                                                  |
| Access switch becomes root                                      | Default priority left in place and access switch has the lowest MAC address | SW1/SW2/SW3 | `show spanning-tree root`                                | Configure root primary and secondary on distribution or core switches        |                                               |                                                                  |
| Secondary root does not take over                               | Backup switch priority is not lower than default switches                   | SW2         | `show spanning-tree vlan 10`                             | Configure `spanning-tree vlan 10 root secondary` or explicit backup priority |                                               |                                                                  |
| Root placement differs by VLAN                                  | Root configured for one VLAN but not another                                | SW1/SW2/SW3 | `show spanning-tree root`                                | Configure root placement for every required VLAN                             |                                               |                                                                  |
| VLAN missing from STP output                                    | VLAN does not exist or is inactive                                          | SW1/SW2/SW3 | `show vlan brief`                                        | Create the VLAN and ensure it is active                                      |                                               |                                                                  |
| VLAN exists but does not cross the topology                     | VLAN is not allowed on trunk links                                          | SW1/SW2/SW3 | `show interfaces trunk`                                  | Add the VLAN to the trunk allowed list                                       |                                               |                                                                  |
| Root macro does not set expected priority                       | Another bridge already has an equal or better priority                      | SW1         | `show spanning-tree vlan 10`                             | Use explicit `spanning-tree vlan 10 priority <value>`                        |                                               |                                                                  |
| Root priority looks like 24586 instead of 24576                 | VLAN system ID extension is added to the base priority                      | SW1         | `show spanning-tree vlan 10`                             | Treat this as normal; compare bridge IDs correctly                           |                                               |                                                                  |
| Root changes after adding a switch                              | New switch has a better bridge ID                                           | SW1/SW2/SW3 | `show spanning-tree root`                                | Preconfigure new switches with safe priority before attaching them           |                                               |                                                                  |
| Traffic takes an inefficient path                               | Root bridge is placed too far from the aggregation point                    | SW1/SW2/SW3 | `show spanning-tree vlan 10`                             | Move root to the correct core or distribution switch                         |                                               |                                                                  |
| Root bridge placement is correct but blocked port is unexpected | Path cost or port priority favors a different link                          | SW2/SW3     | `show spanning-tree vlan 10`                             | Tune STP cost or port priority only after confirming root placement          |                                               |                                                                  |
| STP blocks a needed trunk                                       | Redundant topology requires one blocked path                                | SW1/SW2/SW3 | `show spanning-tree vlan 10`                             | Verify whether the block is correct before changing cost or root placement   |                                               |                                                                  |
| MAC addresses flap                                              | Layer 2 loop or unstable STP topology                                       | SW1/SW2/SW3 | `show logging                                            | include MACFLAP                                                              | SPANTREE`                                     | Confirm one root, expected blocked ports, and no unsafe bridging |
| Host reachability fails after root change                       | STP reconvergence, trunk issue, or VLAN missing on path                     | SW1/SW2/SW3 | `show spanning-tree vlan 10` and `show interfaces trunk` | Verify root, port roles, forwarding state, and VLAN trunking                 |                                               |                                                                  |
| Root bridge changes after reload                                | Configuration was not saved                                                 | SW1/SW2     | `show startup-config                                     | include spanning-tree vlan`                                                  | Reapply root placement and run `write memory` |                                                                  |
| Manual priority rejected                                        | Priority value is not an increment of 4096                                  | SW1/SW2     | `spanning-tree vlan 10 priority <value>`                 | Use 0, 4096, 8192, 12288, up to 61440                                        |                                               |                                                                  |
| Diameter keyword causes unexpected timer values                 | Root macro changed timers from the root bridge                              | SW1         | `show spanning-tree vlan 10`                             | Avoid diameter unless timer tuning is intended                               |                                               |                                                                  |
| Load-sharing design sends VLANs to wrong root                   | Primary and secondary roles were not configured per VLAN                    | SW1/SW2     | `show spanning-tree root`                                | Correct per-VLAN root primary and secondary assignments                      |                                               |                                                                  |