
IGMP_Snooping_Layer2_Multicast_Containment.md

IGMP_Snooping_Layer2_Multicast_Containment

# IGMP_Snooping_Layer2_Multicast_Containment_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IGMP snooping | Layer 2 switch feature that listens to IGMP messages and builds a multicast forwarding table per VLAN |
| Problem solved | Without snooping, unknown multicast in a VLAN behaves like broadcast and can flood all switchports |
| Containment goal | Forward multicast only to ports with interested receivers and to multicast-router ports |
| VLAN scoped | IGMP snooping is applied and verified per VLAN |
| Receiver port | A switchport where an attached host or test router sends an IGMP membership report |
| Router port / mrouter port | A switchport leading toward a multicast router, IGMP querier, or PIM router |
| IGMP querier dependency | Snooping works best when an IGMP querier exists in the VLAN to keep receiver membership state refreshed |
| Switch querier | A Layer 2 switch can act as the IGMP querier in a VLAN when no routed multicast interface exists |
| Static mrouter port | Manual way to tell the switch where multicast-router traffic should be forwarded |
| Static group entry | Manual way to bind a multicast group to one or more switchports |
| Immediate leave | Removes a receiver port quickly after an IGMP leave. Use carefully, mainly when only one receiver exists per access port |
| Not multicast routing | IGMP snooping does not replace PIM, RP, RPF, or `show ip mroute`. It only optimizes Layer 2 forwarding inside the VLAN |
| Verification truth | `show ip igmp snooping`, `show ip igmp snooping vlan <VLAN_ID>`, `show ip igmp snooping groups`, and traffic counters prove containment |
# IGMP_Snooping_Layer2_Multicast_Containment_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the multicast receiver VLAN | Layer 2 switch | `show vlan brief` | Target VLAN exists and receiver ports are assigned to it |
| 2 | Identify receiver-facing access ports | Layer 2 switch | `show interfaces status` | Receiver access ports are connected and in the expected VLAN |
| 3 | Identify the multicast-router-facing port | Layer 2 switch | `show interfaces trunk`<br>`show cdp neighbors` | Uplink or routed neighbor toward multicast router is known |
| 4 | Confirm IGMP snooping global state | Layer 2 switch | `show ip igmp snooping` | IGMP snooping is enabled globally |
| 5 | Enable IGMP snooping globally if disabled | Layer 2 switch | `conf t`<br>`ip igmp snooping` | Switch can perform IGMP snooping globally |
| 6 | Confirm IGMP snooping state for the VLAN | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | VLAN shows IGMP snooping enabled |
| 7 | Enable IGMP snooping for the VLAN if disabled | Layer 2 switch | `conf t`<br>`ip igmp snooping vlan <VLAN_ID>` | Snooping is active for the receiver VLAN |
| 8 | Confirm receiver ports are in the correct access VLAN | Layer 2 switch | `show running-config interface <RECEIVER_ACCESS_INTERFACE>` | Interface is configured as access port in `<VLAN_ID>` |
| 9 | Configure receiver access port if needed | Layer 2 switch | `conf t`<br>`interface <RECEIVER_ACCESS_INTERFACE>`<br>`switchport mode access`<br>`switchport access vlan <VLAN_ID>` | Receiver port is placed into the multicast receiver VLAN |
| 10 | Confirm multicast router uplink carries the VLAN | Layer 2 switch | `show interfaces trunk` | `<VLAN_ID>` is allowed and forwarding on the router-facing trunk |
| 11 | Configure the router-facing trunk if needed | Layer 2 switch | `conf t`<br>`interface <MROUTER_UPLINK_INTERFACE>`<br>`switchport mode trunk`<br>`switchport trunk allowed vlan add <VLAN_ID>` | Multicast router uplink carries the receiver VLAN |
| 12 | Verify whether a querier is present in the VLAN | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | Output shows an IGMP querier address if one exists |
| 13 | Enable switch IGMP querier if no routed querier exists | Layer 2 switch | `conf t`<br>`ip igmp snooping querier` | Switch can generate IGMP queries for snooping state maintenance |
| 14 | Configure VLAN-specific querier address if required | Layer 2 switch | `conf t`<br>`ip igmp snooping vlan <VLAN_ID> querier address <QUERIER_IP>` | Querier source address is deterministic for the VLAN |
| 15 | Verify querier operation | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | VLAN shows switch querier enabled or querier present |
| 16 | Manually define a multicast-router port if automatic detection fails | Layer 2 switch | `conf t`<br>`ip igmp snooping vlan <VLAN_ID> mrouter interface <MROUTER_UPLINK_INTERFACE>` | Switch forwards multicast control/data toward the multicast router port |
| 17 | Verify multicast-router port detection | Layer 2 switch | `show ip igmp snooping mrouter vlan <VLAN_ID>` | Expected router-facing interface appears as an mrouter port |
| 18 | Create receiver interest from a test receiver | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver sends IGMP membership report for the group |
| 19 | Verify snooped group membership on the switch | Layer 2 switch | `show ip igmp snooping groups vlan <VLAN_ID>` | Multicast group maps to the receiver-facing access port |
| 20 | Verify last-hop router sees IGMP membership if a routed multicast gateway exists | Last-hop multicast router | `show ip igmp groups` | Group appears on the VLAN-facing routed interface |
| 21 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Multicast traffic is generated toward the receiver group |
| 22 | Verify multicast is constrained to interested ports | Layer 2 switch | `show mac address-table multicast vlan <VLAN_ID>`<br>`show ip igmp snooping groups vlan <VLAN_ID>` | Multicast group points only to receiver and mrouter ports |
| 23 | Add a second receiver and confirm membership expansion | Receiver/test router 2 | `conf t`<br>`interface <RECEIVER_INTERFACE_2>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Snooping group entry adds the second receiver port |
| 24 | Remove receiver membership and observe aging/leave behavior | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`no ip igmp join-group <MULTICAST_GROUP>` | Receiver port is removed after leave processing or timeout |
| 25 | Enable immediate leave only for single-host access-port testing | Layer 2 switch | `conf t`<br>`ip igmp snooping vlan <VLAN_ID> immediate-leave` | Port is removed quickly after IGMP leave |
| 26 | Verify immediate-leave behavior | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | VLAN shows fast/immediate leave behavior enabled if supported |
| 27 | Save the working IGMP snooping configuration | Layer 2 switch | `copy running-config startup-config` | Layer 2 multicast containment baseline is preserved |
# IGMP_Snooping_Layer2_Multicast_Containment_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <VLAN_ID>                   = multicast receiver VLAN
! <RECEIVER_ACCESS_INTERFACE> = switchport facing receiver
! <RECEIVER_ACCESS_INTERFACE_2> = optional second receiver switchport
! <MROUTER_UPLINK_INTERFACE>  = switchport/trunk toward multicast router
! <RECEIVER_INTERFACE>        = router/host interface used to join group
! <RECEIVER_INTERFACE_2>      = optional second receiver interface
! <MULTICAST_GROUP>           = multicast group being joined
! <QUERIER_IP>                = source IP for switch IGMP querier
! <SOURCE_INTERFACE_OR_IP>    = source interface or source IP for multicast test
! =========================================================
! LAYER 2 SWITCH BASELINE
! =========================================================
conf t
 vlan <VLAN_ID>
 exit
 ip igmp snooping
 ip igmp snooping vlan <VLAN_ID>
end
! =========================================================
! RECEIVER ACCESS PORT
! =========================================================
conf t
 interface <RECEIVER_ACCESS_INTERFACE>
  switchport mode access
  switchport access vlan <VLAN_ID>
 exit
end
! =========================================================
! OPTIONAL SECOND RECEIVER ACCESS PORT
! =========================================================
conf t
 interface <RECEIVER_ACCESS_INTERFACE_2>
  switchport mode access
  switchport access vlan <VLAN_ID>
 exit
end
! =========================================================
! MULTICAST ROUTER UPLINK
! Use trunk if the multicast router is reached across a trunk.
! =========================================================
conf t
 interface <MROUTER_UPLINK_INTERFACE>
  switchport mode trunk
  switchport trunk allowed vlan add <VLAN_ID>
 exit
end
! =========================================================
! SWITCH IGMP QUERIER
! Use only when no routed multicast gateway/querier exists
! in the VLAN.
! =========================================================
conf t
 ip igmp snooping querier
 ip igmp snooping vlan <VLAN_ID> querier address <QUERIER_IP>
end
! =========================================================
! STATIC MULTICAST ROUTER PORT
! Use only if automatic mrouter detection fails.
! =========================================================
conf t
 ip igmp snooping vlan <VLAN_ID> mrouter interface <MROUTER_UPLINK_INTERFACE>
end
! =========================================================
! IMMEDIATE LEAVE
! Use mainly for single-host access ports.
! Do not use blindly on ports with multiple receivers behind them.
! =========================================================
conf t
 ip igmp snooping vlan <VLAN_ID> immediate-leave
end
! =========================================================
! TEST RECEIVER JOIN
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
show vlan brief
show ip igmp snooping
show ip igmp snooping vlan <VLAN_ID>
show ip igmp snooping groups vlan <VLAN_ID>
show ip igmp snooping mrouter vlan <VLAN_ID>
show mac address-table multicast vlan <VLAN_ID>
# IGMP_Snooping_Layer2_Multicast_Containment_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| VLAN exists | Layer 2 switch | `show vlan brief` | `<VLAN_ID>` exists and receiver access ports are assigned |
| Receiver access port state | Layer 2 switch | `show interfaces status` | Receiver port is connected and in the expected VLAN |
| Trunk carries VLAN | Layer 2 switch | `show interfaces trunk` | `<VLAN_ID>` is allowed and forwarding on the mrouter uplink |
| Global snooping state | Layer 2 switch | `show ip igmp snooping` | IGMP snooping is enabled globally |
| VLAN snooping state | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | IGMP snooping is enabled for the VLAN |
| Querier presence | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | Querier address is present or switch querier is enabled |
| Router-port detection | Layer 2 switch | `show ip igmp snooping mrouter vlan <VLAN_ID>` | Expected multicast-router port is listed |
| Snooped group membership | Layer 2 switch | `show ip igmp snooping groups vlan <VLAN_ID>` | Multicast group maps to receiver access ports |
| Multicast MAC forwarding | Layer 2 switch | `show mac address-table multicast vlan <VLAN_ID>` | Multicast MAC/group entry points to expected ports only |
| Receiver local membership | Receiver/test router | `show ip igmp groups` | Joined group appears locally |
| Last-hop router membership | Multicast router | `show ip igmp groups` | Group appears on VLAN-facing routed interface |
| Multicast route state | Multicast router | `show ip mroute <MULTICAST_GROUP>` | OIL includes VLAN-facing interface when receiver interest exists |
| Traffic test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver responds if using router `join-group` and multicast routing is correct |
| Counter check | Switch | `show interfaces <RECEIVER_ACCESS_INTERFACE> counters` | Receiver port counters increment during multicast traffic |
| Nonreceiver containment check | Switch | `show interfaces <NONRECEIVER_ACCESS_INTERFACE> counters` | Nonreceiver port should not show the same multicast traffic increase |
# IGMP_Snooping_Layer2_Multicast_Containment_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOINS
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
 interface <RECEIVER_INTERFACE_2>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE IMMEDIATE LEAVE
! =========================================================
conf t
 no ip igmp snooping vlan <VLAN_ID> immediate-leave
end
! =========================================================
! REMOVE STATIC MROUTER PORT
! =========================================================
conf t
 no ip igmp snooping vlan <VLAN_ID> mrouter interface <MROUTER_UPLINK_INTERFACE>
end
! =========================================================
! REMOVE SWITCH QUERIER CONFIGURATION
! =========================================================
conf t
 no ip igmp snooping vlan <VLAN_ID> querier address <QUERIER_IP>
 no ip igmp snooping querier
end
! =========================================================
! DISABLE VLAN-SPECIFIC SNOOPING ONLY IF THE LAB REQUIRES CLEAN REMOVAL
! =========================================================
conf t
 no ip igmp snooping vlan <VLAN_ID>
end
! =========================================================
! DISABLE GLOBAL IGMP SNOOPING ONLY IN A DISPOSABLE LAB
! Do not do this in a production-like campus switch unless required.
! =========================================================
conf t
 no ip igmp snooping
end
! =========================================================
! OPTIONAL PORT CLEANUP
! =========================================================
conf t
 interface <RECEIVER_ACCESS_INTERFACE>
  no switchport access vlan <VLAN_ID>
 exit
 interface <MROUTER_UPLINK_INTERFACE>
  switchport trunk allowed vlan remove <VLAN_ID>
 exit
end
! =========================================================
! CLEAR DYNAMIC STATE
! Command support varies by platform.
! =========================================================
clear ip igmp snooping
clear mac address-table dynamic vlan <VLAN_ID>
# IGMP_Snooping_Layer2_Multicast_Containment_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Multicast floods all ports in the VLAN | IGMP snooping disabled globally or for the VLAN | Layer 2 switch | `show ip igmp snooping`<br>`show ip igmp snooping vlan <VLAN_ID>` | Configure `ip igmp snooping` and `ip igmp snooping vlan <VLAN_ID>` |
| Snooping is enabled but group entries never appear | No receiver IGMP reports are being seen | Layer 2 switch | `show ip igmp snooping groups vlan <VLAN_ID>` | Verify receiver join and receiver port VLAN assignment |
| Receiver joined but switch does not learn group | Receiver is in the wrong VLAN or wrong access port | Layer 2 switch | `show vlan brief`<br>`show running-config interface <RECEIVER_ACCESS_INTERFACE>` | Put receiver port in the correct VLAN |
| Group entries age out repeatedly | No IGMP querier exists in the VLAN | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | Enable routed querier/SVI or configure switch IGMP querier |
| Multicast router does not receive joins | Mrouter port not detected | Layer 2 switch | `show ip igmp snooping mrouter vlan <VLAN_ID>` | Add static mrouter port or fix PIM/IGMP query detection |
| Router-facing trunk does not carry multicast VLAN | VLAN missing from trunk allowed list | Layer 2 switch | `show interfaces trunk` | Add VLAN to trunk allowed list |
| Nonreceiver port still gets multicast traffic | Unknown multicast flooding, no group entry, or snooping unsupported/disabled | Layer 2 switch | `show ip igmp snooping groups vlan <VLAN_ID>`<br>`show mac address-table multicast vlan <VLAN_ID>` | Fix snooping, querier, and receiver group learning |
| Receiver loses traffic after another receiver leaves | Immediate leave enabled on a port with multiple receivers behind it | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | Disable immediate leave on shared receiver ports |
| Static mrouter port does not work | Wrong interface or wrong VLAN selected | Layer 2 switch | `show running-config | include mrouter`<br>`show ip igmp snooping mrouter vlan <VLAN_ID>` | Correct the static mrouter interface and VLAN |
| Switch querier fails to appear | Querier address missing, unsupported syntax, or another querier is already active | Layer 2 switch | `show ip igmp snooping vlan <VLAN_ID>` | Configure valid querier address or use routed SVI/gateway querier |
| Last-hop router has no IGMP groups | Layer 2 snooping is fine, but routed IGMP/PIM is missing or receiver never joined | Multicast router | `show ip igmp groups`<br>`show ip igmp interface` | Enable PIM/IGMP on routed VLAN interface and verify receiver joins |
| Multicast route OIL is empty | IGMP snooping is not the problem; upstream multicast routing has no receiver interest or PIM state | Multicast router | `show ip mroute <MULTICAST_GROUP>` | Fix IGMP membership, PIM, RP, or RPF upstream |
| Command rejected on IOSv-L2 | Image does not support the exact snooping subcommand | Layer 2 switch | `show version`<br>`show ip igmp snooping ?` | Use supported syntax for that image or test on IOS XE/NX-OS-capable switch image |
| Clear command fails | Platform uses different clear syntax | Layer 2 switch | `clear ip igmp ?`<br>`clear mac address-table ?` | Use platform-supported clear command or wait for state aging |
##### Source_Basis
# IGMP_Snooping_Layer2_Multicast_Containment_Mental_Model
# IGMP_Snooping_Layer2_Multicast_Containment_Configuration_Checklist
# IGMP_Snooping_Layer2_Multicast_Containment_Skeleton
# IGMP_Snooping_Layer2_Multicast_Containment_Verification_Commands
# IGMP_Snooping_Layer2_Multicast_Containment_Rollback
# IGMP_Snooping_Layer2_Multicast_Containment_Failure_Checks

