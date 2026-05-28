

Cisco’s IOS XE NAT guide gives the route-map NAT forms for static NAT with route-map, pool-based route-map NAT with optional reversible, and verification with show ip nat translations [verbose]. It also notes that route-map based NAT can have platform behavior differences, so the lab checklist includes an image/platform support check.  

NAT_Route_Map_Policy_NAT.md

NAT_Route_Map_Policy_NAT

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `iosxe_combined_pdfs_.md` NAT guide reference | Points IOS XE NAT configuration content to the IOS XE combined source |
| `iosxe_combined_pdfs_.md` | `NAT Configuration Guide` > `Configuring NAT for IP Address Conservation` | Supports classic NAT inside/outside roles, NAT ACLs, NAT pools, PAT, and NAT verification |
| `iosxe_combined_pdfs_.md` | Cisco 1000 Series NAT example | Shows `ip nat inside source route-map <name> overload` style usage in an IOS XE configuration example |
| Cisco IOS XE 17.x IP Addressing Configuration Guide | `Enabling NAT Route Maps Outside-to-Inside Support` | Confirms `ip nat inside source route-map <name> pool <pool-name> [reversible]` syntax |
| Cisco IOS XE 17.x IP Addressing Configuration Guide | Static NAT route-map syntax | Confirms `ip nat inside source static <local-ip> <global-ip> route-map <map-name>` |
| Related lab | `nat-policy-route-maps-final` | Lab target for route-map selected NAT |
| Related lab | `nat-policy-route-maps-tXAHEA-final` | Second lab target for route-map selected NAT |
# NAT_Route_Map_Policy_NAT_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Policy NAT | NAT rule selection is controlled by a route map instead of only a simple NAT ACL. |
| NAT route map | The route map decides which traffic qualifies for a specific NAT rule. It can match an ACL and, when supported, the selected outgoing interface. |
| NAT ACL | The ACL inside the route map identifies the packet flow to translate. Extended ACLs are useful because they can match source and destination. |
| PBR versus NAT route map | PBR changes forwarding. NAT route maps select translation behavior. They both use route maps, but they are not the same feature. |
| Multi-exit NAT | Route-map NAT is commonly used when different traffic should translate through different ISP links, pools, or public addresses. |
| Pool-based policy NAT | `ip nat inside source route-map <MAP> pool <POOL>` translates matching traffic into the selected pool. |
| Interface PAT policy NAT | `ip nat inside source route-map <MAP> interface <OUTSIDE_INTERFACE> overload` translates matching traffic to the selected interface address using PAT. |
| Static policy NAT | `ip nat inside source static <LOCAL_IP> <GLOBAL_IP> route-map <MAP>` makes a static mapping conditional on route-map policy. |
| Reversible | The `reversible` keyword allows outside-to-inside initiated sessions to use route-map NAT for destination-based NAT cases. |
| Order matters | NAT is evaluated against configured NAT rules. More specific static or policy rules should be checked before generic overload behavior during troubleshooting. |
| Routing still matters | Route-map NAT does not replace the route table. The router still needs a valid route toward the destination and return path. |
| Platform caveat | Some IOS XE switching platforms or images do not support every route-map NAT form. Confirm support with context help before assuming the syntax exists. |
# NAT_Route_Map_Policy_NAT_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the NAT router interfaces are up | NAT-RTR | `show ip interface brief` | Inside and outside interfaces show up/up |
| 2 | Confirm current NAT configuration before edits | NAT-RTR | `show running-config | include ip nat` | Existing NAT rules are visible before adding policy NAT |
| 3 | Confirm inside and outside NAT interface roles | NAT-RTR | `show running-config | section interface` | Inside interface has `ip nat inside`; outside interface has `ip nat outside` |
| 4 | Confirm route-map NAT syntax support on the image | NAT-RTR | `configure terminal` then `ip nat inside source route-map ?` | Device shows supported route-map NAT options |
| 5 | Create an extended ACL for policy NAT traffic toward path A | NAT-RTR | `ip access-list extended <NAT_ACL_A>` | Device enters extended ACL configuration mode |
| 6 | Match the source and destination flow for path A | NAT-RTR | `permit ip <INSIDE_SUBNET> <INSIDE_WILDCARD> <DESTINATION_A> <DESTINATION_A_WILDCARD>` | ACL identifies traffic that should use NAT policy A |
| 7 | Exit ACL configuration | NAT-RTR | `exit` | Device returns to global configuration mode |
| 8 | Create route map sequence for NAT policy A | NAT-RTR | `route-map <NAT_MAP_A> permit 10` | Device enters route-map configuration mode |
| 9 | Attach the ACL match to NAT policy A | NAT-RTR | `match ip address <NAT_ACL_A>` | Route map matches traffic selected by the ACL |
| 10 | Optional: match the outgoing interface for path A | NAT-RTR | `match interface <OUTSIDE_INTERFACE_A>` | Route map also requires traffic to egress the selected interface |
| 11 | Exit route-map configuration | NAT-RTR | `exit` | Device returns to global configuration mode |
| 12 | Optional: create a NAT pool for path A | NAT-RTR | `ip nat pool <POOL_A> <POOL_A_START_IP> <POOL_A_END_IP> netmask <POOL_A_MASK>` | Pool A is available for selected translations |
| 13 | Configure pool-based policy NAT for path A | NAT-RTR | `ip nat inside source route-map <NAT_MAP_A> pool <POOL_A> overload` | Matching traffic translates to Pool A using PAT |
| 14 | Optional: configure interface PAT policy NAT for path A instead of pool NAT | NAT-RTR | `ip nat inside source route-map <NAT_MAP_A> interface <OUTSIDE_INTERFACE_A> overload` | Matching traffic translates to the outside interface address |
| 15 | Optional: configure reversible policy NAT if outside-to-inside initiation is required | NAT-RTR | `ip nat inside source route-map <NAT_MAP_A> pool <POOL_A> reversible` | Outside-to-inside initiated sessions can use the route-map NAT rule |
| 16 | Optional: create an extended ACL for policy NAT traffic toward path B | NAT-RTR | `ip access-list extended <NAT_ACL_B>` | Device enters ACL configuration for the second policy |
| 17 | Match the source and destination flow for path B | NAT-RTR | `permit ip <INSIDE_SUBNET> <INSIDE_WILDCARD> <DESTINATION_B> <DESTINATION_B_WILDCARD>` | ACL identifies traffic that should use NAT policy B |
| 18 | Create route map sequence for NAT policy B | NAT-RTR | `route-map <NAT_MAP_B> permit 10` | Device enters route-map configuration mode |
| 19 | Attach ACL and optional interface match to NAT policy B | NAT-RTR | `match ip address <NAT_ACL_B>` then `match interface <OUTSIDE_INTERFACE_B>` | Route map selects traffic for path B |
| 20 | Optional: create NAT pool for path B | NAT-RTR | `ip nat pool <POOL_B> <POOL_B_START_IP> <POOL_B_END_IP> netmask <POOL_B_MASK>` | Pool B is available for selected translations |
| 21 | Configure policy NAT for path B | NAT-RTR | `ip nat inside source route-map <NAT_MAP_B> pool <POOL_B> overload` | Matching traffic translates to Pool B |
| 22 | Optional: configure static policy NAT | NAT-RTR | `ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP> route-map <STATIC_NAT_MAP>` | Static NAT is conditional on the route-map match |
| 23 | Confirm route maps are installed | NAT-RTR | `show route-map` | Route-map names, sequence numbers, and match statements are visible |
| 24 | Confirm ACLs are installed | NAT-RTR | `show access-lists` | NAT policy ACLs are visible and ready to match traffic |
| 25 | Confirm NAT rules are installed | NAT-RTR | `show running-config | include ip nat inside source` | Route-map NAT rules are present |
| 26 | Generate traffic matching policy A | Inside Host | `ping <DESTINATION_A_TEST_IP>` | Traffic triggers policy A translation |
| 27 | Verify policy A ACL hit count | NAT-RTR | `show access-lists <NAT_ACL_A>` | ACL hit count increments for policy A traffic |
| 28 | Verify policy A translation | NAT-RTR | `show ip nat translations` | Inside local address translates using Pool A or outside interface A |
| 29 | Generate traffic matching policy B if configured | Inside Host | `ping <DESTINATION_B_TEST_IP>` | Traffic triggers policy B translation |
| 30 | Verify policy B ACL hit count | NAT-RTR | `show access-lists <NAT_ACL_B>` | ACL hit count increments for policy B traffic |
| 31 | Verify detailed NAT state | NAT-RTR | `show ip nat translations verbose` | Translation details show the expected local and global address mapping |
| 32 | Confirm NAT statistics | NAT-RTR | `show ip nat statistics` | NAT counters, inside/outside interfaces, and pool usage are visible |
| 33 | Save the working configuration | NAT-RTR | `write memory` | Policy NAT configuration survives reload |
# NAT_Route_Map_Policy_NAT_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 description INSIDE_LAN
 ip address <INSIDE_INTERFACE_IP> <INSIDE_MASK>
 ip nat inside
 no shutdown
!
interface <OUTSIDE_INTERFACE_A>
 description OUTSIDE_PATH_A
 ip address <OUTSIDE_A_IP> <OUTSIDE_A_MASK>
 ip nat outside
 no shutdown
!
interface <OUTSIDE_INTERFACE_B>
 description OUTSIDE_PATH_B
 ip address <OUTSIDE_B_IP> <OUTSIDE_B_MASK>
 ip nat outside
 no shutdown
!
ip access-list extended <NAT_ACL_A>
 permit ip <INSIDE_SUBNET> <INSIDE_WILDCARD> <DESTINATION_A> <DESTINATION_A_WILDCARD>
exit
!
route-map <NAT_MAP_A> permit 10
 match ip address <NAT_ACL_A>
 match interface <OUTSIDE_INTERFACE_A>
exit
!
ip nat pool <POOL_A> <POOL_A_START_IP> <POOL_A_END_IP> netmask <POOL_A_MASK>
ip nat inside source route-map <NAT_MAP_A> pool <POOL_A> overload
!
ip access-list extended <NAT_ACL_B>
 permit ip <INSIDE_SUBNET> <INSIDE_WILDCARD> <DESTINATION_B> <DESTINATION_B_WILDCARD>
exit
!
route-map <NAT_MAP_B> permit 10
 match ip address <NAT_ACL_B>
 match interface <OUTSIDE_INTERFACE_B>
exit
!
ip nat pool <POOL_B> <POOL_B_START_IP> <POOL_B_END_IP> netmask <POOL_B_MASK>
ip nat inside source route-map <NAT_MAP_B> pool <POOL_B> overload
!
ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP>
!
end
write memory
# NAT_Route_Map_Policy_NAT_Interface_PAT_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 ip nat inside
!
interface <OUTSIDE_INTERFACE_A>
 ip nat outside
!
ip access-list extended <NAT_ACL_A>
 permit ip <INSIDE_SUBNET> <INSIDE_WILDCARD> <DESTINATION_A> <DESTINATION_A_WILDCARD>
exit
!
route-map <NAT_MAP_A> permit 10
 match ip address <NAT_ACL_A>
 match interface <OUTSIDE_INTERFACE_A>
exit
!
ip nat inside source route-map <NAT_MAP_A> interface <OUTSIDE_INTERFACE_A> overload
!
end
write memory
# NAT_Route_Map_Policy_NAT_Reversible_Pool_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 ip nat inside
!
interface <OUTSIDE_INTERFACE_A>
 ip nat outside
!
ip nat pool <POOL_A> <POOL_A_START_IP> <POOL_A_END_IP> netmask <POOL_A_MASK>
!
ip access-list extended <NAT_ACL_A>
 permit ip <INSIDE_SUBNET> <INSIDE_WILDCARD> <DESTINATION_A> <DESTINATION_A_WILDCARD>
exit
!
route-map <NAT_MAP_A> permit 10
 match ip address <NAT_ACL_A>
exit
!
ip nat inside source route-map <NAT_MAP_A> pool <POOL_A> reversible
!
end
write memory
# NAT_Route_Map_Policy_NAT_Static_Route_Map_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 ip nat inside
!
interface <OUTSIDE_INTERFACE>
 ip nat outside
!
ip access-list extended <STATIC_NAT_ACL>
 permit ip host <INSIDE_LOCAL_IP> <DESTINATION_MATCH> <DESTINATION_WILDCARD>
exit
!
route-map <STATIC_NAT_MAP> permit 10
 match ip address <STATIC_NAT_ACL>
exit
!
ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP> route-map <STATIC_NAT_MAP>
!
end
write memory
# NAT_Route_Map_Policy_NAT_Verification_Commands
show ip interface brief
show running-config | section interface
show running-config | include ip nat
show running-config | include route-map
show running-config | section route-map
show running-config | section ip access-list
show access-lists
show access-lists <NAT_ACL_A>
show access-lists <NAT_ACL_B>
show route-map
show ip route
show ip route <DESTINATION_A_TEST_IP>
show ip route <DESTINATION_B_TEST_IP>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
show ip nat translations | include <INSIDE_LOCAL_IP>
ping <DESTINATION_A_TEST_IP>
ping <DESTINATION_B_TEST_IP>
traceroute <DESTINATION_A_TEST_IP>
traceroute <DESTINATION_B_TEST_IP>
clear ip nat translation *
show ip nat translations
# NAT_Route_Map_Policy_NAT_Rollback
conf t
!
no ip nat inside source route-map <NAT_MAP_A> pool <POOL_A> overload
no ip nat inside source route-map <NAT_MAP_A> pool <POOL_A> reversible
no ip nat inside source route-map <NAT_MAP_A> interface <OUTSIDE_INTERFACE_A> overload
no ip nat inside source route-map <NAT_MAP_B> pool <POOL_B> overload
no ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP> route-map <STATIC_NAT_MAP>
!
no ip nat pool <POOL_A> <POOL_A_START_IP> <POOL_A_END_IP> netmask <POOL_A_MASK>
no ip nat pool <POOL_B> <POOL_B_START_IP> <POOL_B_END_IP> netmask <POOL_B_MASK>
!
no route-map <NAT_MAP_A>
no route-map <NAT_MAP_B>
no route-map <STATIC_NAT_MAP>
!
no ip access-list extended <NAT_ACL_A>
no ip access-list extended <NAT_ACL_B>
no ip access-list extended <STATIC_NAT_ACL>
!
interface <INSIDE_INTERFACE>
 no ip nat inside
!
interface <OUTSIDE_INTERFACE_A>
 no ip nat outside
!
interface <OUTSIDE_INTERFACE_B>
 no ip nat outside
!
end
clear ip nat translation *
write memory
# NAT_Route_Map_Policy_NAT_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| NAT route-map command is rejected | Platform or image does not support that route-map NAT form | `ip nat inside source route-map ?` | Use a supported NAT form or move the lab to a router image that supports route-map NAT |
| No translations appear | Traffic does not match the route-map ACL | `show access-lists <NAT_ACL>` | Correct the ACL source, destination, and wildcard masks |
| ACL hits increment but NAT does not translate | Route map also matches the wrong outgoing interface | `show route-map` and `show ip route <DESTINATION>` | Correct the `match interface` line or remove it if not needed |
| Traffic translates through the wrong ISP or pool | Route-map conditions are too broad or generic NAT rule is catching traffic | `show running-config | include ip nat inside source` | Make route-map ACLs more specific and remove conflicting generic NAT rules |
| Pool NAT fails after limited sessions | NAT pool is exhausted or overload is missing | `show ip nat statistics` | Add `overload` or increase the NAT pool size |
| Static route-map NAT does not work | Route map does not match the destination flow | `show access-lists <STATIC_NAT_ACL>` | Match the correct source host and destination criteria |
| Outside-to-inside initiation fails | `reversible` was not configured where required | `show running-config | include reversible` | Add `reversible` to the pool-based route-map NAT rule if the design requires outside initiation |
| PBR works but NAT does not | PBR route map and NAT route map were confused | `show running-config | section route-map` | Build a separate NAT route map and attach it with `ip nat inside source route-map` |
| NAT works for ping but not app traffic | ACL matches ICMP test path but not the actual service path | `show access-lists` during app test | Match the real application source and destination flow |
| NAT stops after edits | Old translations remain active | `show ip nat translations verbose` | Clear translations with `clear ip nat translation *` during maintenance |
| Return traffic fails | Upstream or destination path does not route back to the selected global address | `show ip route <INSIDE_GLOBAL_IP>` on upstream path | Fix upstream routing, provider route, or selected NAT pool |
| Route-map counters look misleading | NAT route maps are used for NAT selection, not normal PBR forwarding policy | `show ip nat translations` and `show access-lists` | Validate NAT using ACL hits and NAT translation table, not only route-map counters |
| Interface PAT command fails | The image does not support route-map NAT with interface overload | `ip nat inside source route-map <MAP> ?` | Use pool-based route-map NAT with overload instead |
| Conditional outside source route-map NAT is expected to work | Cisco documents conditional translation as unsupported with `ip nat outside source route-map` | `show running-config | include ip nat outside source route-map` | Redesign using supported inside source route-map NAT, static NAT, or route leaking as appropriate |
##### Source_Basis
# NAT_Route_Map_Policy_NAT_Mental_Model
# NAT_Route_Map_Policy_NAT_Configuration_Checklist
# NAT_Route_Map_Policy_NAT_Skeleton
# NAT_Route_Map_Policy_NAT_Interface_PAT_Skeleton
# NAT_Route_Map_Policy_NAT_Reversible_Pool_Skeleton
# NAT_Route_Map_Policy_NAT_Static_Route_Map_Skeleton
# NAT_Route_Map_Policy_NAT_Verification_Commands
# NAT_Route_Map_Policy_NAT_Rollback
# NAT_Route_Map_Policy_NAT_Failure_Checks
