RIP_Passive_Interface_Boundary_Control.md
# RIP_Passive_Interface_Boundary_Control

# RIP_Passive_Interface_Boundary_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Passive-interface | Suppresses RIP updates out of an interface |
| Boundary control | Prevents RIP advertisements from being sent toward hosts, access VLANs, or untrusted links |
| Still advertised | A passive RIP interface can still have its connected network advertised to other RIP neighbors if the network is included under `router rip` |
| RIP-specific behavior | RIP passive-interface mainly stops outbound updates; it is not an OSPF-style adjacency shutdown model |
| Transit interface | Router-to-router links that must exchange RIP updates should remain active |
| Edge interface | LAN, host-facing, and non-RIP-facing interfaces should usually be passive |
| Default-passive model | `passive-interface default` makes all RIP interfaces quiet until specific transit interfaces are allowed with `no passive-interface` |
| Safer design | Default-passive is cleaner in larger labs because new interfaces do not accidentally start sending RIP updates |
| Verification target | A correct design advertises LAN prefixes into RIP while sending updates only on intended router-to-router transit links |

# RIP_Passive_Interface_Boundary_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interface state before changing RIP behavior | R1/R2/R3 | `show ip interface brief` | LAN and transit interfaces are up/up |
| 2 | Confirm current RIP configuration | R1/R2/R3 | `show running-config | section router rip` | RIP is enabled with `version 2`, `no auto-summary`, and expected `network` statements |
| 3 | Confirm current RIP process behavior | R1/R2/R3 | `show ip protocols` | Output shows RIP routing for the expected networks |
| 4 | Identify interfaces that should exchange RIP updates | R1/R2/R3 | `show cdp neighbors` | Router-to-router transit interfaces are identified |
| 5 | Identify interfaces that should not send RIP updates | R1/R2/R3 | `show ip interface brief` | LAN, host-facing, loopback, or untrusted interfaces are identified |
| 6 | Enter global configuration mode | R1/R2/R3 | `configure terminal` | Router enters global configuration mode |
| 7 | Enter RIP router configuration mode | R1/R2/R3 | `router rip` | Router enters RIP configuration mode |
| 8 | Use the safer default-passive model | R1/R2/R3 | `passive-interface default` | All RIP-enabled interfaces stop sending RIP updates |
| 9 | Re-enable RIP updates on the R1-to-R2 transit interface | R1/R2 | `no passive-interface <R1_R2_TRANSIT_INTERFACE>` | RIP updates are allowed on the R1-to-R2 router link |
| 10 | Re-enable RIP updates on the R2-to-R3 transit interface | R2/R3 | `no passive-interface <R2_R3_TRANSIT_INTERFACE>` | RIP updates are allowed on the R2-to-R3 router link |
| 11 | Keep LAN interfaces passive | R1/R2/R3 | `passive-interface <LAN_INTERFACE>` | LAN interfaces do not send RIP updates |
| 12 | Keep loopbacks passive if they are advertised | R1/R2/R3 | `passive-interface <LOOPBACK_INTERFACE>` | Loopback prefixes can be advertised without sending RIP updates out the loopback |
| 13 | Exit configuration mode | R1/R2/R3 | `end` | Router returns to privileged EXEC mode |
| 14 | Save the configuration | R1/R2/R3 | `write memory` | Passive-interface configuration persists after reload |
| 15 | Verify passive interfaces in the RIP process | R1/R2/R3 | `show ip protocols` | Output lists passive interfaces and active routing-for-networks entries |
| 16 | Verify LAN prefixes are still advertised | Neighbor routers | `show ip route rip` | Remote LAN prefixes still appear as RIP routes |
| 17 | Verify transit updates still work | R1/R2/R3 | `debug ip rip` | RIP updates are sent and received on transit interfaces only |
| 18 | Stop debugging | R1/R2/R3 | `undebug all` | Debugging is disabled |
| 19 | Verify end-to-end reachability | R1/R3 | `ping <REMOTE_LAN_IP>` | Ping succeeds through the RIP-learned route |
| 20 | Verify forwarding path | R1/R3 | `traceroute <REMOTE_LAN_IP>` | Path follows the expected RIP transit routers |

# RIP_Passive_Interface_Boundary_Control_Skeleton
| Device | Configuration |
|---|---|
| R1 | `configure terminal` |
| R1 | `router rip` |
| R1 | `version 2` |
| R1 | `no auto-summary` |
| R1 | `network <R1_LAN_MAJOR_NETWORK>` |
| R1 | `network <R1_R2_TRANSIT_MAJOR_NETWORK>` |
| R1 | `passive-interface default` |
| R1 | `no passive-interface <R1_R2_TRANSIT_INTERFACE>` |
| R1 | `end` |
| R1 | `write memory` |
| R2 | `configure terminal` |
| R2 | `router rip` |
| R2 | `version 2` |
| R2 | `no auto-summary` |
| R2 | `network <R1_R2_TRANSIT_MAJOR_NETWORK>` |
| R2 | `network <R2_R3_TRANSIT_MAJOR_NETWORK>` |
| R2 | `passive-interface default` |
| R2 | `no passive-interface <R1_R2_TRANSIT_INTERFACE>` |
| R2 | `no passive-interface <R2_R3_TRANSIT_INTERFACE>` |
| R2 | `end` |
| R2 | `write memory` |
| R3 | `configure terminal` |
| R3 | `router rip` |
| R3 | `version 2` |
| R3 | `no auto-summary` |
| R3 | `network <R2_R3_TRANSIT_MAJOR_NETWORK>` |
| R3 | `network <R3_LAN_MAJOR_NETWORK>` |
| R3 | `passive-interface default` |
| R3 | `no passive-interface <R2_R3_TRANSIT_INTERFACE>` |
| R3 | `end` |
| R3 | `write memory` |

# RIP_Passive_Interface_Boundary_Control_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before and after passive-interface changes | Required interfaces are up/up |
| `show running-config | section router rip` | Confirms RIP and passive-interface configuration | Shows `passive-interface default` and `no passive-interface <TRANSIT_INTERFACE>` |
| `show ip protocols` | Confirms RIP active networks and passive interfaces | Passive interfaces are listed; transit interfaces remain active |
| `show ip route rip` | Confirms RIP routes are still being learned | Remote routes still appear with code `R` |
| `show ip route <REMOTE_PREFIX>` | Confirms exact next hop and route source | Route points through the expected transit router |
| `debug ip rip` | Confirms where RIP updates are sent and received | Updates appear on active transit interfaces, not passive LAN interfaces |
| `undebug all` | Stops active debugging | Debugging is disabled |
| `ping <REMOTE_LAN_IP>` | Verifies end-to-end reachability | ICMP succeeds |
| `traceroute <REMOTE_LAN_IP>` | Verifies forwarding path | Path follows the expected RIP transit path |

# RIP_Passive_Interface_Boundary_Control_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | R1/R2/R3 | `configure terminal` | Router enters global configuration mode |
| 2 | Enter RIP router configuration mode | R1/R2/R3 | `router rip` | Router enters RIP configuration mode |
| 3 | Remove default-passive behavior | R1/R2/R3 | `no passive-interface default` | RIP can send updates on RIP-enabled interfaces again |
| 4 | Remove specific passive-interface command if used | R1/R2/R3 | `no passive-interface <INTERFACE>` | Interface is no longer marked passive |
| 5 | Exit configuration mode | R1/R2/R3 | `end` | Router returns to privileged EXEC mode |
| 6 | Verify passive-interface state | R1/R2/R3 | `show ip protocols` | Passive-interface list no longer contains the removed interface or default-passive behavior |
| 7 | Save the rollback | R1/R2/R3 | `write memory` | Rollback persists after reload |

# RIP_Passive_Interface_Boundary_Control_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Neighbor no longer receives RIP routes | Transit interface was left passive | `show ip protocols` | Configure `no passive-interface <TRANSIT_INTERFACE>` |
| LAN route disappears from neighbors | LAN network was removed from RIP instead of only making the interface passive | `show running-config | section router rip` | Keep the LAN `network <LAN_MAJOR_NETWORK>` statement under RIP |
| RIP updates still leave a LAN interface | LAN interface was not made passive | `show ip protocols` and `debug ip rip` | Configure `passive-interface <LAN_INTERFACE>` or use `passive-interface default` |
| All RIP learning stops after enabling default-passive | No transit interfaces were re-enabled | `show running-config | section router rip` | Add `no passive-interface <TRANSIT_INTERFACE>` for each router-to-router RIP link |
| Route exists but ping fails | Return path is missing from the far router | `show ip route <SOURCE_PREFIX>` on the remote router | Confirm both sides still advertise LAN prefixes |
| Route exists but next hop is wrong | Another equal or better RIP path is being selected | `show ip route <REMOTE_PREFIX>` | Check topology, metrics, and offset-lists if present |
| RIP route is not installed | Competing route source has better administrative distance | `show ip route <PREFIX>` | Check static, OSPF, EIGRP, or BGP routes |
| Debug output is noisy on user-facing links | Passive-interface boundary was not applied | `debug ip rip` | Make access-facing interfaces passive |
| Passive-interface command appears correct but route is missing | Wrong RIP `network` statement | `show ip protocols` | Add the correct classful `network <MAJOR_NETWORK>` statement |
| Traffic dies at the middle router | Middle router has passive transit interface or missing route | `show ip protocols` and `show ip route rip` on the middle router | Re-enable RIP updates on the transit link and verify learned routes |

# Index
RIP_Passive_Interface_Boundary_Control.md
RIP_Passive_Interface_Boundary_Control
RIP_Passive_Interface_Boundary_Control_Mental_Model
RIP_Passive_Interface_Boundary_Control_Configuration_Checklist
RIP_Passive_Interface_Boundary_Control_Skeleton
RIP_Passive_Interface_Boundary_Control_Verification_Commands
RIP_Passive_Interface_Boundary_Control_Rollback
RIP_Passive_Interface_Boundary_Control_Failure_Checks