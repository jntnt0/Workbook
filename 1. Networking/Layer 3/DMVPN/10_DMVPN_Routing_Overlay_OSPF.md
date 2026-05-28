

DMVPN_Routing_Overlay_OSPF.md

# DMVPN_Routing_Overlay_OSPF_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 8, OSPF | Supports OSPF areas, network types, neighbor states, authentication, LSA behavior, and OSPF verification |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports mGRE, NHRP multicast mapping, tunnel IP to NBMA mapping, and DMVPN phase behavior |
| `All_combined_part3.md` | Book 6, Chapter 8, OSPF | Supports OSPF network types, neighbor formation, DR/BDR control, timers, route verification, and failure checks |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Tunnels and OSPF sections | Supports modern DMVPN and OSPF verification using `show dmvpn`, `show ip nhrp`, `show ip ospf neighbor`, `show ip ospf interface`, and `show ip route ospf` |

# DMVPN_Routing_Overlay_OSPF_Mental_Model

| Concept | Operational Meaning |
|---|---|
| OSPF over DMVPN | OSPF is the routing protocol running over an already-working DMVPN overlay |
| DMVPN first | OSPF does not fix broken underlay, mGRE, tunnel source, or NHRP |
| OSPF adjacency | Neighbors form over the DMVPN tunnel interface |
| OSPF multicast | OSPF uses 224.0.0.5 and 224.0.0.6, so NHRP multicast mapping matters |
| Tunnel interface | OSPF should run on the tunnel, not the public NBMA interface |
| OSPF network type | Controls neighbor discovery, DR/BDR behavior, next-hop behavior, and route presentation |
| Point-to-multipoint | Clean lab model because it avoids DR/BDR election and supports dynamic neighbors |
| Broadcast model | Works when the hub is forced as DR and spokes are forced to priority 0 |
| Non-broadcast model | Requires manual neighbor statements and is more brittle |
| Phase 1 behavior | Spoke-to-spoke traffic should transit the hub |
| Phase 2 behavior | Direct spoke-to-spoke forwarding depends on OSPF route next-hop behavior and NHRP resolution |
| Phase 3 behavior | Routes can point toward the hub while NHRP redirect and shortcut optimize forwarding |
| Passive-interface | LANs are usually passive; the tunnel must not be passive |
| Area design | Keep DMVPN tunnel in Area 0 unless the lab explicitly tests multi-area behavior |
| Hard rule | If `show dmvpn` and `show ip nhrp` are not clean, do not troubleshoot OSPF yet |

# DMVPN_Routing_Overlay_OSPF_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before OSPF is configured |
| 2 | Confirm tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | DMVPN peers show up |
| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spokes and spokes have hub mapping |
| 5 | Confirm hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has dynamic multicast mapping |
| 6 | Confirm spoke multicast mapping to hub | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast to the hub NBMA |
| 7 | Confirm the DMVPN phase | Hub, Spokes | `show running-config interface Tunnel<id>` | Phase 1, Phase 2, or Phase 3 behavior is known |
| 8 | Choose OSPF network type intentionally | All Routers | Design check | Network type is selected before OSPF neighbors are formed |
| 9 | Configure point-to-multipoint model if using the clean lab design | Hub, Spokes | See `OSPF_Point_To_Multipoint_Tunnel_Config` | OSPF avoids DR/BDR election |
| 10 | Configure broadcast model only if the lab requires DR behavior | Hub, Spokes | See `OSPF_Broadcast_Tunnel_Config` | Hub becomes DR and spokes stay DROTHER |
| 11 | Configure non-broadcast model only if the lab requires manual neighbors | Hub, Spokes | See `OSPF_Non_Broadcast_Tunnel_Config` | OSPF uses static neighbor statements |
| 12 | Enter OSPF process on hub | Hub | See `Hub_OSPF_Base_Config` | Hub OSPF process is enabled |
| 13 | Set stable OSPF router ID on hub | Hub | See `Hub_OSPF_Base_Config` | Hub has predictable OSPF identity |
| 14 | Make hub passive by default | Hub | See `Hub_OSPF_Base_Config` | Hub does not form unintended adjacencies |
| 15 | Enable OSPF on hub tunnel | Hub | See `Hub_OSPF_Base_Config` | Hub can form OSPF neighbors over DMVPN |
| 16 | Advertise hub tunnel into OSPF | Hub | See `Hub_OSPF_Interface_Enablement` | Tunnel participates in the selected OSPF area |
| 17 | Advertise hub LAN or loopback networks | Hub | See `Hub_OSPF_Interface_Enablement` | Hub internal prefixes enter OSPF |
| 18 | Enter OSPF process on spokes | Spokes | See `Spoke_OSPF_Base_Config` | Spoke OSPF process is enabled |
| 19 | Set stable OSPF router ID on spokes | Spokes | See `Spoke_OSPF_Base_Config` | Each spoke has unique OSPF router ID |
| 20 | Make spokes passive by default | Spokes | See `Spoke_OSPF_Base_Config` | Spokes do not form unintended adjacencies |
| 21 | Enable OSPF on spoke tunnel | Spokes | See `Spoke_OSPF_Base_Config` | Spokes form OSPF adjacency over DMVPN |
| 22 | Advertise spoke tunnel into OSPF | Spokes | See `Spoke_OSPF_Interface_Enablement` | Tunnel participates in the selected OSPF area |
| 23 | Advertise spoke LAN or loopback networks | Spokes | See `Spoke_OSPF_Interface_Enablement` | Spoke internal prefixes enter OSPF |
| 24 | Optional exact network-statement model if preferred | Hub, Spokes | See `OSPF_Exact_Network_Statement_Model` | OSPF matches exact tunnel and LAN interface IPs |
| 25 | Verify OSPF neighbors | Hub, Spokes | `show ip ospf neighbor` | Expected neighbors reach FULL state |
| 26 | Verify OSPF tunnel network type | Hub, Spokes | `show ip ospf interface Tunnel<id>` | Tunnel shows intended OSPF network type |
| 27 | Verify OSPF process settings | Hub, Spokes | `show ip protocols` | Router ID, area, passive state, and networks are correct |
| 28 | Verify OSPF routes | Hub, Spokes | `show ip route ospf` | Remote prefixes appear as OSPF routes |
| 29 | Verify OSPF database | Hub, Spokes | `show ip ospf database` | Expected LSAs are present |
| 30 | Verify Phase 1 forwarding | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Spoke-to-spoke traffic goes through hub |
| 31 | Verify Phase 2 route next hop | Spokes | `show ip route <remote-spoke-lan-prefix>` | Next hop supports direct spoke-to-spoke behavior if expected |
| 32 | Verify Phase 2 NHRP resolution | Spokes | `show ip nhrp` | Remote spoke mapping appears after traffic trigger |
| 33 | Verify Phase 3 route model | Spokes | `show ip route <remote-spoke-lan-prefix>` | Route may point to hub, summary, or default |
| 34 | Verify Phase 3 shortcut | Spokes | `show ip nhrp` | Shortcut entry appears after traffic trigger |
| 35 | Test hub-to-spoke LAN reachability | Hub | `ping <spoke-lan-ip>` | Hub can reach spoke LAN through OSPF |
| 36 | Test spoke-to-hub LAN reachability | Spokes | `ping <hub-lan-ip>` | Spokes can reach hub LAN through OSPF |
| 37 | Test spoke-to-spoke LAN reachability | Spokes | `ping <remote-spoke-lan-ip>` | Spokes can reach each other through expected DMVPN phase behavior |
| 38 | Save working OSPF-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_Routing_Overlay_OSPF_Skeleton

```text
# OSPF_Point_To_Multipoint_Tunnel_Config
conf t
interface Tunnel<id>
 ip ospf network point-to-multipoint
 ip ospf hello-interval <seconds>
 ip ospf dead-interval <seconds>
end
```

```text
# OSPF_Broadcast_Tunnel_Config
! Hub
conf t
interface Tunnel<id>
 ip ospf network broadcast
 ip ospf priority 255
end

! Spokes
conf t
interface Tunnel<id>
 ip ospf network broadcast
 ip ospf priority 0
end
```

```text
# OSPF_Non_Broadcast_Tunnel_Config
conf t
interface Tunnel<id>
 ip ospf network non-broadcast
end

! Hub neighbor statements
router ospf <process-id>
 neighbor <spoke1-tunnel-ip>
 neighbor <spoke2-tunnel-ip>
 neighbor <spoke3-tunnel-ip>
end

! Spoke neighbor statement
router ospf <process-id>
 neighbor <hub-tunnel-ip>
end
```

```text
# Hub_OSPF_Base_Config
conf t
router ospf <process-id>
 router-id <hub-router-id>
 passive-interface default
 no passive-interface Tunnel<id>
end
```

```text
# Hub_OSPF_Interface_Enablement
conf t
interface Tunnel<id>
 ip ospf <process-id> area <area-id>
!
interface <hub-lan-or-loopback-interface>
 ip ospf <process-id> area <area-id>
end
```

```text
# Spoke_OSPF_Base_Config
conf t
router ospf <process-id>
 router-id <spoke-router-id>
 passive-interface default
 no passive-interface Tunnel<id>
end
```

```text
# Spoke_OSPF_Interface_Enablement
conf t
interface Tunnel<id>
 ip ospf <process-id> area <area-id>
!
interface <spoke-lan-or-loopback-interface>
 ip ospf <process-id> area <area-id>
end
```

```text
# OSPF_Exact_Network_Statement_Model
conf t
router ospf <process-id>
 router-id <router-id>
 passive-interface default
 no passive-interface Tunnel<id>
 network <tunnel-interface-ip> 0.0.0.0 area <area-id>
 network <lan-or-loopback-interface-ip> 0.0.0.0 area <area-id>
end
```

```text
# Optional_OSPF_Tunnel_Cost_Control
conf t
interface Tunnel<id>
 ip ospf cost <cost>
end
```

```text
# Optional_OSPF_Authentication
conf t
interface Tunnel<id>
 ip ospf authentication message-digest
 ip ospf message-digest-key <key-id> md5 <key-string>
end
```

```text
# Phase1_OSPF_Reminder
! Phase 1 should send spoke-to-spoke traffic through the hub.
! Spokes use point-to-point GRE toward the hub.
! If spoke-to-spoke traceroute bypasses the hub, the lab is not behaving like Phase 1.
```

```text
# Phase2_OSPF_Reminder
! Phase 2 direct forwarding depends on the OSPF route next hop and NHRP resolution.
! If the route points to the hub, direct spoke-to-spoke forwarding will not happen.
! OSPF over DMVPN is sensitive to network type choice.
```

```text
# Phase3_OSPF_Reminder
! Phase 3 can route toward the hub while NHRP shortcuts optimize forwarding.
! Hub uses ip nhrp redirect.
! Spokes use ip nhrp shortcut.
! First traffic may go through the hub, then later traffic should shortcut directly.
```

# DMVPN_Routing_Overlay_OSPF_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show up |
| NHRP mappings | Hub, Spokes | `show ip nhrp` | Expected tunnel-to-NBMA mappings exist |
| Tunnel state | Hub, Spokes | `show ip interface brief` | Tunnel interface shows `up/up` |
| Hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has multicast mapping needed for OSPF |
| Spoke multicast mapping | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast to hub |
| OSPF neighbors | Hub, Spokes | `show ip ospf neighbor` | Expected neighbors are FULL |
| OSPF tunnel detail | Hub, Spokes | `show ip ospf interface Tunnel<id>` | Correct network type, area, timer, cost, and neighbor count appear |
| OSPF process state | Hub, Spokes | `show ip protocols` | Router ID, passive interfaces, and area participation are correct |
| OSPF routes | Hub, Spokes | `show ip route ospf` | Remote prefixes appear as OSPF routes |
| OSPF database | Hub, Spokes | `show ip ospf database` | Expected LSAs are present |
| Broadcast DR role | Hub | `show ip ospf neighbor` | Hub is DR if broadcast model is used |
| Broadcast spoke role | Spokes | `show ip ospf neighbor` | Spokes are DROTHER if broadcast model is used |
| Non-broadcast neighbors | Hub, Spokes | `show running-config section router ospf` | Manual neighbor statements exist if non-broadcast model is used |
| Hub-to-spoke LAN ping | Hub | `ping <spoke-lan-ip>` | Ping succeeds |
| Spoke-to-hub LAN ping | Spokes | `ping <hub-lan-ip>` | Ping succeeds |
| Spoke-to-spoke LAN ping | Spokes | `ping <remote-spoke-lan-ip>` | Ping succeeds |
| Phase 1 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes through hub |
| Phase 2 route next hop | Spokes | `show ip route <remote-spoke-lan-prefix>` | Next hop supports direct spoke behavior if required |
| Phase 2 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes direct after NHRP resolution if route next hop is correct |
| Phase 3 shortcut state | Spokes | `show ip nhrp` | Shortcut entry appears after traffic |
| Phase 3 CEF state | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves toward remote spoke after shortcut |

# DMVPN_Routing_Overlay_OSPF_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove OSPF process if fully reverting | Hub, Spokes | `no router ospf <process-id>` | OSPF is removed |
| 2 | Remove OSPF from tunnel only | Hub, Spokes | See `Rollback_OSPF_Tunnel_Enablement` | Tunnel stops participating in OSPF |
| 3 | Remove OSPF from LAN or loopback only | Hub, Spokes | See `Rollback_OSPF_LAN_Enablement` | LAN or loopback stops being advertised |
| 4 | Remove point-to-multipoint network type | Hub, Spokes | See `Rollback_OSPF_Network_Type` | Tunnel network type override is removed |
| 5 | Remove broadcast network type and priority | Hub, Spokes | See `Rollback_OSPF_Network_Type` | Broadcast model settings are removed |
| 6 | Remove non-broadcast network type and neighbors | Hub, Spokes | See `Rollback_OSPF_Non_Broadcast` | Manual neighbor model is removed |
| 7 | Remove custom OSPF timers | Hub, Spokes | See `Rollback_OSPF_Timers` | Timers return to defaults |
| 8 | Remove OSPF tunnel cost if configured | Hub, Spokes | See `Rollback_OSPF_Cost` | Tunnel cost returns to default |
| 9 | Remove OSPF authentication if configured | Hub, Spokes | See `Rollback_OSPF_Authentication` | OSPF authentication is removed |
| 10 | Clear OSPF process after major changes | Hub, Spokes | `clear ip ospf process` | OSPF restarts and recalculates |
| 11 | Verify OSPF routes are removed if fully reverted | Hub, Spokes | `show ip route ospf` | No OSPF-learned routes remain |
| 12 | Verify DMVPN remains stable after OSPF rollback | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN core remains operational |

```text
# Rollback_OSPF_Tunnel_Enablement
conf t
interface Tunnel<id>
 no ip ospf <process-id> area <area-id>
end
```

```text
# Rollback_OSPF_LAN_Enablement
conf t
interface <lan-or-loopback-interface>
 no ip ospf <process-id> area <area-id>
end
```

```text
# Rollback_OSPF_Network_Type
conf t
interface Tunnel<id>
 no ip ospf network point-to-multipoint
 no ip ospf network broadcast
 no ip ospf network non-broadcast
 no ip ospf priority
end
```

```text
# Rollback_OSPF_Non_Broadcast
conf t
router ospf <process-id>
 no neighbor <neighbor-tunnel-ip>
end
```

```text
# Rollback_OSPF_Timers
conf t
interface Tunnel<id>
 no ip ospf hello-interval
 no ip ospf dead-interval
end
```

```text
# Rollback_OSPF_Cost
conf t
interface Tunnel<id>
 no ip ospf cost
end
```

```text
# Rollback_OSPF_Authentication
conf t
interface Tunnel<id>
 no ip ospf authentication message-digest
 no ip ospf message-digest-key <key-id>
end
```

```text
# Rollback_Remove_OSPF_Process
conf t
no router ospf <process-id>
end
```

# DMVPN_Routing_Overlay_OSPF_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No OSPF neighbors form | Tunnel passive, area mismatch, network type mismatch, or DMVPN multicast missing | `show ip ospf neighbor` | Fix passive state, area, network type, or NHRP multicast |
| Neighbor stuck in INIT | One-way hellos or multicast issue | `show ip ospf neighbor` | Fix NHRP multicast and tunnel reachability |
| Neighbor stuck in 2-WAY | Broadcast DR behavior or priority issue | `show ip ospf neighbor` | Confirm hub is DR and spokes are DROTHER |
| Neighbor stuck in EXSTART or EXCHANGE | MTU mismatch or database exchange issue | `show ip ospf neighbor` | Match MTU or use `ip ospf mtu-ignore` only when appropriate |
| Neighbor flaps | Timer mismatch, tunnel instability, or NHRP issue | `show ip ospf interface Tunnel<id>` | Match timers and fix DMVPN stability |
| Wrong router becomes DR | Broadcast priority not controlled | `show ip ospf neighbor` | Set hub priority high and spokes priority 0 |
| Non-broadcast model fails | Manual neighbor statements missing | `show running-config section router ospf` | Add correct neighbor statements |
| No OSPF routes appear | Interfaces not enabled in OSPF | `show ip protocols` | Add interface-level OSPF or exact network statements |
| Routes appear but traffic fails | NHRP or CEF resolution problem | `show ip nhrp` and `show ip cef <destination>` | Fix DMVPN phase behavior |
| Phase 1 path does not hit hub | Wrong DMVPN phase behavior | `traceroute <remote-spoke-lan-ip>` | Fix Phase 1 tunnel model |
| Phase 2 path hits hub | OSPF next hop points to hub | `show ip route <remote-spoke-lan-prefix>` | Change network type/design or use Phase 3 |
| Phase 3 shortcut does not form | Missing redirect or shortcut | `show running-config interface Tunnel<id>` | Add hub redirect and spoke shortcut |
| Too many host routes appear | Point-to-multipoint OSPF behavior | `show ip route ospf` | Accept it if intended or change design |
| Wrong hub or cloud is preferred | OSPF cost is not engineered | `show ip route <prefix>` | Tune tunnel OSPF cost |
| Authentication mismatch | OSPF auth key or mode differs | `show ip ospf interface Tunnel<id>` | Match authentication settings |
| Large packets fail | MTU or MSS issue | `ping <destination> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames OSPF too early | DMVPN core is unstable | `show dmvpn` and `show ip nhrp` | Fix DMVPN first, then OSPF |

##### Source_Basis
# DMVPN_Routing_Overlay_OSPF_Mental_Model
# DMVPN_Routing_Overlay_OSPF_Configuration_Checklist
# DMVPN_Routing_Overlay_OSPF_Skeleton
# DMVPN_Routing_Overlay_OSPF_Verification_Commands
# DMVPN_Routing_Overlay_OSPF_Rollback
# DMVPN_Routing_Overlay_OSPF_Failure_Checks
