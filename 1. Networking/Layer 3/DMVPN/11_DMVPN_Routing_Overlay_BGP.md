
DMVPN_Routing_Overlay_BGP.md

# DMVPN_Routing_Overlay_BGP_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 10, Border Gateway Protocol | Supports iBGP, eBGP, route reflectors, BGP attributes, communities, and route policy |
| `_KB_INDEX.md` | Book 5, Advanced BGP Design and Implementation | Supports route reflection, iBGP scaling, attributes, route maps, prefix lists, and BGP design behavior |
| `_KB_INDEX.md` | IOS XE BGP Configuration Guide | Points practical IOS XE BGP syntax to `iosxe_combined_pdfs_.md` |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports mGRE, NHRP mapping, tunnel IP to NBMA mapping, and DMVPN phase behavior |
| `All_combined_part3.md` | Book 5 and Book 6 BGP sections | Supports iBGP, eBGP, route-reflector-client, next-hop behavior, attributes, and route verification |
| `iosxe_combined_pdfs_.md` | BGP Configuration Guide | Supports `neighbor`, `remote-as`, `update-source`, `next-hop-self`, `route-reflector-client`, `aggregate-address`, route maps, and BGP verification syntax |

# DMVPN_Routing_Overlay_BGP_Mental_Model

| Concept | Operational Meaning |
|---|---|
| BGP over DMVPN | BGP is the overlay routing protocol running across an already-working DMVPN tunnel |
| DMVPN first | BGP does not fix broken underlay, mGRE, tunnel source, or NHRP |
| Tunnel peering | BGP neighbors should normally peer using tunnel IPs or loopbacks reachable through the tunnel |
| Hub role | Hub commonly acts as iBGP route reflector or central eBGP transit peer |
| Spoke role | Spokes usually peer only with the hub, not every other spoke |
| iBGP route reflector | Lets spokes learn other spoke prefixes without full-mesh iBGP |
| eBGP hub-spoke | Each spoke can use a different ASN, with the hub acting as the central BGP peer |
| Update source | Forces the BGP session to source from the tunnel or loopback address |
| Next hop | The BGP next-hop attribute decides where the router forwards traffic |
| Route reflector next hop | A route reflector does not automatically rewrite NEXT_HOP unless configured to do so |
| Phase 1 behavior | Hub should remain transit for spoke-to-spoke traffic |
| Phase 2 behavior | Direct spoke-to-spoke forwarding needs the route next hop to resolve to the remote spoke |
| Phase 3 behavior | Routes may point toward the hub, summary, or default while NHRP shortcuts optimize forwarding |
| Summarization | Good for Phase 3, risky for Phase 2 if it hides real spoke next hops |
| Policy control | Prefix lists and route maps keep the overlay from accepting or advertising junk |
| Hard rule | If BGP is up but forwarding is wrong, check BGP next hop, RIB next hop, CEF, then NHRP |

# DMVPN_Routing_Overlay_BGP_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before BGP is configured |
| 2 | Confirm tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | DMVPN peers show up |
| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spokes and spokes have hub mapping |
| 5 | Confirm tunnel IP reachability | Hub, Spokes | `ping <remote-tunnel-ip>` | BGP peer tunnel IPs are reachable |
| 6 | Confirm the DMVPN phase | Hub, Spokes | `show running-config interface Tunnel<id>` | Phase 1, Phase 2, or Phase 3 behavior is known |
| 7 | Choose iBGP route-reflector or eBGP hub-spoke model | All Routers | Design check | BGP ASN model is known before configuration |
| 8 | Choose BGP peer source addresses | All Routers | Design check | Peering uses tunnel IPs or loopbacks reachable through DMVPN |
| 9 | Configure iBGP route-reflector hub if using one shared AS | Hub | See `Hub_iBGP_Route_Reflector_Config` | Hub peers with spokes and reflects routes |
| 10 | Configure iBGP spokes if using one shared AS | Spokes | See `Spoke_iBGP_Client_Config` | Spokes peer with hub route reflector |
| 11 | Configure eBGP hub if using separate spoke ASNs | Hub | See `Hub_eBGP_Central_Peer_Config` | Hub peers with each spoke ASN |
| 12 | Configure eBGP spokes if using separate spoke ASNs | Spokes | See `Spoke_eBGP_Client_Config` | Spokes peer with hub ASN |
| 13 | Configure update source when peering from tunnel interface | Hub, Spokes | See BGP config blocks | BGP uses the expected tunnel source address |
| 14 | Configure eBGP multihop only when required | Hub, Spokes | See `eBGP_Multihop_Option` | eBGP can form when peers are not directly connected |
| 15 | Activate IPv4 unicast neighbors | Hub, Spokes | See BGP config blocks | Neighbors exchange IPv4 unicast routes |
| 16 | Advertise hub prefixes | Hub | See BGP config blocks | Hub LAN or loopback prefixes enter BGP |
| 17 | Advertise spoke prefixes | Spokes | See BGP config blocks | Spoke LAN or loopback prefixes enter BGP |
| 18 | Configure route-reflector-client on hub for iBGP | Hub | See `Hub_iBGP_Route_Reflector_Config` | Hub reflects spoke routes to other spokes |
| 19 | Apply Phase 1 next-hop behavior if hub transit is required | Hub | See `BGP_Phase1_Next_Hop_Policy` | Spokes use hub as forwarding path |
| 20 | Preserve direct next-hop behavior if Phase 2 requires direct forwarding | Hub, Spokes | See `BGP_Phase2_Next_Hop_Reminder` | Remote spoke routes keep a usable remote spoke next hop |
| 21 | Use summary or default advertisement if Phase 3 requires scale | Hub | See `BGP_Phase3_Summary_Default_Options` | Spokes can route toward hub while NHRP shortcuts optimize forwarding |
| 22 | Add prefix filtering if lab requires route control | Hub, Spokes | See `BGP_Prefix_Filtering_Template` | Only intended prefixes are accepted or advertised |
| 23 | Add route-map policy if hub or cloud preference is required | Hub, Spokes | See `BGP_Path_Policy_Template` | Preferred hub or cloud path is deterministic |
| 24 | Verify BGP session state | Hub, Spokes | `show ip bgp summary` | Neighbor state shows prefix counts, not Idle or Active |
| 25 | Verify BGP table | Hub, Spokes | `show ip bgp` | Expected prefixes appear in BGP table |
| 26 | Verify BGP route installation | Hub, Spokes | `show ip route bgp` | Expected prefixes install into the routing table |
| 27 | Verify BGP next-hop reachability | Hub, Spokes | `show ip bgp <prefix>` and `show ip route <next-hop>` | BGP next hop is reachable |
| 28 | Verify Phase 1 forwarding | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Spoke-to-spoke traffic goes through hub |
| 29 | Verify Phase 2 route next hop | Spokes | `show ip bgp <remote-spoke-prefix>` | Next hop supports direct spoke-to-spoke forwarding |
| 30 | Verify Phase 2 NHRP resolution | Spokes | `show ip nhrp` | Remote spoke mapping appears after traffic trigger |
| 31 | Verify Phase 3 route model | Spokes | `show ip route <remote-spoke-lan-prefix>` | Route may point to hub, summary, or default |
| 32 | Verify Phase 3 shortcut | Spokes | `show ip nhrp` and `show ip cef <remote-spoke-lan-ip>` | Shortcut and forwarding entry appear after traffic trigger |
| 33 | Test hub-to-spoke LAN reachability | Hub | `ping <spoke-lan-ip>` | Hub can reach spoke LAN through BGP |
| 34 | Test spoke-to-hub LAN reachability | Spokes | `ping <hub-lan-ip>` | Spokes can reach hub LAN through BGP |
| 35 | Test spoke-to-spoke LAN reachability | Spokes | `ping <remote-spoke-lan-ip>` | Spokes can reach each other through expected DMVPN phase behavior |
| 36 | Verify advertised routes | Hub, Spokes | `show ip bgp neighbors <peer-ip> advertised-routes` | Only intended prefixes are advertised |
| 37 | Verify received routes if supported | Hub, Spokes | `show ip bgp neighbors <peer-ip> received-routes` | Only intended prefixes are received |
| 38 | Save working BGP-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_Routing_Overlay_BGP_Skeleton

```text
# Hub_iBGP_Route_Reflector_Config
conf t
router bgp <asn>
 bgp router-id <hub-router-id>
 no bgp default ipv4-unicast
 neighbor <spoke1-tunnel-ip> remote-as <asn>
 neighbor <spoke1-tunnel-ip> update-source Tunnel<id>
 neighbor <spoke2-tunnel-ip> remote-as <asn>
 neighbor <spoke2-tunnel-ip> update-source Tunnel<id>
 !
 address-family ipv4 unicast
  neighbor <spoke1-tunnel-ip> activate
  neighbor <spoke1-tunnel-ip> route-reflector-client
  neighbor <spoke2-tunnel-ip> activate
  neighbor <spoke2-tunnel-ip> route-reflector-client
  network <hub-prefix> mask <mask>
 exit-address-family
end
```

```text
# Spoke_iBGP_Client_Config
conf t
router bgp <asn>
 bgp router-id <spoke-router-id>
 no bgp default ipv4-unicast
 neighbor <hub-tunnel-ip> remote-as <asn>
 neighbor <hub-tunnel-ip> update-source Tunnel<id>
 !
 address-family ipv4 unicast
  neighbor <hub-tunnel-ip> activate
  network <spoke-prefix> mask <mask>
 exit-address-family
end
```

```text
# Hub_eBGP_Central_Peer_Config
conf t
router bgp <hub-asn>
 bgp router-id <hub-router-id>
 no bgp default ipv4-unicast
 neighbor <spoke1-tunnel-ip> remote-as <spoke1-asn>
 neighbor <spoke1-tunnel-ip> update-source Tunnel<id>
 neighbor <spoke2-tunnel-ip> remote-as <spoke2-asn>
 neighbor <spoke2-tunnel-ip> update-source Tunnel<id>
 !
 address-family ipv4 unicast
  neighbor <spoke1-tunnel-ip> activate
  neighbor <spoke2-tunnel-ip> activate
  network <hub-prefix> mask <mask>
 exit-address-family
end
```

```text
# Spoke_eBGP_Client_Config
conf t
router bgp <spoke-asn>
 bgp router-id <spoke-router-id>
 no bgp default ipv4-unicast
 neighbor <hub-tunnel-ip> remote-as <hub-asn>
 neighbor <hub-tunnel-ip> update-source Tunnel<id>
 !
 address-family ipv4 unicast
  neighbor <hub-tunnel-ip> activate
  network <spoke-prefix> mask <mask>
 exit-address-family
end
```

```text
# eBGP_Multihop_Option
! Use when eBGP peers are not directly connected by their BGP source addresses.
! Common when peering between loopbacks.

conf t
router bgp <asn>
 neighbor <peer-ip> ebgp-multihop <ttl>
end
```

```text
# BGP_Phase1_Next_Hop_Policy
! Phase 1 is hub transit.
! The hub should remain the forwarding path for spoke-to-spoke traffic.

conf t
router bgp <asn>
 address-family ipv4 unicast
  neighbor <spoke-tunnel-ip> next-hop-self
 exit-address-family
end
```

```text
# BGP_Phase2_Next_Hop_Reminder
! Phase 2 direct forwarding requires the remote spoke route to resolve to the remote spoke.
! For iBGP route reflection, avoid forcing next-hop-self on the hub if direct spoke-to-spoke forwarding is required.
! Route reflector behavior should preserve the client route next hop unless you intentionally rewrite it.
! For eBGP hub-spoke, normal eBGP next-hop behavior often makes the hub the next hop.
! If the route points to the hub, Phase 2 direct forwarding will not happen.
```

```text
# BGP_Phase3_Summary_Default_Options
! Phase 3 can use summary or default routing from the hub.
! Hub redirect and spoke shortcut handle forwarding optimization after traffic is triggered.

conf t
router bgp <asn>
 address-family ipv4 unicast
  aggregate-address <summary-prefix> <summary-mask> summary-only
 exit-address-family
end
```

```text
# BGP_Default_Origination_Option
! Use only if the default route exists in the local routing table.

conf t
router bgp <asn>
 address-family ipv4 unicast
  network 0.0.0.0
 exit-address-family
end
```

```text
# BGP_Prefix_Filtering_Template
conf t
ip prefix-list <PL-NAME> seq 10 permit <prefix>/<length>
!
route-map <RM-IN> permit 10
 match ip address prefix-list <PL-NAME>
!
router bgp <asn>
 address-family ipv4 unicast
  neighbor <peer-ip> route-map <RM-IN> in
 exit-address-family
end
```

```text
# BGP_Outbound_Filtering_Template
conf t
ip prefix-list <PL-OUT> seq 10 permit <prefix>/<length>
!
route-map <RM-OUT> permit 10
 match ip address prefix-list <PL-OUT>
!
router bgp <asn>
 address-family ipv4 unicast
  neighbor <peer-ip> route-map <RM-OUT> out
 exit-address-family
end
```

```text
# BGP_Path_Policy_Template
! Example knobs.
! Pick the attribute that matches the lab design.

conf t
route-map <RM-PREFER> permit 10
 set local-preference <value>
!
route-map <RM-MED> permit 10
 set metric <med-value>
!
router bgp <asn>
 address-family ipv4 unicast
  neighbor <peer-ip> route-map <RM-PREFER> in
 exit-address-family
end
```

```text
# Phase1_BGP_Reminder
! Phase 1 should send spoke-to-spoke traffic through the hub.
! Hub next-hop-self is usually correct for hub transit.
```

```text
# Phase2_BGP_Reminder
! Phase 2 direct spoke-to-spoke forwarding depends on route next hop.
! If BGP next hop points to the hub, traffic will hairpin through the hub.
! Check show ip bgp, show ip route, show ip cef, and show ip nhrp.
```

```text
# Phase3_BGP_Reminder
! Phase 3 can use summary or default routing from the hub.
! Hub uses ip nhrp redirect.
! Spokes use ip nhrp shortcut.
! First traffic may go through hub, then later traffic should shortcut directly.
```

# DMVPN_Routing_Overlay_BGP_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show up |
| NHRP mappings | Hub, Spokes | `show ip nhrp` | Expected tunnel-to-NBMA mappings exist |
| Tunnel state | Hub, Spokes | `show ip interface brief` | Tunnel interface shows `up/up` |
| Tunnel peer reachability | Hub, Spokes | `ping <remote-tunnel-ip>` | BGP peer tunnel IP is reachable |
| BGP summary | Hub, Spokes | `show ip bgp summary` | Neighbor state shows received prefix count |
| BGP neighbor detail | Hub, Spokes | `show ip bgp neighbors <peer-ip>` | Neighbor is established and IPv4 unicast is active |
| BGP table | Hub, Spokes | `show ip bgp` | Expected prefixes appear |
| BGP prefix detail | Hub, Spokes | `show ip bgp <prefix>` | Best path and next hop are correct |
| BGP routes in RIB | Hub, Spokes | `show ip route bgp` | Expected BGP routes install |
| Next-hop reachability | Hub, Spokes | `show ip route <bgp-next-hop>` | Next hop is reachable |
| CEF forwarding | Hub, Spokes | `show ip cef <destination-ip>` | Forwarding resolves through expected tunnel path |
| Route reflector config | Hub | `show running-config section router bgp` | Spokes are route-reflector clients if using iBGP RR |
| Hub next-hop policy | Hub | `show running-config section router bgp` | `next-hop-self` exists only where intended |
| Advertised routes | Hub, Spokes | `show ip bgp neighbors <peer-ip> advertised-routes` | Advertised prefixes match policy |
| Received routes | Hub, Spokes | `show ip bgp neighbors <peer-ip> received-routes` | Received prefixes match policy if display is supported |
| Phase 1 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes through hub |
| Phase 2 next hop | Spokes | `show ip bgp <remote-spoke-prefix>` | Next hop is usable for direct spoke forwarding |
| Phase 2 route | Spokes | `show ip route <remote-spoke-prefix>` | Route does not force hub unless intended |
| Phase 2 path | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes direct after NHRP resolution |
| Phase 3 shortcut state | Spokes | `show ip nhrp` | Shortcut entry appears after traffic |
| Phase 3 CEF state | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves toward remote spoke after shortcut |
| Hub preference | Spokes | `show ip bgp <prefix>` | Local preference, weight, AS path, or MED matches design |

# DMVPN_Routing_Overlay_BGP_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove BGP process if fully reverting | Hub, Spokes | `no router bgp <asn>` | BGP is removed |
| 2 | Remove one BGP neighbor only | Hub, Spokes | See `Rollback_BGP_Neighbor` | Selected peer is removed |
| 3 | Deactivate neighbor from IPv4 unicast only | Hub, Spokes | See `Rollback_BGP_AF_Activation` | Neighbor remains defined but stops IPv4 exchange |
| 4 | Remove route-reflector-client role | Hub | See `Rollback_BGP_Route_Reflector` | Hub stops reflecting routes for that spoke |
| 5 | Remove next-hop-self policy | Hub | See `Rollback_BGP_Next_Hop_Self` | Hub no longer forces itself as next hop |
| 6 | Remove advertised network statement | Hub, Spokes | See `Rollback_BGP_Network` | Prefix is no longer originated into BGP |
| 7 | Remove aggregate summary | Hub | See `Rollback_BGP_Aggregate` | Summary route is no longer advertised |
| 8 | Remove inbound route map | Hub, Spokes | See `Rollback_BGP_Route_Map` | Inbound policy is removed |
| 9 | Remove outbound route map | Hub, Spokes | See `Rollback_BGP_Route_Map` | Outbound policy is removed |
| 10 | Remove unused route-map object | Hub, Spokes | `no route-map <name>` | Route map is deleted |
| 11 | Remove unused prefix-list object | Hub, Spokes | `no ip prefix-list <name>` | Prefix list is deleted |
| 12 | Soft clear BGP after policy changes | Hub, Spokes | `clear ip bgp <peer-ip> soft in` | Inbound policy refreshes |
| 13 | Soft clear outbound BGP after policy changes | Hub, Spokes | `clear ip bgp <peer-ip> soft out` | Outbound policy refreshes |
| 14 | Hard clear only if needed | Hub, Spokes | `clear ip bgp <peer-ip>` | BGP session resets |
| 15 | Verify BGP routes are removed if fully reverted | Hub, Spokes | `show ip route bgp` | No BGP-learned routes remain |
| 16 | Verify DMVPN remains stable after BGP rollback | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN core remains operational |

```text
# Rollback_BGP_Neighbor
conf t
router bgp <asn>
 no neighbor <peer-ip>
end
```

```text
# Rollback_BGP_AF_Activation
conf t
router bgp <asn>
 address-family ipv4 unicast
  no neighbor <peer-ip> activate
 exit-address-family
end
```

```text
# Rollback_BGP_Route_Reflector
conf t
router bgp <asn>
 address-family ipv4 unicast
  no neighbor <spoke-tunnel-ip> route-reflector-client
 exit-address-family
end
```

```text
# Rollback_BGP_Next_Hop_Self
conf t
router bgp <asn>
 address-family ipv4 unicast
  no neighbor <spoke-tunnel-ip> next-hop-self
 exit-address-family
end
```

```text
# Rollback_BGP_Network
conf t
router bgp <asn>
 address-family ipv4 unicast
  no network <prefix> mask <mask>
 exit-address-family
end
```

```text
# Rollback_BGP_Aggregate
conf t
router bgp <asn>
 address-family ipv4 unicast
  no aggregate-address <summary-prefix> <summary-mask> summary-only
 exit-address-family
end
```

```text
# Rollback_BGP_Route_Map
conf t
router bgp <asn>
 address-family ipv4 unicast
  no neighbor <peer-ip> route-map <name> in
  no neighbor <peer-ip> route-map <name> out
 exit-address-family
end
```

```text
# Rollback_Remove_BGP_Process
conf t
no router bgp <asn>
end
```

# DMVPN_Routing_Overlay_BGP_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| BGP neighbor stuck Idle | No route to peer, wrong peer IP, or missing config | `show ip bgp summary` and `show ip route <peer-ip>` | Fix reachability or neighbor configuration |
| BGP neighbor stuck Active | TCP session cannot complete | `ping <peer-ip>` and `show ip bgp neighbors <peer-ip>` | Fix reachability, ACL, update-source, or peer settings |
| BGP wrong AS error | Remote AS mismatch | `show running-config section router bgp` | Correct `neighbor <peer-ip> remote-as <asn>` |
| Peering from tunnel IP fails | Missing update-source | `show ip bgp neighbors <peer-ip>` | Configure `neighbor <peer-ip> update-source Tunnel<id>` |
| eBGP loopback peering fails | TTL is too low | `show ip bgp summary` | Configure `neighbor <peer-ip> ebgp-multihop <ttl>` |
| BGP session up but no IPv4 routes | Neighbor not activated under address family | `show running-config section router bgp` | Add `neighbor <peer-ip> activate` under IPv4 unicast |
| Network statement does not advertise | Exact route is missing from RIB | `show ip route <prefix>` | Install matching route or fix network statement |
| iBGP spokes do not learn each other | Hub is not route reflector and no full mesh exists | `show running-config section router bgp` | Configure `route-reflector-client` on hub |
| Routes in BGP but not RIB | Next hop unreachable or better route exists | `show ip bgp <prefix>` and `show ip route <next-hop>` | Fix next-hop reachability or path preference |
| Phase 1 does not transit hub | Hub is not forcing transit path | `show ip bgp <prefix>` | Use next-hop policy so hub stays forwarding path |
| Phase 2 hairpins through hub | Hub rewrites next hop or summary hides spoke next hop | `show ip bgp <prefix>` and `show ip route <prefix>` | Preserve usable remote spoke next hop or use Phase 3 |
| Phase 2 route is direct but traffic fails | NHRP cannot resolve remote spoke | `show ip nhrp` | Fix mGRE and NHRP spoke-to-spoke resolution |
| Phase 3 shortcut does not form | Missing redirect or shortcut | `show running-config interface Tunnel<id>` | Add hub redirect and spoke shortcut |
| Wrong hub or cloud is preferred | BGP attributes are not engineered | `show ip bgp <prefix>` | Tune local preference, weight, MED, AS path, or route maps |
| Prefix is blocked | Prefix list or route map is too strict | `show ip prefix-list` and `show route-map` | Fix policy match conditions |
| BGP recursion points wrong way | Next hop resolves through wrong interface | `show ip route <next-hop>` and `show ip cef <next-hop>` | Fix next-hop policy or underlay route |
| BGP flaps | DMVPN instability, tunnel MTU, or TCP reachability issue | `show dmvpn` and `show ip bgp summary` | Fix DMVPN first, then BGP |
| Large packets fail | MTU or MSS issue | `ping <destination> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames BGP too early | DMVPN core is unstable | `show dmvpn` and `show ip nhrp` | Fix DMVPN first, then BGP |

##### Source_Basis
# DMVPN_Routing_Overlay_BGP_Mental_Model
# DMVPN_Routing_Overlay_BGP_Configuration_Checklist
# DMVPN_Routing_Overlay_BGP_Skeleton
# DMVPN_Routing_Overlay_BGP_Verification_Commands
# DMVPN_Routing_Overlay_BGP_Rollback
# DMVPN_Routing_Overlay_BGP_Failure_Checks