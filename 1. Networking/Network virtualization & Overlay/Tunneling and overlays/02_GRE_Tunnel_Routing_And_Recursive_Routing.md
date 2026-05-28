
GRE_Tunnel_Routing_And_Recursive_Routing.md
# GRE_Tunnel_Routing_And_Recursive_Routing
# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | ENARSI Chapter 19, DMVPN Tunnels, GRE Tunnels | Points GRE tunnel and recursive routing topics to the ENARSI GRE/DMVPN tunnel section |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Generic Routing Encapsulation Tunnels | Defines GRE as an overlay built on an underlay transport network |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | GRE Tunnel Configuration | Provides `interface Tunnel`, `tunnel source`, `tunnel destination`, `ip address`, `bandwidth`, `keepalive`, and `ip mtu` syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | GRE Sample Configuration | Shows underlay routing kept separate from overlay routing across Tunnel100 |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Problems with Overlay Networks, Recursive Routing Problems | Explains tunnel recursion when the remote tunnel destination is learned through the tunnel |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Recursive Routing Syslog Messages on R11 for GRE Tunnels | Supports verification of `%TUN-5-RECURDOWN` and midchain loop syslog symptoms |
# GRE_Tunnel_Routing_And_Recursive_Routing_Mental_Model
| Concept | Operational Meaning |
|---|---|
| GRE overlay routing | Routing protocols or static routes can use the tunnel interface to reach remote LAN prefixes |
| Underlay route | The route used to reach the remote tunnel destination address |
| Tunnel destination | The remote underlay IP address used as the outer IP packet destination |
| Recursive routing | The router tries to reach the tunnel destination through the tunnel it is trying to build |
| Correct dependency | Underlay reachability must exist before the GRE tunnel can work |
| Wrong dependency | Overlay routing must never become the best path to the tunnel destination |
| Transport network | Physical or provider-facing subnet used only to build the GRE tunnel |
| Overlay network | Tunnel subnet used for routing adjacency and remote LAN reachability |
| Safe routing design | Keep underlay prefixes in the underlay routing process and overlay prefixes in the overlay routing process |
| Dangerous routing design | Advertising transport prefixes into the overlay IGP can make the tunnel route to itself |
| Common trigger | EIGRP across the GRE tunnel learns the remote underlay prefix and beats OSPF/RIP/static design expectations |
| Failure symptom | Tunnel flaps, routing adjacency drops, and syslog may show recursive routing or midchain loop messages |
| Clean fix | Remove transport prefixes from the overlay routing process or force tunnel destination reachability through the underlay |
# GRE_Tunnel_Routing_And_Recursive_Routing_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify local tunnel source IP | R1 | `show ip interface brief` | Local underlay interface or loopback address is known |
| 2 | Identify remote tunnel destination IP | R1 | `show ip route <R2_UNDERLAY_IP>` | Remote tunnel destination has an underlay path before GRE is used |
| 3 | Verify reverse underlay reachability | R2 | `show ip route <R1_UNDERLAY_IP>` | Peer also has an underlay path back to R1 |
| 4 | Test underlay reachability from R1 | R1 | `ping <R2_UNDERLAY_IP> source <R1_UNDERLAY_IP_OR_INTERFACE>` | Ping succeeds without using Tunnel100 |
| 5 | Test underlay reachability from R2 | R2 | `ping <R1_UNDERLAY_IP> source <R2_UNDERLAY_IP_OR_INTERFACE>` | Ping succeeds without using Tunnel100 |
| 6 | Create or confirm a non-tunnel route to the remote tunnel destination | R1 | `ip route <R2_UNDERLAY_IP> 255.255.255.255 <R1_UNDERLAY_NEXT_HOP>` | R1 has a stable underlay route to R2 tunnel destination |
| 7 | Create or confirm a non-tunnel route to the remote tunnel destination | R2 | `ip route <R1_UNDERLAY_IP> 255.255.255.255 <R2_UNDERLAY_NEXT_HOP>` | R2 has a stable underlay route to R1 tunnel destination |
| 8 | Verify the tunnel destination route does not point to the tunnel | R1 | `show ip route <R2_UNDERLAY_IP>` | Outgoing interface is physical underlay, not Tunnel100 |
| 9 | Verify the tunnel destination route does not point to the tunnel | R2 | `show ip route <R1_UNDERLAY_IP>` | Outgoing interface is physical underlay, not Tunnel100 |
| 10 | Configure the GRE tunnel interface on R1 | R1 | `interface Tunnel100` | Tunnel interface exists |
| 11 | Assign R1 overlay IP | R1 | `ip address <R1_TUNNEL_IP> <TUNNEL_MASK>` | R1 has an overlay tunnel address |
| 12 | Set R1 tunnel source | R1 | `tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>` | Tunnel uses the correct local underlay source |
| 13 | Set R1 tunnel destination | R1 | `tunnel destination <R2_UNDERLAY_IP>` | Tunnel points to the remote underlay address |
| 14 | Configure GRE operational values on R1 | R1 | `bandwidth <KBPS>` then `ip mtu 1400` then `keepalive 5 3` | Tunnel has bandwidth reference, lower MTU, and optional keepalive |
| 15 | Configure the GRE tunnel interface on R2 | R2 | `interface Tunnel100` | Peer tunnel interface exists |
| 16 | Assign R2 overlay IP | R2 | `ip address <R2_TUNNEL_IP> <TUNNEL_MASK>` | R2 has an overlay tunnel address |
| 17 | Set R2 tunnel source | R2 | `tunnel source <R2_UNDERLAY_INTERFACE_OR_IP>` | Tunnel uses the correct local underlay source |
| 18 | Set R2 tunnel destination | R2 | `tunnel destination <R1_UNDERLAY_IP>` | Tunnel points to the remote underlay address |
| 19 | Configure GRE operational values on R2 | R2 | `bandwidth <KBPS>` then `ip mtu 1400` then `keepalive 5 3` | Peer tunnel has matching operational values |
| 20 | Verify tunnel state before adding overlay routing | R1/R2 | `show interface Tunnel100` | Tunnel is up/up and shows GRE/IP transport |
| 21 | Verify overlay tunnel endpoint reachability | R1 | `ping <R2_TUNNEL_IP> source <R1_TUNNEL_IP>` | Overlay endpoint ping succeeds |
| 22 | Verify reverse overlay endpoint reachability | R2 | `ping <R1_TUNNEL_IP> source <R2_TUNNEL_IP>` | Reverse overlay endpoint ping succeeds |
| 23 | Configure overlay routing only for tunnel and LAN prefixes | R1/R2 | `router eigrp GRE-OVERLAY` then `address-family ipv4 unicast autonomous-system 100` then `network <TUNNEL_NETWORK>` then `network <LOCAL_LAN_NETWORK>` | Overlay routing forms across the tunnel and advertises LAN prefixes |
| 24 | Keep transport prefixes out of the overlay routing process | R1/R2 | Do not configure `network <UNDERLAY_NETWORK>` under the overlay IGP | Tunnel destination is not advertised through the tunnel |
| 25 | If redistribution is used, filter underlay prefixes from overlay advertisements | R1/R2 | `distribute-list prefix BLOCK-UNDERLAY out Tunnel100` | Underlay tunnel endpoint prefixes are not sent over the tunnel |
| 26 | Verify overlay routing neighbor | R1/R2 | `show ip eigrp neighbors` | Neighbor forms across Tunnel100 |
| 27 | Verify remote LAN route uses the tunnel | R1/R2 | `show ip route <REMOTE_LAN_PREFIX>` | Remote LAN resolves through Tunnel100 or remote tunnel IP |
| 28 | Verify tunnel destination still uses underlay after overlay routing forms | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Remote tunnel destination still resolves through physical underlay |
| 29 | Check logs for recursion symptoms | R1/R2 | `show logging | include RECURDOWN|Midchain|looped|Tunnel100` | No recursive routing or midchain loop messages appear |
| 30 | Save working state | R1/R2 | `write memory` | Stable non-recursive GRE routing design is saved |
# GRE_Tunnel_Routing_And_Recursive_Routing_Skeleton
! =========================================================
! DESIGN RULE
! =========================================================
! Underlay routes reach tunnel destinations.
! Overlay routes reach tunnel endpoints and remote LANs.
! Never let the tunnel destination be learned through the tunnel.
! =========================================================
! R1 UNDERLAY PROTECTION
! =========================================================
conf t
!
! Option A: Static host route to remote tunnel destination
ip route <R2_UNDERLAY_IP> 255.255.255.255 <R1_UNDERLAY_NEXT_HOP>
!
! Option B: Underlay IGP only on transport links
router ospf 10
 router-id <R1_ROUTER_ID>
 network <R1_UNDERLAY_NETWORK> <WILDCARD> area 0
 passive-interface default
 no passive-interface <R1_UNDERLAY_INTERFACE>
!
end
! =========================================================
! R2 UNDERLAY PROTECTION
! =========================================================
conf t
!
! Option A: Static host route to remote tunnel destination
ip route <R1_UNDERLAY_IP> 255.255.255.255 <R2_UNDERLAY_NEXT_HOP>
!
! Option B: Underlay IGP only on transport links
router ospf 10
 router-id <R2_ROUTER_ID>
 network <R2_UNDERLAY_NETWORK> <WILDCARD> area 0
 passive-interface default
 no passive-interface <R2_UNDERLAY_INTERFACE>
!
end
! =========================================================
! R1 GRE TUNNEL
! =========================================================
conf t
!
interface Tunnel100
 description GRE_TO_R2_OVERLAY
 bandwidth 4000
 ip address <R1_TUNNEL_IP> <TUNNEL_MASK>
 ip mtu 1400
 keepalive 5 3
 tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R2_UNDERLAY_IP>
 no shutdown
!
end
! =========================================================
! R2 GRE TUNNEL
! =========================================================
conf t
!
interface Tunnel100
 description GRE_TO_R1_OVERLAY
 bandwidth 4000
 ip address <R2_TUNNEL_IP> <TUNNEL_MASK>
 ip mtu 1400
 keepalive 5 3
 tunnel source <R2_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R1_UNDERLAY_IP>
 no shutdown
!
end
! =========================================================
! R1 OVERLAY ROUTING EXAMPLE
! Do not include the underlay/transport network here.
! =========================================================
conf t
!
router eigrp GRE-OVERLAY
 address-family ipv4 unicast autonomous-system 100
  network <R1_LAN_NETWORK>
  network <TUNNEL_NETWORK>
 exit-address-family
!
end
! =========================================================
! R2 OVERLAY ROUTING EXAMPLE
! Do not include the underlay/transport network here.
! =========================================================
conf t
!
router eigrp GRE-OVERLAY
 address-family ipv4 unicast autonomous-system 100
  network <R2_LAN_NETWORK>
  network <TUNNEL_NETWORK>
 exit-address-family
!
end
! =========================================================
! OPTIONAL OVERLAY PREFIX FILTER
! Use this when redistribution or broad network statements may leak underlay prefixes.
! =========================================================
conf t
!
ip prefix-list BLOCK-UNDERLAY seq 5 deny <UNDERLAY_SUMMARY> le 32
ip prefix-list BLOCK-UNDERLAY seq 100 permit 0.0.0.0/0 le 32
!
router eigrp GRE-OVERLAY
 address-family ipv4 unicast autonomous-system 100
  af-interface Tunnel100
  exit-af-interface
  topology base
   distribute-list prefix BLOCK-UNDERLAY out Tunnel100
  exit-af-topology
 exit-address-family
!
end
write memory
# GRE_Tunnel_Routing_And_Recursive_Routing_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Confirm underlay interface state | R1/R2 | `show ip interface brief` | Physical underlay interface is up/up |
| Confirm route to remote tunnel destination | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Route exits a physical underlay interface |
| Confirm route does not recurse through tunnel | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Output does not show Tunnel100 |
| Confirm tunnel source and destination | R1/R2 | `show interface Tunnel100` | Source and destination match the design |
| Confirm tunnel line protocol | R1/R2 | `show interface Tunnel100` | Tunnel is up/up |
| Confirm GRE transport | R1/R2 | `show interface Tunnel100` | Output shows GRE/IP tunnel transport |
| Confirm overlay endpoint ping | R1 | `ping <R2_TUNNEL_IP> source <R1_TUNNEL_IP>` | Ping succeeds |
| Confirm reverse overlay endpoint ping | R2 | `ping <R1_TUNNEL_IP> source <R2_TUNNEL_IP>` | Ping succeeds |
| Confirm overlay routing neighbor | R1/R2 | `show ip eigrp neighbors` | Neighbor is established over Tunnel100 |
| Confirm remote LAN route | R1/R2 | `show ip route <REMOTE_LAN_PREFIX>` | Remote LAN uses Tunnel100 or remote tunnel IP |
| Confirm tunnel destination route after overlay forms | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Underlay route is still preferred |
| Confirm underlay prefixes are not learned through overlay | R1/R2 | `show ip route eigrp | include <UNDERLAY_PREFIX>` | No remote tunnel destination route is learned from overlay EIGRP |
| Confirm no recursive routing logs | R1/R2 | `show logging | include RECURDOWN|Midchain|looped|Tunnel100` | No recursion messages appear |
| Confirm forwarding to remote LAN | R1/R2 | `traceroute <REMOTE_LAN_HOST> source <LOCAL_LAN_IP>` | Traffic uses the GRE overlay for remote LAN reachability |
| Confirm forwarding to tunnel destination | R1/R2 | `traceroute <REMOTE_UNDERLAY_IP> source <LOCAL_UNDERLAY_IP>` | Traffic follows the physical underlay path |
# GRE_Tunnel_Routing_And_Recursive_Routing_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove overlay route filtering if it was added only for testing | R1/R2 | `router eigrp GRE-OVERLAY` then `address-family ipv4 unicast autonomous-system 100` then `topology base` then `no distribute-list prefix BLOCK-UNDERLAY out Tunnel100` | Overlay route filter is removed |
| 2 | Remove overlay routing process if rolling back GRE routing entirely | R1/R2 | `no router eigrp GRE-OVERLAY` | Overlay EIGRP process is removed |
| 3 | Remove static remote LAN routes if used | R1/R2 | `no ip route <REMOTE_LAN_PREFIX> <MASK> <REMOTE_TUNNEL_IP>` | Remote LAN no longer routes through GRE |
| 4 | Shut the tunnel interface | R1/R2 | `interface Tunnel100` then `shutdown` | Tunnel stops forwarding |
| 5 | Remove the GRE tunnel interface | R1/R2 | `no interface Tunnel100` | Tunnel100 is deleted |
| 6 | Remove static underlay host route only if it was added for this lab | R1/R2 | `no ip route <REMOTE_UNDERLAY_IP> 255.255.255.255 <UNDERLAY_NEXT_HOP>` | Static tunnel destination protection route is removed |
| 7 | Remove prefix-list if unused | R1/R2 | `no ip prefix-list BLOCK-UNDERLAY` | Prefix-list cleanup is complete |
| 8 | Verify tunnel is gone | R1/R2 | `show ip interface brief | include Tunnel100` | No Tunnel100 interface appears |
| 9 | Verify remaining underlay routing state | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Underlay routing returns to intended baseline |
| 10 | Save rollback state | R1/R2 | `write memory` | Rollback persists after reload |
# GRE_Tunnel_Routing_And_Recursive_Routing_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Tunnel repeatedly goes up/down | Recursive routing is occurring | R1/R2 | `show logging | include RECURDOWN|Midchain|looped` | Remove tunnel destination prefix from overlay routing |
| `%TUN-5-RECURDOWN` appears | Point-to-point GRE tunnel destination is learned through the tunnel | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Force tunnel destination route through underlay |
| `Midchain parent maintenance` message appears | Encapsulation path is trying to stack back onto the tunnel | R1/R2 | `show logging | include Midchain` | Fix recursive route to tunnel destination |
| EIGRP neighbor flaps over Tunnel100 | Tunnel drops because destination route changes | R1/R2 | `show ip eigrp neighbors` and `show logging` | Remove underlay network from overlay EIGRP |
| Remote LAN route works briefly then fails | Overlay IGP is destabilizing the tunnel endpoint route | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Keep transport prefixes out of overlay IGP |
| Tunnel destination route points to Tunnel100 | Underlay prefix leaked into overlay | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Add static host route or filter underlay prefix |

| Tunnel is down/down or line protocol down | No route to tunnel destination | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Restore underlay route before troubleshooting overlay |
| Overlay neighbor never forms | Tunnel endpoint ping fails or IGP not enabled on tunnel | R1/R2 | `ping <REMOTE_TUNNEL_IP> source <LOCAL_TUNNEL_IP>` | Fix GRE tunnel first, then enable overlay IGP |
| Remote LAN prefix is missing | LAN network not advertised into overlay IGP | R1/R2 | `show run | section router eigrp` | Add LAN network to overlay routing process |
| Transport subnet appears in overlay route table | Bad network statement or redistribution | R1/R2 | `show ip route eigrp | include <UNDERLAY_PREFIX>` | Remove broad network statement or apply prefix filter |
| Static protection route missing after reload | Config was not saved | R1/R2 | `show running-config | include ip route <REMOTE_UNDERLAY_IP>` | Reapply route and run `write memory` |
| Route to tunnel destination uses default route unexpectedly | Underlay design relies only on generic default routing | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Add explicit host route to tunnel destination for lab stability |
##### Source_Basis
# GRE_Tunnel_Routing_And_Recursive_Routing_Mental_Model
# GRE_Tunnel_Routing_And_Recursive_Routing_Configuration_Checklist
# GRE_Tunnel_Routing_And_Recursive_Routing_Skeleton
# GRE_Tunnel_Routing_And_Recursive_Routing_Verification_Commands
# GRE_Tunnel_Routing_And_Recursive_Routing_Rollback
# GRE_Tunnel_Routing_And_Recursive_Routing_Failure_Checks
# index of each title throughout note, not in table format