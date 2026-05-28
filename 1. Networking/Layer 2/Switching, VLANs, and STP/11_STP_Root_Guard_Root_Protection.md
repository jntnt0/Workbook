STP_Root_Guard_Root_Protection.md
# STP_Root_Guard_Root_Protection

# STP_Root_Guard_Root_Protection_Index

- STP_Root_Guard_Root_Protection_Mental_Model
- STP_Root_Guard_Root_Protection_Configuration_Checklist
- STP_Root_Guard_Root_Protection_Skeleton
- STP_Root_Guard_Root_Protection_Verification_Commands
- STP_Root_Guard_Root_Protection_Rollback
- STP_Root_Guard_Root_Protection_Failure_Checks

# STP_Root_Guard_Root_Protection_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Root Guard | STP protection feature that prevents a protected port from becoming a root port |
| Root bridge protection | Root Guard enforces the intended root bridge placement by rejecting superior BPDUs on protected ports |
| Superior BPDU | A BPDU advertising a better root bridge ID than the current root information |
| Designated-port protection | Root Guard is applied on ports that should remain designated ports toward downstream switches |
| Root-inconsistent state | If a Root Guard port receives superior BPDUs, STP places the port into root-inconsistent blocking state |
| Automatic recovery | When superior BPDUs stop, the port automatically leaves root-inconsistent and reconverges |
| No manual shut/no shut normally required | Root Guard recovery happens through STP once the bad BPDU condition clears |
| Downstream switch boundary | Use Root Guard on ports facing access switches, customer switches, lab switches, or unmanaged downstream Layer 2 devices that must not become root |
| Not for root ports | Do not place Root Guard on a port that is supposed to become a root port |
| Not for alternate ports | Do not place Root Guard on ports where Loop Guard or alternate-port behavior is the protection goal |
| Not BPDU Guard | BPDU Guard disables a PortFast edge port when any BPDU arrives; Root Guard blocks a switch-facing port only when superior BPDUs arrive |
| Not Loop Guard | Loop Guard protects against missing BPDUs on root or alternate ports; Root Guard protects against superior BPDUs on designated ports |
| Root placement first | Configure the intended primary and secondary root before applying Root Guard |
| Per-interface feature | Root Guard is enabled under the interface with `spanning-tree guard root` |
| VLAN impact | On a trunk, Root Guard can affect the VLAN instances where superior BPDUs are received |
| Verification rule | Confirm root placement, protected interface state, inconsistent ports, and logs after testing |

# STP_Root_Guard_Root_Protection_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the physical Layer 2 topology | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended design |
| 2 | Confirm interswitch links are physically up | SW1/SW2/SW3 | `show interfaces status` | Switch-facing ports show `connected` |
| 3 | Confirm trunk links are working if the protected link is a trunk | SW1/SW2/SW3 | `show interfaces trunk` | Required interswitch links show `trunking` |
| 4 | Confirm the target VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| 5 | Confirm current STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before protection is applied |
| 6 | Confirm current root bridge for VLAN 10 | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Intended root bridge is known |
| 7 | Configure the intended primary root before applying Root Guard | SW1 | `configure terminal` then `spanning-tree vlan 10 root primary` then `end` | SW1 becomes preferred root for VLAN 10 |
| 8 | Configure the intended secondary root | SW2 | `configure terminal` then `spanning-tree vlan 10 root secondary` then `end` | SW2 becomes backup root for VLAN 10 |
| 9 | Verify SW1 is the root for VLAN 10 | SW1 | `show spanning-tree vlan 10` | Output shows `This bridge is the root` |
| 10 | Verify SW2 sees SW1 as root | SW2 | `show spanning-tree vlan 10` | Root ID points to SW1 |
| 11 | Identify the downstream-facing port to protect | SW2 | `show cdp neighbors` | Interface facing SW3 or downstream switch is identified |
| 12 | Confirm the downstream-facing port is currently designated, not root | SW2 | `show spanning-tree vlan 10 interface GigabitEthernet0/2 detail` | Port is designated or expected to remain downstream-facing |
| 13 | Confirm Loop Guard is not configured on the same interface | SW2 | `show running-config interface GigabitEthernet0/2` | Interface does not contain conflicting Loop Guard configuration |
| 14 | Enter interface configuration mode on the protected port | SW2 | `configure terminal` then `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 15 | Add a useful interface description | SW2 | `description DOWNSTREAM_TO_SW3_ROOT_GUARD` | Interface purpose is clear in show commands |
| 16 | Ensure the port is a trunk if carrying VLANs downstream | SW2 | `switchport mode trunk` | Interface is administratively trunking |
| 17 | Permit the target VLAN on the protected trunk | SW2 | `switchport trunk allowed vlan add 10` | VLAN 10 is allowed without overwriting the existing allowed list |
| 18 | Enable Root Guard on the downstream-facing port | SW2 | `spanning-tree guard root` | Root Guard is enabled on the interface |
| 19 | Bring up the protected port | SW2 | `no shutdown` | Interface is administratively up |
| 20 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 21 | Verify Root Guard is enabled on the protected interface | SW2 | `show spanning-tree interface GigabitEthernet0/2 detail` | Output shows Root Guard is enabled |
| 22 | Verify no inconsistent ports exist before the test | SW2 | `show spanning-tree inconsistentports` | No ports are listed |
| 23 | Verify normal forwarding before the test | SW2 | `show spanning-tree vlan 10` | Protected port is forwarding if no superior BPDU is received |
| 24 | Start continuous host reachability test if endpoints exist | H1 | `ping <remote-same-vlan-host-ip> repeat 1000000` | Traffic establishes baseline reachability |
| 25 | Simulate a rogue or downstream switch trying to become root | SW3 | `configure terminal` then `spanning-tree vlan 10 priority 0` then `end` | SW3 advertises superior BPDUs for VLAN 10 |
| 26 | Verify Root Guard blocks the protected port | SW2 | `show spanning-tree inconsistentports` | Protected interface appears as root-inconsistent |
| 27 | Verify VLAN 10 state on the protected port | SW2 | `show spanning-tree vlan 10` | Protected port shows root-inconsistent or blocking behavior |
| 28 | Check Root Guard log messages | SW2 | `show logging | include ROOTGUARD|Root guard|root-inconsistent|SPANTREE` | Logs show Root Guard blocking the interface or VLAN |
| 29 | Verify the intended root was not replaced by the rogue switch | SW1/SW2 | `show spanning-tree vlan 10` | SW1 remains the root bridge for VLAN 10 |
| 30 | Remove the rogue superior BPDU condition | SW3 | `configure terminal` then `spanning-tree vlan 10 priority 32768` then `end` | SW3 returns to default priority and stops trying to become root |
| 31 | Verify automatic Root Guard recovery | SW2 | `show spanning-tree inconsistentports` | Protected interface disappears from inconsistent port list |
| 32 | Verify the protected port returns to normal STP state | SW2 | `show spanning-tree vlan 10` | Port reconverges to forwarding if the topology allows it |
| 33 | Verify host reachability after recovery | H1 | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works after recovery |
| 34 | Verify no loop or MAC flap symptoms exist | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | No active loop or MAC flap messages appear |
| 35 | Save the intended configuration | SW1/SW2/SW3 | `write memory` | Root placement and Root Guard configuration survive reload |

# STP_Root_Guard_Root_Protection_Skeleton

Primary root switch:

configure terminal
!
spanning-tree vlan 10 root primary
!
end
write memory

Secondary root or upstream distribution switch with Root Guard toward downstream switch:

configure terminal
!
spanning-tree vlan 10 root secondary
!
interface GigabitEthernet0/2
 description DOWNSTREAM_TO_SW3_ROOT_GUARD
 switchport mode trunk
 switchport trunk allowed vlan add 10
 spanning-tree guard root
 no shutdown
!
end
write memory

Downstream switch baseline:

configure terminal
!
vlan 10
 name USERS
!
end
write memory

Rogue-root test from downstream switch:

configure terminal
!
spanning-tree vlan 10 priority 0
!
end

Verify Root Guard reaction on protected upstream switch:

show spanning-tree inconsistentports
show spanning-tree vlan 10
show spanning-tree interface GigabitEthernet0/2 detail
show logging | include ROOTGUARD|Root guard|root-inconsistent|SPANTREE

Clear rogue-root test condition:

configure terminal
!
spanning-tree vlan 10 priority 32768
!
end

Verify automatic recovery:

show spanning-tree inconsistentports
show spanning-tree vlan 10
show spanning-tree interface GigabitEthernet0/2 detail

Remove Root Guard from an interface:

configure terminal
!
interface GigabitEthernet0/2
 no spanning-tree guard root
!
end
write memory

# STP_Root_Guard_Root_Protection_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known and consistent with the lab |
| Confirm VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| Confirm trunk operation | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 10 is allowed and active on required trunks |
| Confirm intended root bridge | SW1 | `show spanning-tree vlan 10` | SW1 shows `This bridge is the root` |
| Confirm downstream switch is not root | SW3 | `show spanning-tree vlan 10` | SW3 does not show itself as root during normal operation |
| Confirm protected interface role before test | SW2 | `show spanning-tree vlan 10 interface GigabitEthernet0/2 detail` | Port is designated or expected downstream-facing |
| Confirm Root Guard is configured | SW2 | `show spanning-tree interface GigabitEthernet0/2 detail` | Output shows Root Guard is enabled |
| Confirm interface running config | SW2 | `show running-config interface GigabitEthernet0/2` | Interface contains `spanning-tree guard root` |
| Confirm no inconsistent ports during normal operation | SW2 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm Root Guard block during rogue-root test | SW2 | `show spanning-tree inconsistentports` | Protected port appears as root-inconsistent |
| Confirm VLAN-specific blocked state | SW2 | `show spanning-tree vlan 10` | Protected port shows root-inconsistent or blocking behavior |
| Confirm Root Guard log event | SW2 | `show logging | include ROOTGUARD|Root guard|root-inconsistent|SPANTREE` | Log shows Root Guard blocking or unblocking event |
| Confirm intended root was protected | SW1/SW2 | `show spanning-tree vlan 10` | SW1 remains root for VLAN 10 |
| Confirm automatic recovery after superior BPDUs stop | SW2 | `show spanning-tree inconsistentports` | Protected port is no longer listed |
| Confirm recovered port state | SW2 | `show spanning-tree vlan 10 interface GigabitEthernet0/2 detail` | Port reconverges normally after the bad BPDU condition clears |
| Confirm no Loop Guard conflict | SW2 | `show running-config interface GigabitEthernet0/2` | Interface does not contain both Root Guard and Loop Guard |
| Confirm no err-disabled condition | SW2 | `show interfaces status err-disabled` | Protected port is not err-disabled |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | No active loop or MAC flap messages appear |
| Confirm host reachability after recovery | HOSTS | `ping <remote-same-vlan-host-ip>` | Traffic works after Root Guard recovery |
| Confirm saved Root Guard configuration | SW2 | `show startup-config interface GigabitEthernet0/2` | Startup config contains `spanning-tree guard root` if intentionally retained |

# STP_Root_Guard_Root_Protection_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Clear the rogue-root condition first | SW3 | `configure terminal` then `spanning-tree vlan 10 priority 32768` then `end` | SW3 stops advertising superior BPDUs |
| 2 | Verify Root Guard has recovered before removing it | SW2 | `show spanning-tree inconsistentports` | Protected port is no longer root-inconsistent |
| 3 | Enter interface configuration mode on the protected port | SW2 | `configure terminal` then `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 4 | Remove Root Guard from the interface | SW2 | `no spanning-tree guard root` | Root Guard is removed from the interface |
| 5 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 6 | Verify Root Guard is removed from the interface | SW2 | `show running-config interface GigabitEthernet0/2` | `spanning-tree guard root` is no longer present |
| 7 | Verify STP state after removing Root Guard | SW2 | `show spanning-tree vlan 10` | Port roles and states reflect normal STP calculation |
| 8 | Reset root priority on SW1 if the root-placement test is being fully removed | SW1 | `configure terminal` then `spanning-tree vlan 10 priority 32768` then `end` | SW1 returns to default base priority |
| 9 | Reset secondary root priority on SW2 if the root-placement test is being fully removed | SW2 | `configure terminal` then `spanning-tree vlan 10 priority 32768` then `end` | SW2 returns to default base priority |
| 10 | Verify final root bridge after rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root bridge is recalculated from remaining priorities |
| 11 | Verify no inconsistent ports remain | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 12 | Verify host reachability | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works |
| 13 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |

# STP_Root_Guard_Root_Protection_Failure_Checks

| Symptom                                                       | Likely Cause                                                                              | Device      | Command                                                                                         | Fix                                                                                      |           |                                                  |                                              |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | --------- | ------------------------------------------------ | -------------------------------------------- |
| Protected port goes root-inconsistent                         | Downstream switch is sending superior BPDUs                                               | SW2         | `show spanning-tree inconsistentports`                                                          | Correct downstream bridge priority or remove unauthorized switch                         |           |                                                  |                                              |
| Root Guard does not trigger during test                       | Downstream BPDU is not superior                                                           | SW2/SW3     | `show spanning-tree vlan 10`                                                                    | Lower downstream priority for the test or verify current root ID                         |           |                                                  |                                              |
| Root Guard does not trigger during test                       | Root Guard is configured on the wrong interface                                           | SW2         | `show running-config interface <interface>`                                                     | Configure `spanning-tree guard root` on the actual downstream-facing designated port     |           |                                                  |                                              |
| Root Guard does not trigger during test                       | Protected link is not carrying the VLAN                                                   | SW2         | `show interfaces trunk`                                                                         | Allow the VLAN on the trunk and verify it is active                                      |           |                                                  |                                              |
| Root Guard does not trigger during test                       | Interface is down or not trunking                                                         | SW2         | `show interfaces status` and `show interfaces trunk`                                            | Bring up the link and correct trunk mode                                                 |           |                                                  |                                              |
| Intended root changes anyway                                  | Root Guard was not placed on the boundary where superior BPDUs enter                      | SW1/SW2/SW3 | `show spanning-tree vlan 10`                                                                    | Apply Root Guard on downstream-facing designated ports at the boundary                   |           |                                                  |                                              |
| Port stays root-inconsistent after fixing downstream priority | Superior BPDUs are still arriving                                                         | SW2         | `show spanning-tree vlan 10` and `show logging                                                  | include ROOTGUARD                                                                        | SPANTREE` | Find and correct the remaining downstream source |                                              |
| Port stays root-inconsistent after condition clears           | STP has not reconverged yet                                                               | SW2         | `show spanning-tree interface GigabitEthernet0/2 detail`                                        | Wait for STP to reconverge and verify BPDUs stopped                                      |           |                                                  |                                              |
| Port is err-disabled instead of root-inconsistent             | BPDU Guard or another protection feature triggered                                        | SW2         | `show interfaces status err-disabled`                                                           | Fix the triggering feature and shut/no shut the port                                     |           |                                                  |                                              |
| Loop Guard command conflicts with Root Guard                  | Both protections are being applied to the same interface                                  | SW2         | `show running-config interface GigabitEthernet0/2`                                              | Use Root Guard on designated downstream ports and Loop Guard on root or alternate ports  |           |                                                  |                                              |
| Host port is blocked by Root Guard test                       | Root Guard was placed on a normal user edge port or a switch was connected to a user port | Access SW   | `show spanning-tree inconsistentports`                                                          | Use BPDU Guard for true host edge ports; use Root Guard for downstream switch boundaries |           |                                                  |                                              |
| Downstream access switch loses connectivity                   | Downstream switch is trying to become root                                                | SW2/SW3     | `show spanning-tree vlan 10`                                                                    | Restore correct downstream bridge priority                                               |           |                                                  |                                              |
| VLAN 10 is blocked but other VLANs work                       | Superior BPDUs are affecting only one VLAN instance                                       | SW2         | `show spanning-tree vlan 10` and `show spanning-tree vlan <other-vlan>`                         | Correct per-VLAN root priority on the downstream switch                                  |           |                                                  |                                              |
| Logs show Root Guard block but no current inconsistent port   | Bad BPDUs stopped and port recovered automatically                                        | SW2         | `show spanning-tree inconsistentports`                                                          | Treat as historical unless the event repeats                                             |           |                                                  |                                              |
| MAC addresses flap after removing Root Guard                  | Downstream switch became root or loop formed                                              | SW1/SW2/SW3 | `show logging                                                                                   | include MACFLAP                                                                          | LOOP      | SPANTREE`                                        | Reapply Root Guard or correct root placement |
| Wrong switch is root before Root Guard testing                | Root placement was never configured correctly                                             | SW1/SW2/SW3 | `show spanning-tree root`                                                                       | Configure primary and secondary root before testing Root Guard                           |           |                                                  |                                              |
| Root Guard configured but not visible in brief output         | Need interface detail or running config                                                   | SW2         | `show spanning-tree interface GigabitEthernet0/2 detail`                                        | Verify with interface detail or running config                                           |           |                                                  |                                              |
| Protected port does not recover automatically                 | Superior BPDUs continue or the link is physically down                                    | SW2         | `show interfaces status` and `show spanning-tree vlan 10`                                       | Fix the downstream root cause and verify link state                                      |           |                                                  |                                              |
| Traffic fails after recovery                                  | Trunk VLAN, STP state, or MAC relearning issue remains                                    | SW1/SW2/SW3 | `show interfaces trunk`, `show spanning-tree vlan 10`, `show mac address-table dynamic vlan 10` | Fix trunk allowance, STP state, or generate traffic to relearn MACs                      |           |                                                  |                                              |
| Config disappears after reload                                | Configuration was not saved                                                               | SW2         | `show startup-config interface GigabitEthernet0/2`                                              | Reapply Root Guard and run `write memory`                                                |           |                                                  |                                              |