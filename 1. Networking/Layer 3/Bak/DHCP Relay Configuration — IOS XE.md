|      |                                   |             |                                                          |                                      |
| ---- | --------------------------------- | ----------- | -------------------------------------------------------- | ------------------------------------ |
| Step | Task                              | Device      | IOS XE Command                                           | Expected Result                      |
| 1    | Identify client gateway interface | LAN Gateway | show ip interface brief                                  | Client-facing interface is known     |
| 2    | Enter config mode                 | LAN Gateway | conf t                                                   | Global config mode                   |
| 3    | Select client-facing interface    | LAN Gateway | interface <client-gateway-interface>                     | Interface config mode                |
| 4    | Add DHCP relay address            | LAN Gateway | ip helper-address <dhcp-server-ip>                       | DHCP broadcasts relay to DHCP server |
| 5    | Exit config mode                  | LAN Gateway | end                                                      | Back to privileged EXEC              |
| 6    | Verify interface config           | LAN Gateway | show running-config interface <client-gateway-interface> | Helper address is present            |
| 7    | Test client DHCP                  | Client Host | dhclient eth0 or reboot host network                     | Client receives IPv4 address         |


| Example in route in a stick |                                       |
| --------------------------- | ------------------------------------- |
| Step                        | IOS XE Command                        |
| Enter config                | conf t                                |
| Select VLAN 10 gateway      | interface g0/1.10                     |
| Add DHCP server relay       | ip helper-address 192.168.40.10       |
| Exit                        | end                                   |
| Verify                      | show running-config interface g0/1.10 |



| Common DHCP Issues               |                             |                                         |
| -------------------------------- | --------------------------- | --------------------------------------- |
| Symptom                          | Likely Cause                | Check                                   |
| Client has no IPv4 address       | No DHCP response            | show ip dhcp binding                    |
| Client gets wrong subnet         | Wrong pool network          | show running-config \| section dhcp     |
| Client gets IP but cannot route  | Wrong default-router option | show ip dhcp pool                       |
| Remote VLAN clients get no lease | Missing ip helper-address   | show run interface <gateway-int>        |
| Pool exhausted                   | No free addresses           | show ip dhcp pool                       |
| DHCP conflict blocks leases      | Conflict table populated    | show ip dhcp conflict                   |
| Interface up/up but no DHCP      | VLAN/trunk issue            | show vlan brief / show interfaces trunk |
