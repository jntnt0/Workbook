
PIM_Dense_Mode_Flood_And_Prune.md

PIM_Dense_Mode_Flood_And_Prune


# PIM_Dense_Mode_Flood_And_Prune_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PIM dense mode | Dense mode assumes receivers may exist everywhere, so multicast traffic is initially flooded across PIM-enabled interfaces |
| Flood-and-prune model | Traffic is pushed through the multicast domain first. Routers with no downstream receivers send prune messages upstream |
| No RP required | PIM dense mode does not use a Rendezvous Point. It builds source trees directly from the source |
| Source tree behavior | Dense mode primarily creates `(S,G)` state because forwarding is source-specific |
| RPF still rules | A router only accepts multicast traffic if it arrives on the interface used to reach the source |
| Incoming Interface | In `show ip mroute`, the Incoming Interface is the expected upstream interface toward the source |
| Outgoing Interface List | The OIL lists downstream interfaces where traffic is forwarded. Empty OIL usually means no downstream receiver interest |
| Pruned state | A pruned branch may show the `P` flag in mroute output, meaning traffic for that source/group has been pruned downstream |
| Periodic reflooding | Dense mode periodically refloods traffic after prune state expires, then prunes again if no receivers exist |
| IGMP receiver role | IGMP on the last-hop LAN tells the local router that a receiver wants the multicast group |
| PIM neighbor role | PIM neighbors exchange multicast control messages and build the multicast distribution tree |
| Lab goal | Prove that multicast floods first, prunes unwanted branches, keeps receiver branches in the OIL, and passes RPF checks |
# PIM_Dense_Mode_Flood_And_Prune_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify multicast source, transit links, receiver LAN, and nonreceiver branches | All routers | `show ip interface brief` | Required routed interfaces and test segments are known |
| 2 | Confirm unicast reachability to the multicast source | Transit and receiver-side routers | `show ip route <SOURCE_IP>` | Each router has a route back toward the source |
| 3 | Validate expected RPF interface before multicast testing | Transit and receiver-side routers | `show ip rpf <SOURCE_IP>` | RPF interface points toward the expected upstream source path |
| 4 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 5 | Enable PIM dense mode on the source-facing interface | Source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim dense-mode` | Source-facing interface participates in dense-mode multicast |
| 6 | Enable PIM dense mode on transit interfaces | Transit routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim dense-mode` | Transit links can flood and prune multicast traffic |
| 7 | Enable PIM dense mode on the receiver-facing interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim dense-mode` | Receiver-facing interface can receive IGMP joins and join the OIL |
| 8 | Enable PIM dense mode on any intentional nonreceiver branch | Branch router | `conf t`<br>`interface <NONRECEIVER_BRANCH_INTERFACE>`<br>`ip pim dense-mode` | Branch can demonstrate prune behavior when no receiver exists |
| 9 | Verify PIM is active on all required interfaces | All multicast routers | `show ip pim interface` | Required interfaces appear with dense-mode PIM enabled |
| 10 | Verify PIM neighbor formation on routed transit links | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear on transit interfaces |
| 11 | Confirm no RP dependency is needed for dense mode | All multicast routers | `show ip pim rp mapping` | No RP is required for dense-mode forwarding |
| 12 | Configure receiver group membership | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Test receiver joins the multicast group |
| 13 | Verify receiver-side IGMP membership | Last-hop router | `show ip igmp groups` | Multicast group appears on the receiver-facing interface |
| 14 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast traffic is sourced into the dense-mode domain |
| 15 | Verify multicast route state is created | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears for the source and group |
| 16 | Validate RPF on the `(S,G)` entry | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>`<br>`show ip rpf <SOURCE_IP>` | Incoming Interface matches the RPF interface toward the source |
| 17 | Verify receiver branch remains in the OIL | Upstream and transit routers | `show ip mroute <MULTICAST_GROUP>` | OIL includes downstream interfaces toward interested receivers |
| 18 | Verify nonreceiver branches are pruned | Upstream and branch routers | `show ip mroute <MULTICAST_GROUP>` | Nonreceiver branches show no OIL entry or show pruned state |
| 19 | Check multicast packet counters during traffic | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment on active forwarding paths |
| 20 | Clear multicast state for a clean flood-and-prune retest | All multicast routers | `clear ip mroute *` | Dynamic mroute state is removed |
| 21 | Retest multicast traffic after clearing state | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Flood, prune, and receiver forwarding behavior rebuilds |
| 22 | Verify final forwarding state | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Receiver path has OIL entries, nonreceiver paths are pruned or empty |
| 23 | Save the working dense-mode configuration | All changed routers | `copy running-config startup-config` | PIM dense-mode baseline is preserved |
# PIM_Dense_Mode_Flood_And_Prune_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <SOURCE_IP>                    = multicast source IP
! <SOURCE_INTERFACE_OR_IP>       = source interface or source IP used for test traffic
! <SOURCE_FACING_INTERFACE>      = interface connected toward multicast source
! <TRANSIT_INTERFACE>            = routed interface between PIM routers
! <RECEIVER_FACING_INTERFACE>    = last-hop router interface toward receiver LAN
! <RECEIVER_INTERFACE>           = receiver/test router interface
! <NONRECEIVER_BRANCH_INTERFACE> = optional branch interface used to prove pruning
! <MULTICAST_GROUP>              = multicast group used for the lab
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! SOURCE-SIDE ROUTER
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim dense-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! TRANSIT ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! LAST-HOP ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  ip pim dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! OPTIONAL NONRECEIVER BRANCH
! Use this to observe dense-mode prune behavior.
! =========================================================
conf t
 interface <NONRECEIVER_BRANCH_INTERFACE>
  ip pim dense-mode
 exit
end
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
show ip pim interface
show ip pim neighbor
show ip igmp groups
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
# PIM_Dense_Mode_Flood_And_Prune_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Interface baseline | All routers | `show ip interface brief` | Required interfaces are up/up |
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | All multicast routers | `show ip pim interface` | Required interfaces appear with PIM enabled |
| PIM dense-mode config | All multicast routers | `show running-config interface <INTERFACE>` | Interface contains `ip pim dense-mode` |
| PIM neighbors | Transit routers | `show ip pim neighbor` | Expected neighbors appear on routed transit links |
| No RP dependency | All multicast routers | `show ip pim rp mapping` | Dense-mode lab does not require RP mapping |
| Receiver membership | Last-hop router | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| Local receiver join | Receiver/test router | `show ip igmp groups` | Local `join-group` appears with `never` expiration |
| RPF decision | Transit and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF interface points toward source |
| Multicast state | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state exists after source traffic starts |
| Incoming interface | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | Incoming Interface matches RPF interface |
| Receiver OIL | Source-side and transit routers | `show ip mroute <MULTICAST_GROUP>` | OIL includes interfaces toward receivers |
| Pruned branch | Nonreceiver branch router | `show ip mroute <MULTICAST_GROUP>` | Branch shows no receiver OIL or pruned state |
| Packet counters | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Counters increment during test traffic |
| Multicast forwarding cache | Supported IOS routers | `show ip mfib <MULTICAST_GROUP>` | MFIB entry exists for forwarding path |
| Clean retest | All multicast routers | `clear ip mroute *`<br>`show ip mroute <MULTICAST_GROUP>` | State clears, then rebuilds after traffic resumes |
| End-to-end test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver responds when using `ip igmp join-group` |
# PIM_Dense_Mode_Flood_And_Prune_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE PIM DENSE MODE FROM SOURCE-SIDE INTERFACES
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip pim dense-mode
 exit
 interface <TRANSIT_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE PIM DENSE MODE FROM LAST-HOP INTERFACES
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE PIM DENSE MODE FROM OPTIONAL NONRECEIVER BRANCH
! =========================================================
conf t
 interface <NONRECEIVER_BRANCH_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! DISABLE MULTICAST ROUTING GLOBALLY
! Only do this if no other multicast labs or services depend on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR MULTICAST STATE
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_Dense_Mode_Flood_And_Prune_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No multicast state appears | Source traffic never started or multicast routing is disabled | Source and multicast routers | `show running-config | include ip multicast-routing`<br>`show ip mroute <MULTICAST_GROUP>` | Enable `ip multicast-routing` and generate multicast traffic |
| PIM interface missing | `ip pim dense-mode` not configured on the interface | Affected router | `show ip pim interface` | Configure `ip pim dense-mode` on required routed interfaces |
| No PIM neighbor | PIM missing on one side, interface down, IP mismatch, or transit issue | Transit routers | `show ip pim neighbor`<br>`show ip interface brief` | Fix interface reachability and enable PIM on both sides |
| Receiver group missing | Receiver did not send IGMP join or join was configured on wrong interface | Last-hop router | `show ip igmp groups` | Configure `ip igmp join-group <MULTICAST_GROUP>` on the correct receiver interface |
| Multicast ping gets no replies | Receiver used `ip igmp static-group` instead of `ip igmp join-group` | Receiver/test router | `show running-config interface <RECEIVER_INTERFACE>` | Use `ip igmp join-group` for router-based multicast ping testing |
| Incoming interface is wrong | RPF route to source points to the wrong interface | Transit or last-hop router | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>` | Correct unicast routing or add a deliberate `ip mroute` override |
| Traffic is dropped even though PIM neighbors exist | RPF failure | Transit or last-hop router | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Make RPF interface match the actual incoming multicast path |
| OIL is empty on a router that should forward | No downstream receiver interest or downstream branch has pruned | Upstream or transit router | `show ip mroute <MULTICAST_GROUP>`<br>`show ip igmp groups` | Verify receiver joins and downstream PIM state |
| Nonreceiver branch receives initial traffic | Normal dense-mode flood before prune state forms | Nonreceiver branch router | `show ip mroute <MULTICAST_GROUP>` | No fix needed unless dense mode is the wrong design choice |
| Nonreceiver branch keeps receiving traffic | Prune not sent, PIM neighbor missing, or receiver state exists unexpectedly | Nonreceiver branch router | `show ip mroute <MULTICAST_GROUP>`<br>`show ip pim neighbor` | Remove unintended receiver state and fix PIM control-plane behavior |
| Traffic periodically refloods | Normal dense-mode prune expiration behavior | All routers | `show ip mroute <MULTICAST_GROUP>` | Expected dense-mode behavior. Use sparse mode if periodic reflooding is unacceptable |
| RP mapping is missing | Dense mode does not require RP | All routers | `show ip pim rp mapping` | No fix needed for dense mode |
| Dense-mode traffic is too noisy | Dense mode is a poor fit for sparse receiver placement | Network design | `show ip mroute`<br>`show ip pim interface` | Use PIM sparse mode, SSM, or scoped multicast design instead |
| Debug floods console | Debug left enabled during PIM testing | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_Dense_Mode_Flood_And_Prune_Mental_Model
# PIM_Dense_Mode_Flood_And_Prune_Configuration_Checklist
# PIM_Dense_Mode_Flood_And_Prune_Skeleton
# PIM_Dense_Mode_Flood_And_Prune_Verification_Commands
# PIM_Dense_Mode_Flood_And_Prune_Rollback
# PIM_Dense_Mode_Flood_And_Prune_Failure_Checks

