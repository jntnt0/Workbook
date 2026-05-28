STP_Reconvergence_Observation.md
# STP_Reconvergence_Observation

# STP_Reconvergence_Observation_Index

- STP_Reconvergence_Observation_Mental_Model
- STP_Reconvergence_Observation_Configuration_Checklist
- STP_Reconvergence_Observation_Skeleton
- STP_Reconvergence_Observation_Verification_Commands
- STP_Reconvergence_Observation_Rollback
- STP_Reconvergence_Observation_Failure_Checks

# STP_Reconvergence_Observation_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Reconvergence | The process where STP recalculates port roles and states after a topology change |
| Observation note | This note is about watching STP behavior, not changing the root design unless needed for a controlled test |
| Baseline first | Capture root bridge, root ports, designated ports, blocked ports, timers, and MAC table state before creating a failure |
| Controlled failure | Shut one known trunk or access link so the change is intentional and reversible |
| Direct failure | A directly connected failed forwarding port is detected quickly because link state drops |
| Indirect failure | A failure learned through missing BPDUs can take longer because STP waits on timer expiration |
| 802.1D state movement | Classic STP moves through blocking, listening, learning, then forwarding |
| Forward delay | Default forward delay is 15 seconds for listening and 15 seconds for learning |
| Max age | Default max age is 20 seconds and matters when BPDU information must age out |
| Topology change notification | STP uses TCN behavior to tell the topology that MAC tables should age out faster after a change |
| MAC table impact | Reconvergence is not just port role change; MAC entries can flush and relearn |
| Root port change | A non-root switch may select a new root port when its current best path fails |
| Blocked port movement | A blocked redundant port may move toward forwarding after the active path fails |
| Host impact | Packet loss during convergence is measured with continuous ping or traffic generation |
| Per-VLAN behavior | In PVST, reconvergence must be checked per VLAN because each VLAN has its own tree |
| Debug caution | STP debugging is useful in labs, but it can be noisy and should be disabled after observation |
| Final check | After restoring the link, verify the topology returns to the expected root, role, and forwarding state |

# STP_Reconvergence_Observation_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm physical switch adjacency | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended Layer 2 topology |
| 2 | Confirm interswitch links are connected | SW1/SW2/SW3 | `show interfaces status` | Trunk or interswitch ports show `connected` |
| 3 | Confirm trunk operation before testing STP recovery | SW1/SW2/SW3 | `show interfaces trunk` | Required interswitch ports show `trunking` |
| 4 | Confirm the target VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| 5 | Confirm current STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before observation |
| 6 | Confirm root bridge before inducing failure | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge and root port information is documented |
| 7 | Confirm detailed VLAN 10 STP baseline | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root, bridge ID, port roles, port states, costs, and timers are known |
| 8 | Confirm interface-level STP baseline on the primary uplink | SW3 | `show spanning-tree vlan 10 interface GigabitEthernet0/1 detail` | Role, state, timers, link type, and BPDU counters are visible |
| 9 | Confirm interface-level STP baseline on the backup uplink | SW3 | `show spanning-tree vlan 10 interface GigabitEthernet0/2 detail` | Backup or alternate path state is visible |
| 10 | Confirm MAC learning before failure | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | Host MAC addresses are learned on expected ports |
| 11 | Confirm no current inconsistent STP ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 12 | Confirm no loop symptoms before testing | SW1/SW2/SW3 | `show logging | include SPANTREE|MACFLAP|LOOP` | No active STP loop or MAC flap messages appear |
| 13 | Start continuous reachability test from a host | H1 | `ping <remote-same-vlan-host-ip> repeat 1000` | Ping begins and establishes baseline reachability |
| 14 | Optionally enable terminal logging for live observation | SW3 | `terminal monitor` | Syslog/debug output can display in the terminal session |
| 15 | Optionally enable STP event debugging in a lab only | SW3 | `debug spanning-tree events` | STP state changes display live |
| 16 | Record pre-failure STP timestamp | SW3 | `show clock` | Time is recorded for comparison with logs and ping loss |
| 17 | Enter configuration mode on the switch where the active path will be failed | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 18 | Select the active root-facing uplink | SW3 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 19 | Shut the active uplink to create a controlled direct failure | SW3 | `shutdown` | Active STP path is removed from the topology |
| 20 | Exit to privileged EXEC quickly | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 21 | Immediately observe VLAN 10 STP recalculation | SW3 | `show spanning-tree vlan 10` | Root port, designated port, or blocked port role changes are visible |
| 22 | Observe interface detail after failure | SW3 | `show spanning-tree vlan 10 interface GigabitEthernet0/2 detail` | Backup path should move toward forwarding if it is the valid alternate |
| 23 | Check topology change information | SW3 | `show spanning-tree vlan 10 detail | include ieee|occurr|from|change` | Topology change count and source information are visible |
| 24 | Check MAC learning during or after convergence | SW3 | `show mac address-table dynamic vlan 10` | MAC entries may flush and relearn on the new path |
| 25 | Check host traffic impact | H1 | Review ping output | Packet loss indicates the data-plane interruption window |
| 26 | Confirm new forwarding path after convergence | SW1/SW2/SW3 | `show spanning-tree vlan 10` | New stable root port, designated ports, and blocked ports are visible |
| 27 | Confirm trunks still carry VLAN 10 after failure | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 10 remains allowed and forwarding on the surviving path |
| 28 | Restore the failed uplink | SW3 | `configure terminal` then `interface GigabitEthernet0/1` then `no shutdown` | Original link comes back up |
| 29 | Exit configuration mode after restoration | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 30 | Observe the restored link moving through STP states | SW3 | `show spanning-tree vlan 10 interface GigabitEthernet0/1 detail` | Port state changes are visible during reconvergence |
| 31 | Confirm final root and port roles after restoration | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Topology returns to the expected stable state |
| 32 | Confirm final root summary | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge and root ports match the intended design |
| 33 | Confirm final MAC learning | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MAC addresses are learned on expected final ports |
| 34 | Confirm host reachability after restoration | H1 | `ping <remote-same-vlan-host-ip>` | Same-VLAN reachability works after the link is restored |
| 35 | Disable STP debugging if enabled | SW3 | `undebug all` | Debug output stops |
| 36 | Disable terminal monitoring if no longer needed | SW3 | `terminal no monitor` | Live terminal syslog display is disabled |
| 37 | Save only if intentional config changes were made | SW1/SW2/SW3 | `write memory` | Only intended persistent settings are saved |

# STP_Reconvergence_Observation_Skeleton

Baseline observation:

show cdp neighbors
show interfaces status
show interfaces trunk
show vlan brief
show spanning-tree summary
show spanning-tree root
show spanning-tree vlan 10
show spanning-tree vlan 10 detail
show spanning-tree vlan 10 interface GigabitEthernet0/1 detail
show spanning-tree vlan 10 interface GigabitEthernet0/2 detail
show mac address-table dynamic vlan 10
show spanning-tree inconsistentports
show logging | include SPANTREE|MACFLAP|LOOP

Optional lab-only live observation:

terminal monitor
debug spanning-tree events
show clock

Controlled direct failure test:

configure terminal
interface GigabitEthernet0/1
 shutdown
end

Immediate observation after failure:

show clock
show spanning-tree vlan 10
show spanning-tree vlan 10 detail | include ieee|occurr|from|change
show spanning-tree vlan 10 interface GigabitEthernet0/2 detail
show mac address-table dynamic vlan 10
show interfaces trunk

Restore failed link:

configure terminal
interface GigabitEthernet0/1
 no shutdown
end

Observation after restoration:

show clock
show spanning-tree vlan 10
show spanning-tree vlan 10 interface GigabitEthernet0/1 detail
show spanning-tree root
show mac address-table dynamic vlan 10
show logging | include SPANTREE|MACFLAP|LOOP

Stop debug:

undebug all
terminal no monitor

Optional traffic test from Cisco host or router:

ping <remote-same-vlan-host-ip> repeat 1000

Optional traffic test from Linux host:

ping <remote-same-vlan-host-ip>

# STP_Reconvergence_Observation_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm current STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before observing convergence |
| Confirm current root bridge | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge is known for the target VLAN |
| Confirm detailed VLAN STP state | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root ID, bridge ID, port roles, port states, costs, and timers are visible |
| Confirm topology change counters | SW1/SW2/SW3 | `show spanning-tree vlan 10 detail | include occurr|from|change` | Topology change information is visible |
| Confirm forwarding and blocked ports | SW1/SW2/SW3 | `show spanning-tree vlan 10` | One loop-free forwarding topology exists |
| Confirm interface STP detail | SW3 | `show spanning-tree vlan 10 interface GigabitEthernet0/1 detail` | Role, state, timers, and BPDU counters are visible |
| Confirm backup link STP detail | SW3 | `show spanning-tree vlan 10 interface GigabitEthernet0/2 detail` | Backup path state is visible before and after failure |
| Confirm trunks are carrying the VLAN | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 10 is allowed, active, and forwarding where expected |
| Confirm VLAN exists locally | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| Confirm MAC table before and after convergence | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MAC entries move or relearn according to the active path |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no loop or MAC flap symptoms | SW1/SW2/SW3 | `show logging | include SPANTREE|MACFLAP|LOOP` | No active loop or MAC flap messages appear |
| Confirm active interface state | SW3 | `show interfaces GigabitEthernet0/1 status` | Failed or restored interface state is clear |
| Confirm line protocol state | SW3 | `show interfaces GigabitEthernet0/1` | Link state matches the test condition |
| Confirm host packet loss during test | H1 | `ping <remote-same-vlan-host-ip> repeat 1000` | Loss corresponds to the convergence window |
| Confirm restored reachability | H1 | `ping <remote-same-vlan-host-ip>` | Ping succeeds after reconvergence |
| Confirm debugging is disabled | SW3 | `show debugging` | No STP debug remains enabled |
| Confirm final stable topology | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Final root, roles, and states match expected design |

# STP_Reconvergence_Observation_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Disable all debugging | SW1/SW2/SW3 | `undebug all` | Debug output stops |
| 2 | Disable terminal monitoring if enabled | SW1/SW2/SW3 | `terminal no monitor` | Terminal no longer displays live syslog/debug output |
| 3 | Restore the test-shutdown uplink | SW3 | `configure terminal` then `interface GigabitEthernet0/1` then `no shutdown` | Uplink is administratively enabled |
| 4 | Exit configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 5 | Verify restored interface state | SW3 | `show interfaces status` | Restored interface shows `connected` if the far end is up |
| 6 | Verify trunk state after restoration | SW1/SW2/SW3 | `show interfaces trunk` | Restored trunk returns to trunking if configured as trunk |
| 7 | Verify final STP state | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Topology returns to intended stable roles and states |
| 8 | Verify no inconsistent ports remain | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 9 | Verify final host reachability | H1 | `ping <remote-same-vlan-host-ip>` | Ping succeeds |
| 10 | Avoid saving temporary shutdown state | SW3 | `show running-config interface GigabitEthernet0/1` | Interface does not contain unwanted `shutdown` |
| 11 | Save only intentional final configuration | SW1/SW2/SW3 | `write memory` | Startup config reflects only desired settings |

# STP_Reconvergence_Observation_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Backup path never moves to forwarding | Backup path is not actually in the VLAN or not allowed on trunk | SW1/SW2/SW3 | `show interfaces trunk` | Allow VLAN 10 on the backup trunk and verify it is active |
| Backup path does not exist in STP output | VLAN missing on one or more switches | SW1/SW2/SW3 | `show vlan brief` | Create VLAN 10 on every participating switch |
| No topology change is observed | Wrong VLAN or wrong interface was tested | SW3 | `show spanning-tree vlan 10` | Identify the active root port or forwarding trunk before shutting a link |
| Ping does not drop during test | Traffic was not using the failed path | SW3 | `show spanning-tree vlan 10` | Verify the actual forwarding path before testing |
| Ping never recovers | STP did not find a valid alternate forwarding path | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Restore trunking, VLAN allowance, and redundant physical path |
| Port remains blocking after failure | STP still sees another better path or port is blocked by a protection feature | SW3 | `show spanning-tree vlan 10` | Verify root path cost, root placement, and protection states |
| Port is inconsistent | Root Guard, Loop Guard, Bridge Assurance, or dispute condition exists | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | Fix the specific protection trigger before expecting forwarding |
| Port is err-disabled | BPDU Guard, UDLD, port security, or link-flap protection disabled the port | SW1/SW2/SW3 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the port |
| STP debug is too noisy | Debug was enabled on too broad a lab or too many events are occurring | SW1/SW2/SW3 | `show debugging` | Use `undebug all` and narrow observation to show commands |
| MAC addresses flap during reconvergence | Layer 2 loop or unstable STP state | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | Verify one loop-free STP topology and remove unsafe redundant forwarding |
| Topology change count keeps increasing | Flapping trunk, unstable access port, or misclassified edge port | SW1/SW2/SW3 | `show spanning-tree vlan 10 detail | include occurr|from|change` | Identify the source interface and fix the flap |
| Host outage is longer than expected | Classic 802.1D timers, indirect failure, or no rapid mechanism | SW1/SW2/SW3 | `show spanning-tree summary` and `show spanning-tree vlan 10` | Verify STP mode, timers, and whether the failure was direct or indirect |
| Restored link does not rejoin topology | Interface remains shut or trunk negotiation failed | SW3 | `show interfaces status` and `show interfaces trunk` | Apply `no shutdown` and correct trunk settings |
| VLAN forwards before MAC table is stable | MAC table was flushed and is relearning | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | Generate host traffic and confirm MACs relearn on the correct path |
| Root bridge changes unexpectedly after failure | Primary root failed or root priority design is weak | SW1/SW2/SW3 | `show spanning-tree root` | Configure intentional primary and secondary root placement |
| Convergence behavior differs by VLAN | PVST calculates each VLAN separately | SW1/SW2/SW3 | `show spanning-tree vlan <vlan-id>` | Troubleshoot each VLAN independently |
| Trunk shows VLAN allowed but not forwarding | STP is blocking or VLAN is not active | SW1/SW2/SW3 | `show interfaces trunk` | Check STP state and VLAN database |
| Native VLAN mismatch appears during test | Trunk native VLANs differ | SW1/SW2/SW3 | `show logging | include NATIVE|native` | Match native VLAN on both sides |
| Debug remains on after the lab | `undebug all` was not run | SW1/SW2/SW3 | `show debugging` | Run `undebug all` |
| Temporary shutdown is saved by mistake | `write memory` was run while the test link was shut | SW3 | `show startup-config interface GigabitEthernet0/1` | Remove `shutdown`, restore interface, and save again |
