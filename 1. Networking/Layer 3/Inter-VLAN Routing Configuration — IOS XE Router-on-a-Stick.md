|      |                                       |        |                                                                                       |                                        |
| ---- | ------------------------------------- | ------ | ------------------------------------------------------------------------------------- | -------------------------------------- |
| Step | Task                                  | Device | IOS XE Command                                                                        | Expected Result                        |
| 1    | Verify VLANs exist                    | Switch | show vlan brief                                                                       | Required VLANs are present             |
| 2    | Create missing VLANs                  | Switch | conf t -> vlan <vlan-id>    -> name <name>                                            | VLAN exists in VLAN database           |
| 3    | Assign host ports to VLANs            | Switch | interface <access-port> -> switchport mode access -> switchport access vlan <vlan-id> | Hosts are placed in correct VLANs      |
| 4    | Configure router-facing port as trunk | Switch | interface <router-facing-port>                                                        | Interface config mode                  |
| 5    | Set trunk encapsulation if required   | Switch | switchport trunk encapsulation dot1q                                                  | 802.1Q selected                        |
| 6    | Force trunk mode                      | Switch | switchport mode trunk                                                                 | Port becomes trunk                     |
| 7    | Allow required VLANs                  | Switch | switchport trunk allowed vlan <vlan-list>                                             | VLANs permitted across trunk           |
| 8    | Enable trunk port                     | Switch | no shutdown                                                                           | Port up/up                             |
| 9    | Enable router parent interface        | Router | interface <physical-interface> -> no shutdown                                         | Parent interface up                    |
| 10   | Create VLAN subinterface              | Router | interface <physical-interface>.<vlan-id>                                              | Subinterface created                   |
| 11   | Add 802.1Q tag                        | Router | encapsulation dot1Q <vlan-id>                                                         | Subinterface tied to VLAN              |
| 12   | Assign gateway IP                     | Router | ip address <gateway-ip> <subnet-mask>                                                 | VLAN default gateway configured        |
| 13   | Repeat for each VLAN                  | Router | Repeat subinterface steps                                                             | One subinterface per VLAN              |
| 14   | Verify subinterfaces                  | Router | show ip interface brief                                                               | Subinterfaces up/up with correct IPs   |
| 15   | Verify connected routes               | Router | show ip route connected                                                               | Each VLAN network appears as connected |
| 16   | Verify trunk                          | Switch | show interfaces trunk                                                                 | VLANs allowed and active on trunk      |
| 17   | Test host-to-gateway                  | Host   | ping <vlan-gateway-ip>                                                                | Host reaches its default gateway       |
| 18   | Test inter-VLAN traffic               | Host   | ping <host-in-other-vlan>                                                             | Cross-VLAN communication succeeds      |
Example: VLAN 10 and VLAN 20 Router-on-a-Stick

|        |                             |                                       |
| ------ | --------------------------- | ------------------------------------- |
| Device | Step                        | IOS XE Command                        |
| Switch | Enter config                | conf t                                |
| Switch | Create VLAN 10              | vlan 10                               |
| Switch | Name VLAN 10                | name DATA_10                          |
| Switch | Create VLAN 20              | vlan 20                               |
| Switch | Name VLAN 20                | name DATA_20                          |
| Switch | Configure host port VLAN 10 | interface e0/0                        |
| Switch | Access mode                 | switchport mode access                |
| Switch | Assign VLAN 10              | switchport access vlan 10             |
| Switch | Configure host port VLAN 20 | interface e0/1                        |
| Switch | Access mode                 | switchport mode access                |
| Switch | Assign VLAN 20              | switchport access vlan 20             |
| Switch | Configure router trunk      | interface e0/2                        |
| Switch | Set encapsulation if needed | switchport trunk encapsulation dot1q  |
| Switch | Trunk mode                  | switchport mode trunk                 |
| Switch | Allow VLANs                 | switchport trunk allowed vlan 10,20   |
| Switch | Enable trunk                | no shutdown                           |
| Router | Enter config                | conf t                                |
| Router | Enable parent interface     | interface g0/1                        |
| Router | No IP on parent             | no ip address                         |
| Router | Enable parent               | no shutdown                           |
| Router | VLAN 10 subinterface        | interface g0/1.10                     |
| Router | VLAN 10 encapsulation       | encapsulation dot1Q 10                |
| Router | VLAN 10 gateway             | ip address 192.168.10.1 255.255.255.0 |
| Router | VLAN 20 subinterface        | interface g0/1.20                     |
| Router | VLAN 20 encapsulation       | encapsulation dot1Q 20                |
| Router | VLAN 20 gateway             | ip address 192.168.20.1 255.255.255.0 |
| Router | Exit                        | end                                   |

Verification

|   |   |   |
|---|---|---|
|Check|IOS XE Command|Expected Result|
|VLANs exist|show vlan brief|VLAN 10 and 20 present|
|Access ports assigned|show vlan brief|Host ports listed under correct VLAN|
|Trunk forwarding VLANs|show interfaces trunk|VLANs 10 and 20 allowed/active|
|Router subinterfaces|show ip interface brief|g0/1.10 and g0/1.20 up/up|
|Router connected routes|show ip route connected|192.168.10.0/24 and 192.168.20.0/24 installed|
|Host gateway test|ping <default-gateway>|Successful replies|
|Inter-VLAN test|ping <other-vlan-host>|Successful replies|

Common Issues

|   |   |   |
|---|---|---|
|Symptom|Likely Cause|Fix|
|Subinterface down/down|Parent interface down|interface g0/1 â†’ no shutdown|
|Host cannot ping gateway|Wrong access VLAN or trunk missing VLAN|Check show vlan brief and show interfaces trunk|
|One VLAN works, another fails|VLAN not allowed on trunk|Add VLAN to allowed list|
|Router sees no connected route|Subinterface down or missing IP|Check show ip interface brief|
|Switch rejects trunk command|Encapsulation still auto|Use switchport trunk encapsulation dot1q before switchport mode trunk|
|Hosts can ping gateway but not each other|Host firewall, wrong default gateway, ACL|Check host gateway, ACLs, return path|
