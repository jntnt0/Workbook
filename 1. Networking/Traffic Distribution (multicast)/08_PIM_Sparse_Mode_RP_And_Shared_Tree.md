
PIM_Sparse_Mode_RP_And_Shared_Tree.md

PIM_Sparse_Mode_RP_And_Shared_Tree

# PIM_Sparse_Mode_RP_And_Shared_Tree_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PIM sparse mode | Sparse mode assumes receivers are not everywhere. Traffic is forwarded only where receivers explicitly join |
| ASM model | Any Source Multicast uses group joins where receivers ask for traffic for group `G` without initially knowing source `S` |
| Rendezvous Point | The RP is the meeting point for sources and receivers in PIM-SM |
| Shared tree | Receivers build `(*,G)` state toward the RP. This means "any source for this group through the RP" |
| Source registration | The first-hop router near the source sends PIM register messages to the RP so the RP learns about active sources |
| Last-hop router | The router near receivers hears IGMP joins and sends PIM joins upstream toward the RP |
| `(*,G)` state | Group-level shared-tree state toward the RP |
| `(S,G)` state | Source-specific state for one source sending to one group |
| RPF toward RP | For `(*,G)` shared-tree state, RPF points toward the RP |
| RPF toward source | For `(S,G)` source-specific state, RPF points toward the multicast source |
| OIL | The outgoing interface list shows where multicast traffic is forwarded downstream |
| Static RP | Every multicast router must have consistent group-to-RP mapping when static RP is used |
| RP overlap problem | If multiple static RPs cover overlapping groups, the selected RP may not be the one intended |
| Dense mode contrast | Dense mode floods first and prunes. Sparse mode joins first and forwards only after receiver interest exists |
# PIM_Sparse_Mode_RP_And_Shared_Tree_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the multicast source, RP, transit routers, and receiver-side router | All routers | `show ip interface brief` | Source-facing, RP loopback, transit, and receiver-facing interfaces are known |
| 2 | Confirm unicast reachability to the RP address | All multicast routers | `ping <RP_ADDRESS>` | Every router can reach the RP address |
| 3 | Confirm unicast reachability to the multicast source | RP, transit, and last-hop routers | `show ip route <SOURCE_IP>` | Each router has a route toward the source |
| 4 | Confirm unicast reachability from source-side router to RP | Source-side router | `show ip route <RP_ADDRESS>` | Source-side router has a route toward the RP |
| 5 | Confirm unicast reachability from receiver-side router to RP | Last-hop router | `show ip route <RP_ADDRESS>` | Receiver-side router has a route toward the RP |
| 6 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 7 | Enable PIM sparse mode on the RP loopback | RP router | `conf t`<br>`interface <RP_LOOPBACK_INTERFACE>`<br>`ip pim sparse-mode` | RP address participates in PIM sparse-mode multicast control |
| 8 | Enable PIM sparse mode on source-facing interface | Source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-facing interface participates in PIM-SM |
| 9 | Enable PIM sparse mode on transit interfaces | Transit routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-mode` | Transit links can form PIM neighbors and carry joins |
| 10 | Enable PIM sparse mode on receiver-facing interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-facing interface can process IGMP joins and build PIM join state |
| 11 | Verify PIM interface enablement | All multicast routers | `show ip pim interface` | Required interfaces appear as PIM interfaces |
| 12 | Verify PIM neighbor formation | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear on transit links |
| 13 | Create an ACL for the multicast group range if the RP should serve only selected groups | All multicast routers | `conf t`<br>`access-list <RP_GROUP_ACL> permit <MULTICAST_GROUP_RANGE> <WILDCARD_MASK>` | ACL matches the multicast group range assigned to this RP |
| 14 | Configure static RP for all groups if this is a simple one-RP lab | All multicast routers | `conf t`<br>`ip pim rp-address <RP_ADDRESS>` | All multicast groups map to the static RP |
| 15 | Configure static RP for selected group range if using scoped RP mapping | All multicast routers | `conf t`<br>`ip pim rp-address <RP_ADDRESS> <RP_GROUP_ACL>` | Only ACL-matched groups map to the selected RP |
| 16 | Verify RP mapping | All multicast routers | `show ip pim rp mapping` | Multicast group maps to the expected RP |
| 17 | Verify no unintended overlapping RP mapping exists | All multicast routers | `show ip pim rp mapping`<br>`show access-lists` | Group range points to the intended RP only |
| 18 | Configure receiver group membership | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the multicast group |
| 19 | Verify IGMP receiver membership on last-hop router | Last-hop router | `show ip igmp groups` | Joined group appears on the receiver-facing interface |
| 20 | Verify shared-tree state before source traffic if possible | Last-hop and transit routers | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` state appears toward the RP |
| 21 | Validate RPF toward RP for shared-tree state | Last-hop and transit routers | `show ip rpf <RP_ADDRESS>`<br>`show ip mroute <MULTICAST_GROUP>` | `(*,G)` Incoming Interface points toward the RP path |
| 22 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends multicast traffic toward the group |
| 23 | Verify RP learns source-specific state | RP router | `show ip mroute <MULTICAST_GROUP>` | RP shows `(S,G)` state for the active source |
| 24 | Verify source-side PIM register tunnel behavior if supported | Source-side router and RP | `show ip pim tunnel` | PIM encapsulation or decapsulation tunnel appears for register behavior |
| 25 | Verify multicast state on all routers | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` entries exist as expected |
| 26 | Verify OIL on the RP and transit routers | RP and transit routers | `show ip mroute <MULTICAST_GROUP>` | OIL points toward downstream receivers |
| 27 | Validate RPF toward source for source-specific state | RP, transit, and last-hop routers | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | `(S,G)` Incoming Interface points toward the source path |
| 28 | Confirm receiver response or counter movement | Source/test router and multicast routers | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5`<br>`show ip mroute <MULTICAST_GROUP>` | Receiver replies or multicast packet counters increment |
| 29 | Clear multicast state for a clean retest | All multicast routers | `clear ip mroute *` | Old dynamic multicast state is removed |
| 30 | Retest receiver join and source traffic | Receiver and source routers | `show ip igmp groups`<br>`ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Shared-tree and forwarding state rebuild correctly |
| 31 | Save the working PIM-SM baseline | All changed routers | `copy running-config startup-config` | Sparse-mode RP and shared-tree baseline is preserved |
# PIM_Sparse_Mode_RP_And_Shared_Tree_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RP_ADDRESS>                 = RP loopback or stable routed address
! <RP_LOOPBACK_INTERFACE>      = loopback used as RP address
! <RP_GROUP_ACL>               = ACL number or name matching multicast groups for RP
! <MULTICAST_GROUP_RANGE>      = multicast range assigned to RP
! <WILDCARD_MASK>              = wildcard mask for multicast group range
! <MULTICAST_GROUP>            = test multicast group
! <SOURCE_IP>                  = multicast source IP
! <SOURCE_INTERFACE_OR_IP>     = source interface or IP for multicast test traffic
! <SOURCE_FACING_INTERFACE>    = source-side router interface toward source
! <TRANSIT_INTERFACE>          = routed interface between PIM routers
! <RECEIVER_FACING_INTERFACE>  = last-hop router interface toward receivers
! <RECEIVER_INTERFACE>         = test receiver interface
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! RP ROUTER
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! SOURCE-SIDE ROUTER
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! TRANSIT ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! LAST-HOP ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! STATIC RP FOR ALL MULTICAST GROUPS
! Use this for a simple one-RP sparse-mode lab.
! Apply on every multicast router.
! =========================================================
conf t
 ip pim rp-address <RP_ADDRESS>
end
! =========================================================
! STATIC RP FOR A SPECIFIC GROUP RANGE
! Use this when the RP should handle only selected groups.
! Apply on every multicast router.
! =========================================================
conf t
 access-list <RP_GROUP_ACL> permit <MULTICAST_GROUP_RANGE> <WILDCARD_MASK>
 ip pim rp-address <RP_ADDRESS> <RP_GROUP_ACL>
end
! =========================================================
! RECEIVER JOIN
! Use join-group when a router is emulating a receiver
! and should respond to multicast ping tests.
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
show ip pim rp mapping
show ip igmp groups
show ip rpf <RP_ADDRESS>
show ip rpf <SOURCE_IP>
show ip mroute <MULTICAST_GROUP>
show ip pim tunnel
# PIM_Sparse_Mode_RP_And_Shared_Tree_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Interface baseline | All routers | `show ip interface brief` | Required interfaces are up/up |
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM sparse-mode config | All multicast routers | `show running-config interface <INTERFACE>` | Interface contains `ip pim sparse-mode` |
| PIM interface state | All multicast routers | `show ip pim interface` | Required interfaces appear as PIM interfaces |
| PIM neighbors | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| RP reachability | All multicast routers | `ping <RP_ADDRESS>` | RP address replies |
| RP route | All multicast routers | `show ip route <RP_ADDRESS>` | Route to RP exists |
| Static RP config | All multicast routers | `show running-config | include ip pim rp-address` | Expected RP address is configured |
| RP group ACL | All multicast routers | `show access-lists <RP_GROUP_ACL>` | ACL permits the intended multicast group range |
| RP mapping | All multicast routers | `show ip pim rp mapping` | Test group maps to the expected RP |
| Receiver membership | Last-hop router | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| Shared-tree state | Last-hop and transit routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` state exists for the joined group |
| RPF toward RP | Last-hop and transit routers | `show ip rpf <RP_ADDRESS>` | RPF interface points toward the RP |
| Source route | RP and last-hop routers | `show ip route <SOURCE_IP>` | Route to source exists |
| RPF toward source | RP and transit routers | `show ip rpf <SOURCE_IP>` | RPF interface points toward source |
| Source-specific state | RP and multicast routers | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears after source traffic starts |
| RP OIL | RP router | `show ip mroute <MULTICAST_GROUP>` | OIL points toward receivers for the group |
| Last-hop OIL | Last-hop router | `show ip mroute <MULTICAST_GROUP>` | Receiver-facing interface appears in OIL |
| PIM register tunnel | Source-side router and RP | `show ip pim tunnel` | PIM encap or decap tunnel appears if platform displays register tunnels |
| Packet counters | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Counters increment during multicast traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies when using `ip igmp join-group` and forwarding is correct |
# PIM_Sparse_Mode_RP_And_Shared_Tree_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE STATIC RP CONFIGURATION
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS>
 no ip pim rp-address <RP_ADDRESS> <RP_GROUP_ACL>
end
! =========================================================
! REMOVE RP GROUP ACL
! =========================================================
conf t
 no access-list <RP_GROUP_ACL>
end
! =========================================================
! REMOVE PIM SPARSE MODE FROM RP LOOPBACK
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM SPARSE MODE FROM SOURCE-SIDE INTERFACE
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM SPARSE MODE FROM TRANSIT INTERFACES
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM SPARSE MODE FROM RECEIVER-FACING INTERFACE
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim sparse-mode
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
! CLEAR MULTICAST STATE
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_Sparse_Mode_RP_And_Shared_Tree_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No `(*,G)` state appears after receiver joins | Last-hop router does not know the RP or IGMP join is missing | Last-hop router | `show ip pim rp mapping`<br>`show ip igmp groups` | Fix RP mapping and receiver IGMP join |
| No PIM neighbors | PIM missing on transit link or interface problem | Transit routers | `show ip pim neighbor`<br>`show ip pim interface` | Enable `ip pim sparse-mode` on both sides and fix interface reachability |
| RP mapping missing | Static RP not configured on all routers | All multicast routers | `show ip pim rp mapping` | Configure `ip pim rp-address <RP_ADDRESS>` consistently |
| Group maps to wrong RP | Overlapping static RP group ranges or wrong ACL | All multicast routers | `show ip pim rp mapping`<br>`show access-lists` | Correct group ACLs and RP order |
| Receiver group missing | Receiver did not send IGMP join or join configured on wrong interface | Last-hop router | `show ip igmp groups` | Configure `ip igmp join-group <MULTICAST_GROUP>` on correct receiver interface |
| Multicast ping gets no reply | Receiver used `ip igmp static-group` instead of `ip igmp join-group` | Receiver/test router | `show running-config interface <RECEIVER_INTERFACE>` | Use `ip igmp join-group` for router-based multicast ping testing |
| `(*,G)` exists but no source traffic arrives | Source is not sending, source-side PIM missing, or RP cannot reach source | RP and source-side router | `show ip mroute <MULTICAST_GROUP>`<br>`show ip rpf <SOURCE_IP>` | Start source traffic and fix source-side PIM/RPF |
| RP sees no `(S,G)` state | First-hop router cannot register source to RP or RP is unreachable | Source-side router and RP | `show ip route <RP_ADDRESS>`<br>`show ip pim tunnel` | Fix route to RP and PIM sparse-mode source-side configuration |
| Incoming interface is wrong for `(*,G)` | RPF route to RP points the wrong way | Last-hop or transit router | `show ip rpf <RP_ADDRESS>`<br>`show ip mroute <MULTICAST_GROUP>` | Correct unicast route toward RP |
| Incoming interface is wrong for `(S,G)` | RPF route to source points the wrong way | RP, transit, or last-hop router | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Correct unicast route toward source or use deliberate multicast RPF override |
| OIL is empty on RP | No downstream receiver join reached the RP | RP router | `show ip mroute <MULTICAST_GROUP>` | Verify last-hop IGMP, PIM join path, and RP mapping |
| OIL is empty on last-hop router | Receiver-facing interface has no IGMP interest | Last-hop router | `show ip igmp groups`<br>`show ip mroute <MULTICAST_GROUP>` | Fix receiver join |
| Sparse mode behaves like no forwarding at all | No RP known for ASM group | All multicast routers | `show ip pim rp mapping` | Configure static RP, Auto-RP, or BSR |
| Dense-mode expectation in sparse-mode lab | Sparse mode will not flood without joins | All routers | `show ip mroute <MULTICAST_GROUP>` | Add receiver joins and RP mapping instead of expecting flood-and-prune |
| PIM register tunnel command shows nothing | Platform does not expose PIM tunnel output or no source has registered yet | Source-side router and RP | `show ip pim tunnel`<br>`show ip mroute <MULTICAST_GROUP>` | Verify source traffic and use `show ip mroute` as primary proof |
| Debug floods console | Debug left enabled during PIM testing | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_Sparse_Mode_RP_And_Shared_Tree_Mental_Model
# PIM_Sparse_Mode_RP_And_Shared_Tree_Configuration_Checklist
# PIM_Sparse_Mode_RP_And_Shared_Tree_Skeleton
# PIM_Sparse_Mode_RP_And_Shared_Tree_Verification_Commands
# PIM_Sparse_Mode_RP_And_Shared_Tree_Rollback
# PIM_Sparse_Mode_RP_And_Shared_Tree_Failure_Checks
