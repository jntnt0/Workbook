

IGMP_Proxy_Receiver_Delegation.md

IGMP_Proxy_Receiver_Delegation

Cisco’s IOS XE IGMP Proxy documentation is used here only because the project source identifies multicast/IGMP broadly but does not expose the full IGMP proxy command sequence. Cisco defines IGMP proxy as a way for hosts not directly connected to a downstream router to join multicast from an upstream network, and documents ip igmp mroute-proxy, ip igmp helper-address, ip igmp proxy-service, ip igmp unidirectional-link, show ip igmp interface, and show ip igmp udlr for this workflow.  

# Source_Basis
# IGMP_Proxy_Receiver_Delegation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IGMP proxy | A downstream router converts downstream multicast receiver interest into proxied IGMP reports toward an upstream multicast domain |
| Receiver delegation | Receivers are not directly attached to the upstream multicast router, so another router represents their group interest |
| Not normal IGMP only | Normal IGMP works between a host and its directly connected router. IGMP proxy is used when that local membership must be represented upstream |
| Not full PIM adjacency | IGMP proxy can bridge receiver interest across a boundary where the upstream and downstream sides are not simply one continuous PIM domain |
| UDLR use case | In a unidirectional link routing design, the downstream router may need to proxy IGMP reports upstream so the upstream device forwards multicast down the one-way link |
| Non-UDLR use case | IGMP proxy can also be used to move receiver interest across a one-hop boundary, such as a GRE tunnel between separate PIM domains |
| Proxy service interface | The proxy service interface is where the router creates and sends the proxied IGMP report |
| Mroute proxy interface | The downstream-facing interface uses `ip igmp mroute-proxy` to tie matching multicast forwarding entries to the proxy service interface |
| Helper address | `ip igmp helper-address` tells the proxy service interface where to send the proxied IGMP report |
| UDL helper address | `ip igmp helper-address udl <interface>` points proxy reports toward an upstream UDL interface |
| One-hop boundary | IGMP proxy joins are not a multi-hop replacement for PIM. Treat the proxy join boundary as one hop |
| Verification truth | `show ip igmp interface`, `show ip igmp udlr`, `show ip igmp groups`, and `show ip mroute` prove whether the proxy relationship is working |
# IGMP_Proxy_Receiver_Delegation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the upstream multicast-facing interface | Downstream proxy router | `show ip interface brief` | Interface toward the upstream multicast domain is known |
| 2 | Identify the downstream receiver-facing interface | Downstream proxy router | `show ip interface brief` | Interface toward receiver-side multicast state is known |
| 3 | Identify the proxy service interface | Downstream proxy router | `show ip interface brief` | Loopback or selected proxy interface is known |
| 4 | Confirm multicast routing is enabled globally | Downstream proxy router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| 5 | Enable multicast routing globally if missing | Downstream proxy router | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 6 | Confirm PIM is active on the receiver-side interface | Downstream proxy router | `show ip pim interface <RECEIVER_SIDE_INTERFACE>` | Receiver-side interface appears as a PIM interface |
| 7 | Enable PIM on the receiver-side interface if missing | Downstream proxy router | `conf t`<br>`interface <RECEIVER_SIDE_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-side interface can participate in multicast state |
| 8 | Confirm PIM is active on the proxy service interface | Downstream proxy router | `show ip pim interface <PROXY_SERVICE_INTERFACE>` | Proxy service interface appears as a PIM interface |
| 9 | Enable PIM on the proxy service interface if missing | Downstream proxy router | `conf t`<br>`interface <PROXY_SERVICE_INTERFACE>`<br>`ip pim sparse-mode` | Proxy service interface can support IGMP proxy service |
| 10 | Configure static RP if the lab uses PIM sparse mode | Downstream proxy router | `conf t`<br>`ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]` | Router maps multicast groups to the expected RP |
| 11 | Verify RP mapping | Downstream proxy router | `show ip pim rp mapping` | Expected RP appears for the multicast group range |
| 12 | Configure UDL behavior on the upstream-facing UDL interface if this is a UDLR lab | Downstream proxy router | `conf t`<br>`interface <UPSTREAM_UDL_INTERFACE>`<br>`ip igmp unidirectional-link` | Interface is marked for IGMP UDLR behavior |
| 13 | Configure IGMP mroute proxy on the interface facing nondirectly connected receivers | Downstream proxy router | `conf t`<br>`interface <RECEIVER_SIDE_INTERFACE>`<br>`ip igmp mroute-proxy <PROXY_SERVICE_INTERFACE>` | Router forwards matching proxied `(*,G)` reports to the proxy service interface |
| 14 | Configure UDL IGMP helpering on the proxy service interface for UDLR mode | Downstream proxy router | `conf t`<br>`interface <PROXY_SERVICE_INTERFACE>`<br>`ip igmp helper-address udl <UPSTREAM_UDL_INTERFACE>` | Proxy service interface sends IGMP reports toward the upstream UDL interface |
| 15 | Configure non-UDLR IGMP helpering on the proxy service interface if the lab uses a one-hop upstream neighbor instead | Downstream proxy router | `conf t`<br>`interface <PROXY_SERVICE_INTERFACE>`<br>`ip igmp helper-address <UPSTREAM_PROXY_TARGET_IP>` | Proxy service interface sends IGMP proxy joins toward the upstream target |
| 16 | Enable the IGMP proxy service | Downstream proxy router | `conf t`<br>`interface <PROXY_SERVICE_INTERFACE>`<br>`ip igmp proxy-service` | Router begins creating proxied IGMP reports for matching mroute entries |
| 17 | Verify proxy interface status | Downstream proxy router | `show ip igmp interface <PROXY_SERVICE_INTERFACE>` | Proxy service interface shows IGMP state and proxy-related configuration |
| 18 | Verify UDLR status if using UDLR | Downstream proxy router | `show ip igmp udlr` | UDLR helper/interface information appears |
| 19 | Create or confirm receiver-side multicast interest | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver-side multicast membership is created |
| 20 | Verify receiver group membership | Downstream proxy router | `show ip igmp groups` | Receiver group appears on the downstream-facing interface |
| 21 | Verify multicast route state exists for the group | Downstream proxy router | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` and/or `(<SOURCE>,<GROUP>)` state exists |
| 22 | Verify the proxied state is tied to the receiver-side interface | Downstream proxy router | `show ip mroute <MULTICAST_GROUP>` | Receiver-side interface appears where expected in multicast state |
| 23 | Verify upstream device sees receiver interest | Upstream multicast router | `show ip igmp groups`<br>`show ip mroute <MULTICAST_GROUP>` | Upstream device has group state created from the proxied IGMP report |
| 24 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast traffic is generated toward the group |
| 25 | Verify multicast packet counters increment | Upstream and downstream routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during test traffic |
| 26 | Save the working IGMP proxy configuration | Changed routers | `copy running-config startup-config` | IGMP proxy receiver delegation configuration is preserved |
# IGMP_Proxy_Receiver_Delegation_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <UPSTREAM_UDL_INTERFACE>     = interface toward upstream UDL multicast source side
! <RECEIVER_SIDE_INTERFACE>    = interface facing downstream receiver-side multicast state
! <PROXY_SERVICE_INTERFACE>    = loopback or interface used for IGMP proxy service
! <UPSTREAM_PROXY_TARGET_IP>   = upstream one-hop IGMP proxy target for non-UDLR mode
! <RECEIVER_INTERFACE>         = test receiver interface
! <MULTICAST_GROUP>            = multicast group being joined
! <SOURCE_INTERFACE_OR_IP>     = multicast source interface or source IP
! <RP_ADDRESS>                 = RP address if sparse mode is used
! <GROUP_ACL>                  = optional RP group ACL
! =========================================================
! GLOBAL MULTICAST BASELINE
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! DOWNSTREAM PROXY ROUTER BASELINE
! =========================================================
conf t
 interface <RECEIVER_SIDE_INTERFACE>
  ip pim sparse-mode
 exit
 interface <PROXY_SERVICE_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! UDLR MODE
! Mark the upstream UDL interface and proxy IGMP reports
! through the proxy service interface.
! =========================================================
conf t
 interface <UPSTREAM_UDL_INTERFACE>
  ip pim sparse-mode
  ip igmp unidirectional-link
 exit
 interface <RECEIVER_SIDE_INTERFACE>
  ip igmp mroute-proxy <PROXY_SERVICE_INTERFACE>
 exit
 interface <PROXY_SERVICE_INTERFACE>
  ip igmp helper-address udl <UPSTREAM_UDL_INTERFACE>
  ip igmp proxy-service
 exit
end
! =========================================================
! NON-UDLR MODE
! Use this when proxying across a one-hop upstream boundary,
! such as a GRE tunnel endpoint or directly connected upstream device.
! Do not configure this at the same time as the UDLR helper mode
! unless the lab explicitly calls for both cases.
! =========================================================
conf t
 interface <RECEIVER_SIDE_INTERFACE>
  ip igmp mroute-proxy <PROXY_SERVICE_INTERFACE>
 exit
 interface <PROXY_SERVICE_INTERFACE>
  ip igmp helper-address <UPSTREAM_PROXY_TARGET_IP>
  ip igmp proxy-service
 exit
end
! =========================================================
! TEST RECEIVER JOIN
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
! VERIFICATION
! =========================================================
show ip igmp interface
show ip igmp udlr
show ip igmp groups
show ip mroute <MULTICAST_GROUP>
show ip pim rp mapping
# IGMP_Proxy_Receiver_Delegation_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | Downstream proxy router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface status | Downstream proxy router | `show ip pim interface` | Receiver-side and proxy service interfaces appear |
| RP mapping | Downstream proxy router | `show ip pim rp mapping` | Expected RP is mapped to the multicast group range |
| Receiver-side mroute proxy config | Downstream proxy router | `show running-config interface <RECEIVER_SIDE_INTERFACE>` | `ip igmp mroute-proxy <PROXY_SERVICE_INTERFACE>` is present |
| UDL interface config | Downstream proxy router | `show running-config interface <UPSTREAM_UDL_INTERFACE>` | `ip igmp unidirectional-link` is present when using UDLR mode |
| Proxy service config | Downstream proxy router | `show running-config interface <PROXY_SERVICE_INTERFACE>` | `ip igmp proxy-service` and helper address are present |
| IGMP interface state | Downstream proxy router | `show ip igmp interface <PROXY_SERVICE_INTERFACE>` | Proxy service interface shows IGMP state |
| UDLR state | Downstream proxy router | `show ip igmp udlr` | UDLR helper information appears when UDLR mode is used |
| Receiver local join | Receiver/test router | `show ip igmp groups` | Joined multicast group appears locally |
| Downstream receiver membership | Downstream proxy router | `show ip igmp groups` | Receiver group appears on the receiver-side interface |
| Downstream mroute state | Downstream proxy router | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` state exists |
| Upstream group state | Upstream multicast router | `show ip igmp groups` | Upstream device sees group interest from the proxied report |
| Upstream multicast route state | Upstream multicast router | `show ip mroute <MULTICAST_GROUP>` | Upstream device has multicast state for the group |
| Packet counter movement | Upstream and downstream routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during multicast traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver responds if configured with `ip igmp join-group` and routing/RPF are correct |
# IGMP_Proxy_Receiver_Delegation_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE IGMP MROUTE PROXY FROM RECEIVER-SIDE INTERFACE
! =========================================================
conf t
 interface <RECEIVER_SIDE_INTERFACE>
  no ip igmp mroute-proxy <PROXY_SERVICE_INTERFACE>
 exit
end
! =========================================================
! REMOVE UDLR HELPER AND PROXY SERVICE
! =========================================================
conf t
 interface <PROXY_SERVICE_INTERFACE>
  no ip igmp helper-address udl <UPSTREAM_UDL_INTERFACE>
  no ip igmp helper-address <UPSTREAM_PROXY_TARGET_IP>
  no ip igmp proxy-service
 exit
end
! =========================================================
! REMOVE UDL MARKING FROM UPSTREAM UDL INTERFACE
! =========================================================
conf t
 interface <UPSTREAM_UDL_INTERFACE>
  no ip igmp unidirectional-link
 exit
end
! =========================================================
! REMOVE STATIC RP IF IT WAS ONLY USED FOR THIS LAB
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! REMOVE PIM FROM LAB INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <RECEIVER_SIDE_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <PROXY_SERVICE_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <UPSTREAM_UDL_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE GLOBAL MULTICAST ROUTING
! Only do this if no other multicast services depend on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR STATE
! =========================================================
clear ip igmp group *
clear ip mroute *
undebug all
# IGMP_Proxy_Receiver_Delegation_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Proxy commands are rejected | Platform or image does not support IGMP proxy | Downstream proxy router | `show version` | Use a supported IOS XE image/platform or move the lab to a router image that supports the feature |
| No receiver group appears downstream | Receiver did not join or wrong receiver-facing interface was used | Receiver and downstream proxy router | `show ip igmp groups` | Configure `ip igmp join-group <GROUP>` on the correct receiver interface |
| `ip igmp mroute-proxy` is present but no proxy report occurs | No matching `(*,G)` state exists in the multicast forwarding table | Downstream proxy router | `show ip mroute <MULTICAST_GROUP>` | Fix receiver join, RP mapping, PIM, or multicast routing state |
| Proxy service interface does not create reports | `ip igmp proxy-service` missing | Downstream proxy router | `show running-config interface <PROXY_SERVICE_INTERFACE>` | Configure `ip igmp proxy-service` |
| Proxy service configured but helper missing | No helper address is configured toward upstream | Downstream proxy router | `show running-config interface <PROXY_SERVICE_INTERFACE>` | Configure `ip igmp helper-address udl <INTERFACE>` or `ip igmp helper-address <UPSTREAM_IP>` |
| UDLR verification shows nothing useful | UDLR mode not configured or wrong interface marked | Downstream proxy router | `show ip igmp udlr`<br>`show running-config interface <UPSTREAM_UDL_INTERFACE>` | Configure `ip igmp unidirectional-link` on the correct UDL interface |
| Upstream router does not see group interest | Helper target is wrong, proxy report is not reaching upstream, or upstream interface is wrong | Upstream and downstream routers | `show ip igmp groups`<br>`show ip igmp interface` | Correct helper address/interface and confirm one-hop upstream reachability |
| Non-UDLR proxy fails across multiple hops | IGMP proxy join is being used beyond its intended one-hop boundary | Downstream proxy router | `show ip route <UPSTREAM_PROXY_TARGET_IP>` | Use a one-hop GRE/direct boundary or use normal PIM routing instead |
| Group state exists but traffic does not forward | RPF failure or upstream multicast path problem | Upstream and downstream routers | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Correct RPF path, source routing, RP mapping, or PIM adjacency |
| Receiver does not reply to multicast ping | Receiver used `ip igmp static-group` or host does not respond to multicast ICMP | Receiver/test router | `show running-config interface <RECEIVER_INTERFACE>` | Use `ip igmp join-group <GROUP>` for router-based ping testing |
| PIM neighbor missing where expected | PIM not enabled or the design intentionally separates PIM domains | Downstream proxy router | `show ip pim neighbor` | Enable PIM where needed, but do not assume IGMP proxy requires one continuous PIM domain |
| RP mapping missing | Static RP, BSR, or Auto-RP is not configured correctly | Downstream proxy router | `show ip pim rp mapping` | Configure consistent RP mapping for the group range |
| State remains after removing joins | IGMP or mroute state has not aged out | Downstream proxy router | `show ip igmp groups`<br>`show ip mroute <MULTICAST_GROUP>` | Run `clear ip igmp group *` and `clear ip mroute *` |
| Debug output overwhelms console | Debug left on during IGMP testing | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# IGMP_Proxy_Receiver_Delegation_Mental_Model
# IGMP_Proxy_Receiver_Delegation_Configuration_Checklist
# IGMP_Proxy_Receiver_Delegation_Skeleton
# IGMP_Proxy_Receiver_Delegation_Verification_Commands
# IGMP_Proxy_Receiver_Delegation_Rollback
# IGMP_Proxy_Receiver_Delegation_Failure_Checks

