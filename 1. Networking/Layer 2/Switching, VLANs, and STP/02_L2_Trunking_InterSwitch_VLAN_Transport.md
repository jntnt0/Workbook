
L2_Trunking_InterSwitch_VLAN_Transport.md
# L2_Trunking_InterSwitch_VLAN_Transport

# L2_Trunking_InterSwitch_VLAN_Transport_Index

- L2_Trunking_InterSwitch_VLAN_Transport_Mental_Model
- L2_Trunking_InterSwitch_VLAN_Transport_Configuration_Checklist
- L2_Trunking_InterSwitch_VLAN_Transport_Skeleton
- L2_Trunking_InterSwitch_VLAN_Transport_Verification_Commands
- L2_Trunking_InterSwitch_VLAN_Transport_Rollback
- L2_Trunking_InterSwitch_VLAN_Transport_Failure_Checks

# L2_Trunking_InterSwitch_VLAN_Transport_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Trunk link | A switch-to-switch link that carries multiple VLANs over one physical or logical interface |
| 802.1Q tag | Ethernet frames are tagged with a VLAN ID so the receiving switch knows which VLAN the frame belongs to |
| Native VLAN | Untagged traffic on an 802.1Q trunk is placed into the native VLAN |
| Native VLAN match | Both ends of the trunk must use the same native VLAN to avoid misforwarding and CDP/STP warnings |
| Allowed VLAN list | The trunk only carries VLANs permitted in the allowed VLAN list |
| Active VLAN | A VLAN must exist locally and be active before it appears as allowed and active on the trunk |
| STP per VLAN | A VLAN can be allowed on the trunk but still not forwarding if STP blocks it |
| DTP | Dynamic Trunking Protocol can negotiate trunks, but static trunking is cleaner for predictable labs and production baselines |
| Encapsulation selection | Some platforms require `switchport trunk encapsulation dot1q` before `switchport mode trunk`; newer platforms may not support the encapsulation command |
| Trunk pruning | Removing unused VLANs from trunks limits unnecessary flooding and reduces blast radius |
| Access versus trunk boundary | Host-facing ports are access ports; switch-to-switch links are normally trunks |
| Same VLAN across switches | Hosts in the same VLAN on different switches require the VLAN to exist on both switches and be carried across the trunk |

# L2_Trunking_InterSwitch_VLAN_Transport_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the interswitch ports are physically up or connected in the lab topology | SW1/SW2 | `show interfaces status` | The intended interswitch interfaces show `connected` or become connected after cabling |
| 2 | Confirm the target interfaces are Layer 2 switchports | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | `Switchport: Enabled` appears |
| 3 | Enter configuration mode | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 4 | Create VLAN 10 | SW1 | `vlan 10` | VLAN 10 is present in the local VLAN database |
| 5 | Name VLAN 10 | SW1 | `name USERS` | VLAN 10 has a readable name |
| 6 | Create VLAN 20 | SW1 | `vlan 20` | VLAN 20 is present in the local VLAN database |
| 7 | Name VLAN 20 | SW1 | `name SERVERS` | VLAN 20 has a readable name |
| 8 | Create unused native VLAN | SW1 | `vlan 999` | VLAN 999 exists for native VLAN use |
| 9 | Name unused native VLAN | SW1 | `name NATIVE_UNUSED` | Native VLAN purpose is obvious |
| 10 | Select the interswitch interface | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 11 | Add a trunk description | SW1 | `description TRUNK_TO_SW2_Gi0/1` | Interface purpose is visible in show commands |
| 12 | Select 802.1Q encapsulation if required by the platform | SW1 | `switchport trunk encapsulation dot1q` | Platform accepts dot1q encapsulation, or the command is unavailable because dot1q is the only supported option |
| 13 | Force the interface to trunk mode | SW1 | `switchport mode trunk` | Interface is statically configured as a trunk |
| 14 | Disable DTP negotiation on the static trunk | SW1 | `switchport nonegotiate` | Trunk does not depend on DTP negotiation |
| 15 | Set the native VLAN | SW1 | `switchport trunk native vlan 999` | Untagged traffic is associated with VLAN 999 |
| 16 | Restrict the allowed VLAN list | SW1 | `switchport trunk allowed vlan 10,20,999` | Only VLANs 10, 20, and 999 are allowed across the trunk |
| 17 | Administratively enable the trunk | SW1 | `no shutdown` | Trunk interface is not shut down |
| 18 | Exit SW1 interface configuration | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Enter configuration mode | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 20 | Create VLAN 10 | SW2 | `vlan 10` | VLAN 10 is present in the local VLAN database |
| 21 | Name VLAN 10 | SW2 | `name USERS` | VLAN 10 has a readable name |
| 22 | Create VLAN 20 | SW2 | `vlan 20` | VLAN 20 is present in the local VLAN database |
| 23 | Name VLAN 20 | SW2 | `name SERVERS` | VLAN 20 has a readable name |
| 24 | Create unused native VLAN | SW2 | `vlan 999` | VLAN 999 exists for native VLAN use |
| 25 | Name unused native VLAN | SW2 | `name NATIVE_UNUSED` | Native VLAN purpose is obvious |
| 26 | Select the interswitch interface | SW2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 27 | Add a trunk description | SW2 | `description TRUNK_TO_SW1_Gi0/1` | Interface purpose is visible in show commands |
| 28 | Select 802.1Q encapsulation if required by the platform | SW2 | `switchport trunk encapsulation dot1q` | Platform accepts dot1q encapsulation, or the command is unavailable because dot1q is the only supported option |
| 29 | Force the interface to trunk mode | SW2 | `switchport mode trunk` | Interface is statically configured as a trunk |
| 30 | Disable DTP negotiation on the static trunk | SW2 | `switchport nonegotiate` | Trunk does not depend on DTP negotiation |
| 31 | Set the native VLAN to match SW1 | SW2 | `switchport trunk native vlan 999` | Native VLAN matches the far end |
| 32 | Restrict the allowed VLAN list to match SW1 | SW2 | `switchport trunk allowed vlan 10,20,999` | Allowed VLANs match the far end |
| 33 | Administratively enable the trunk | SW2 | `no shutdown` | Trunk interface is not shut down |
| 34 | Exit SW2 interface configuration | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 35 | Verify both switches show the trunk as trunking | SW1/SW2 | `show interfaces trunk` | Port status shows `trunking` and encapsulation shows `802.1q` |
| 36 | Verify allowed and active VLANs | SW1/SW2 | `show interfaces trunk` | VLANs 10, 20, and 999 appear under allowed and active VLANs |
| 37 | Verify native VLAN consistency | SW1/SW2 | `show interfaces trunk` | Native VLAN is 999 on both ends |
| 38 | Verify switchport operational mode | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | Operational mode is trunk |
| 39 | Verify STP forwarding state per VLAN | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1` | Required VLANs are forwarding or intentionally blocked by STP |
| 40 | Verify local VLAN database | SW1/SW2 | `show vlan brief` | VLANs 10, 20, and 999 exist and are active |
| 41 | Verify same-VLAN host reachability across switches | H1/H2 | `ping <remote-same-vlan-host-ip>` | Hosts in the same VLAN on opposite switches can reach each other |
| 42 | Verify MAC learning across the trunk | SW1/SW2 | `show mac address-table dynamic vlan 10` | Remote host MAC addresses are learned through the trunk interface |
| 43 | Save the configuration | SW1/SW2 | `write memory` | Trunk configuration survives reload |

# L2_Trunking_InterSwitch_VLAN_Transport_Skeleton

SW1:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name SERVERS
vlan 999
 name NATIVE_UNUSED
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW2_Gi0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,999
 no shutdown
!
end
write memory

SW2:

configure terminal
!
vlan 10
 name USERS
vlan 20
 name SERVERS
vlan 999
 name NATIVE_UNUSED
!
interface GigabitEthernet0/1
 description TRUNK_TO_SW1_Gi0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,999
 no shutdown
!
end
write memory

If the platform rejects encapsulation selection because dot1q is the only supported trunk encapsulation, omit this line:

switchport trunk encapsulation dot1q

If adding VLANs to an existing production trunk, do not overwrite the whole allowed list. Use this instead:

configure terminal
interface GigabitEthernet0/1
 switchport trunk allowed vlan add 10,20
end
write memory

If removing VLANs from an existing trunk, use this:

configure terminal
interface GigabitEthernet0/1
 switchport trunk allowed vlan remove 10,20
end
write memory

# L2_Trunking_InterSwitch_VLAN_Transport_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm physical link state | SW1/SW2 | `show interfaces status` | Trunk interface shows `connected` |
| Confirm port is operating as trunk | SW1/SW2 | `show interfaces trunk` | Interface appears in trunk output with status `trunking` |
| Confirm trunk encapsulation | SW1/SW2 | `show interfaces trunk` | Encapsulation shows `802.1q` |
| Confirm native VLAN | SW1/SW2 | `show interfaces trunk` | Native VLAN is 999 on both ends |
| Confirm allowed VLAN list | SW1/SW2 | `show interfaces trunk` | VLANs 10, 20, and 999 appear under allowed VLANs |
| Confirm allowed and active VLANs | SW1/SW2 | `show interfaces trunk` | VLANs 10, 20, and 999 appear under allowed and active VLANs |
| Confirm forwarding VLANs | SW1/SW2 | `show interfaces trunk` | Required VLANs appear under forwarding and not pruned |
| Confirm detailed switchport state | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | Administrative and operational mode are trunk |
| Confirm DTP is disabled | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | Negotiation of trunking is off after `switchport nonegotiate` |
| Confirm VLAN database | SW1/SW2 | `show vlan brief` | VLANs 10, 20, and 999 exist locally |
| Confirm STP state on trunk | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1` | VLANs are forwarding or intentionally blocked by STP |
| Confirm STP detail on trunk | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1 detail` | BPDU counters increment and trunk participates in STP |
| Confirm CDP neighbor across trunk | SW1/SW2 | `show cdp neighbors` | The far-end switch appears on the expected interface |
| Confirm LLDP neighbor if enabled | SW1/SW2 | `show lldp neighbors` | The far-end switch appears on the expected interface |
| Confirm MAC learning for VLAN 10 | SW1/SW2 | `show mac address-table dynamic vlan 10` | Remote MACs are learned through the trunk |
| Confirm MAC learning for VLAN 20 | SW1/SW2 | `show mac address-table dynamic vlan 20` | Remote MACs are learned through the trunk |
| Confirm host reachability in VLAN 10 | H1/H2 | `ping <remote-vlan-10-host-ip>` | Same-VLAN hosts across the trunk can reach each other |
| Confirm host reachability in VLAN 20 | H3/H4 | `ping <remote-vlan-20-host-ip>` | Same-VLAN hosts across the trunk can reach each other |
| Confirm no native VLAN mismatch | SW1/SW2 | `show logging | include NATIVE|native|CDP-4-NATIVE_VLAN_MISMATCH` | No native VLAN mismatch messages appear |
| Confirm final interface config | SW1/SW2 | `show running-config interface GigabitEthernet0/1` | Trunk mode, native VLAN, allowed VLANs, and nonegotiate are present |

# L2_Trunking_InterSwitch_VLAN_Transport_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter configuration mode | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 2 | Reset the SW1 trunk interface to defaults | SW1 | `default interface GigabitEthernet0/1` | Trunk-specific configuration is removed |
| 3 | Remove test VLAN 10 if no longer needed | SW1 | `no vlan 10` | VLAN 10 is deleted locally |
| 4 | Remove test VLAN 20 if no longer needed | SW1 | `no vlan 20` | VLAN 20 is deleted locally |
| 5 | Remove unused native VLAN if no longer needed | SW1 | `no vlan 999` | VLAN 999 is deleted locally |
| 6 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 7 | Save rollback | SW1 | `write memory` | Startup configuration reflects rollback |
| 8 | Enter configuration mode | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 9 | Reset the SW2 trunk interface to defaults | SW2 | `default interface GigabitEthernet0/1` | Trunk-specific configuration is removed |
| 10 | Remove test VLAN 10 if no longer needed | SW2 | `no vlan 10` | VLAN 10 is deleted locally |
| 11 | Remove test VLAN 20 if no longer needed | SW2 | `no vlan 20` | VLAN 20 is deleted locally |
| 12 | Remove unused native VLAN if no longer needed | SW2 | `no vlan 999` | VLAN 999 is deleted locally |
| 13 | Exit configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 14 | Save rollback | SW2 | `write memory` | Startup configuration reflects rollback |
| 15 | Verify trunk removal | SW1/SW2 | `show interfaces trunk` | The interface no longer appears as a trunk |
| 16 | Verify VLAN cleanup | SW1/SW2 | `show vlan brief` | Removed VLANs no longer appear |

# L2_Trunking_InterSwitch_VLAN_Transport_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Interface does not show as trunk | Port is still access mode or dynamic negotiation failed | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | Configure `switchport mode trunk` on both ends |
| `switchport mode trunk` is rejected | Encapsulation is still auto on older IOS platforms | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | Configure `switchport trunk encapsulation dot1q` before trunk mode |
| Trunk works on one side only | Far end is not configured as trunk | SW1/SW2 | `show interfaces trunk` | Configure matching trunk mode on the far end |
| Trunk forms but VLAN traffic fails | VLAN missing from allowed list | SW1/SW2 | `show interfaces trunk` | Add the VLAN with `switchport trunk allowed vlan add <vlan-id>` |
| VLAN is allowed but not active | VLAN does not exist locally | SW1/SW2 | `show vlan brief` | Create the VLAN on both switches |
| VLAN is active but not forwarding | STP is blocking that VLAN on the trunk | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/1` | Validate STP topology, root placement, and expected blocked ports |
| Native VLAN mismatch warning appears | Native VLAN differs between trunk ends | SW1/SW2 | `show interfaces trunk` | Configure the same native VLAN on both ends |
| Untagged traffic leaks into wrong VLAN | Native VLAN is used for host traffic or mismatched | SW1/SW2 | `show interfaces trunk` | Use an unused native VLAN and keep it consistent |
| DTP unexpectedly forms a trunk | Dynamic mode left enabled | SW1/SW2 | `show interfaces GigabitEthernet0/1 switchport` | Use static trunk mode and `switchport nonegotiate` |
| Existing VLANs stop working after change | Allowed VLAN list was overwritten | SW1/SW2 | `show running-config interface GigabitEthernet0/1` | Restore full allowed list or use `add` and `remove` keywords |
| MACs are not learned across trunk | VLAN not crossing trunk or no host traffic | SW1/SW2 | `show mac address-table dynamic vlan <vlan-id>` | Verify VLAN allowed, active, forwarding, then generate ARP or ping traffic |
| Hosts in same VLAN across switches cannot ping | VLAN missing, trunk blocked, or endpoint addressing issue | SW1/SW2/HOSTS | `show interfaces trunk` | Verify VLAN exists, is allowed, is forwarding, and hosts share the correct subnet |
| Trunk interface is err-disabled | Protection feature or link fault disabled the port | SW1/SW2 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the interface |
| CDP shows native VLAN mismatch | Native VLAN does not match between neighbors | SW1/SW2 | `show cdp neighbors detail` | Match `switchport trunk native vlan <vlan-id>` on both ends |
| STP topology changes after trunk enablement | New trunk introduced redundant Layer 2 path | SW1/SW2 | `show spanning-tree vlan <vlan-id>` | Confirm intended root bridge and expected blocked ports |
| VLAN appears under allowed but not under forwarding | VLAN is pruned or blocked | SW1/SW2 | `show interfaces trunk` | Check STP state, pruning behavior, and VLAN existence |
| `switchport nonegotiate` breaks trunk | Far end depends on DTP dynamic negotiation | SW1/SW2 | `show interfaces trunk` | Statically configure trunk mode on both ends before disabling DTP |
| Port shows routed instead of trunk | Interface is configured as Layer 3 | SW1/SW2 | `show interfaces status` | Apply `switchport`, then configure trunk settings |
