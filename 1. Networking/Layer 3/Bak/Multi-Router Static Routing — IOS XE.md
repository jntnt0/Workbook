|      |                                             |                 |                                                |                                                              |
| ---- | ------------------------------------------- | --------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Step | Task                                        | Device          | IOS XE Command                                 | Expected Result                                              |
| 1    | Identify all local LAN networks             | All routers     | show ip route connected                        | Each router shows its directly connected LAN and WAN links   |
| 2    | Identify remote LANs each router must reach | All routers     | Topology / addressing table                    | You know which prefixes are not directly connected           |
| 3    | Verify WAN next-hop reachability            | All routers     | ping <wan-neighbor-ip>                         | Each router can reach its directly connected router neighbor |
| 4    | Enter config mode                           | Router          | conf t                                         | Global config mode                                           |
| 5    | Add static route to remote LAN              | Router          | ip route <remote-network> <mask> <next-hop-ip> | Router has path to remote subnet                             |
| 6    | Repeat for every missing remote LAN         | Router          | ip route <remote-network> <mask> <next-hop-ip> | All required remote routes exist                             |
| 7    | Add return routes on opposite routers       | Remote routers  | ip route <source-network> <mask> <next-hop-ip> | Return traffic knows how to get back                         |
| 8    | Verify static routes                        | All routers     | show ip route static                           | Static routes installed                                      |
| 9    | Verify specific destination route           | All routers     | show ip route <remote-host-ip>                 | Correct next hop selected                                    |
| 10   | Verify CEF forwarding                       | All routers     | show ip cef <remote-host-ip> detail            | Recursive next-hop resolves to exit interface                |
| 11   | Test end-to-end reachability                | Hosts / routers | ping <remote-host-ip>                          | Replies received                                             |
| 12   | Validate path                               | Hosts / routers | traceroute <remote-host-ip>                    | Path follows expected router chain                           |
| 13   | Save config                                 | All routers     | write memory                                   | Static routes persist after reload                           |
Verification

|   |   |   |
|---|---|---|
|Check|IOS XE Command|Expected Result|
|Connected routes|show ip route connected|Local LAN and WAN links present|
|Static routes|show ip route static|Remote networks listed as static|
|Specific route lookup|show ip route <remote-host-ip>|Correct next-hop shown|
|Forwarding lookup|show ip cef <remote-host-ip> detail|Recursive via next-hop and attached interface|
|Next-hop reachability|ping <next-hop-ip>|Replies received|
|End-to-end ping|ping <remote-host-ip>|Replies received|
|Path validation|traceroute <remote-host-ip>|Expected router sequence|
|Running config|show running-config \| include ^ip route|Static route statements present|


Common Issues

|   |   |   |
|---|---|---|
|Symptom|Likely Cause|Fix|
|Router says % Network not in table|Missing static route|Add route to remote prefix|
|Traffic leaves source but replies fail|Missing return route|Add reverse route on destination-side router|
|Traceroute stops at hub|Hub missing route to destination LAN|Add destination route on hub|
|CEF says no route|RIB has no usable route|Fix static route or next-hop|
|Static route configured but not installed|Next-hop not reachable or interface down|Verify connected route and interface state|
|Wrong path chosen|AD/longest-match issue|Check show ip route and adjust AD|
|Host can ping gateway only|Router missing remote routes|Add static/default routes|
|Router can ping, host cannot|Host default gateway wrong|Fix host gateway/DHCP option|



Three router hub and spoke example

|               |                  |                                                                 |
| ------------- | ---------------- | --------------------------------------------------------------- |
| Router        | LAN              | WAN Link                                                        |
| AtlantaR1     | 192.168.100.0/24 | To Chattanooga: 192.168.250.0/30; To Savannah: 192.168.250.4/30 |
| ChattanoogaR1 | 192.168.10.0/24  | To Atlanta: next-hop 192.168.250.1                              |
| SavannahR1    | 192.168.50.0/24  | To Atlanta: next-hop 192.168.250.5                              |

ATL R1

|                       |                                                   |
| --------------------- | ------------------------------------------------- |
| Purpose               | IOS XE Command                                    |
| Reach Chattanooga LAN | ip route 192.168.10.0 255.255.255.0 192.168.250.2 |
| Reach Savannah LAN    | ip route 192.168.50.0 255.255.255.0 192.168.250.6 |

Chatt R1

|                                    |                                                    |
| ---------------------------------- | -------------------------------------------------- |
| Purpose                            | IOS XE Command                                     |
| Reach Atlanta LAN                  | ip route 192.168.100.0 255.255.255.0 192.168.250.1 |
| Reach Savannah LAN through Atlanta | ip route 192.168.50.0 255.255.255.0 192.168.250.1  |

Or use default route

|                                      |                                        |
| ------------------------------------ | -------------------------------------- |
| Purpose                              | IOS XE Command                         |
| Send all nonlocal traffic to Atlanta | ip route 0.0.0.0 0.0.0.0 192.168.250.1 |

Savahanna R1

|                                       |                                                    |
| ------------------------------------- | -------------------------------------------------- |
| Purpose                               | IOS XE Command                                     |
| Reach Atlanta LAN                     | ip route 192.168.100.0 255.255.255.0 192.168.250.5 |
| Reach Chattanooga LAN through Atlanta | ip route 192.168.10.0 255.255.255.0 192.168.250.5  |