

Cisco’s EVN shared-services guide says route replication is configured under the destination VRF’s IPv4 address family, uses the RIB instead of BGP, supports static, EIGRP, and OSPF routes, and replicated routes show with a + in show ip route vrf. It also says redistribution is needed when those replicated routes must be advertised through the destination VRF’s IGP.  

EVN_Shared_Services_Route_Replication.md

EVN_Shared_Services_Route_Replication

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Local KB index check | No dedicated EVN shared-services source was found locally, so Cisco EVN documentation is used for EVN-specific syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 18, `Implementing and Verifying VRF-Lite` | Supports VRF isolation, VRF-aware verification, and the contrast between VRF-Lite route leaking and EVN route replication |
| `Cisco IOS XE 17.x IP Addressing Configuration Guide` | `Configuring Easy Virtual Network Shared Services` | Supports EVN shared services, route replication, route redistribution, and the `route-replicate` workflow |
| `Cisco IOS XE 17.x IP Addressing Configuration Guide` | `Route Replication Process in Easy Virtual Network` | Supports the RIB-based route replication model and the statement that EVN route replication has no BGP dependency |
| `Cisco IOS XE 17.x IP Addressing Configuration Guide` | `Route Replication Behavior for Easy Virtual Network` | Supports IPv4 address-family placement, replicated-route behavior, source protocol handling, and route preference rules |
| `Cisco IOS XE 17.x IP Addressing Configuration Guide` | `Configuring Redistribution to Share Services in Easy Virtual Network` | Supports `redistribute vrf <source-vrf> <protocol> <process-id> subnets` under the destination VRF IGP |
# EVN_Shared_Services_Route_Replication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Shared-services VRF | The VRF that owns common services such as DNS, DHCP, management, or application servers |
| User VRF | A client VRF that needs controlled access to the shared-services VRF |
| Route replication | EVN copies selected routes from one VRF RIB into another VRF RIB |
| Destination VRF placement | Configure `route-replicate` under the VRF that needs to receive routes |
| Bidirectional requirement | User VRFs need service routes, and the services VRF needs return routes to user prefixes |
| No BGP requirement | EVN route replication does not use RD, RT, import, or export |
| BGP restriction | EVN route replication is not the method for BGP routes; use BGP import/export if BGP is the leaking mechanism |
| Route map filter | Use a route map to copy only the intended service or user prefixes |
| Loop prevention | Do not blindly replicate and redistribute everything back into the source protocol |
| IGP redistribution | Replicated routes may need `redistribute vrf <source-vrf> <protocol>` so they propagate through the destination VRF |
| Replicated route marker | `show ip route vrf <VRF>` marks replicated routes with `+` |
| Best placement | Configure replication close to the shared-service subnet to reduce unnecessary redistribution and loop risk |
# EVN_Shared_Services_Route_Replication_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EVN baseline exists before shared services | EVN router | `show vnet tag` | SERVICES, RED, and GREEN have VNET tags |
| 2 | Confirm VNET trunks are working | EVN router | `show vnet detail` | VNETs appear with expected trunk and edge interfaces |
| 3 | Confirm VRF interfaces are in the correct tables | EVN router | `show vrf ipv4 unicast interfaces` | Service and user interfaces appear under the correct VRFs |
| 4 | Confirm SERVICES owns the shared service prefix | Shared-services router | `show ip route vrf SERVICES` | Shared prefix such as `192.168.100.0/24` exists in SERVICES |
| 5 | Confirm RED owns its user prefix | User router | `show ip route vrf RED` | RED user prefix such as `10.10.10.0/24` exists in RED |
| 6 | Confirm GREEN owns its user prefix | User router | `show ip route vrf GREEN` | GREEN user prefix such as `172.16.10.0/24` exists in GREEN |
| 7 | Prove RED cannot reach SERVICES before replication | EVN router | `ping vrf RED 192.168.100.10 source 10.10.10.1` | Ping fails before the shared-services route is replicated |
| 8 | Prove GREEN cannot reach SERVICES before replication | EVN router | `ping vrf GREEN 192.168.100.10 source 172.16.10.1` | Ping fails before the shared-services route is replicated |
| 9 | Prove SERVICES cannot return to RED before replication | EVN router | `ping vrf SERVICES 10.10.10.10 source 192.168.100.1` | Ping fails before the RED return route is replicated |
| 10 | Prove SERVICES cannot return to GREEN before replication | EVN router | `ping vrf SERVICES 172.16.10.10 source 192.168.100.1` | Ping fails before the GREEN return route is replicated |
| 11 | Identify the source protocol for the service route | Shared-services router | `show ip route vrf SERVICES 192.168.100.0 255.255.255.0` | Route source is known, such as connected, static, or OSPF 99 |
| 12 | Identify the source protocol for the RED user route | User router | `show ip route vrf RED 10.10.10.0 255.255.255.0` | Route source is known, such as connected, static, or OSPF 98 |
| 13 | Identify the source protocol for the GREEN user route | User router | `show ip route vrf GREEN 172.16.10.0 255.255.255.0` | Route source is known, such as connected, static, or OSPF 97 |
| 14 | Create a prefix list for shared service routes | EVN router | `ip prefix-list SERVICES_SHARED seq 5 permit 192.168.100.0/24` | Only the shared-service subnet is matched |
| 15 | Create a route map for service-to-user replication | EVN router | `route-map SERVICES_TO_USERS permit 10` then `match ip address prefix-list SERVICES_SHARED` | Service route replication is filtered |
| 16 | Create a prefix list for RED return routes | EVN router | `ip prefix-list RED_USERS seq 5 permit 10.10.10.0/24` | Only RED user subnet is matched |
| 17 | Create a route map for RED-to-services replication | EVN router | `route-map RED_TO_SERVICES permit 10` then `match ip address prefix-list RED_USERS` | RED return route replication is filtered |
| 18 | Create a prefix list for GREEN return routes | EVN router | `ip prefix-list GREEN_USERS seq 5 permit 172.16.10.0/24` | Only GREEN user subnet is matched |
| 19 | Create a route map for GREEN-to-services replication | EVN router | `route-map GREEN_TO_SERVICES permit 10` then `match ip address prefix-list GREEN_USERS` | GREEN return route replication is filtered |
| 20 | Enter RED VRF address-family mode | EVN router | `vrf definition RED` then `address-family ipv4` | Router enters RED IPv4 VRF address-family mode |
| 21 | Replicate SERVICES routes into RED | EVN router | `route-replicate from vrf SERVICES unicast ospf 99 route-map SERVICES_TO_USERS` | RED receives selected SERVICES OSPF routes |
| 22 | Enter GREEN VRF address-family mode | EVN router | `vrf definition GREEN` then `address-family ipv4` | Router enters GREEN IPv4 VRF address-family mode |
| 23 | Replicate SERVICES routes into GREEN | EVN router | `route-replicate from vrf SERVICES unicast ospf 99 route-map SERVICES_TO_USERS` | GREEN receives selected SERVICES OSPF routes |
| 24 | Enter SERVICES VRF address-family mode | EVN router | `vrf definition SERVICES` then `address-family ipv4` | Router enters SERVICES IPv4 VRF address-family mode |
| 25 | Replicate RED return routes into SERVICES | EVN router | `route-replicate from vrf RED unicast ospf 98 route-map RED_TO_SERVICES` | SERVICES receives selected RED routes |
| 26 | Replicate GREEN return routes into SERVICES | EVN router | `route-replicate from vrf GREEN unicast ospf 97 route-map GREEN_TO_SERVICES` | SERVICES receives selected GREEN routes |
| 27 | Verify replicated service route appears in RED | EVN router | `show ip route vrf RED 192.168.100.0 255.255.255.0` | Route appears with `+` as a replicated route |
| 28 | Verify replicated service route appears in GREEN | EVN router | `show ip route vrf GREEN 192.168.100.0 255.255.255.0` | Route appears with `+` as a replicated route |
| 29 | Verify RED return route appears in SERVICES | EVN router | `show ip route vrf SERVICES 10.10.10.0 255.255.255.0` | Route appears with `+` as a replicated route |
| 30 | Verify GREEN return route appears in SERVICES | EVN router | `show ip route vrf SERVICES 172.16.10.0 255.255.255.0` | Route appears with `+` as a replicated route |
| 31 | Redistribute SERVICES replicated routes into RED OSPF if RED has downstream routers | EVN router | `router ospf 98 vrf RED` then `redistribute vrf SERVICES ospf 99 subnets route-map SERVICES_TO_USERS` | RED OSPF can advertise the replicated SERVICES routes |
| 32 | Redistribute SERVICES replicated routes into GREEN OSPF if GREEN has downstream routers | EVN router | `router ospf 97 vrf GREEN` then `redistribute vrf SERVICES ospf 99 subnets route-map SERVICES_TO_USERS` | GREEN OSPF can advertise the replicated SERVICES routes |
| 33 | Redistribute RED replicated routes into SERVICES OSPF if SERVICES has downstream routers | EVN router | `router ospf 99 vrf SERVICES` then `redistribute vrf RED ospf 98 subnets route-map RED_TO_SERVICES` | SERVICES OSPF can advertise RED return routes |
| 34 | Redistribute GREEN replicated routes into SERVICES OSPF if SERVICES has downstream routers | EVN router | `router ospf 99 vrf SERVICES` then `redistribute vrf GREEN ospf 97 subnets route-map GREEN_TO_SERVICES` | SERVICES OSPF can advertise GREEN return routes |
| 35 | Verify RED OSPF knows redistribution is enabled | EVN router | `show ip protocols vrf RED` | RED OSPF shows redistribution from SERVICES OSPF |
| 36 | Verify GREEN OSPF knows redistribution is enabled | EVN router | `show ip protocols vrf GREEN` | GREEN OSPF shows redistribution from SERVICES OSPF |
| 37 | Verify SERVICES OSPF knows redistribution is enabled | EVN router | `show ip protocols vrf SERVICES` | SERVICES OSPF shows redistribution from RED and GREEN |
| 38 | Test RED access to shared service | EVN router | `ping vrf RED 192.168.100.10 source 10.10.10.1` | RED can reach the shared service |
| 39 | Test GREEN access to shared service | EVN router | `ping vrf GREEN 192.168.100.10 source 172.16.10.1` | GREEN can reach the shared service |
| 40 | Test SERVICES return path to RED | EVN router | `ping vrf SERVICES 10.10.10.10 source 192.168.100.1` | SERVICES can return to RED |
| 41 | Test SERVICES return path to GREEN | EVN router | `ping vrf SERVICES 172.16.10.10 source 192.168.100.1` | SERVICES can return to GREEN |
| 42 | Prove RED and GREEN are still isolated from each other | EVN router | `ping vrf RED 172.16.10.10 source 10.10.10.1` | Ping fails unless RED-to-GREEN replication was intentionally configured |
| 43 | Confirm no BGP route replication was attempted | EVN router | `show running-config | include route-replicate.*bgp` | No BGP-based EVN route replication appears |
| 44 | Save the working EVN shared-services baseline | EVN router | `write memory` | Route replication and redistribution survive reload |
# EVN_Shared_Services_Route_Replication_Skeleton
configure terminal
ip prefix-list SERVICES_SHARED seq 5 permit 192.168.100.0/24
ip prefix-list RED_USERS seq 5 permit 10.10.10.0/24
ip prefix-list GREEN_USERS seq 5 permit 172.16.10.0/24
route-map SERVICES_TO_USERS permit 10
 match ip address prefix-list SERVICES_SHARED
route-map RED_TO_SERVICES permit 10
 match ip address prefix-list RED_USERS
route-map GREEN_TO_SERVICES permit 10
 match ip address prefix-list GREEN_USERS
vrf definition RED
 address-family ipv4
  route-replicate from vrf SERVICES unicast ospf 99 route-map SERVICES_TO_USERS
 exit-address-family
exit
vrf definition GREEN
 address-family ipv4
  route-replicate from vrf SERVICES unicast ospf 99 route-map SERVICES_TO_USERS
 exit-address-family
exit
vrf definition SERVICES
 address-family ipv4
  route-replicate from vrf RED unicast ospf 98 route-map RED_TO_SERVICES
  route-replicate from vrf GREEN unicast ospf 97 route-map GREEN_TO_SERVICES
 exit-address-family
exit
router ospf 98 vrf RED
 redistribute vrf SERVICES ospf 99 subnets route-map SERVICES_TO_USERS
router ospf 97 vrf GREEN
 redistribute vrf SERVICES ospf 99 subnets route-map SERVICES_TO_USERS
router ospf 99 vrf SERVICES
 redistribute vrf RED ospf 98 subnets route-map RED_TO_SERVICES
 redistribute vrf GREEN ospf 97 subnets route-map GREEN_TO_SERVICES
end
write memory
! Connected-route variant:
! Use this if the source route is connected instead of OSPF.
configure terminal
vrf definition RED
 address-family ipv4
  route-replicate from vrf SERVICES unicast connected route-map SERVICES_TO_USERS
 exit-address-family
exit
router ospf 98 vrf RED
 redistribute vrf SERVICES connected subnets route-map SERVICES_TO_USERS
end
write memory
! Static-route variant:
! Use this if the source route is static instead of OSPF.
configure terminal
vrf definition RED
 address-family ipv4
  route-replicate from vrf SERVICES unicast static route-map SERVICES_TO_USERS
 exit-address-family
exit
router ospf 98 vrf RED
 redistribute vrf SERVICES static subnets route-map SERVICES_TO_USERS
end
write memory
# EVN_Shared_Services_Route_Replication_Verification_Commands
show vnet tag
show vnet detail
show vnet counters
show running-config vnet
show vrf
show vrf detail
show vrf ipv4 unicast interfaces
show ip vrf interfaces
show running-config | section ^vrf definition
show running-config | section ^router ospf
show running-config | section ^ip prefix-list
show running-config | section ^route-map
show ip route vrf SERVICES
show ip route vrf RED
show ip route vrf GREEN
show ip route vrf RED 192.168.100.0 255.255.255.0
show ip route vrf GREEN 192.168.100.0 255.255.255.0
show ip route vrf SERVICES 10.10.10.0 255.255.255.0
show ip route vrf SERVICES 172.16.10.0 255.255.255.0
show ip protocols vrf SERVICES
show ip protocols vrf RED
show ip protocols vrf GREEN
show ip ospf vrf SERVICES neighbor
show ip ospf vrf RED neighbor
show ip ospf vrf GREEN neighbor
show ip cef vrf RED 192.168.100.10
show ip cef vrf GREEN 192.168.100.10
show ip cef vrf SERVICES 10.10.10.10
show ip cef vrf SERVICES 172.16.10.10
ping vrf RED 192.168.100.10 source 10.10.10.1
ping vrf GREEN 192.168.100.10 source 172.16.10.1
ping vrf SERVICES 10.10.10.10 source 192.168.100.1
ping vrf SERVICES 172.16.10.10 source 192.168.100.1
ping vrf RED 172.16.10.10 source 10.10.10.1
ping vrf GREEN 10.10.10.10 source 172.16.10.1
traceroute vrf RED 192.168.100.10 source 10.10.10.1
traceroute vrf GREEN 192.168.100.10 source 172.16.10.1
# EVN_Shared_Services_Route_Replication_Rollback
configure terminal
router ospf 98 vrf RED
 no redistribute vrf SERVICES ospf 99 subnets route-map SERVICES_TO_USERS
 no redistribute vrf SERVICES connected subnets route-map SERVICES_TO_USERS
 no redistribute vrf SERVICES static subnets route-map SERVICES_TO_USERS
router ospf 97 vrf GREEN
 no redistribute vrf SERVICES ospf 99 subnets route-map SERVICES_TO_USERS
router ospf 99 vrf SERVICES
 no redistribute vrf RED ospf 98 subnets route-map RED_TO_SERVICES
 no redistribute vrf GREEN ospf 97 subnets route-map GREEN_TO_SERVICES
vrf definition RED
 address-family ipv4
  no route-replicate from vrf SERVICES unicast ospf 99 route-map SERVICES_TO_USERS
  no route-replicate from vrf SERVICES unicast connected route-map SERVICES_TO_USERS
  no route-replicate from vrf SERVICES unicast static route-map SERVICES_TO_USERS
 exit-address-family
exit
vrf definition GREEN
 address-family ipv4
  no route-replicate from vrf SERVICES unicast ospf 99 route-map SERVICES_TO_USERS
 exit-address-family
exit
vrf definition SERVICES
 address-family ipv4
  no route-replicate from vrf RED unicast ospf 98 route-map RED_TO_SERVICES
  no route-replicate from vrf GREEN unicast ospf 97 route-map GREEN_TO_SERVICES
 exit-address-family
exit
no route-map SERVICES_TO_USERS
no route-map RED_TO_SERVICES
no route-map GREEN_TO_SERVICES
no ip prefix-list SERVICES_SHARED
no ip prefix-list RED_USERS
no ip prefix-list GREEN_USERS
end
write memory
# EVN_Shared_Services_Route_Replication_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `route-replicate` command is rejected | Command entered in the wrong mode | `show running-config | section ^vrf definition` | Configure it under `vrf definition <VRF>` then `address-family ipv4` |
| Replicated route does not appear in RED | Source protocol does not match the actual route source | `show ip route vrf SERVICES <prefix>` | Change `ospf 99`, `connected`, or `static` to match the real source |
| Replicated route appears but downstream RED routers do not learn it | Missing redistribution into RED IGP | `show ip protocols vrf RED` | Add `redistribute vrf SERVICES <protocol> subnets route-map SERVICES_TO_USERS` |
| RED reaches service, but service cannot return | SERVICES lacks RED return route | `show ip route vrf SERVICES 10.10.10.0 255.255.255.0` | Replicate RED into SERVICES and redistribute if needed |
| GREEN reaches service, but service cannot return | SERVICES lacks GREEN return route | `show ip route vrf SERVICES 172.16.10.0 255.255.255.0` | Replicate GREEN into SERVICES and redistribute if needed |
| RED can reach GREEN unexpectedly | User VRFs are being replicated into each other or service redistribution is too broad | `show running-config | include route-replicate|redistribute vrf` | Remove RED-to-GREEN or GREEN-to-RED leakage and tighten route maps |
| Too many routes are replicated | Used `all` without a restrictive route map | `show ip route vrf <VRF>` | Add prefix lists and route maps, or replicate only the needed protocol |
| Route map matches nothing | Prefix list is wrong | `show ip prefix-list` and `show route-map` | Correct the prefix, mask length, or route-map match statement |
| Replicated route loses to native route | Native route is preferred over replicated route | `show ip route vrf <VRF> <prefix>` | Fix the native route if it is wrong, or remove the conflicting route |
| BGP routes are not replicated | EVN route replication is not the BGP leaking method | `show running-config | include route-replicate.*bgp` | Use BGP import/export with RD and RT instead |
| OSPF external routes flood too broadly | Redistribution route map is missing or too permissive | `show ip route vrf <VRF> ospf` | Apply tighter redistribution route maps |
| Ping fails but route exists | Wrong source address or missing reverse route | `ping vrf <VRF> <destination> source <source-ip>` | Test with the correct VRF source and verify both directions |
| CEF does not resolve replicated route | Next-hop or source VRF route is unresolved | `show ip cef vrf <VRF> <destination>` | Fix source VRF reachability and IGP adjacency |
| Lab works on one router but not another | Replication was configured away from the shared-services edge | `show ip route vrf SERVICES` on the service-edge router | Move or duplicate the replication point near the shared-services subnet |
##### Source_Basis
# EVN_Shared_Services_Route_Replication_Mental_Model
# EVN_Shared_Services_Route_Replication_Configuration_Checklist
# EVN_Shared_Services_Route_Replication_Skeleton
# EVN_Shared_Services_Route_Replication_Verification_Commands
# EVN_Shared_Services_Route_Replication_Rollback
# EVN_Shared_Services_Route_Replication_Failure_Checks
