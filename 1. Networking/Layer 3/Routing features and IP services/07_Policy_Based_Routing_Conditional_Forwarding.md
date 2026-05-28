Policy_Based_Routing_Conditional_Forwarding.md
# Policy_Based_Routing_Conditional_Forwarding

# Policy_Based_Routing_Conditional_Forwarding_Mental_Model

| Concept | Operational Meaning |
|---|---|
| PBR | Policy-Based Routing changes forwarding for selected traffic before normal destination-based routing is used |
| Ingress feature | Interface PBR is applied inbound on the interface where traffic enters the router |
| Route-map dependency | PBR uses a route map to match traffic and set the forwarding action |
| Match logic | ACLs usually identify the source, destination, or protocol traffic that should be policy routed |
| Set logic | `set ip next-hop` or related set actions tell the router where matching traffic should go |
| Normal routing fallback | Traffic that does not match a PBR permit sequence is routed normally |
| Route-map deny in PBR | A deny sequence does not drop the packet by itself; it prevents PBR from applying and falls back to normal routing |
| RIB boundary | PBR does not install routes into the routing table |
| CEF boundary | `show ip cef exact-route` shows normal CEF lookup, not necessarily the final PBR-forwarded path |
| Local PBR | Router-generated traffic needs `ip local policy route-map`; interface PBR affects transit ingress traffic |
| Recursive risk | If the configured next hop is unreachable, PBR can fail or fall back depending on the set action and platform behavior |
| Operational risk | PBR can make traffic ignore the obvious routing table, so always verify interface attachment and route-map counters |

# Policy_Based_Routing_Conditional_Forwarding_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are up | RTR/L3SW | `show ip interface brief` | Ingress, egress, and next-hop interfaces are `up/up` |
| 2 | Confirm Layer 3 forwarding is enabled on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 3 | Confirm normal destination routing before applying PBR | RTR/L3SW | `show ip route <DESTINATION_IP>` | Router has a baseline route to the destination |
| 4 | Confirm PBR next-hop reachability | RTR/L3SW | `ping <PBR_NEXT_HOP>` | PBR next hop responds or is reachable through a valid route |
| 5 | Confirm existing route maps before adding a new one | RTR/L3SW | `show route-map` | Existing policies are reviewed to avoid overwriting a reused name |
| 6 | Confirm existing ACLs before adding a new match object | RTR/L3SW | `show access-lists` | Existing ACLs are reviewed to avoid accidental reuse |
| 7 | Create an ACL to match traffic that should be policy routed | RTR/L3SW | `conf t`<br>`ip access-list extended ACL-PBR-TRAFFIC`<br>` permit ip <SOURCE_SUBNET> <SOURCE_WILDCARD> <DESTINATION_SUBNET> <DESTINATION_WILDCARD>`<br>`end` | ACL identifies only the traffic that should be policy routed |
| 8 | Create the PBR route map | RTR/L3SW | `conf t`<br>`route-map RM-PBR permit 10`<br>` match ip address ACL-PBR-TRAFFIC`<br>` set ip next-hop <PBR_NEXT_HOP>`<br>`end` | Matching traffic is selected and pointed to the configured next hop |
| 9 | Optional: add a second PBR next hop for fallback | RTR/L3SW | `conf t`<br>`route-map RM-PBR permit 10`<br>` set ip next-hop <PBR_NEXT_HOP_1> <PBR_NEXT_HOP_2>`<br>`end` | Router attempts the listed next hops in order where supported |
| 10 | Optional: use default next-hop behavior only when normal routing should be tried first | RTR/L3SW | `conf t`<br>`route-map RM-PBR-DEFAULT permit 10`<br>` match ip address ACL-PBR-TRAFFIC`<br>` set ip default next-hop <DEFAULT_PBR_NEXT_HOP>`<br>`end` | PBR next hop is used only when normal routing has no explicit route |
| 11 | Optional: create a deny sequence for traffic that must bypass PBR | RTR/L3SW | `conf t`<br>`route-map RM-PBR deny 5`<br>` match ip address ACL-PBR-BYPASS`<br>`end` | Matching bypass traffic is not policy routed and uses normal routing |
| 12 | Optional: add a final permit sequence with no set action only if you want counters for unmatched allowed traffic | RTR/L3SW | `conf t`<br>`route-map RM-PBR permit 999`<br>`end` | Remaining traffic matches the sequence but is forwarded normally because no PBR set action exists |
| 13 | Verify route-map structure before attachment | RTR/L3SW | `show route-map RM-PBR` | Route map has the correct sequence order, match ACL, and set next-hop action |
| 14 | Verify ACL match logic before attachment | RTR/L3SW | `show access-lists ACL-PBR-TRAFFIC` | ACL permits only the traffic intended for PBR |
| 15 | Apply PBR inbound on the ingress interface | RTR/L3SW | `conf t`<br>`interface <INGRESS_INTERFACE>`<br>` ip policy route-map RM-PBR`<br>`end` | PBR is attached to the interface where matching traffic enters |
| 16 | Verify interface PBR attachment | RTR/L3SW | `show ip policy` | Ingress interface shows `RM-PBR` as the active policy route map |
| 17 | Verify the running config under the ingress interface | RTR/L3SW | `show running-config interface <INGRESS_INTERFACE>` | Interface contains `ip policy route-map RM-PBR` |
| 18 | Generate matching traffic through the router | Host/RTR | `ping <DESTINATION_IP> source <MATCHING_SOURCE_IP_OR_INTERFACE>` | Traffic matching the ACL is sent through the policy path if downstream routing allows it |
| 19 | Verify route-map match counters | RTR/L3SW | `show route-map RM-PBR` | Match counters increase for traffic hitting the PBR route map |
| 20 | Trace the forwarding path for matching traffic | Host/RTR | `traceroute <DESTINATION_IP> source <MATCHING_SOURCE_IP>` | First routed hop follows the PBR next hop, not necessarily the normal RIB next hop |
| 21 | Generate nonmatching traffic through the router | Host/RTR | `ping <DESTINATION_IP> source <NONMATCHING_SOURCE_IP_OR_INTERFACE>` | Nonmatching traffic follows normal destination-based routing |
| 22 | Trace the forwarding path for nonmatching traffic | Host/RTR | `traceroute <DESTINATION_IP> source <NONMATCHING_SOURCE_IP>` | First routed hop follows the normal routing table path |
| 23 | Optional: apply local PBR for router-generated traffic | RTR/L3SW | `conf t`<br>`ip local policy route-map RM-PBR-LOCAL`<br>`end` | Router-generated traffic matching the local PBR policy is policy routed |
| 24 | Verify local PBR attachment | RTR/L3SW | `show running-config | include ip local policy` | Local PBR route map is visible in global configuration |
| 25 | Confirm normal routing table did not change because of PBR | RTR/L3SW | `show ip route <DESTINATION_IP>` | Routing table remains unchanged; PBR affects forwarding policy, not RIB contents |
| 26 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | PBR policy and interface attachment survive reload |

# Policy_Based_Routing_Conditional_Forwarding_Skeleton

conf t
!
ip routing
!
ip access-list extended ACL-PBR-TRAFFIC
 permit ip <SOURCE_SUBNET> <SOURCE_WILDCARD> <DESTINATION_SUBNET> <DESTINATION_WILDCARD>
!
route-map RM-PBR permit 10
 match ip address ACL-PBR-TRAFFIC
 set ip next-hop <PBR_NEXT_HOP>
!
interface <INGRESS_INTERFACE>
 ip policy route-map RM-PBR
!
end
write memory

# Policy_Based_Routing_Bypass_Skeleton

conf t
!
ip access-list extended ACL-PBR-BYPASS
 permit ip <BYPASS_SOURCE_SUBNET> <BYPASS_SOURCE_WILDCARD> <BYPASS_DESTINATION_SUBNET> <BYPASS_DESTINATION_WILDCARD>
!
ip access-list extended ACL-PBR-TRAFFIC
 permit ip <SOURCE_SUBNET> <SOURCE_WILDCARD> <DESTINATION_SUBNET> <DESTINATION_WILDCARD>
!
route-map RM-PBR deny 5
 match ip address ACL-PBR-BYPASS
!
route-map RM-PBR permit 10
 match ip address ACL-PBR-TRAFFIC
 set ip next-hop <PBR_NEXT_HOP>
!
interface <INGRESS_INTERFACE>
 ip policy route-map RM-PBR
!
end
write memory

# Policy_Based_Routing_Default_Next_Hop_Skeleton

conf t
!
ip access-list extended ACL-PBR-TRAFFIC
 permit ip <SOURCE_SUBNET> <SOURCE_WILDCARD> <DESTINATION_SUBNET> <DESTINATION_WILDCARD>
!
route-map RM-PBR-DEFAULT permit 10
 match ip address ACL-PBR-TRAFFIC
 set ip default next-hop <DEFAULT_PBR_NEXT_HOP>
!
interface <INGRESS_INTERFACE>
 ip policy route-map RM-PBR-DEFAULT
!
end
write memory

# Local_Policy_Based_Routing_Skeleton

conf t
!
ip access-list extended ACL-LOCAL-PBR
 permit ip host <ROUTER_SOURCE_IP> <DESTINATION_SUBNET> <DESTINATION_WILDCARD>
!
route-map RM-PBR-LOCAL permit 10
 match ip address ACL-LOCAL-PBR
 set ip next-hop <LOCAL_PBR_NEXT_HOP>
!
ip local policy route-map RM-PBR-LOCAL
!
end
write memory

# Policy_Based_Routing_Conditional_Forwarding_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config interface <INGRESS_INTERFACE>
show running-config | include ip policy
show running-config | include ip local policy
show ip policy
show route-map
show route-map RM-PBR
show access-lists
show access-lists ACL-PBR-TRAFFIC
show ip route <DESTINATION_IP>
show ip route <PBR_NEXT_HOP>
show ip cef <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
ping <PBR_NEXT_HOP>
ping <DESTINATION_IP> source <MATCHING_SOURCE_IP_OR_INTERFACE>
ping <DESTINATION_IP> source <NONMATCHING_SOURCE_IP_OR_INTERFACE>
traceroute <DESTINATION_IP> source <MATCHING_SOURCE_IP>
traceroute <DESTINATION_IP> source <NONMATCHING_SOURCE_IP>
show interfaces <INGRESS_INTERFACE>
show interfaces <EGRESS_INTERFACE>
debug ip policy

# Policy_Based_Routing_Conditional_Forwarding_Rollback

conf t
!
interface <INGRESS_INTERFACE>
 no ip policy route-map RM-PBR
!
no ip local policy route-map RM-PBR-LOCAL
!
no route-map RM-PBR
no route-map RM-PBR-DEFAULT
no route-map RM-PBR-LOCAL
!
no ip access-list extended ACL-PBR-TRAFFIC
no ip access-list extended ACL-PBR-BYPASS
no ip access-list extended ACL-LOCAL-PBR
!
end
write memory

# Policy_Based_Routing_Conditional_Forwarding_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| PBR has no effect | Route map is not attached to the ingress interface | `show ip policy` | Apply `ip policy route-map <NAME>` under the correct ingress interface |
| Route map counters do not increment | Traffic does not match the ACL or enters through another interface | `show route-map RM-PBR` and `show access-lists ACL-PBR-TRAFFIC` | Correct ACL logic or apply PBR on the true ingress interface |
| Traffic follows normal route instead of PBR path | No PBR permit sequence matched | `show route-map RM-PBR` | Fix match conditions or sequence order |
| Traffic bypasses PBR unexpectedly | A route-map deny sequence matched first | `show route-map RM-PBR` | Move, remove, or correct the deny sequence |
| Traffic is dropped after PBR applies | PBR next hop is unreachable or downstream path is broken | `ping <PBR_NEXT_HOP>` and `traceroute <DESTINATION_IP> source <SOURCE_IP>` | Fix next-hop reachability, downstream routing, or return path |
| Routing table looks unchanged | PBR does not modify the RIB | `show ip route <DESTINATION_IP>` | Verify with `show ip policy`, route-map counters, and traceroute |
| `show ip cef exact-route` disagrees with observed path | CEF exact-route shows normal routing, not final PBR policy result | `show ip policy` and `show route-map RM-PBR` | Use PBR verification commands and traceroute for policy-forwarded traffic |
| Router-generated ping is not policy routed | Interface PBR only affects transit ingress traffic | `show running-config | include ip local policy` | Configure `ip local policy route-map <NAME>` for local traffic |
| PBR only works from some hosts | ACL source match is too narrow or wrong wildcard mask is used | `show access-lists ACL-PBR-TRAFFIC` | Correct ACL source subnet and wildcard |
| PBR affects too much traffic | ACL match is too broad | `show access-lists ACL-PBR-TRAFFIC` | Narrow the ACL or add earlier bypass deny sequence |
| PBR route map is configured but unsupported set action is ignored | Set action is invalid for PBR or platform behavior differs | `route-map RM-PBR permit 10` then `set ?` | Use supported PBR set actions for the platform |
| Backup next hop is never used | Multiple next-hop behavior is unsupported or first next hop still resolves | `show route-map RM-PBR` and `show ip route <NEXT_HOP>` | Use IP SLA tracking design if true reachability-based failover is required |
| `set ip default next-hop` does not change forwarding | A normal route already exists for the destination | `show ip route <DESTINATION_IP>` | Use `set ip next-hop` if policy should override existing routes |
| Normal traffic breaks after applying PBR | PBR applied to wrong ingress interface or ACL is too broad | `show ip policy` and `show access-lists` | Remove PBR from wrong interface or correct ACL |
| Return traffic fails | Forward path is policy routed but reverse path is missing or asymmetric | Traceroute from both directions | Add reverse routes or adjust downstream routing |
| PBR works before reload but disappears afterward | Configuration was not saved | `show startup-config | include ip policy` | Reconfigure and save with `copy running-config startup-config` |
| CPU impact during troubleshooting | `debug ip policy` left running during traffic | `show debugging` | Use debug briefly in lab only, then run `undebug all` |
| PBR appears to ignore interface shutdown | Set next hop is still recursively reachable through another path | `show ip route <PBR_NEXT_HOP>` | Use tracking or change next-hop recursion behavior |
| ACL hit counters increase but route-map counters do not | ACL is used elsewhere or traffic is not hitting the PBR route map | `show route-map RM-PBR` and `show ip policy` | Confirm attachment and traffic ingress point |
| Policy path selected but first hop is not expected | Next hop recursion points out a different interface than assumed | `show ip route <PBR_NEXT_HOP>` | Use a directly connected next hop or fix recursion |

# Index

Policy_Based_Routing_Conditional_Forwarding.md
Policy_Based_Routing_Conditional_Forwarding
Policy_Based_Routing_Conditional_Forwarding_Mental_Model
Policy_Based_Routing_Conditional_Forwarding_Configuration_Checklist
Policy_Based_Routing_Conditional_Forwarding_Skeleton
Policy_Based_Routing_Bypass_Skeleton
Policy_Based_Routing_Default_Next_Hop_Skeleton
Local_Policy_Based_Routing_Skeleton
Policy_Based_Routing_Conditional_Forwarding_Verification_Commands
Policy_Based_Routing_Conditional_Forwarding_Rollback
Policy_Based_Routing_Conditional_Forwarding_Failure_Checks