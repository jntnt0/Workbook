
Multicast_RPF_Tunnel_And_Overlay_Failure.md

Multicast_RPF_Tunnel_And_Overlay_Failure


# Multicast_RPF_Tunnel_And_Overlay_Failure_Mental_Model
| Concept | Operational Meaning |
|---|---|
| RPF over tunnel | Multicast can arrive through a tunnel, but the router still checks whether that tunnel is the correct reverse path back to the source |
| Underlay vs overlay | The underlay makes the tunnel work. The overlay is where multicast routing may be expected to run |
| RPF failure | If multicast arrives on Tunnel0 but the unicast route to the source points out a physical underlay interface, the router drops the multicast packet |
| Tunnel interface as multicast path | PIM neighbors can form over GRE/DMVPN tunnels when PIM is enabled on the tunnel interface |
| Physical transport interface | The physical interface should not automatically be treated as the multicast path just because it carries tunnel packets |
| Unicast route controls RPF | Unless changed, the unicast routing table decides the RPF interface for the multicast source |
| Static mroute | `ip mroute` can override the multicast RPF path without changing normal unicast forwarding |
| Correct fix hierarchy | First fix bad unicast routing if unicast should also follow the overlay. Use `ip mroute` when multicast needs a different RPF path than unicast |
| PIM adjacency is not enough | A PIM neighbor over the tunnel proves control-plane adjacency, not that RPF accepts data-plane multicast |
| Mroute table is the final proof | `show ip mroute <GROUP>` must show the expected Incoming Interface, RPF neighbor, and Outgoing Interface List |
# Multicast_RPF_Tunnel_And_Overlay_Failure_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the multicast source, group, expected tunnel path, and receiver segment | All routers | `show ip interface brief` | Source-facing, tunnel, transit, and receiver-facing interfaces are known |
| 2 | Confirm underlay reachability to tunnel endpoints | Tunnel routers | `ping <REMOTE_TUNNEL_SOURCE_OR_DESTINATION>` | Tunnel endpoints are reachable through the underlay |
| 3 | Verify the tunnel interface is up | Tunnel routers | `show interface <TUNNEL_INTERFACE>` | Tunnel interface is up/up |
| 4 | Verify tunnel source and destination/NHRP mapping | Tunnel routers | `show running-config interface <TUNNEL_INTERFACE>` | Tunnel source, tunnel destination or NHRP mapping, and tunnel mode match the lab design |
| 5 | Confirm baseline unicast route to the multicast source | Transit and last-hop routers | `show ip route <SOURCE_IP>` | Router has a route to the source, even if this route later proves to be the wrong RPF path |
| 6 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 7 | Enable PIM on the source-facing routed interface | Source-side router | `interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-side interface participates in multicast |
| 8 | Enable PIM on the tunnel interface | Tunnel routers | `interface <TUNNEL_INTERFACE>`<br>`ip pim sparse-mode` | PIM can form neighbor relationships across the overlay |
| 9 | Enable PIM on the receiver-facing routed interface | Last-hop router | `interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Last-hop router can accept receiver joins |
| 10 | Do not enable PIM on underlay transport interfaces unless the lab explicitly requires underlay multicast | Tunnel routers | `show running-config interface <UNDERLAY_INTERFACE>` | Underlay interface does not accidentally become the multicast control-plane path |
| 11 | Configure a static RP for sparse-mode testing | All multicast routers | `ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]` | All routers agree on the RP for the multicast group |
| 12 | Verify RP mapping | All multicast routers | `show ip pim rp mapping` | Group maps to the expected RP |
| 13 | Verify PIM neighbors across the tunnel | Tunnel routers | `show ip pim neighbor` | PIM neighbor appears on the tunnel interface |
| 14 | Join the multicast group from the receiver side | Receiver or last-hop test router | `interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver membership exists for the group |
| 15 | Verify IGMP receiver state | Last-hop router | `show ip igmp groups` | Multicast group appears on the receiver-facing interface |
| 16 | Generate multicast traffic from the source | Source router | `ping <MULTICAST_GROUP> source <SOURCE_IP_OR_INTERFACE> repeat 5` | Source sends multicast traffic toward the group |
| 17 | Check whether the multicast route exists | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` state appears |
| 18 | Identify the RPF decision for the source | Transit and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF interface is displayed |
| 19 | Compare actual multicast incoming interface to the expected tunnel path | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | Incoming Interface should be `<TUNNEL_INTERFACE>` if multicast is expected over the overlay |
| 20 | Confirm the failure condition | Transit or last-hop router | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Failure is confirmed when multicast arrives through the tunnel but RPF points to a different interface |
| 21 | Correct the RPF path with unicast routing when overlay should also be the unicast path | Transit or last-hop router | `conf t`<br>`ip route <SOURCE_IP> <SOURCE_MASK> <TUNNEL_NEXT_HOP>` | Unicast route to source points through the overlay path |
| 22 | Correct the RPF path with static mroute when only multicast should use the overlay | Transit or last-hop router | `conf t`<br>`ip mroute <SOURCE_IP> <SOURCE_MASK> <TUNNEL_NEXT_HOP_OR_TUNNEL_INTERFACE>` | Multicast RPF path is overridden without changing normal unicast routing |
| 23 | Clear stale multicast state after correction | Affected routers | `clear ip mroute *` | Old failed multicast state is removed |
| 24 | Retest multicast traffic | Source router | `ping <MULTICAST_GROUP> source <SOURCE_IP_OR_INTERFACE> repeat 5` | Receiver responds or multicast counters increment |
| 25 | Verify corrected RPF and mroute state | Transit and last-hop routers | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | RPF interface matches tunnel path and OIL points downstream |
| 26 | Save the corrected configuration | All changed routers | `copy running-config startup-config` | Corrected tunnel multicast baseline is preserved |
# Multicast_RPF_Tunnel_And_Overlay_Failure_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <SOURCE_IP>                    = multicast source address
! <SOURCE_MASK>                  = source host mask or prefix mask
! <MULTICAST_GROUP>              = multicast group address
! <TUNNEL_INTERFACE>             = tunnel carrying multicast overlay traffic
! <TUNNEL_NEXT_HOP>              = overlay next hop toward multicast source
! <UNDERLAY_INTERFACE>           = physical transport-facing interface
! <RECEIVER_INTERFACE>           = interface with receiver or test join
! <RECEIVER_FACING_INTERFACE>    = last-hop router interface toward receiver
! <RP_ADDRESS>                   = rendezvous point address for PIM sparse mode
! =========================================================
! BASELINE CHECKS
! =========================================================
show ip interface brief
show interface <TUNNEL_INTERFACE>
show running-config interface <TUNNEL_INTERFACE>
show ip route <SOURCE_IP>
show ip rpf <SOURCE_IP>
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on all multicast routers
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! SOURCE-SIDE ROUTER
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 interface <TUNNEL_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS>
end
! =========================================================
! TUNNEL / TRANSIT ROUTER
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS>
end
! =========================================================
! LAST-HOP ROUTER
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS>
end
! =========================================================
! RECEIVER TEST JOIN
! Use this when a router is emulating a multicast receiver
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! FAILURE CONFIRMATION
! RPF fails when the route to the source points away from
! the interface where multicast traffic actually arrives.
! =========================================================
show ip route <SOURCE_IP>
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
show ip pim neighbor
! =========================================================
! FIX OPTION 1
! Use this when unicast and multicast should both prefer the overlay
! =========================================================
conf t
 ip route <SOURCE_IP> <SOURCE_MASK> <TUNNEL_NEXT_HOP>
end
! =========================================================
! FIX OPTION 2
! Use this when only multicast RPF should prefer the overlay
! This is the classic fix when unicast must stay on the underlay
! but multicast must be accepted from the tunnel.
! =========================================================
conf t
 ip mroute <SOURCE_IP> <SOURCE_MASK> <TUNNEL_NEXT_HOP_OR_TUNNEL_INTERFACE>
end
! =========================================================
! CLEAR AND RETEST
! =========================================================
clear ip mroute *
ping <MULTICAST_GROUP> source <SOURCE_IP_OR_INTERFACE> repeat 5
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
# Multicast_RPF_Tunnel_And_Overlay_Failure_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Tunnel routers | `ping <REMOTE_TUNNEL_ENDPOINT>` | Tunnel endpoint replies |
| Tunnel state | Tunnel routers | `show interface <TUNNEL_INTERFACE>` | Tunnel is up/up |
| Tunnel configuration | Tunnel routers | `show running-config interface <TUNNEL_INTERFACE>` | Tunnel source, destination or NHRP mapping, and mode are correct |
| Multicast global enablement | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface status | All multicast routers | `show ip pim interface` | Tunnel and required routed interfaces appear |
| PIM tunnel adjacency | Tunnel routers | `show ip pim neighbor` | PIM neighbor is learned over the tunnel |
| RP mapping | All multicast routers | `show ip pim rp mapping` | Group maps to the expected RP |
| Receiver membership | Last-hop router | `show ip igmp groups` | Receiver group is present |
| Unicast route to source | Transit and last-hop routers | `show ip route <SOURCE_IP>` | Route exists toward the source |
| RPF path | Transit and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF interface matches the intended multicast incoming interface |
| Multicast state | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` entries exist |
| Incoming interface | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | Incoming Interface is the expected tunnel when multicast is carried over the overlay |
| Outgoing interface list | Source-side and transit routers | `show ip mroute <MULTICAST_GROUP>` | OIL contains downstream tunnel or receiver-facing interfaces |
| Counter movement | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during test traffic |
| Static mroute override | Affected router | `show running-config | include ^ip mroute` | Static mroute exists only where RPF override is required |
| End-to-end test | Source router | `ping <MULTICAST_GROUP> source <SOURCE_IP_OR_INTERFACE> repeat 5` | Receiver replies or downstream packet counters increment |
# Multicast_RPF_Tunnel_And_Overlay_Failure_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE STATIC MROUTE RPF OVERRIDE
! =========================================================
conf t
 no ip mroute <SOURCE_IP> <SOURCE_MASK> <TUNNEL_NEXT_HOP_OR_TUNNEL_INTERFACE>
end
! =========================================================
! REMOVE UNICAST STATIC ROUTE IF IT WAS ONLY ADDED FOR THE LAB
! =========================================================
conf t
 no ip route <SOURCE_IP> <SOURCE_MASK> <TUNNEL_NEXT_HOP>
end
! =========================================================
! REMOVE STATIC RP CONFIGURATION
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS>
end
! =========================================================
! REMOVE PIM FROM TUNNEL AND ROUTED INTERFACES
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE MULTICAST ROUTING ONLY IF NOTHING ELSE DEPENDS ON IT
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR MULTICAST STATE
! =========================================================
clear ip mroute *
clear ip igmp group *
# Multicast_RPF_Tunnel_And_Overlay_Failure_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Tunnel is down | Underlay reachability, tunnel source, tunnel destination, or NHRP mapping is wrong | Tunnel routers | `show interface <TUNNEL_INTERFACE>`<br>`show running-config interface <TUNNEL_INTERFACE>` | Fix tunnel source/destination, underlay route, or NHRP mapping |
| No PIM neighbor over tunnel | PIM missing on tunnel, tunnel down, or multicast control traffic not supported across tunnel design | Tunnel routers | `show ip pim neighbor`<br>`show ip pim interface` | Enable `ip pim sparse-mode` on the tunnel and fix tunnel reachability |
| PIM neighbor exists but multicast fails | RPF does not point to the tunnel | Transit or last-hop router | `show ip rpf <SOURCE_IP>`<br>`show ip route <SOURCE_IP>` | Correct unicast route or add `ip mroute` override |
| Incoming interface is physical instead of tunnel | Underlay route is being used for RPF | Transit or last-hop router | `show ip mroute <MULTICAST_GROUP>` | Change RPF with unicast route correction or static mroute |
| Multicast arrives on tunnel but is dropped | RPF check rejects packet | Transit or last-hop router | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Make RPF interface match tunnel path |
| `(*,G)` exists but no `(S,G)` traffic forwards | Receiver joined but source traffic is not accepted or not reaching RP/last-hop path | RP, transit, and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | Verify source, RP mapping, PIM tunnel adjacency, and RPF path |
| OIL is empty | No downstream receiver interest or downstream PIM state missing | Transit router | `show ip mroute <MULTICAST_GROUP>`<br>`show ip igmp groups` | Fix IGMP receiver join or downstream PIM |
| RP mapping missing | Static RP not configured consistently | All multicast routers | `show ip pim rp mapping` | Configure `ip pim rp-address <RP_ADDRESS>` consistently |
| Wrong RP selected | Overlapping RP group ranges or inconsistent RP ACLs | All multicast routers | `show ip pim rp mapping` | Correct RP group ACLs and apply consistently |
| Static mroute has no effect | Wrong source prefix, wrong next hop, or wrong interface used | Affected router | `show running-config | include ^ip mroute`<br>`show ip rpf <SOURCE_IP>` | Correct the `ip mroute` source prefix and next hop/interface |
| Multicast ping gives multiple replies | Ping is sourced from multiple multicast-enabled interfaces | Source router | `show ip pim interface` | Use a specific source interface/IP in the ping command |
| Multicast works until routes reconverge | RPF follows changing unicast route | Transit or last-hop router | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>` | Stabilize routing or use a deliberate static mroute |
##### Source_Basis
# Multicast_RPF_Tunnel_And_Overlay_Failure_Mental_Model
# Multicast_RPF_Tunnel_And_Overlay_Failure_Configuration_Checklist
# Multicast_RPF_Tunnel_And_Overlay_Failure_Skeleton
# Multicast_RPF_Tunnel_And_Overlay_Failure_Verification_Commands
# Multicast_RPF_Tunnel_And_Overlay_Failure_Rollback
# Multicast_RPF_Tunnel_And_Overlay_Failure_Failure_Checks

