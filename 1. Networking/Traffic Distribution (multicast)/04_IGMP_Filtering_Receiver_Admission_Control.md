IGMP_Filtering_Receiver_Admission_Control.md

IGMP_Filtering_Receiver_Admission_Control

# IGMP_Filtering_Receiver_Admission_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IGMP filtering | IGMP filtering controls which multicast groups receivers are allowed to join on a local interface |
| Receiver admission control | The router accepts or denies IGMP membership reports before adding the receiver-facing interface to multicast group state |
| Not a data-plane ACL | `ip igmp access-group` filters IGMP group membership, not normal routed traffic |
| Not a PIM filter | This does not filter PIM neighbors, RP mappings, register messages, or multicast traffic already admitted upstream |
| Standard ACL role | The standard ACL matches multicast group addresses, not receiver host addresses |
| Permit logic | Permitted multicast group addresses are allowed to be joined |
| Deny logic | Denied multicast group addresses are rejected when receivers try to join |
| Implicit deny | If the ACL does not end with `permit any`, all other group joins are denied |
| Interface scope | The filter applies only to the interface where `ip igmp access-group` is configured |
| Last-hop control point | IGMP filtering belongs on the last-hop router interface facing receivers |
| Verification truth | `show ip igmp interface` proves the access group is attached. `show ip igmp groups` proves whether the group was admitted |
| Debug proof | `debug ip igmp` shows IGMP reports and access-denied messages during testing |
# IGMP_Filtering_Receiver_Admission_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the receiver-facing interface where joins must be controlled | Last-hop router | `show ip interface brief` | Receiver-facing interface and subnet are known |
| 2 | Identify the multicast groups to deny | Last-hop router | `show ip igmp groups` | Existing multicast memberships are visible |
| 3 | Identify the multicast groups to allow | Last-hop router | `show ip igmp groups` | Allowed groups are known before applying the filter |
| 4 | Confirm multicast routing is enabled | Last-hop router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| 5 | Enable multicast routing globally if missing | Last-hop router | `conf t`<br>`ip multicast-routing` | Router can process multicast control-plane and forwarding state |
| 6 | Confirm PIM and IGMP are enabled on the receiver-facing interface | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | IGMP is enabled on the receiver-facing interface |
| 7 | Enable PIM on the receiver-facing interface if IGMP is not active | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim dense-mode` | PIM enables IGMP on the interface |
| 8 | Create a named standard ACL for IGMP group admission control | Last-hop router | `conf t`<br>`ip access-list standard <IGMP_FILTER_ACL>` | Standard ACL configuration mode opens |
| 9 | Deny the first blocked multicast group | Last-hop router | `deny <DENIED_MULTICAST_GROUP_1>` | Receivers cannot join this group through the filtered interface |
| 10 | Deny the second blocked multicast group if needed | Last-hop router | `deny <DENIED_MULTICAST_GROUP_2>` | Receivers cannot join this additional group through the filtered interface |
| 11 | Permit all other multicast groups if the policy is deny-specific | Last-hop router | `permit any` | All other group joins remain allowed |
| 12 | Exit ACL configuration mode | Last-hop router | `exit` | Router returns to global configuration mode |
| 13 | Apply the IGMP access group to the receiver-facing interface | Last-hop router | `interface <RECEIVER_FACING_INTERFACE>`<br>`ip igmp access-group <IGMP_FILTER_ACL>` | IGMP join filtering is active on the receiver-facing interface |
| 14 | Verify the IGMP access group is attached | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | Output shows `Inbound IGMP access group is <IGMP_FILTER_ACL>` |
| 15 | Enable IGMP debugging for a controlled test | Last-hop router | `debug ip igmp` | IGMP report, leave, and deny messages are displayed |
| 16 | Attempt to join a denied group from a test receiver | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <DENIED_MULTICAST_GROUP_1>` | Receiver sends an IGMP report for the denied group |
| 17 | Verify the denied group is rejected | Last-hop router | `show ip igmp groups | include <DENIED_MULTICAST_GROUP_1>` | Denied group does not appear as an admitted receiver membership |
| 18 | Confirm the denial in debug output | Last-hop router | `debug ip igmp` | Output shows the group report and access denied message |
| 19 | Attempt to join an allowed group from a test receiver | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <ALLOWED_MULTICAST_GROUP>` | Receiver sends an IGMP report for the allowed group |
| 20 | Verify the allowed group is admitted | Last-hop router | `show ip igmp groups | include <ALLOWED_MULTICAST_GROUP>` | Allowed group appears on the receiver-facing interface |
| 21 | Verify multicast route state for the allowed group | Last-hop router | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Allowed group creates multicast forwarding state |
| 22 | Verify denied group does not create receiver-driven mroute state | Last-hop router | `show ip mroute <DENIED_MULTICAST_GROUP_1>` | Denied group is absent or has no receiver-facing OIL entry |
| 23 | Disable debugging after testing | Last-hop router | `undebug all` | Debugging is disabled |
| 24 | Save the working IGMP filtering configuration | Last-hop router | `copy running-config startup-config` | IGMP receiver admission policy is preserved |
# IGMP_Filtering_Receiver_Admission_Control_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RECEIVER_FACING_INTERFACE> = last-hop router interface toward receivers
! <RECEIVER_INTERFACE>        = test receiver interface
! <IGMP_FILTER_ACL>           = standard ACL name used for IGMP group admission
! <DENIED_MULTICAST_GROUP_1>  = blocked multicast group
! <DENIED_MULTICAST_GROUP_2>  = optional second blocked multicast group
! <ALLOWED_MULTICAST_GROUP>   = permitted multicast group used for positive test
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
! NAMED STANDARD ACL
! Deny specific multicast groups, then permit everything else.
! Standard ACL entries match multicast group addresses here.
! =========================================================
conf t
 ip access-list standard <IGMP_FILTER_ACL>
  deny <DENIED_MULTICAST_GROUP_1>
  deny <DENIED_MULTICAST_GROUP_2>
  permit any
 exit
end
! =========================================================
! APPLY IGMP RECEIVER ADMISSION FILTER
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  ip igmp access-group <IGMP_FILTER_ACL>
 exit
end
! =========================================================
! DENIED GROUP TEST
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <DENIED_MULTICAST_GROUP_1>
 exit
end
! =========================================================
! ALLOWED GROUP TEST
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <ALLOWED_MULTICAST_GROUP>
 exit
end
! =========================================================
! VERIFICATION
! =========================================================
show access-lists <IGMP_FILTER_ACL>
show ip igmp interface <RECEIVER_FACING_INTERFACE>
show ip igmp groups
show ip mroute <ALLOWED_MULTICAST_GROUP>
show ip mroute <DENIED_MULTICAST_GROUP_1>
! =========================================================
! OPTIONAL DEBUG
! Use only during a controlled test.
! =========================================================
debug ip igmp
undebug all
# IGMP_Filtering_Receiver_Admission_Control_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | Last-hop router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| IGMP interface state | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | IGMP is enabled on the receiver-facing interface |
| PIM interface state | Last-hop router | `show ip pim interface` | Receiver-facing interface appears in PIM interface output |
| ACL contents | Last-hop router | `show access-lists <IGMP_FILTER_ACL>` | Denied groups and final permit statement are present |
| IGMP access group attachment | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | Output shows `Inbound IGMP access group is <IGMP_FILTER_ACL>` |
| Denied group membership | Last-hop router | `show ip igmp groups | include <DENIED_MULTICAST_GROUP_1>` | Denied group is not listed |
| Allowed group membership | Last-hop router | `show ip igmp groups | include <ALLOWED_MULTICAST_GROUP>` | Allowed group is listed on the receiver-facing interface |
| Allowed mroute state | Last-hop router | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Receiver-facing interface appears in the OIL when traffic/state exists |
| Denied mroute state | Last-hop router | `show ip mroute <DENIED_MULTICAST_GROUP_1>` | Denied group is absent or has no receiver-facing OIL entry |
| Receiver-side join config | Receiver/test router | `show running-config interface <RECEIVER_INTERFACE>` | Intended `ip igmp join-group` commands are present |
| Receiver local group state | Receiver/test router | `show ip igmp interface <RECEIVER_INTERFACE> | begin Multicast groups` | Local joined groups are shown |
| IGMP debug denial | Last-hop router | `debug ip igmp` | Denied group report produces an access denied message |
| Debug disabled | Last-hop router | `undebug all` | All debugging is disabled |
# IGMP_Filtering_Receiver_Admission_Control_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOINS
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <DENIED_MULTICAST_GROUP_1>
  no ip igmp join-group <DENIED_MULTICAST_GROUP_2>
  no ip igmp join-group <ALLOWED_MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE IGMP ACCESS GROUP FROM RECEIVER-FACING INTERFACE
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip igmp access-group <IGMP_FILTER_ACL>
 exit
end
! =========================================================
! REMOVE THE IGMP FILTER ACL
! =========================================================
conf t
 no ip access-list standard <IGMP_FILTER_ACL>
end
! =========================================================
! REMOVE PIM FROM RECEIVER-FACING INTERFACE
! Only do this if no other multicast labs depend on the interface.
! =========================================================
conf t
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim dense-mode
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
! CLEAR DYNAMIC STATE AND DEBUGGING
! =========================================================
clear ip igmp group *
clear ip mroute *
undebug all
# IGMP_Filtering_Receiver_Admission_Control_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Denied group still appears in IGMP groups | ACL not applied to the correct receiver-facing interface | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>` | Apply `ip igmp access-group <IGMP_FILTER_ACL>` to the correct interface |
| Denied group still appears after policy change | Old IGMP state has not aged out yet | Last-hop router | `show ip igmp groups` | Clear state with `clear ip igmp group *` or wait for expiration |
| All group joins are denied | ACL is missing `permit any` after specific deny entries | Last-hop router | `show access-lists <IGMP_FILTER_ACL>` | Add `permit any` at the end of the ACL |
| Allowed group is denied | ACL entry order is wrong | Last-hop router | `show access-lists <IGMP_FILTER_ACL>` | Put specific permits before broader denies, or use deny-specific plus `permit any` |
| Filter appears attached but no joins work | IGMP is not enabled because PIM is missing | Last-hop router | `show ip igmp interface <RECEIVER_FACING_INTERFACE>`<br>`show ip pim interface` | Enable PIM on the receiver-facing interface |
| `show ip igmp interface` says inbound access group is not set | `ip igmp access-group` was not applied under the interface | Last-hop router | `show running-config interface <RECEIVER_FACING_INTERFACE>` | Apply `ip igmp access-group <IGMP_FILTER_ACL>` |
| ACL matches receiver IPs instead of groups | Standard ACL was built with host addresses instead of multicast group addresses | Last-hop router | `show access-lists <IGMP_FILTER_ACL>` | Rewrite ACL to match multicast group addresses |
| Debug shows reports but no access denied message | Group is not denied by the ACL | Last-hop router | `debug ip igmp`<br>`show access-lists <IGMP_FILTER_ACL>` | Add a deny entry for the multicast group |
| Receiver says it joined but last-hop router does not show group | Join was configured on the wrong receiver interface or wrong VLAN/subnet | Receiver/test router and last-hop router | `show running-config interface <RECEIVER_INTERFACE>`<br>`show ip igmp groups` | Configure the join on the interface connected to the last-hop router |
| Denied group has mroute state upstream | State may be from another receiver, another interface, or stale multicast state | Last-hop and upstream routers | `show ip mroute <DENIED_MULTICAST_GROUP_1>`<br>`show ip igmp groups` | Check all receiver-facing interfaces and clear stale state |
| Multicast traffic still reaches receiver | Traffic was already forwarded by another path or the filter is on the wrong router/interface | Receiver path routers | `show ip mroute <DENIED_MULTICAST_GROUP_1>` | Apply IGMP filtering at the actual last-hop receiver interface |
| Console becomes noisy | `debug ip igmp` left enabled | Last-hop router | `show debugging` | Run `undebug all` |
##### Source_Basis
# IGMP_Filtering_Receiver_Admission_Control_Mental_Model
# IGMP_Filtering_Receiver_Admission_Control_Configuration_Checklist
# IGMP_Filtering_Receiver_Admission_Control_Skeleton
# IGMP_Filtering_Receiver_Admission_Control_Verification_Commands
# IGMP_Filtering_Receiver_Admission_Control_Rollback
# IGMP_Filtering_Receiver_Admission_Control_Failure_Checks
