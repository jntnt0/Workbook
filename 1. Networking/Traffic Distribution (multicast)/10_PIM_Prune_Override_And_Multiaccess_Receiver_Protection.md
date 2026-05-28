PIM_Prune_Override_And_Multiaccess_Receiver_Protection.md

PIM_Prune_Override_And_Multiaccess_Receiver_Protection

# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Prune override | On a shared multiaccess segment, one downstream router may prune traffic, but another downstream router may still need the flow |
| Multiaccess segment | Ethernet-style shared segment where multiple PIM routers hear each other's PIM messages |
| The danger | If one router's prune immediately stopped traffic on the shared segment, receivers behind another router would lose multicast |
| Override join | A router that still needs the traffic sends a join to override the prune before the upstream router removes the shared segment from the OIL |
| Receiver protection | Prune override protects active receivers from being blackholed by another router's prune |
| Dense-mode fit | This behavior is easiest to see in PIM dense mode because traffic floods first, then routers with no downstream receivers prune |
| No special enable command | In normal IOS-style PIM operation, prune override is protocol behavior. You build the correct shared PIM segment and verify the behavior |
| Upstream router role | The upstream router has the shared LAN in its OIL while at least one downstream router still wants the multicast flow |
| Pruning router role | A downstream router with no local receivers sends a prune upstream |
| Overriding router role | A downstream router with receivers sends a join to keep the upstream router forwarding on the shared segment |
| RPF still applies | Prune override does not bypass RPF. The multicast packet must still arrive on the correct RPF interface |
| OIL is the proof | The shared multiaccess interface should remain in the upstream router's OIL while any downstream receiver still exists |
| Failure pattern | If the shared segment disappears from the upstream OIL while a downstream receiver still exists, prune override behavior or receiver state is broken |
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the upstream router, shared multiaccess segment, receiver branch, and nonreceiver branch | All routers | `show ip interface brief` | Source-facing, shared-segment, receiver-facing, and nonreceiver interfaces are known |
| 2 | Confirm all routers on the shared segment are in the same IP subnet | Shared-segment routers | `show ip interface brief` | Upstream, receiver-branch, and nonreceiver-branch interfaces share the same subnet |
| 3 | Confirm unicast reachability to the multicast source | Downstream routers | `show ip route <SOURCE_IP>` | Downstream routers have a route toward the source through the upstream path |
| 4 | Confirm the RPF interface points to the shared segment | Downstream routers | `show ip rpf <SOURCE_IP>` | RPF interface is the shared multiaccess interface toward the upstream router |
| 5 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Routers can build multicast control-plane and forwarding state |
| 6 | Enable PIM dense mode on the upstream source-facing interface | Upstream router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim dense-mode` | Upstream router can receive multicast from the source side |
| 7 | Enable PIM dense mode on the upstream shared-segment interface | Upstream router | `conf t`<br>`interface <UPSTREAM_SHARED_INTERFACE>`<br>`ip pim dense-mode` | Upstream router can forward multicast onto the shared multiaccess segment |
| 8 | Enable PIM dense mode on the receiver-branch shared interface | Receiver-branch router | `conf t`<br>`interface <RECEIVER_BRANCH_SHARED_INTERFACE>`<br>`ip pim dense-mode` | Receiver-branch router participates in PIM on the shared segment |
| 9 | Enable PIM dense mode on the receiver-facing interface | Receiver-branch router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim dense-mode` | Receiver-facing interface can process IGMP joins |
| 10 | Enable PIM dense mode on the nonreceiver-branch shared interface | Nonreceiver-branch router | `conf t`<br>`interface <NONRECEIVER_BRANCH_SHARED_INTERFACE>`<br>`ip pim dense-mode` | Nonreceiver-branch router participates in PIM and can send prune state |
| 11 | Enable PIM dense mode on the nonreceiver downstream interface if the lab has one | Nonreceiver-branch router | `conf t`<br>`interface <NONRECEIVER_DOWNSTREAM_INTERFACE>`<br>`ip pim dense-mode` | Nonreceiver branch can demonstrate empty downstream interest |
| 12 | Verify PIM is enabled on required interfaces | All multicast routers | `show ip pim interface` | Source, shared-segment, receiver-facing, and branch interfaces appear |
| 13 | Verify PIM neighbors on the shared multiaccess segment | Shared-segment routers | `show ip pim neighbor` | Upstream router sees both downstream routers as PIM neighbors on the shared LAN |
| 14 | Configure receiver group membership behind the receiver-branch router | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the multicast group |
| 15 | Verify receiver membership on the receiver-branch router | Receiver-branch router | `show ip igmp groups` | Multicast group appears on the receiver-facing interface |
| 16 | Confirm the nonreceiver branch has no receiver membership | Nonreceiver-branch router | `show ip igmp groups` | Target multicast group is absent on nonreceiver-facing interfaces |
| 17 | Enable controlled PIM debugging if the lab requires observing prune and override messages | Shared-segment routers | `debug ip pim` | PIM Join/Prune messages are visible during the test |
| 18 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Dense-mode traffic is sourced into the multicast domain |
| 19 | Verify multicast state on the upstream router | Upstream router | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state exists and the shared segment appears in the OIL |
| 20 | Verify the receiver branch keeps multicast state | Receiver-branch router | `show ip mroute <MULTICAST_GROUP>` | Receiver-facing interface appears in the OIL for the group |
| 21 | Verify the nonreceiver branch prunes the flow | Nonreceiver-branch router | `show ip mroute <MULTICAST_GROUP>` | Nonreceiver branch shows no receiver OIL or pruned state for the group |
| 22 | Verify the upstream shared segment remains protected | Upstream router | `show ip mroute <MULTICAST_GROUP>` | Shared multiaccess interface remains in the OIL because receiver branch still needs traffic |
| 23 | Confirm packet counters increment on the receiver branch | Receiver-branch router | `show ip mroute <MULTICAST_GROUP>` | Packet counters increase during test traffic |
| 24 | Confirm nonreceiver branch is not forwarding unnecessarily downstream | Nonreceiver-branch router | `show ip mroute <MULTICAST_GROUP>` | No downstream OIL exists for the multicast group |
| 25 | Remove receiver membership to prove the shared segment can finally prune | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`no ip igmp join-group <MULTICAST_GROUP>` | Receiver no longer requests the group |
| 26 | Clear multicast state for a clean retest | All multicast routers | `clear ip mroute *` | Dynamic mroute state is removed |
| 27 | Retest source traffic with no receivers | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Branches without receivers prune or show empty OIL |
| 28 | Re-add receiver membership to prove protection returns | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver branch joins the group again |
| 29 | Retest source traffic with receiver restored | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Upstream shared interface remains in OIL while receiver exists |
| 30 | Disable debugging after testing | All routers | `undebug all` | Debugging is disabled |
| 31 | Save the working prune override lab baseline | All changed routers | `copy running-config startup-config` | PIM prune override and receiver protection baseline is preserved |
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <SOURCE_IP>                              = multicast source IP
! <SOURCE_INTERFACE_OR_IP>                 = source interface or IP used for multicast test traffic
! <SOURCE_FACING_INTERFACE>                = upstream router interface toward source
! <UPSTREAM_SHARED_INTERFACE>              = upstream router interface on shared multiaccess segment
! <RECEIVER_BRANCH_SHARED_INTERFACE>       = receiver-branch router interface on shared segment
! <NONRECEIVER_BRANCH_SHARED_INTERFACE>    = nonreceiver-branch router interface on shared segment
! <RECEIVER_FACING_INTERFACE>              = receiver-branch interface toward receiver LAN
! <NONRECEIVER_DOWNSTREAM_INTERFACE>       = optional nonreceiver branch downstream interface
! <RECEIVER_INTERFACE>                     = test receiver interface
! <MULTICAST_GROUP>                        = multicast group used for prune override testing
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! UPSTREAM ROUTER
! This router receives source traffic and forwards onto the
! shared multiaccess segment.
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim dense-mode
 exit
 interface <UPSTREAM_SHARED_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! RECEIVER-BRANCH ROUTER
! This router has downstream receiver interest and should
! override another router's prune on the shared segment.
! =========================================================
conf t
 interface <RECEIVER_BRANCH_SHARED_INTERFACE>
  ip pim dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! NONRECEIVER-BRANCH ROUTER
! This router has no interested receivers and should prune.
! =========================================================
conf t
 interface <NONRECEIVER_BRANCH_SHARED_INTERFACE>
  ip pim dense-mode
 exit
 interface <NONRECEIVER_DOWNSTREAM_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! TEST RECEIVER JOIN
! Use join-group when a router is emulating a receiver
! and should respond to multicast ping tests.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! OPTIONAL CONTROLLED DEBUG
! Use briefly only during the prune override test.
! =========================================================
debug ip pim
! =========================================================
! TEST TRAFFIC
! =========================================================
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show ip pim interface
show ip pim neighbor
show ip igmp groups
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
! =========================================================
! CLEAN DEBUG
! =========================================================
undebug all
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Interface baseline | All routers | `show ip interface brief` | Required interfaces are up/up |
| Shared segment addressing | Shared-segment routers | `show ip interface brief` | Upstream, receiver-branch, and nonreceiver-branch interfaces are in the same subnet |
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM dense-mode config | All multicast routers | `show running-config interface <INTERFACE>` | Interface contains `ip pim dense-mode` |
| PIM interface state | All multicast routers | `show ip pim interface` | Required interfaces appear as PIM interfaces |
| Shared segment PIM neighbors | Shared-segment routers | `show ip pim neighbor` | Routers see each other as PIM neighbors on the shared LAN |
| Route to source | Downstream routers | `show ip route <SOURCE_IP>` | Route points upstream toward the source |
| RPF path to source | Downstream routers | `show ip rpf <SOURCE_IP>` | RPF interface is the shared multiaccess segment |
| Receiver membership | Receiver-branch router | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| No receiver membership | Nonreceiver-branch router | `show ip igmp groups` | Target group is absent on nonreceiver branch |
| Upstream mroute state | Upstream router | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state exists after traffic starts |
| Upstream OIL protection | Upstream router | `show ip mroute <MULTICAST_GROUP>` | Shared segment remains in OIL while one downstream router still has a receiver |
| Receiver-branch OIL | Receiver-branch router | `show ip mroute <MULTICAST_GROUP>` | Receiver-facing interface appears in OIL |
| Nonreceiver prune behavior | Nonreceiver-branch router | `show ip mroute <MULTICAST_GROUP>` | No downstream OIL or pruned state is shown for the group |
| Packet counters | Upstream and receiver-branch routers | `show ip mroute <MULTICAST_GROUP>` | Counters increment during traffic |
| PIM Join/Prune visibility | Shared-segment routers | `debug ip pim` | Join/Prune activity appears during receiver join and prune testing |
| Receiver restored test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver responds when `ip igmp join-group` is active |
| Debug cleanup | All routers | `show debugging`<br>`undebug all` | No PIM debug remains active |
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE PIM FROM UPSTREAM ROUTER INTERFACES
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip pim dense-mode
 exit
 interface <UPSTREAM_SHARED_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE PIM FROM RECEIVER-BRANCH ROUTER INTERFACES
! =========================================================
conf t
 interface <RECEIVER_BRANCH_SHARED_INTERFACE>
  no ip pim dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE PIM FROM NONRECEIVER-BRANCH ROUTER INTERFACES
! =========================================================
conf t
 interface <NONRECEIVER_BRANCH_SHARED_INTERFACE>
  no ip pim dense-mode
 exit
 interface <NONRECEIVER_DOWNSTREAM_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE GLOBAL MULTICAST ROUTING
! Only do this if no other multicast labs or services depend on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR MULTICAST STATE AND DEBUGGING
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No PIM neighbors on shared segment | Interfaces are not in same subnet, PIM is missing, or L2 segment is broken | Shared-segment routers | `show ip pim neighbor`<br>`show ip interface brief` | Fix addressing/L2 reachability and enable `ip pim dense-mode` on shared interfaces |
| Shared segment disappears from upstream OIL while receiver exists | Receiver-branch router did not send join override or receiver state is missing | Upstream and receiver-branch routers | `show ip mroute <MULTICAST_GROUP>`<br>`show ip igmp groups` | Restore receiver join and verify PIM on the shared segment |
| Receiver branch has no OIL toward receiver | IGMP membership missing or receiver-facing PIM/IGMP is not active | Receiver-branch router | `show ip igmp groups`<br>`show ip pim interface` | Configure receiver join and enable PIM on receiver-facing interface |
| Nonreceiver branch never prunes | It has unintended receiver state or downstream PIM/IGMP state | Nonreceiver-branch router | `show ip igmp groups`<br>`show ip mroute <MULTICAST_GROUP>` | Remove unintended joins or receiver state |
| Multicast traffic fails even though OIL looks correct | RPF failure | Downstream routers | `show ip rpf <SOURCE_IP>`<br>`show ip route <SOURCE_IP>` | Fix unicast route toward source or use a deliberate `ip mroute` override |
| Incoming interface is wrong | Route to source points away from the shared segment | Downstream routers | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Correct routing so RPF points to the shared upstream segment |
| No multicast state appears | Source traffic never started or multicast routing is disabled | Source and multicast routers | `show running-config | include ip multicast-routing`<br>`show ip mroute <MULTICAST_GROUP>` | Enable multicast routing and generate source traffic |
| Receiver does not reply to multicast ping | Receiver used `ip igmp static-group`, wrong interface, or RPF/forwarding failure exists | Receiver/test router and multicast routers | `show running-config interface <RECEIVER_INTERFACE>`<br>`show ip mroute <MULTICAST_GROUP>` | Use `ip igmp join-group` and fix forwarding/RPF |
| PIM debug shows prune but no override join | Receiver branch does not believe it needs traffic or PIM neighbor state is broken | Receiver-branch router | `debug ip pim`<br>`show ip igmp groups` | Verify receiver join and PIM neighbor on shared segment |
| Dense-mode reflooding is observed | Normal dense-mode behavior after prune state expires | All routers | `show ip mroute <MULTICAST_GROUP>` | Expected behavior. Use sparse mode if periodic reflooding is not acceptable |
| Shared segment receives initial traffic on all routers | Normal dense-mode initial flood before pruning | Shared-segment routers | `show ip mroute <MULTICAST_GROUP>` | No fix needed unless dense mode is the wrong design choice |
| Upstream router has empty OIL after all receivers are removed | Expected behavior after no downstream routers need the flow | Upstream router | `show ip mroute <MULTICAST_GROUP>` | No fix needed |
| Prune override cannot be proven from output | State converges too quickly or debug was not enabled during event | Shared-segment routers | `debug ip pim`<br>`clear ip mroute *` | Enable debug briefly, clear state, then retest traffic and joins |
| Debug floods the console | `debug ip pim` left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Mental_Model
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Configuration_Checklist
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Skeleton
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Verification_Commands
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Rollback
# PIM_Prune_Override_And_Multiaccess_Receiver_Protection_Failure_Checks

