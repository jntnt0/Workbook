# NAT_Classic_Inside_Outside_Domain_Baseline_Mental_Model
| Concept                             | Operational Meaning                                                                                                                                           |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Classic NAT domain model            | IOS NAT depends on interface roles. Traffic must cross from an `ip nat inside` interface to an `ip nat outside` interface, or the reverse for return traffic. |
| Inside local                        | The real address of the internal host before translation. Example: `10.10.10.10`.                                                                             |
| Inside global                       | The translated address seen by the outside network. Example: the outside interface address or a NAT pool address.                                             |
| Outside global                      | The real outside host address as it exists in the outside network.                                                                                            |
| Outside local                       | The outside host address as represented inside the local NAT domain. This matters more in outside source NAT and overlapping designs.                         |
| NAT does not replace routing        | The router still needs routes before NAT can work. NAT translates packets that routing is already able to forward.                                            |
| ACL selects translated sources      | The NAT ACL identifies inside local addresses eligible for translation. It is not a security ACL by itself.                                                   |
| Static rules beat dynamic rules     | If a packet matches a static translation, that translation takes precedence before dynamic pool or PAT rules.                                                 |
| PAT overload                        | Multiple inside hosts can share one inside global address by using unique transport ports.                                                                    |
| Translation table is traffic driven | Dynamic NAT and PAT translations usually appear only after matching traffic crosses the NAT router.                                                           |
# NAT_Classic_Inside_Outside_Domain_Baseline_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the routed baseline before configuring NAT | NAT-RTR | `show ip interface brief` | Inside and outside interfaces are up/up with correct IP addresses |
| 2 | Confirm inside network reachability to the NAT router | Inside Host | `ping <INSIDE_GATEWAY_IP>` | Inside host can reach its default gateway |
| 3 | Confirm the NAT router has a route toward the outside network | NAT-RTR | `show ip route <OUTSIDE_DESTINATION_IP>` | Route points out the intended outside interface or next hop |
| 4 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters configuration mode |
| 5 | Configure the inside interface | NAT-RTR | `interface <INSIDE_INTERFACE>` | Device enters inside interface configuration mode |
| 6 | Mark the interface as the NAT inside domain | NAT-RTR | `ip nat inside` | Interface is now classified as the inside NAT side |
| 7 | Exit inside interface configuration | NAT-RTR | `exit` | Device returns to global configuration mode |
| 8 | Configure the outside interface | NAT-RTR | `interface <OUTSIDE_INTERFACE>` | Device enters outside interface configuration mode |
| 9 | Mark the interface as the NAT outside domain | NAT-RTR | `ip nat outside` | Interface is now classified as the outside NAT side |
| 10 | Exit outside interface configuration | NAT-RTR | `exit` | Device returns to global configuration mode |
| 11 | Define the inside source addresses eligible for NAT | NAT-RTR | `access-list <ACL_ID> permit <INSIDE_SUBNET> <WILDCARD_MASK>` | ACL matches inside local addresses that should be translated |
| 12 | Configure simple PAT using the outside interface address | NAT-RTR | `ip nat inside source list <ACL_ID> interface <OUTSIDE_INTERFACE> overload` | Inside hosts can share the outside interface address using PAT |
| 13 | Optional: configure a NAT pool instead of interface PAT | NAT-RTR | `ip nat pool <POOL_NAME> <START_IP> <END_IP> netmask <MASK>` | Pool of inside global addresses is created |
| 14 | Optional: bind the ACL to the NAT pool | NAT-RTR | `ip nat inside source list <ACL_ID> pool <POOL_NAME> overload` | Matching inside hosts translate to the pool with PAT |
| 15 | Confirm NAT interface roles | NAT-RTR | `show ip nat statistics` | Inside and outside interfaces are listed correctly |
| 16 | Generate inside-to-outside traffic | Inside Host | `ping <OUTSIDE_DESTINATION_IP>` | Traffic crosses from inside to outside and triggers translation |
| 17 | Verify active translations | NAT-RTR | `show ip nat translations` | Inside local and inside global mappings appear |
| 18 | Verify detailed translations if needed | NAT-RTR | `show ip nat translations verbose` | Detailed protocol and timeout data appears |
| 19 | Confirm end-to-end forwarding path | NAT-RTR | `show ip route <OUTSIDE_DESTINATION_IP>` | Route lookup still points toward the outside path |
| 20 | Save the working configuration | NAT-RTR | `write memory` | NAT baseline survives reload |
# NAT_Classic_Inside_Outside_Domain_Baseline_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 description INSIDE_LAN
 ip address <INSIDE_GATEWAY_IP> <INSIDE_MASK>
 ip nat inside
 no shutdown
!
interface <OUTSIDE_INTERFACE>
 description OUTSIDE_WAN
 ip address <OUTSIDE_IP> <OUTSIDE_MASK>
 ip nat outside
 no shutdown
!
access-list <ACL_ID> permit <INSIDE_SUBNET> <WILDCARD_MASK>
!
ip nat inside source list <ACL_ID> interface <OUTSIDE_INTERFACE> overload
!
ip route 0.0.0.0 0.0.0.0 <OUTSIDE_NEXT_HOP>
!
end
write memory
# NAT_Classic_Inside_Outside_Domain_Baseline_Optional_Pool_Skeleton
conf t
!
ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
access-list <ACL_ID> permit <INSIDE_SUBNET> <WILDCARD_MASK>
ip nat inside source list <ACL_ID> pool <POOL_NAME> overload
!
interface <INSIDE_INTERFACE>
 ip nat inside
!
interface <OUTSIDE_INTERFACE>
 ip nat outside
!
end
write memory
# NAT_Classic_Inside_Outside_Domain_Baseline_Verification_Commands
show ip interface brief
show running-config | section interface
show running-config | include ip nat
show running-config | include access-list
show access-lists
show ip route
show ip route <OUTSIDE_DESTINATION_IP>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
ping <OUTSIDE_DESTINATION_IP>
traceroute <OUTSIDE_DESTINATION_IP>
clear ip nat translation *
show ip nat translations
# NAT_Classic_Inside_Outside_Domain_Baseline_Rollback
conf t
!
no ip nat inside source list <ACL_ID> interface <OUTSIDE_INTERFACE> overload
no ip nat inside source list <ACL_ID> pool <POOL_NAME> overload
no ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
no access-list <ACL_ID>
!
interface <INSIDE_INTERFACE>
 no ip nat inside
!
interface <OUTSIDE_INTERFACE>
 no ip nat outside
!
end
clear ip nat translation *
write memory
# NAT_Classic_Inside_Outside_Domain_Baseline_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No translations appear | Traffic is not crossing inside to outside NAT domains | `show ip nat statistics` and `show ip route <DESTINATION>` | Correct `ip nat inside`, `ip nat outside`, or routing |
| Inside host cannot reach outside host | Missing default route or wrong next hop | `show ip route 0.0.0.0` | Add or correct `ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>` |
| NAT ACL has zero matches | ACL does not match inside local source addresses | `show access-lists` | Correct the ACL source and wildcard mask |
| NAT interface roles are reversed | Inside marked as outside or outside marked as inside | `show running-config | section interface` | Move `ip nat inside` and `ip nat outside` to correct interfaces |
| Ping from router works but host fails | Host default gateway or inside path is wrong | Check host gateway and `show arp` | Point host default gateway to the NAT router inside IP |
| Translations appear but replies fail | Upstream routing does not know return path, or outside filtering drops return traffic | `show ip nat translations`, upstream route check | Use PAT to outside interface or fix upstream route/security policy |
| Pool NAT fails | NAT pool is exhausted or wrong mask was used | `show ip nat statistics` | Expand pool, use overload, or correct pool mask |
| Static or old translation interferes | Stale dynamic translation or conflicting static NAT exists | `show ip nat translations verbose` | Remove bad static NAT or clear dynamic translations |
| NAT works only after clearing table | Old translation state was pinned to earlier config | `clear ip nat translation *` | Clear translations after changing NAT rules |
| Outside source NAT lab fails | Route to outside local address is missing | `show ip route <OUTSIDE_LOCAL_IP>` | Add route for the outside local address as required |
##### Source_Basis
# NAT_Classic_Inside_Outside_Domain_Baseline_Mental_Model
# NAT_Classic_Inside_Outside_Domain_Baseline_Configuration_Checklist
# NAT_Classic_Inside_Outside_Domain_Baseline_Skeleton
# NAT_Classic_Inside_Outside_Domain_Baseline_Optional_Pool_Skeleton
# NAT_Classic_Inside_Outside_Domain_Baseline_Verification_Commands
# NAT_Classic_Inside_Outside_Domain_Baseline_Rollback
# NAT_Classic_Inside_Outside_Domain_Baseline_Failure_Checks
