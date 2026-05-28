
PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback.md

PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback



# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Sparse-dense mode | Interface mode where PIM uses sparse mode for groups with known RP mapping and dense mode for groups with no known RP |
| Known RP group | If `show ip pim rp mapping` shows an RP for the group, the router treats the group as sparse-mode traffic |
| Unknown RP group | If no RP is known for the group, sparse-dense mode falls back to dense-mode flood-and-prune behavior |
| Dense fallback | Dense fallback prevents traffic from silently failing when no RP is known, but it can also cause unwanted flooding |
| Auto-RP dependency | Auto-RP uses 224.0.1.39 for RP-Announce and 224.0.1.40 for RP-Discovery |
| Auto-RP bootstrap problem | Routers need RP discovery messages to learn the RP, but classic sparse mode needs an RP before joining multicast groups |
| Why sparse-dense exists | Sparse-dense mode lets Auto-RP discovery groups move in dense mode before RP mapping exists |
| Auto-RP listener alternative | `ip pim autorp listener` lets routers use sparse mode interfaces while still dense-flooding Auto-RP discovery groups |
| Candidate RP | Router that advertises itself as an RP for one or more multicast group ranges using RP-Announce messages |
| Mapping agent | Router that listens to candidate RP announcements and advertises selected RP-to-group mappings using RP-Discovery messages |
| Mapping agent election logic | If multiple candidate RPs advertise the same group range, the mapping agent selects the RP with the highest RP address |
| Risk | Sparse-dense mode is convenient for Auto-RP labs, but dangerous in production-like designs because unknown groups may flood densely |
| Verification truth | `show ip pim rp mapping` proves whether a group should run sparse mode. `show ip mroute` proves whether traffic built shared-tree or dense-mode source-tree state |
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify source, receiver, transit, RP, and optional mapping-agent routers | All multicast routers | `show ip interface brief` | Loopbacks, transit links, source-facing interfaces, and receiver-facing interfaces are known |
| 2 | Confirm unicast reachability to the RP and mapping agent loopbacks | All multicast routers | `show ip route <RP_ADDRESS>`<br>`show ip route <MAPPING_AGENT_ADDRESS>` | All routers can reach RP and mapping-agent addresses |
| 3 | Confirm unicast reachability to the source | RP, transit, and last-hop routers | `show ip route <SOURCE_IP>` | Routers have a route back toward the multicast source |
| 4 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 5 | Enable PIM sparse-dense mode on RP loopback | RP router | `conf t`<br>`interface <RP_LOOPBACK_INTERFACE>`<br>`ip pim sparse-dense-mode` | RP loopback can source or participate in multicast control traffic |
| 6 | Enable PIM sparse-dense mode on mapping-agent loopback if Auto-RP is tested | Mapping-agent router | `conf t`<br>`interface <MAPPING_AGENT_LOOPBACK>`<br>`ip pim sparse-dense-mode` | Mapping-agent loopback can source Auto-RP discovery |
| 7 | Enable PIM sparse-dense mode on source-facing interface | First-hop/source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-dense-mode` | Source-facing interface participates in sparse-dense multicast |
| 8 | Enable PIM sparse-dense mode on transit interfaces | Transit routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-dense-mode` | PIM neighbors can form across routed transit links |
| 9 | Enable PIM sparse-dense mode on receiver-facing interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-dense-mode` | Receiver-facing interface can process IGMP joins and PIM joins |
| 10 | Verify PIM interface mode | All multicast routers | `show ip pim interface` | Required interfaces appear as PIM interfaces |
| 11 | Verify PIM neighbors | Transit routers | `show ip pim neighbor` | Expected neighbors appear on transit links |
| 12 | Create RP group ACL for known sparse-mode group range | All multicast routers or candidate RPs | `conf t`<br>`access-list <RP_GROUP_ACL> permit <KNOWN_RP_GROUP_RANGE> <GROUP_WILDCARD>` | ACL matches groups that should use the RP |
| 13 | Configure static RP mapping for a simple sparse-dense test | All multicast routers | `conf t`<br>`ip pim rp-address <RP_ADDRESS> <RP_GROUP_ACL>` | Known group range maps to the RP |
| 14 | Verify known group RP mapping | All multicast routers | `show ip pim rp mapping` | `<KNOWN_RP_GROUP>` maps to `<RP_ADDRESS>` |
| 15 | Verify unknown fallback group has no RP mapping | All multicast routers | `show ip pim rp mapping` | `<UNKNOWN_RP_GROUP>` is not covered by any RP mapping |
| 16 | Join a known-RP multicast group from the receiver side | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <KNOWN_RP_GROUP>` | Receiver joins a group that should operate in sparse mode |
| 17 | Generate traffic to the known-RP group | Source/test router | `ping <KNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast traffic is generated for the sparse-mode group |
| 18 | Verify known group uses sparse-mode behavior | Last-hop, RP, and transit routers | `show ip mroute <KNOWN_RP_GROUP>` | `(*,G)` state points toward the RP and `(S,G)` state appears after source traffic |
| 19 | Join an unknown-RP multicast group from the receiver side | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <UNKNOWN_RP_GROUP>` | Receiver joins a group with no RP mapping |
| 20 | Generate traffic to the unknown-RP group | Source/test router | `ping <UNKNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast traffic is generated for dense fallback testing |
| 21 | Verify unknown group falls back to dense-mode behavior | All multicast routers | `show ip mroute <UNKNOWN_RP_GROUP>` | `(S,G)` state appears and traffic behaves like dense-mode flood-and-prune |
| 22 | Configure candidate RP for Auto-RP testing if the lab requires dynamic RP discovery | Candidate RP router | `conf t`<br>`access-list <AUTO_RP_GROUP_ACL> permit <AUTO_RP_GROUP_RANGE> <GROUP_WILDCARD>`<br>`ip pim send-rp-announce <RP_LOOPBACK_INTERFACE> scope <TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>` | Candidate RP sends RP-Announce messages to 224.0.1.39 |
| 23 | Configure mapping agent for Auto-RP testing | Mapping-agent router | `conf t`<br>`ip pim send-rp-discovery <MAPPING_AGENT_LOOPBACK> scope <TTL_SCOPE>` | Mapping agent sends RP-Discovery messages to 224.0.1.40 |
| 24 | Verify Auto-RP learned mappings | All multicast routers | `show ip pim rp mapping` | Group range shows RP elected via Auto-RP with mapping-agent info source |
| 25 | Verify Auto-RP discovery groups exist when Auto-RP is used | All multicast routers | `show ip mroute 224.0.1.39`<br>`show ip mroute 224.0.1.40` | Auto-RP announcement and discovery group state exists |
| 26 | Use `autorp listener` only if the design uses sparse-mode interfaces instead of sparse-dense mode | All multicast routers | `conf t`<br>`ip pim autorp listener` | Auto-RP groups can be dense-flooded while normal interfaces remain sparse mode |
| 27 | Clear stale multicast state after changing RP discovery mode | All multicast routers | `clear ip mroute *` | Old multicast forwarding state is removed |
| 28 | Retest known-RP group forwarding | Source/test router | `ping <KNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Known group forwards through sparse-mode/RP logic |
| 29 | Retest unknown-RP fallback behavior | Source/test router | `ping <UNKNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Unknown group still falls back to dense-mode behavior unless RP mapping now covers it |
| 30 | Verify RPF toward RP and source | Last-hop, RP, and transit routers | `show ip rpf <RP_ADDRESS>`<br>`show ip rpf <SOURCE_IP>` | RPF paths are correct for shared-tree and source-tree state |
| 31 | Save the working sparse-dense baseline | All changed routers | `copy running-config startup-config` | Sparse-dense mode and fallback lab configuration is preserved |
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RP_ADDRESS>                  = RP loopback or stable RP address
! <RP_LOOPBACK_INTERFACE>       = RP loopback interface
! <MAPPING_AGENT_ADDRESS>       = mapping agent loopback address
! <MAPPING_AGENT_LOOPBACK>      = mapping agent loopback interface
! <SOURCE_IP>                   = multicast source IP
! <SOURCE_INTERFACE_OR_IP>      = source interface or IP for multicast test traffic
! <SOURCE_FACING_INTERFACE>     = source-side interface toward source
! <TRANSIT_INTERFACE>           = routed interface between PIM routers
! <RECEIVER_FACING_INTERFACE>   = last-hop interface toward receiver LAN
! <RECEIVER_INTERFACE>          = test receiver interface
! <RP_GROUP_ACL>                = ACL matching statically mapped RP group range
! <AUTO_RP_GROUP_ACL>           = ACL matching Auto-RP advertised group range
! <KNOWN_RP_GROUP_RANGE>        = multicast range with known RP mapping
! <AUTO_RP_GROUP_RANGE>         = multicast range advertised by candidate RP
! <GROUP_WILDCARD>              = wildcard mask for multicast group range
! <KNOWN_RP_GROUP>              = group that should use sparse mode
! <UNKNOWN_RP_GROUP>            = group with no RP mapping, used to test dense fallback
! <TTL_SCOPE>                   = Auto-RP TTL scope value
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! PIM SPARSE-DENSE MODE ON ALL MULTICAST INTERFACES
! Apply on source-facing, receiver-facing, transit, RP loopback,
! and mapping-agent loopback interfaces as required.
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  ip pim sparse-dense-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-dense-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-dense-mode
 exit
end
! =========================================================
! SIMPLE STATIC RP MAPPING
! Used to prove known groups use sparse mode.
! Apply consistently on all multicast routers.
! =========================================================
conf t
 access-list <RP_GROUP_ACL> permit <KNOWN_RP_GROUP_RANGE> <GROUP_WILDCARD>
 ip pim rp-address <RP_ADDRESS> <RP_GROUP_ACL>
end
! =========================================================
! RECEIVER JOIN FOR KNOWN RP GROUP
! This group should use sparse-mode behavior.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <KNOWN_RP_GROUP>
 exit
end
! =========================================================
! RECEIVER JOIN FOR UNKNOWN RP GROUP
! This group should fall back to dense-mode behavior.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <UNKNOWN_RP_GROUP>
 exit
end
! =========================================================
! AUTO-RP CANDIDATE RP
! Use when testing Auto-RP over sparse-dense mode.
! =========================================================
conf t
 access-list <AUTO_RP_GROUP_ACL> permit <AUTO_RP_GROUP_RANGE> <GROUP_WILDCARD>
 ip pim send-rp-announce <RP_LOOPBACK_INTERFACE> scope <TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! AUTO-RP MAPPING AGENT
! Use when testing Auto-RP over sparse-dense mode.
! =========================================================
conf t
 interface <MAPPING_AGENT_LOOPBACK>
  ip pim sparse-dense-mode
 exit
 ip pim send-rp-discovery <MAPPING_AGENT_LOOPBACK> scope <TTL_SCOPE>
end
! =========================================================
! ALTERNATIVE TO SPARSE-DENSE FOR AUTO-RP
! Use only when interfaces are configured sparse-mode and
! Auto-RP discovery still needs to work.
! =========================================================
conf t
 ip pim autorp listener
end
! =========================================================
! TEST TRAFFIC
! =========================================================
ping <KNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
ping <UNKNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show ip pim interface
show ip pim neighbor
show ip pim rp mapping
show ip mroute <KNOWN_RP_GROUP>
show ip mroute <UNKNOWN_RP_GROUP>
show ip mroute 224.0.1.39
show ip mroute 224.0.1.40
show ip rpf <RP_ADDRESS>
show ip rpf <SOURCE_IP>
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| Sparse-dense interface config | All multicast routers | `show running-config interface <INTERFACE>` | Interface contains `ip pim sparse-dense-mode` |
| PIM interface state | All multicast routers | `show ip pim interface` | Required interfaces appear as PIM interfaces |
| PIM neighbors | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear on routed transit links |
| Static RP config | All multicast routers | `show running-config | include ip pim rp-address` | Expected RP mapping command appears |
| RP mapping | All multicast routers | `show ip pim rp mapping` | Known group range maps to expected RP |
| Known group sparse behavior | Last-hop, RP, and transit routers | `show ip mroute <KNOWN_RP_GROUP>` | `(*,G)` state exists toward RP and `(S,G)` appears after source traffic |
| Unknown group fallback behavior | All multicast routers | `show ip mroute <UNKNOWN_RP_GROUP>` | Group has no RP mapping and builds dense-mode style `(S,G)` state |
| RPF toward RP | Last-hop and transit routers | `show ip rpf <RP_ADDRESS>` | RPF path points toward RP for known sparse group shared-tree state |
| RPF toward source | RP, transit, and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF path points toward source for source-specific traffic |
| Auto-RP candidate config | Candidate RP | `show running-config | include send-rp-announce` | Candidate RP advertises the intended group range |
| Auto-RP mapping agent config | Mapping-agent router | `show running-config | include send-rp-discovery` | Mapping agent advertises RP discovery messages |
| Auto-RP learned mapping | All multicast routers | `show ip pim rp mapping` | Group range shows RP elected via Auto-RP |
| Auto-RP announce group | Multicast routers | `show ip mroute 224.0.1.39` | RP-Announce group state exists when Auto-RP is running |
| Auto-RP discovery group | Multicast routers | `show ip mroute 224.0.1.40` | RP-Discovery group state exists when Auto-RP is running |
| Auto-RP listener config | All multicast routers | `show running-config | include autorp listener` | `ip pim autorp listener` is present only if sparse-mode Auto-RP workaround is used |
| Known group traffic test | Source/test router | `ping <KNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies or multicast counters increment |
| Unknown group traffic test | Source/test router | `ping <UNKNOWN_RP_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Fallback group forwards through dense-mode behavior if receiver/path exists |
| Packet counters | All multicast routers | `show ip mroute <KNOWN_RP_GROUP>`<br>`show ip mroute <UNKNOWN_RP_GROUP>` | Counters increment during active traffic |
| Auto-RP debug | Candidate RP and mapping agent | `debug ip pim auto-rp` | RP-Announce and RP-Discovery messages appear during controlled test |
| Debug cleanup | All routers | `show debugging`<br>`undebug all` | No multicast debug remains active |
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Rollback
! =========================================================
! REMOVE RECEIVER JOINS
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <KNOWN_RP_GROUP>
  no ip igmp join-group <UNKNOWN_RP_GROUP>
 exit
end
! =========================================================
! REMOVE AUTO-RP CANDIDATE RP CONFIGURATION
! =========================================================
conf t
 no ip pim send-rp-announce <RP_LOOPBACK_INTERFACE> scope <TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! REMOVE AUTO-RP MAPPING AGENT CONFIGURATION
! =========================================================
conf t
 no ip pim send-rp-discovery <MAPPING_AGENT_LOOPBACK> scope <TTL_SCOPE>
end
! =========================================================
! REMOVE AUTO-RP LISTENER IF USED
! =========================================================
conf t
 no ip pim autorp listener
end
! =========================================================
! REMOVE STATIC RP CONFIGURATION
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS> <RP_GROUP_ACL>
 no ip pim rp-address <RP_ADDRESS>
end
! =========================================================
! REMOVE GROUP ACLS
! =========================================================
conf t
 no access-list <RP_GROUP_ACL>
 no access-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! REMOVE PIM SPARSE-DENSE MODE FROM LAB INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  no ip pim sparse-dense-mode
 exit
 interface <MAPPING_AGENT_LOOPBACK>
  no ip pim sparse-dense-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
  no ip pim sparse-dense-mode
 exit
 interface <TRANSIT_INTERFACE>
  no ip pim sparse-dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim sparse-dense-mode
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
! CLEAR MULTICAST STATE AND DEBUGGING
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Known group behaves like dense mode | RP mapping missing for the group | All multicast routers | `show ip pim rp mapping` | Configure `ip pim rp-address <RP_ADDRESS> <ACL>` or fix Auto-RP/BSR mapping |
| Unknown group floods unexpectedly | Sparse-dense mode fell back to dense mode because no RP was known | All multicast routers | `show ip pim rp mapping`<br>`show ip mroute <UNKNOWN_RP_GROUP>` | Add RP mapping for the group or use pure sparse mode instead of sparse-dense mode |
| No Auto-RP mappings appear | RP-Announce or RP-Discovery messages are not reaching routers | Candidate RP, mapping agent, and transit routers | `show ip mroute 224.0.1.39`<br>`show ip mroute 224.0.1.40` | Use `ip pim sparse-dense-mode` on required interfaces or configure `ip pim autorp listener` |
| Auto-RP fails with sparse-mode interfaces | 224.0.1.39 and 224.0.1.40 discovery groups cannot bootstrap through pure sparse mode | All multicast routers | `show ip pim interface`<br>`show ip pim rp mapping` | Change interfaces to `ip pim sparse-dense-mode` or configure `ip pim autorp listener` |
| Candidate RP command fails | PIM is not enabled on the loopback used as RP-Announce source | Candidate RP | `show running-config interface <RP_LOOPBACK_INTERFACE>` | Configure `ip pim sparse-dense-mode` on the loopback |
| Mapping agent does not advertise mappings | `ip pim send-rp-discovery` missing or mapping agent loopback lacks PIM | Mapping-agent router | `show running-config | include send-rp-discovery`<br>`show ip pim interface` | Configure mapping agent loopback with PIM and add `ip pim send-rp-discovery` |
| Wrong RP is elected by Auto-RP | Mapping agent selected the highest RP address for overlapping group ranges | Mapping-agent router | `show ip pim rp mapping` | Change candidate RP addresses, group ranges, or use static RP/BSR if deterministic priority is needed |
| Group maps to unexpected RP | Overlapping RP ACLs or Auto-RP advertised ranges overlap | All multicast routers | `show ip pim rp mapping`<br>`show access-lists` | Correct group-list ACLs and remove unintended overlap |
| Receiver group missing | Receiver did not join or joined on wrong interface | Last-hop router and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <GROUP>` on correct receiver interface |
| Known group has `(*,G)` but no traffic | Source is not sending, source registration fails, or RPF is broken | RP and first-hop router | `show ip mroute <KNOWN_RP_GROUP>`<br>`show ip rpf <SOURCE_IP>` | Generate source traffic and fix PIM/RPF path |
| Unknown fallback group has no forwarding | Dense-mode path is pruned, RPF is wrong, or receiver interest is absent | All multicast routers | `show ip mroute <UNKNOWN_RP_GROUP>`<br>`show ip rpf <SOURCE_IP>` | Fix receiver join, PIM neighbors, and RPF path |
| PIM neighbor missing | PIM not enabled on one side or interface reachability is broken | Transit routers | `show ip pim neighbor`<br>`show ip interface brief` | Enable PIM on both sides and fix interface reachability |
| Incoming interface is wrong | RPF route to source or RP points to wrong interface | Transit and last-hop routers | `show ip rpf <SOURCE_IP>`<br>`show ip rpf <RP_ADDRESS>` | Correct unicast routing or use deliberate multicast RPF override |
| Dense fallback is considered unacceptable | Sparse-dense mode is the wrong design for unknown groups | Network design | `show ip pim rp mapping`<br>`show ip mroute` | Use pure sparse mode with explicit RP mapping or SSM |
| Debug floods the console | Auto-RP or PIM debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Mental_Model
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Configuration_Checklist
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Skeleton
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Verification_Commands
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Rollback
# PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback_Failure_Checks

