RIP_RIPv2_Core_And_Network_Activation.md
# RIP_RIPv2_Core_And_Network_Activation

# RIP_RIPv2_Core_And_Network_Activation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| RIP process | `router rip` starts the RIP routing process on the router |
| RIPv2 | `version 2` enables classless RIP updates that carry subnet mask information |
| Network activation | The `network` command enables RIP on interfaces whose IP address belongs to that major network |
| Classful network statement | RIP `network` statements do not use wildcard masks; use the major network such as `10.0.0.0`, `172.16.0.0`, or `192.168.1.0` |
| Route advertisement | Interfaces activated by `network` send RIP updates and advertise connected routes through RIP |
| No auto-summary | `no auto-summary` prevents automatic classful summarization at major network boundaries |
| Distance-vector behavior | RIP routers learn routes from neighbor updates, increment hop count, and install the lowest-hop route |
| Hop-count limit | RIP treats 15 hops as reachable and 16 hops as unreachable |
| Verification target | A working baseline means interfaces are up, RIP is enabled on the correct links, RIPv2 is active, auto-summary is disabled, and remote routes appear as `R` routes |

# RIP_RIPv2_Core_And_Network_Activation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the routed interfaces are up before enabling RIP | R1/R2 | `show ip interface brief` | Transit and LAN interfaces are up/up with the expected IPv4 addresses |
| 2 | Confirm the connected routes that RIP should advertise | R1/R2 | `show ip route connected` | Local LAN and transit prefixes appear as connected routes |
| 3 | Enter global configuration mode | R1/R2 | `configure terminal` | Router enters global configuration mode |
| 4 | Start the RIP process | R1/R2 | `router rip` | Router enters RIP router configuration mode |
| 5 | Force RIP version 2 | R1/R2 | `version 2` | Router sends and receives RIPv2 updates |
| 6 | Disable classful automatic summarization | R1/R2 | `no auto-summary` | RIP advertises subnet routes instead of auto-summarizing at major network boundaries |
| 7 | Activate RIP on the transit major network | R1/R2 | `network <TRANSIT_MAJOR_NETWORK>` | RIP runs on the interface whose IP belongs to the transit network |
| 8 | Activate RIP on the local LAN major network | R1 | `network <R1_LAN_MAJOR_NETWORK>` | R1 advertises its local LAN into RIP |
| 9 | Activate RIP on the remote LAN major network | R2 | `network <R2_LAN_MAJOR_NETWORK>` | R2 advertises its local LAN into RIP |
| 10 | Exit router configuration mode | R1/R2 | `end` | Router returns to privileged EXEC mode |
| 11 | Save the configuration | R1/R2 | `write memory` | RIP configuration persists after reload |
| 12 | Verify RIP process settings | R1/R2 | `show ip protocols` | Output shows RIP, version 2, no auto-summary behavior, and the correct routing-for-networks entries |
| 13 | Verify learned RIP routes | R1/R2 | `show ip route rip` | Remote prefixes appear with route code `R` |
| 14 | Verify end-to-end forwarding | Host/R1/R2 | `ping <REMOTE_LAN_IP>` | Ping succeeds across the RIP-learned path |
| 15 | Verify forwarding path if ping fails | R1/R2 | `traceroute <REMOTE_LAN_IP>` | Traffic follows the expected next hop through the RIP transit link |

# RIP_RIPv2_Core_And_Network_Activation_Skeleton
| Device | Configuration |
|---|---|
| R1 | `configure terminal` |
| R1 | `router rip` |
| R1 | `version 2` |
| R1 | `no auto-summary` |
| R1 | `network <R1_TRANSIT_MAJOR_NETWORK>` |
| R1 | `network <R1_LAN_MAJOR_NETWORK>` |
| R1 | `end` |
| R1 | `write memory` |
| R2 | `configure terminal` |
| R2 | `router rip` |
| R2 | `version 2` |
| R2 | `no auto-summary` |
| R2 | `network <R2_TRANSIT_MAJOR_NETWORK>` |
| R2 | `network <R2_LAN_MAJOR_NETWORK>` |
| R2 | `end` |
| R2 | `write memory` |

# RIP_RIPv2_Core_And_Network_Activation_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state and addressing | Required interfaces are up/up |
| `show ip route connected` | Confirms local connected prefixes exist before advertising them | LAN and transit networks appear as connected |
| `show running-config | section router rip` | Confirms RIP configuration in the running config | Shows `router rip`, `version 2`, `no auto-summary`, and expected `network` statements |
| `show ip protocols` | Confirms RIP process behavior | Shows RIP is routing for the expected networks |
| `show ip route rip` | Confirms received RIP routes | Remote routes appear with code `R` |
| `show ip route <REMOTE_PREFIX>` | Confirms exact route selection | Route points to the expected RIP next hop |
| `ping <REMOTE_INTERFACE_OR_HOST>` | Tests reachability | ICMP succeeds |
| `traceroute <REMOTE_INTERFACE_OR_HOST>` | Tests forwarding path | Path follows the expected RIP next hop |
| `debug ip rip` | Live troubleshooting of RIP updates | Shows RIPv2 updates being sent and received |
| `undebug all` | Stops active debugging | Debugging is disabled |

# RIP_RIPv2_Core_And_Network_Activation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | R1/R2 | `configure terminal` | Router enters global configuration mode |
| 2 | Remove the RIP process completely | R1/R2 | `no router rip` | RIP process and RIP network statements are removed |
| 3 | Exit configuration mode | R1/R2 | `end` | Router returns to privileged EXEC mode |
| 4 | Confirm RIP is removed | R1/R2 | `show running-config | section router rip` | No RIP process appears |
| 5 | Confirm RIP routes are gone | R1/R2 | `show ip route rip` | No RIP-learned routes appear |
| 6 | Save the rollback | R1/R2 | `write memory` | Rollback persists after reload |

# RIP_RIPv2_Core_And_Network_Activation_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No RIP routes appear | RIP not activated on the transit interface | `show ip protocols` | Add the correct `network <TRANSIT_MAJOR_NETWORK>` statement |
| RIP runs on the wrong interface | Broad classful `network` statement matched more interfaces than intended | `show ip protocols` | Use passive-interface later for edge control, or redesign addressing boundaries |
| Remote subnet missing | LAN network not included under RIP | `show running-config | section router rip` | Add `network <LAN_MAJOR_NETWORK>` |
| Subnet is summarized incorrectly | Auto-summary still enabled | `show ip protocols` | Configure `no auto-summary` |
| Neighbor does not receive updates | Interface is down or wrong subnet/mask | `show ip interface brief` | Fix interface IP addressing or bring interface up |
| Route exists but ping fails | Return path missing | `show ip route <SOURCE_PREFIX>` on remote router | Make sure both sides advertise their LAN networks |
| Route exists but forwarding goes wrong way | Lower hop-count path is not the intended physical path | `show ip route <REMOTE_PREFIX>` | Adjust topology or use later metric-control mechanisms such as offset-lists |
| Debug shows v1 behavior | `version 2` missing | `show running-config | section router rip` | Add `version 2` under `router rip` |
| RIP route replaced by another protocol | Administrative distance preference favors another source | `show ip route <PREFIX>` | Check competing static, OSPF, EIGRP, or BGP route |
| RIP route marked unreachable | Hop count reached 16 | `show ip route rip` / `debug ip rip` | Reduce path length or fix loop/metric issue |

# Index
RIP_RIPv2_Core_And_Network_Activation.md
RIP_RIPv2_Core_And_Network_Activation
RIP_RIPv2_Core_And_Network_Activation_Mental_Model
RIP_RIPv2_Core_And_Network_Activation_Configuration_Checklist
RIP_RIPv2_Core_And_Network_Activation_Skeleton
RIP_RIPv2_Core_And_Network_Activation_Verification_Commands
RIP_RIPv2_Core_And_Network_Activation_Rollback
RIP_RIPv2_Core_And_Network_Activation_Failure_Checks