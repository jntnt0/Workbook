RIP_Default_Route_Origination.md
# RIP_Default_Route_Origination

# RIP_Default_Route_Origination_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Default route | `0.0.0.0/0` is the route of last resort when no more specific route matches |
| Default originator | The edge router injects the default route into the RIP domain |
| Downstream routers | Internal RIP routers learn the default route as a RIP route and point toward the edge router |
| RIPv2 requirement | Use `version 2` and `no auto-summary` so the RIP domain stays classless and predictable |
| Origination trigger | The edge router should have a valid local default route before advertising default information |
| RIP advertisement | `default-information originate` tells RIP to advertise a default route to RIP neighbors |
| Gateway of last resort | The local routing table should show a valid gateway of last resort on the edge router |
| Route code expectation | Downstream routers should show the default route as `R* 0.0.0.0/0` or equivalent RIP-learned default output |
| Failure boundary | If downstream routers lack the default, check the edge router first, then RIP activation on the transit link |
| Verification target | Edge router owns the upstream default, advertises it through RIP, and downstream routers use it for unknown destinations |

# RIP_Default_Route_Origination_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm edge router interfaces are up | EDGE | `show ip interface brief` | Inside RIP-facing interface and upstream-facing interface are up/up |
| 2 | Confirm downstream router interfaces are up | R2/R3 | `show ip interface brief` | RIP-facing transit and LAN interfaces are up/up |
| 3 | Confirm current connected routes | EDGE/R2/R3 | `show ip route connected` | Local LAN and transit networks appear as connected |
| 4 | Configure the edge router upstream default route | EDGE | `configure terminal` | EDGE enters global configuration mode |
| 5 | Add the static default route toward the upstream next hop | EDGE | `ip route 0.0.0.0 0.0.0.0 <UPSTREAM_NEXT_HOP>` | EDGE has a candidate default route in the routing table |
| 6 | Start or enter RIP on the edge router | EDGE | `router rip` | EDGE enters RIP router configuration mode |
| 7 | Force RIPv2 on the edge router | EDGE | `version 2` | EDGE uses RIPv2 updates |
| 8 | Disable auto-summary on the edge router | EDGE | `no auto-summary` | EDGE does not classfully summarize RIP advertisements |
| 9 | Activate RIP on the inside RIP-facing major network | EDGE | `network <EDGE_INSIDE_MAJOR_NETWORK>` | RIP runs on the edge interface facing downstream routers |
| 10 | Originate the default route into RIP | EDGE | `default-information originate` | EDGE advertises `0.0.0.0/0` into the RIP domain |
| 11 | Optional, make non-RIP upstream interface passive or keep it out of RIP | EDGE | `passive-interface <UPSTREAM_INTERFACE>` | EDGE does not send RIP updates toward the upstream network |
| 12 | Exit and save edge configuration | EDGE | `end` then `write memory` | EDGE default route and RIP origination are saved |
| 13 | Enter global configuration mode on downstream router | R2/R3 | `configure terminal` | Downstream router enters global configuration mode |
| 14 | Start or enter RIP on downstream router | R2/R3 | `router rip` | Router enters RIP router configuration mode |
| 15 | Force RIPv2 on downstream router | R2/R3 | `version 2` | Router uses RIPv2 updates |
| 16 | Disable auto-summary on downstream router | R2/R3 | `no auto-summary` | Router does not classfully summarize RIP advertisements |
| 17 | Activate RIP on the transit major network toward EDGE | R2/R3 | `network <EDGE_TRANSIT_MAJOR_NETWORK>` | RIP runs on the transit interface toward EDGE |
| 18 | Activate RIP on downstream LAN major network | R2/R3 | `network <DOWNSTREAM_LAN_MAJOR_NETWORK>` | Downstream LAN is advertised into RIP |
| 19 | Optional, suppress RIP updates on host-facing LAN | R2/R3 | `passive-interface <LAN_INTERFACE>` | LAN prefix is advertised, but RIP updates are not sent toward hosts |
| 20 | Exit and save downstream configuration | R2/R3 | `end` then `write memory` | Downstream RIP configuration is saved |
| 21 | Verify edge gateway of last resort | EDGE | `show ip route 0.0.0.0` | EDGE shows a valid default route toward the upstream next hop |
| 22 | Verify edge RIP configuration | EDGE | `show running-config | section router rip` | Output shows `router rip`, `version 2`, `no auto-summary`, `network`, and `default-information originate` |
| 23 | Verify edge RIP process behavior | EDGE | `show ip protocols` | RIP is enabled on the intended inside network |
| 24 | Verify downstream learned default route | R2/R3 | `show ip route 0.0.0.0` | Downstream router shows a RIP-learned default route toward EDGE |
| 25 | Verify downstream RIP routes | R2/R3 | `show ip route rip` | Default route and expected remote RIP routes appear |
| 26 | Test downstream unknown-destination forwarding | R2/R3 | `ping <UPSTREAM_TEST_IP>` | Ping succeeds if upstream reachability and return path exist |
| 27 | Trace downstream path to upstream destination | R2/R3 | `traceroute <UPSTREAM_TEST_IP>` | First hop points toward EDGE |
| 28 | Confirm RIP default advertisement during troubleshooting | EDGE | `debug ip rip` | EDGE sends default route information in RIP updates |
| 29 | Stop debugging | EDGE | `undebug all` | Debugging is disabled |

# RIP_Default_Route_Origination_Skeleton
| Device | Configuration |
|---|---|
| EDGE | `configure terminal` |
| EDGE | `ip route 0.0.0.0 0.0.0.0 <UPSTREAM_NEXT_HOP>` |
| EDGE | `router rip` |
| EDGE | `version 2` |
| EDGE | `no auto-summary` |
| EDGE | `network <EDGE_INSIDE_MAJOR_NETWORK>` |
| EDGE | `default-information originate` |
| EDGE | `passive-interface <UPSTREAM_INTERFACE>` |
| EDGE | `end` |
| EDGE | `write memory` |
| R2 | `configure terminal` |
| R2 | `router rip` |
| R2 | `version 2` |
| R2 | `no auto-summary` |
| R2 | `network <EDGE_TRANSIT_MAJOR_NETWORK>` |
| R2 | `network <R2_LAN_MAJOR_NETWORK>` |
| R2 | `passive-interface <R2_LAN_INTERFACE>` |
| R2 | `end` |
| R2 | `write memory` |
| R3 | `configure terminal` |
| R3 | `router rip` |
| R3 | `version 2` |
| R3 | `no auto-summary` |
| R3 | `network <RIP_TRANSIT_MAJOR_NETWORK>` |
| R3 | `network <R3_LAN_MAJOR_NETWORK>` |
| R3 | `passive-interface <R3_LAN_INTERFACE>` |
| R3 | `end` |
| R3 | `write memory` |

# RIP_Default_Route_Origination_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state | Required interfaces are up/up |
| `show ip route connected` | Confirms local connected networks exist | Transit and LAN prefixes appear as connected |
| `show ip route 0.0.0.0` | Confirms default route presence | EDGE has a static default; downstream routers have RIP-learned default |
| `show ip route` | Confirms gateway of last resort | Gateway of last resort is set |
| `show ip route rip` | Displays RIP-learned routes | Downstream routers show RIP default route and other remote RIP routes |
| `show running-config | section router rip` | Confirms RIP default origination configuration | EDGE shows `default-information originate` |
| `show ip protocols` | Confirms RIP process behavior | RIP is routing for the expected networks |
| `show ip route <UPSTREAM_TEST_PREFIX>` | Confirms route selection for an upstream destination | Downstream router uses default route toward EDGE |
| `ping <UPSTREAM_TEST_IP>` | Tests data-plane reachability through the default route | ICMP succeeds if upstream and return path are valid |
| `traceroute <UPSTREAM_TEST_IP>` | Confirms forwarding direction | First hop from downstream router is EDGE |
| `debug ip rip` | Confirms RIP updates include default route information | EDGE advertises default route to RIP neighbors |
| `undebug all` | Stops active debugging | Debugging is disabled |

# RIP_Default_Route_Origination_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode on EDGE | EDGE | `configure terminal` | EDGE enters global configuration mode |
| 2 | Enter RIP router configuration mode | EDGE | `router rip` | EDGE enters RIP configuration mode |
| 3 | Stop originating the default route into RIP | EDGE | `no default-information originate` | EDGE no longer advertises default route through RIP |
| 4 | Exit RIP router configuration mode | EDGE | `exit` | EDGE returns to global configuration mode |
| 5 | Remove the static default route if the lab requires full rollback | EDGE | `no ip route 0.0.0.0 0.0.0.0 <UPSTREAM_NEXT_HOP>` | Static default route is removed from EDGE |
| 6 | Exit configuration mode | EDGE | `end` | EDGE returns to privileged EXEC mode |
| 7 | Confirm EDGE no longer originates default | EDGE | `show running-config | section router rip` | `default-information originate` is absent |
| 8 | Confirm downstream default route disappears | R2/R3 | `show ip route 0.0.0.0` | RIP-learned default route is removed after RIP convergence |
| 9 | Save rollback | EDGE/R2/R3 | `write memory` | Rollback persists after reload |

# RIP_Default_Route_Origination_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Downstream router has no default route | EDGE is not originating default into RIP | `show running-config | section router rip` on EDGE | Add `default-information originate` under `router rip` |
| EDGE has no gateway of last resort | Missing static default route or upstream route | `show ip route 0.0.0.0` on EDGE | Add `ip route 0.0.0.0 0.0.0.0 <UPSTREAM_NEXT_HOP>` |
| EDGE default route points the wrong way | Wrong upstream next hop configured | `show ip route 0.0.0.0` | Correct the static default route |
| Downstream router learns normal RIP routes but not default | Default origination is missing or filtered | `show ip protocols` and `show running-config | section router rip` | Add default origination or remove outbound filter blocking `0.0.0.0` |
| Downstream router receives no RIP routes at all | RIP not active on the EDGE-to-downstream transit link | `show ip protocols` | Add the correct `network <TRANSIT_MAJOR_NETWORK>` on both routers |
| Default route appears but ping fails | Upstream network lacks return route | `traceroute <UPSTREAM_TEST_IP>` and upstream routing table | Add return route, NAT, or upstream routing as required |
| Default route appears but traffic uses another path | More specific route or better administrative distance route exists | `show ip route <DESTINATION>` | Remove or adjust competing route |
| EDGE advertises default toward the upstream side | Upstream interface participates in RIP | `show ip protocols` | Keep upstream interface out of RIP or configure it passive |
| Default route flaps | Static default next hop is unstable | `show ip route 0.0.0.0` and interface counters | Fix upstream link, next hop, or tracking design |
| Debug shows RIP updates but no default | EDGE is not injecting `0.0.0.0` into RIP | `debug ip rip` | Confirm `default-information originate` and local default route |
| Route is learned but marked unreachable | RIP metric reached 16 | `show ip route rip` and `debug ip rip` | Fix excessive hop count or routing loop |
| Default route disappears after reload | Configuration was not saved | `show startup-config | section router rip` | Reconfigure and run `write memory` |

# Index
RIP_Default_Route_Origination.md
RIP_Default_Route_Origination
RIP_Default_Route_Origination_Mental_Model
RIP_Default_Route_Origination_Configuration_Checklist
RIP_Default_Route_Origination_Skeleton
RIP_Default_Route_Origination_Verification_Commands
RIP_Default_Route_Origination_Rollback
RIP_Default_Route_Origination_Failure_Checks