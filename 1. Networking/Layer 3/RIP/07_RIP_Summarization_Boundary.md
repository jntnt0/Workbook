RIP_Summarization_Boundary.md
# RIP_Summarization_Boundary

# RIP_Summarization_Boundary_Mental_Model
| Concept | Operational Meaning |
|---|---|
| RIP summarization | Multiple specific RIP routes are advertised as one shorter summary prefix |
| Boundary router | The router performing summarization must sit at the edge between detailed routing and summarized routing |
| Outbound interface control | Manual RIPv2 summarization is applied on the interface sending updates toward the summarized side |
| Specific route suppression | The summarizing router advertises the summary out the selected interface instead of advertising every more-specific route |
| RIPv2 requirement | Use RIPv2 because RIPv1 is classful and cannot reliably carry subnet mask information |
| No auto-summary | `no auto-summary` prevents unwanted classful summarization and keeps manual summaries intentional |
| Summary ownership | Only summarize prefixes that are actually reachable behind the summarizing router |
| Blackhole risk | If the summary covers networks the router cannot reach, traffic can be dropped or looped |
| Null0 safety route | A static discard route to `Null0` for the summary can protect against forwarding loops for missing more-specific routes |
| Verification target | Downstream routers see the summary route, not all specific routes, while the summarizing router still has valid more-specific reachability |

# RIP_Summarization_Boundary_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are up | SUMMARY_ROUTER | `show ip interface brief` | LAN, internal, and outbound RIP interfaces are up/up |
| 2 | Confirm baseline RIP is working | SUMMARY_ROUTER | `show running-config | section router rip` | RIP is configured with `version 2`, `no auto-summary`, and expected `network` statements |
| 3 | Confirm the specific routes exist before summarizing | SUMMARY_ROUTER | `show ip route <SPECIFIC_PREFIX>` | More-specific routes are present and reachable |
| 4 | Confirm all prefixes fit inside the intended summary | SUMMARY_ROUTER | `show ip route` | Specific routes fall within `<SUMMARY_PREFIX> <SUMMARY_MASK>` |
| 5 | Identify the outbound interface toward the summarized side | SUMMARY_ROUTER | `show ip route <DOWNSTREAM_PREFIX>` | Next-hop direction and outbound interface are known |
| 6 | Confirm downstream router currently sees specific routes | DOWNSTREAM_ROUTER | `show ip route rip` | More-specific routes appear as RIP routes before summarization |
| 7 | Enter global configuration mode | SUMMARY_ROUTER | `configure terminal` | Router enters global configuration mode |
| 8 | Confirm RIPv2 and disable auto-summary | SUMMARY_ROUTER | `router rip` | Router enters RIP configuration mode |
| 9 | Force RIPv2 | SUMMARY_ROUTER | `version 2` | RIP uses RIPv2 updates |
| 10 | Disable automatic classful summarization | SUMMARY_ROUTER | `no auto-summary` | RIP does not perform unwanted classful summarization |
| 11 | Confirm RIP network activation for specific routes | SUMMARY_ROUTER | `network <INTERNAL_MAJOR_NETWORK>` | Internal prefixes remain advertised into RIP |
| 12 | Confirm RIP network activation toward downstream router | SUMMARY_ROUTER | `network <OUTBOUND_MAJOR_NETWORK>` | RIP runs on the outbound interface |
| 13 | Exit to global configuration mode | SUMMARY_ROUTER | `exit` | Router returns to global configuration mode |
| 14 | Enter the outbound interface toward the summarized side | SUMMARY_ROUTER | `interface <OUTBOUND_INTERFACE>` | Router enters interface configuration mode |
| 15 | Apply manual RIPv2 summary on the outbound interface | SUMMARY_ROUTER | `ip summary-address rip <SUMMARY_PREFIX> <SUMMARY_MASK>` | Summary route is advertised out this interface |
| 16 | Exit interface configuration mode | SUMMARY_ROUTER | `exit` | Router returns to global configuration mode |
| 17 | Add a protective discard route for the summary if the lab requires loop protection | SUMMARY_ROUTER | `ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0` | Missing more-specific traffic inside the summary is discarded locally |
| 18 | Exit configuration mode | SUMMARY_ROUTER | `end` | Router returns to privileged EXEC mode |
| 19 | Clear RIP routes if immediate testing is needed | DOWNSTREAM_ROUTER | `clear ip route *` | Downstream router relearns routes after convergence |
| 20 | Verify summary interface configuration | SUMMARY_ROUTER | `show running-config interface <OUTBOUND_INTERFACE>` | Interface shows `ip summary-address rip <SUMMARY_PREFIX> <SUMMARY_MASK>` |
| 21 | Verify RIP process state | SUMMARY_ROUTER | `show ip protocols` | RIP shows RIPv2, no auto-summary, and expected active networks |
| 22 | Verify local specific route ownership | SUMMARY_ROUTER | `show ip route <SPECIFIC_PREFIX>` | More-specific routes remain reachable on the summarizing router |
| 23 | Verify downstream receives the summary | DOWNSTREAM_ROUTER | `show ip route <SUMMARY_PREFIX>` | Summary route appears as a RIP route via the summary router |
| 24 | Verify downstream no longer receives suppressed specifics | DOWNSTREAM_ROUTER | `show ip route rip` | Summary route appears instead of the individual more-specific routes from that boundary |
| 25 | Verify forwarding into reachable specific networks | DOWNSTREAM_ROUTER | `ping <INSIDE_SPECIFIC_DESTINATION>` | Ping succeeds to reachable networks inside the summary |
| 26 | Verify forwarding path into the summary | DOWNSTREAM_ROUTER | `traceroute <INSIDE_SPECIFIC_DESTINATION>` | Path goes toward the summary router |
| 27 | Verify summary advertisement during troubleshooting | SUMMARY_ROUTER | `debug ip rip` | RIP update shows the summary prefix advertised out the selected interface |
| 28 | Stop debugging | SUMMARY_ROUTER | `undebug all` | Debugging is disabled |
| 29 | Save the configuration | SUMMARY_ROUTER | `write memory` | Summary configuration persists after reload |

# RIP_Summarization_Boundary_Skeleton
| Device | Configuration |
|---|---|
| SUMMARY_ROUTER | `configure terminal` |
| SUMMARY_ROUTER | `router rip` |
| SUMMARY_ROUTER | `version 2` |
| SUMMARY_ROUTER | `no auto-summary` |
| SUMMARY_ROUTER | `network <INTERNAL_MAJOR_NETWORK>` |
| SUMMARY_ROUTER | `network <OUTBOUND_MAJOR_NETWORK>` |
| SUMMARY_ROUTER | `exit` |
| SUMMARY_ROUTER | `interface <OUTBOUND_INTERFACE>` |
| SUMMARY_ROUTER | `ip summary-address rip <SUMMARY_PREFIX> <SUMMARY_MASK>` |
| SUMMARY_ROUTER | `exit` |
| SUMMARY_ROUTER | `ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0` |
| SUMMARY_ROUTER | `end` |
| SUMMARY_ROUTER | `write memory` |

# RIP_Summarization_Boundary_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state | Internal and outbound RIP interfaces are up/up |
| `show running-config | section router rip` | Confirms RIP baseline | Shows `router rip`, `version 2`, `no auto-summary`, and expected `network` statements |
| `show running-config interface <OUTBOUND_INTERFACE>` | Confirms manual RIP summary | Shows `ip summary-address rip <SUMMARY_PREFIX> <SUMMARY_MASK>` |
| `show ip protocols` | Confirms RIP process behavior | RIP is active on the correct networks |
| `show ip route <SPECIFIC_PREFIX>` | Confirms the summary router still owns specific reachability | Specific route is present and points toward the correct internal path |
| `show ip route <SUMMARY_PREFIX>` | Confirms summary route presence | Downstream router sees the summary as a RIP route |
| `show ip route rip` | Confirms route table effect | Downstream router sees the summary instead of all suppressed specifics |
| `show ip route static` | Confirms protective discard route if used | Summary static route to `Null0` appears |
| `ping <INSIDE_SPECIFIC_DESTINATION>` | Tests reachability through the summary | ICMP succeeds |
| `traceroute <INSIDE_SPECIFIC_DESTINATION>` | Confirms forwarding direction | Path goes toward the summary router |
| `debug ip rip` | Shows live RIP advertisements | Summary prefix is advertised out the selected interface |
| `undebug all` | Stops active debugging | Debugging is disabled |

# RIP_Summarization_Boundary_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | SUMMARY_ROUTER | `configure terminal` | Router enters global configuration mode |
| 2 | Enter the outbound interface | SUMMARY_ROUTER | `interface <OUTBOUND_INTERFACE>` | Router enters interface configuration mode |
| 3 | Remove manual RIP summary | SUMMARY_ROUTER | `no ip summary-address rip <SUMMARY_PREFIX> <SUMMARY_MASK>` | Specific RIP routes can be advertised again out the interface |
| 4 | Exit interface configuration mode | SUMMARY_ROUTER | `exit` | Router returns to global configuration mode |
| 5 | Remove protective discard route if it was added only for the summary | SUMMARY_ROUTER | `no ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0` | Static discard route is removed |
| 6 | Exit configuration mode | SUMMARY_ROUTER | `end` | Router returns to privileged EXEC mode |
| 7 | Clear route table if immediate testing is needed | DOWNSTREAM_ROUTER | `clear ip route *` | Downstream router relearns routes |
| 8 | Verify summary command is gone | SUMMARY_ROUTER | `show running-config interface <OUTBOUND_INTERFACE>` | `ip summary-address rip` command is absent |
| 9 | Verify downstream sees specifics again | DOWNSTREAM_ROUTER | `show ip route rip` | More-specific RIP routes appear again after convergence |
| 10 | Save rollback | SUMMARY_ROUTER | `write memory` | Rollback persists after reload |

# RIP_Summarization_Boundary_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Downstream router still sees all specific routes | Summary applied on the wrong interface | `show running-config interface <OUTBOUND_INTERFACE>` | Apply `ip summary-address rip` on the interface sending updates toward the downstream router |
| Downstream router sees no summary | RIP is not active on the outbound interface | `show ip protocols` | Add the correct `network <OUTBOUND_MAJOR_NETWORK>` under `router rip` |
| Summary is not advertised | Missing specific routes inside the summary | `show ip route <SPECIFIC_PREFIX>` | Restore internal reachability to the specific prefixes |
| Summary is not advertised correctly | Wrong summary mask or network address | `show running-config interface <OUTBOUND_INTERFACE>` | Correct `<SUMMARY_PREFIX> <SUMMARY_MASK>` |
| Specific networks become unreachable | Summary covers networks the router does not actually reach | `show ip route` on SUMMARY_ROUTER | Narrow the summary or restore missing specific routes |
| Traffic loops after summarization | Summary route attracts traffic for missing specifics without a discard route | `traceroute <MISSING_SPECIFIC_DESTINATION>` | Add `ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0` or correct the summary boundary |
| Route table shows classful behavior | Auto-summary is still enabled | `show ip protocols` | Configure `no auto-summary` under `router rip` |
| Summary appears but ping fails | Return path missing | `show ip route <SOURCE_PREFIX>` on inside router | Ensure return routing exists toward downstream networks |
| Summary appears but wrong path is used | Another route with better administrative distance or lower RIP metric exists | `show ip route <DESTINATION>` | Remove or adjust competing route source |
| Debug shows specifics sent out boundary | Summary is missing or not applied on the outbound interface | `debug ip rip` | Apply summary on the correct outbound interface |
| Downstream route does not update quickly | RIP convergence timer delay | `show ip route rip` | Wait for convergence or clear the affected route in a lab |
| Configuration disappears after reload | Running config was not saved | `show startup-config interface <OUTBOUND_INTERFACE>` | Reconfigure and run `write memory` |

# Index
RIP_Summarization_Boundary.md
RIP_Summarization_Boundary
RIP_Summarization_Boundary_Mental_Model
RIP_Summarization_Boundary_Configuration_Checklist
RIP_Summarization_Boundary_Skeleton
RIP_Summarization_Boundary_Verification_Commands
RIP_Summarization_Boundary_Rollback
RIP_Summarization_Boundary_Failure_Checks