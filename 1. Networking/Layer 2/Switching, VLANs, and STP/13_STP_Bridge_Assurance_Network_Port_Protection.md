STP_Bridge_Assurance_Network_Port_Protection.md
# STP_Bridge_Assurance_Network_Port_Protection

# STP_Bridge_Assurance_Network_Port_Protection_Index

- STP_Bridge_Assurance_Network_Port_Protection_Mental_Model
- STP_Bridge_Assurance_Network_Port_Protection_Configuration_Checklist
- STP_Bridge_Assurance_Network_Port_Protection_Skeleton
- STP_Bridge_Assurance_Network_Port_Protection_Verification_Commands
- STP_Bridge_Assurance_Network_Port_Protection_Rollback
- STP_Bridge_Assurance_Network_Port_Protection_Failure_Checks

# STP_Bridge_Assurance_Network_Port_Protection_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Bridge Assurance | STP protection feature that keeps switch-to-switch links safe by requiring BPDUs to be received on network ports |
| Network port | A spanning-tree port type used for switch-to-switch links, not host-facing access ports |
| BPDU expectation | Bridge Assurance expects BPDUs to be sent and received continuously on network ports, including blocked or alternate ports |
| Point-to-point requirement | Bridge Assurance is intended for point-to-point switch links |
| Both-ends requirement | Both sides of the link must support Bridge Assurance and be configured as network ports |
| Rapid STP requirement | Bridge Assurance is supported with Rapid PVST+ and MST, not legacy 802.1D STP |
| BA-inconsistent state | If expected BPDUs stop arriving, the port or VLAN instance is blocked to prevent a Layer 2 loop |
| Automatic recovery | When valid BPDUs return, the port can leave inconsistent state and reconverge normally |
| Loop prevention goal | Bridge Assurance protects against failures where a device keeps forwarding frames but stops sending BPDUs |
| Not PortFast | PortFast is for host-facing edge ports; Bridge Assurance is for switch-facing network ports |
| Not BPDU Guard | BPDU Guard shuts down edge ports that receive BPDUs; Bridge Assurance blocks network ports that stop receiving expected BPDUs |
| Not Root Guard | Root Guard blocks superior BPDUs from downstream switches; Bridge Assurance blocks missing or invalid BPDU behavior on network ports |
| Not Loop Guard | Loop Guard protects root or alternate ports when BPDUs disappear; Bridge Assurance expects BPDUs on all network ports |
| Host-port danger | If a host-facing port is configured as a network port, it can block because hosts do not send BPDUs |
| Global default risk | Configuring network port type globally requires all host-facing ports to be explicitly marked as edge ports |
| Verification rule | Verify STP mode, Bridge Assurance state, port type, BPDU counters, inconsistent ports, trunks, and final forwarding state |

# STP_Bridge_Assurance_Network_Port_Protection_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the physical switch topology | SW1/SW2/SW3 | `show cdp neighbors` | Neighbor relationships match the intended switch-to-switch design |
| 2 | Confirm interswitch links are physically up | SW1/SW2/SW3 | `show interface status` | Switch-facing interfaces show connected |
| 3 | Confirm current STP mode before enabling Bridge Assurance behavior | SW1/SW2/SW3 | `show spanning-tree summary` | Current STP mode is known before changes |
| 4 | Use Rapid PVST+ or MST, not legacy 802.1D STP | SW1/SW2/SW3 | `configure terminal` then `spanning-tree mode rapid-pvst` then `end` | Switches run Rapid PVST+ |
| 5 | Confirm Bridge Assurance global state | SW1/SW2/SW3 | `show spanning-tree summary | include Bridge Assurance` | Bridge Assurance state is visible |
| 6 | Enable Bridge Assurance globally if the platform requires it | SW1/SW2/SW3 | `configure terminal` then `spanning-tree bridge assurance` then `end` | Bridge Assurance is enabled globally if supported |
| 7 | Confirm the target VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| 8 | Confirm the switch-to-switch link is a trunk if carrying multiple VLANs | SW1/SW2/SW3 | `show interfaces trunk` | Required interswitch links show trunking |
| 9 | Confirm VLAN 10 is carried on the interswitch trunk | SW1/SW2/SW3 | `show interfaces trunk` | VLAN 10 is allowed and active on the trunk |
| 10 | Confirm root placement before adding protection | SW1/SW2/SW3 | `show spanning-tree root` | Root bridge and root ports are known |
| 11 | Confirm detailed VLAN 10 STP state | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root ID, bridge ID, roles, states, and port types are known |
| 12 | Enter interface configuration mode on SW1 switch-facing link | SW1 | `configure terminal` then `interface Ethernet1/1` | Prompt changes to interface configuration mode |
| 13 | Add a clear description on SW1 | SW1 | `description NETWORK_PORT_TO_SW2_Eth1/1` | Interface purpose is visible in show commands |
| 14 | Configure the interface as a Layer 2 trunk | SW1 | `switchport` then `switchport mode trunk` | Interface is a Layer 2 trunk |
| 15 | Allow the target VLAN on the trunk without overwriting existing VLANs | SW1 | `switchport trunk allowed vlan add 10` | VLAN 10 is allowed on the trunk |
| 16 | Configure the switch-facing port as a spanning-tree network port | SW1 | `spanning-tree port type network` | Bridge Assurance applies on the network port |
| 17 | Bring up the SW1 network port | SW1 | `no shutdown` | Interface is administratively up |
| 18 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Enter interface configuration mode on SW2 switch-facing link | SW2 | `configure terminal` then `interface Ethernet1/1` | Prompt changes to interface configuration mode |
| 20 | Add a clear description on SW2 | SW2 | `description NETWORK_PORT_TO_SW1_Eth1/1` | Interface purpose is visible in show commands |
| 21 | Configure the interface as a Layer 2 trunk | SW2 | `switchport` then `switchport mode trunk` | Interface is a Layer 2 trunk |
| 22 | Allow the target VLAN on the trunk without overwriting existing VLANs | SW2 | `switchport trunk allowed vlan add 10` | VLAN 10 is allowed on the trunk |
| 23 | Configure the switch-facing port as a spanning-tree network port | SW2 | `spanning-tree port type network` | Bridge Assurance applies on the network port |
| 24 | Bring up the SW2 network port | SW2 | `no shutdown` | Interface is administratively up |
| 25 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 26 | Configure host-facing ports as edge ports, not network ports | SW1/SW2 | `configure terminal` then `interface Ethernet1/10` then `spanning-tree port type edge` then `end` | Host-facing ports are edge ports and do not expect BPDUs |
| 27 | Enable BPDU Guard on edge ports if used in the lab design | SW1/SW2 | `configure terminal` then `spanning-tree port type edge bpduguard default` then `end` | Edge ports are protected from accidental switch attachment |
| 28 | Verify Bridge Assurance global state | SW1/SW2 | `show spanning-tree summary | include Bridge Assurance` | Bridge Assurance is enabled |
| 29 | Verify the switch-facing port type | SW1/SW2 | `show spanning-tree interface Ethernet1/1 detail` | Interface shows network port behavior and Bridge Assurance participation |
| 30 | Verify BPDU transmit and receive counters | SW1/SW2 | `show spanning-tree interface Ethernet1/1 detail | include BPDU|sent|received` | BPDU counters increment in both directions |
| 31 | Verify the link is not inconsistent during normal operation | SW1/SW2 | `show spanning-tree inconsistentports` | No Bridge Assurance inconsistent ports are listed |
| 32 | Verify VLAN 10 STP state | SW1/SW2 | `show spanning-tree vlan 10` | Port roles and states are stable and loop-free |
| 33 | Verify the trunk still carries VLAN 10 | SW1/SW2 | `show interfaces trunk` | VLAN 10 is allowed, active, and forwarding where expected |
| 34 | Start continuous traffic before a controlled test | HOSTS | `ping <remote-same-vlan-host-ip> repeat 1000000` | Baseline traffic is working before the test |
| 35 | Simulate a Bridge Assurance mismatch by changing the far end to a normal port in a lab only | SW2 | `configure terminal` then `interface Ethernet1/1` then `spanning-tree port type normal` then `end` | SW2 no longer matches the expected network-port behavior |
| 36 | Verify Bridge Assurance blocks the unsafe link | SW1/SW2 | `show spanning-tree inconsistentports` | Interface appears as Bridge Assurance inconsistent or blocked |
| 37 | Check Bridge Assurance or STP log messages | SW1/SW2 | `show logging | include Bridge|BRIDGE|BKN|inconsistent|SPANTREE` | Logs show Bridge Assurance blocking or inconsistency |
| 38 | Restore the far end to a network port | SW2 | `configure terminal` then `interface Ethernet1/1` then `spanning-tree port type network` then `end` | Both ends are network ports again |
| 39 | Verify automatic recovery after BPDU expectation is restored | SW1/SW2 | `show spanning-tree inconsistentports` | Interface disappears from inconsistent port list |
| 40 | Verify final STP state after recovery | SW1/SW2 | `show spanning-tree vlan 10` | Port returns to normal STP forwarding or blocking based on topology |
| 41 | Verify host reachability after recovery | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic works after recovery |
| 42 | Save the intended configuration | SW1/SW2/SW3 | `copy running-config startup-config` | Bridge Assurance and port-type settings survive reload |

# STP_Bridge_Assurance_Network_Port_Protection_Skeleton

Rapid PVST+ baseline:

configure terminal
!
spanning-tree mode rapid-pvst
spanning-tree bridge assurance
!
vlan 10
 name USERS
!
end
copy running-config startup-config

SW1 switch-facing network port:

configure terminal
!
interface Ethernet1/1
 description NETWORK_PORT_TO_SW2_Eth1/1
 switchport
 switchport mode trunk
 switchport trunk allowed vlan add 10
 spanning-tree port type network
 no shutdown
!
end
copy running-config startup-config

SW2 switch-facing network port:

configure terminal
!
interface Ethernet1/1
 description NETWORK_PORT_TO_SW1_Eth1/1
 switchport
 switchport mode trunk
 switchport trunk allowed vlan add 10
 spanning-tree port type network
 no shutdown
!
end
copy running-config startup-config

Host-facing edge port:

configure terminal
!
interface Ethernet1/10
 description HOST_EDGE_PORT
 switchport
 switchport mode access
 switchport access vlan 10
 spanning-tree port type edge
 no shutdown
!
end
copy running-config startup-config

Global edge-port protection:

configure terminal
!
spanning-tree port type edge bpduguard default
!
end
copy running-config startup-config

Optional global default network-port model:

configure terminal
!
spanning-tree port type network default
!
end

Use the global network-port default only when all non-edge ports are true switch-to-switch links and host-facing ports are explicitly configured as edge ports.

Bridge Assurance mismatch test:

configure terminal
!
interface Ethernet1/1
 spanning-tree port type normal
!
end

Verify blocked state:

show spanning-tree inconsistentports
show spanning-tree vlan 10
show spanning-tree interface Ethernet1/1 detail
show logging | include Bridge|BRIDGE|BKN|inconsistent|SPANTREE

Restore network-port behavior:

configure terminal
!
interface Ethernet1/1
 spanning-tree port type network
!
end

Verify recovery:

show spanning-tree inconsistentports
show spanning-tree vlan 10
show spanning-tree interface Ethernet1/1 detail
show interfaces trunk

Disable Bridge Assurance globally if required for rollback:

configure terminal
!
no spanning-tree bridge assurance
!
end
copy running-config startup-config

# STP_Bridge_Assurance_Network_Port_Protection_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm STP mode | SW1/SW2/SW3 | `show spanning-tree summary` | Switches run Rapid PVST+ or MST |
| Confirm Bridge Assurance global state | SW1/SW2/SW3 | `show spanning-tree summary | include Bridge Assurance` | Bridge Assurance state is visible and enabled when intended |
| Confirm default port type | SW1/SW2/SW3 | `show spanning-tree summary | include Port Type Default` | Default port type is known |
| Confirm VLAN exists | SW1/SW2/SW3 | `show vlan brief` | VLAN 10 exists and is active |
| Confirm trunk state | SW1/SW2 | `show interfaces trunk` | Switch-facing link is trunking if it should carry multiple VLANs |
| Confirm VLAN allowance | SW1/SW2 | `show interfaces trunk` | VLAN 10 is allowed and active on the trunk |
| Confirm root bridge | SW1/SW2/SW3 | `show spanning-tree root` | Intended root bridge is known |
| Confirm detailed VLAN STP state | SW1/SW2/SW3 | `show spanning-tree vlan 10` | Root ID, roles, states, costs, and type are visible |
| Confirm network-port behavior | SW1/SW2 | `show spanning-tree interface Ethernet1/1 detail` | Interface is treated as a network port |
| Confirm BPDU counters | SW1/SW2 | `show spanning-tree interface Ethernet1/1 detail | include BPDU|sent|received` | BPDU sent and received counters increment |
| Confirm no Bridge Assurance inconsistency | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No Bridge Assurance inconsistent ports are listed during normal operation |
| Confirm blocked state during mismatch test | SW1/SW2 | `show spanning-tree inconsistentports` | Interface appears as inconsistent when the far end is not a valid network port |
| Confirm STP logs | SW1/SW2 | `show logging | include Bridge|BRIDGE|BKN|inconsistent|SPANTREE` | Logs show Bridge Assurance block or recovery events |
| Confirm host ports are edge ports | SW1/SW2 | `show spanning-tree interface Ethernet1/10 detail` | Host port is edge, not network |
| Confirm BPDU Guard on edge ports | SW1/SW2 | `show spanning-tree summary | include BPDU Guard|bpduguard` | BPDU Guard default is enabled if intended |
| Confirm no host port is blocked because of network type | SW1/SW2 | `show spanning-tree inconsistentports` | Host-facing ports are not Bridge Assurance inconsistent |
| Confirm physical errors are clean | SW1/SW2 | `show interface Ethernet1/1 counters errors` | Error counters are not climbing |
| Confirm line protocol state | SW1/SW2 | `show interface Ethernet1/1` | Interface is up/up after repair |
| Confirm no err-disabled state | SW1/SW2 | `show interface status err-disabled` | Required links are not err-disabled |
| Confirm no loop symptoms | SW1/SW2/SW3 | `show logging | include MACFLAP|LOOP|SPANTREE` | No MAC flap or loop messages appear |
| Confirm MAC learning is stable | SW1/SW2/SW3 | `show mac address-table dynamic vlan 10` | MAC addresses are learned on expected ports |
| Confirm data-plane recovery | HOSTS | `ping <remote-same-vlan-host-ip>` | Same-VLAN traffic succeeds after recovery |
| Confirm running interface config | SW1/SW2 | `show running-config interface Ethernet1/1` | Interface contains `spanning-tree port type network` |
| Confirm saved interface config | SW1/SW2 | `show startup-config interface Ethernet1/1` | Startup config contains intended network-port setting |

# STP_Bridge_Assurance_Network_Port_Protection_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current Bridge Assurance state before rollback | SW1/SW2/SW3 | `show spanning-tree summary` | STP mode, Bridge Assurance state, and default port type are documented |
| 2 | Record current inconsistent state | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | Current inconsistent ports are documented |
| 3 | Restore any test-changed far-end network port | SW2 | `configure terminal` then `interface Ethernet1/1` then `spanning-tree port type network` then `end` | Far end returns to network-port behavior |
| 4 | Verify inconsistent state clears before removing production protection | SW1/SW2 | `show spanning-tree inconsistentports` | No Bridge Assurance inconsistent ports remain |
| 5 | Enter interface configuration mode on SW1 network port | SW1 | `configure terminal` then `interface Ethernet1/1` | Prompt changes to interface configuration mode |
| 6 | Return SW1 port to normal STP port type if removing Bridge Assurance behavior on that link | SW1 | `spanning-tree port type normal` | Interface no longer operates as a network port |
| 7 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 8 | Enter interface configuration mode on SW2 network port | SW2 | `configure terminal` then `interface Ethernet1/1` | Prompt changes to interface configuration mode |
| 9 | Return SW2 port to normal STP port type if removing Bridge Assurance behavior on that link | SW2 | `spanning-tree port type normal` | Interface no longer operates as a network port |
| 10 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 11 | Remove global network-port default if it was used only for the lab | SW1/SW2/SW3 | `configure terminal` then `no spanning-tree port type network default` then `end` | Default port type returns to normal behavior if supported |
| 12 | Disable Bridge Assurance globally only if required by the rollback plan | SW1/SW2/SW3 | `configure terminal` then `no spanning-tree bridge assurance` then `end` | Bridge Assurance is disabled globally if supported |
| 13 | Verify final STP summary | SW1/SW2/SW3 | `show spanning-tree summary` | Bridge Assurance and port-type defaults match rollback target |
| 14 | Verify final interface port type | SW1/SW2 | `show spanning-tree interface Ethernet1/1 detail` | Interface no longer shows network-port behavior if removed |
| 15 | Verify no inconsistent ports remain | SW1/SW2/SW3 | `show spanning-tree inconsistentports` | No ports are listed |
| 16 | Verify trunk and VLAN state | SW1/SW2 | `show interfaces trunk` | Required VLANs still carry if the trunk remains in use |
| 17 | Verify host reachability | HOSTS | `ping <remote-same-vlan-host-ip>` | Traffic works after rollback |
| 18 | Save rollback | SW1/SW2/SW3 | `copy running-config startup-config` | Startup configuration reflects rollback |

# STP_Bridge_Assurance_Network_Port_Protection_Failure_Checks

| Symptom                                                     | Likely Cause                                                                     | Device      | Command                                                                                         | Fix                                                                                  |                                                                                                    |                                |                                                                     |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | ------------------------------ | ------------------------------------------------------------------- |
| Port becomes Bridge Assurance inconsistent                  | Expected BPDUs are not being received on a network port                          | SW1/SW2     | `show spanning-tree inconsistentports`                                                          | Restore BPDU exchange, port type, trunking, or far-end STP support                   |                                                                                                    |                                |                                                                     |
| Port blocks immediately after configuring network port type | Far end is not configured as a network port or does not support Bridge Assurance | SW1/SW2     | `show spanning-tree interface Ethernet1/1 detail`                                               | Configure `spanning-tree port type network` on both ends or remove network port type |                                                                                                    |                                |                                                                     |
| Host-facing port blocks after global network default        | Host port was made a network port and hosts do not send BPDUs                    | Access SW   | `show spanning-tree inconsistentports`                                                          | Configure host ports as `spanning-tree port type edge`                               |                                                                                                    |                                |                                                                     |
| Bridge Assurance not shown as enabled                       | Global feature is disabled or platform syntax differs                            | SW1/SW2     | `show spanning-tree summary                                                                     | include Bridge Assurance`                                                            | Enable Bridge Assurance globally if supported                                                      |                                |                                                                     |
| Bridge Assurance test does not trigger                      | Both ends still receive valid BPDUs                                              | SW1/SW2     | `show spanning-tree interface Ethernet1/1 detail                                                | include BPDU`                                                                        | Verify the test actually stopped expected BPDUs or mismatched port type                            |                                |                                                                     |
| BPDU counters do not increment                              | BPDU suppression, broken link, or non-STP neighbor                               | SW1/SW2     | `show spanning-tree interface Ethernet1/1 detail                                                | include BPDU`                                                                        | Remove BPDU Filter, fix physical path, or connect to an STP-capable switch                         |                                |                                                                     |
| BPDU Filter is configured on network port                   | Misapplied edge feature suppresses BPDUs                                         | SW1/SW2     | `show running-config interface Ethernet1/1                                                      | include bpdufilter`                                                                  | Remove BPDU Filter from interswitch links                                                          |                                |                                                                     |
| Port is edge instead of network                             | Wrong spanning-tree port type                                                    | SW1/SW2     | `show spanning-tree interface Ethernet1/1 detail`                                               | Configure `spanning-tree port type network`                                          |                                                                                                    |                                |                                                                     |
| Bridge Assurance blocks only one VLAN                       | VLAN-specific STP or trunk state differs                                         | SW1/SW2     | `show spanning-tree vlan <vlan-id>`                                                             | Check that VLAN on trunks, STP state, and BPDU exchange                              |                                                                                                    |                                |                                                                     |
| VLAN not carried on trunk                                   | VLAN missing from allowed list or local VLAN database                            | SW1/SW2     | `show interfaces trunk` and `show vlan brief`                                                   | Add the VLAN and create it locally                                                   |                                                                                                    |                                |                                                                     |
| Native VLAN mismatch appears                                | Trunk native VLAN differs between ends                                           | SW1/SW2     | `show logging                                                                                   | include NATIVE                                                                       | native`                                                                                            | Match native VLAN on both ends |                                                                     |
| Port is err-disabled instead of BA-inconsistent             | BPDU Guard, UDLD, link-flap, or port security triggered                          | SW1/SW2     | `show interface status err-disabled`                                                            | Fix the specific err-disable trigger and shut/no shut the port                       |                                                                                                    |                                |                                                                     |
| UDLD reports unidirectional link                            | One-way physical failure                                                         | SW1/SW2     | `show udld interface Ethernet1/1`                                                               | Replace cable, optics, patch path, or transceiver                                    |                                                                                                    |                                |                                                                     |
| Physical errors are climbing                                | Bad cable, optics, speed, duplex, or hardware path                               | SW1/SW2     | `show interface Ethernet1/1 counters errors`                                                    | Fix the physical layer                                                               |                                                                                                    |                                |                                                                     |
| STP mode is legacy 802.1D                                   | Bridge Assurance is not supported with legacy STP                                | SW1/SW2     | `show spanning-tree summary`                                                                    | Use Rapid PVST+ or MST                                                               |                                                                                                    |                                |                                                                     |
| Loop Guard expected behavior does not match                 | Bridge Assurance and Loop Guard protect different failure models                 | SW1/SW2     | `show running-config interface Ethernet1/1                                                      | include guard`                                                                       | Use Bridge Assurance for network-port BPDU expectation; use Loop Guard for root or alternate ports |                                |                                                                     |
| Root Guard expected behavior does not match                 | Root Guard blocks superior BPDUs, not missing BPDUs                              | SW1/SW2     | `show spanning-tree inconsistentports`                                                          | Use Root Guard only on downstream designated ports that must not become root         |                                                                                                    |                                |                                                                     |
| MAC addresses flap after disabling Bridge Assurance         | Unsafe Layer 2 forwarding or loop was exposed                                    | SW1/SW2/SW3 | `show logging                                                                                   | include MACFLAP                                                                      | LOOP                                                                                               | SPANTREE`                      | Re-enable protection or correct the physical topology and STP state |
| Port remains inconsistent after restoring far end           | BPDUs still not received or far end still mismatched                             | SW1/SW2     | `show spanning-tree interface Ethernet1/1 detail`                                               | Verify both ends are network ports and BPDU counters increment                       |                                                                                                    |                                |                                                                     |
| Traffic fails after recovery                                | Trunk, VLAN, STP, or MAC relearning issue remains                                | SW1/SW2     | `show interfaces trunk`, `show spanning-tree vlan 10`, `show mac address-table dynamic vlan 10` | Fix trunk allowance, STP state, or generate traffic to relearn MACs                  |                                                                                                    |                                |                                                                     |
| Config disappears after reload                              | Configuration was not saved                                                      | SW1/SW2     | `show startup-config interface Ethernet1/1`                                                     | Reapply intended settings and run `copy running-config startup-config`               |                                                                                                    |                                |                                                                     |