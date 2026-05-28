DMVPN_Routing_Overlay_EIGRP.md

# DMVPN_Routing_Overlay_EIGRP_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 3, EIGRP for IP | Supports EIGRP concepts, DUAL, metrics, feasible successors, summarization, filtering, redistribution, stub routing, and authentication |
| `_KB_INDEX.md` | Book 6, Chapter 7, EIGRP | Supports CCIE lab-level EIGRP configuration and verification workflows |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports mGRE, NHRP multicast mapping, tunnel IP to NBMA mapping, and DMVPN phase behavior |
| `All_combined_part3.md` | Book 3 and Book 6 EIGRP sections | Supports EIGRP neighbors, metrics, split horizon, next-hop behavior, summarization, stub routing, and route verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Tunnels and EIGRP sections | Supports modern DMVPN and EIGRP verification using `show dmvpn`, `show ip nhrp`, `show ip eigrp neighbors`, `show ip eigrp topology`, and `show ip route eigrp` |

# DMVPN_Routing_Overlay_EIGRP_Mental_Model

| Concept | Operational Meaning |
|---|---|
| EIGRP over DMVPN | EIGRP is the routing protocol running over an already-working DMVPN tunnel |
| DMVPN first | EIGRP does not fix broken underlay, mGRE, tunnel source, or NHRP |
| EIGRP adjacency | Neighbors form over the DMVPN tunnel interface |
| EIGRP multicast | EIGRP uses 224.0.0.10, so NHRP multicast mapping must exist |
| Hub role | Hub peers with spokes and advertises routes across the overlay |
| Spoke role | Spokes usually peer with the hub, not every other spoke manually |
| Split horizon | Hub tunnel often needs `no ip split-horizon eigrp <asn>` so spoke routes can be advertised back out the tunnel |
| Next-hop-self | Controls whether the hub rewrites the next hop to itself |
| Phase 1 behavior | Hub should remain the next hop because spoke-to-spoke traffic transits the hub |
| Phase 2 behavior | Hub should not rewrite next hop if direct spoke-to-spoke forwarding is required |
| Phase 3 behavior | Hub can summarize or advertise a default while NHRP redirect and shortcut optimize forwarding |
| EIGRP stub | Common on spokes to reduce query scope, but bad if the spoke must act as transit |
| Summarization | Good in Phase 3, dangerous in Phase 2 if it hides the real spoke next hop |
| Metric control | Bandwidth, delay, variance, and maximum paths control path choice |
| Hard rule | If EIGRP routes point to the wrong next hop, fix EIGRP policy or phase design before blaming NHRP |

# DMVPN_Routing_Overlay_EIGRP_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before EIGRP is configured |
| 2 | Confirm tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | DMVPN peers show up |
| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spokes and spokes have hub mapping |
| 5 | Confirm hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has dynamic multicast mapping |
| 6 | Confirm spoke multicast mapping to hub | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast to the hub NBMA |
| 7 | Confirm the DMVPN phase | Hub, Spokes | `show running-config interface Tunnel<id>` | Phase 1, Phase 2, or Phase 3 behavior is known |
| 8 | Enter EIGRP process on hub | Hub | See `Hub_EIGRP_Base_Config` | Hub EIGRP process is enabled |
| 9 | Set stable EIGRP router ID on hub | Hub | See `Hub_EIGRP_Base_Config` | Hub has predictable EIGRP identity |
| 10 | Disable auto-summary if using classic EIGRP | Hub | See `Hub_EIGRP_Base_Config` | Hub avoids classful summaries |
| 11 | Make hub passive by default | Hub | See `Hub_EIGRP_Base_Config` | Hub does not form unintended adjacencies |
| 12 | Enable EIGRP on hub tunnel | Hub | See `Hub_EIGRP_Base_Config` | Hub can form spoke adjacencies |
| 13 | Advertise hub tunnel network | Hub | See `Hub_EIGRP_Base_Config` | EIGRP runs on the DMVPN tunnel |
| 14 | Advertise hub LAN or loopback networks | Hub | See `Hub_EIGRP_Base_Config` | Hub internal prefixes enter EIGRP |
| 15 | Disable split horizon on hub tunnel when spokes need each other’s routes | Hub | See `Hub_EIGRP_Tunnel_Common_Adjustments` | Hub can advertise spoke routes back out the tunnel |
| 16 | Apply Phase 1 next-hop behavior if using Phase 1 | Hub | See `Hub_EIGRP_Phase1_Next_Hop` | Spokes use hub as next hop |
| 17 | Apply Phase 2 next-hop behavior if using Phase 2 | Hub | See `Hub_EIGRP_Phase2_Next_Hop` | Spokes can preserve remote spoke next hop |
| 18 | Apply Phase 3 summary or default if using Phase 3 | Hub | See `Hub_EIGRP_Phase3_Summary_Default` | Spokes can use hub summary or default |
| 19 | Enter EIGRP process on spokes | Spokes | See `Spoke_EIGRP_Base_Config` | Spoke EIGRP process is enabled |
| 20 | Set stable EIGRP router ID on spokes | Spokes | See `Spoke_EIGRP_Base_Config` | Each spoke has unique EIGRP router ID |
| 21 | Disable auto-summary on spokes if using classic EIGRP | Spokes | See `Spoke_EIGRP_Base_Config` | Spokes avoid classful summaries |
| 22 | Make spokes passive by default | Spokes | See `Spoke_EIGRP_Base_Config` | Spokes do not form unintended adjacencies |
| 23 | Enable EIGRP on spoke tunnel | Spokes | See `Spoke_EIGRP_Base_Config` | Spokes form EIGRP adjacency over DMVPN |
| 24 | Advertise spoke tunnel network | Spokes | See `Spoke_EIGRP_Base_Config` | EIGRP runs on spoke tunnel |
| 25 | Advertise spoke LAN or loopback networks | Spokes | See `Spoke_EIGRP_Base_Config` | Spoke internal prefixes enter EIGRP |
| 26 | Configure spoke stub if design is hub-and-spoke only | Spokes | See `Spoke_EIGRP_Stub_Option` | Query scope is reduced |
| 27 | Do not configure stub on transit spokes | Spokes | Design check | Any transit spoke is not blocked by stub behavior |
| 28 | Optional named EIGRP mode if lab requires it | Hub, Spokes | See `Named_EIGRP_Overlay_Template` | Named mode EIGRP runs over tunnel |
| 29 | Optional EIGRP add-path or multipath tuning if lab requires it | Hub, Spokes | See `EIGRP_Path_Control_Options` | Multiple usable paths are allowed by policy |
| 30 | Verify EIGRP neighbors | Hub, Spokes | `show ip eigrp neighbors` | Hub sees spokes and spokes see hub |
| 31 | Verify EIGRP protocol settings | Hub, Spokes | `show ip protocols` | AS, router ID, networks, and passive interfaces are correct |
| 32 | Verify EIGRP topology table | Hub, Spokes | `show ip eigrp topology` | Expected prefixes appear in topology table |
| 33 | Verify EIGRP routes in RIB | Hub, Spokes | `show ip route eigrp` | Remote prefixes appear as EIGRP routes |
| 34 | Verify Phase 1 next-hop behavior | Spokes | `show ip route <remote-spoke-prefix>` | Remote spoke routes point to hub |
| 35 | Verify Phase 2 next-hop behavior | Spokes | `show ip route <remote-spoke-prefix>` | Remote spoke routes point to remote spoke tunnel IP |
| 36 | Verify Phase 3 summary or default behavior | Spokes | `show ip route 0.0.0.0` | Summary or default route points toward hub |
| 37 | Test hub-to-spoke LAN reachability | Hub | `ping <spoke-lan-ip>` | Hub reaches spoke LAN through EIGRP |
| 38 | Test spoke-to-hub LAN reachability | Spokes | `ping <hub-lan-ip>` | Spokes reach hub LAN through EIGRP |
| 39 | Test spoke-to-spoke LAN reachability | Spokes | `ping <remote-spoke-lan-ip>` | Spokes reach each other through expected DMVPN phase |
| 40 | Prove Phase 1 forwarding path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes through hub |
| 41 | Prove Phase 2 forwarding path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes direct if next hop is correct |
| 42 | Prove Phase 3 shortcut path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Later traffic goes direct after shortcut |
| 43 | Verify Phase 3 NHRP and CEF shortcut | Spokes | `show ip nhrp` and `show ip cef <remote-spoke-lan-ip>` | Shortcut and forwarding entry exist |
| 44 | Save working EIGRP-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_Routing_Overlay_EIGRP_Skeleton

```text
# Hub_EIGRP_Base_Config
conf t
router eigrp <asn>
 eigrp router-id <hub-router-id>
 no auto-summary
 passive-interface default
 no passive-interface Tunnel<id>
 network <tunnel-network> <wildcard>
 network <hub-lan-or-loopback-network> <wildcard>
end
```

```text
# Hub_EIGRP_Tunnel_Common_Adjustments
conf t
interface Tunnel<id>
 no ip split-horizon eigrp <asn>
end
```

```text
# Hub_EIGRP_Phase1_Next_Hop
conf t
interface Tunnel<id>
 ip next-hop-self eigrp <asn>
end
```

```text
# Hub_EIGRP_Phase2_Next_Hop
conf t
interface Tunnel<id>
 no ip next-hop-self eigrp <asn>
end
```

```text
# Hub_EIGRP_Phase3_Summary_Default
conf t
interface Tunnel<id>
 ip summary-address eigrp <asn> <summary-prefix> <summary-mask>
end
```

```text
# Hub_EIGRP_Phase3_Default_Route
conf t
interface Tunnel<id>
 ip summary-address eigrp <asn> 0.0.0.0 0.0.0.0
end
```

```text
# Spoke_EIGRP_Base_Config
conf t
router eigrp <asn>
 eigrp router-id <spoke-router-id>
 no auto-summary
 passive-interface default
 no passive-interface Tunnel<id>
 network <tunnel-network> <wildcard>
 network <spoke-lan-or-loopback-network> <wildcard>
end
```

```text
# Spoke_EIGRP_Stub_Option
conf t
router eigrp <asn>
 eigrp stub connected summary
end
```

```text
# Named_EIGRP_Overlay_Template
conf t
router eigrp <name>
 address-family ipv4 unicast autonomous-system <asn>
  eigrp router-id <router-id>
  network <tunnel-network> <wildcard>
  network <lan-or-loopback-network> <wildcard>
  af-interface default
   passive-interface
  exit-af-interface
  af-interface Tunnel<id>
   no passive-interface
  exit-af-interface
 exit-address-family
end
```

```text
# Named_EIGRP_Hub_Tunnel_Adjustments
conf t
router eigrp <name>
 address-family ipv4 unicast autonomous-system <asn>
  af-interface Tunnel<id>
   no split-horizon
   no next-hop-self
  exit-af-interface
 exit-address-family
end
```

```text
# EIGRP_Path_Control_Options
! Use only when the lab calls for multiple path behavior.
! Exact platform support can vary by IOS XE image.

conf t
router eigrp <asn>
 variance <multiplier>
 maximum-paths <number>
end

show ip eigrp topology all-links
show ip route eigrp
```

```text
# Phase1_EIGRP_Reminder
! Hub transit is expected.
! Hub next-hop-self is normally retained.
! Spoke-to-spoke traceroute should go through the hub.
```

```text
# Phase2_EIGRP_Reminder
! Direct spoke-to-spoke forwarding needs the remote spoke as next hop.
! Hub should not rewrite next hop if direct forwarding is required.
! Hub summaries or default-only routing can break Phase 2 direct forwarding.
```

```text
# Phase3_EIGRP_Reminder
! Hub can summarize or advertise a default.
! Hub uses ip nhrp redirect.
! Spokes use ip nhrp shortcut.
! First traffic may go through the hub, then shortcut directly.
```

# DMVPN_Routing_Overlay_EIGRP_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show up |
| NHRP mappings | Hub, Spokes | `show ip nhrp` | Expected tunnel-to-NBMA mappings exist |
| Tunnel state | Hub, Spokes | `show ip interface brief` | Tunnel interface shows `up/up` |
| Hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has multicast mapping needed for EIGRP |
| Spoke multicast mapping | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast to hub |
| EIGRP neighbors | Hub, Spokes | `show ip eigrp neighbors` | Expected hub-spoke adjacencies appear |
| EIGRP protocol state | Hub, Spokes | `show ip protocols` | AS, router ID, networks, and passive interfaces are correct |
| EIGRP topology | Hub, Spokes | `show ip eigrp topology` | Expected prefixes appear |
| EIGRP all links | Hub, Spokes | `show ip eigrp topology all-links` | Alternate paths appear when expected |
| EIGRP routes | Hub, Spokes | `show ip route eigrp` | Remote prefixes appear as EIGRP routes |
| Hub split horizon | Hub | `show running-config interface Tunnel<id>` | `no ip split-horizon eigrp <asn>` is present when required |
| Hub next-hop behavior | Hub | `show running-config interface Tunnel<id>` | Phase 1 uses next-hop-self, Phase 2 avoids it |
| Phase 3 hub redirect | Hub | `show running-config interface Tunnel<id>` | `ip nhrp redirect` is present if using Phase 3 |
| Phase 3 spoke shortcut | Spokes | `show running-config interface Tunnel<id>` | `ip nhrp shortcut` is present if using Phase 3 |
| EIGRP stub state | Spokes | `show ip protocols` | Stub appears only where intended |
| Hub-to-spoke LAN ping | Hub | `ping <spoke-lan-ip>` | Ping succeeds |
| Spoke-to-hub LAN ping | Spokes | `ping <hub-lan-ip>` | Ping succeeds |
| Spoke-to-spoke LAN ping | Spokes | `ping <remote-spoke-lan-ip>` | Ping succeeds |
| Phase 1 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes through hub |
| Phase 2 next hop | Spokes | `show ip route <remote-spoke-prefix>` | Next hop is remote spoke tunnel IP |
| Phase 2 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes direct after NHRP resolution |
| Phase 3 shortcut state | Spokes | `show ip nhrp` | Shortcut entry appears after traffic |
| Phase 3 CEF state | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves toward remote spoke after shortcut |

# DMVPN_Routing_Overlay_EIGRP_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove EIGRP process from hub if fully reverting | Hub | `no router eigrp <asn>` | EIGRP is removed from hub |
| 2 | Remove EIGRP process from spokes if fully reverting | Spokes | `no router eigrp <asn>` | EIGRP is removed from spokes |
| 3 | Remove named EIGRP process if named mode was used | Hub, Spokes | `no router eigrp <name>` | Named EIGRP is removed |
| 4 | Remove only selected network advertisement | Hub, Spokes | See `Rollback_EIGRP_Networks` | Selected prefix stops being advertised |
| 5 | Make tunnel passive if stopping adjacency only | Hub, Spokes | See `Rollback_EIGRP_Tunnel_Adjacency` | EIGRP adjacency over tunnel drops |
| 6 | Restore hub split horizon if reverting to default | Hub | See `Rollback_Hub_EIGRP_Split_Horizon` | Hub returns to default split horizon behavior |
| 7 | Restore next-hop-self if reverting from Phase 2 behavior | Hub | See `Rollback_Hub_EIGRP_Next_Hop` | Hub rewrites next hop again |
| 8 | Remove Phase 3 summary or default advertisement | Hub | See `Rollback_Hub_EIGRP_Summary` | Hub stops advertising summary/default |
| 9 | Remove spoke stub if it causes reachability problems | Spokes | See `Rollback_Spoke_EIGRP_Stub` | Spoke is no longer stub |
| 10 | Remove path-control tuning if not needed | Hub, Spokes | See `Rollback_EIGRP_Path_Control` | Variance and maximum paths return to default |
| 11 | Clear EIGRP neighbors after major policy change | Hub, Spokes | `clear ip eigrp neighbors` | EIGRP adjacencies reset and rebuild |
| 12 | Clear routes after major changes | Hub, Spokes | `clear ip route *` | Routing table recalculates |
| 13 | Verify EIGRP routes are gone if fully removed | Hub, Spokes | `show ip route eigrp` | No EIGRP-learned routes remain |
| 14 | Verify DMVPN remains stable after EIGRP rollback | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN core remains operational |

```text
# Rollback_EIGRP_Networks
conf t
router eigrp <asn>
 no network <lan-or-loopback-network> <wildcard>
 no network <tunnel-network> <wildcard>
end
```

```text
# Rollback_EIGRP_Tunnel_Adjacency
conf t
router eigrp <asn>
 passive-interface Tunnel<id>
end
```

```text
# Rollback_Hub_EIGRP_Split_Horizon
conf t
interface Tunnel<id>
 ip split-horizon eigrp <asn>
end
```

```text
# Rollback_Hub_EIGRP_Next_Hop
conf t
interface Tunnel<id>
 ip next-hop-self eigrp <asn>
end
```

```text
# Rollback_Hub_EIGRP_Summary
conf t
interface Tunnel<id>
 no ip summary-address eigrp <asn> <summary-prefix> <summary-mask>
 no ip summary-address eigrp <asn> 0.0.0.0 0.0.0.0
end
```

```text
# Rollback_Spoke_EIGRP_Stub
conf t
router eigrp <asn>
 no eigrp stub connected summary
end
```

```text
# Rollback_EIGRP_Path_Control
conf t
router eigrp <asn>
 no variance <multiplier>
 maximum-paths 4
end
```

```text
# Rollback_Remove_EIGRP_Process
conf t
no router eigrp <asn>
end
```

# DMVPN_Routing_Overlay_EIGRP_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No EIGRP neighbors form | Tunnel passive, AS mismatch, missing network, or multicast mapping missing | `show ip eigrp neighbors` | Fix AS, network statement, passive state, or NHRP multicast |
| Hub sees spokes but spokes do not learn each other | Split horizon still enabled on hub tunnel | `show running-config interface Tunnel<id>` | Configure `no ip split-horizon eigrp <asn>` |
| Phase 1 path does not hit hub | Wrong next-hop or phase behavior | `show ip route <remote-spoke-prefix>` | Keep hub as next hop |
| Phase 2 path hits hub | Hub is setting next-hop-self or summarizing | `show ip route <remote-spoke-prefix>` | Use `no ip next-hop-self eigrp <asn>` and avoid summaries |
| Phase 2 route points direct but traffic fails | NHRP cannot resolve remote spoke | `show ip nhrp` | Fix NHRP and mGRE spoke-to-spoke resolution |
| Phase 3 shortcut does not form | Missing redirect or shortcut | `show running-config interface Tunnel<id>` | Add hub redirect and spoke shortcut |
| Spoke routes disappear after stub | Stub blocks expected advertisements | `show ip protocols` | Remove or adjust EIGRP stub |
| Query scope is too large | Spokes are not stub where they should be | `show ip protocols` | Configure EIGRP stub on non-transit spokes |
| Route is in topology but not RIB | Better route exists or feasibility condition not met | `show ip eigrp topology <prefix>` | Check successor, feasible successor, and metrics |
| Feasible successor missing | Feasibility condition not satisfied | `show ip eigrp topology all-links` | Tune metrics or accept that it is not feasible |
| Unequal-cost path not used | Variance or feasibility condition missing | `show ip eigrp topology all-links` | Configure variance and confirm feasible successor |
| Wrong hub is preferred | EIGRP metrics are not engineered | `show ip route <prefix>` | Tune delay or bandwidth intentionally |
| EIGRP flaps | DMVPN, NHRP, multicast, timers, or MTU issue | `show dmvpn` and `show ip eigrp neighbors detail` | Fix DMVPN stability before tuning EIGRP |
| Large packets fail | MTU or MSS issue | `ping <destination> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames EIGRP too early | DMVPN core is unstable | `show dmvpn` and `show ip nhrp` | Fix DMVPN first, then EIGRP |

##### Source_Basis
# DMVPN_Routing_Overlay_EIGRP_Mental_Model
# DMVPN_Routing_Overlay_EIGRP_Configuration_Checklist
# DMVPN_Routing_Overlay_EIGRP_Skeleton
# DMVPN_Routing_Overlay_EIGRP_Verification_Commands
# DMVPN_Routing_Overlay_EIGRP_Rollback
# DMVPN_Routing_Overlay_EIGRP_Failure_Checks
