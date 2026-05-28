

Multicast_RPF_And_Mroute_Baseline.md

Multicast_RPF_And_Mroute_Baseline


# Multicast_RPF_And_Mroute_Baseline_Mental_Model
| Concept                    | Operational Meaning                                                                                                                                      |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Multicast forwarding       | Multicast does not flood everywhere by default. Routers build multicast forwarding state based on source, group, receiver interest, and PIM behavior     |
| RPF check                  | Reverse Path Forwarding is the multicast loop-prevention check. A multicast packet must arrive on the interface the router would use to reach the source |
| Unicast route controls RPF | The unicast routing table normally determines the correct RPF interface unless a multicast route or multicast-specific RIB changes the decision          |
| Incoming Interface         | In `show ip mroute`, the Incoming Interface is the interface where multicast traffic is expected to arrive from the source                               |
| RPF neighbor               | The RPF neighbor is the upstream next-hop router toward the multicast source                                                                             |
| Outgoing Interface List    | The OIL lists the downstream interfaces where multicast traffic is forwarded toward receivers                                                            |
| `(*,G)` state              | Shared tree or group-level state. It means "any source for this group"                                                                                   |
| `(S,G)` state              | Source-specific state. It means "this exact source sending to this exact group"                                                                          |
| IGMP role                  | IGMP tells the local multicast router that receivers want traffic for a group                                                                            |
| PIM role                   | PIM builds router-to-router multicast forwarding state                                                                                                   |
| Mroute table               | `show ip mroute` is the multicast forwarding truth table. If the IIF, RPF neighbor, or OIL is wrong, forwarding will fail                                |
| Baseline lab goal          | Prove that unicast reachability, PIM interfaces, IGMP receiver state, RPF, and mroute forwarding state all line up                                       |
# Multicast_RPF_And_Mroute_Baseline_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the source, transit, and receiver interfaces are up | All multicast routers | `show ip interface brief` | Required routed interfaces are up/up with correct IP addresses |
| 2 | Confirm unicast reachability toward the multicast source | All multicast routers | `show ip route <SOURCE_IP>` | Each router has a route toward the multicast source |
| 3 | Confirm the expected RPF interface before enabling multicast | Transit and receiver-side routers | `show ip route <SOURCE_IP>` | The outgoing interface toward `<SOURCE_IP>` is the interface where multicast traffic should arrive from upstream |
| 4 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router is allowed to build multicast forwarding state |
| 5 | Enable PIM on the source-facing interface | Source-side router | `interface <SOURCE_FACING_INTERFACE>`<br>`ip pim dense-mode` | Interface participates in PIM and can forward multicast traffic |
| 6 | Enable PIM on transit interfaces | Transit routers | `interface <TRANSIT_INTERFACE>`<br>`ip pim dense-mode` | PIM is active across routed multicast transit links |
| 7 | Enable PIM on the receiver-facing routed interface | Last-hop router | `interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim dense-mode` | IGMP is enabled with PIM and the router can learn receiver interest |
| 8 | Verify PIM is enabled on required interfaces | All multicast routers | `show ip pim interface` | Required interfaces appear in PIM interface output |
| 9 | Verify PIM neighbor formation on routed transit links | Transit routers | `show ip pim neighbor` | PIM neighbors appear on expected transit interfaces |
| 10 | Join a multicast group from a test receiver router or host-facing test interface | Receiver/test router | `interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Router emulates a receiver for the multicast group |
| 11 | Verify IGMP group membership on the last-hop router | Last-hop router | `show ip igmp groups` | Joined multicast group appears on the receiver-facing interface |
| 12 | Generate multicast traffic from the source side | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast traffic is generated from the expected source |
| 13 | Verify multicast route state was created | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` and/or `(<SOURCE>,<GROUP>)` entries appear |
| 14 | Validate the incoming interface and RPF neighbor | Transit and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | Incoming Interface matches the RPF path back to the source |
| 15 | Validate the outgoing interface list | Source-side and transit routers | `show ip mroute <MULTICAST_GROUP>` | OIL includes downstream interfaces toward interested receivers |
| 16 | Verify the RPF decision directly | Transit and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF interface and RPF neighbor match the expected upstream path |
| 17 | Confirm receiver response if using `join-group` on a router | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver/test router responds if configured with `ip igmp join-group` |
| 18 | Save the working baseline | All multicast routers | `copy running-config startup-config` | Baseline multicast configuration is preserved |
# Multicast_RPF_And_Mroute_Baseline_Skeleton
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
 interface <TRANSIT_INTERFACE_TO_DOWNSTREAM_ROUTER>
  ip pim dense-mode
 exit
end
! =========================================================
! TRANSIT ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE_TO_SOURCE_SIDE>
  ip pim dense-mode
 exit
 interface <TRANSIT_INTERFACE_TO_RECEIVER_SIDE>
  ip pim dense-mode
 exit
end
! =========================================================
! LAST-HOP ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE_TO_UPSTREAM_ROUTER>
  ip pim dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! TEST RECEIVER
! Use this when the receiver is a router emulating a host
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! OPTIONAL STATIC MROUTE FOR RPF CORRECTION
! Do not use this in a clean baseline unless the lab requires
! overriding the normal unicast RPF path
! =========================================================
conf t
 ip mroute <SOURCE_IP> <SOURCE_MASK> <RPF_NEXT_HOP_OR_INTERFACE>
end
! =========================================================
! TEST TRAFFIC
! =========================================================
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
# Multicast_RPF_And_Mroute_Baseline_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Interface state | All routers | `show ip interface brief` | Required routed interfaces are up/up |
| Unicast route to source | Transit and last-hop routers | `show ip route <SOURCE_IP>` | Route points toward expected upstream source path |
| PIM-enabled interfaces | All multicast routers | `show ip pim interface` | Required interfaces are listed |
| PIM neighbors | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear on transit links |
| IGMP receiver state | Last-hop router | `show ip igmp groups` | Receiver group appears on receiver-facing interface |
| IGMP interface status | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | IGMP and multicast routing are enabled |
| Multicast forwarding table | All multicast routers | `show ip mroute` | Multicast groups and source/group entries are present |
| Specific group state | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` entries exist |
| RPF decision | Transit and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF interface matches incoming multicast interface |
| OIL validation | Source-side and transit routers | `show ip mroute <MULTICAST_GROUP>` | Outgoing Interface List includes downstream receiver path |
| Multicast packet counters | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during multicast test traffic |
| Active multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Replies occur when receiver is configured with `ip igmp join-group` |
# Multicast_RPF_And_Mroute_Baseline_Rollback
! =========================================================
! REMOVE TEST RECEIVER GROUP JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE PIM FROM INTERFACES
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip pim dense-mode
 exit
 interface <TRANSIT_INTERFACE>
  no ip pim dense-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE OPTIONAL STATIC MROUTE
! =========================================================
conf t
 no ip mroute <SOURCE_IP> <SOURCE_MASK> <RPF_NEXT_HOP_OR_INTERFACE>
end
! =========================================================
! DISABLE MULTICAST ROUTING GLOBALLY
! Only do this if no other multicast labs or services depend on it
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR STATE AFTER ROLLBACK
! =========================================================
clear ip mroute *
clear ip igmp group *
# Multicast_RPF_And_Mroute_Baseline_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No multicast route entries appear | Multicast routing not enabled globally | All multicast routers | `show running-config | include ip multicast-routing` | Configure `ip multicast-routing` |
| PIM interface missing | PIM not enabled on required interface | Affected router | `show ip pim interface` | Configure `ip pim dense-mode` under the interface |
| No PIM neighbor | PIM missing on one side, interface down, IP mismatch, or blocked control traffic | Transit routers | `show ip pim neighbor` | Fix interface/IP adjacency and enable PIM on both sides |
| Receiver group missing | Receiver did not send IGMP join or test router lacks `join-group` | Last-hop router | `show ip igmp groups` | Configure receiver or `ip igmp join-group <GROUP>` |
| Mroute exists but traffic does not forward | RPF failure | Transit or last-hop router | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <GROUP>` | Fix unicast route toward source or use a deliberate `ip mroute` override |
| Incoming interface is wrong | Unicast route to source points to wrong path | Transit or last-hop router | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>` | Correct IGP/static routing so RPF points upstream |
| OIL is empty | No downstream receiver interest or no PIM downstream path | Source-side or transit router | `show ip mroute <GROUP>` | Verify IGMP group membership and downstream PIM |
| Ping to multicast group gets no replies | Receiver used `static-group` instead of `join-group`, or no receiver is actually processing packets | Receiver/test router | `show running-config interface <RECEIVER_INTERFACE>` | Use `ip igmp join-group <GROUP>` for router-based ping testing |
| Counters do not increment | Source is not sending, wrong source address used, or RPF drop occurs | Source and transit routers | `show ip mroute <GROUP>` | Generate traffic with correct source and verify RPF |
| Multicast works one direction only | Multicast tree/RPF path is source-dependent | Transit routers | `show ip rpf <SOURCE_IP>` | Validate RPF from every router back to each source |
| Auto-RP group appears unexpectedly | PIM created default multicast control-plane state | Any PIM router | `show ip mroute 224.0.1.40` | Normal when PIM is enabled. Do not treat as data-plane failure |
##### Source_Basis
# Multicast_RPF_And_Mroute_Baseline_Mental_Model
# Multicast_RPF_And_Mroute_Baseline_Configuration_Checklist
# Multicast_RPF_And_Mroute_Baseline_Skeleton
# Multicast_RPF_And_Mroute_Baseline_Verification_Commands
# Multicast_RPF_And_Mroute_Baseline_Rollback
# Multicast_RPF_And_Mroute_Baseline_Failure_Checks

