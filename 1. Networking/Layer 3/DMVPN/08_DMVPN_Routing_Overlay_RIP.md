
---

DMVPN_Routing_Overlay_RIP.md

# DMVPN_Routing_Overlay_RIP_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 9, Redistribution | Supports route control context where RIP participates with other routing protocols |
| `_KB_INDEX.md` | Book 6, Chapter 11, IPv6 | Mentions RIPng separately, useful for keeping this note scoped to IPv4 RIP over DMVPN |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports mGRE, NHRP multicast mapping, tunnel IP to NBMA mapping, and DMVPN phase behavior |
| `All_combined_part3.md` | Book 6 routing sections | Supports RIPv2 process configuration, `version 2`, `no auto-summary`, `network`, passive interfaces, and route verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Tunnels | Supports modern DMVPN verification using `show dmvpn`, `show ip nhrp`, and tunnel state checks before routing overlays are added |

# DMVPN_Routing_Overlay_RIP_Mental_Model

| Concept | Operational Meaning |
|---|---|
| RIP over DMVPN | RIP is the IPv4 routing protocol riding over an already-working DMVPN overlay |
| DMVPN first | RIP does not fix broken underlay, mGRE, tunnel source, or NHRP |
| RIPv2 | Use `version 2` so RIP supports classless routing information |
| Auto-summary | Disable it with `no auto-summary` to avoid classful summary problems |
| RIP multicast | RIPv2 uses multicast 224.0.0.9, so NHRP multicast mapping matters |
| Tunnel interface | RIP should run across the DMVPN tunnel, not the public NBMA interface |
| LAN interfaces | Advertise LAN networks, but usually keep LAN interfaces passive |
| Hub role | Hub learns spoke routes and must advertise them back toward other spokes |
| Split horizon | Hub tunnel may need `no ip split-horizon` so spoke routes can be advertised back out the same tunnel |
| Phase 1 behavior | Spoke-to-spoke traffic should transit the hub |
| Phase 2 behavior | Direct spoke-to-spoke forwarding depends on route next-hop and NHRP resolution |
| Phase 3 behavior | Spokes may route toward hub/default while NHRP redirect and shortcut optimize forwarding |
| RIP metric | Hop count only; 15 is usable max and 16 means unreachable |
| Hard rule | If `show dmvpn` and `show ip nhrp` are not clean, do not troubleshoot RIP yet |

# DMVPN_Routing_Overlay_RIP_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before RIP is configured |
| 2 | Confirm tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | DMVPN peers show up |
| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spokes and spokes have hub mapping |
| 5 | Confirm hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has dynamic multicast mapping if multicast-based routing is used |
| 6 | Confirm spoke multicast mapping to hub | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast to the hub NBMA |
| 7 | Confirm the DMVPN phase | Hub, Spokes | `show running-config interface Tunnel<id>` | Phase 1, Phase 2, or Phase 3 behavior is known |
| 8 | Enter RIP process on hub | Hub | See `Hub_RIP_Base_Config` | Hub RIP process is enabled |
| 9 | Force RIPv2 on hub | Hub | See `Hub_RIP_Base_Config` | Hub sends and receives RIPv2 updates |
| 10 | Disable RIP auto-summary on hub | Hub | See `Hub_RIP_Base_Config` | Hub avoids classful summaries |
| 11 | Make hub interfaces passive by default | Hub | See `Hub_RIP_Base_Config` | Hub does not send RIP updates on unintended interfaces |
| 12 | Enable RIP updates on hub tunnel | Hub | See `Hub_RIP_Base_Config` | Hub can exchange RIP updates over DMVPN |
| 13 | Advertise hub tunnel network | Hub | See `Hub_RIP_Base_Config` | RIP runs on the DMVPN tunnel |
| 14 | Advertise hub LAN network | Hub | See `Hub_RIP_Base_Config` | Hub LAN prefix is advertised into RIP |
| 15 | Disable split horizon on hub tunnel when spokes need each other’s routes | Hub | See `Hub_RIP_Tunnel_Split_Horizon_Adjustment` | Hub can advertise spoke routes back out the tunnel |
| 16 | Enter RIP process on each spoke | Spokes | See `Spoke_RIP_Base_Config` | Spoke RIP process is enabled |
| 17 | Force RIPv2 on each spoke | Spokes | See `Spoke_RIP_Base_Config` | Spokes send and receive RIPv2 updates |
| 18 | Disable RIP auto-summary on each spoke | Spokes | See `Spoke_RIP_Base_Config` | Spokes avoid classful summaries |
| 19 | Make spoke interfaces passive by default | Spokes | See `Spoke_RIP_Base_Config` | Spokes do not send RIP updates on unintended interfaces |
| 20 | Enable RIP updates on spoke tunnel | Spokes | See `Spoke_RIP_Base_Config` | Spokes can exchange RIP updates with hub |
| 21 | Advertise spoke tunnel network | Spokes | See `Spoke_RIP_Base_Config` | RIP runs on the spoke tunnel |
| 22 | Advertise spoke LAN network | Spokes | See `Spoke_RIP_Base_Config` | Spoke LAN prefix is advertised into RIP |
| 23 | Verify RIP process settings | Hub, Spokes | `show ip protocols` | RIP shows version 2, no auto-summary, and correct active tunnel |
| 24 | Verify RIP routes exist | Hub, Spokes | `show ip route rip` | Remote LAN routes appear as RIP routes |
| 25 | Verify RIP database | Hub, Spokes | `show ip rip database` | Expected remote prefixes appear in RIP database |
| 26 | Verify hub learned spoke LAN routes | Hub | `show ip route rip` | Hub learns spoke LAN prefixes |
| 27 | Verify spokes learned hub LAN routes | Spokes | `show ip route rip` | Spokes learn hub LAN prefix |
| 28 | Verify spokes learned other spoke LAN routes | Spokes | `show ip route rip` | Spokes learn remote spoke LAN prefixes |
| 29 | Verify Phase 1 forwarding | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Spoke-to-spoke traffic goes through hub |
| 30 | Verify Phase 2 route next hop | Spokes | `show ip route <remote-spoke-lan-prefix>` | Next hop supports direct spoke-to-spoke behavior if expected |
| 31 | Verify Phase 2 NHRP resolution | Spokes | `show ip nhrp` | Remote spoke mapping appears after traffic trigger |
| 32 | Verify Phase 3 route model | Spokes | `show ip route <remote-spoke-lan-prefix>` | Route may point to hub, summary, or default |
| 33 | Verify Phase 3 shortcut | Spokes | `show ip nhrp` | Shortcut entry appears after traffic trigger |
| 34 | Test hub-to-spoke LAN reachability | Hub | `ping <spoke-lan-ip>` | Hub can reach spoke LAN through RIP |
| 35 | Test spoke-to-hub LAN reachability | Spokes | `ping <hub-lan-ip>` | Spokes can reach hub LAN through RIP |
| 36 | Test spoke-to-spoke LAN reachability | Spokes | `ping <remote-spoke-lan-ip>` | Spokes can reach each other through expected DMVPN phase behavior |
| 37 | Debug RIP only if routes are missing | Hub, Spokes | See `RIP_Debug_Checks` | RIPv2 updates are seen on the tunnel |
| 38 | Save working RIP-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_Routing_Overlay_RIP_Skeleton

```text
# Hub_RIP_Base_Config
conf t
router rip
 version 2
 no auto-summary
 passive-interface default
 no passive-interface Tunnel<id>
 network <tunnel-major-network>
 network <hub-lan-major-network>
end
```

```text
# Hub_RIP_Tunnel_Split_Horizon_Adjustment
conf t
interface Tunnel<id>
 no ip split-horizon
end
```

```text
# Spoke_RIP_Base_Config
conf t
router rip
 version 2
 no auto-summary
 passive-interface default
 no passive-interface Tunnel<id>
 network <tunnel-major-network>
 network <spoke-lan-major-network>
end
```

```text
# Optional_Targeted_Passive_Model
conf t
router rip
 version 2
 no auto-summary
 network <tunnel-major-network>
 network <lan-major-network>
 passive-interface <lan-interface>
end
```

```text
# RIP_Debug_Checks
debug ip rip
undebug all
```

```text
# Phase1_RIP_Reminder
! Phase 1 should send spoke-to-spoke traffic through the hub.
! Spokes use point-to-point GRE toward the hub.
! If spoke-to-spoke traceroute bypasses the hub, the lab is not behaving like Phase 1.
```

```text
# Phase2_RIP_Reminder
! Phase 2 requires direct spoke-to-spoke forwarding to be supported by routing next-hop behavior and NHRP.
! If the remote spoke LAN route points to the hub, direct spoke-to-spoke forwarding will not happen.
! Avoid hub summaries or default-only design if direct Phase 2 forwarding is required.
```

```text
# Phase3_RIP_Reminder
! Phase 3 can use hub summary or default routing.
! Hub must use ip nhrp redirect.
! Spokes must use ip nhrp shortcut.
! First traffic may go through the hub, then later traffic should shortcut directly.
```

# DMVPN_Routing_Overlay_RIP_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show up |
| NHRP mappings | Hub, Spokes | `show ip nhrp` | Expected tunnel-to-NBMA mappings exist |
| Tunnel state | Hub, Spokes | `show ip interface brief` | Tunnel interface shows `up/up` |
| Hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has multicast mapping needed for RIP updates |
| Spoke multicast mapping | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast to hub |
| RIP process | Hub, Spokes | `show ip protocols` | RIP is running as version 2 |
| Auto-summary state | Hub, Spokes | `show ip protocols` | Auto-summary is disabled |
| Active RIP tunnel | Hub, Spokes | `show ip protocols` | Tunnel is not passive |
| RIP routes | Hub, Spokes | `show ip route rip` | Remote prefixes appear as RIP routes |
| RIP database | Hub, Spokes | `show ip rip database` | Expected remote prefixes are present |
| Hub split horizon | Hub | `show running-config interface Tunnel<id>` | `no ip split-horizon` is present when required |
| Hub-to-spoke LAN ping | Hub | `ping <spoke-lan-ip>` | Ping succeeds |
| Spoke-to-hub LAN ping | Spokes | `ping <hub-lan-ip>` | Ping succeeds |
| Spoke-to-spoke LAN ping | Spokes | `ping <remote-spoke-lan-ip>` | Ping succeeds |
| Phase 1 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Path goes through hub |
| Phase 2 route next hop | Spokes | `show ip route <remote-spoke-lan-prefix>` | Next hop supports direct spoke behavior if required |
| Phase 2 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Path goes direct after NHRP resolution if route next hop is correct |
| Phase 3 shortcut state | Spokes | `show ip nhrp` | Shortcut entry appears after traffic |
| Phase 3 CEF state | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves toward remote spoke after shortcut |
| RIP debug | Hub, Spokes | `debug ip rip` | RIPv2 updates are sent and received on tunnel |
| Stop debug | Hub, Spokes | `undebug all` | Debugging stops |

# DMVPN_Routing_Overlay_RIP_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove RIP process from hub if fully reverting | Hub | `no router rip` | RIP is removed from hub |
| 2 | Remove RIP process from spokes if fully reverting | Spokes | `no router rip` | RIP is removed from spokes |
| 3 | Remove only hub LAN advertisement if needed | Hub | See `Rollback_RIP_Networks` | Hub stops advertising selected LAN |
| 4 | Remove only spoke LAN advertisement if needed | Spokes | See `Rollback_RIP_Networks` | Spoke stops advertising selected LAN |
| 5 | Make tunnel passive if stopping RIP updates only | Hub, Spokes | See `Rollback_RIP_Tunnel_Adjacency` | RIP updates stop on tunnel |
| 6 | Restore split horizon on hub if reverting to default | Hub | See `Rollback_Hub_RIP_Split_Horizon` | Hub returns to default split horizon behavior |
| 7 | Stop RIP debug if enabled | Hub, Spokes | `undebug all` | Debugging is disabled |
| 8 | Clear routes after major RIP changes | Hub, Spokes | `clear ip route *` | Routing table recalculates |
| 9 | Verify RIP routes are removed if fully reverted | Hub, Spokes | `show ip route rip` | No RIP-learned routes remain |
| 10 | Verify DMVPN remains stable after RIP rollback | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN core remains operational |

```text
# Rollback_RIP_Networks
conf t
router rip
 no network <lan-major-network>
 no network <tunnel-major-network>
end
```

```text
# Rollback_RIP_Tunnel_Adjacency
conf t
router rip
 passive-interface Tunnel<id>
end
```

```text
# Rollback_Hub_RIP_Split_Horizon
conf t
interface Tunnel<id>
 ip split-horizon
end
```

```text
# Rollback_Remove_RIP_Process
conf t
no router rip
end
```

```text
# Clear_Routes_After_RIP_Change
clear ip route *
```

# DMVPN_Routing_Overlay_RIP_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No RIP routes appear | Tunnel network not included in RIP | `show ip protocols` | Add tunnel major network under RIP |
| RIP routes are classful or wrong | Auto-summary still enabled | `show ip protocols` | Configure `no auto-summary` |
| RIP updates not sent on tunnel | Tunnel is passive | `show ip protocols` | Configure `no passive-interface Tunnel<id>` |
| RIP updates not crossing DMVPN | NHRP multicast mapping is missing | `show running-config interface Tunnel<id>` | Add hub dynamic multicast and spoke multicast mapping |
| Hub learns spokes but spokes do not learn each other | Split horizon still enabled on hub tunnel | `show running-config interface Tunnel<id>` | Configure `no ip split-horizon` on hub tunnel |
| RIP updates leak onto LAN | LAN interfaces are not passive | `show ip protocols` | Use passive-interface default and unpassive only the tunnel |
| Phase 1 path does not hit hub | Wrong DMVPN phase behavior | `traceroute <remote-spoke-lan-ip>` | Fix Phase 1 tunnel design |
| Phase 2 path keeps hitting hub | Route next hop points to hub | `show ip route <remote-spoke-lan-prefix>` | Fix route next-hop behavior or use Phase 3 |
| Phase 2 breaks after summary/default | Hub summary hides real spoke routes | `show ip route` | Remove summary/default or redesign as Phase 3 |
| Phase 3 shortcut does not form | Missing redirect or shortcut | `show ip nhrp` and `show running-config interface Tunnel<id>` | Add hub redirect and spoke shortcut |
| Route exists but traffic fails | NHRP or CEF resolution problem | `show ip nhrp` and `show ip cef <destination>` | Fix DMVPN phase behavior and NHRP resolution |
| RIP metric reaches 16 | Route is unreachable by RIP metric | `show ip rip database` | Reduce path, fix redistribution, or use a stronger routing protocol |
| RIP flaps | Tunnel, NHRP, multicast, or timer instability | `show dmvpn` and `debug ip rip` | Fix DMVPN stability before tuning RIP |
| Large packets fail | MTU or MSS issue | `ping <destination> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames RIP too early | DMVPN core is unstable | `show dmvpn` and `show ip nhrp` | Fix DMVPN first, then RIP |

##### Source_Basis
# DMVPN_Routing_Overlay_RIP_Mental_Model
# DMVPN_Routing_Overlay_RIP_Configuration_Checklist
# DMVPN_Routing_Overlay_RIP_Skeleton
# DMVPN_Routing_Overlay_RIP_Verification_Commands
# DMVPN_Routing_Overlay_RIP_Rollback
# DMVPN_Routing_Overlay_RIP_Failure_Checks
