Private_VLAN_MultiSwitch_Transport.md
# Private_VLAN_MultiSwitch_Transport

# Private_VLAN_MultiSwitch_Transport_Index

- Private_VLAN_MultiSwitch_Transport_Mental_Model
- Private_VLAN_MultiSwitch_Transport_Configuration_Checklist
- Private_VLAN_MultiSwitch_Transport_Skeleton
- Private_VLAN_MultiSwitch_Transport_Verification_Commands
- Private_VLAN_MultiSwitch_Transport_Rollback
- Private_VLAN_MultiSwitch_Transport_Failure_Checks

# Private_VLAN_MultiSwitch_Transport_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Multi-switch PVLAN | Extends private VLAN isolation across more than one switch |
| Primary VLAN | Parent VLAN that represents the shared subnet and promiscuous gateway side |
| Secondary VLAN | Child VLAN associated with the primary VLAN; can be isolated or community |
| Isolated secondary VLAN | Hosts in this secondary VLAN cannot communicate with each other, even across switches |
| Community secondary VLAN | Hosts in the same community VLAN can communicate with each other, even across switches |
| Promiscuous port | Gateway-facing port that can communicate with all mapped secondary VLANs |
| PVLAN host port | Endpoint-facing port associated with one primary VLAN and one secondary VLAN |
| PVLAN association consistency | Every switch carrying the PVLAN must have the same primary-to-secondary association |
| PVLAN trunk transport | The interswitch link must carry the primary VLAN and all required secondary VLANs |
| Secondary trunk role | On platforms that support it, a private-vlan secondary trunk carries PVLAN host-side traffic between switches |
| Promiscuous trunk role | On platforms that support it, a private-vlan promiscuous trunk carries mapped secondary VLANs toward a router, firewall, or upstream gateway |
| Normal trunk option | Some platforms transport PVLANs across normal 802.1Q trunks, but the primary and secondary VLANs must share the same STP topology |
| STP dependency | Primary and secondary VLANs must not take different STP paths across the same PVLAN transport design |
| Same subnet isolation | PVLAN keeps hosts in the same IP subnet while controlling Layer 2 host-to-host communication |
| Gateway placement | The gateway can sit on one switch as a promiscuous port or SVI, while isolated/community hosts exist on multiple switches |
| Cross-switch isolation test | Isolated host on SW1 should not reach isolated host on SW2, but both should reach the promiscuous gateway |
| Cross-switch community test | Community hosts in the same community VLAN should reach each other across switches |
| Verification rule | Verify VLAN association, interswitch transport, promiscuous mapping, host association, STP state, MAC learning, and the ping matrix |

# Private_VLAN_MultiSwitch_Transport_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm both switches support private VLANs | SW1/SW2 | `show vlan private-vlan` | Command is accepted and PVLAN table is displayed |
| 2 | Confirm interswitch link is physically up | SW1/SW2 | `show interfaces status` | Interswitch interfaces show `connected` |
| 3 | Confirm endpoint-facing links are physically up | SW1/SW2 | `show interfaces status` | Host and gateway ports show `connected` where devices are attached |
| 4 | Confirm existing VLAN database before changes | SW1/SW2 | `show vlan brief` | Existing VLANs are documented |
| 5 | Confirm existing switchport modes before changes | SW1/SW2 | `show interfaces switchport` | Existing access, trunk, and PVLAN states are known |
| 6 | Confirm existing STP state before PVLAN transport | SW1/SW2 | `show spanning-tree vlan 100` | Existing STP behavior is documented if VLAN 100 already exists |
| 7 | Enter configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 8 | Create primary VLAN 100 | SW1 | `vlan 100` | VLAN 100 configuration mode opens |
| 9 | Name primary VLAN 100 | SW1 | `name PVLAN_PRIMARY` | VLAN 100 has a readable name |
| 10 | Mark VLAN 100 as primary | SW1 | `private-vlan primary` | VLAN 100 becomes a PVLAN primary VLAN |
| 11 | Associate secondary VLANs to primary VLAN | SW1 | `private-vlan association 101,102` | VLANs 101 and 102 are associated with VLAN 100 |
| 12 | Exit primary VLAN config | SW1 | `exit` | Prompt returns to global configuration mode |
| 13 | Create isolated secondary VLAN 101 | SW1 | `vlan 101` | VLAN 101 configuration mode opens |
| 14 | Name isolated VLAN 101 | SW1 | `name PVLAN_ISOLATED` | VLAN 101 has a readable name |
| 15 | Mark VLAN 101 as isolated | SW1 | `private-vlan isolated` | VLAN 101 becomes an isolated secondary VLAN |
| 16 | Exit isolated VLAN config | SW1 | `exit` | Prompt returns to global configuration mode |
| 17 | Create community secondary VLAN 102 | SW1 | `vlan 102` | VLAN 102 configuration mode opens |
| 18 | Name community VLAN 102 | SW1 | `name PVLAN_COMMUNITY` | VLAN 102 has a readable name |
| 19 | Mark VLAN 102 as community | SW1 | `private-vlan community` | VLAN 102 becomes a community secondary VLAN |
| 20 | Exit community VLAN config | SW1 | `exit` | Prompt returns to global configuration mode |
| 21 | Configure SW1 gateway-facing promiscuous port | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 22 | Add gateway port description | SW1 | `description PVLAN_PROMISCUOUS_GATEWAY` | Interface purpose is clear |
| 23 | Force Layer 2 switchport behavior if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 24 | Configure promiscuous mode | SW1 | `switchport mode private-vlan promiscuous` | Port becomes a PVLAN promiscuous port |
| 25 | Map primary VLAN to secondary VLANs on gateway port | SW1 | `switchport private-vlan mapping 100 101,102` | Gateway can reach both isolated and community VLANs |
| 26 | Enable gateway-facing port | SW1 | `no shutdown` | Interface is administratively up |
| 27 | Configure SW1 isolated host port | SW1 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 28 | Add isolated host description | SW1 | `description H1_ISOLATED_PVLAN_101` | Interface purpose is clear |
| 29 | Force Layer 2 switchport behavior if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 30 | Configure private VLAN host mode | SW1 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 31 | Associate host port with primary and isolated VLAN | SW1 | `switchport private-vlan host-association 100 101` | Host is assigned to primary 100 and isolated secondary 101 |
| 32 | Enable PortFast on host port | SW1 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 33 | Enable SW1 isolated host port | SW1 | `no shutdown` | Interface is administratively up |
| 34 | Configure SW1 community host port | SW1 | `interface GigabitEthernet0/3` | Prompt changes to interface configuration mode |
| 35 | Add community host description | SW1 | `description H3_COMMUNITY_PVLAN_102` | Interface purpose is clear |
| 36 | Force Layer 2 switchport behavior if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 37 | Configure private VLAN host mode | SW1 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 38 | Associate host port with primary and community VLAN | SW1 | `switchport private-vlan host-association 100 102` | Host is assigned to primary 100 and community secondary 102 |
| 39 | Enable PortFast on host port | SW1 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 40 | Enable SW1 community host port | SW1 | `no shutdown` | Interface is administratively up |
| 41 | Configure SW1 interswitch PVLAN transport port | SW1 | `interface GigabitEthernet0/24` | Prompt changes to interswitch interface |
| 42 | Add transport description | SW1 | `description PVLAN_TRANSPORT_TO_SW2` | Interface purpose is clear |
| 43 | Force Layer 2 switchport behavior if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 44 | Configure PVLAN secondary trunk if supported | SW1 | `switchport mode private-vlan trunk secondary` | Interface carries secondary PVLAN traffic between switches |
| 45 | Associate primary and secondary VLANs on the PVLAN trunk | SW1 | `switchport private-vlan association trunk 100 101,102` | Trunk understands the 100 to 101,102 association |
| 46 | Allow the PVLANs on the private VLAN trunk if supported | SW1 | `switchport private-vlan trunk allowed vlan 100,101,102` | Primary and secondary VLANs are permitted |
| 47 | Bring up SW1 interswitch PVLAN transport | SW1 | `no shutdown` | Interface is administratively up |
| 48 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 49 | Enter configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 50 | Create primary VLAN 100 | SW2 | `vlan 100` | VLAN 100 configuration mode opens |
| 51 | Name primary VLAN 100 | SW2 | `name PVLAN_PRIMARY` | VLAN 100 has a readable name |
| 52 | Mark VLAN 100 as primary | SW2 | `private-vlan primary` | VLAN 100 becomes a PVLAN primary VLAN |
| 53 | Associate secondary VLANs to primary VLAN | SW2 | `private-vlan association 101,102` | VLANs 101 and 102 are associated with VLAN 100 |
| 54 | Exit primary VLAN config | SW2 | `exit` | Prompt returns to global configuration mode |
| 55 | Create isolated secondary VLAN 101 | SW2 | `vlan 101` | VLAN 101 configuration mode opens |
| 56 | Name isolated VLAN 101 | SW2 | `name PVLAN_ISOLATED` | VLAN 101 has a readable name |
| 57 | Mark VLAN 101 as isolated | SW2 | `private-vlan isolated` | VLAN 101 becomes an isolated secondary VLAN |
| 58 | Exit isolated VLAN config | SW2 | `exit` | Prompt returns to global configuration mode |
| 59 | Create community secondary VLAN 102 | SW2 | `vlan 102` | VLAN 102 configuration mode opens |
| 60 | Name community VLAN 102 | SW2 | `name PVLAN_COMMUNITY` | VLAN 102 has a readable name |
| 61 | Mark VLAN 102 as community | SW2 | `private-vlan community` | VLAN 102 becomes a community secondary VLAN |
| 62 | Exit community VLAN config | SW2 | `exit` | Prompt returns to global configuration mode |
| 63 | Configure SW2 isolated host port | SW2 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 64 | Add isolated host description | SW2 | `description H2_ISOLATED_PVLAN_101` | Interface purpose is clear |
| 65 | Force Layer 2 switchport behavior if required | SW2 | `switchport` | Interface is a Layer 2 switchport |
| 66 | Configure private VLAN host mode | SW2 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 67 | Associate host port with primary and isolated VLAN | SW2 | `switchport private-vlan host-association 100 101` | Host is assigned to primary 100 and isolated secondary 101 |
| 68 | Enable PortFast on host port | SW2 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 69 | Enable SW2 isolated host port | SW2 | `no shutdown` | Interface is administratively up |
| 70 | Configure SW2 community host port | SW2 | `interface GigabitEthernet0/3` | Prompt changes to interface configuration mode |
| 71 | Add community host description | SW2 | `description H4_COMMUNITY_PVLAN_102` | Interface purpose is clear |
| 72 | Force Layer 2 switchport behavior if required | SW2 | `switchport` | Interface is a Layer 2 switchport |
| 73 | Configure private VLAN host mode | SW2 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 74 | Associate host port with primary and community VLAN | SW2 | `switchport private-vlan host-association 100 102` | Host is assigned to primary 100 and community secondary 102 |
| 75 | Enable PortFast on host port | SW2 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 76 | Enable SW2 community host port | SW2 | `no shutdown` | Interface is administratively up |
| 77 | Configure SW2 interswitch PVLAN transport port | SW2 | `interface GigabitEthernet0/24` | Prompt changes to interswitch interface |
| 78 | Add transport description | SW2 | `description PVLAN_TRANSPORT_TO_SW1` | Interface purpose is clear |
| 79 | Force Layer 2 switchport behavior if required | SW2 | `switchport` | Interface is a Layer 2 switchport |
| 80 | Configure PVLAN secondary trunk if supported | SW2 | `switchport mode private-vlan trunk secondary` | Interface carries secondary PVLAN traffic between switches |
| 81 | Associate primary and secondary VLANs on the PVLAN trunk | SW2 | `switchport private-vlan association trunk 100 101,102` | Trunk understands the 100 to 101,102 association |
| 82 | Allow the PVLANs on the private VLAN trunk if supported | SW2 | `switchport private-vlan trunk allowed vlan 100,101,102` | Primary and secondary VLANs are permitted |
| 83 | Bring up SW2 interswitch PVLAN transport | SW2 | `no shutdown` | Interface is administratively up |
| 84 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 85 | Verify PVLAN database on both switches | SW1/SW2 | `show vlan private-vlan` | Primary 100 is associated with secondary 101 and 102 on both switches |
| 86 | Verify SW1 promiscuous mapping | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Port is PVLAN promiscuous and maps 100 to 101,102 |
| 87 | Verify SW1 isolated host association | SW1 | `show interfaces GigabitEthernet0/2 switchport` | Port is PVLAN host associated with 100/101 |
| 88 | Verify SW2 isolated host association | SW2 | `show interfaces GigabitEthernet0/2 switchport` | Port is PVLAN host associated with 100/101 |
| 89 | Verify SW1 community host association | SW1 | `show interfaces GigabitEthernet0/3 switchport` | Port is PVLAN host associated with 100/102 |
| 90 | Verify SW2 community host association | SW2 | `show interfaces GigabitEthernet0/3 switchport` | Port is PVLAN host associated with 100/102 |
| 91 | Verify interswitch PVLAN transport switchport state | SW1/SW2 | `show interfaces GigabitEthernet0/24 switchport` | Interface shows PVLAN trunk association or equivalent transport state |
| 92 | Verify STP for primary and secondary VLANs | SW1/SW2 | `show spanning-tree vlan 100,101,102` | Primary and secondary VLANs share safe forwarding behavior |
| 93 | Verify trunk state if using a normal 802.1Q trunk fallback | SW1/SW2 | `show interfaces trunk` | VLANs 100, 101, and 102 are allowed and active |
| 94 | Test isolated host on SW1 to gateway | H1 | `ping 192.0.2.1` | Ping succeeds |
| 95 | Test isolated host on SW2 to gateway | H2 | `ping 192.0.2.1` | Ping succeeds |
| 96 | Test isolated host on SW1 to isolated host on SW2 | H1 | `ping 192.0.2.12` | Ping fails because isolated hosts cannot communicate |
| 97 | Test isolated host to community host across switches | H1 | `ping 192.0.2.14` | Ping fails because isolated and community hosts are separated |
| 98 | Test community host on SW1 to community host on SW2 | H3 | `ping 192.0.2.14` | Ping succeeds because both hosts are in community VLAN 102 |
| 99 | Verify MAC learning after tests | SW1/SW2 | `show mac address-table dynamic vlan 100` | MAC learning is stable and does not flap |
| 100 | Save configuration | SW1/SW2 | `write memory` | PVLAN multi-switch transport survives reload |

# Private_VLAN_MultiSwitch_Transport_Skeleton

SW1 with promiscuous gateway, isolated host, community host, and PVLAN transport to SW2:

configure terminal
!
vlan 100
 name PVLAN_PRIMARY
 private-vlan primary
 private-vlan association 101,102
!
vlan 101
 name PVLAN_ISOLATED
 private-vlan isolated
!
vlan 102
 name PVLAN_COMMUNITY
 private-vlan community
!
interface GigabitEthernet0/1
 description PVLAN_PROMISCUOUS_GATEWAY
 switchport
 switchport mode private-vlan promiscuous
 switchport private-vlan mapping 100 101,102
 no shutdown
!
interface GigabitEthernet0/2
 description H1_ISOLATED_PVLAN_101
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 101
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/3
 description H3_COMMUNITY_PVLAN_102
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 102
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/24
 description PVLAN_TRANSPORT_TO_SW2
 switchport
 switchport mode private-vlan trunk secondary
 switchport private-vlan association trunk 100 101,102
 switchport private-vlan trunk allowed vlan 100,101,102
 no shutdown
!
end
write memory

SW2 with isolated host, community host, and PVLAN transport to SW1:

configure terminal
!
vlan 100
 name PVLAN_PRIMARY
 private-vlan primary
 private-vlan association 101,102
!
vlan 101
 name PVLAN_ISOLATED
 private-vlan isolated
!
vlan 102
 name PVLAN_COMMUNITY
 private-vlan community
!
interface GigabitEthernet0/2
 description H2_ISOLATED_PVLAN_101
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 101
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/3
 description H4_COMMUNITY_PVLAN_102
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 102
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/24
 description PVLAN_TRANSPORT_TO_SW1
 switchport
 switchport mode private-vlan trunk secondary
 switchport private-vlan association trunk 100 101,102
 switchport private-vlan trunk allowed vlan 100,101,102
 no shutdown
!
end
write memory

Normal 802.1Q trunk fallback if PVLAN trunk mode is not supported on the platform:

SW1:

configure terminal
!
interface GigabitEthernet0/24
 description DOT1Q_TRANSPORT_TO_SW2_FOR_PVLAN_PRIMARY_AND_SECONDARIES
 switchport mode trunk
 switchport trunk allowed vlan 100,101,102
 no shutdown
!
end
write memory

SW2:

configure terminal
!
interface GigabitEthernet0/24
 description DOT1Q_TRANSPORT_TO_SW1_FOR_PVLAN_PRIMARY_AND_SECONDARIES
 switchport mode trunk
 switchport trunk allowed vlan 100,101,102
 no shutdown
!
end
write memory

PVLAN promiscuous trunk option for an upstream gateway/firewall trunk, if supported:

configure terminal
!
interface GigabitEthernet0/10
 description PVLAN_PROMISCUOUS_TRUNK_TO_FIREWALL
 switchport
 switchport mode private-vlan trunk promiscuous
 switchport private-vlan mapping trunk 100 101,102
 switchport private-vlan trunk allowed vlan 100,101,102
 no shutdown
!
end
write memory

Switch SVI gateway option on SW1:

configure terminal
!
interface Vlan100
 description PVLAN_PRIMARY_GATEWAY_SVI
 ip address 192.0.2.1 255.255.255.0
 private-vlan mapping 101,102
 no shutdown
!
end
write memory

Endpoint addressing example:

H1 on SW1 isolated:
IP address: 192.0.2.11/24
Default gateway: 192.0.2.1

H2 on SW2 isolated:
IP address: 192.0.2.12/24
Default gateway: 192.0.2.1

H3 on SW1 community:
IP address: 192.0.2.13/24
Default gateway: 192.0.2.1

H4 on SW2 community:
IP address: 192.0.2.14/24
Default gateway: 192.0.2.1

Expected ping matrix:

H1 to gateway 192.0.2.1: permit
H2 to gateway 192.0.2.1: permit
H3 to gateway 192.0.2.1: permit
H4 to gateway 192.0.2.1: permit
H1 to H2: deny
H1 to H3: deny
H1 to H4: deny
H2 to H3: deny
H2 to H4: deny
H3 to H4: permit

# Private_VLAN_MultiSwitch_Transport_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm PVLAN database | SW1/SW2 | `show vlan private-vlan` | Primary 100 is associated with secondary VLANs 101 and 102 |
| Confirm VLAN database | SW1/SW2 | `show vlan brief` | VLANs 100, 101, and 102 exist on both switches |
| Confirm primary VLAN configuration | SW1/SW2 | `show running-config | section vlan 100` | VLAN 100 is primary and associated with 101,102 |
| Confirm isolated VLAN configuration | SW1/SW2 | `show running-config | section vlan 101` | VLAN 101 is isolated |
| Confirm community VLAN configuration | SW1/SW2 | `show running-config | section vlan 102` | VLAN 102 is community |
| Confirm SW1 promiscuous port | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Port is PVLAN promiscuous |
| Confirm SW1 promiscuous mapping | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Mapping shows primary 100 to secondary 101,102 |
| Confirm SW1 isolated host association | SW1 | `show interfaces GigabitEthernet0/2 switchport` | Port is PVLAN host associated to 100/101 |
| Confirm SW2 isolated host association | SW2 | `show interfaces GigabitEthernet0/2 switchport` | Port is PVLAN host associated to 100/101 |
| Confirm SW1 community host association | SW1 | `show interfaces GigabitEthernet0/3 switchport` | Port is PVLAN host associated to 100/102 |
| Confirm SW2 community host association | SW2 | `show interfaces GigabitEthernet0/3 switchport` | Port is PVLAN host associated to 100/102 |
| Confirm interswitch PVLAN transport mode | SW1/SW2 | `show interfaces GigabitEthernet0/24 switchport` | Interface shows private-vlan trunk association or expected transport mode |
| Confirm normal trunk fallback state | SW1/SW2 | `show interfaces trunk` | VLANs 100, 101, and 102 are allowed and active if using normal trunks |
| Confirm primary and secondary STP state | SW1/SW2 | `show spanning-tree vlan 100,101,102` | Primary and secondary VLANs have safe forwarding state |
| Confirm no STP inconsistency | SW1/SW2 | `show spanning-tree inconsistentports` | No ports are listed |
| Confirm no err-disabled ports | SW1/SW2 | `show interfaces status err-disabled` | Required PVLAN ports are not err-disabled |
| Confirm SVI mapping if SW1 routes | SW1 | `show running-config interface Vlan100` | SVI contains `private-vlan mapping 101,102` |
| Confirm SVI status if SW1 routes | SW1 | `show ip interface brief | include Vlan100` | Vlan100 is up/up if active mapped ports exist |
| Confirm gateway reachability from SW1 isolated host | H1 | `ping 192.0.2.1` | Ping succeeds |
| Confirm gateway reachability from SW2 isolated host | H2 | `ping 192.0.2.1` | Ping succeeds |
| Confirm gateway reachability from SW1 community host | H3 | `ping 192.0.2.1` | Ping succeeds |
| Confirm gateway reachability from SW2 community host | H4 | `ping 192.0.2.1` | Ping succeeds |
| Confirm isolated-to-isolated isolation across switches | H1 | `ping 192.0.2.12` | Ping fails |
| Confirm isolated-to-community isolation across switches | H1 | `ping 192.0.2.14` | Ping fails |
| Confirm community-to-community across switches | H3 | `ping 192.0.2.14` | Ping succeeds |
| Confirm ARP toward gateway | H1/H2/H3/H4 | `show arp` or `ip neigh` | Gateway MAC is learned |
| Confirm MAC learning in primary VLAN | SW1/SW2 | `show mac address-table dynamic vlan 100` | MACs are learned on expected ports if platform reports primary view |
| Confirm MAC learning in isolated VLAN | SW1/SW2 | `show mac address-table dynamic vlan 101` | Isolated host MACs appear if platform reports secondary VLANs separately |
| Confirm MAC learning in community VLAN | SW1/SW2 | `show mac address-table dynamic vlan 102` | Community host MACs appear if platform reports secondary VLANs separately |
| Confirm no MAC flapping | SW1/SW2 | `show logging | include MACFLAP|LOOP|SPANTREE` | No active loop or MAC flap messages appear |
| Confirm saved PVLAN configuration | SW1/SW2 | `show startup-config | include private-vlan` | Startup config contains intended PVLAN commands |

# Private_VLAN_MultiSwitch_Transport_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record PVLAN state before rollback | SW1/SW2 | `show vlan private-vlan` | Current PVLAN associations are documented |
| 2 | Record interswitch transport state | SW1/SW2 | `show interfaces GigabitEthernet0/24 switchport` | Current PVLAN trunk or trunk state is documented |
| 3 | Record host-port associations | SW1/SW2 | `show interfaces switchport | include Name|private-vlan|Operational Mode` | Current PVLAN host and promiscuous roles are documented |
| 4 | Enter configuration mode on SW1 | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 5 | Reset SW1 promiscuous gateway port | SW1 | `default interface GigabitEthernet0/1` | Promiscuous mapping is removed |
| 6 | Reset SW1 isolated host port | SW1 | `default interface GigabitEthernet0/2` | PVLAN host association is removed |
| 7 | Reset SW1 community host port | SW1 | `default interface GigabitEthernet0/3` | PVLAN host association is removed |
| 8 | Reset SW1 interswitch PVLAN transport | SW1 | `default interface GigabitEthernet0/24` | PVLAN trunk or normal trunk config is removed |
| 9 | Remove SW1 SVI mapping if configured | SW1 | `interface Vlan100` then `no private-vlan mapping 101,102` | Secondary VLAN mapping is removed from the SVI |
| 10 | Remove SW1 SVI IP if the lab is being fully cleared | SW1 | `no ip address` | Gateway IP is removed |
| 11 | Shut SW1 SVI if fully clearing the lab | SW1 | `shutdown` | SVI is administratively down |
| 12 | Exit SW1 SVI configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 13 | Remove SW1 primary-to-secondary association | SW1 | `vlan 100` then `no private-vlan association 101,102` | PVLAN association is removed |
| 14 | Exit SW1 VLAN configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 15 | Remove SW1 isolated secondary VLAN | SW1 | `no vlan 101` | VLAN 101 is deleted |
| 16 | Remove SW1 community secondary VLAN | SW1 | `no vlan 102` | VLAN 102 is deleted |
| 17 | Remove SW1 primary VLAN | SW1 | `no vlan 100` | VLAN 100 is deleted |
| 18 | Exit SW1 configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Enter configuration mode on SW2 | SW2 | `configure terminal` | Prompt changes to global configuration mode |
| 20 | Reset SW2 isolated host port | SW2 | `default interface GigabitEthernet0/2` | PVLAN host association is removed |
| 21 | Reset SW2 community host port | SW2 | `default interface GigabitEthernet0/3` | PVLAN host association is removed |
| 22 | Reset SW2 interswitch PVLAN transport | SW2 | `default interface GigabitEthernet0/24` | PVLAN trunk or normal trunk config is removed |
| 23 | Remove SW2 primary-to-secondary association | SW2 | `vlan 100` then `no private-vlan association 101,102` | PVLAN association is removed |
| 24 | Exit SW2 VLAN configuration | SW2 | `exit` | Prompt returns to global configuration mode |
| 25 | Remove SW2 isolated secondary VLAN | SW2 | `no vlan 101` | VLAN 101 is deleted |
| 26 | Remove SW2 community secondary VLAN | SW2 | `no vlan 102` | VLAN 102 is deleted |
| 27 | Remove SW2 primary VLAN | SW2 | `no vlan 100` | VLAN 100 is deleted |
| 28 | Exit SW2 configuration mode | SW2 | `end` | Prompt returns to privileged EXEC mode |
| 29 | Verify PVLAN cleanup | SW1/SW2 | `show vlan private-vlan` | Test PVLAN associations are gone |
| 30 | Verify VLAN cleanup | SW1/SW2 | `show vlan brief` | VLANs 100, 101, and 102 are removed if full cleanup was intended |
| 31 | Verify interface cleanup | SW1/SW2 | `show running-config interface GigabitEthernet0/24` | Transport port no longer has PVLAN config |
| 32 | Save rollback | SW1/SW2 | `write memory` | Startup config reflects rollback |

# Private_VLAN_MultiSwitch_Transport_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| PVLAN commands are rejected | Platform or image does not support private VLANs or PVLAN trunk mode | SW1/SW2 | `show vlan private-vlan` | Use supported switch platform or use normal trunk fallback if supported |
| Hosts on different switches cannot reach gateway | Interswitch link is not carrying primary and secondary VLANs | SW1/SW2 | `show interfaces GigabitEthernet0/24 switchport` and `show interfaces trunk` | Correct PVLAN trunk or normal trunk allowed VLANs |
| SW2 isolated host cannot reach SW1 gateway | PVLAN association missing on SW2 | SW2 | `show vlan private-vlan` | Configure the same primary and secondary VLAN association on SW2 |
| SW2 isolated host cannot reach SW1 gateway | SW1 promiscuous mapping missing secondary VLAN | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Map primary 100 to secondary 101,102 on the promiscuous port |
| Isolated hosts across switches can ping each other | Ports are not actually in isolated PVLAN host mode | SW1/SW2 | `show interfaces GigabitEthernet0/2 switchport` | Configure host mode and association 100/101 on both isolated ports |
| Community hosts across switches cannot ping | Community VLAN not transported or host associations differ | SW1/SW2 | `show interfaces trunk`, `show vlan private-vlan`, `show interfaces switchport` | Carry VLAN 102 and associate both community hosts to 100/102 |
| Isolated host can reach community host | Wrong host association or normal access VLAN used accidentally | SW1/SW2 | `show interfaces switchport` | Correct PVLAN host association and remove normal access-mode configuration |
| VLAN exists on SW1 but not SW2 | VLAN database not consistent across switches | SW1/SW2 | `show vlan brief` | Create primary and secondary VLANs on every PVLAN switch |
| Primary and secondary association differs between switches | PVLAN mapping was not duplicated | SW1/SW2 | `show vlan private-vlan` | Make primary-to-secondary association identical |
| Interswitch PVLAN trunk does not form | PVLAN trunk syntax unsupported or far end mismatch | SW1/SW2 | `show interfaces GigabitEthernet0/24 switchport` | Use matching PVLAN trunk mode or normal trunk fallback |
| Normal trunk fallback fails | Primary and secondary VLANs are not all allowed | SW1/SW2 | `show interfaces trunk` | Allow VLANs 100, 101, and 102 |
| Normal trunk fallback causes odd behavior | Primary and secondary VLANs have different STP topology | SW1/SW2 | `show spanning-tree vlan 100,101,102` | Make STP topology identical for primary and secondary VLANs |
| STP blocks one secondary VLAN differently | Per-VLAN STP root or cost differs | SW1/SW2 | `show spanning-tree vlan 100,101,102` | Align STP root, cost, and trunk path for all PVLAN VLANs |
| Port-channel config becomes inactive | Port-channel member was configured as PVLAN port | SW1/SW2 | `show etherchannel summary` | Do not make port-channel member ports PVLAN host or promiscuous ports |
| Promiscuous gateway cannot ARP for remote hosts | Promiscuous mapping or PVLAN transport is incomplete | SW1/SW2 | `show interfaces switchport`, `show vlan private-vlan` | Fix gateway mapping and interswitch PVLAN transport |
| Switch SVI gateway cannot reach secondary hosts | SVI lacks private VLAN mapping | SW1 | `show running-config interface Vlan100` | Add `private-vlan mapping 101,102` under interface Vlan100 |
| SVI is down | No active primary or associated secondary ports are up | SW1 | `show ip interface brief | include Vlan100` | Bring up mapped PVLAN host or promiscuous ports |
| Hosts have wrong default gateway | Endpoint IP configuration issue | HOSTS | `show arp`, `ip neigh`, or host IP settings | Use the PVLAN gateway IP as default gateway |
| Hosts are in different subnets | PVLAN is same-subnet Layer 2 isolation, not subnet routing | HOSTS | Endpoint IP settings | Put test hosts and gateway in the same subnet |
| DHCP fails across PVLAN transport | DHCP snooping, relay, or server path does not account for primary and secondary VLANs | SW1/SW2 | `show ip dhcp snooping` | Configure DHCP handling for PVLANs as required |
| MAC addresses flap | Loop, STP mismatch, or unsafe redundant trunking | SW1/SW2 | `show logging | include MACFLAP|LOOP|SPANTREE` | Fix STP topology and avoid different paths for PVLAN primary and secondary VLANs |
| MAC addresses do not appear | No traffic generated or platform reports PVLAN MACs under different VLAN view | SW1/SW2 | `show mac address-table dynamic vlan 100`, `show mac address-table dynamic vlan 101`, `show mac address-table dynamic vlan 102` | Generate ARP or ping traffic and check primary and secondary VLAN views |
| Port is err-disabled | BPDU Guard, port security, UDLD, or link-flap protection triggered | SW1/SW2 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the port |
| Host port delays traffic after link up | PortFast missing on endpoint-facing PVLAN host port | SW1/SW2 | `show spanning-tree interface GigabitEthernet0/2 detail` | Add `spanning-tree portfast` on true host-facing ports |
| Trunk allowed list overwrote other VLANs | Replacement syntax was used instead of add/remove | SW1/SW2 | `show running-config interface GigabitEthernet0/24` | Restore full allowed list or use add/remove carefully |
| Native VLAN mismatch appears | Interswitch trunk native VLAN differs | SW1/SW2 | `show logging | include NATIVE|native` | Match native VLANs if using normal trunk fallback |
| Configuration disappears after reload | Configuration was not saved | SW1/SW2 | `show startup-config | include private-vlan` | Reapply PVLAN configuration and run `write memory` |
