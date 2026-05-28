Cisco IOS XE confirms the core BIDIR syntax used here: ip pim bidir-enable, ip pim rp-address <RP> <ACL> bidir, and show ip pim interface df for designated-forwarder verification.  

PIM_BIDIR_Shared_Tree_And_Designated_Forwarder.md

PIM_BIDIR_Shared_Tree_And_Designated_Forwarder


# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PIM BIDIR | Bidirectional PIM builds a shared tree rooted at the RP and uses that shared tree for traffic in both directions |
| Shared tree only | BIDIR does not switch to a shortest-path tree. Do not expect normal PIM-SM `(S,G)` SPT behavior |
| RP requirement | BIDIR still requires an RP for the group range |
| RP placement | The RP is more important in BIDIR than in normal sparse mode because the shared tree remains the forwarding structure |
| No PIM register | BIDIR does not use the normal PIM-SM source register process |
| No source-specific state goal | BIDIR is designed to reduce `(S,G)` state in many-source environments |
| `(*,G)` state | BIDIR primarily builds shared-tree group state for the multicast group |
| Bidir mroute flag | In `show ip mroute`, the `B` or `bidir` indication shows the group is operating as BIDIR |
| Designated Forwarder | The DF is elected on each multiaccess segment for each RP and controls forwarding responsibility on that segment |
| DF election basis | DF is elected based on the best route or metric toward the RP on that segment |
| Upstream forwarding | If a packet arrives on an interface where the router is DF, it can be forwarded upstream toward the RP |
| Downstream forwarding | If a packet arrives from the RP-facing direction, it is forwarded downstream toward interested receivers |
| DR versus DF | PIM DR represents receiver/source segment responsibility in ASM/SSM. BIDIR uses DF for forwarding responsibility toward the RP tree |
| RPF toward RP | BIDIR forwarding logic is tied to the path toward the RP, not a per-source SPT |
| Best fit | BIDIR fits many-to-many applications where many sources and receivers share the same group range |
| Bad fit | BIDIR is not the clean choice when source-specific control, SSM behavior, or shortest-path optimization is required |
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify RP router, BIDIR group range, source segments, receiver segments, and shared multiaccess segments | All multicast routers | `show ip interface brief` | RP loopback, source-facing, receiver-facing, and transit interfaces are known |
| 2 | Confirm unicast reachability to the RP address | All multicast routers | `show ip route <RP_ADDRESS>` | Every multicast router has a route toward the RP |
| 3 | Confirm unicast reachability from RP to all multicast routers | RP router | `show ip route <ROUTER_LOOPBACK_OR_INTERFACE>` | RP has return reachability through the unicast routing table |
| 4 | Confirm source and receiver interfaces are up | Source-side and receiver-side routers | `show ip interface brief` | Source-facing and receiver-facing interfaces are up/up |
| 5 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 6 | Enable BIDIR PIM globally | All multicast routers | `conf t`<br>`ip pim bidir-enable` | Router supports BIDIR PIM operation |
| 7 | Enable PIM sparse mode on RP loopback | RP router | `conf t`<br>`interface <RP_LOOPBACK_INTERFACE>`<br>`ip pim sparse-mode` | RP address participates in PIM control plane |
| 8 | Enable PIM sparse mode on transit interfaces | All multicast routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-mode` | PIM neighbors can form across routed transit links |
| 9 | Enable PIM sparse mode on source-facing interfaces | First-hop/source-side routers | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-facing interfaces participate in BIDIR forwarding |
| 10 | Enable PIM sparse mode on receiver-facing interfaces | Last-hop/receiver-side routers | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-facing interfaces can process IGMP joins and BIDIR forwarding |
| 11 | Verify PIM interface state | All multicast routers | `show ip pim interface` | RP loopback, transit, source-facing, and receiver-facing interfaces appear |
| 12 | Verify PIM neighbor state | Transit and multiaccess routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| 13 | Create standard ACL for BIDIR group range | All multicast routers | `conf t`<br>`ip access-list standard <BIDIR_GROUP_ACL>`<br>`permit <BIDIR_GROUP_RANGE> <GROUP_WILDCARD>` | ACL matches multicast groups that should operate in BIDIR mode |
| 14 | Configure static BIDIR RP mapping | All multicast routers | `conf t`<br>`ip pim rp-address <RP_ADDRESS> <BIDIR_GROUP_ACL> bidir` | Test group range maps to the RP as BIDIR |
| 15 | Verify BIDIR RP mapping | All multicast routers | `show ip pim rp mapping` | BIDIR group range maps to `<RP_ADDRESS>` |
| 16 | Verify RP route used for DF election | All multicast routers | `show ip route <RP_ADDRESS>`<br>`show ip rpf <RP_ADDRESS>` | RPF path toward RP is stable and expected |
| 17 | Verify DF election on all PIM interfaces | All multicast routers | `show ip pim interface df` | DF winner is shown per interface and RP |
| 18 | Verify local router DF role on a specific interface | Multiaccess segment routers | `show ip pim interface <MULTIACCESS_INTERFACE> df <RP_ADDRESS>` | Output identifies which router is DF for that segment and RP |
| 19 | Configure receiver membership for BIDIR group | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <BIDIR_MULTICAST_GROUP>` | Receiver joins the BIDIR multicast group |
| 20 | Verify receiver membership | Last-hop router | `show ip igmp groups` | BIDIR group appears on receiver-facing interface |
| 21 | Generate multicast traffic from first source | Source/test router 1 | `ping <BIDIR_MULTICAST_GROUP> source <SOURCE_1_INTERFACE_OR_IP> repeat 5` | Source 1 sends traffic to BIDIR group |
| 22 | Generate multicast traffic from second source if testing many-to-many behavior | Source/test router 2 | `ping <BIDIR_MULTICAST_GROUP> source <SOURCE_2_INTERFACE_OR_IP> repeat 5` | Source 2 sends traffic to same BIDIR group |
| 23 | Verify BIDIR mroute state | All multicast routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | `(*,G)` state exists and group is marked BIDIR |
| 24 | Confirm no normal PIM-SM register tunnel behavior is required | First-hop routers and RP | `show ip pim tunnel` | No normal PIM register dependency is required for BIDIR forwarding |
| 25 | Confirm forwarding state follows RP shared tree | Transit and last-hop routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | Incoming interface and OIL reflect RP-rooted shared tree behavior |
| 26 | Confirm packet counters increment | All forwarding routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | Packet counters increment during source traffic |
| 27 | Confirm source-specific SPT state is not the design goal | All multicast routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | BIDIR group should not depend on normal `(S,G)` SPT switchover |
| 28 | Clear multicast state for clean retest | All multicast routers | `clear ip mroute *` | Old dynamic multicast state is removed |
| 29 | Retest receiver joins and source traffic | Receiver and source routers | `show ip igmp groups`<br>`ping <BIDIR_MULTICAST_GROUP> source <SOURCE_1_INTERFACE_OR_IP> repeat 5` | BIDIR group state rebuilds and forwards correctly |
| 30 | Save the working BIDIR configuration | All changed routers | `copy running-config startup-config` | BIDIR shared-tree and DF baseline is preserved |
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RP_ADDRESS>                  = RP loopback or stable RP address
! <RP_LOOPBACK_INTERFACE>       = RP loopback interface
! <BIDIR_GROUP_ACL>             = standard ACL matching BIDIR group range
! <BIDIR_GROUP_RANGE>           = multicast group range that should use BIDIR
! <GROUP_WILDCARD>              = wildcard mask for BIDIR group range
! <BIDIR_MULTICAST_GROUP>       = test multicast group inside BIDIR range
! <TRANSIT_INTERFACE>           = routed interface between PIM routers
! <MULTIACCESS_INTERFACE>       = shared LAN segment where DF election is tested
! <SOURCE_FACING_INTERFACE>     = first-hop router interface toward source
! <RECEIVER_FACING_INTERFACE>   = last-hop router interface toward receivers
! <RECEIVER_INTERFACE>          = test receiver interface
! <SOURCE_1_INTERFACE_OR_IP>    = first source interface or IP
! <SOURCE_2_INTERFACE_OR_IP>    = optional second source interface or IP
! =========================================================
! GLOBAL MULTICAST AND BIDIR ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
 ip pim bidir-enable
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
! TRANSIT ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
 interface <MULTIACCESS_INTERFACE>
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
! STATIC BIDIR RP MAPPING
! Apply consistently on all multicast routers.
! Standard ACL matches multicast group addresses.
! =========================================================
conf t
 ip access-list standard <BIDIR_GROUP_ACL>
  permit <BIDIR_GROUP_RANGE> <GROUP_WILDCARD>
 exit
 ip pim rp-address <RP_ADDRESS> <BIDIR_GROUP_ACL> bidir
end
! =========================================================
! RECEIVER JOIN
! Use join-group when a router is emulating a receiver
! and should respond to multicast ping tests.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <BIDIR_MULTICAST_GROUP>
 exit
end
! =========================================================
! TEST TRAFFIC FROM ONE OR MORE SOURCES
! =========================================================
ping <BIDIR_MULTICAST_GROUP> source <SOURCE_1_INTERFACE_OR_IP> repeat 5
ping <BIDIR_MULTICAST_GROUP> source <SOURCE_2_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show ip pim interface
show ip pim neighbor
show ip pim rp mapping
show ip pim interface df
show ip pim interface <MULTIACCESS_INTERFACE> df <RP_ADDRESS>
show ip igmp groups
show ip rpf <RP_ADDRESS>
show ip mroute <BIDIR_MULTICAST_GROUP>
show ip pim tunnel
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| BIDIR globally enabled | All multicast routers | `show running-config | include ip pim bidir-enable` | `ip pim bidir-enable` is present or platform default behavior supports BIDIR |
| PIM interface state | All multicast routers | `show ip pim interface` | Required RP, transit, source-facing, and receiver-facing interfaces appear |
| PIM neighbor state | Transit and multiaccess routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| BIDIR group ACL | All multicast routers | `show access-lists <BIDIR_GROUP_ACL>` | ACL permits the intended BIDIR group range |
| BIDIR RP mapping config | All multicast routers | `show running-config | include rp-address` | `ip pim rp-address <RP_ADDRESS> <ACL> bidir` appears |
| RP mapping verification | All multicast routers | `show ip pim rp mapping` | BIDIR group maps to the expected RP |
| Route to RP | All multicast routers | `show ip route <RP_ADDRESS>` | Route toward RP exists |
| RPF toward RP | All multicast routers | `show ip rpf <RP_ADDRESS>` | RPF interface points toward RP |
| DF election summary | All multicast routers | `show ip pim interface df` | DF winner appears per interface and RP |
| Interface-specific DF election | Multiaccess routers | `show ip pim interface <MULTIACCESS_INTERFACE> df <RP_ADDRESS>` | Expected DF winner is shown for the segment |
| Receiver membership | Last-hop router | `show ip igmp groups` | BIDIR group appears on receiver-facing interface |
| Local receiver join | Receiver/test router | `show ip igmp groups` | Local `join-group` appears with `never` expiration |
| BIDIR mroute state | All multicast routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | `(*,G)` state exists and group is marked BIDIR |
| Source traffic test 1 | Source/test router 1 | `ping <BIDIR_MULTICAST_GROUP> source <SOURCE_1_INTERFACE_OR_IP> repeat 5` | Receiver replies or counters increment |
| Source traffic test 2 | Source/test router 2 | `ping <BIDIR_MULTICAST_GROUP> source <SOURCE_2_INTERFACE_OR_IP> repeat 5` | Receiver replies or counters increment |
| Packet counters | All forwarding routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | Packet counters increment during source traffic |
| Register tunnel check | First-hop routers and RP | `show ip pim tunnel` | Normal PIM-SM register dependency is absent or not used for BIDIR forwarding |
| MFIB state if supported | All forwarding routers | `show ip mfib <BIDIR_MULTICAST_GROUP>` | Forwarding entry exists for BIDIR group |
| Debug cleanup | All routers | `show debugging`<br>`undebug all` | No multicast debug remains active |
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Rollback
! =========================================================
! REMOVE RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <BIDIR_MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE BIDIR STATIC RP MAPPING
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS> <BIDIR_GROUP_ACL> bidir
end
! =========================================================
! REMOVE BIDIR GROUP ACL
! =========================================================
conf t
 no ip access-list standard <BIDIR_GROUP_ACL>
end
! =========================================================
! DISABLE BIDIR PIM
! Only do this if no other BIDIR groups depend on it.
! =========================================================
conf t
 no ip pim bidir-enable
end
! =========================================================
! REMOVE PIM FROM RP LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM FROM TRANSIT AND MULTIACCESS INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <MULTIACCESS_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM FROM SOURCE AND RECEIVER FACING INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
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
! CLEAR MULTICAST STATE AND DEBUGGING
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| BIDIR RP mapping does not appear | `bidir` keyword missing from RP mapping or ACL does not match group | All multicast routers | `show ip pim rp mapping`<br>`show access-lists <BIDIR_GROUP_ACL>` | Configure `ip pim rp-address <RP_ADDRESS> <ACL> bidir` and correct the ACL |
| Group behaves like normal sparse mode | Group is mapped to RP without `bidir` keyword | All multicast routers | `show running-config | include rp-address`<br>`show ip mroute <GROUP>` | Reconfigure RP mapping with `bidir` for the BIDIR group range |
| No DF information appears | BIDIR not enabled or group/RP mapping is missing | Router on multiaccess segment | `show ip pim interface df`<br>`show ip pim rp mapping` | Configure `ip pim bidir-enable` and valid BIDIR RP mapping |
| Wrong router is DF | Route metric toward RP makes another router the best DF winner | Multiaccess segment routers | `show ip pim interface <MULTIACCESS_INTERFACE> df <RP_ADDRESS>`<br>`show ip route <RP_ADDRESS>` | Adjust unicast routing metrics toward RP if deterministic DF is required |
| PIM neighbors missing | PIM missing on one side, interface down, or L3 adjacency issue | Transit routers | `show ip pim neighbor`<br>`show ip interface brief` | Enable PIM on both sides and fix interface reachability |
| Receiver group missing | Receiver did not join or joined on wrong interface | Last-hop router and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <BIDIR_MULTICAST_GROUP>` on correct receiver interface |
| Multicast traffic does not flow upstream toward RP | Router receiving source traffic is not DF on that segment or RPF toward RP is wrong | Source-side router | `show ip pim interface df`<br>`show ip rpf <RP_ADDRESS>` | Fix DF election or route toward RP |
| Multicast traffic does not flow downstream to receiver | Receiver OIL missing, IGMP state absent, or downstream PIM path broken | Last-hop and transit routers | `show ip igmp groups`<br>`show ip mroute <BIDIR_MULTICAST_GROUP>` | Fix receiver join, PIM path, or OIL state |
| Source-specific `(S,G)` SPT expected but not shown | BIDIR does not use normal PIM-SM SPT switchover | All multicast routers | `show ip mroute <BIDIR_MULTICAST_GROUP>` | No fix. Use normal PIM-SM or SSM if source-specific SPT behavior is required |
| PIM register tunnel expected but absent | BIDIR does not use the normal PIM-SM register process | First-hop routers and RP | `show ip pim tunnel` | No fix. This is expected BIDIR behavior |
| RP outage causes design confusion | BIDIR can keep some existing forwarding behavior, but RP reachability is still central to tree building and DF calculation | All routers | `show ip route <RP_ADDRESS>`<br>`show ip pim rp mapping`<br>`show ip pim interface df` | Restore RP reachability and stable RP mapping |
| RP placement causes bad forwarding path | BIDIR shared tree keeps RP in the forwarding model | Network design | `show ip mroute <GROUP>`<br>`traceroute <RP_ADDRESS>` | Move RP closer to traffic center or choose a better RP location |
| Static RP conflicts with Auto-RP or BSR mapping | Multiple RP discovery methods overlap for the same group | All multicast routers | `show ip pim rp mapping`<br>`show running-config | include rp-address|rp-candidate|send-rp` | Keep RP mapping method consistent and avoid overlapping group ranges |
| ACL matches receiver/source addresses instead of multicast groups | BIDIR group ACL was written for host IPs instead of group range | All multicast routers | `show access-lists <BIDIR_GROUP_ACL>` | Rewrite ACL to match multicast group addresses |
| Packet counters stay at zero | No source traffic, no receiver interest, or forwarding path broken | Source, RP, and last-hop routers | `show ip mroute <BIDIR_MULTICAST_GROUP>`<br>`show ip igmp groups` | Generate traffic and fix receiver/PIM/DF state |
| Debug floods the console | Multicast debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Mental_Model
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Configuration_Checklist
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Skeleton
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Verification_Commands
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Rollback
# PIM_BIDIR_Shared_Tree_And_Designated_Forwarder_Failure_Checks

