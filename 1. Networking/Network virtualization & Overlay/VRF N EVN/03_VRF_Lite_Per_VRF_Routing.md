
VRF_Lite_Per_VRF_Routing.md

VRF_Lite_Per_VRF_Routing

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Cisco Press / ENARSI source mapping | Confirms Cisco Press ENARSI material as the primary local source for VRF-Lite routing |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 18, `Implementing and Verifying VRF-Lite` | Defines VRF-Lite as isolated routing and forwarding instances on one physical router |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-15, `show ip route vrf RED` | Shows that a VRF table initially contains only connected and local routes until static or dynamic routing is added |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-16, `Configuring EIGRP for Multiple VRF Instances` | Provides named EIGRP `address-family ipv4 vrf <VRF> autonomous-system <AS>` syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Examples 18-17 through 18-20 | Provides EIGRP per-VRF interface, neighbor, route, and ping verification commands |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | OSPFv2 VRF note after Example 18-21 | Supports separate OSPFv2 process per VRF using `router ospf <process-id> vrf <VRF>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-22 | Supports OSPFv3 address-family per VRF syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-23 | Supports BGP `address-family ipv4 vrf <VRF>` syntax for VRF-Lite |
# VRF_Lite_Per_VRF_Routing_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VRF routing table | Each VRF has its own RIB. Routes in RED do not automatically exist in GREEN, BLUE, or global |
| Connected-only baseline | After interface binding, the VRF table only knows connected and local routes |
| Per-VRF routing | Every VRF needs its own routing logic if it must learn remote networks |
| Dynamic routing per VRF | EIGRP, OSPF, or BGP must be instantiated inside the correct VRF context |
| Same protocol, separate worlds | RED EIGRP neighbors are not GREEN EIGRP neighbors even if they run on the same physical router |
| No implicit route leaking | Per-VRF routing builds reachability inside one VRF only. It does not share routes between VRFs |
| VRF-aware verification | Use `show ip route vrf <VRF>`, `ping vrf <VRF>`, and protocol-specific VRF commands |
| Wrong-context failure | Most failures come from checking or configuring the global table instead of the target VRF |
# VRF_Lite_Per_VRF_Routing_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm VRFs exist before routing is configured | Router | `show vrf` | Required VRFs appear with IPv4 enabled |
| 2 | Confirm interfaces are already bound to the correct VRFs | Router | `show vrf ipv4 unicast interfaces` | Each access and transport interface appears under the correct VRF |
| 3 | Confirm each VRF currently has only connected and local routes | Router | `show ip route vrf RED` | RED table shows connected and local routes before dynamic routing |
| 4 | Confirm GREEN connected baseline | Router | `show ip route vrf GREEN` | GREEN table shows connected and local routes before dynamic routing |
| 5 | Confirm BLUE connected baseline | Router | `show ip route vrf BLUE` | BLUE table shows connected and local routes before dynamic routing |
| 6 | Decide the routing method for the lab | All routers | `EIGRP`, `OSPF`, or `BGP` | One per-VRF routing method is selected and used consistently |
| 7 | Start named EIGRP for per-VRF routing | Router | `router eigrp VRFEXAMPLE` | Router enters named EIGRP configuration mode |
| 8 | Create RED EIGRP address family | Router | `address-family ipv4 vrf RED autonomous-system 10` | RED gets its own EIGRP IPv4 routing context |
| 9 | Advertise RED access interface address | Router | `network 10.0.1.1 0.0.0.0` | RED access interface participates in RED EIGRP |
| 10 | Advertise RED transport interface address | Router | `network 10.0.12.1 0.0.0.0` | RED transport interface participates in RED EIGRP |
| 11 | Create GREEN EIGRP address family | Router | `address-family ipv4 vrf GREEN autonomous-system 172` | GREEN gets its own EIGRP IPv4 routing context |
| 12 | Advertise GREEN access interface address | Router | `network 172.16.1.1 0.0.0.0` | GREEN access interface participates in GREEN EIGRP |
| 13 | Advertise GREEN transport interface address | Router | `network 172.16.12.1 0.0.0.0` | GREEN transport interface participates in GREEN EIGRP |
| 14 | Create BLUE EIGRP address family | Router | `address-family ipv4 vrf BLUE autonomous-system 192` | BLUE gets its own EIGRP IPv4 routing context |
| 15 | Advertise BLUE access interface address | Router | `network 192.168.1.1 0.0.0.0` | BLUE access interface participates in BLUE EIGRP |
| 16 | Advertise BLUE transport interface address | Router | `network 192.168.12.1 0.0.0.0` | BLUE transport interface participates in BLUE EIGRP |
| 17 | Repeat the matching per-VRF EIGRP configuration on neighbor routers | Neighbor routers | `router eigrp VRFEXAMPLE` | Neighbor routers run the same VRF-specific routing model |
| 18 | Verify RED EIGRP interfaces | Router | `show ip eigrp vrf RED interfaces` | Only RED interfaces appear in RED EIGRP |
| 19 | Verify GREEN EIGRP interfaces | Router | `show ip eigrp vrf GREEN interfaces` | Only GREEN interfaces appear in GREEN EIGRP |
| 20 | Verify BLUE EIGRP interfaces | Router | `show ip eigrp vrf BLUE interfaces` | Only BLUE interfaces appear in BLUE EIGRP |
| 21 | Verify RED neighbor adjacency | Router | `show ip eigrp vrf RED neighbors` | RED neighbor appears on the RED transport interface |
| 22 | Verify GREEN neighbor adjacency | Router | `show ip eigrp vrf GREEN neighbors` | GREEN neighbor appears on the GREEN transport interface |
| 23 | Verify BLUE neighbor adjacency | Router | `show ip eigrp vrf BLUE neighbors` | BLUE neighbor appears on the BLUE transport interface |
| 24 | Verify RED learned routes | Router | `show ip route vrf RED eigrp` | RED table contains EIGRP-learned remote RED routes |
| 25 | Verify GREEN learned routes | Router | `show ip route vrf GREEN eigrp` | GREEN table contains EIGRP-learned remote GREEN routes |
| 26 | Verify BLUE learned routes | Router | `show ip route vrf BLUE eigrp` | BLUE table contains EIGRP-learned remote BLUE routes |
| 27 | Test RED end-to-end VRF reachability | Router | `ping vrf RED 10.0.3.3` | RED destination replies through the RED routing table |
| 28 | Test GREEN end-to-end VRF reachability | Router | `ping vrf GREEN 172.16.3.3` | GREEN destination replies through the GREEN routing table |
| 29 | Test BLUE end-to-end VRF reachability | Router | `ping vrf BLUE 192.168.3.3` | BLUE destination replies through the BLUE routing table |
| 30 | Prove the global table is not being used | Router | `ping 10.0.3.3` | Ping fails unless global routing also has a valid path |
| 31 | Prove cross-VRF isolation still exists | Router | `ping vrf RED 172.16.3.3` | Ping fails because GREEN routes are not in RED |
| 32 | Save the working per-VRF routing baseline | Router | `write memory` | Per-VRF routing configuration survives reload |
# VRF_Lite_Per_VRF_Routing_Skeleton
configure terminal
router eigrp VRFEXAMPLE
 address-family ipv4 vrf RED autonomous-system 10
  network 10.0.1.1 0.0.0.0
  network 10.0.12.1 0.0.0.0
 exit-address-family
 address-family ipv4 vrf GREEN autonomous-system 172
  network 172.16.1.1 0.0.0.0
  network 172.16.12.1 0.0.0.0
 exit-address-family
 address-family ipv4 vrf BLUE autonomous-system 192
  network 192.168.1.1 0.0.0.0
  network 192.168.12.1 0.0.0.0
 exit-address-family
end
write memory
! OSPFv2 alternative, use one OSPF process per VRF
configure terminal
router ospf 1 vrf RED
 network 10.0.1.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.255 area 0
router ospf 2 vrf GREEN
 network 172.16.1.0 0.0.0.255 area 0
 network 172.16.12.0 0.0.0.255 area 0
router ospf 3 vrf BLUE
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
end
write memory
! BGP alternative, use one BGP process with one address family per VRF
configure terminal
router bgp 65000
 address-family ipv4 vrf RED
  neighbor 10.0.12.2 remote-as 65000
  network 10.0.1.0 mask 255.255.255.0
 exit-address-family
 address-family ipv4 vrf GREEN
  neighbor 172.16.12.2 remote-as 65000
  network 172.16.1.0 mask 255.255.255.0
 exit-address-family
 address-family ipv4 vrf BLUE
  neighbor 192.168.12.2 remote-as 65000
  network 192.168.1.0 mask 255.255.255.0
 exit-address-family
end
write memory
# VRF_Lite_Per_VRF_Routing_Verification_Commands
show vrf
show vrf ipv4 unicast interfaces
show ip route
show ip route vrf RED
show ip route vrf GREEN
show ip route vrf BLUE
show ip eigrp vrf RED interfaces
show ip eigrp vrf GREEN interfaces
show ip eigrp vrf BLUE interfaces
show ip eigrp vrf RED neighbors
show ip eigrp vrf GREEN neighbors
show ip eigrp vrf BLUE neighbors
show ip route vrf RED eigrp
show ip route vrf GREEN eigrp
show ip route vrf BLUE eigrp
show ip eigrp vrf RED topology
show ip eigrp vrf GREEN topology
show ip eigrp vrf BLUE topology
show ip ospf vrf RED neighbor
show ip ospf vrf GREEN neighbor
show ip ospf vrf BLUE neighbor
show ip route vrf RED ospf
show ip route vrf GREEN ospf
show ip route vrf BLUE ospf
show bgp ipv4 unicast vrf RED summary
show bgp ipv4 unicast vrf GREEN summary
show bgp ipv4 unicast vrf BLUE summary
show ip route vrf RED bgp
show ip route vrf GREEN bgp
show ip route vrf BLUE bgp
ping vrf RED 10.0.3.3
ping vrf GREEN 172.16.3.3
ping vrf BLUE 192.168.3.3
ping 10.0.3.3
ping vrf RED 172.16.3.3
traceroute vrf RED 10.0.3.3
traceroute vrf GREEN 172.16.3.3
traceroute vrf BLUE 192.168.3.3
# VRF_Lite_Per_VRF_Routing_Rollback
! EIGRP rollback
configure terminal
router eigrp VRFEXAMPLE
 no address-family ipv4 vrf RED autonomous-system 10
 no address-family ipv4 vrf GREEN autonomous-system 172
 no address-family ipv4 vrf BLUE autonomous-system 192
end
write memory
! OSPFv2 rollback
configure terminal
no router ospf 1 vrf RED
no router ospf 2 vrf GREEN
no router ospf 3 vrf BLUE
end
write memory
! BGP VRF address-family rollback
configure terminal
router bgp 65000
 no address-family ipv4 vrf RED
 no address-family ipv4 vrf GREEN
 no address-family ipv4 vrf BLUE
end
write memory
# VRF_Lite_Per_VRF_Routing_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| VRF has only connected and local routes | No per-VRF static or dynamic routing configured | `show ip route vrf <VRF>` | Configure EIGRP, OSPF, BGP, or static routes inside the VRF |
| EIGRP interface missing from RED | Wrong `network` statement or interface not in RED | `show ip eigrp vrf RED interfaces` | Add the correct interface IP with a host wildcard or fix VRF binding |
| EIGRP neighbor missing | AS mismatch, wrong VRF, down interface, or passive-interface issue | `show ip eigrp vrf <VRF> neighbors` | Match AS number, VRF, interface state, and passive settings |
| Routes learned in one VRF but not another | Only one VRF address family was configured | `show running-config | section router eigrp` | Add the missing `address-family ipv4 vrf <VRF>` |
| OSPF neighbor missing | OSPF process is not tied to the correct VRF | `show ip ospf vrf <VRF> neighbor` | Use `router ospf <process-id> vrf <VRF>` and correct network statements |
| BGP neighbor stuck idle | Neighbor configured in global BGP context instead of VRF address family | `show bgp ipv4 unicast vrf <VRF> summary` | Move neighbor under `address-family ipv4 vrf <VRF>` |
| Ping fails even though route exists | Ping is using global table | `show ip route` and `show ip route vrf <VRF>` | Use `ping vrf <VRF> <destination>` |
| Cross-VRF ping succeeds unexpectedly | Route leaking is configured | `show running-config | include ip route vrf|route-target|import|export` | Remove route leaking when testing pure per-VRF routing |
| Routes appear in global table instead of VRF | Interface or routing process was configured in the wrong context | `show ip route` and `show vrf ipv4 unicast interfaces` | Rebind interface and rebuild routing under the correct VRF |
| Same protocol works in RED but not GREEN | Addressing, VLAN, or neighbor mismatch in one VRF only | `show vrf ipv4 unicast interfaces` and protocol neighbor command | Fix the failed VRF independently instead of assuming all VRFs share state |
##### Source_Basis
# VRF_Lite_Per_VRF_Routing_Mental_Model
# VRF_Lite_Per_VRF_Routing_Configuration_Checklist
# VRF_Lite_Per_VRF_Routing_Skeleton
# VRF_Lite_Per_VRF_Routing_Verification_Commands
# VRF_Lite_Per_VRF_Routing_Rollback
# VRF_Lite_Per_VRF_Routing_Failure_Checks