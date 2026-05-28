RIP_Offset_List_Metric_Control.md
# RIP_Offset_List_Metric_Control

# RIP_Offset_List_Metric_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Offset-list | Adds hop-count cost to selected RIP routes |
| Metric control | RIP only uses hop count, so offset-lists steer traffic by making a route look farther away |
| Inbound offset | Changes the metric of routes as they are received by the local router |
| Outbound offset | Changes the metric of routes as they are advertised to neighbors |
| ACL match | The access list identifies which route prefixes receive the offset |
| Interface scope | Optional interface binding limits the offset-list to routes received on or sent out a specific interface |
| RIP hop limit | A route with metric 16 is unreachable |
| Traffic steering | Offset-lists can prefer one RIP path over another by increasing the metric on the less-preferred path |
| Route poisoning risk | A large offset can accidentally push a route to 16 and make it unreachable |
| Verification target | The selected prefix appears with the intended RIP metric, next hop changes if expected, and end-to-end reachability still works |

# RIP_Offset_List_Metric_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are up | R1/R2/R3 | `show ip interface brief` | All RIP transit and LAN interfaces are up/up |
| 2 | Confirm baseline RIP configuration | R1/R2/R3 | `show running-config | section router rip` | RIP uses `version 2`, `no auto-summary`, and expected `network` statements |
| 3 | Confirm current RIP routes before metric change | TARGET_ROUTER | `show ip route rip` | RIP routes are installed with current hop-count metrics |
| 4 | Confirm the current metric for the specific route | TARGET_ROUTER | `show ip route <PREFIX>` | Route shows current RIP metric and next hop |
| 5 | Identify the prefix to manipulate | TARGET_ROUTER | `show ip route rip` | Prefix to be offset is known |
| 6 | Identify the direction of control | TARGET_ROUTER | `show ip route <PREFIX>` | Use `in` if changing received routes; use `out` if changing advertised routes |
| 7 | Identify the interface scope if needed | TARGET_ROUTER | `show ip route <PREFIX>` | Interface used for the selected route is known |
| 8 | Enter global configuration mode | TARGET_ROUTER | `configure terminal` | Router enters global configuration mode |
| 9 | Create a standard ACL matching the RIP route prefix | TARGET_ROUTER | `access-list <ACL_NUMBER> permit <PREFIX_NETWORK> <WILDCARD_MASK>` | ACL matches the route prefix that should receive added metric |
| 10 | Enter RIP router configuration mode | TARGET_ROUTER | `router rip` | Router enters RIP configuration mode |
| 11 | Apply an inbound offset-list to selected learned routes | TARGET_ROUTER | `offset-list <ACL_NUMBER> in <OFFSET_VALUE> <INTERFACE>` | Matching routes received on the interface have their metric increased |
| 12 | Apply an outbound offset-list to selected advertised routes | TARGET_ROUTER | `offset-list <ACL_NUMBER> out <OFFSET_VALUE> <INTERFACE>` | Matching routes advertised out the interface have their metric increased |
| 13 | Keep RIPv2 enabled | TARGET_ROUTER | `version 2` | Router sends and receives RIPv2 updates |
| 14 | Keep auto-summary disabled | TARGET_ROUTER | `no auto-summary` | Router does not classfully summarize RIP routes |
| 15 | Exit configuration mode | TARGET_ROUTER | `end` | Router returns to privileged EXEC mode |
| 16 | Clear the affected RIP route if immediate reconvergence is needed | TARGET_ROUTER | `clear ip route <PREFIX>` | Route is relearned with the new metric |
| 17 | Verify the updated metric locally | TARGET_ROUTER | `show ip route <PREFIX>` | RIP route shows the expected higher metric |
| 18 | Verify the updated RIP table | TARGET_ROUTER | `show ip route rip` | Matching prefix has adjusted metric; unrelated routes remain unchanged |
| 19 | Verify neighbor route impact if using outbound offset | NEIGHBOR_ROUTER | `show ip route <PREFIX>` | Neighbor sees the adjusted RIP metric |
| 20 | Verify path selection changed if that was the goal | TARGET_ROUTER | `show ip route <PREFIX>` | Next hop changes to preferred path when an alternate equal or better route exists |
| 21 | Verify forwarding still works | TARGET_ROUTER | `ping <DESTINATION_IP>` | Ping succeeds |
| 22 | Verify actual forwarding path | TARGET_ROUTER | `traceroute <DESTINATION_IP>` | Path follows the intended RIP next hop |
| 23 | Verify RIP updates during troubleshooting | TARGET_ROUTER | `debug ip rip` | Debug shows prefix advertised or received with adjusted metric |
| 24 | Stop debugging | TARGET_ROUTER | `undebug all` | Debugging is disabled |
| 25 | Save the configuration | TARGET_ROUTER | `write memory` | Offset-list configuration persists after reload |

# RIP_Offset_List_Metric_Control_Skeleton
| Device | Configuration |
|---|---|
| TARGET_ROUTER | `configure terminal` |
| TARGET_ROUTER | `access-list <ACL_NUMBER> permit <PREFIX_NETWORK> <WILDCARD_MASK>` |
| TARGET_ROUTER | `router rip` |
| TARGET_ROUTER | `version 2` |
| TARGET_ROUTER | `no auto-summary` |
| TARGET_ROUTER | `offset-list <ACL_NUMBER> in <OFFSET_VALUE> <INTERFACE>` |
| TARGET_ROUTER | `offset-list <ACL_NUMBER> out <OFFSET_VALUE> <INTERFACE>` |
| TARGET_ROUTER | `end` |
| TARGET_ROUTER | `clear ip route <PREFIX>` |
| TARGET_ROUTER | `show ip route <PREFIX>` |
| TARGET_ROUTER | `show ip route rip` |
| TARGET_ROUTER | `write memory` |

# RIP_Offset_List_Metric_Control_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state | RIP transit interfaces are up/up |
| `show running-config | section router rip` | Confirms RIP and offset-list configuration | Shows `router rip`, `version 2`, `no auto-summary`, and the expected `offset-list` |
| `show access-lists` | Confirms route-matching ACL | ACL permits the intended prefix |
| `show ip protocols` | Confirms RIP process behavior | RIP is enabled on expected networks |
| `show ip route rip` | Confirms RIP-learned routes | Target route appears with adjusted metric |
| `show ip route <PREFIX>` | Confirms exact route metric and next hop | Route shows expected `[120/<METRIC>] via <NEXT_HOP>` |
| `clear ip route <PREFIX>` | Forces the route to be relearned | Prefix is removed and relearned with adjusted metric |
| `ping <DESTINATION_IP>` | Tests data-plane reachability | ICMP succeeds |
| `traceroute <DESTINATION_IP>` | Confirms forwarding path | Path follows intended next hop |
| `debug ip rip` | Shows live RIP route advertisements and metrics | Matching prefix appears with expected offset metric |
| `undebug all` | Stops active debugging | Debugging is disabled |

# RIP_Offset_List_Metric_Control_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | TARGET_ROUTER | `configure terminal` | Router enters global configuration mode |
| 2 | Enter RIP router configuration mode | TARGET_ROUTER | `router rip` | Router enters RIP configuration mode |
| 3 | Remove inbound offset-list | TARGET_ROUTER | `no offset-list <ACL_NUMBER> in <OFFSET_VALUE> <INTERFACE>` | Inbound metric manipulation is removed |
| 4 | Remove outbound offset-list | TARGET_ROUTER | `no offset-list <ACL_NUMBER> out <OFFSET_VALUE> <INTERFACE>` | Outbound metric manipulation is removed |
| 5 | Exit RIP router configuration mode | TARGET_ROUTER | `exit` | Router returns to global configuration mode |
| 6 | Remove the ACL if it is only used for this offset-list | TARGET_ROUTER | `no access-list <ACL_NUMBER>` | ACL is removed |
| 7 | Exit configuration mode | TARGET_ROUTER | `end` | Router returns to privileged EXEC mode |
| 8 | Clear affected route | TARGET_ROUTER | `clear ip route <PREFIX>` | Route is relearned without offset manipulation |
| 9 | Verify metric returns to baseline | TARGET_ROUTER | `show ip route <PREFIX>` | RIP metric returns to normal hop count |
| 10 | Verify offset-list is absent | TARGET_ROUTER | `show running-config | section router rip` | Removed offset-list no longer appears |
| 11 | Save rollback | TARGET_ROUTER | `write memory` | Rollback persists after reload |

# RIP_Offset_List_Metric_Control_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Route metric does not change | ACL does not match the route prefix | `show access-lists` | Correct the ACL network and wildcard mask |
| Route metric does not change | Offset-list applied in wrong direction | `show running-config | section router rip` | Use `in` for received routes or `out` for advertised routes |
| Route metric does not change | Offset-list bound to wrong interface | `show ip route <PREFIX>` | Apply offset-list to the correct receive or advertise interface |
| Route disappears | Offset pushed RIP metric to 16 | `show ip route rip` and `debug ip rip` | Lower the offset value so final metric stays below 16 |
| Neighbor does not see changed metric | Offset-list was applied inbound instead of outbound | `show running-config | section router rip` on advertising router | Apply outbound offset-list on the advertising router |
| Local route does not change | Offset-list was applied outbound instead of inbound | `show running-config | section router rip` on local router | Apply inbound offset-list on the receiving router |
| Wrong prefixes are affected | ACL is too broad | `show access-lists` and `show ip route rip` | Narrow the ACL to the exact route prefix |
| Unrelated route path changes | Offset-list matched more routes than intended | `show ip route rip` | Use a more specific ACL or interface-scoped offset-list |
| Path does not change after metric increase | No alternate route exists or alternate route is still worse | `show ip route <PREFIX>` | Confirm an alternate RIP path exists with better metric |
| Route still prefers old path | Offset value is too small | `show ip route <PREFIX>` | Increase offset enough to make the alternate path better |
| Route is not installed at all | Better administrative distance route wins | `show ip route <PREFIX>` | Remove or adjust competing static, OSPF, EIGRP, or BGP route |
| Configuration disappears after reload | Running config was not saved | `show startup-config | section router rip` | Reconfigure and run `write memory` |

# Index
RIP_Offset_List_Metric_Control.md
RIP_Offset_List_Metric_Control
RIP_Offset_List_Metric_Control_Mental_Model
RIP_Offset_List_Metric_Control_Configuration_Checklist
RIP_Offset_List_Metric_Control_Skeleton
RIP_Offset_List_Metric_Control_Verification_Commands
RIP_Offset_List_Metric_Control_Rollback
RIP_Offset_List_Metric_Control_Failure_Checks