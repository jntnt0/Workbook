STP_Dispute_Inconsistent_Port_Handling.md
# STP_Dispute_Inconsistent_Port_Handling

# STP_Dispute_Inconsistent_Port_Handling_Index

- STP_Dispute_Inconsistent_Port_Handling_Mental_Model
- STP_Dispute_Inconsistent_Port_Handling_Configuration_Checklist
- STP_Dispute_Inconsistent_Port_Handling_Skeleton
- STP_Dispute_Inconsistent_Port_Handling_Verification_Commands
- STP_Dispute_Inconsistent_Port_Handling_Rollback
- STP_Dispute_Inconsistent_Port_Handling_Failure_Checks

# STP_Dispute_Inconsistent_Port_Handling_Mental_Model

| Concept | Operational Meaning |
|---|---|
| STP dispute | RSTP protection behavior that prevents a port from forwarding when the local and remote switches disagree about safe forwarding state |
| Inconsistent port | A port or VLAN instance blocked by an STP protection condition |
| Dispute state | A broken state where the port is blocked because STP sees unsafe BPDU behavior from the neighbor |
| Loop prevention | Dispute state is protective, not random; the switch is stopping a possible Layer 2 loop |
| RSTP behavior | Dispute handling is part of RSTP/Rapid PVST behavior and is not normally configured as a separate feature |
| Inferior BPDU | A BPDU that advertises worse root information than the local switch already has |
| Unsafe neighbor state | If the neighbor appears to be forwarding when the local switch believes it should not, dispute can block the port |
| Unidirectional symptoms | One-way BPDU flow, bad fiber strand, cabling issue, or hardware fault can create inconsistent STP behavior |
| BPDU filtering risk | BPDU Filter can hide BPDUs and create unsafe STP state if used on interswitch links |
| Trunk mismatch risk | Native VLAN, allowed VLAN, or inconsistent trunk state can make STP behavior differ per VLAN |
| EtherChannel risk | Bundle mismatch can leave one side treating links as a port-channel while the other treats them as individual links |
| CDP capability | CDP may show `s - Supports-STP-Dispute`, meaning the neighbor supports the dispute mechanism |
| Automatic recovery | When the unsafe BPDU condition clears, STP can reconverge and the port can leave dispute state |
| Not Root Guard | Root Guard blocks superior BPDUs on downstream designated ports with root-inconsistent state |
| Not Loop Guard | Loop Guard blocks root or alternate ports when BPDUs stop arriving with loop-inconsistent state |
| Not BPDU Guard | BPDU Guard err-disables PortFast ports when any BPDU is received |
| Handling rule | Do not force the port forwarding; find the unsafe BPDU or link condition and correct it |
| Verification rule | Use `show spanning-tree inconsistentports`, per-VLAN STP output, interface STP detail, logs, trunks, and physical link checks |

# STP_Dispute_Inconsistent_Port_Handling_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the physical Layer 2 topology | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended topology |
| 2 | Confirm whether neighbors support STP dispute | SW1/SW2/SW3 | `show cdp neighbors` | Capability legend may show `s - Supports-STP-Dispute` |
| 3 | Confirm current STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is known before troubleshooting |
| 4 | Use Rapid PVST for an RSTP dispute lab if intentionally testing dispute behavior | SW1/SW2/SW3 | `configure terminal` then `spanning-tree mode rapid-pvst` then `end` | Switches run Rapid PVST+ |
| 5 | Confirm target VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| 6 | Confirm interswitch links are physically connected | SW1/SW2/SW3 | `show interfaces status` | Trunk or switch-facing ports show `connected` |
| 7 | Confirm interswitch trunk operation | SW1/SW2/SW3 | `show interfaces trunk` | Required links show `trunking` |
| 8 | Confirm VLAN 10 is carried across the trunk | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 10 is allowed, active, and forwarding where expected |
| 9 | Confirm baseline root placement | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge and root ports are known |
| 10 | Confirm detailed VLAN 10 STP baseline | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root ID, bridge ID, port roles, states, costs, and type are visible |
| 11 | Check whether any port is already inconsistent | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports should be listed before the test or repair |
| 12 | Check STP interface detail on the suspect link | SW1 | `show spanning-tree vlan 10 interface GigabitEthernet0/1 detail` | Role, state, BPDU counters, link type, and timers are visible |
| 13 | Check the far-end STP interface detail | SW2 | `show spanning-tree vlan 10 interface GigabitEthernet0/1 detail` | Far-end role, state, BPDU counters, link type, and timers are visible |
| 14 | Check BPDU counters on both ends | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1 detail | include BPDU|sent|received` | Both ends should send and receive BPDUs on interswitch links |
| 15 | Check for BPDU Filter on the suspect link | SW1/SW2 | `show running-config interface GigabitEthernet0/1 | include bpdufilter` | BPDU Filter should not be enabled on normal interswitch trunks |
| 16 | Check for PortFast on the suspect trunk | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1 detail | include portfast|edge|PortFast` | Trunk should not be treated as an edge host port |
| 17 | Check for Root Guard or Loop Guard on the suspect link | SW1/SW2 | `show running-config interface GigabitEthernet0/1 | include guard` | Guard features are known before changing anything |
| 18 | Check for current logs related to dispute or inconsistent state | SW1/SW2/SW3 | `show logging | include SPANTREE|Dispute|dispute|inconsistent|BKN|LOOP` | Logs show whether STP placed the port into an inconsistent condition |
| 19 | Check for MAC flapping symptoms | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP` | No active MAC flap or loop symptoms should remain |
| 20 | Check physical errors on the suspect link | SW1/SW2 | `show interfaces GigabitEthernet0/1 counters errors` | No excessive CRC, input, output, or link errors should appear |
| 21 | Check line protocol and duplex state | SW1/SW2 | `show interfaces GigabitEthernet0/1` | Interface is up/up and duplex/speed are sane |
| 22 | Check UDLD status if enabled or supported | SW1/SW2 | `show udld interface GigabitEthernet0/1` | UDLD does not show unidirectional or err-disabled state |
| 23 | Check EtherChannel state if the suspect link is bundled | SW1/SW2 | `show etherchannel summary` | Member links are bundled consistently on both ends |
| 24 | Check the port-channel STP state if bundled | SW1/SW2 | `show spanning-tree vlan 10 interface Port-channel1 detail` | STP runs on the logical port-channel, not mismatched physical links |
| 25 | Check native VLAN consistency on trunks | SW1/SW2 | `show interfaces trunk` | Native VLAN matches on both ends |
| 26 | Check allowed VLAN consistency on trunks | SW1/SW2 | `show interfaces trunk` | VLAN 10 is allowed on both ends |
| 27 | Remove BPDU Filter from an interswitch link if found | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `no spanning-tree bpdufilter enable` then `end` | BPDUs are no longer suppressed on the interswitch link |
| 28 | Remove PortFast from an interswitch trunk if found | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `spanning-tree portfast disable` then `end` | Trunk is treated as a non-edge STP port |
| 29 | Correct trunk mode if the link is supposed to be a trunk | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `switchport mode trunk` then `end` | Link is administratively trunking |
| 30 | Correct VLAN allowance if VLAN 10 is missing | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `switchport trunk allowed vlan add 10` then `end` | VLAN 10 is carried without overwriting the existing allowed list |
| 31 | Correct native VLAN mismatch if found | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `switchport trunk native vlan 999` then `end` | Native VLAN matches on both trunk ends |
| 32 | Correct EtherChannel mismatch if found | SW1/SW2 | `show running-config interface GigabitEthernet0/1` | Channel-group mode and port-channel config match on both ends |
| 33 | Shut and restore the suspect port only after fixing the underlying issue | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `shutdown` then `no shutdown` then `end` | Link restarts cleanly after configuration correction |
| 34 | Verify the inconsistent condition clears | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No dispute or inconsistent ports are listed |
| 35 | Verify VLAN 10 STP state after recovery | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Port roles and states return to normal forwarding or blocking design |
| 36 | Verify BPDU counters after recovery | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1 detail | include BPDU|sent|received` | BPDU sent and received counters increment normally |
| 37 | Verify trunk state after recovery | SW1/SW2 | `show interfaces trunk` | Trunk is up and carrying required VLANs |
| 38 | Verify MAC learning after recovery | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MACs relearn on expected ports and do not flap |
| 39 | Verify host reachability after recovery | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works after STP recovery |
| 40 | Save only the intended permanent fixes | SW1/SW2/SW3 | `write memory` | Corrected configuration survives reload |

# STP_Dispute_Inconsistent_Port_Handling_Skeleton

Baseline discovery:

show cdp neighbors
show spanning-tree summary
show vlan brief
show interfaces status
show interfaces trunk
show spanning-tree root
show spanning-tree vlan 10
show spanning-tree inconsistentports
show logging | include SPANTREE|Dispute|dispute|inconsistent|BKN|LOOP|MACFLAP

Interface-level STP inspection:

show spanning-tree vlan 10 interface GigabitEthernet0/1 detail
show spanning-tree interface GigabitEthernet0/1 detail | include BPDU|sent|received|PortFast|edge|Link|Role|State
show running-config interface GigabitEthernet0/1
show interfaces GigabitEthernet0/1
show interfaces GigabitEthernet0/1 counters errors

Trunk inspection:

show interfaces trunk
show interfaces GigabitEthernet0/1 switchport
show running-config interface GigabitEthernet0/1

EtherChannel inspection if applicable:

show etherchannel summary
show etherchannel detail
show running-config interface Port-channel1
show spanning-tree vlan 10 interface Port-channel1 detail

UDLD inspection if applicable:

show udld
show udld interface GigabitEthernet0/1
show interfaces status err-disabled

Correct BPDU Filter on an interswitch link:

configure terminal
!
interface GigabitEthernet0/1
 no spanning-tree bpdufilter enable
 spanning-tree portfast disable
 no shutdown
!
end

Correct trunk baseline:

configure terminal
!
interface GigabitEthernet0/1
 description TRUNK_TO_NEIGHBOR
 switchport mode trunk
 switchport trunk allowed vlan add 10
 switchport trunk native vlan 999
 spanning-tree portfast disable
 no shutdown
!
end

Restart the link after correcting the root cause:

configure terminal
!
interface GigabitEthernet0/1
 shutdown
 no shutdown
!
end

Verify recovery:

show spanning-tree inconsistentports
show spanning-tree vlan 10
show spanning-tree vlan 10 interface GigabitEthernet0/1 detail
show spanning-tree interface GigabitEthernet0/1 detail | include BPDU|sent|received
show interfaces trunk
show mac address-table dynamic vlan 10
show logging | include SPANTREE|Dispute|dispute|inconsistent|BKN|LOOP|MACFLAP

Save final intended fixes:

write memory

# STP_Dispute_Inconsistent_Port_Handling_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode is visible and known |
| Confirm VLAN presence | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| Confirm trunk state | SW1/SW2 | `show interfaces trunk` | Suspect link is trunking if it should be a trunk |
| Confirm native VLAN consistency | SW1/SW2 | `show interfaces trunk` | Native VLAN matches on both ends |
| Confirm VLAN allowance | SW1/SW2 | `show interfaces trunk` | VLAN 10 is allowed, active, and forwarding where expected |
| Confirm root bridge | SW1/SW2/SW3 | `show spanning-tree root` | Intended root bridge is known |
| Confirm detailed STP state | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root ID, bridge ID, roles, states, costs, and type are visible |
| Confirm inconsistent ports | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | Dispute or other inconsistent ports are shown if present |
| Confirm suspect interface STP detail | SW1/SW2 | `show spanning-tree vlan 10 interface GigabitEthernet0/1 detail` | Interface role, state, BPDU counters, and timers are visible |
| Confirm BPDUs flow both ways | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1 detail | include BPDU|sent|received` | BPDU sent and received counters increment |
| Confirm no BPDU Filter on trunk | SW1/SW2 | `show running-config interface GigabitEthernet0/1 | include bpdufilter` | No interface-level BPDU Filter exists on interswitch trunk |
| Confirm trunk is not edge | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1 detail | include PortFast|edge` | Switch-to-switch trunk is not operating as an edge port |
| Confirm guard features | SW1/SW2 | `show running-config interface GigabitEthernet0/1 | include guard` | Root Guard or Loop Guard state is known |
| Confirm physical errors | SW1/SW2 | `show interfaces GigabitEthernet0/1 counters errors` | Error counters are not climbing |
| Confirm interface line state | SW1/SW2 | `show interfaces GigabitEthernet0/1` | Interface is up/up after repair |
| Confirm err-disabled status | SW1/SW2 | `show interfaces status err-disabled` | Suspect interface is not err-disabled |
| Confirm UDLD state | SW1/SW2 | `show udld interface GigabitEthernet0/1` | No unidirectional condition exists if UDLD is enabled |
| Confirm EtherChannel state | SW1/SW2 | `show etherchannel summary` | Port-channel members are bundled correctly if used |
| Confirm port-channel STP state | SW1/SW2 | `show spanning-tree vlan 10 interface Port-channel1 detail` | Logical bundle has expected STP state |
| Confirm log evidence | SW1/SW2/SW3 | `show logging | include SPANTREE|Dispute|dispute|inconsistent|BKN` | Logs show dispute or recovery events if present |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP` | No active MAC flap or loop messages appear |
| Confirm MAC learning is stable | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MAC addresses are stable on expected ports |
| Confirm data-plane recovery | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic succeeds |
| Confirm final running config | SW1/SW2 | `show running-config interface GigabitEthernet0/1` | Corrected trunk and STP config is present |
| Confirm saved config | SW1/SW2 | `show startup-config interface GigabitEthernet0/1` | Startup config contains intended final fixes |

# STP_Dispute_Inconsistent_Port_Handling_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current inconsistent state before rollback | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | Current inconsistent ports are documented |
| 2 | Record current STP state before rollback | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Current root, roles, and states are documented |
| 3 | Record suspect interface configuration | SW1/SW2 | `show running-config interface GigabitEthernet0/1` | Current interface config is documented |
| 4 | Remove lab-only BPDU Filter if used to trigger the condition | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `no spanning-tree bpdufilter enable` then `end` | BPDUs are no longer suppressed |
| 5 | Remove lab-only PortFast from interswitch trunk if configured | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `spanning-tree portfast disable` then `end` | Trunk is not treated as an edge port |
| 6 | Restore correct trunk mode | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `switchport mode trunk` then `end` | Interface returns to intended trunk operation |
| 7 | Restore VLAN allowance if it was changed during testing | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `switchport trunk allowed vlan add 10` then `end` | VLAN 10 is allowed again |
| 8 | Restore native VLAN if it was changed during testing | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `switchport trunk native vlan 999` then `end` | Native VLAN matches the intended design |
| 9 | Restore EtherChannel membership if changed during testing | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `channel-group 1 mode active` then `end` | Link rejoins the intended port-channel if applicable |
| 10 | Restore any test-shutdown interface | SW1/SW2 | `configure terminal` then `interface GigabitEthernet0/1` then `no shutdown` then `end` | Interface is administratively up |
| 11 | Verify physical recovery | SW1/SW2 | `show interfaces status` | Interface shows connected if the far end is up |
| 12 | Verify trunk recovery | SW1/SW2 | `show interfaces trunk` | Trunk carries required VLANs |
| 13 | Verify STP inconsistent state clears | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No dispute or inconsistent ports remain |
| 14 | Verify final VLAN 10 STP state | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Port roles and states match the expected topology |
| 15 | Verify host reachability | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works |
| 16 | Save rollback only after confirming the intended final state | SW1/SW2/SW3 | `write memory` | Startup config reflects the restored safe state |

# STP_Dispute_Inconsistent_Port_Handling_Failure_Checks

| Symptom                                         | Likely Cause                                                 | Device      | Command                                                                                     | Fix                                                                            |                                                              |                                                                        |                                                              |
| ----------------------------------------------- | ------------------------------------------------------------ | ----------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------------------- | ------------------------------------------------------------ |
| Port appears as dispute or inconsistent         | Unsafe RSTP state between neighbors                          | SW1/SW2     | `show spanning-tree inconsistentports`                                                      | Inspect BPDU flow, trunk state, and far-end STP state                          |                                                              |                                                                        |                                                              |
| Dispute returns after clearing the port         | Root cause still exists                                      | SW1/SW2     | `show logging                                                                               | include Dispute                                                                | SPANTREE                                                     | inconsistent`                                                          | Fix BPDU suppression, unidirectional link, or trunk mismatch |
| BPDU counters only increment one way            | Unidirectional link or BPDU suppression                      | SW1/SW2     | `show spanning-tree interface GigabitEthernet0/1 detail                                     | include BPDU`                                                                  | Check fiber, cabling, UDLD, BPDU Filter, and physical errors |                                                                        |                                                              |
| Interface is up/up but BPDUs are missing        | One-way link or BPDU Filter                                  | SW1/SW2     | `show interfaces GigabitEthernet0/1` and `show running-config interface GigabitEthernet0/1` | Remove BPDU Filter or fix the physical one-way condition                       |                                                              |                                                                        |                                                              |
| BPDU Filter is configured on trunk              | Misapplied edge-port feature                                 | SW1/SW2     | `show running-config interface GigabitEthernet0/1                                           | include bpdufilter`                                                            | Remove BPDU Filter from interswitch links                    |                                                                        |                                                              |
| Trunk is treated as edge                        | PortFast was applied to a switch-facing link                 | SW1/SW2     | `show spanning-tree interface GigabitEthernet0/1 detail                                     | include PortFast                                                               | edge`                                                        | Apply `spanning-tree portfast disable`                                 |                                                              |
| VLAN is inconsistent on trunk                   | VLAN is allowed on one side but not the other                | SW1/SW2     | `show interfaces trunk`                                                                     | Fix allowed VLAN list on both ends                                             |                                                              |                                                                        |                                                              |
| Native VLAN mismatch appears                    | Native VLAN differs between trunk ends                       | SW1/SW2     | `show interfaces trunk` and `show logging                                                   | include NATIVE                                                                 | native`                                                      | Configure matching native VLANs                                        |                                                              |
| Dispute affects only one VLAN                   | Per-VLAN trunk or STP state differs                          | SW1/SW2     | `show spanning-tree vlan <vlan-id>`                                                         | Fix that VLAN's trunk allowance, STP state, or BPDU flow                       |                                                              |                                                                        |                                                              |
| Port-channel behaves inconsistently             | EtherChannel mismatch or split STP view                      | SW1/SW2     | `show etherchannel summary`                                                                 | Fix channel-group mode, member config, and use STP on the logical port-channel |                                                              |                                                                        |                                                              |
| Physical member forwards outside the bundle     | One side is bundled and the other is not                     | SW1/SW2     | `show etherchannel summary`                                                                 | Correct EtherChannel config on both ends                                       |                                                              |                                                                        |                                                              |
| Root bridge is not the intended switch          | Root priority design is wrong                                | SW1/SW2/SW3 | `show spanning-tree root`                                                                   | Configure deterministic primary and secondary root placement                   |                                                              |                                                                        |                                                              |
| Port remains blocked after fix                  | STP has not reconverged or another protection state exists   | SW1/SW2     | `show spanning-tree vlan 10`                                                                | Check root, role, state, guard features, and timers                            |                                                              |                                                                        |                                                              |
| Port shows root-inconsistent instead of dispute | Root Guard is blocking superior BPDUs                        | SW1/SW2     | `show spanning-tree inconsistentports`                                                      | Correct downstream root priority or remove unauthorized switch                 |                                                              |                                                                        |                                                              |
| Port shows loop-inconsistent instead of dispute | Loop Guard is blocking because BPDUs stopped                 | SW1/SW2     | `show spanning-tree inconsistentports`                                                      | Restore BPDU reception on the root or alternate port                           |                                                              |                                                                        |                                                              |
| Port is err-disabled instead of inconsistent    | BPDU Guard, UDLD aggressive, or port security triggered      | SW1/SW2     | `show interfaces status err-disabled`                                                       | Fix trigger, then shut/no shut the interface                                   |                                                              |                                                                        |                                                              |
| UDLD reports unidirectional                     | Physical link is one-way                                     | SW1/SW2     | `show udld interface GigabitEthernet0/1`                                                    | Replace cable, optics, patch path, or transceiver                              |                                                              |                                                                        |                                                              |
| Error counters climb                            | Bad cable, optics, duplex, or hardware path                  | SW1/SW2     | `show interfaces GigabitEthernet0/1 counters errors`                                        | Fix the physical layer                                                         |                                                              |                                                                        |                                                              |
| MAC addresses flap                              | Loop or unstable STP forwarding                              | SW1/SW2/SW3 | `show logging                                                                               | include MACFLAP                                                                | LOOP`                                                        | Keep the disputed port blocked until BPDU and topology state are fixed |                                                              |
| Host traffic fails after dispute clears         | Trunk, VLAN, or MAC table still unstable                     | SW1/SW2/SW3 | `show interfaces trunk`, `show mac address-table dynamic vlan 10`                           | Verify VLAN forwarding and generate traffic to relearn MACs                    |                                                              |                                                                        |                                                              |
| Clearing with shut/no shut does not fix it      | Shut/no shut treats the symptom only                         | SW1/SW2     | `show spanning-tree inconsistentports`                                                      | Fix the underlying BPDU, trunk, physical, or EtherChannel issue                |                                                              |                                                                        |                                                              |
| Dispute appears after adding a new switch       | New switch has mismatched STP, trunk, or EtherChannel config | New SW      | `show spanning-tree summary`, `show interfaces trunk`, `show etherchannel summary`          | Correct the new switch before reconnecting it                                  |                                                              |                                                                        |                                                              |
| Config disappears after reload                  | Fix was not saved                                            | SW1/SW2     | `show startup-config interface GigabitEthernet0/1`                                          | Reapply correction and run `write memory`                                      |                                                              |                                                                        |                                                              |