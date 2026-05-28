
PIM_DR_Receiver_Segment_And_Last_Hop_Router.md

PIM_DR_Receiver_Segment_And_Last_Hop_Router

# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PIM DR | On a multiaccess segment with multiple PIM routers, one router is elected as the PIM Designated Router |
| Last-hop router | The router connected to the receiver LAN processes IGMP receiver interest and represents that receiver interest upstream |
| Receiver LAN DR | On a shared receiver segment, the PIM DR is responsible for sending PIM Join/Prune messages upstream when local receivers join or leave groups |
| Source LAN DR | On a shared source segment in PIM sparse mode, the PIM DR is responsible for registering active sources to the RP |
| ASM behavior | In Any Source Multicast, the receiver-side DR sends joins toward the RP for `(*,G)` shared-tree state |
| SSM behavior | In Source Specific Multicast, the receiver-side DR sends `(S,G)` joins toward the source |
| DR election | PIM DR election is based on PIM hello information. Higher DR priority wins. If priority ties, the higher IP address wins |
| Default priority | PIM DR priority defaults to 1 on IOS-style platforms |
| DR versus IGMP querier | PIM DR and IGMP querier are not the same job. PIM DR handles multicast routing responsibility. IGMP querier manages receiver membership queries |
| Important contrast | IGMPv2 querier election usually prefers the lowest IP address, while PIM DR tiebreak usually prefers the highest IP address |
| FHRP mismatch | HSRP/VRRP active gateway and PIM DR can be different routers unless deliberately aligned |
| Verification truth | `show ip pim interface` proves the elected PIM DR. `show ip igmp interface` also shows the multicast DR on the receiver-facing interface |
| Failure pattern | If the wrong router is DR, receiver joins, source registers, or FHRP-aligned multicast forwarding may not behave as expected |
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the shared receiver segment with multiple multicast routers | Receiver-segment routers | `show ip interface brief` | Routers connected to the same receiver LAN are known |
| 2 | Identify the intended PIM DR for the receiver segment | Receiver-segment routers | `show ip interface brief` | Intended DR interface and IP are known |
| 3 | Confirm receiver-facing interfaces are up | Receiver-segment routers | `show ip interface brief` | Receiver-facing interfaces are up/up |
| 4 | Confirm unicast reachability to the RP or source | Receiver-segment routers | `show ip route <RP_ADDRESS>`<br>`show ip route <SOURCE_IP>` | Routers have routes toward the RP for ASM or source for SSM |
| 5 | Enable multicast routing globally | Receiver-segment routers | `conf t`<br>`ip multicast-routing` | Routers can build multicast control-plane and forwarding state |
| 6 | Enable PIM sparse mode on receiver-facing interfaces | Receiver-segment routers | `conf t`<br>`interface <RECEIVER_LAN_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-facing interfaces participate in PIM and IGMP |
| 7 | Enable PIM sparse mode on upstream transit interfaces | Receiver-segment routers | `conf t`<br>`interface <UPSTREAM_INTERFACE>`<br>`ip pim sparse-mode` | Routers can send joins upstream toward RP or source |
| 8 | Configure RP mapping for ASM multicast | Receiver-segment routers | `conf t`<br>`ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]` | ASM group maps to the expected RP |
| 9 | Verify PIM interfaces | Receiver-segment routers | `show ip pim interface` | Receiver LAN and upstream interfaces appear |
| 10 | Verify PIM neighbors on the shared receiver LAN | Receiver-segment routers | `show ip pim neighbor` | Routers see each other as PIM neighbors on the receiver LAN |
| 11 | Check the current PIM DR election | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>` | DR field shows the currently elected PIM DR |
| 12 | Verify multicast DR from the IGMP interface view | Receiver-segment routers | `show ip igmp interface <RECEIVER_LAN_INTERFACE>` | Output shows `Multicast designated router (DR)` |
| 13 | Configure higher PIM DR priority on the intended DR | Intended DR router | `conf t`<br>`interface <RECEIVER_LAN_INTERFACE>`<br>`ip pim dr-priority <HIGH_PRIORITY>` | Intended router advertises higher PIM DR priority |
| 14 | Configure lower PIM DR priority on non-DR routers if deterministic election is required | Non-DR routers | `conf t`<br>`interface <RECEIVER_LAN_INTERFACE>`<br>`ip pim dr-priority <LOW_PRIORITY>` | Non-DR routers lose the election predictably |
| 15 | Clear PIM neighbor state or wait for hello update | Receiver-segment routers | `clear ip pim neighbor *` | PIM neighbor and DR election state refresh |
| 16 | Recheck PIM DR election | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>` | Intended router appears as the DR |
| 17 | Recheck IGMP interface DR view | Receiver-segment routers | `show ip igmp interface <RECEIVER_LAN_INTERFACE>` | Intended router appears as multicast DR |
| 18 | Configure receiver group membership | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the multicast group |
| 19 | Verify IGMP membership on the receiver segment | Intended DR and non-DR routers | `show ip igmp groups` | Receiver membership appears on the receiver-facing segment |
| 20 | Verify shared-tree state on the elected DR | Intended DR router | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` state appears for ASM receiver interest |
| 21 | Verify non-DR router does not incorrectly own receiver-side join responsibility | Non-DR router | `show ip mroute <MULTICAST_GROUP>` | Non-DR should not be the router driving receiver-side PIM joins for the segment |
| 22 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends multicast traffic toward the group |
| 23 | Verify packet counters and OIL on the elected DR | Intended DR router | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment and receiver LAN appears correctly in OIL |
| 24 | Verify RPF path from the receiver-side DR | Intended DR router | `show ip rpf <RP_ADDRESS>`<br>`show ip rpf <SOURCE_IP>` | RPF points toward RP for `(*,G)` and source for `(S,G)` |
| 25 | If HSRP or VRRP is present, compare FHRP active router to PIM DR | Receiver-segment routers | `show standby brief`<br>`show ip pim interface <RECEIVER_LAN_INTERFACE>` | FHRP active and PIM DR alignment is known |
| 26 | Align PIM DR with FHRP active router if the lab requires deterministic gateway and multicast ownership | Intended active router | `conf t`<br>`interface <RECEIVER_LAN_INTERFACE>`<br>`ip pim dr-priority <HIGH_PRIORITY>` | FHRP active router is also PIM DR |
| 27 | Save the working PIM DR configuration | Receiver-segment routers | `copy running-config startup-config` | PIM DR and last-hop receiver-segment baseline is preserved |
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RECEIVER_LAN_INTERFACE> = shared LAN interface facing receivers
! <UPSTREAM_INTERFACE>     = interface toward RP or source path
! <RECEIVER_INTERFACE>     = test receiver interface
! <MULTICAST_GROUP>        = multicast group being joined
! <RP_ADDRESS>             = RP address for ASM sparse-mode testing
! <GROUP_ACL>              = optional ACL matching multicast groups for RP
! <SOURCE_IP>              = multicast source IP
! <SOURCE_INTERFACE_OR_IP> = source interface or IP for test traffic
! <HIGH_PRIORITY>          = higher PIM DR priority, example 100
! <LOW_PRIORITY>           = lower PIM DR priority, example 1
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on receiver-segment multicast routers
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! RECEIVER-SEGMENT ROUTER BASELINE
! Apply on all routers attached to the shared receiver LAN
! =========================================================
conf t
 interface <RECEIVER_LAN_INTERFACE>
  ip pim sparse-mode
 exit
 interface <UPSTREAM_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! INTENDED PIM DR
! Highest DR priority wins.
! If priorities tie, highest IP address wins.
! =========================================================
conf t
 interface <RECEIVER_LAN_INTERFACE>
  ip pim dr-priority <HIGH_PRIORITY>
 exit
end
! =========================================================
! NON-DR ROUTER
! Optional, but useful for deterministic labs.
! =========================================================
conf t
 interface <RECEIVER_LAN_INTERFACE>
  ip pim dr-priority <LOW_PRIORITY>
 exit
end
! =========================================================
! REFRESH PIM STATE
! Use after changing DR priority if you want faster convergence.
! =========================================================
clear ip pim neighbor *
! =========================================================
! TEST RECEIVER JOIN
! Use join-group when a router is emulating a receiver
! and should respond to multicast pings.
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
show ip pim interface <RECEIVER_LAN_INTERFACE>
show ip pim neighbor
show ip igmp interface <RECEIVER_LAN_INTERFACE>
show ip igmp groups
show ip pim rp mapping
show ip rpf <RP_ADDRESS>
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
show standby brief
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Interface baseline | Receiver-segment routers | `show ip interface brief` | Receiver LAN and upstream interfaces are up/up |
| Multicast routing enabled | Receiver-segment routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | Receiver-segment routers | `show ip pim interface` | Receiver LAN and upstream interfaces appear |
| PIM receiver LAN neighbor | Receiver-segment routers | `show ip pim neighbor` | Neighbor appears on the shared receiver LAN |
| PIM DR election | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>` | DR field shows the intended router |
| PIM DR priority | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>` | DR priority reflects configured priority |
| IGMP interface DR view | Receiver-segment routers | `show ip igmp interface <RECEIVER_LAN_INTERFACE>` | `Multicast designated router (DR)` shows intended DR |
| RP mapping | Receiver-segment routers | `show ip pim rp mapping` | ASM group maps to expected RP |
| Receiver membership | Intended DR router | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| Receiver local join | Receiver/test router | `show ip igmp groups` | Local `join-group` appears with `never` expiration |
| Shared-tree state | Intended DR router | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` state appears for ASM receiver interest |
| Source-specific state | Intended DR router | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears after source traffic starts |
| RPF toward RP | Intended DR router | `show ip rpf <RP_ADDRESS>` | RPF interface points upstream toward RP |
| RPF toward source | Intended DR router | `show ip rpf <SOURCE_IP>` | RPF interface points upstream toward source |
| Receiver OIL | Intended DR router | `show ip mroute <MULTICAST_GROUP>` | Receiver LAN appears in OIL when appropriate |
| Packet counters | Intended DR router | `show ip mroute <MULTICAST_GROUP>` | Counters increment during test traffic |
| FHRP active router | Receiver-segment routers | `show standby brief` | Active HSRP router is identified if HSRP is present |
| FHRP and PIM DR alignment | Receiver-segment routers | `show standby brief`<br>`show ip pim interface <RECEIVER_LAN_INTERFACE>` | Active FHRP router and PIM DR are aligned if required |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies when using `ip igmp join-group` and multicast forwarding is correct |
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE PIM DR PRIORITY FROM INTENDED DR
! =========================================================
conf t
 interface <RECEIVER_LAN_INTERFACE>
  no ip pim dr-priority
 exit
end
! =========================================================
! REMOVE STATIC RP IF IT WAS ONLY USED FOR THIS LAB
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS>
 no ip pim rp-address <RP_ADDRESS> <GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM RECEIVER LAN AND UPSTREAM INTERFACES
! Only do this if no other multicast labs depend on these interfaces.
! =========================================================
conf t
 interface <RECEIVER_LAN_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <UPSTREAM_INTERFACE>
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
! CLEAR MULTICAST AND PIM STATE
! =========================================================
clear ip pim neighbor *
clear ip igmp group *
clear ip mroute *
undebug all
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Wrong router becomes PIM DR | Higher priority or higher IP address exists on another router | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>` | Configure higher `ip pim dr-priority` on intended DR |
| PIM DR and IGMP querier are different routers | Normal election difference: PIM DR and IGMP querier use different roles and election logic | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>`<br>`show ip igmp interface <RECEIVER_LAN_INTERFACE>` | No fix needed unless the lab requires alignment |
| PIM neighbor missing on receiver LAN | PIM missing on one router, interface down, wrong VLAN, or IP mismatch | Receiver-segment routers | `show ip pim neighbor`<br>`show ip interface brief` | Enable PIM on both receiver LAN interfaces and fix L2/L3 connectivity |
| Receiver group missing | Receiver did not send IGMP join or join was configured on the wrong interface | Intended DR and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <MULTICAST_GROUP>` on the correct receiver interface |
| Receiver join exists but no `(*,G)` state | RP mapping missing or elected DR is not sending joins upstream | Intended DR router | `show ip pim rp mapping`<br>`show ip mroute <MULTICAST_GROUP>` | Fix RP mapping and verify elected DR/upstream PIM path |
| Source traffic exists but no `(S,G)` state | Source is not reaching multicast domain, source-side PIM missing, or RP/source RPF failure | Source-side and intended DR routers | `show ip mroute <MULTICAST_GROUP>`<br>`show ip rpf <SOURCE_IP>` | Fix source traffic, PIM, and RPF path |
| PIM DR priority changed but DR did not change immediately | PIM hello/election state has not refreshed yet | Receiver-segment routers | `show ip pim interface <RECEIVER_LAN_INTERFACE>` | Wait for hello update or run `clear ip pim neighbor *` |
| HSRP active router is not PIM DR | FHRP and PIM DR elections are independent | Receiver-segment routers | `show standby brief`<br>`show ip pim interface <RECEIVER_LAN_INTERFACE>` | Set PIM DR priority so HSRP active router wins if required |
| Multicast traffic follows unexpected path | RPF path points to a different upstream interface than expected | Intended DR router | `show ip rpf <RP_ADDRESS>`<br>`show ip rpf <SOURCE_IP>` | Correct unicast routing or use deliberate multicast RPF correction |
| OIL missing receiver LAN | IGMP receiver interest is absent or not learned by last-hop router | Intended DR router | `show ip igmp groups`<br>`show ip mroute <MULTICAST_GROUP>` | Fix receiver join and receiver LAN PIM/IGMP state |
| Non-DR appears to have unexpected state | Shared LAN multicast state may exist, but DR owns join/register responsibility | Non-DR router | `show ip pim interface`<br>`show ip mroute <MULTICAST_GROUP>` | Verify actual DR and OIL before treating this as a failure |
| Multicast ping gets no replies | Receiver used `ip igmp static-group`, wrong receiver path, or RPF failure | Receiver and multicast routers | `show running-config interface <RECEIVER_INTERFACE>`<br>`show ip rpf <SOURCE_IP>` | Use `ip igmp join-group` and fix RPF or forwarding state |
| Debug output overwhelms console | Debug left enabled during multicast testing | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Mental_Model
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Configuration_Checklist
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Skeleton
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Verification_Commands
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Rollback
# PIM_DR_Receiver_Segment_And_Last_Hop_Router_Failure_Checks
