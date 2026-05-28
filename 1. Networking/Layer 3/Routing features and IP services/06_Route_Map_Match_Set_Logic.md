Route_Map_Match_Set_Logic.md
# Route_Map_Match_Set_Logic

# Route_Map_Match_Set_Logic_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Route map | Ordered policy logic made of sequence numbers |
| Sequence number | Lowest sequence is evaluated first; higher sequences are evaluated later |
| Permit sequence | Matching item is allowed to continue through the feature and optional `set` actions are applied |
| Deny sequence | Matching item is rejected by the feature context using the route map |
| Match condition | Selects what the route-map sequence applies to |
| Set action | Changes an attribute or forwarding behavior after the sequence matches |
| No match statement | A route-map sequence with no `match` statement matches everything that reaches that sequence |
| Same match type with multiple objects | Uses OR logic; only one listed object must match |
| Different match types | Uses AND logic; all different match conditions must match |
| First-match behavior | Processing stops at the first matching sequence unless `continue` is configured |
| Implicit deny | Anything that reaches the end without matching a permit sequence is denied by the route map |
| Match object deny line | A `deny` inside an ACL or prefix list means the object does not match that sequence; it is not the same thing as a route-map deny |
| Feature attachment | A route map does nothing until attached to a feature such as BGP, redistribution, distribute-list, PBR, or NAT |

# Route_Map_Match_Set_Logic_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Define the policy purpose before configuring syntax | RTR/L3SW | `! Example purpose: match RFC1918 prefixes and set a route tag` | Route map has a clear feature context and expected action |
| 2 | Confirm the current route-map state | RTR/L3SW | `show route-map` | Existing route maps, sequence numbers, match clauses, and set clauses are visible |
| 3 | Confirm existing ACLs that may be reused | RTR/L3SW | `show access-lists` | Existing ACLs are reviewed before reuse |
| 4 | Confirm existing prefix lists that may be reused | RTR/L3SW | `show ip prefix-list` | Existing prefix lists are reviewed before reuse |
| 5 | Create a prefix list for route-prefix matching | RTR/L3SW | `conf t`<br>`ip prefix-list PL-RFC1918 seq 10 permit 10.0.0.0/8 le 32`<br>`ip prefix-list PL-RFC1918 seq 20 permit 172.16.0.0/12 le 32`<br>`ip prefix-list PL-RFC1918 seq 30 permit 192.168.0.0/16 le 32`<br>`end` | Prefix list matches RFC1918 routes with prefix lengths up to /32 |
| 6 | Create an ACL for packet or address matching if the feature requires ACL logic | RTR/L3SW | `conf t`<br>`ip access-list extended ACL-POLICY-TRAFFIC`<br>` permit ip 10.10.10.0 0.0.0.255 10.20.20.0 0.0.0.255`<br>`end` | ACL matches the intended traffic or address pattern |
| 7 | Create the first route-map sequence | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET permit 10`<br>`end` | Route map sequence 10 exists |
| 8 | Add a prefix-list match to the route-map sequence | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET permit 10`<br>` match ip address prefix-list PL-RFC1918`<br>`end` | Sequence 10 matches prefixes permitted by `PL-RFC1918` |
| 9 | Add a set action for route tagging | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET permit 10`<br>` set tag 100`<br>`end` | Matching routes are permitted and tagged with tag `100` in feature contexts that support route tags |
| 10 | Add a second route-map sequence for everything else that should be permitted | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET permit 20`<br>`end` | Sequence 20 matches all remaining items because it has no `match` statement |
| 11 | Optional: add a set action to the catch-all permit sequence | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET permit 20`<br>` set metric 200`<br>`end` | Remaining permitted routes receive metric `200` in feature contexts that support metric setting |
| 12 | Verify route-map sequence order | RTR/L3SW | `show route-map RM-MATCH-SET` | Sequence 10 appears before sequence 20 |
| 13 | Verify route-map match objects | RTR/L3SW | `show route-map RM-MATCH-SET` | Match clause references the intended ACL, prefix list, tag, metric, or community object |
| 14 | Verify route-map set actions | RTR/L3SW | `show route-map RM-MATCH-SET` | Set clauses show the intended tag, metric, next hop, community, local preference, weight, or other supported action |
| 15 | Validate prefix-list logic before attachment | RTR/L3SW | `show ip prefix-list PL-RFC1918` | Prefix list order and permit/deny logic are correct |
| 16 | Validate ACL logic before attachment | RTR/L3SW | `show access-lists ACL-POLICY-TRAFFIC` | ACL matches only intended traffic or addresses |
| 17 | Optional: configure same-type OR matching | RTR/L3SW | `conf t`<br>`route-map RM-OR-EXAMPLE permit 10`<br>` match ip address prefix-list PL-RFC1918 PL-SERVERS`<br>` set tag 110`<br>`end` | Sequence matches if either `PL-RFC1918` or `PL-SERVERS` matches |
| 18 | Optional: configure different-type AND matching | RTR/L3SW | `conf t`<br>`route-map RM-AND-EXAMPLE permit 10`<br>` match ip address prefix-list PL-RFC1918`<br>` match tag 50`<br>` set tag 150`<br>`end` | Sequence matches only when both prefix-list and tag conditions match |
| 19 | Optional: configure an explicit deny sequence | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET deny 5`<br>` match ip address prefix-list PL-BLOCKED`<br>`end` | Items matching `PL-BLOCKED` are denied before later permit sequences are evaluated |
| 20 | Optional: configure an explicit final permit sequence when the feature should allow unmatched routes | RTR/L3SW | `conf t`<br>`route-map RM-MATCH-SET permit 999`<br>`end` | Unmatched items are permitted instead of being dropped by implicit deny |
| 21 | Attach the route map to the intended feature only after verification | RTR/L3SW | `! Example attachment points below` | Route map begins affecting traffic or routes only after feature attachment |
| 22 | Example: attach to redistribution | RTR/L3SW | `router ospf 10`<br>` redistribute connected subnets route-map RM-MATCH-SET` | Connected routes redistributed into OSPF are filtered or modified by the route map |
| 23 | Example: attach to BGP neighbor inbound policy | RTR/L3SW | `router bgp <ASN>`<br>` address-family ipv4 unicast`<br>`  neighbor <PEER_IP> route-map RM-MATCH-SET in` | Received BGP routes from the neighbor are filtered or modified by the route map |
| 24 | Example: attach to PBR on an ingress interface | RTR/L3SW | `interface <INGRESS_INTERFACE>`<br>` ip policy route-map RM-PBR` | Matching ingress packets are policy routed according to the route map |
| 25 | Verify attachment under the feature | RTR/L3SW | `show running-config | section route-map`<br>`show running-config | include route-map` | Route map is present and attached to the intended feature |
| 26 | Verify operational effect in the feature context | RTR/L3SW | `show ip route`<br>`show ip bgp`<br>`show ip policy` | Routing, BGP, redistribution, or PBR behavior reflects the route-map policy |
| 27 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | Route-map logic and feature attachment survive reload |

# Route_Map_Match_Set_Logic_Skeleton

conf t
!
ip prefix-list PL-RFC1918 seq 10 permit 10.0.0.0/8 le 32
ip prefix-list PL-RFC1918 seq 20 permit 172.16.0.0/12 le 32
ip prefix-list PL-RFC1918 seq 30 permit 192.168.0.0/16 le 32
!
ip prefix-list PL-BLOCKED seq 10 permit 192.0.2.0/24
!
route-map RM-MATCH-SET deny 5
 match ip address prefix-list PL-BLOCKED
!
route-map RM-MATCH-SET permit 10
 match ip address prefix-list PL-RFC1918
 set tag 100
!
route-map RM-MATCH-SET permit 20
 set metric 200
!
route-map RM-MATCH-SET permit 999
!
end
write memory

# Route_Map_OR_Logic_Skeleton

conf t
!
ip prefix-list PL-RFC1918 seq 10 permit 10.0.0.0/8 le 32
ip prefix-list PL-SERVERS seq 10 permit 10.50.0.0/16 le 32
!
route-map RM-OR-EXAMPLE permit 10
 match ip address prefix-list PL-RFC1918 PL-SERVERS
 set tag 110
!
end
write memory

# Route_Map_AND_Logic_Skeleton

conf t
!
ip prefix-list PL-RFC1918 seq 10 permit 10.0.0.0/8 le 32
!
route-map RM-AND-EXAMPLE permit 10
 match ip address prefix-list PL-RFC1918
 match tag 50
 set tag 150
!
end
write memory

# Route_Map_Attachment_Examples

conf t
!
! Redistribution attachment example.
router ospf 10
 redistribute connected subnets route-map RM-MATCH-SET
!
! BGP inbound policy attachment example.
router bgp <ASN>
 address-family ipv4 unicast
  neighbor <PEER_IP> route-map RM-MATCH-SET in
 exit-address-family
!
! PBR attachment example.
interface <INGRESS_INTERFACE>
 ip policy route-map RM-PBR
!
! Local PBR attachment example.
ip local policy route-map RM-LOCAL-PBR
!
end
write memory

# Route_Map_Match_Set_Logic_Verification_Commands

show route-map
show route-map RM-MATCH-SET
show running-config | section route-map
show running-config | include route-map
show ip prefix-list
show ip prefix-list PL-RFC1918
show access-lists
show access-lists ACL-POLICY-TRAFFIC
show ip route
show ip route tag 100
show ip protocols
show ip bgp
show ip bgp neighbors <PEER_IP> received-routes
show ip bgp neighbors <PEER_IP> advertised-routes
show ip policy
show running-config interface <INGRESS_INTERFACE>
show running-config | section router ospf
show running-config | section router bgp
show running-config | include redistribute

# Route_Map_Match_Set_Logic_Rollback

conf t
!
! Remove feature attachments first.
router ospf 10
 no redistribute connected subnets route-map RM-MATCH-SET
!
router bgp <ASN>
 address-family ipv4 unicast
  no neighbor <PEER_IP> route-map RM-MATCH-SET in
 exit-address-family
!
interface <INGRESS_INTERFACE>
 no ip policy route-map RM-PBR
!
no ip local policy route-map RM-LOCAL-PBR
!
! Remove route maps after detaching them.
no route-map RM-MATCH-SET
no route-map RM-OR-EXAMPLE
no route-map RM-AND-EXAMPLE
!
! Remove match objects only if they are not used elsewhere.
no ip prefix-list PL-RFC1918
no ip prefix-list PL-BLOCKED
no ip prefix-list PL-SERVERS
no ip access-list extended ACL-POLICY-TRAFFIC
!
end
write memory

# Route_Map_Match_Set_Logic_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Route map has no effect | Route map is configured but not attached to any feature | `show running-config | include route-map` | Attach it under the intended feature |
| Routes or traffic are unexpectedly denied | Missing final permit sequence and implicit deny is active | `show route-map <NAME>` | Add an explicit permit sequence for allowed unmatched items |
| A deny in a prefix list does not deny the route-map sequence as expected | Match object deny means the item does not match that sequence | `show ip prefix-list <NAME>` and `show route-map <NAME>` | Put deny behavior in the route-map sequence if that is the intended policy |
| Catch-all sequence matches too much | Route-map sequence has no `match` statement | `show route-map <NAME>` | Add a specific match condition or move the catch-all later |
| Route map sequence is skipped | Match object does not permit the route or packet | `show ip prefix-list`, `show access-lists`, `show route-map` | Correct ACL, prefix list, community list, tag, or metric match logic |
| Same-type match behaves broader than expected | Multiple objects on one match line use OR logic | `show route-map <NAME>` | Split logic into separate sequences or use more specific match objects |
| Different match statements are too restrictive | Different match types use AND logic | `show route-map <NAME>` | Remove unnecessary match lines or split policy into multiple sequences |
| Later sequence never takes effect | First matching sequence stops processing | `show route-map <NAME>` | Reorder sequences or use `continue` only when deliberately needed |
| Route map denies everything | No sequence matches and implicit deny catches the rest | `show route-map <NAME>` | Add a final `route-map <NAME> permit <SEQ>` with no match clause if pass-through is required |
| Set action is visible but operational behavior does not change | Feature context does not support that set action | Feature-specific verification command | Use a supported `set` action for that feature |
| BGP route-map change does not show immediately | BGP policy needs route refresh or session update | `show ip bgp neighbors <PEER_IP>` | Use route refresh or clear soft in/out as appropriate |
| Redistribution policy does not tag or filter as expected | Route-map attached to wrong protocol or wrong redistribution statement | `show running-config | include redistribute` | Attach the route map to the correct redistribution source and destination |
| PBR route-map does not affect router-generated traffic | Interface PBR affects transit ingress traffic, not local router traffic | `show ip policy` | Use `ip local policy route-map <NAME>` for locally generated traffic |
| PBR route-map changes forwarding but routing table looks unchanged | PBR does not modify the RIB | `show ip route` and `show ip policy` | Verify PBR with policy commands, traceroute, and packet path testing |
| Route-map sequence edits create wrong order | Sequence numbers were inserted incorrectly | `show route-map <NAME>` | Use spaced sequence numbers and insert policy in the correct order |
| Removing an ACL breaks another feature | Match object was reused elsewhere | `show running-config | include <ACL_OR_PREFIX_NAME>` | Detach or replace all references before deleting shared objects |
| Configuration rollback leaves policy active | Route map was deleted but feature attachment remains invalid or stale | `show running-config | include route-map` | Remove the feature attachment first, then remove route-map objects |
| Metrics look wrong after route-map set | Wrong metric format for the routing protocol | `show ip protocols` and protocol route output | Use the metric format required by the destination protocol |
| Route tag is not visible | Feature does not carry or display tags in that context | `show ip route tag <TAG>` | Verify tag support and apply route map where route tags are supported |
| Prefix list matches wrong prefix lengths | Incorrect `ge` or `le` values | `show ip prefix-list <NAME>` | Correct high-order bit pattern and prefix-length range |

# Index

Route_Map_Match_Set_Logic.md
Route_Map_Match_Set_Logic
Route_Map_Match_Set_Logic_Mental_Model
Route_Map_Match_Set_Logic_Configuration_Checklist
Route_Map_Match_Set_Logic_Skeleton
Route_Map_OR_Logic_Skeleton
Route_Map_AND_Logic_Skeleton
Route_Map_Attachment_Examples
Route_Map_Match_Set_Logic_Verification_Commands
Route_Map_Match_Set_Logic_Rollback
Route_Map_Match_Set_Logic_Failure_Checks