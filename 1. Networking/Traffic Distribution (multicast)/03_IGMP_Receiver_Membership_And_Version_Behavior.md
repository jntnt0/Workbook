
IGMP_Receiver_Membership_And_Version_Behavior.md

IGMP_Receiver_Membership_And_Version_Behavior

# IGMP_Receiver_Membership_And_Version_Behavior_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IGMP role | IGMP is receiver-to-router signaling. It tells the local multicast router which groups have interested receivers on a directly connected segment |
| IGMP is not routing | IGMP does not build router-to-router multicast paths. PIM handles multicast routing between routers |
| PIM enables IGMP | On IOS, enabling PIM on an interface also enables IGMP on that interface |
| Last-hop router | The router connected to the receiver LAN learns IGMP memberships and adds the receiver-facing interface to the multicast outgoing interface list |
| IGMPv1 behavior | IGMPv1 supports receiver membership reports but does not have formal IGMP querier election or explicit leave behavior |
| IGMPv2 behavior | IGMPv2 adds formal querier election, group-specific leave behavior, and advertised max query response time |
| Querier election | In IGMPv2, the router with the lowest source IP address on the LAN becomes the IGMP querier |
| Query interval | The query interval controls how often the querier asks whether receivers still exist for multicast groups |
| Querier timeout | The querier timeout controls how long other routers wait before electing a new querier after queries stop |
| Max response time | The max query response time tells hosts how long they may wait before responding to a general query |
| `join-group` | `ip igmp join-group` makes the router emulate a receiver and process/respond to multicast pings |
| `static-group` | `ip igmp static-group` creates multicast group state but does not make the router process multicast ping replies |
| Group expiration | Dynamic IGMP memberships age out if no receiver reports are heard. Locally configured `join-group` entries show `never` expiration |
| Version mismatch | A segment can fail or behave strangely when the router and receiver expectations do not match the intended IGMP version |
# IGMP_Receiver_Membership_And_Version_Behavior_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the receiver-facing LAN and multicast test group | Last-hop router | `show ip interface brief` | Receiver-facing interface and IP subnet are known |
| 2 | Confirm multicast routing is enabled globally | Last-hop router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| 3 | Enable multicast routing globally if missing | Last-hop router | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 4 | Enable PIM on the receiver-facing interface | Last-hop router | `interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim dense-mode` | PIM and IGMP are enabled on the receiver-facing interface |
| 5 | Verify IGMP is active on the interface | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | Output shows `IGMP is enabled on interface` |
| 6 | Verify the current IGMP host and router version | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include Current IGMP` | Current IGMP host and router versions are displayed |
| 7 | Configure IGMPv1 for a version 1 receiver lab | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip igmp version 1` | Interface uses IGMP version 1 behavior |
| 8 | Verify IGMPv1 interface behavior | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | Current IGMP router version shows 1 |
| 9 | Configure IGMPv2 for a version 2 receiver lab | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip igmp version 2` | Interface uses IGMP version 2 behavior |
| 10 | Verify IGMPv2 interface behavior | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | Current IGMP router version shows 2 |
| 11 | Confirm IGMP query interval | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include query interval` | Default or configured query interval is visible |
| 12 | Configure IGMP query interval if the lab requires a nondefault value | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip igmp query-interval <SECONDS>` | Querier sends general queries at the configured interval |
| 13 | Confirm IGMP querier timeout | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include querier timeout` | Default or configured querier timeout is visible |
| 14 | Configure IGMP querier timeout if the lab requires faster querier failover | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip igmp querier-timeout <SECONDS>` | Other routers wait the configured time before querier reelection |
| 15 | Confirm IGMP max query response time | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include max query` | Default or configured max response time is visible |
| 16 | Configure IGMP max query response time if required | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip igmp query-max-response-time <SECONDS>` | Queries advertise the configured max response time |
| 17 | Configure a router to emulate a multicast receiver | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Test router joins the multicast group and can respond to multicast pings |
| 18 | Verify local receiver join on the test router | Receiver/test router | `show ip igmp interface <RECEIVER_INTERFACE> | begin Multicast groups` | Joined group appears under multicast groups joined by this system |
| 19 | Verify receiver membership on the last-hop router | Last-hop router | `show ip igmp groups` | Multicast group appears on the receiver-facing interface |
| 20 | Verify group expiration behavior | Last-hop router | `show ip igmp groups <RECEIVER_FACING_INTERFACE>` | Dynamic receiver entries show an expiration timer |
| 21 | Verify locally configured join expiration behavior | Receiver/test router | `show ip igmp groups` | Locally configured `join-group` shows `never` in the Expires column |
| 22 | Generate multicast test traffic | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver/test router responds when `ip igmp join-group` is used |
| 23 | Verify multicast route state from the receiver join | Last-hop and transit routers | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` and/or `(<SOURCE>,<GROUP>)` entries exist |
| 24 | Test leave behavior for IGMPv2 | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`no ip igmp join-group <MULTICAST_GROUP>` | Receiver sends leave behavior and the last-hop router eventually removes the group |
| 25 | Verify the group is removed or aging out | Last-hop router | `show ip igmp groups <RECEIVER_FACING_INTERFACE>` | Group disappears or expiration timer decreases after receiver leaves |
| 26 | Save the working IGMP baseline | All changed routers | `copy running-config startup-config` | IGMP version and receiver membership lab baseline is preserved |
# IGMP_Receiver_Membership_And_Version_Behavior_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RECEIVER_FACING_INTERFACE> = last-hop router interface toward receiver LAN
! <RECEIVER_INTERFACE>        = test receiver interface
! <MULTICAST_GROUP>           = multicast group address
! <SOURCE_INTERFACE_OR_IP>    = source interface or source IP for multicast test
! <SECONDS>                   = timer value required by lab
! =========================================================
! LAST-HOP ROUTER BASELINE
! =========================================================
conf t
 ip multicast-routing
 interface <RECEIVER_FACING_INTERFACE>
  ip pim dense-mode
 exit
end
! =========================================================
! FORCE IGMP VERSION 1
! Use for multicast-igmp-v1-final style testing
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  ip igmp version 1
 exit
end
! =========================================================
! FORCE IGMP VERSION 2
! Use for igmp-v2-final and igmp-version-2-final style testing
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  ip igmp version 2
 exit
end
! =========================================================
! OPTIONAL IGMP TIMER TUNING
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  ip igmp query-interval <SECONDS>
  ip igmp querier-timeout <SECONDS>
  ip igmp query-max-response-time <SECONDS>
 exit
end
! =========================================================
! TEST RECEIVER JOIN
! Use join-group when you want the router to process and respond
! to multicast ping tests.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! OPTIONAL STATIC MEMBERSHIP
! Use static-group only when you need group state maintained
! without requiring the router to process multicast ping replies.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp static-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! BASIC TEST
! =========================================================
show ip igmp interface <RECEIVER_FACING_INTERFACE>
show ip igmp groups
show ip mroute <MULTICAST_GROUP>
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
# IGMP_Receiver_Membership_And_Version_Behavior_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | Last-hop router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | Last-hop router | `show ip pim interface` | Receiver-facing interface appears |
| IGMP interface state | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | `IGMP is enabled on interface` appears |
| IGMP version | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include Current IGMP` | Host and router IGMP version match the intended lab |
| IGMP querier | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include querying router` | Expected router is the IGMP querier |
| Query interval | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include query interval` | Configured or default query interval appears |
| Querier timeout | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include querier timeout` | Configured or default querier timeout appears |
| Max response time | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE> | include max query` | Configured or default max query response time appears |
| Receiver membership | Last-hop router | `show ip igmp groups` | Joined multicast group appears on receiver-facing interface |
| Interface-specific group state | Last-hop router | `show ip igmp groups <RECEIVER_FACING_INTERFACE>` | Group, uptime, expires, and last reporter are visible |
| Local receiver join | Receiver/test router | `show ip igmp interface <RECEIVER_INTERFACE> | begin Multicast groups` | Joined group appears under local interface membership |
| Mroute state | Last-hop router | `show ip mroute <MULTICAST_GROUP>` | Multicast route state exists for the joined group |
| Dynamic expiration | Last-hop router | `show ip igmp groups` | Dynamic memberships show an expiration timer |
| Local join expiration | Receiver/test router | `show ip igmp groups` | Local `join-group` membership shows `never` expiration |
| Multicast ping response | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver responds when configured with `ip igmp join-group` |
| IGMP debug | Last-hop router | `debug ip igmp` | Reports, queries, and leaves are visible during testing |
| Disable debug | Last-hop router | `undebug all` | IGMP debug is disabled |
# IGMP_Receiver_Membership_And_Version_Behavior_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE OPTIONAL STATIC GROUP
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp static-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! RESTORE DEFAULT IGMP VERSION BEHAVIOR
! IOS defaults to IGMPv2 when PIM enables IGMP on the interface.
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip igmp version
 exit
end
! =========================================================
! REMOVE IGMP TIMER TUNING
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip igmp query-interval
  no ip igmp querier-timeout
  no ip igmp query-max-response-time
 exit
end
! =========================================================
! REMOVE PIM FROM RECEIVER-FACING INTERFACE
! Only do this if the interface is not needed for other multicast labs.
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim dense-mode
 exit
end
! =========================================================
! REMOVE GLOBAL MULTICAST ROUTING
! Only do this if no other multicast feature depends on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR DYNAMIC STATE
! =========================================================
clear ip igmp group *
clear ip mroute *
undebug all
# IGMP_Receiver_Membership_And_Version_Behavior_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| `show ip igmp interface` does not show IGMP enabled | PIM is not enabled on the interface or multicast routing is missing | Last-hop router | `show running-config | include ip multicast-routing`<br>`show ip pim interface` | Configure `ip multicast-routing` and `ip pim dense-mode` or `ip pim sparse-mode` on the interface |
| Receiver group does not appear | Receiver did not send IGMP report or wrong interface was used | Last-hop router | `show ip igmp groups` | Configure `ip igmp join-group <GROUP>` on the correct receiver interface |
| Multicast ping gets no reply | Receiver used `ip igmp static-group` instead of `ip igmp join-group` | Receiver/test router | `show running-config interface <RECEIVER_INTERFACE>` | Use `ip igmp join-group <GROUP>` for router-based multicast ping testing |
| Group appears but expires | Receiver is no longer reporting membership | Last-hop router | `show ip igmp groups` | Restore receiver join or check IGMP query/report behavior |
| Local test receiver shows `never` expiration | Expected behavior with locally configured `join-group` | Receiver/test router | `show ip igmp groups` | No fix needed unless the join should be removed |
| IGMP version is wrong | Interface is using default IGMPv2 or was forced to another version | Last-hop router | `show ip igmp interface <INTERFACE> | include Current IGMP` | Configure `ip igmp version 1` or `ip igmp version 2` as required |
| IGMPv1 lab does not show IGMPv2 leave behavior | IGMPv1 does not use IGMPv2 leave behavior | Last-hop router | `show ip igmp interface <INTERFACE>` | Use IGMPv2 if explicit leave behavior is required |
| Querier is not the expected router | IGMPv2 querier election selected the lowest source IP address | Routers on receiver LAN | `show ip igmp interface <INTERFACE> | include querying router` | Adjust interface IPs only if the lab requires a specific querier |
| Querier failover is too slow | Querier timeout is too high | Routers on receiver LAN | `show ip igmp interface <INTERFACE> | include querier timeout` | Configure `ip igmp querier-timeout <SECONDS>` |
| Host report timing looks too bursty | Query max response time is too low | Last-hop router | `show ip igmp interface <INTERFACE> | include max query` | Configure `ip igmp query-max-response-time <SECONDS>` |
| No multicast forwarding after IGMP join | IGMP membership exists but multicast route or PIM path is missing | Last-hop and upstream routers | `show ip igmp groups`<br>`show ip mroute <GROUP>`<br>`show ip pim neighbor` | Fix PIM, RPF, source traffic, or upstream multicast routing |
| Debug output floods the console | `debug ip igmp` left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# IGMP_Receiver_Membership_And_Version_Behavior_Mental_Model
# IGMP_Receiver_Membership_And_Version_Behavior_Configuration_Checklist
# IGMP_Receiver_Membership_And_Version_Behavior_Skeleton
# IGMP_Receiver_Membership_And_Version_Behavior_Verification_Commands
# IGMP_Receiver_Membership_And_Version_Behavior_Rollback
# IGMP_Receiver_Membership_And_Version_Behavior_Failure_Checks
