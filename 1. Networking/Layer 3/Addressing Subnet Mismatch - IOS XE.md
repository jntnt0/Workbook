|Step|Check|Command|What You’re Looking For|
|---|---|---|---|
|1|Check local IP/mask|`show ip interface brief`|Correct IP on the correct interface|
|2|Check full mask|`show running-config interface <interface>`|Correct subnet mask|
|3|Check neighbor IP|Topology / remote device config|Neighbor should be in same subnet on directly connected link|
|4|Ping neighbor|`ping <neighbor-ip>`|Should succeed if same subnet and link is up|
|5|Check ARP|`show ip arp <neighbor-ip>`|MAC should resolve|
|6|Fix local address|`conf t` → `interface <interface>` → `ip address <ip> <mask>`|Correct IP/mask applied|
|7|Re-test|`ping <neighbor-ip>`|Neighbor reachable|



|Symptom|Likely Cause|
|---|---|
|Interface is `up/up` but ping fails|IP/mask mismatch|
|ARP incomplete|Neighbor not in same L2/subnet or wrong gateway|
|Route looks connected but neighbor unreachable|Wrong mask or wrong IP|
|One side `/30`, other side `/24`|Subnet mismatch|





| Step             | Command                                    |
| ---------------- | ------------------------------------------ |
| Enter config     | `conf t`                                   |
| Select interface | `interface g0/0`                           |
| Correct IP       | `ip address 192.168.250.2 255.255.255.252` |
| Enable           | `no shutdown`                              |
| Exit             | `end`                                      |
| Verify           | `show ip interface brief`                  |