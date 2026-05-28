

VRF_Lite_BGP_Route_Leaking_RD_RT.md

VRF_Lite_BGP_Route_Leaking_RD_RT

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Cisco Press / ENARSI and Advanced BGP source mapping | Confirms local Cisco Press material as the primary source path for VRF-Lite, BGP, RD, and RT behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 18, `Implementing and Verifying VRF-Lite` | Supports VRF-Lite isolation, per-VRF routing tables, `show ip route vrf`, and `ping vrf` validation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-23, `Configuring MP-BGPv4 Address Families for Multiple VRF Instances` | Provides `router bgp <asn>` and `address-family ipv4 vrf <VRF>` syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Route distinguisher and route target discussion after Example 18-23 | Defines RD uniqueness and RT import/export control for VRF route exchange |
| `All_combined_part3.md` | Advanced BGP / MPLS VPN section | Supports `vrf definition`, `rd`, `route-target export`, `route-target import`, `route-target both`, and VPNv4 verification logic |
| `All_combined_part3.md` | Advanced BGP verification examples | Supports `show bgp vpnv4 unicast all`, `show vrf detail`, `show ip vrf interfaces`, and per-VRF route verification |
# VRF_Lite_BGP_Route_Leaking_RD_RT_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VRF isolation | VRFs do not share routes by default |
| BGP route leaking | BGP can copy selected VRF routes into another VRF by using RD and RT logic |
| RD | Route distinguisher makes a VRF route unique inside BGP, especially when prefixes overlap |
| RT export | Marks routes leaving a VRF with a route target |
| RT import | Tells a VRF which tagged BGP routes it is allowed to import |
| Export/import match | A route leaks only when one VRF exports an RT that another VRF imports |
| Direction matters | RED exporting to GREEN does not automatically mean GREEN exports back to RED |
| Return path | Bidirectional traffic requires routes in both directions |
| BGP injection | Connected, static, or learned routes must enter BGP under the correct `address-family ipv4 vrf <VRF>` |
| VRF BGP table | `show bgp vpnv4 unicast all` proves the routes entered BGP with RD and RT context |
| VRF RIB | `show ip route vrf <VRF> bgp` proves imported BGP routes were installed into the VRF routing table |
| Common failure | RTs are configured correctly, but no routes are redistributed or advertised into BGP |
# VRF_Lite_BGP_Route_Leaking_RD_RT_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm VRFs exist before adding RD and RT policy | Leak router | `show vrf` | RED and GREEN VRFs exist |
| 2 | Confirm interfaces are already bound to the correct VRFs | Leak router | `show vrf ipv4 unicast interfaces` | RED and GREEN interfaces appear under the correct VRF |
| 3 | Confirm pre-leak isolation from RED to GREEN | Leak router | `ping vrf RED 172.16.10.10` | Ping fails before route leaking is configured |
| 4 | Confirm pre-leak isolation from GREEN to RED | Leak router | `ping vrf GREEN 10.10.10.10` | Ping fails before route leaking is configured |
| 5 | Enter global configuration mode | Leak router | `configure terminal` | Device enters configuration mode |
| 6 | Enter RED VRF definition mode | Leak router | `vrf definition RED` | RED VRF configuration mode opens |
| 7 | Assign RED route distinguisher | Leak router | `rd 65000:10` | RED routes receive unique VPNv4 identity in BGP |
| 8 | Enter RED IPv4 VRF address family | Leak router | `address-family ipv4` | RED IPv4 VRF address-family mode opens |
| 9 | Export RED routes with RED route target | Leak router | `route-target export 65000:10` | RED BGP routes are tagged with RT `65000:10` |
| 10 | Import GREEN routes into RED | Leak router | `route-target import 65000:20` | RED is allowed to import routes tagged by GREEN |
| 11 | Exit RED VRF address family | Leak router | `exit-address-family` | Device returns to VRF definition mode |
| 12 | Enter GREEN VRF definition mode | Leak router | `vrf definition GREEN` | GREEN VRF configuration mode opens |
| 13 | Assign GREEN route distinguisher | Leak router | `rd 65000:20` | GREEN routes receive unique VPNv4 identity in BGP |
| 14 | Enter GREEN IPv4 VRF address family | Leak router | `address-family ipv4` | GREEN IPv4 VRF address-family mode opens |
| 15 | Export GREEN routes with GREEN route target | Leak router | `route-target export 65000:20` | GREEN BGP routes are tagged with RT `65000:20` |
| 16 | Import RED routes into GREEN | Leak router | `route-target import 65000:10` | GREEN is allowed to import routes tagged by RED |
| 17 | Exit GREEN VRF address family | Leak router | `exit-address-family` | Device returns to VRF definition mode |
| 18 | Start BGP process for VRF route leaking | Leak router | `router bgp 65000` | BGP process starts |
| 19 | Enter RED BGP VRF address family | Leak router | `address-family ipv4 vrf RED` | BGP is scoped to the RED VRF |
| 20 | Inject RED connected routes into BGP for the lab | Leak router | `redistribute connected` | RED connected routes enter BGP and can be exported with RED RT |
| 21 | Inject RED static routes if RED has static-only prefixes | Leak router | `redistribute static` | RED static routes enter BGP if present |
| 22 | Exit RED BGP VRF address family | Leak router | `exit-address-family` | Device returns to BGP router mode |
| 23 | Enter GREEN BGP VRF address family | Leak router | `address-family ipv4 vrf GREEN` | BGP is scoped to the GREEN VRF |
| 24 | Inject GREEN connected routes into BGP for the lab | Leak router | `redistribute connected` | GREEN connected routes enter BGP and can be exported with GREEN RT |
| 25 | Inject GREEN static routes if GREEN has static-only prefixes | Leak router | `redistribute static` | GREEN static routes enter BGP if present |
| 26 | Exit GREEN BGP VRF address family | Leak router | `exit-address-family` | Device returns to BGP router mode |
| 27 | Verify RD and RT configuration | Leak router | `show vrf detail` | RED and GREEN show RD plus import and export RTs |
| 28 | Verify RED routes entered BGP as VPNv4 routes | Leak router | `show bgp vpnv4 unicast all | include Route Distinguisher|10.10.10.0` | RED prefix appears under RD `65000:10` |
| 29 | Verify GREEN routes entered BGP as VPNv4 routes | Leak router | `show bgp vpnv4 unicast all | include Route Distinguisher|172.16.10.0` | GREEN prefix appears under RD `65000:20` |
| 30 | Verify GREEN imported the RED route | Leak router | `show ip route vrf GREEN bgp` | GREEN table contains RED BGP-leaked prefix |
| 31 | Verify RED imported the GREEN route | Leak router | `show ip route vrf RED bgp` | RED table contains GREEN BGP-leaked prefix |
| 32 | Test GREEN-to-RED reachability | Leak router | `ping vrf GREEN 10.10.10.10 source 172.16.10.1` | Ping succeeds if GREEN imported RED and RED has return route |
| 33 | Test RED-to-GREEN reachability | Leak router | `ping vrf RED 172.16.10.10 source 10.10.10.1` | Ping succeeds if RED imported GREEN and GREEN has return route |
| 34 | Confirm unrelated VRFs did not import leaked routes | Leak router | `show ip route vrf BLUE bgp` | BLUE has no RED or GREEN leaked routes unless intentionally configured |
| 35 | Confirm global table did not receive VRF routes | Leak router | `show ip route | include 10.10.10.0|172.16.10.0` | RED and GREEN customer routes do not appear in global table |
| 36 | Save the working BGP route-leak baseline | Leak router | `write memory` | RD, RT, and BGP VRF route-leak configuration survives reload |
# VRF_Lite_BGP_Route_Leaking_RD_RT_Skeleton
configure terminal
vrf definition RED
 rd 65000:10
 address-family ipv4
  route-target export 65000:10
  route-target import 65000:20
 exit-address-family
exit
vrf definition GREEN
 rd 65000:20
 address-family ipv4
  route-target export 65000:20
  route-target import 65000:10
 exit-address-family
exit
router bgp 65000
 address-family ipv4 vrf RED
  redistribute connected
  redistribute static
 exit-address-family
 address-family ipv4 vrf GREEN
  redistribute connected
  redistribute static
 exit-address-family
end
write memory
! Optional shared-services pattern:
! RED imports only shared-services routes.
! SHARED imports RED only if return reachability is required.
configure terminal
vrf definition RED
 rd 65000:10
 address-family ipv4
  route-target export 65000:10
  route-target import 65000:100
 exit-address-family
exit
vrf definition SHARED
 rd 65000:100
 address-family ipv4
  route-target export 65000:100
  route-target import 65000:10
 exit-address-family
exit
router bgp 65000
 address-family ipv4 vrf RED
  redistribute connected
 exit-address-family
 address-family ipv4 vrf SHARED
  redistribute connected
  redistribute static
 exit-address-family
end
write memory
# VRF_Lite_BGP_Route_Leaking_RD_RT_Verification_Commands
show vrf
show vrf detail
show ip vrf
show ip vrf interfaces
show vrf ipv4 unicast interfaces
show running-config | section ^vrf definition
show running-config | section ^router bgp
show bgp vpnv4 unicast all
show bgp vpnv4 unicast all summary
show bgp vpnv4 unicast all | include Route Distinguisher|10.10.10.0|172.16.10.0
show bgp ipv4 unicast vrf RED
show bgp ipv4 unicast vrf GREEN
show ip route
show ip route vrf RED
show ip route vrf GREEN
show ip route vrf RED bgp
show ip route vrf GREEN bgp
show ip cef vrf RED 172.16.10.10
show ip cef vrf GREEN 10.10.10.10
ping vrf RED 172.16.10.10 source 10.10.10.1
ping vrf GREEN 10.10.10.10 source 172.16.10.1
traceroute vrf RED 172.16.10.10 source 10.10.10.1
traceroute vrf GREEN 10.10.10.10 source 172.16.10.1
# VRF_Lite_BGP_Route_Leaking_RD_RT_Rollback
configure terminal
router bgp 65000
 address-family ipv4 vrf RED
  no redistribute connected
  no redistribute static
 exit-address-family
 address-family ipv4 vrf GREEN
  no redistribute connected
  no redistribute static
 exit-address-family
exit
vrf definition RED
 address-family ipv4
  no route-target export 65000:10
  no route-target import 65000:20
 exit-address-family
 no rd 65000:10
exit
vrf definition GREEN
 address-family ipv4
  no route-target export 65000:20
  no route-target import 65000:10
 exit-address-family
 no rd 65000:20
exit
end
write memory
! Full cleanup only if the lab should remove BGP entirely.
configure terminal
no router bgp 65000
end
write memory
# VRF_Lite_BGP_Route_Leaking_RD_RT_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| VRF routes do not appear in BGP | Routes were never injected under the VRF BGP address family | `show bgp ipv4 unicast vrf RED` | Add `network`, `redistribute connected`, or `redistribute static` under `address-family ipv4 vrf RED` |
| Imported routes do not appear in the receiving VRF | Import RT does not match the exporting VRF RT | `show vrf detail` | Match receiving `route-target import` to sending `route-target export` |
| RED can reach GREEN, but GREEN cannot reach RED | Route leaking is configured in only one direction | `show ip route vrf RED bgp` and `show ip route vrf GREEN bgp` | Configure reverse export/import RT and inject reverse routes into BGP |
| BGP table has routes, but VRF RIB does not | RT import policy is missing or wrong | `show bgp vpnv4 unicast all` and `show ip route vrf <VRF> bgp` | Correct `route-target import` on the receiving VRF |
| Routes appear under wrong RD | RD configured under wrong VRF or reused incorrectly | `show bgp vpnv4 unicast all` | Correct `rd <asn>:<value>` under the intended VRF |
| Global table sees VRF routes | Redistribution or static route was configured in the wrong context | `show ip route` | Remove global route injection and configure under `address-family ipv4 vrf <VRF>` |
| Unrelated VRF imports routes | RT value is reused too broadly | `show vrf detail` | Use unique RTs or remove unnecessary `route-target import` commands |
| Ping fails even though BGP route exists | Missing return path or wrong source address | `show ip route vrf <VRF> <prefix>` | Verify both directions and use `ping vrf <VRF> <destination> source <source-ip>` |
| `show bgp vpnv4 unicast all` is empty | No RD/RT BGP VPNv4 route was created | `show running-config | section ^vrf definition` and `show running-config | section ^router bgp` | Configure RD, RT, and BGP route injection |
| `redistribute connected` leaks too much | Broad redistribution imported every connected prefix in the VRF | `show bgp ipv4 unicast vrf <VRF>` | Replace broad redistribution with selective `network` statements or route maps |
| Neighbor-based VRF BGP does not come up | Neighbor is configured under global BGP instead of VRF AF | `show bgp ipv4 unicast vrf <VRF> summary` | Move neighbor under `address-family ipv4 vrf <VRF>` |
| BGP route exists but forwarding fails | CEF or next-hop recursion problem | `show ip cef vrf <VRF> <destination>` | Fix next-hop reachability, interface state, or return routing |
##### Source_Basis
# VRF_Lite_BGP_Route_Leaking_RD_RT_Mental_Model
# VRF_Lite_BGP_Route_Leaking_RD_RT_Configuration_Checklist
# VRF_Lite_BGP_Route_Leaking_RD_RT_Skeleton
# VRF_Lite_BGP_Route_Leaking_RD_RT_Verification_Commands
# VRF_Lite_BGP_Route_Leaking_RD_RT_Rollback
# VRF_Lite_BGP_Route_Leaking_RD_RT_Failure_Checks

