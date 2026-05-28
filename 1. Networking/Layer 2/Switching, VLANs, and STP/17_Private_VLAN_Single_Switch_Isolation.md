Private_VLAN_Single_Switch_Isolation.md
# Private_VLAN_Single_Switch_Isolation

# Private_VLAN_Single_Switch_Isolation_Index

- Private_VLAN_Single_Switch_Isolation_Mental_Model
- Private_VLAN_Single_Switch_Isolation_Configuration_Checklist
- Private_VLAN_Single_Switch_Isolation_Skeleton
- Private_VLAN_Single_Switch_Isolation_Verification_Commands
- Private_VLAN_Single_Switch_Isolation_Rollback
- Private_VLAN_Single_Switch_Isolation_Failure_Checks

# Private_VLAN_Single_Switch_Isolation_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Private VLAN | Splits one Layer 2 VLAN/subnet into smaller Layer 2 communication zones |
| Primary VLAN | The parent VLAN that represents the main subnet and carries the promiscuous gateway side |
| Secondary VLAN | A child VLAN associated to the primary VLAN |
| Isolated VLAN | Secondary VLAN where hosts cannot talk to each other, only to promiscuous ports |
| Community VLAN | Secondary VLAN where hosts can talk to other hosts in the same community and to promiscuous ports |
| Promiscuous port | Port that can communicate with all mapped secondary VLANs, usually the router, firewall, or gateway |
| Host port | PVLAN user/server-facing port assigned to a primary plus secondary VLAN pair |
| Host association | Binds a PVLAN host port to one primary VLAN and one secondary VLAN |
| Promiscuous mapping | Maps the promiscuous port to the secondary VLANs it is allowed to reach |
| Same subnet isolation | PVLAN isolates hosts at Layer 2 while keeping them in the same IP subnet |
| Gateway reachability | Isolated and community hosts should still reach the promiscuous gateway |
| Host-to-host isolation | Isolated hosts should not communicate with other isolated hosts |
| Community behavior | Community hosts can communicate within the same community, but not with isolated hosts or other communities |
| Single-switch scope | This note validates PVLAN behavior on one switch only, without PVLAN trunk propagation |
| SVI mapping | If the switch SVI is the gateway, map secondary VLANs under the primary SVI |
| Routed gateway option | If an external router or firewall is the gateway, place its port in promiscuous mode and map the secondary VLANs |
| Verification rule | Test gateway reachability, isolated-to-isolated denial, isolated-to-community denial, and community-to-community allowance |

# Private_VLAN_Single_Switch_Isolation_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the switch platform supports private VLANs | SW1 | `show vlan private-vlan` | Command is accepted or PVLAN table is displayed |
| 2 | Confirm target interfaces exist and are connected | SW1 | `show interfaces status` | Gateway and host-facing ports are visible and expected links are connected |
| 3 | Confirm current VLAN database before changes | SW1 | `show vlan brief` | Existing VLANs are known before PVLAN configuration |
| 4 | Confirm current switchport state before changes | SW1 | `show interfaces switchport` | Ports are not already in conflicting modes |
| 5 | Enter global configuration mode | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 6 | Create the primary VLAN | SW1 | `vlan 100` | Switch enters VLAN configuration mode for VLAN 100 |
| 7 | Name the primary VLAN | SW1 | `name PVLAN_PRIMARY` | VLAN 100 has a readable name |
| 8 | Designate VLAN 100 as the primary private VLAN | SW1 | `private-vlan primary` | VLAN 100 becomes a PVLAN primary VLAN |
| 9 | Associate secondary VLANs to the primary VLAN | SW1 | `private-vlan association 101,102` | VLANs 101 and 102 are associated to primary VLAN 100 |
| 10 | Exit primary VLAN configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 11 | Create the isolated secondary VLAN | SW1 | `vlan 101` | Switch enters VLAN configuration mode for VLAN 101 |
| 12 | Name the isolated secondary VLAN | SW1 | `name PVLAN_ISOLATED` | VLAN 101 has a readable name |
| 13 | Designate VLAN 101 as isolated | SW1 | `private-vlan isolated` | VLAN 101 becomes an isolated secondary VLAN |
| 14 | Exit isolated VLAN configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 15 | Create the community secondary VLAN | SW1 | `vlan 102` | Switch enters VLAN configuration mode for VLAN 102 |
| 16 | Name the community secondary VLAN | SW1 | `name PVLAN_COMMUNITY` | VLAN 102 has a readable name |
| 17 | Designate VLAN 102 as community | SW1 | `private-vlan community` | VLAN 102 becomes a community secondary VLAN |
| 18 | Exit community VLAN configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 19 | Configure the promiscuous gateway-facing port | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 20 | Add gateway port description | SW1 | `description PVLAN_PROMISCUOUS_GATEWAY` | Interface purpose is visible in show commands |
| 21 | Force the gateway port to Layer 2 mode if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 22 | Configure the gateway port as PVLAN promiscuous | SW1 | `switchport mode private-vlan promiscuous` | Port becomes a promiscuous PVLAN port |
| 23 | Map the promiscuous port to the primary and secondary VLANs | SW1 | `switchport private-vlan mapping 100 101,102` | Gateway port can reach isolated and community secondary VLANs |
| 24 | Enable the gateway port | SW1 | `no shutdown` | Gateway-facing port is administratively up |
| 25 | Configure first isolated host port | SW1 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 26 | Add isolated host description | SW1 | `description H1_ISOLATED_PVLAN_101` | Interface purpose is visible |
| 27 | Force first isolated host port to Layer 2 mode if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 28 | Configure first isolated host port mode | SW1 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 29 | Associate first isolated host with primary and isolated VLAN | SW1 | `switchport private-vlan host-association 100 101` | Host is placed into isolated secondary VLAN 101 under primary VLAN 100 |
| 30 | Enable PortFast on first isolated host port | SW1 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 31 | Enable first isolated host port | SW1 | `no shutdown` | Interface is administratively up |
| 32 | Configure second isolated host port | SW1 | `interface GigabitEthernet0/3` | Prompt changes to interface configuration mode |
| 33 | Add isolated host description | SW1 | `description H2_ISOLATED_PVLAN_101` | Interface purpose is visible |
| 34 | Force second isolated host port to Layer 2 mode if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 35 | Configure second isolated host port mode | SW1 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 36 | Associate second isolated host with primary and isolated VLAN | SW1 | `switchport private-vlan host-association 100 101` | Host is placed into isolated secondary VLAN 101 under primary VLAN 100 |
| 37 | Enable PortFast on second isolated host port | SW1 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 38 | Enable second isolated host port | SW1 | `no shutdown` | Interface is administratively up |
| 39 | Configure first community host port | SW1 | `interface GigabitEthernet0/4` | Prompt changes to interface configuration mode |
| 40 | Add community host description | SW1 | `description H3_COMMUNITY_PVLAN_102` | Interface purpose is visible |
| 41 | Force first community host port to Layer 2 mode if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 42 | Configure first community host port mode | SW1 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 43 | Associate first community host with primary and community VLAN | SW1 | `switchport private-vlan host-association 100 102` | Host is placed into community secondary VLAN 102 under primary VLAN 100 |
| 44 | Enable PortFast on first community host port | SW1 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 45 | Enable first community host port | SW1 | `no shutdown` | Interface is administratively up |
| 46 | Configure second community host port | SW1 | `interface GigabitEthernet0/5` | Prompt changes to interface configuration mode |
| 47 | Add community host description | SW1 | `description H4_COMMUNITY_PVLAN_102` | Interface purpose is visible |
| 48 | Force second community host port to Layer 2 mode if required | SW1 | `switchport` | Interface is a Layer 2 switchport |
| 49 | Configure second community host port mode | SW1 | `switchport mode private-vlan host` | Port becomes a PVLAN host port |
| 50 | Associate second community host with primary and community VLAN | SW1 | `switchport private-vlan host-association 100 102` | Host is placed into community secondary VLAN 102 under primary VLAN 100 |
| 51 | Enable PortFast on second community host port | SW1 | `spanning-tree portfast` | Host port transitions quickly to forwarding |
| 52 | Enable second community host port | SW1 | `no shutdown` | Interface is administratively up |
| 53 | Configure primary SVI if SW1 is the default gateway | SW1 | `interface Vlan100` | Prompt changes to SVI configuration mode |
| 54 | Add primary SVI description | SW1 | `description PVLAN_PRIMARY_GATEWAY_SVI` | SVI purpose is visible |
| 55 | Configure gateway IP address if SW1 is routing | SW1 | `ip address 192.0.2.1 255.255.255.0` | SVI has the default gateway IP |
| 56 | Map secondary VLANs to the primary SVI | SW1 | `private-vlan mapping 101,102` | Primary SVI can route for associated secondary VLANs |
| 57 | Enable the primary SVI | SW1 | `no shutdown` | SVI is administratively up |
| 58 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 59 | Verify PVLAN database | SW1 | `show vlan private-vlan` | Primary VLAN 100 is associated with secondary VLANs 101 and 102 |
| 60 | Verify promiscuous port mode and mapping | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Port is PVLAN promiscuous and maps 100 to 101,102 |
| 61 | Verify isolated host association | SW1 | `show interfaces GigabitEthernet0/2 switchport` | Port is PVLAN host associated to 100/101 |
| 62 | Verify community host association | SW1 | `show interfaces GigabitEthernet0/4 switchport` | Port is PVLAN host associated to 100/102 |
| 63 | Verify SVI mapping if used | SW1 | `show running-config interface Vlan100` | SVI includes `private-vlan mapping 101,102` |
| 64 | Verify STP forwarding state | SW1 | `show spanning-tree vlan 100` | PVLAN-related ports are not blocked unexpectedly |
| 65 | Test isolated host to gateway | H1 | `ping 192.0.2.1` | Isolated host reaches promiscuous gateway |
| 66 | Test second isolated host to gateway | H2 | `ping 192.0.2.1` | Isolated host reaches promiscuous gateway |
| 67 | Test isolated host to isolated host | H1 | `ping 192.0.2.12` | Ping fails because isolated hosts cannot talk to each other |
| 68 | Test isolated host to community host | H1 | `ping 192.0.2.13` | Ping fails because isolated and community hosts are separated |
| 69 | Test community host to community host | H3 | `ping 192.0.2.14` | Ping succeeds because community hosts in the same community can talk |
| 70 | Verify MAC learning in PVLANs | SW1 | `show mac address-table dynamic vlan 100` | MAC learning is visible for the primary VLAN view, depending on platform |
| 71 | Verify MAC learning in secondary VLANs if platform displays them separately | SW1 | `show mac address-table dynamic vlan 101` | Isolated hosts are learned on expected ports if supported |
| 72 | Save configuration | SW1 | `write memory` | PVLAN configuration survives reload |

# Private_VLAN_Single_Switch_Isolation_Skeleton

Single-switch PVLAN with external gateway on promiscuous port:

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
 description H2_ISOLATED_PVLAN_101
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 101
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/4
 description H3_COMMUNITY_PVLAN_102
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 102
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/5
 description H4_COMMUNITY_PVLAN_102
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 102
 spanning-tree portfast
 no shutdown
!
end
write memory

Single-switch PVLAN with switch SVI as gateway:

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
interface Vlan100
 description PVLAN_PRIMARY_GATEWAY_SVI
 ip address 192.0.2.1 255.255.255.0
 private-vlan mapping 101,102
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
 description H2_ISOLATED_PVLAN_101
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 101
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/4
 description H3_COMMUNITY_PVLAN_102
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 102
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/5
 description H4_COMMUNITY_PVLAN_102
 switchport
 switchport mode private-vlan host
 switchport private-vlan host-association 100 102
 spanning-tree portfast
 no shutdown
!
end
write memory

Example endpoint addressing:

H1 isolated:
ip address 192.0.2.11/24
default gateway 192.0.2.1

H2 isolated:
ip address 192.0.2.12/24
default gateway 192.0.2.1

H3 community:
ip address 192.0.2.13/24
default gateway 192.0.2.1

H4 community:
ip address 192.0.2.14/24
default gateway 192.0.2.1

Expected ping matrix:

H1 to gateway 192.0.2.1: permit
H2 to gateway 192.0.2.1: permit
H3 to gateway 192.0.2.1: permit
H4 to gateway 192.0.2.1: permit
H1 to H2: deny
H1 to H3: deny
H1 to H4: deny
H3 to H4: permit

# Private_VLAN_Single_Switch_Isolation_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm PVLAN feature and database | SW1 | `show vlan private-vlan` | Primary VLAN 100 is associated with secondary VLANs 101 and 102 |
| Confirm normal VLAN database | SW1 | `show vlan brief` | VLANs 100, 101, and 102 exist |
| Confirm primary VLAN configuration | SW1 | `show running-config | section vlan 100` | VLAN 100 is primary and associated with 101,102 |
| Confirm isolated VLAN configuration | SW1 | `show running-config | section vlan 101` | VLAN 101 is isolated |
| Confirm community VLAN configuration | SW1 | `show running-config | section vlan 102` | VLAN 102 is community |
| Confirm promiscuous port mode | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Port mode is private-vlan promiscuous |
| Confirm promiscuous mapping | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Mapping shows primary 100 to secondary 101,102 |
| Confirm isolated host port mode | SW1 | `show interfaces GigabitEthernet0/2 switchport` | Port mode is private-vlan host |
| Confirm isolated host association | SW1 | `show interfaces GigabitEthernet0/2 switchport` | Host association shows 100/101 |
| Confirm second isolated host association | SW1 | `show interfaces GigabitEthernet0/3 switchport` | Host association shows 100/101 |
| Confirm community host association | SW1 | `show interfaces GigabitEthernet0/4 switchport` | Host association shows 100/102 |
| Confirm second community host association | SW1 | `show interfaces GigabitEthernet0/5 switchport` | Host association shows 100/102 |
| Confirm SVI mapping if switch routes | SW1 | `show running-config interface Vlan100` | SVI contains `private-vlan mapping 101,102` |
| Confirm SVI line state if switch routes | SW1 | `show ip interface brief | include Vlan100` | Vlan100 is up/up if active ports exist |
| Confirm switchport physical state | SW1 | `show interfaces status` | PVLAN ports show connected where endpoints are attached |
| Confirm STP state | SW1 | `show spanning-tree vlan 100` | PVLAN ports are forwarding unless intentionally blocked |
| Confirm no err-disabled ports | SW1 | `show interfaces status err-disabled` | No required PVLAN ports are err-disabled |
| Confirm MAC learning on primary VLAN | SW1 | `show mac address-table dynamic vlan 100` | Host or gateway MACs appear where the platform reports them |
| Confirm MAC learning on isolated VLAN | SW1 | `show mac address-table dynamic vlan 101` | Isolated host MACs appear if the platform reports secondary VLANs separately |
| Confirm MAC learning on community VLAN | SW1 | `show mac address-table dynamic vlan 102` | Community host MACs appear if the platform reports secondary VLANs separately |
| Confirm isolated host reaches gateway | H1 | `ping 192.0.2.1` | Ping succeeds |
| Confirm second isolated host reaches gateway | H2 | `ping 192.0.2.1` | Ping succeeds |
| Confirm community host reaches gateway | H3 | `ping 192.0.2.1` | Ping succeeds |
| Confirm isolated-to-isolated isolation | H1 | `ping 192.0.2.12` | Ping fails |
| Confirm isolated-to-community isolation | H1 | `ping 192.0.2.13` | Ping fails |
| Confirm community-to-community communication | H3 | `ping 192.0.2.14` | Ping succeeds |
| Confirm ARP behavior to gateway | H1/H2/H3/H4 | `show arp` or `ip neigh` | Gateway MAC is learned |
| Confirm ARP does not prove isolated peer reachability | H1 | `show arp` or `ip neigh` | Isolated peer may not resolve or traffic still fails |
| Confirm final saved configuration | SW1 | `show startup-config | include private-vlan` | Startup configuration contains PVLAN commands |

# Private_VLAN_Single_Switch_Isolation_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record PVLAN state before rollback | SW1 | `show vlan private-vlan` | Current primary and secondary associations are documented |
| 2 | Record PVLAN interface state before rollback | SW1 | `show interfaces switchport | include Name|private-vlan|Operational Mode` | Current PVLAN port roles are documented |
| 3 | Enter configuration mode | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 4 | Reset promiscuous gateway port | SW1 | `default interface GigabitEthernet0/1` | Promiscuous PVLAN configuration is removed |
| 5 | Reset first isolated host port | SW1 | `default interface GigabitEthernet0/2` | PVLAN host association is removed |
| 6 | Reset second isolated host port | SW1 | `default interface GigabitEthernet0/3` | PVLAN host association is removed |
| 7 | Reset first community host port | SW1 | `default interface GigabitEthernet0/4` | PVLAN host association is removed |
| 8 | Reset second community host port | SW1 | `default interface GigabitEthernet0/5` | PVLAN host association is removed |
| 9 | Remove SVI mapping if configured | SW1 | `interface Vlan100` then `no private-vlan mapping 101,102` | SVI no longer maps secondary VLANs |
| 10 | Remove SVI IP if the lab is being fully cleared | SW1 | `no ip address` | Gateway IP is removed |
| 11 | Shut the SVI if the lab is being fully cleared | SW1 | `shutdown` | SVI is administratively down |
| 12 | Exit SVI configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 13 | Remove primary VLAN association before deleting VLANs | SW1 | `vlan 100` then `no private-vlan association 101,102` | Secondary VLAN association is removed |
| 14 | Exit VLAN configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 15 | Remove isolated secondary VLAN | SW1 | `no vlan 101` | Isolated VLAN is deleted |
| 16 | Remove community secondary VLAN | SW1 | `no vlan 102` | Community VLAN is deleted |
| 17 | Remove primary VLAN | SW1 | `no vlan 100` | Primary VLAN is deleted |
| 18 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Verify PVLAN cleanup | SW1 | `show vlan private-vlan` | Test PVLAN association is gone |
| 20 | Verify VLAN cleanup | SW1 | `show vlan brief` | VLANs 100, 101, and 102 are removed if full cleanup was intended |
| 21 | Verify interface cleanup | SW1 | `show running-config interface GigabitEthernet0/2` | Interface-specific PVLAN commands are gone |
| 22 | Save rollback | SW1 | `write memory` | Startup configuration reflects rollback |

# Private_VLAN_Single_Switch_Isolation_Failure_Checks

| Symptom                                               | Likely Cause                                                             | Device    | Command                                                                  | Fix                                                                                         |                                                                               |
| ----------------------------------------------------- | ------------------------------------------------------------------------ | --------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `private-vlan` command is rejected                    | Platform or image does not support PVLAN                                 | SW1       | `show vlan private-vlan`                                                 | Use a switch image or license that supports private VLANs                                   |                                                                               |
| Secondary VLAN does not associate to primary          | Secondary VLAN is missing or not configured as isolated/community        | SW1       | `show running-config                                                     | section vlan`                                                                               | Create secondary VLANs and mark them isolated or community before association |
| Host port does not enter PVLAN host mode              | Interface is routed or in conflicting switchport mode                    | SW1       | `show interfaces GigabitEthernet0/2 switchport`                          | Make the interface a Layer 2 switchport and configure PVLAN host mode                       |                                                                               |
| Promiscuous port cannot reach hosts                   | Promiscuous mapping is missing or wrong                                  | SW1       | `show interfaces GigabitEthernet0/1 switchport`                          | Configure `switchport private-vlan mapping 100 101,102`                                     |                                                                               |
| Isolated host cannot reach gateway                    | Gateway port is not promiscuous or SVI mapping is missing                | SW1       | `show interfaces switchport` and `show running-config interface Vlan100` | Configure promiscuous mapping or SVI `private-vlan mapping`                                 |                                                                               |
| Isolated host can reach another isolated host         | Ports are not actually in isolated secondary VLAN or PVLAN not active    | SW1       | `show interfaces GigabitEthernet0/2 switchport`                          | Configure both ports as PVLAN hosts associated to 100/101                                   |                                                                               |
| Community hosts cannot reach each other               | Community ports are in different secondary VLANs or not PVLAN host ports | SW1       | `show interfaces GigabitEthernet0/4 switchport`                          | Put both community hosts in the same community secondary VLAN                               |                                                                               |
| Isolated host can reach community host                | Wrong host association or port not in PVLAN mode                         | SW1       | `show interfaces switchport`                                             | Correct host associations to isolated and community secondary VLANs                         |                                                                               |
| SVI is down/down                                      | No active ports in the primary or associated secondary VLANs             | SW1       | `show ip interface brief                                                 | include Vlan100`                                                                            | Bring up associated host or promiscuous ports                                 |
| SVI is up but hosts cannot reach it                   | SVI lacks secondary VLAN mapping                                         | SW1       | `show running-config interface Vlan100`                                  | Add `private-vlan mapping 101,102` under interface Vlan100                                  |                                                                               |
| Gateway router cannot reach hosts                     | Router-facing port is not promiscuous                                    | SW1       | `show interfaces GigabitEthernet0/1 switchport`                          | Configure promiscuous mode and secondary VLAN mapping                                       |                                                                               |
| Hosts are in different IP subnets                     | PVLAN is same-subnet isolation, not routing segmentation                 | HOSTS     | `ip addr`, `show ip interface brief`, or endpoint IP settings            | Put test hosts and gateway in the same subnet                                               |                                                                               |
| ARP fails to gateway                                  | Promiscuous mapping, SVI mapping, or endpoint IP issue                   | SW1/HOSTS | `show arp`, `ip neigh`, `show mac address-table dynamic`                 | Fix mapping, IP addressing, and port association                                            |                                                                               |
| MAC addresses do not appear                           | No traffic generated or wrong VLAN display view                          | SW1       | `show mac address-table dynamic vlan 100`                                | Generate ping or ARP traffic and check primary and secondary VLAN tables                    |                                                                               |
| Port is err-disabled                                  | BPDU Guard, port security, or link-flap disabled the port                | SW1       | `show interfaces status err-disabled`                                    | Fix the trigger, then shut/no shut the interface                                            |                                                                               |
| Host port takes too long to come up                   | PortFast missing on host-facing PVLAN ports                              | SW1       | `show spanning-tree interface GigabitEthernet0/2 detail`                 | Add `spanning-tree portfast` on host-facing ports                                           |                                                                               |
| PVLAN port is blocked by STP                          | STP state prevents forwarding                                            | SW1       | `show spanning-tree vlan 100`                                            | Verify STP topology and remove unintended loop or protection state                          |                                                                               |
| DHCP does not work for PVLAN hosts                    | DHCP snooping, relay, or server path does not account for PVLANs         | SW1       | `show ip dhcp snooping`                                                  | Enable DHCP handling on the primary and associated VLANs as required                        |                                                                               |
| Port security command is rejected                     | Platform does not support port security on PVLAN ports                   | SW1       | `show running-config interface GigabitEthernet0/2`                       | Do not combine unsupported port security with PVLAN host ports                              |                                                                               |
| SPAN destination config is rejected                   | PVLAN ports cannot be used for that SPAN role on some platforms          | SW1       | `show monitor session all`                                               | Use a supported destination interface                                                       |                                                                               |
| Trunk commands are used for a single-switch PVLAN lab | PVLAN trunking is not needed for single-switch isolation                 | SW1       | `show interfaces trunk`                                                  | Keep this lab local to one switch; use the multi-switch PVLAN note for PVLAN trunk behavior |                                                                               |
| Configuration disappears after reload                 | Configuration was not saved                                              | SW1       | `show startup-config                                                     | include private-vlan`                                                                       | Reapply PVLAN configuration and run `write memory`                            |