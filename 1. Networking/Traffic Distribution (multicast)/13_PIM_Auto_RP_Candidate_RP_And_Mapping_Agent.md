

PIM_Auto_RP_Candidate_RP_And_Mapping_Agent.md

PIM_Auto_RP_Candidate_RP_And_Mapping_Agent

# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Auto-RP | Cisco dynamic RP discovery mechanism for PIM sparse-mode multicast |
| Candidate RP | Router that advertises itself as an RP for one or more multicast group ranges |
| RP-Announce | Candidate RP sends RP-Announce messages to 224.0.1.39 |
| Mapping agent | Router that listens to candidate RP announcements, selects RP-to-group mappings, and advertises the result |
| RP-Discovery | Mapping agent sends RP-Discovery messages to 224.0.1.40 |
| Group-list ACL | Standard ACL used by candidate RP to define which multicast group ranges it advertises |
| Scope TTL | Limits how far Auto-RP announce or discovery messages travel |
| Candidate RP source interface | Usually a loopback. The interface used as the source of RP-Announce must have PIM enabled |
| Mapping agent source interface | Usually a loopback. The interface used as the source of RP-Discovery must have PIM enabled |
| Election rule | If multiple candidate RPs advertise the same group range, the mapping agent chooses the highest RP source IP address |
| Backup RP implication | With Auto-RP, backup behavior is not priority-based. The higher candidate RP address wins for overlapping ranges |
| Sparse-dense requirement | Classic Auto-RP needs dense-mode behavior for 224.0.1.39 and 224.0.1.40 before RP mapping exists |
| Auto-RP listener | `ip pim autorp listener` allows Auto-RP discovery groups to flood while normal interfaces can remain sparse mode |
| RP mapping truth | `show ip pim rp mapping` proves which RP is elected, which group range it owns, and which mapping agent advertised it |
| Discovery group truth | `show ip mroute 224.0.1.39` and `show ip mroute 224.0.1.40` prove Auto-RP control groups are moving |
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify candidate RP routers, mapping-agent router, source routers, receiver routers, and group range | All multicast routers | `show ip interface brief` | RP loopbacks, mapping-agent loopback, transit links, source-facing, and receiver-facing interfaces are known |
| 2 | Confirm unicast reachability to candidate RP loopbacks | All multicast routers | `show ip route <CANDIDATE_RP_ADDRESS>` | Every router has a route to each candidate RP address |
| 3 | Confirm unicast reachability to the mapping-agent loopback | All multicast routers | `show ip route <MAPPING_AGENT_ADDRESS>` | Every router has a route to the mapping-agent address |
| 4 | Confirm unicast reachability to the multicast source | RP and last-hop routers | `show ip route <SOURCE_IP>` | Routers have a unicast route toward the multicast source |
| 5 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 6 | Enable PIM sparse-dense mode on candidate RP loopback | Candidate RP router | `conf t`<br>`interface <CANDIDATE_RP_LOOPBACK>`<br>`ip pim sparse-dense-mode` | Candidate RP loopback can source RP-Announce messages |
| 7 | Enable PIM sparse-dense mode on mapping-agent loopback | Mapping-agent router | `conf t`<br>`interface <MAPPING_AGENT_LOOPBACK>`<br>`ip pim sparse-dense-mode` | Mapping-agent loopback can source RP-Discovery messages |
| 8 | Enable PIM sparse-dense mode on transit interfaces | All multicast transit routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-dense-mode` | Auto-RP control groups and multicast data can cross transit links |
| 9 | Enable PIM sparse-dense mode on source-facing interface | First-hop/source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-dense-mode` | Source-side router participates in PIM |
| 10 | Enable PIM sparse-dense mode on receiver-facing interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-dense-mode` | Last-hop router can process IGMP joins and build PIM state |
| 11 | Verify PIM interface state | All multicast routers | `show ip pim interface` | Required loopbacks, transit links, source-facing, and receiver-facing interfaces appear |
| 12 | Verify PIM neighbors on transit links | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| 13 | Create group-list ACL for the advertised RP group range | Candidate RP router | `conf t`<br>`access-list <AUTO_RP_GROUP_ACL> permit <MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>` | ACL matches the multicast group range advertised by the candidate RP |
| 14 | Configure primary candidate RP announcement | Primary candidate RP | `conf t`<br>`ip pim send-rp-announce <PRIMARY_RP_SOURCE_INTERFACE> scope <ANNOUNCE_TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>` | Primary candidate RP sends RP-Announce messages to 224.0.1.39 |
| 15 | Configure backup candidate RP announcement if the lab has redundant RPs | Backup candidate RP | `conf t`<br>`ip pim send-rp-announce <BACKUP_RP_SOURCE_INTERFACE> scope <ANNOUNCE_TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>` | Backup candidate RP also advertises the same group range |
| 16 | Confirm primary RP address is higher if it must win Auto-RP election | Candidate RPs | `show ip interface brief` | Primary RP source address is higher than backup RP source address for overlapping group range |
| 17 | Configure the mapping agent | Mapping-agent router | `conf t`<br>`ip pim send-rp-discovery <MAPPING_AGENT_SOURCE_INTERFACE> scope <DISCOVERY_TTL_SCOPE>` | Mapping agent sends RP-Discovery messages to 224.0.1.40 |
| 18 | Verify RP-Announce multicast state | Candidate RPs, mapping agent, and transit routers | `show ip mroute 224.0.1.39` | RP-Announce group state exists and reaches the mapping agent |
| 19 | Verify RP-Discovery multicast state | Mapping agent and downstream routers | `show ip mroute 224.0.1.40` | RP-Discovery group state exists and reaches multicast routers |
| 20 | Verify learned Auto-RP mapping | All multicast routers | `show ip pim rp mapping` | Group range maps to the elected RP with info source shown as the mapping agent |
| 21 | Verify candidate RP self-identification | Candidate RP router | `show ip pim rp mapping` | Output shows `This system is an RP (Auto-RP)` |
| 22 | Verify mapping-agent self-identification | Mapping-agent router | `show ip pim rp mapping` | Output shows `This system is an RP-mapping agent` |
| 23 | Configure Auto-RP listener only if using sparse-mode interfaces instead of sparse-dense mode | All multicast routers | `conf t`<br>`ip pim autorp listener` | Auto-RP groups can flood while normal interfaces remain sparse mode |
| 24 | Clear stale multicast state after Auto-RP changes | All multicast routers | `clear ip mroute *` | Old multicast state is removed |
| 25 | Reverify RP mapping after state refresh | All multicast routers | `show ip pim rp mapping` | Auto-RP mapping returns and expiration timer refreshes |
| 26 | Configure receiver membership for a group inside the Auto-RP range | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins a group covered by Auto-RP mapping |
| 27 | Verify receiver membership on the last-hop router | Last-hop router | `show ip igmp groups` | Joined group appears on the receiver-facing interface |
| 28 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends multicast traffic to the Auto-RP learned group |
| 29 | Verify sparse-mode forwarding state | RP, transit, and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` state exists and OIL points toward receivers |
| 30 | Verify RPF toward RP and source | Last-hop, RP, and transit routers | `show ip rpf <ELECTED_RP_ADDRESS>`<br>`show ip rpf <SOURCE_IP>` | RPF paths are correct for shared-tree and source-tree state |
| 31 | Use controlled Auto-RP debugging if mapping is not visible | Candidate RP and mapping agent | `debug ip pim auto-rp` | RP-Announce and RP-Discovery messages are visible |
| 32 | Disable debugging after the test | All routers | `undebug all` | Debugging is disabled |
| 33 | Save the working Auto-RP configuration | All changed routers | `copy running-config startup-config` | Auto-RP candidate RP and mapping-agent baseline is preserved |
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <CANDIDATE_RP_ADDRESS>          = candidate RP loopback address
! <PRIMARY_RP_SOURCE_INTERFACE>   = primary candidate RP loopback/interface
! <BACKUP_RP_SOURCE_INTERFACE>    = backup candidate RP loopback/interface
! <CANDIDATE_RP_LOOPBACK>         = candidate RP loopback interface
! <MAPPING_AGENT_ADDRESS>         = mapping agent loopback address
! <MAPPING_AGENT_LOOPBACK>        = mapping agent loopback interface
! <MAPPING_AGENT_SOURCE_INTERFACE> = mapping agent source loopback/interface
! <TRANSIT_INTERFACE>             = routed interface between PIM routers
! <SOURCE_FACING_INTERFACE>       = first-hop router interface toward source
! <RECEIVER_FACING_INTERFACE>     = last-hop router interface toward receiver LAN
! <RECEIVER_INTERFACE>            = test receiver interface
! <AUTO_RP_GROUP_ACL>             = standard ACL matching multicast group range
! <MULTICAST_GROUP_RANGE>         = multicast range advertised by Auto-RP
! <GROUP_WILDCARD>                = wildcard mask for multicast group range
! <MULTICAST_GROUP>               = test multicast group inside advertised range
! <SOURCE_IP>                     = multicast source IP
! <SOURCE_INTERFACE_OR_IP>        = source interface or IP for test traffic
! <ELECTED_RP_ADDRESS>            = RP elected by mapping agent
! <ANNOUNCE_TTL_SCOPE>            = TTL scope for RP-Announce
! <DISCOVERY_TTL_SCOPE>           = TTL scope for RP-Discovery
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! PIM SPARSE-DENSE MODE BASELINE
! Apply on candidate RP loopbacks, mapping-agent loopback,
! transit links, source-facing interfaces, and receiver-facing interfaces.
! =========================================================
conf t
 interface <CANDIDATE_RP_LOOPBACK>
  ip pim sparse-dense-mode
 exit
 interface <MAPPING_AGENT_LOOPBACK>
  ip pim sparse-dense-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-dense-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-dense-mode
 exit
end
! =========================================================
! CANDIDATE RP GROUP RANGE
! Standard ACL matches multicast groups, not receiver hosts.
! =========================================================
conf t
 access-list <AUTO_RP_GROUP_ACL> permit <MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>
end
! =========================================================
! PRIMARY CANDIDATE RP
! Sends RP-Announce to 224.0.1.39.
! Source interface must have PIM enabled.
! =========================================================
conf t
 ip pim send-rp-announce <PRIMARY_RP_SOURCE_INTERFACE> scope <ANNOUNCE_TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! BACKUP CANDIDATE RP
! If both advertise the same group range, the mapping agent
! elects the candidate RP with the highest source IP address.
! =========================================================
conf t
 ip pim send-rp-announce <BACKUP_RP_SOURCE_INTERFACE> scope <ANNOUNCE_TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! MAPPING AGENT
! Receives RP-Announce on 224.0.1.39 and sends RP-Discovery
! to 224.0.1.40.
! =========================================================
conf t
 ip pim send-rp-discovery <MAPPING_AGENT_SOURCE_INTERFACE> scope <DISCOVERY_TTL_SCOPE>
end
! =========================================================
! ALTERNATIVE BOOTSTRAP MODEL
! Use this only when interfaces are sparse-mode instead of
! sparse-dense-mode and Auto-RP discovery still must work.
! =========================================================
conf t
 ip pim autorp listener
end
! =========================================================
! RECEIVER JOIN
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
show ip mroute 224.0.1.39
show ip mroute 224.0.1.40
show ip igmp groups
show ip mroute <MULTICAST_GROUP>
show ip rpf <ELECTED_RP_ADDRESS>
show ip rpf <SOURCE_IP>
debug ip pim auto-rp
undebug all
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | All multicast routers | `show ip pim interface` | Candidate RP loopback, mapping-agent loopback, transit, source-facing, and receiver-facing interfaces appear |
| PIM neighbor state | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear on routed transit links |
| Candidate RP announce config | Candidate RP routers | `show running-config | include send-rp-announce` | `ip pim send-rp-announce` is present with source interface, scope, and group-list |
| Mapping-agent discovery config | Mapping-agent router | `show running-config | include send-rp-discovery` | `ip pim send-rp-discovery` is present with source interface and scope |
| Group-list ACL | Candidate RP routers | `show access-lists <AUTO_RP_GROUP_ACL>` | ACL permits the intended multicast group range |
| RP-Announce group state | Candidate RPs, mapping agent, transit routers | `show ip mroute 224.0.1.39` | Auto-RP announce group state exists |
| RP-Discovery group state | Mapping agent and all multicast routers | `show ip mroute 224.0.1.40` | Auto-RP discovery group state exists |
| Auto-RP learned mapping | All multicast routers | `show ip pim rp mapping` | Group range maps to elected RP and info source is mapping agent |
| Candidate RP identity | Candidate RP router | `show ip pim rp mapping` | Output shows `This system is an RP (Auto-RP)` |
| Mapping-agent identity | Mapping-agent router | `show ip pim rp mapping` | Output shows `This system is an RP-mapping agent` |
| Election result | All multicast routers | `show ip pim rp mapping` | Highest candidate RP source IP wins for overlapping group range |
| Mapping expiration refresh | All multicast routers | `show ip pim rp mapping` | RP mapping expiration timer refreshes as RP-Discovery messages arrive |
| Auto-RP listener config | All multicast routers | `show running-config | include autorp listener` | `ip pim autorp listener` is present only if sparse-mode Auto-RP workaround is used |
| Receiver membership | Last-hop router | `show ip igmp groups` | Receiver group appears on receiver-facing interface |
| Multicast state | RP, transit, and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` state exists for the Auto-RP mapped group |
| RPF toward RP | Last-hop and transit routers | `show ip rpf <ELECTED_RP_ADDRESS>` | RPF path points toward elected RP |
| RPF toward source | RP, transit, and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF path points toward source |
| Packet counters | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Counters increment during source traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies when using `ip igmp join-group` and multicast forwarding is correct |
| Auto-RP debug | Candidate RP and mapping agent | `debug ip pim auto-rp` | RP-Announce and RP-Discovery messages appear during controlled testing |
| Debug cleanup | All routers | `show debugging`<br>`undebug all` | No multicast debug remains active |
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Rollback
! =========================================================
! REMOVE RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE PRIMARY CANDIDATE RP ANNOUNCEMENT
! =========================================================
conf t
 no ip pim send-rp-announce <PRIMARY_RP_SOURCE_INTERFACE> scope <ANNOUNCE_TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! REMOVE BACKUP CANDIDATE RP ANNOUNCEMENT
! =========================================================
conf t
 no ip pim send-rp-announce <BACKUP_RP_SOURCE_INTERFACE> scope <ANNOUNCE_TTL_SCOPE> group-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! REMOVE MAPPING AGENT DISCOVERY
! =========================================================
conf t
 no ip pim send-rp-discovery <MAPPING_AGENT_SOURCE_INTERFACE> scope <DISCOVERY_TTL_SCOPE>
end
! =========================================================
! REMOVE AUTO-RP LISTENER IF USED
! =========================================================
conf t
 no ip pim autorp listener
end
! =========================================================
! REMOVE AUTO-RP GROUP ACL
! =========================================================
conf t
 no access-list <AUTO_RP_GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM CANDIDATE RP LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <CANDIDATE_RP_LOOPBACK>
  no ip pim sparse-dense-mode
 exit
end
! =========================================================
! REMOVE PIM FROM MAPPING-AGENT LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <MAPPING_AGENT_LOOPBACK>
  no ip pim sparse-dense-mode
 exit
end
! =========================================================
! REMOVE PIM FROM TRANSIT, SOURCE, AND RECEIVER INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  no ip pim sparse-dense-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
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
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| `ip pim send-rp-announce` returns interface error | PIM is not enabled on the source interface used for RP-Announce | Candidate RP | `show ip pim interface`<br>`show running-config interface <PRIMARY_RP_SOURCE_INTERFACE>` | Configure `ip pim sparse-dense-mode` or supported PIM mode on the RP source interface |
| No Auto-RP mapping appears | RP-Announce or RP-Discovery messages are not reaching routers | Candidate RP, mapping agent, transit routers | `show ip mroute 224.0.1.39`<br>`show ip mroute 224.0.1.40` | Use `ip pim sparse-dense-mode` or configure `ip pim autorp listener` |
| Candidate RP announces but mapping agent does not learn it | TTL scope too low, PIM path missing, or 224.0.1.39 blocked | Candidate RP and mapping agent | `debug ip pim auto-rp`<br>`show ip mroute 224.0.1.39` | Increase scope, fix PIM path, and remove multicast boundary blocking announce group |
| Mapping agent learns RP but routers do not learn mapping | RP-Discovery messages to 224.0.1.40 are not reaching routers | Mapping agent and downstream routers | `show ip mroute 224.0.1.40` | Fix PIM reachability, scope, or multicast boundary for 224.0.1.40 |
| Wrong RP is elected | Auto-RP mapping agent chooses highest RP source IP for overlapping group ranges | All multicast routers | `show ip pim rp mapping` | Use higher source IP for desired primary, avoid overlap, or use BSR/static RP for priority control |
| Backup RP never becomes active | Primary RP announcements are still being received | Mapping agent and all routers | `show ip pim rp mapping` | Remove primary announcement, break primary reachability intentionally for test, or wait for mapping expiration |
| Group range missing from mapping | Candidate RP group-list ACL does not match the multicast group | Candidate RP | `show access-lists <AUTO_RP_GROUP_ACL>` | Correct ACL to match the multicast group range |
| All groups are advertised unintentionally | Candidate RP missing group-list or ACL is too broad | Candidate RP | `show running-config | include send-rp-announce`<br>`show access-lists` | Add or narrow `group-list` ACL |
| Auto-RP fails on sparse-mode-only interfaces | Classic Auto-RP discovery groups cannot bootstrap through pure sparse mode | All multicast routers | `show ip pim interface`<br>`show ip mroute 224.0.1.39`<br>`show ip mroute 224.0.1.40` | Use `ip pim sparse-dense-mode` or add `ip pim autorp listener` |
| Mapping expiration timer counts down and disappears | Routers are not receiving recurring RP-Discovery messages | Routers missing mapping | `show ip pim rp mapping` | Fix mapping-agent reachability, scope, dense fallback, or multicast boundary |
| PIM neighbors missing | PIM not enabled on one side or interface reachability is broken | Transit routers | `show ip pim neighbor`<br>`show ip interface brief` | Enable PIM on both sides and fix interface addressing/connectivity |
| RP mapping exists but traffic fails | Auto-RP worked, but PIM forwarding, IGMP, RPF, or source traffic is broken | RP, last-hop, and source-side routers | `show ip igmp groups`<br>`show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Fix receiver join, source traffic, RPF, or PIM path |
| Receiver group missing | Receiver did not join or joined on wrong interface | Last-hop router and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <MULTICAST_GROUP>` on correct receiver interface |
| Multicast boundary blocks Auto-RP | Boundary ACL blocks 224.0.1.39 or 224.0.1.40 | Boundary router | `show running-config | include multicast boundary`<br>`show ip mroute 224.0.1.39`<br>`show ip mroute 224.0.1.40` | Permit Auto-RP control groups across the required boundary or place mapping agents appropriately |
| Debug floods the console | Auto-RP debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Mental_Model
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Configuration_Checklist
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Skeleton
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Verification_Commands
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Rollback
# PIM_Auto_RP_Candidate_RP_And_Mapping_Agent_Failure_Checks

