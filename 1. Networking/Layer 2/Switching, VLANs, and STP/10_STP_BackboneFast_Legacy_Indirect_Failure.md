STP_BackboneFast_Legacy_Indirect_Failure.md
# STP_BackboneFast_Legacy_Indirect_Failure

# STP_BackboneFast_Legacy_Indirect_Failure_Index

- STP_BackboneFast_Legacy_Indirect_Failure_Mental_Model
- STP_BackboneFast_Legacy_Indirect_Failure_Configuration_Checklist
- STP_BackboneFast_Legacy_Indirect_Failure_Skeleton
- STP_BackboneFast_Legacy_Indirect_Failure_Verification_Commands
- STP_BackboneFast_Legacy_Indirect_Failure_Rollback
- STP_BackboneFast_Legacy_Indirect_Failure_Failure_Checks

# STP_BackboneFast_Legacy_Indirect_Failure_Mental_Model

| Concept | Operational Meaning |
|---|---|
| BackboneFast | Cisco-proprietary legacy STP feature that speeds convergence after an indirectly connected link failure |
| Legacy STP feature | BackboneFast belongs to classic 802.1D/PVST environments, not modern Rapid PVST designs |
| Indirect failure | A switch detects that another switch lost its path to the root bridge because it receives inferior BPDUs |
| Inferior BPDU | A BPDU that claims worse root information than the receiver already has |
| Nondesignated port | A blocked or non-forwarding port that receives BPDUs from another switch |
| Max age expiration | BackboneFast shortens convergence by expiring the max-age timer instead of waiting the full default timer |
| Default max age | Classic STP normally uses a 20-second max-age timer before old root information ages out |
| Listening and learning still happen | BackboneFast does not skip normal listening and learning states like UplinkFast does |
| Expected improvement | BackboneFast usually saves the max-age wait, so convergence is improved but still not instant |
| RLQ | Root Link Query is the request and response process used to verify whether the root bridge is still reachable |
| RLQ request | A switch sends an RLQ request upstream through its root port to check root reachability |
| RLQ response | An upstream switch replies if it still has connectivity to the root bridge |
| All-switch requirement | BackboneFast must be enabled on all switches in the STP domain for proper operation |
| Not UplinkFast | UplinkFast handles directly connected access uplink failure; BackboneFast handles indirect failure |
| Not Root Guard | Root Guard protects root placement; BackboneFast speeds indirect-failure convergence |
| Not timer tuning | BackboneFast improves convergence without manually changing hello, forward-delay, or max-age timers |
| Test trigger | Shut a link between another switch and the root bridge, then observe convergence on a third switch |
| Verification focus | Look for BackboneFast enabled status, inferior BPDU behavior, RLQ logic, topology change, and final port roles |

# STP_BackboneFast_Legacy_Indirect_Failure_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the Layer 2 topology before enabling legacy convergence features | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended triangle or redundant topology |
| 2 | Confirm interswitch links are physically up | SW1/SW2/SW3 | `show interfaces status` | Interswitch ports show `connected` |
| 3 | Confirm trunks are operating if VLANs cross switch links | SW1/SW2/SW3 | `show interfaces trunk` | Required interswitch links show `trunking` |
| 4 | Confirm the target VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 1 or the target VLAN exists and is active |
| 5 | Confirm current STP mode before using BackboneFast | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before changes |
| 6 | Confirm root placement before the test | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Intended root bridge is known |
| 7 | Confirm the root bridge has no root port | SW1 | `show spanning-tree vlan 1` | Root bridge output shows it is the root |
| 8 | Confirm the root bridge has no blocked ports for the target VLAN | SW1 | `show spanning-tree blockedports` | Root bridge should not have blocked ports for the active topology |
| 9 | Confirm downstream switches have root ports and possible blocked paths | SW2/SW3 | `show spanning-tree vlan 1` | Non-root switches show root, designated, and blocked or alternate paths as expected |
| 10 | Record current topology change state | SW1/SW2/SW3 | `show spanning-tree vlan 1 detail | include occurr|from|change` | Baseline topology change information is documented |
| 11 | Check current BackboneFast state before configuration | SW1/SW2/SW3 | `show spanning-tree summary | include BackboneFast` | BackboneFast state is known before changes |
| 12 | Enter global configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 13 | Enable BackboneFast on SW1 | SW1 | `spanning-tree backbonefast` | BackboneFast is enabled globally |
| 14 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 15 | Enter global configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 16 | Enable BackboneFast on SW2 | SW2 | `spanning-tree backbonefast` | BackboneFast is enabled globally |
| 17 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 18 | Enter global configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 19 | Enable BackboneFast on SW3 | SW3 | `spanning-tree backbonefast` | BackboneFast is enabled globally |
| 20 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 21 | Verify BackboneFast is enabled on every switch | SW1/SW2/SW3 | `show spanning-tree summary | include BackboneFast` | Each switch shows BackboneFast is enabled |
| 22 | Verify STP baseline after enabling BackboneFast | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Root bridge and port roles remain logically correct |
| 23 | Verify no inconsistent ports exist before testing | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No inconsistent ports are listed |
| 24 | Start continuous traffic before the failure test | H1/R1 | `ping <remote-same-vlan-host-ip> repeat 1000000` | Traffic begins so loss can be observed during reconvergence |
| 25 | Enable terminal monitoring for live lab observation | SW3 | `terminal monitor` | Syslog or debug output can display in the session |
| 26 | Enable STP event debugging in the lab | SW3 | `debug spanning-tree events` | STP event messages display live |
| 27 | Record the time before the indirect failure test | SW3 | `show clock` | Test timestamp is documented |
| 28 | Create an indirect failure by shutting SW2 link toward the root bridge | SW2 | `configure terminal` then `interface FastEthernet0/19` then `shutdown` | SW2 loses its direct path to the root bridge |
| 29 | Exit configuration mode quickly after the failure | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 30 | Observe STP events on the switch that receives inferior BPDUs | SW3 | Review terminal output | SW3 detects inferior BPDU behavior and STP state movement |
| 31 | Verify VLAN 1 convergence after BackboneFast operation | SW3 | `show spanning-tree vlan 1` | SW3 port roles and states update toward the new stable topology |
| 32 | Verify SW2 selected a new root port after losing the original path | SW2 | `show spanning-tree vlan 1` | SW2 uses the remaining path toward the root bridge |
| 33 | Verify the root bridge still remains the root | SW1 | `show spanning-tree vlan 1` | SW1 still shows itself as the root bridge |
| 34 | Check topology change counters after the test | SW1/SW2/SW3 | `show spanning-tree vlan 1 detail | include occurr|from|change` | Topology change source and count reflect the test |
| 35 | Check MAC learning after convergence | SW1/SW2/SW3 | `show mac address-table dynamic vlan 1` | MAC entries relearn on the new forwarding path |
| 36 | Verify traffic recovers after reconvergence | H1/R1 | Review ping output | Packet loss stops after STP reconverges |
| 37 | Restore the failed link after observation | SW2 | `configure terminal` then `interface FastEthernet0/19` then `no shutdown` | Original link comes back up |
| 38 | Exit configuration mode after restoration | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 39 | Observe STP after restoring the link | SW2/SW3 | `show spanning-tree vlan 1` | Ports reconverge into the final stable state |
| 40 | Confirm no inconsistent ports after restoration | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No inconsistent ports are listed |
| 41 | Disable STP debugging after the lab | SW3 | `undebug all` | Debug output stops |
| 42 | Disable terminal monitoring if no longer needed | SW3 | `terminal no monitor` | Live terminal log display stops |
| 43 | Save the configuration if BackboneFast should remain enabled | SW1/SW2/SW3 | `write memory` | BackboneFast configuration survives reload |

# STP_BackboneFast_Legacy_Indirect_Failure_Skeleton

Enable BackboneFast on every switch in the STP domain:

SW1:

configure terminal
!
spanning-tree backbonefast
!
end
write memory

SW2:

configure terminal
!
spanning-tree backbonefast
!
end
write memory

SW3:

configure terminal
!
spanning-tree backbonefast
!
end
write memory

Baseline verification before testing:

show spanning-tree summary
show spanning-tree summary | include BackboneFast
show spanning-tree root
show spanning-tree vlan 1
show spanning-tree vlan 1 detail | include occurr|from|change
show spanning-tree blockedports
show spanning-tree inconsistentports
show interfaces trunk
show mac address-table dynamic vlan 1

Lab-only live observation on the switch expected to receive inferior BPDUs:

terminal monitor
debug spanning-tree events
show clock

Indirect failure trigger from a different switch:

configure terminal
interface FastEthernet0/19
 shutdown
end

Immediate observation after failure:

show clock
show spanning-tree vlan 1
show spanning-tree vlan 1 detail | include occurr|from|change
show mac address-table dynamic vlan 1
show logging | include SPANTREE|Backbone|BACKBONE|RLQ|inferior|Topology

Restore the failed link:

configure terminal
interface FastEthernet0/19
 no shutdown
end

Final verification after restoration:

show spanning-tree vlan 1
show spanning-tree root
show spanning-tree inconsistentports
show mac address-table dynamic vlan 1
show logging | include SPANTREE|Backbone|BACKBONE|RLQ|inferior|Topology

Stop debug:

undebug all
terminal no monitor

Optional traffic test from Cisco host or router:

ping <remote-same-vlan-host-ip> repeat 1000000

Optional traffic test from Linux host:

ping <remote-same-vlan-host-ip>

# STP_BackboneFast_Legacy_Indirect_Failure_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm BackboneFast is enabled | SW1/SW2/SW3 | `show spanning-tree summary | include BackboneFast` | Every switch shows BackboneFast enabled |
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | Classic STP or PVST behavior is known before testing |
| Confirm root bridge | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge is known for the target VLAN |
| Confirm detailed VLAN state | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Root ID, bridge ID, roles, states, costs, and timers are visible |
| Confirm root switch status | SW1 | `show spanning-tree vlan 1` | SW1 shows itself as root if SW1 is intended root |
| Confirm root bridge has no blocked ports | SW1 | `show spanning-tree blockedports` | Root bridge has no blocked ports in the active topology |
| Confirm non-root root ports | SW2/SW3 | `show spanning-tree vlan 1` | Non-root switches have root ports toward the root bridge |
| Confirm blocked or alternate paths | SW2/SW3 | `show spanning-tree vlan 1` | Redundant path exists before failure testing |
| Confirm topology change information | SW1/SW2/SW3 | `show spanning-tree vlan 1 detail | include occurr|from|change` | Topology change count and source can be observed |
| Confirm inferior BPDU or STP event logs | SW3 | `show logging | include SPANTREE|inferior|RLQ|Backbone|BACKBONE` | Logs may show STP events tied to the indirect failure |
| Confirm final SW2 root port after failure | SW2 | `show spanning-tree vlan 1` | SW2 uses the remaining path as root port |
| Confirm final SW3 forwarding state after failure | SW3 | `show spanning-tree vlan 1` | SW3 moves the correct segment toward forwarding |
| Confirm MAC learning after convergence | SW1/SW2/SW3 | `show mac address-table dynamic vlan 1` | MAC entries relearn on the new forwarding path |
| Confirm trunks still carry the VLAN | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 1 or target VLAN is active and forwarding where expected |
| Confirm no inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no err-disabled ports | SW1/SW2/SW3 | `show interfaces status err-disabled` | Required interswitch ports are not err-disabled |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | No active MAC flap or loop messages appear |
| Confirm traffic recovery | H1/R1 | `ping <remote-same-vlan-host-ip> repeat 100` | Ping succeeds after convergence completes |
| Confirm restored link state | SW2 | `show interfaces FastEthernet0/19 status` | Restored link shows connected after `no shutdown` |
| Confirm debugging is disabled | SW3 | `show debugging` | No STP debug remains enabled |
| Confirm saved BackboneFast configuration | SW1/SW2/SW3 | `show startup-config | include spanning-tree backbonefast` | Startup configuration contains BackboneFast if intentionally retained |

# STP_BackboneFast_Legacy_Indirect_Failure_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop debugging before rollback | SW1/SW2/SW3 | `undebug all` | Debug output stops |
| 2 | Disable terminal monitoring if enabled | SW1/SW2/SW3 | `terminal no monitor` | Live terminal log display stops |
| 3 | Restore any test-shutdown root-facing link | SW2 | `configure terminal` then `interface FastEthernet0/19` then `no shutdown` | Test link is administratively enabled |
| 4 | Exit configuration mode after restoring the link | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 5 | Verify restored interface state | SW2 | `show interfaces FastEthernet0/19 status` | Interface shows connected if the far end is up |
| 6 | Verify STP is stable before removing BackboneFast | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Root, roles, and states are stable |
| 7 | Enter configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 8 | Disable BackboneFast on SW1 | SW1 | `no spanning-tree backbonefast` | BackboneFast is disabled on SW1 |
| 9 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 10 | Enter configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 11 | Disable BackboneFast on SW2 | SW2 | `no spanning-tree backbonefast` | BackboneFast is disabled on SW2 |
| 12 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 13 | Enter configuration mode on SW3 | SW3 | `configure terminal` | Prompt changes to global configuration mode |
| 14 | Disable BackboneFast on SW3 | SW3 | `no spanning-tree backbonefast` | BackboneFast is disabled on SW3 |
| 15 | Exit SW3 configuration mode | SW3 | `end` | Prompt returns to privileged EXEC mode |
| 16 | Verify BackboneFast is disabled on every switch | SW1/SW2/SW3 | `show spanning-tree summary | include BackboneFast` | Each switch shows BackboneFast disabled |
| 17 | Verify final STP state after rollback | SW1/SW2/SW3 | `show spanning-tree vlan 1` | STP roles and states are stable |
| 18 | Verify no inconsistent ports after rollback | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 19 | Save rollback | SW1/SW2/SW3 | `write memory` | Startup configuration reflects rollback |

# STP_BackboneFast_Legacy_Indirect_Failure_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| BackboneFast does not appear enabled | Feature was not configured or not saved | SW1/SW2/SW3 | `show spanning-tree summary | include BackboneFast` | Configure `spanning-tree backbonefast` globally |
| BackboneFast only works inconsistently | Feature is not enabled on all switches | SW1/SW2/SW3 | `show spanning-tree summary | include BackboneFast` | Enable BackboneFast on every switch in the STP domain |
| Convergence does not improve | The failure is not an indirect failure | SW1/SW2/SW3 | `show spanning-tree vlan 1` | Use UplinkFast for direct local access-uplink failure or Rapid PVST for modern designs |
| Convergence does not improve | No inferior BPDU condition is created | SW3 | `debug spanning-tree events` | Test by failing another switch’s root-facing link, not the local active port |
| Test shuts the wrong interface | The failed port was not the intended root-facing link | SW2 | `show spanning-tree vlan 1` | Identify the current root port before shutting an interface |
| SW3 sees no change during test | SW3 is not on the segment receiving inferior BPDUs | SW3 | `show spanning-tree vlan 1` | Observe from the switch connected to the affected downstream segment |
| VLAN missing from STP output | VLAN does not exist or has no active port | SW1/SW2/SW3 | `show vlan brief` | Create the VLAN and activate access or trunk ports |
| VLAN not carried across topology | VLAN not allowed or not active on trunk | SW1/SW2/SW3 | `show interfaces trunk` | Add the VLAN to trunk allowed lists and verify it is active |
| Port remains blocked after expected convergence | STP still sees a better or unsafe path | SW2/SW3 | `show spanning-tree vlan 1` | Verify root placement, port roles, path cost, and protection states |
| Port is inconsistent | Root Guard, Loop Guard, Bridge Assurance, or dispute condition is active | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | Fix the protection trigger before expecting forwarding |
| Port is err-disabled | Protection feature or physical issue disabled the port | SW1/SW2/SW3 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the interface |
| Ping loss still lasts a long time | Classic listening and learning still occur after max-age is bypassed | H1/R1 | Review ping output | This is expected for BackboneFast; use Rapid PVST for faster modern convergence |
| Expectation is instant failover | BackboneFast is being confused with UplinkFast or RSTP | SW1/SW2/SW3 | `show spanning-tree summary` | Use the correct feature for the failure type |
| MAC addresses flap after test | Layer 2 loop or unstable STP state | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | Verify one loop-free topology and correct blocked ports |
| Topology change count keeps increasing | Flapping link or unstable port state | SW1/SW2/SW3 | `show spanning-tree vlan 1 detail | include occurr|from|change` | Identify the source port and fix the instability |
| Root bridge changes during test | Root priority design is weak or root bridge path failed | SW1/SW2/SW3 | `show spanning-tree root` | Configure deterministic root and secondary root placement |
| Root bridge has blocked ports | Root placement or topology expectation is wrong | SW1 | `show spanning-tree blockedports` | Recheck root placement and physical topology |
| Debug output is missing | Terminal monitor or debug was not enabled in the active session | SW3 | `show debugging` | Use `terminal monitor` and `debug spanning-tree events` in the lab |
| Debug output remains enabled after testing | Debug was not disabled | SW3 | `show debugging` | Run `undebug all` |
| Temporary shutdown remains after test | The test link was not restored | SW2 | `show running-config interface FastEthernet0/19` | Apply `no shutdown` and verify the link state |
| Temporary shutdown was saved | Configuration was saved while test link was shut | SW2 | `show startup-config interface FastEthernet0/19` | Remove `shutdown`, restore the link, and save again |
| BackboneFast disappears after reload | Configuration was not saved | SW1/SW2/SW3 | `show startup-config | include spanning-tree backbonefast` | Reapply `spanning-tree backbonefast` and run `write memory` |
