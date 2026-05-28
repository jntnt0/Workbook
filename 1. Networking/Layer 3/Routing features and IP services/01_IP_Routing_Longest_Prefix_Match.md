IP_Routing_Longest_Prefix_Match.md
# IP_Routing_Longest_Prefix_Match

# IP_Routing_Longest_Prefix_Match_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Longest prefix match | The router forwards traffic using the most specific installed route that matches the destination IP address |
| Prefix length priority | A /28 beats a /26, a /26 beats a /24, and any specific route beats the default route when the destination matches |
| RIB before forwarding | Routes must first be installed in the routing table before the forwarding plane can use them |
| CEF forwarding | CEF programs the forwarding table from the routing table and performs the actual packet forwarding lookup |
| AD and metric boundary | Administrative distance and metric choose between routes to the same prefix length; longest prefix match chooses between different prefix lengths |
| Default route | `0.0.0.0/0` is the least specific route and is used only when no more specific route matches |
| Static route specificity | Overlapping static routes can intentionally steer different parts of a larger network toward different next hops |
| Troubleshooting mistake | Do not assume the route with the lowest AD wins if a more specific installed prefix also matches the destination |

# IP_Routing_Longest_Prefix_Match_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Layer 3 forwarding is enabled if using a multilayer switch | L3SW/RTR | `show running-config | include ^ip routing` | `ip routing` is present on a multilayer switch; routers normally route by default |
| 2 | Confirm routed interfaces or SVIs are up | L3SW/RTR | `show ip interface brief` | Transit and local interfaces are `up/up` with correct IPv4 addresses |
| 3 | Confirm next-hop reachability before adding recursive static routes | L3SW/RTR | `ping <NEXT_HOP_IP>` | Router can reach the next-hop IP address |
| 4 | Add the broad fallback route | L3SW/RTR | `ip route 10.0.3.0 255.255.255.0 10.3.3.3` | `10.0.3.0/24` is installed as the least specific matching route for that block |
| 5 | Add a more specific route for part of the block | L3SW/RTR | `ip route 10.0.3.0 255.255.255.192 10.2.2.2` | `10.0.3.0/26` is installed and preferred for destinations inside `10.0.3.0-10.0.3.63` |
| 6 | Add the most specific route for the smallest part of the block | L3SW/RTR | `ip route 10.0.3.0 255.255.255.240 10.1.1.1` | `10.0.3.0/28` is installed and preferred for destinations inside `10.0.3.0-10.0.3.15` |
| 7 | Verify all overlapping prefixes are present | L3SW/RTR | `show ip route 10.0.3.0` | The routing table shows matching static routes for the overlapping prefixes |
| 8 | Verify the /28 destination follows the most specific route | L3SW/RTR | `show ip route 10.0.3.14` | Best match resolves through the `10.0.3.0/28` route |
| 9 | Verify the /26 destination follows the next most specific route | L3SW/RTR | `show ip route 10.0.3.42` | Best match resolves through the `10.0.3.0/26` route |
| 10 | Verify the /24-only destination follows the broad route | L3SW/RTR | `show ip route 10.0.3.100` | Best match resolves through the `10.0.3.0/24` route |
| 11 | Verify CEF agrees with the routing decision | L3SW/RTR | `show ip cef 10.0.3.14` | CEF entry points toward the next hop for the /28 route |
| 12 | Verify CEF for the /26 test destination | L3SW/RTR | `show ip cef 10.0.3.42` | CEF entry points toward the next hop for the /26 route |
| 13 | Verify CEF for the /24-only test destination | L3SW/RTR | `show ip cef 10.0.3.100` | CEF entry points toward the next hop for the /24 route |
| 14 | Test actual forwarding to each route bucket | L3SW/RTR | `ping 10.0.3.14` | Traffic toward the /28 destination succeeds if the downstream path and return route exist |
| 15 | Trace forwarding path for each bucket | L3SW/RTR | `traceroute 10.0.3.14` | First hop matches the expected next hop for the most specific route |
| 16 | Save the working configuration | L3SW/RTR | `copy running-config startup-config` | Longest-prefix route test survives reload |

# IP_Routing_Longest_Prefix_Match_Skeleton

conf t
!
! Required on multilayer switches. Usually not needed on routers.
ip routing
!
! Example routed/transit interfaces.
interface GigabitEthernet0/0
 description TRANSIT_TO_NH_10.1.1.1
 ip address 10.1.1.254 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description TRANSIT_TO_NH_10.2.2.2
 ip address 10.2.2.254 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/2
 description TRANSIT_TO_NH_10.3.3.3
 ip address 10.3.3.254 255.255.255.0
 no shutdown
!
! Broad route.
ip route 10.0.3.0 255.255.255.0 10.3.3.3
!
! More specific route.
ip route 10.0.3.0 255.255.255.192 10.2.2.2
!
! Most specific route.
ip route 10.0.3.0 255.255.255.240 10.1.1.1
!
end
write memory

# IP_Routing_Longest_Prefix_Match_Verification_Commands

show running-config | include ^ip routing
show ip interface brief
show running-config | include ^ip route
show ip route
show ip route 10.0.3.0
show ip route 10.0.3.14
show ip route 10.0.3.42
show ip route 10.0.3.100
show ip cef 10.0.3.14
show ip cef 10.0.3.42
show ip cef 10.0.3.100
ping 10.0.3.14
ping 10.0.3.42
ping 10.0.3.100
traceroute 10.0.3.14
traceroute 10.0.3.42
traceroute 10.0.3.100

# IP_Routing_Longest_Prefix_Match_Rollback

conf t
no ip route 10.0.3.0 255.255.255.240 10.1.1.1
no ip route 10.0.3.0 255.255.255.192 10.2.2.2
no ip route 10.0.3.0 255.255.255.0 10.3.3.3
end
write memory

# IP_Routing_Longest_Prefix_Match_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| More specific route does not appear in the routing table | Recursive next hop is unreachable | `show ip route <NEXT_HOP_IP>` | Add or repair the route to the next hop |
| Traffic follows the /24 instead of the /28 | The /28 route is missing from the RIB | `show ip route 10.0.3.14` | Add the /28 route or fix next-hop reachability |
| Route exists but forwarding does not match expected path | CEF entry is stale or unexpected | `show ip cef <DESTINATION_IP>` | Recheck route installation, adjacency, and next-hop resolution |
| Same prefix chooses unexpected path | AD or metric selected another route to the same prefix | `show ip route <PREFIX>` | Adjust AD, metric, or remove competing route |
| Longest prefix logic appears ignored | PBR is overriding destination routing | `show route-map` and `show ip policy` | Remove or correct policy-based routing on the ingress interface |
| Ping fails but route lookup is correct | Return path is missing downstream | Test from both ends and check downstream `show ip route` | Add reverse route or fix downstream routing |
| Traceroute stops at first hop | ACL, firewall, or ICMP filtering blocks probes | `show access-lists` and path device logs | Permit required ICMP/UDP traceroute behavior for testing |
| Static route using only exit Ethernet interface causes ARP noise | Directly attached static route used on multiaccess Ethernet | `show running-config | include ^ip route` | Use next-hop or fully specified static route |
| Multilayer switch does not route between interfaces | `ip routing` missing | `show running-config | include ^ip routing` | Configure `ip routing` |
| Default route wins unexpectedly | Specific route is absent, unresolved, or removed | `show ip route <DESTINATION_IP>` | Restore the missing specific route |

# Index

IP_Routing_Longest_Prefix_Match.md
IP_Routing_Longest_Prefix_Match
IP_Routing_Longest_Prefix_Match_Mental_Model
IP_Routing_Longest_Prefix_Match_Configuration_Checklist
IP_Routing_Longest_Prefix_Match_Skeleton
IP_Routing_Longest_Prefix_Match_Verification_Commands
IP_Routing_Longest_Prefix_Match_Rollback
IP_Routing_Longest_Prefix_Match_Failure_Checks
