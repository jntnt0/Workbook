|      |                           |                                              |
| ---- | ------------------------- | -------------------------------------------- |
| Step | Task                      | IOS XE Command                               |
| 1    | Enter config mode         | conf t                                       |
| 2    | Select physical interface | interface g0/1                               |
| 3    | Enable parent interface   | no shutdown                                  |
| 4    | Create subinterface       | interface g0/1.<vlan-id>                     |
| 5    | Assign 802.1Q VLAN tag    | encapsulation dot1Q <vlan-id>                |
| 6    | Assign gateway IP         | ip address <gateway-ip> <subnet-mask>        |
| 7    | Exit                      | end                                          |
| 8    | Verify status             | show ip interface brief                      |
| 9    | Verify config             | show running-config interface g0/1.<vlan-id> |
| 10   | Verify connected route    | show ip route connected                      |




|   |   |
|---|---|
|Purpose|Command|
|Enter config|conf t|
|Enable parent interface|interface g0/1|
|Bring up parent|no shutdown|
|Create VLAN 10 subinterface|interface g0/1.10|
|Tag VLAN 10|encapsulation dot1Q 10|
|Assign gateway IP|ip address 192.168.10.1 255.255.255.0|
|Finish|end|
|Verify|show ip interface brief|





|                                 |                                       |
| ------------------------------- | ------------------------------------- |
| Purpose                         | Command                               |
| Create native VLAN subinterface | interface g0/1.99                     |
| Set native VLAN tag             | encapsulation dot1Q 99 native         |
| Assign IP                       | ip address 192.168.99.1 255.255.255.0 |


|   |   |
|---|---|
|Symptom|Likely Cause|
|Subinterface is down/down|Parent interface is down|
|Subinterface is up/up but hosts fail|Switch trunk missing VLAN|
|Ping gateway fails|Wrong VLAN tag or host in wrong VLAN|
|Connected route missing|Subinterface not up/up or no IP|
|Native VLAN mismatch|Native VLAN differs between router and switch|