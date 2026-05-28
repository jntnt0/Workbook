
Used the local KB for the VRF-Lite baseline. I used Cisco’s official command reference for the ip route [vrf vrf-name] ... [global] static route syntax and Cisco’s GRT/VRF route-leak note for the static-leak behavior.  

VRF_Lite_Static_Route_Leaking.md

VRF_Lite_Static_Route_Leaking

# Source_Basis

| Source | Relevant Section | What It Supports |

|---|---|---|

| `_KB_INDEX.md` | Cisco Press / ENARSI source mapping | Confirms Cisco Press ENARSI material as the primary local source for VRF-Lite foundations |

| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 18, `Implementing and Verifying VRF-Lite` | Supports VRF isolation, VRF-specific routing tables, `show ip route vrf`, and `ping vrf` verification |

| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-14 | Shows that default pings use the global table unless the VRF is explicitly specified |

| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-15 | Shows that VRF tables initially contain only connected and local routes before static or dynamic routing is added |

| `Cisco IOS IP Routing: Protocol-Independent Command Reference` | `ip route` command | Provides `ip route [vrf vrf-name] prefix mask {ip-address | interface} [global]` static route syntax |

| `Cisco TAC: Configure Route Leak Between Global and VRF Routing Table without Next-Hop` | Background Information | Confirms that static routes can be used to leak between the global routing table and a VRF |

# VRF_Lite_Static_Route_Leaking_Mental_Model

| Concept | Operational Meaning |

|---|---|

| Static route leak | A manually configured exception that allows selected prefixes to cross routing-table boundaries |

| VRF isolation default | RED, GREEN, BLUE, and global do not share routes unless you explicitly leak routes |

| Direction matters | A leak from RED to global is not the same as a leak from global to RED |

| Return path required | One-way leaking usually creates one-way reachability failure |

| Receiving table | The static route is installed in the table that needs to learn the destination |

| `global` keyword | Used when a VRF static route must resolve the next hop through the global routing table |

| Global-to-VRF leak | Usually requires a global static route pointing toward the VRF-facing interface or next hop |

| VRF-to-global leak | Usually requires `ip route vrf <VRF> <prefix> <mask> <global-next-hop> global` |

| Static leak scaling limit | Static leaking is fine for small labs and controlled exceptions, but ugly at scale |

| Better scalable option | Use BGP route leaking with route targets when many prefixes or policy control are required |

# VRF_Lite_Static_Route_Leaking_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Confirm VRFs exist before leaking routes | Leak router | `show vrf` | Required VRFs appear with IPv4 enabled |

| 2 | Confirm interfaces are bound to the correct VRFs | Leak router | `show vrf ipv4 unicast interfaces` | RED, GREEN, BLUE, and global interfaces are in the expected routing tables |

| 3 | Confirm RED has only its normal local and learned routes | Leak router | `show ip route vrf RED` | RED does not yet contain the global or other-VRF destination prefix |

| 4 | Confirm the global table has only its normal routes | Leak router | `show ip route` | Global table does not yet contain the RED prefix unless already configured |

| 5 | Prove the destination is currently isolated from RED | Leak router | `ping vrf RED 192.0.2.10` | Ping fails before the static leak is installed |

| 6 | Prove the global table can reach the shared-service next hop | Leak router | `ping 203.0.113.1` | Global next hop replies |

| 7 | Install a RED-to-global static leak for a shared-service prefix | Leak router | `ip route vrf RED 192.0.2.0 255.255.255.0 203.0.113.1 global` | RED learns `192.0.2.0/24` and resolves the next hop through the global table |

| 8 | Verify the leaked route appears in RED | Leak router | `show ip route vrf RED 192.0.2.0 255.255.255.0` | RED table shows a static route for `192.0.2.0/24` |

| 9 | Verify RED forwarding resolves the leaked prefix | Leak router | `show ip cef vrf RED 192.0.2.10` | CEF shows a valid next hop or adjacency |

| 10 | Install the return route in the global table toward the RED prefix | Leak router | `ip route 10.10.10.0 255.255.255.0 GigabitEthernet0/0.100 10.0.100.2` | Global table learns how to return traffic to the RED subnet |

| 11 | Verify the global return route | Leak router | `show ip route 10.10.10.0 255.255.255.0` | Global table shows a static route toward the RED-facing interface or next hop |

| 12 | Verify global forwarding toward RED | Leak router | `show ip cef 10.10.10.10` | CEF shows a valid adjacency toward RED |

| 13 | Test RED-to-global shared-service reachability | Leak router | `ping vrf RED 192.0.2.10 source 10.10.10.1` | Ping succeeds if forward and return routes are correct |

| 14 | Test from the shared-service/global side back to RED | Global-side router | `ping 10.10.10.10` | Ping succeeds if the global return path is correct |

| 15 | Add a GREEN-to-global leak only if GREEN also needs the shared service | Leak router | `ip route vrf GREEN 192.0.2.0 255.255.255.0 203.0.113.1 global` | GREEN learns the shared-service prefix through the global table |

| 16 | Add a global return route for the GREEN subnet | Leak router | `ip route 172.16.10.0 255.255.255.0 GigabitEthernet0/0.200 172.16.100.2` | Global table learns the GREEN return path |

| 17 | Verify GREEN leaked route | Leak router | `show ip route vrf GREEN 192.0.2.0 255.255.255.0` | GREEN table shows the intended static leak |

| 18 | Verify GREEN reachability to the shared service | Leak router | `ping vrf GREEN 192.0.2.10 source 172.16.10.1` | Ping succeeds if both directions are routed |

| 19 | Confirm RED and GREEN are still isolated from each other unless intentionally leaked | Leak router | `ping vrf RED 172.16.10.10` | Ping fails unless a RED-to-GREEN leak was intentionally configured |

| 20 | Confirm leaked routes did not appear in unrelated VRFs | Leak router | `show ip route vrf BLUE 192.0.2.0 255.255.255.0` | BLUE does not contain the leaked route unless configured |

| 21 | Confirm no dynamic route leaking was accidentally configured | Leak router | `show running-config | include route-target|import|export|redistribute` | No unwanted BGP or redistribution leak exists |

| 22 | Confirm static route ownership | Leak router | `show running-config | include ^ip route` | Static leak commands are visible and match the intended prefixes |

| 23 | Save the working static leak baseline | Leak router | `write memory` | Static route leak survives reload |

# VRF_Lite_Static_Route_Leaking_Skeleton

  

configure terminal

  

! Pattern 1:

! RED VRF reaches a shared-service prefix in the global routing table.

! The "global" keyword tells the RED VRF route to resolve the next hop in the global table.

  

ip route vrf RED 192.0.2.0 255.255.255.0 203.0.113.1 global

  

! Return path:

! The global table must know how to return traffic to the RED subnet.

! For Ethernet or multi-access links, include the next hop when available.

  

ip route 10.10.10.0 255.255.255.0 GigabitEthernet0/0.100 10.0.100.2

  

  

! Pattern 2:

! GREEN VRF also reaches the same shared-service prefix in the global table.

  

ip route vrf GREEN 192.0.2.0 255.255.255.0 203.0.113.1 global

  

! Return path for GREEN.

  

ip route 172.16.10.0 255.255.255.0 GigabitEthernet0/0.200 172.16.100.2

  

  

! Optional Pattern 3:

! Default-route leak from a VRF to the global table.

! Use this when the VRF should use the global/default internet or service path.

  

ip route vrf RED 0.0.0.0 0.0.0.0 203.0.113.1 global

  

! Return route still required for RED source prefixes.

  

ip route 10.10.10.0 255.255.255.0 GigabitEthernet0/0.100 10.0.100.2

  

end

write memory

# VRF_Lite_Static_Route_Leaking_Verification_Commands

  

show vrf

show vrf ipv4 unicast interfaces

  

show ip route

show ip route vrf RED

show ip route vrf GREEN

show ip route vrf BLUE

  

show ip route vrf RED 192.0.2.0 255.255.255.0

show ip route vrf GREEN 192.0.2.0 255.255.255.0

show ip route 10.10.10.0 255.255.255.0

show ip route 172.16.10.0 255.255.255.0

  

show ip cef vrf RED 192.0.2.10

show ip cef vrf GREEN 192.0.2.10

show ip cef 10.10.10.10

show ip cef 172.16.10.10

  

show running-config | include ^ip route

show running-config | include route-target|import|export|redistribute

  

ping vrf RED 192.0.2.10 source 10.10.10.1

ping vrf GREEN 192.0.2.10 source 172.16.10.1

ping 10.10.10.10

ping 172.16.10.10

  

traceroute vrf RED 192.0.2.10 source 10.10.10.1

traceroute vrf GREEN 192.0.2.10 source 172.16.10.1

# VRF_Lite_Static_Route_Leaking_Rollback

  

configure terminal

  

no ip route vrf RED 192.0.2.0 255.255.255.0 203.0.113.1 global

no ip route 10.10.10.0 255.255.255.0 GigabitEthernet0/0.100 10.0.100.2

  

no ip route vrf GREEN 192.0.2.0 255.255.255.0 203.0.113.1 global

no ip route 172.16.10.0 255.255.255.0 GigabitEthernet0/0.200 172.16.100.2

  

no ip route vrf RED 0.0.0.0 0.0.0.0 203.0.113.1 global

  

end

write memory

# VRF_Lite_Static_Route_Leaking_Failure_Checks

| Symptom | Likely Cause | Check | Fix |

|---|---|---|---|

| RED cannot reach global shared service | Missing RED static leak | `show ip route vrf RED 192.0.2.0 255.255.255.0` | Add `ip route vrf RED <prefix> <mask> <global-next-hop> global` |

| Static route is configured but not installed | Next hop cannot be resolved | `show ip route` and `show ip cef vrf RED <destination>` | Fix the global route to the next hop or use a valid exit interface and next hop |

| Ping works one way only | Missing return route | `show ip route <RED-prefix>` from the return side | Add the reverse static route in the return routing table |

| VRF ping fails but global ping works | Test used the wrong routing table | `ping <destination>` versus `ping vrf RED <destination>` | Use `ping vrf <VRF> <destination>` |

| Global table cannot return to RED | Global route to RED prefix is missing or points to wrong interface | `show ip route 10.10.10.0 255.255.255.0` | Add or correct the global static return route |

| GREEN reaches service but RED does not | Static leak was configured only under GREEN | `show running-config | include ip route vrf` | Add the RED-specific static leak |

| BLUE unexpectedly reaches the shared service | Leak was configured too broadly or default route leaked unintentionally | `show ip route vrf BLUE` | Remove unintended BLUE static route |

| Route appears in wrong table | Static route was entered without `vrf <VRF>` or with the wrong VRF name | `show ip route` and `show ip route vrf <VRF>` | Remove the wrong route and re-add it under the correct table |

| Next hop resolves in wrong table | Missing `global` keyword on VRF-to-global static route | `show running-config | include ip route vrf RED` | Reconfigure with the `global` keyword |

| Static route works until topology changes | Static leaking has no dynamic failover | `show ip route vrf <VRF>` and `show track` if tracking is used | Use object tracking, floating statics, or BGP leaking for scalable failover |

| Inter-VRF reachability is wider than intended | Static routes leaked entire subnets instead of specific hosts | `show running-config | include ^ip route` | Replace broad static routes with host routes or narrower prefixes |

| Troubleshooting shows no ACL issue but traffic fails | CEF or ARP adjacency missing | `show ip cef vrf <VRF> <destination>` and `show arp` | Fix next-hop reachability, interface state, or L2 adjacency |

##### Source_Basis

# VRF_Lite_Static_Route_Leaking_Mental_Model

# VRF_Lite_Static_Route_Leaking_Configuration_Checklist

# VRF_Lite_Static_Route_Leaking_Skeleton

# VRF_Lite_Static_Route_Leaking_Verification_Commands

# VRF_Lite_Static_Route_Leaking_Rollback

# VRF_Lite_Static_Route_Leaking_Failure_Checks