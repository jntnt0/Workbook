|      |                                         |        |                                                 |                                              |
| ---- | --------------------------------------- | ------ | ----------------------------------------------- | -------------------------------------------- |
| Step | Task                                    | Device | IOS XE Command                                  | Expected Result                              |
| 1    | Verify candidate interfaces             | Switch | show interfaces status                          | Member interfaces exist and are usable       |
| 2    | Check existing EtherChannels            | Switch | show etherchannel summary                       | No conflicting channel-group already applied |
| 3    | Enter config mode                       | Switch | conf t                                          | Global config mode                           |
| 4    | Select member interfaces                | Switch | interface range <interface-range>               | Interface range config mode                  |
| 5    | Clear conflicting config if needed      | Switch | default interface range <interface-range>       | Interfaces reset before bundling             |
| 6    | Set switchport mode                     | Switch | switchport mode trunk or switchport mode access | All members use same L2 mode                 |
| 7    | Set trunk encapsulation if required     | Switch | switchport trunk encapsulation dot1q            | 802.1Q selected on older/lab IOS images      |
| 8    | Configure allowed VLANs if trunk        | Switch | switchport trunk allowed vlan <vlan-list>       | Same VLAN list applied to all members        |
| 9    | Assign LACP channel group               | Switch | channel-group <number> mode active              | Interfaces join LACP EtherChannel            |
| 10   | Configure logical Port-channel          | Switch | interface port-channel <number>                 | Port-channel interface config mode           |
| 11   | Apply final L2 settings to Port-channel | Switch | switchport mode trunk or switchport mode access | Logical bundle carries intended traffic      |
| 12   | Enable interfaces                       | Switch | no shutdown                                     | Member links enabled                         |
| 13   | Exit config mode                        | Switch | end                                             | Back to privileged EXEC                      |
| 14   | Verify bundle status                    | Switch | show etherchannel summary                       | Port-channel shows SU, members show P        |
| 15   | Verify LACP neighbor                    | Switch | show lacp neighbor                              | Neighbor detected on bundled links           |
| 16   | Verify trunk/access behavior            | Switch | show interfaces trunk or show vlan brief        | Bundle carries expected VLANs                |

| Example |                               |                                        |
| ------- | ----------------------------- | -------------------------------------- |
| Device  | Step                          | IOS XE Command                         |
| SW1     | Enter config                  | conf t                                 |
| SW1     | Select member links           | interface range g0/1 - 2               |
| SW1     | Set encapsulation if required | switchport trunk encapsulation dot1q   |
| SW1     | Set trunk mode                | switchport mode trunk                  |
| SW1     | Allow VLANs                   | switchport trunk allowed vlan 10,20,99 |
| SW1     | Enable LACP active            | channel-group 1 mode active            |
| SW1     | Configure Port-channel        | interface port-channel 1               |
| SW1     | Set trunk mode on bundle      | switchport mode trunk                  |
| SW1     | Allow VLANs on bundle         | switchport trunk allowed vlan 10,20,99 |
| SW1     | Finish                        | end                                    |
| SW2     | Enter config                  | conf t                                 |
| SW2     | Select member links           | interface range g0/1 - 2               |
| SW2     | Set encapsulation if required | switchport trunk encapsulation dot1q   |
| SW2     | Set trunk mode                | switchport mode trunk                  |
| SW2     | Allow VLANs                   | switchport trunk allowed vlan 10,20,99 |
| SW2     | Enable LACP active            | channel-group 1 mode active            |
| SW2     | Configure Port-channel        | interface port-channel 1               |
| SW2     | Set trunk mode on bundle      | switchport mode trunk                  |
| SW2     | Allow VLANs on bundle         | switchport trunk allowed vlan 10,20,99 |
| SW2     | Finish                        | end                                    |








| Ex                           |                             |
| ---------------------------- | --------------------------- |
| Step                         | IOS XE Command              |
| Enter config                 | conf t                      |
| Select member links          | interface range g0/1 - 2    |
| Set access mode              | switchport mode access      |
| Assign access VLAN           | switchport access vlan 10   |
| Enable LACP                  | channel-group 1 mode active |
| Configure Port-channel       | interface port-channel 1    |
| Set access mode on bundle    | switchport mode access      |
| Assign access VLAN on bundle | switchport access vlan 10   |
| Exit                         | end                         |
| Verify                       | show etherchannel summary   |

Verification

|                               |                                                     |                                                  |
| ----------------------------- | --------------------------------------------------- | ------------------------------------------------ |
| Check                         | IOS XE Command                                      | Expected Result                                  |
| EtherChannel formed           | show etherchannel summary                           | Port-channel shows SU; member ports show P       |
| LACP neighbor detected        | show lacp neighbor                                  | Remote system and ports visible                  |
| Port-channel interface status | show interfaces port-channel <number>               | Port-channel is up/up                            |
| Trunk VLANs carried           | show interfaces trunk                               | Port-channel listed as trunk with expected VLANs |
| Member interface config       | show running-config interface <interface>           | Member links have matching channel-group         |
| Port-channel config           | show running-config interface port-channel <number> | Logical interface has final access/trunk config  |

EtherChannel Status Codes

|      |                                       |
| ---- | ------------------------------------- |
| Code | Meaning                               |
| SU   | Layer 2 Port-channel is up and in use |
| RU   | Layer 3 Port-channel is up and routed |
| P    | Member port is bundled correctly      |
| I    | Standalone, not bundled               |
| s    | Suspended                             |
| D    | Down                                  |
| w    | Waiting to aggregate                  |


Common Issues

|   |   |   |
|---|---|---|
|Symptom|Likely Cause|Fix|
|Member ports suspended|Mismatch between member links|Match speed, duplex, trunk mode, VLANs, native VLAN|
|Port-channel not trunking|Trunk configured on members but not Port-channel|Configure trunk settings on port-channel|
|One side bundles, other side does not|LACP mode mismatch or wrong cabling|Use mode active on both sides|
|VLAN traffic missing|Allowed VLAN list incomplete|Add VLANs to allowed list on Port-channel|
|Native VLAN mismatch|Different native VLANs|Match native VLAN on both sides|
|STP blocking unexpected link|EtherChannel not formed, links seen separately|Fix LACP bundle before troubleshooting STP|
|Channel-group rejected|Existing incompatible config|Use default interface range <range> before reconfiguring|