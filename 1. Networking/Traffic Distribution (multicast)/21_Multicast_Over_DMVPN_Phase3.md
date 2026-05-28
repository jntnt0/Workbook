
Multicast_Over_DMVPN_Phase3.md

Multicast_Over_DMVPN_Phase3

# Multicast_Over_DMVPN_Phase3_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DMVPN Phase 3 | Hub and spokes use mGRE plus NHRP. Hub sends NHRP redirects. Spokes use NHRP shortcuts to resolve better spoke-to-spoke unicast paths |
| NHRP registration | Spokes register tunnel-to-NBMA mappings with the hub |
| NHRP redirect | Hub tells a spoke that a better direct path may exist to another spoke |
| NHRP shortcut | Spoke accepts redirects and installs shortcut forwarding information after NHRP resolution |
| Multicast over DMVPN | Multicast control traffic does not magically work because the tunnel is up. NHRP multicast mapping is required |
| Hub multicast map | `ip nhrp map multicast dynamic` lets the hub replicate multicast/broadcast control traffic to dynamically registered spokes |
| Spoke multicast map | `ip nhrp map multicast <HUB_NBMA>` lets a spoke send multicast/broadcast control traffic to the hub NBMA address |
| PIM over tunnel | PIM neighbors can form across the DMVPN tunnel after PIM is enabled on the tunnel and NHRP multicast mapping is correct |
| Routing protocol impact | EIGRP, OSPF, and PIM all depend on multicast or multicast-like control behavior over the tunnel |
| Multicast data reality | Phase 3 shortcuts optimize unicast spoke-to-spoke forwarding. Multicast replication and PIM tree behavior still depend on PIM, RP, RPF, and mroute state |
| RPF over tunnel | Multicast is accepted only if it arrives on the interface the router uses to reach the source or RP |
| RP placement | If using PIM sparse mode, the RP must be reachable through the DMVPN overlay and consistently mapped on all routers |
| IGMP role | Receiver-side spokes learn local group interest from IGMP and send PIM joins upstream |
| Mroute truth | `show ip mroute` proves incoming interface, RPF neighbor, outgoing interface list, and whether multicast is actually forwarding |
| DMVPN truth | `show dmvpn`, `show ip nhrp`, and `show adjacency tunnel <ID> detail` prove NHRP registration and shortcut behavior |
# Multicast_Over_DMVPN_Phase3_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify hub, spokes, tunnel subnet, NBMA addresses, source segment, receiver segment, RP address, and multicast group | All routers | `show ip interface brief` | Hub/spoke roles, tunnel interfaces, NBMA addresses, source, receiver, and RP are known |
| 2 | Confirm NBMA underlay reachability to the hub | Spokes | `ping <HUB_NBMA_IP>` | Spokes can reach the hub NBMA address |
| 3 | Confirm NBMA underlay reachability from hub to spokes | Hub | `ping <SPOKE_NBMA_IP>` | Hub can reach each spoke NBMA address |
| 4 | Configure hub mGRE tunnel interface | Hub | `conf t`<br>`interface <TUNNEL_INTERFACE>`<br>`ip address <HUB_TUNNEL_IP> <TUNNEL_MASK>`<br>`tunnel source <HUB_NBMA_INTERFACE_OR_IP>`<br>`tunnel mode gre multipoint`<br>`ip nhrp network-id <NHRP_NETWORK_ID>` | Hub tunnel is configured as multipoint GRE |
| 5 | Enable dynamic multicast mapping on the hub | Hub | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp map multicast dynamic` | Hub can replicate multicast control packets to dynamically registered spokes |
| 6 | Enable Phase 3 redirect on the hub | Hub | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp redirect` | Hub can send NHRP redirect messages to spokes |
| 7 | Configure spoke mGRE tunnel interface | Spokes | `conf t`<br>`interface <TUNNEL_INTERFACE>`<br>`ip address <SPOKE_TUNNEL_IP> <TUNNEL_MASK>`<br>`tunnel source <SPOKE_NBMA_INTERFACE_OR_IP>`<br>`tunnel mode gre multipoint`<br>`ip nhrp network-id <NHRP_NETWORK_ID>` | Spoke tunnel is configured as multipoint GRE |
| 8 | Configure spoke NHS | Spokes | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp nhs <HUB_TUNNEL_IP>` | Spoke registers with the hub as NHRP server |
| 9 | Configure spoke static hub tunnel-to-NBMA map | Spokes | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp map <HUB_TUNNEL_IP> <HUB_NBMA_IP>` | Spoke can resolve the hub tunnel address to hub NBMA address |
| 10 | Configure spoke multicast map toward hub NBMA | Spokes | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp map multicast <HUB_NBMA_IP>` | Spoke can send multicast control packets toward the hub |
| 11 | Enable Phase 3 shortcut on spokes | Spokes | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp shortcut` | Spokes can install shortcut forwarding after NHRP resolution |
| 12 | Optionally lower NHRP registration timer for lab convergence | Spokes | `interface <TUNNEL_INTERFACE>`<br>`ip nhrp registration timeout <SECONDS>` | Spokes refresh NHRP registration more quickly during lab testing |
| 13 | Verify DMVPN tunnel state | Hub and spokes | `show dmvpn` | Hub shows dynamic spoke entries. Spokes show hub entry |
| 14 | Verify NHRP cache | Hub and spokes | `show ip nhrp` | Hub has dynamic spoke registrations. Spokes have static hub mapping and learned shortcuts after traffic |
| 15 | Verify tunnel reachability | Hub and spokes | `ping <REMOTE_TUNNEL_IP>` | Tunnel endpoints are reachable |
| 16 | Configure overlay routing across the tunnel | Hub and spokes | `router <IGP_PROCESS>`<br>`network <TUNNEL_NETWORK_OR_INTERFACE>` | Routers exchange overlay reachability |
| 17 | Disable split horizon on hub tunnel if using EIGRP spoke-to-spoke route advertisement | Hub | `interface <TUNNEL_INTERFACE>`<br>`no ip split-horizon eigrp <ASN>` | Hub can advertise spoke routes back out the tunnel |
| 18 | Optionally summarize or inject default route from hub for Phase 3 scaling | Hub | `interface <TUNNEL_INTERFACE>`<br>`ip summary-address eigrp <ASN> 0.0.0.0 0.0.0.0` | Spokes can use hub-learned summary/default while Phase 3 shortcuts still allow direct spoke resolution |
| 19 | Verify overlay routes to source, receiver, and RP | All routers | `show ip route <SOURCE_IP>`<br>`show ip route <RECEIVER_PREFIX>`<br>`show ip route <RP_ADDRESS>` | Overlay routes exist through tunnel or shortcut-capable next hop |
| 20 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Routers can build multicast control-plane and forwarding state |
| 21 | Enable PIM sparse mode on DMVPN tunnel | Hub and spokes | `interface <TUNNEL_INTERFACE>`<br>`ip pim sparse-mode` | PIM can operate across the DMVPN overlay |
| 22 | Enable PIM sparse mode on source-facing interface | Source-side spoke/router | `interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-side interface participates in PIM |
| 23 | Enable PIM sparse mode on receiver-facing interface | Receiver-side spoke/router | `interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-side interface can process IGMP joins and PIM state |
| 24 | Configure static RP mapping for the multicast group | All multicast routers | `ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]` | All routers map the test group to the same RP |
| 25 | Verify RP mapping | All multicast routers | `show ip pim rp mapping` | Test group maps to `<RP_ADDRESS>` |
| 26 | Verify PIM neighbors over DMVPN | Hub and spokes | `show ip pim neighbor` | Spokes form PIM neighbor relationship with hub across the tunnel |
| 27 | Verify PIM interface state | Hub and spokes | `show ip pim interface` | Tunnel and relevant LAN interfaces appear |
| 28 | Configure receiver group membership | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the multicast group |
| 29 | Verify receiver membership on receiver-side spoke | Receiver-side spoke | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| 30 | Verify shared-tree state toward RP | Receiver-side spoke and hub/RP | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` state exists toward the RP |
| 31 | Verify RPF toward the RP over DMVPN | Receiver-side spoke | `show ip rpf <RP_ADDRESS>` | RPF path points toward the tunnel/expected RP path |
| 32 | Generate multicast traffic from source-side spoke | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends multicast traffic into the multicast domain |
| 33 | Verify source-specific multicast state | Source-side spoke, hub/RP, receiver-side spoke | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears after source traffic starts |
| 34 | Verify RPF toward source | Hub/RP and receiver-side spoke | `show ip rpf <SOURCE_IP>` | RPF path points toward the expected overlay path to the source |
| 35 | Verify OIL across DMVPN and receiver segment | Hub/RP and receiver-side spoke | `show ip mroute <MULTICAST_GROUP>` | OIL includes tunnel or receiver-facing interface where expected |
| 36 | Verify packet counters increment | All forwarding routers | `show ip mroute <MULTICAST_GROUP>` | Counters increment during multicast traffic |
| 37 | Trigger spoke-to-spoke unicast shortcut separately | Spoke | `traceroute <REMOTE_SPOKE_PREFIX_OR_LOOPBACK> numeric` | Initial traffic may go through hub, then shortcut resolution may occur |
| 38 | Verify Phase 3 shortcut state after unicast traffic | Spokes | `show ip nhrp`<br>`show adjacency <TUNNEL_INTERFACE> detail` | Spoke learns direct NHRP entry or shortcut adjacency for remote spoke destination |
| 39 | Confirm multicast RPF did not break after shortcut learning | Receiver-side spoke | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Incoming interface still matches valid RPF path |
| 40 | Clear stale multicast state for clean retest | All multicast routers | `clear ip mroute *` | Old multicast state is removed |
| 41 | Retest receiver join and multicast traffic | Receiver and source routers | `show ip igmp groups`<br>`ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast state rebuilds and forwards correctly |
| 42 | Save the working DMVPN multicast configuration | All changed routers | `copy running-config startup-config` | DMVPN Phase 3 and multicast overlay baseline is preserved |
# Multicast_Over_DMVPN_Phase3_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <TUNNEL_INTERFACE>             = DMVPN tunnel interface, example Tunnel1
! <HUB_TUNNEL_IP>                = hub tunnel IP
! <SPOKE_TUNNEL_IP>              = spoke tunnel IP
! <TUNNEL_MASK>                  = tunnel subnet mask
! <TUNNEL_NETWORK_OR_INTERFACE>  = IGP network statement or interface reference
! <HUB_NBMA_IP>                  = hub public/transport/NBMA IP
! <SPOKE_NBMA_IP>                = spoke public/transport/NBMA IP
! <HUB_NBMA_INTERFACE_OR_IP>     = hub tunnel source
! <SPOKE_NBMA_INTERFACE_OR_IP>   = spoke tunnel source
! <NHRP_NETWORK_ID>              = NHRP network ID
! <SECONDS>                      = optional NHRP registration timeout
! <IGP_PROCESS>                  = overlay routing process
! <ASN>                          = EIGRP AS if EIGRP is used
! <SOURCE_IP>                    = multicast source IP
! <SOURCE_INTERFACE_OR_IP>       = source interface or IP used for multicast test traffic
! <SOURCE_FACING_INTERFACE>      = source-side LAN interface
! <RECEIVER_PREFIX>              = receiver-side subnet or loopback
! <RECEIVER_FACING_INTERFACE>    = receiver-side LAN interface
! <RECEIVER_INTERFACE>           = receiver/test router interface
! <MULTICAST_GROUP>              = multicast group
! <RP_ADDRESS>                   = RP address reachable through overlay
! <GROUP_ACL>                    = optional RP group ACL
! =========================================================
! HUB DMVPN PHASE 3 BASELINE
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  ip address <HUB_TUNNEL_IP> <TUNNEL_MASK>
  tunnel source <HUB_NBMA_INTERFACE_OR_IP>
  tunnel mode gre multipoint
  ip nhrp network-id <NHRP_NETWORK_ID>
  ip nhrp map multicast dynamic
  ip nhrp redirect
 exit
end
! =========================================================
! SPOKE DMVPN PHASE 3 BASELINE
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  ip address <SPOKE_TUNNEL_IP> <TUNNEL_MASK>
  tunnel source <SPOKE_NBMA_INTERFACE_OR_IP>
  tunnel mode gre multipoint
  ip nhrp network-id <NHRP_NETWORK_ID>
  ip nhrp nhs <HUB_TUNNEL_IP>
  ip nhrp map <HUB_TUNNEL_IP> <HUB_NBMA_IP>
  ip nhrp map multicast <HUB_NBMA_IP>
  ip nhrp shortcut
  ip nhrp registration timeout <SECONDS>
 exit
end
! =========================================================
! OVERLAY ROUTING
! Example only. Use your lab routing process.
! =========================================================
conf t
 router eigrp <ASN>
  network <TUNNEL_NETWORK_OR_INTERFACE>
 exit
end
! =========================================================
! EIGRP HUB TUNNEL SETTINGS
! Needed when the hub must advertise spoke routes back out
! the same mGRE tunnel.
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  no ip split-horizon eigrp <ASN>
  ip summary-address eigrp <ASN> 0.0.0.0 0.0.0.0
 exit
end
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on all multicast routers.
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! PIM OVER DMVPN TUNNEL
! Apply on hub and spokes.
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! SOURCE-SIDE SPOKE OR ROUTER
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! RECEIVER-SIDE SPOKE OR ROUTER
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! HUB / RP ROUTER
! =========================================================
conf t
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! TEST RECEIVER JOIN
! Use join-group when a router is emulating a receiver.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! TEST TRAFFIC
! =========================================================
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! DMVPN VERIFICATION
! =========================================================
show dmvpn
show ip nhrp
show adjacency <TUNNEL_INTERFACE> detail
traceroute <RECEIVER_PREFIX> numeric
! =========================================================
! MULTICAST VERIFICATION
! =========================================================
show ip pim interface
show ip pim neighbor
show ip pim rp mapping
show ip igmp groups
show ip rpf <RP_ADDRESS>
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
# Multicast_Over_DMVPN_Phase3_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability to hub | Spokes | `ping <HUB_NBMA_IP>` | Hub NBMA replies |
| Underlay reachability to spokes | Hub | `ping <SPOKE_NBMA_IP>` | Spoke NBMA replies |
| Tunnel interface state | Hub and spokes | `show ip interface brief | include Tunnel` | Tunnel interface is up/up |
| Hub NHRP dynamic registrations | Hub | `show ip nhrp` | Spokes appear as dynamic registered entries |
| Spoke NHRP hub mapping | Spokes | `show ip nhrp` | Hub tunnel-to-NBMA mapping appears as static |
| DMVPN summary | Hub and spokes | `show dmvpn` | Hub shows spokes up. Spokes show hub up |
| Hub multicast NHRP map | Hub | `show running-config interface <TUNNEL_INTERFACE>` | `ip nhrp map multicast dynamic` is present |
| Spoke multicast NHRP map | Spokes | `show running-config interface <TUNNEL_INTERFACE>` | `ip nhrp map multicast <HUB_NBMA_IP>` is present |
| Phase 3 hub redirect | Hub | `show running-config interface <TUNNEL_INTERFACE>` | `ip nhrp redirect` is present |
| Phase 3 spoke shortcut | Spokes | `show running-config interface <TUNNEL_INTERFACE>` | `ip nhrp shortcut` is present |
| Overlay route to source | Receiver-side spoke | `show ip route <SOURCE_IP>` | Route exists through the overlay |
| Overlay route to RP | All multicast routers | `show ip route <RP_ADDRESS>` | Route to RP exists |
| Shortcut learning | Spokes | `show ip nhrp`<br>`show adjacency <TUNNEL_INTERFACE> detail` | Direct NHRP/CEF shortcut appears after spoke-to-spoke unicast traffic |
| Multicast routing enabled | Hub and spokes | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM tunnel state | Hub and spokes | `show ip pim interface <TUNNEL_INTERFACE>` | Tunnel is PIM-enabled |
| PIM neighbors over DMVPN | Hub and spokes | `show ip pim neighbor` | Hub and spokes form PIM neighbors across the tunnel |
| RP mapping | All multicast routers | `show ip pim rp mapping` | Test group maps to expected RP |
| Receiver membership | Receiver-side spoke | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| RPF toward RP | Receiver-side spoke and transit routers | `show ip rpf <RP_ADDRESS>` | RPF path points toward expected overlay path to RP |
| RPF toward source | Hub/RP and receiver-side spoke | `show ip rpf <SOURCE_IP>` | RPF path points toward expected overlay path to source |
| Shared-tree state | Receiver-side spoke and hub/RP | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` state exists for receiver interest |
| Source-specific state | Source-side spoke, hub/RP, receiver-side spoke | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears after source traffic |
| Incoming interface | Multicast forwarding routers | `show ip mroute <MULTICAST_GROUP>` | Incoming Interface matches RPF path |
| Outgoing interface list | Hub/RP and receiver-side spoke | `show ip mroute <MULTICAST_GROUP>` | OIL includes tunnel or receiver-facing interface where expected |
| Packet counters | All forwarding routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies if using `ip igmp join-group` and multicast forwarding is correct |
# Multicast_Over_DMVPN_Phase3_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE STATIC RP MAPPING
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS>
 no ip pim rp-address <RP_ADDRESS> <GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM DMVPN TUNNEL AND LAN INTERFACES
! Only do this if no other multicast labs depend on them.
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
! REMOVE GLOBAL MULTICAST ROUTING
! Only do this if no other multicast features depend on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! REMOVE EIGRP HUB TUNNEL OPTIONS IF USED ONLY FOR THIS LAB
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  no ip summary-address eigrp <ASN> 0.0.0.0 0.0.0.0
  ip split-horizon eigrp <ASN>
 exit
end
! =========================================================
! REMOVE SPOKE PHASE 3 AND MULTICAST NHRP SETTINGS
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  no ip nhrp shortcut
  no ip nhrp map multicast <HUB_NBMA_IP>
  no ip nhrp map <HUB_TUNNEL_IP> <HUB_NBMA_IP>
  no ip nhrp nhs <HUB_TUNNEL_IP>
  no ip nhrp registration timeout <SECONDS>
 exit
end
! =========================================================
! REMOVE HUB PHASE 3 AND MULTICAST NHRP SETTINGS
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  no ip nhrp redirect
  no ip nhrp map multicast dynamic
 exit
end
! =========================================================
! REMOVE DMVPN TUNNEL BASELINE
! Only do this in a full lab cleanup.
! =========================================================
conf t
 interface <TUNNEL_INTERFACE>
  no ip nhrp network-id <NHRP_NETWORK_ID>
  no tunnel mode gre multipoint
  no tunnel source <HUB_NBMA_INTERFACE_OR_IP>
  no tunnel source <SPOKE_NBMA_INTERFACE_OR_IP>
  no ip address
 exit
end
! =========================================================
! CLEAR STATE AND DEBUGGING
! Command support varies by platform.
! =========================================================
clear ip nhrp *
clear ip mroute *
clear ip igmp group *
undebug all
# Multicast_Over_DMVPN_Phase3_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Tunnel is up but PIM neighbors do not form | NHRP multicast mapping is missing | Hub and spokes | `show running-config interface <TUNNEL_INTERFACE>`<br>`show ip pim neighbor` | Configure hub `ip nhrp map multicast dynamic` and spoke `ip nhrp map multicast <HUB_NBMA_IP>` |
| Spokes do not register with hub | NHS, static hub map, NHRP network ID, or underlay reachability is wrong | Spokes and hub | `show ip nhrp`<br>`show dmvpn` | Fix `ip nhrp nhs`, `ip nhrp map`, network ID, and NBMA reachability |
| Hub has no dynamic NHRP entries | Spokes are not registering | Hub | `show ip nhrp` | Fix spoke NHS/static hub mapping and underlay route |
| Phase 3 shortcut never appears | `ip nhrp redirect` missing on hub, `ip nhrp shortcut` missing on spokes, or no spoke-to-spoke traffic triggered shortcut | Hub and spokes | `show running-config interface <TUNNEL_INTERFACE>`<br>`show ip nhrp` | Configure redirect/shortcut and generate spoke-to-spoke unicast traffic |
| EIGRP or other multicast-based overlay protocol does not form | NHRP multicast mapping missing or split horizon blocks spoke route advertisement | Hub and spokes | `show ip eigrp neighbors`<br>`show running-config interface <TUNNEL_INTERFACE>` | Fix NHRP multicast mapping and disable split horizon on hub if required |
| PIM tunnel interface missing | PIM not enabled on the tunnel | Hub and spokes | `show ip pim interface` | Configure `ip pim sparse-mode` under tunnel interface |
| RP mapping missing | Static RP not configured consistently or RP unreachable | All multicast routers | `show ip pim rp mapping`<br>`show ip route <RP_ADDRESS>` | Configure consistent `ip pim rp-address` and fix overlay route to RP |
| Receiver group missing | Receiver did not join or receiver-facing interface lacks PIM/IGMP | Receiver-side spoke | `show ip igmp groups`<br>`show ip pim interface` | Configure `ip igmp join-group <GROUP>` and enable PIM on receiver-facing interface |
| `(*,G)` exists but no `(S,G)` state | Source traffic not reaching PIM domain, source-side PIM missing, or RP registration/RPF issue | Source-side spoke and RP | `show ip mroute <GROUP>`<br>`show ip rpf <SOURCE_IP>` | Generate source traffic and fix source-side PIM/RPF |
| Incoming interface is wrong | RPF route to source or RP points away from the DMVPN path | Hub/RP and receiver-side spoke | `show ip rpf <SOURCE_IP>`<br>`show ip rpf <RP_ADDRESS>`<br>`show ip mroute <GROUP>` | Correct overlay routing or use deliberate multicast RPF correction |
| OIL is empty on hub/RP | No downstream receiver join reached the hub/RP | Hub/RP | `show ip mroute <GROUP>`<br>`show ip igmp groups` | Fix receiver-side IGMP, PIM neighbor, and RP mapping |
| OIL is empty on receiver-side spoke | Receiver-facing interface has no IGMP interest | Receiver-side spoke | `show ip igmp groups`<br>`show ip mroute <GROUP>` | Restore receiver join |
| Multicast works before shortcut, fails after shortcut | RPF path or route changed after Phase 3 resolution | Receiver-side spoke | `show ip nhrp`<br>`show ip rpf <SOURCE_IP>`<br>`show ip mroute <GROUP>` | Stabilize overlay routing or add deliberate multicast RPF override |
| Source ping uses wrong source address | Receiver joined traffic from a source different from the actual source IP | Source/test router and receiver-side spoke | `show ip interface brief`<br>`show ip mroute <GROUP>` | Source multicast traffic from the expected source interface/IP |
| Multicast control groups flood unexpectedly | Sparse-dense or Auto-RP behavior is being used over DMVPN | All multicast routers | `show ip mroute 224.0.1.39`<br>`show ip mroute 224.0.1.40` | Use static RP/BSR or control Auto-RP scope deliberately |
| DMVPN route summarization breaks RPF | Summary/default route changes RPF in a way multicast does not expect | Spokes | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>` | Adjust overlay routing or configure static mroute for the multicast source |
| NBMA routes disappear after removing default route | Spokes lost underlay reachability to hub or other spokes | Spokes | `show ip route <HUB_NBMA_IP>`<br>`show ip route <SPOKE_NBMA_IP>` | Add specific static routes to required NBMA addresses |
| Packet counters stay at zero | Source not sending, receiver not joined, PIM missing, or RPF failure | All multicast routers | `show ip mroute <GROUP>`<br>`show ip pim neighbor`<br>`show ip igmp groups` | Fix source traffic, receiver join, PIM, or RPF |
| Clear command fails | Platform uses different clear syntax | Affected router | `clear ip nhrp ?`<br>`clear ip mroute ?` | Use platform-supported clear command or wait for state aging |
| Debug floods the console | NHRP, PIM, or multicast debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# Multicast_Over_DMVPN_Phase3_Mental_Model
# Multicast_Over_DMVPN_Phase3_Configuration_Checklist
# Multicast_Over_DMVPN_Phase3_Skeleton
# Multicast_Over_DMVPN_Phase3_Verification_Commands
# Multicast_Over_DMVPN_Phase3_Rollback
# Multicast_Over_DMVPN_Phase3_Failure_Checks