

Multicast_Admin_Boundary_And_Scope_Control.md

Multicast_Admin_Boundary_And_Scope_Control

# Multicast_Admin_Boundary_And_Scope_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Multicast boundary | An interface-level control that prevents selected multicast groups from crossing that interface |
| Scope control | Defines where multicast groups are allowed to spread |
| Administrative scoping | Keeps selected multicast groups inside a region, domain, site, or lab boundary |
| Boundary ACL | Standard ACL matches multicast group addresses, not receiver host addresses |
| Deny means block | In the boundary ACL, denied multicast groups are blocked at the boundary |
| Permit means allow | In the boundary ACL, permitted multicast groups are allowed to cross the boundary |
| Implicit deny risk | If the ACL does not end with `permit any`, other multicast groups may also be blocked unintentionally |
| Interface placement | The boundary must be applied on the interface where multicast should stop crossing |
| Auto-RP impact | Blocking 224.0.1.40 stops RP-Discovery messages from crossing the boundary |
| RP-Announce impact | Blocking 224.0.1.39 stops Candidate RP announcements from reaching mapping agents |
| RP mapping symptom | If Auto-RP discovery is blocked, downstream routers may lose RP mapping after the mapping expires |
| Data-plane symptom | If a data group is blocked, `show ip mroute` may show missing OIL entries beyond the boundary |
| Not an RP filter only | `ip multicast boundary` can block control groups and data groups depending on the ACL |
| Best use | Use boundaries deliberately for site/domain scoping, Auto-RP containment, and preventing multicast leakage |
# Multicast_Admin_Boundary_And_Scope_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the multicast boundary router and boundary-facing interface | Boundary router | `show ip interface brief` | Interface where multicast should stop is known |
| 2 | Identify multicast groups that must be blocked | Boundary router | `show ip mroute` | Groups that should not cross the boundary are known |
| 3 | Identify groups that must remain allowed | Boundary router | `show ip pim rp mapping`<br>`show ip mroute` | Required Auto-RP, RP, and data groups are known before filtering |
| 4 | Confirm multicast routing is enabled globally | Boundary router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| 5 | Enable multicast routing if missing | Boundary router | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 6 | Confirm PIM is enabled on the boundary-facing interface | Boundary router | `show ip pim interface <BOUNDARY_INTERFACE>` | Boundary-facing interface appears as a PIM interface |
| 7 | Enable PIM on the boundary-facing interface if missing | Boundary router | `conf t`<br>`interface <BOUNDARY_INTERFACE>`<br>`ip pim sparse-dense-mode` | Interface participates in multicast control-plane forwarding |
| 8 | Create a standard ACL for the multicast boundary | Boundary router | `conf t`<br>`ip access-list standard <BOUNDARY_ACL>` | Standard ACL configuration mode opens |
| 9 | Block an administratively scoped data group | Boundary router | `deny <BLOCKED_MULTICAST_GROUP>` | Selected multicast data group is blocked from crossing boundary |
| 10 | Block an Auto-RP discovery group if the lab intentionally isolates RP mapping | Boundary router | `deny host 224.0.1.40` | RP-Discovery messages are blocked from crossing boundary |
| 11 | Block Auto-RP announce group if the lab intentionally isolates candidate RP advertisements | Boundary router | `deny host 224.0.1.39` | RP-Announce messages are blocked from crossing boundary |
| 12 | Permit all other multicast groups if this is a deny-specific boundary | Boundary router | `permit any` | Other multicast groups are not accidentally blocked |
| 13 | Apply the multicast boundary to the boundary-facing interface | Boundary router | `conf t`<br>`interface <BOUNDARY_INTERFACE>`<br>`ip multicast boundary <BOUNDARY_ACL>` | Boundary policy is active on the selected interface |
| 14 | Verify the boundary ACL exists | Boundary router | `show access-lists <BOUNDARY_ACL>` | ACL contains blocked groups and final permit statement |
| 15 | Verify boundary is applied to the correct interface | Boundary router | `show running-config interface <BOUNDARY_INTERFACE>` | Interface contains `ip multicast boundary <BOUNDARY_ACL>` |
| 16 | Clear old multicast state for clean boundary testing | Boundary and downstream routers | `clear ip mroute *` | Old mroute state is removed |
| 17 | Test blocked data group from the source side | Source/test router | `ping <BLOCKED_MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Traffic is generated toward the blocked group |
| 18 | Verify blocked group does not cross the boundary | Downstream router | `show ip mroute <BLOCKED_MULTICAST_GROUP>` | Blocked group is absent or has no forwarding state beyond the boundary |
| 19 | Verify allowed multicast group still crosses the boundary | Source/test router | `ping <ALLOWED_MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Allowed group traffic still forwards normally |
| 20 | Verify allowed group state downstream | Downstream router | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Allowed group appears with expected upstream and OIL state |
| 21 | If blocking Auto-RP discovery, verify RP mapping expiration downstream | Downstream router | `show ip pim rp mapping` | RP mapping sourced from the blocked mapping agent expires or disappears |
| 22 | Verify Auto-RP discovery group is blocked where intended | Boundary and downstream routers | `show ip mroute 224.0.1.40` | 224.0.1.40 does not cross the boundary-facing interface |
| 23 | Verify Auto-RP announce group is blocked where intended | Boundary and mapping-agent routers | `show ip mroute 224.0.1.39` | 224.0.1.39 does not cross the boundary-facing interface |
| 24 | Confirm required RP mappings still exist for allowed groups | All required multicast routers | `show ip pim rp mapping` | Allowed multicast group ranges still map to the correct RP |
| 25 | Confirm RPF remains correct for allowed groups | Downstream routers | `show ip rpf <SOURCE_IP>`<br>`show ip rpf <RP_ADDRESS>` | Allowed group forwarding still has valid RPF paths |
| 26 | Use controlled PIM or multicast debugging only if state is unclear | Boundary router | `debug ip pim` | Boundary-related control-plane behavior is visible during test |
| 27 | Disable debugging after the test | Boundary router | `undebug all` | Debugging is disabled |
| 28 | Save the working boundary configuration | Boundary router | `copy running-config startup-config` | Multicast boundary policy is preserved |
# Multicast_Admin_Boundary_And_Scope_Control_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <BOUNDARY_INTERFACE>          = interface where multicast should stop crossing
! <BOUNDARY_ACL>                = standard ACL used by ip multicast boundary
! <BLOCKED_MULTICAST_GROUP>     = data multicast group to block
! <ALLOWED_MULTICAST_GROUP>     = data multicast group to allow
! <SOURCE_INTERFACE_OR_IP>      = source interface or IP used for multicast test traffic
! <SOURCE_IP>                   = multicast source IP
! <RP_ADDRESS>                  = RP address for sparse-mode groups
! =========================================================
! BASELINE MULTICAST
! =========================================================
conf t
 ip multicast-routing
 interface <BOUNDARY_INTERFACE>
  ip pim sparse-dense-mode
 exit
end
! =========================================================
! BOUNDARY ACL
! Denied multicast groups are blocked.
! Permitted multicast groups are allowed.
! Always add permit any when the policy is deny-specific.
! =========================================================
conf t
 ip access-list standard <BOUNDARY_ACL>
  deny <BLOCKED_MULTICAST_GROUP>
  permit any
 exit
end
! =========================================================
! AUTO-RP DISCOVERY BOUNDARY EXAMPLE
! Blocks RP-Discovery messages from crossing the interface.
! 224.0.1.40 = Auto-RP RP-Discovery
! =========================================================
conf t
 ip access-list standard <BOUNDARY_ACL>
  deny host 224.0.1.40
  permit any
 exit
end
! =========================================================
! AUTO-RP ANNOUNCE BOUNDARY EXAMPLE
! Blocks Candidate RP announcements from crossing the interface.
! 224.0.1.39 = Auto-RP RP-Announce
! =========================================================
conf t
 ip access-list standard <BOUNDARY_ACL>
  deny host 224.0.1.39
  permit any
 exit
end
! =========================================================
! APPLY MULTICAST BOUNDARY
! =========================================================
conf t
 interface <BOUNDARY_INTERFACE>
  ip multicast boundary <BOUNDARY_ACL>
 exit
end
! =========================================================
! CLEAN STATE BEFORE TESTING
! =========================================================
clear ip mroute *
! =========================================================
! TEST BLOCKED GROUP
! =========================================================
ping <BLOCKED_MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! TEST ALLOWED GROUP
! =========================================================
ping <ALLOWED_MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show access-lists <BOUNDARY_ACL>
show running-config interface <BOUNDARY_INTERFACE>
show ip mroute <BLOCKED_MULTICAST_GROUP>
show ip mroute <ALLOWED_MULTICAST_GROUP>
show ip mroute 224.0.1.39
show ip mroute 224.0.1.40
show ip pim rp mapping
show ip rpf <SOURCE_IP>
show ip rpf <RP_ADDRESS>
# Multicast_Admin_Boundary_And_Scope_Control_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | Boundary router | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | Boundary router | `show ip pim interface <BOUNDARY_INTERFACE>` | Boundary-facing interface appears as PIM-enabled |
| Boundary ACL contents | Boundary router | `show access-lists <BOUNDARY_ACL>` | ACL denies blocked groups and permits allowed groups |
| Boundary applied | Boundary router | `show running-config interface <BOUNDARY_INTERFACE>` | Interface contains `ip multicast boundary <BOUNDARY_ACL>` |
| Blocked data group state | Downstream router | `show ip mroute <BLOCKED_MULTICAST_GROUP>` | Blocked group is absent or not forwarded beyond boundary |
| Allowed data group state | Downstream router | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Allowed group has expected multicast state |
| Auto-RP announce group | Boundary and mapping-agent routers | `show ip mroute 224.0.1.39` | RP-Announce group crosses only where allowed |
| Auto-RP discovery group | Boundary and downstream routers | `show ip mroute 224.0.1.40` | RP-Discovery group crosses only where allowed |
| RP mapping downstream | Downstream router | `show ip pim rp mapping` | Mapping exists when Auto-RP discovery is allowed, expires or disappears when blocked |
| RP mapping upstream | Upstream router | `show ip pim rp mapping` | Upstream mapping remains intact if only downstream boundary is blocking |
| RPF to source | Downstream router | `show ip rpf <SOURCE_IP>` | Allowed group source path has valid RPF |
| RPF to RP | Downstream router | `show ip rpf <RP_ADDRESS>` | Sparse-mode shared-tree path has valid RPF |
| OIL check | Boundary router | `show ip mroute <MULTICAST_GROUP>` | Boundary-facing interface is absent from OIL for blocked groups |
| Allowed traffic test | Source/test router | `ping <ALLOWED_MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies or packet counters increment |
| Blocked traffic test | Source/test router | `ping <BLOCKED_MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | No receiver response beyond the boundary |
| Debug cleanup | Boundary router | `show debugging`<br>`undebug all` | No multicast debug remains active |
# Multicast_Admin_Boundary_And_Scope_Control_Rollback
! =========================================================
! REMOVE MULTICAST BOUNDARY FROM INTERFACE
! =========================================================
conf t
 interface <BOUNDARY_INTERFACE>
  no ip multicast boundary <BOUNDARY_ACL>
 exit
end
! =========================================================
! REMOVE BOUNDARY ACL
! =========================================================
conf t
 no ip access-list standard <BOUNDARY_ACL>
end
! =========================================================
! REMOVE PIM FROM BOUNDARY INTERFACE
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <BOUNDARY_INTERFACE>
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
# Multicast_Admin_Boundary_And_Scope_Control_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Blocked group still crosses boundary | ACL permits the group or boundary is applied to wrong interface | Boundary router | `show access-lists <BOUNDARY_ACL>`<br>`show running-config interface <BOUNDARY_INTERFACE>` | Deny the correct group and apply `ip multicast boundary` on the correct interface |
| All multicast stops crossing | ACL lacks `permit any` and implicit deny blocks everything else | Boundary router | `show access-lists <BOUNDARY_ACL>` | Add `permit any` if only specific groups should be blocked |
| Auto-RP mapping disappears downstream | Boundary blocks 224.0.1.40 RP-Discovery | Downstream router | `show ip pim rp mapping`<br>`show ip mroute 224.0.1.40` | Permit 224.0.1.40 or place mapping agent inside the scoped region |
| Mapping agent never learns candidate RP | Boundary blocks 224.0.1.39 RP-Announce | Mapping-agent and boundary routers | `show ip mroute 224.0.1.39` | Permit 224.0.1.39 between candidate RP and mapping agent |
| Blocked group still has stale state | Old mroute state has not aged out | Boundary and downstream routers | `show ip mroute <BLOCKED_MULTICAST_GROUP>` | Run `clear ip mroute *` and retest |
| Boundary ACL matches receiver IPs instead of groups | Standard ACL was built with host/receiver addresses instead of multicast group addresses | Boundary router | `show access-lists <BOUNDARY_ACL>` | Rewrite ACL to match multicast group addresses |
| Boundary applied but PIM state is missing entirely | PIM or multicast routing is not enabled | Boundary router | `show ip pim interface`<br>`show running-config | include ip multicast-routing` | Enable `ip multicast-routing` and PIM on required interfaces |
| Allowed group fails after boundary is added | ACL order blocks the allowed group or RP mapping/RPF failed | Boundary and downstream routers | `show access-lists <BOUNDARY_ACL>`<br>`show ip pim rp mapping`<br>`show ip rpf <SOURCE_IP>` | Correct ACL order, RP mapping, or RPF route |
| Boundary only works one direction unexpectedly | Boundary placement does not match the actual multicast forwarding direction | Boundary router | `show ip mroute <GROUP>` | Apply boundary on the interface where the group must be contained |
| `show ip pim rp mapping` still shows old mapping after blocking 224.0.1.40 | Auto-RP mapping has not expired yet | Downstream router | `show ip pim rp mapping` | Wait for expiration or clear multicast state |
| Receiver still responds to blocked group | Receiver is on same side of the boundary as the source or another path bypasses the boundary | Boundary and receiver-side routers | `show ip mroute <BLOCKED_MULTICAST_GROUP>` | Move boundary to the correct choke point or block all alternate multicast paths |
| OIL still includes boundary interface for blocked group | ACL did not match group or stale state remains | Boundary router | `show ip mroute <BLOCKED_MULTICAST_GROUP>`<br>`show access-lists <BOUNDARY_ACL>` | Correct ACL and clear multicast state |
| Auto-RP control groups are blocked unintentionally | Boundary ACL denies 224.0.1.39 or 224.0.1.40 too broadly | Boundary router | `show access-lists <BOUNDARY_ACL>` | Permit required Auto-RP control groups or use a more specific scope design |
| Debug floods console | Multicast debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# Multicast_Admin_Boundary_And_Scope_Control_Mental_Model
# Multicast_Admin_Boundary_And_Scope_Control_Configuration_Checklist
# Multicast_Admin_Boundary_And_Scope_Control_Skeleton
# Multicast_Admin_Boundary_And_Scope_Control_Verification_Commands
# Multicast_Admin_Boundary_And_Scope_Control_Rollback
# Multicast_Admin_Boundary_And_Scope_Control_Failure_Checks