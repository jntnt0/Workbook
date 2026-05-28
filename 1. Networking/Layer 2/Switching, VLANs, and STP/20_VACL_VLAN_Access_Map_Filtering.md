


VACL_VLAN_Access_Map_Filtering.md
# VACL_VLAN_Access_Map_Filtering

# VACL_VLAN_Access_Map_Filtering_Index

- VACL_VLAN_Access_Map_Filtering_Mental_Model
- VACL_VLAN_Access_Map_Filtering_Configuration_Checklist
- VACL_VLAN_Access_Map_Filtering_Skeleton
- VACL_VLAN_Access_Map_Filtering_Verification_Commands
- VACL_VLAN_Access_Map_Filtering_Rollback
- VACL_VLAN_Access_Map_Filtering_Failure_Checks

# VACL_VLAN_Access_Map_Filtering_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VACL | VLAN access control list that filters traffic inside a VLAN or traffic entering and leaving a VLAN |
| VLAN access-map | Policy object that contains ordered sequences with match and action logic |
| Match ACL | ACL used only as a classifier for the VLAN access-map sequence |
| ACL permit in VACL context | `permit` inside the ACL means "match this traffic", not automatically "allow this traffic" |
| Action statement | The VLAN access-map action decides whether matched traffic is forwarded or dropped |
| Drop action | `action drop` blocks traffic matched by that sequence |
| Drop log action | `action drop log` drops matched traffic and logs supported drop events |
| Forward action | `action forward` permits matched traffic through the VLAN filter |
| Sequence order | Lower sequence numbers are evaluated first |
| Final forward sequence | A final catch-all forward sequence prevents unplanned blocking of traffic not explicitly matched earlier |
| VLAN filter | `vlan filter <map-name> vlan-list <vlans>` attaches the VLAN access-map to one or more VLANs |
| Bridged traffic filtering | VACLs can filter same-VLAN host-to-host traffic that a routed ACL would not see |
| Routed traffic filtering | VACLs can also filter traffic routed into or out of a VLAN |
| PACL difference | PACLs apply to a switchport; VACLs apply to the VLAN |
| RACL difference | Router ACLs apply to Layer 3 interfaces; VACLs apply at the VLAN forwarding layer |
| Direction model | VACLs are not configured as simple inbound or outbound interface ACLs; they are attached to VLANs |
| IP match | `match ip address <acl-name>` matches IPv4 traffic using a standard, extended, or named IPv4 ACL |
| MAC match | `match mac address <acl-name>` matches non-IP or Layer 2 traffic using a named MAC ACL |
| Operational danger | A VACL with only drop entries and no final forward behavior can break more traffic than intended |
| Verification rule | Always verify ACL matches, VLAN access-map sequences, VLAN filter attachment, MAC learning, and the actual traffic matrix |

# VACL_VLAN_Access_Map_Filtering_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the switch supports VLAN access maps | SW1 | `show vlan access-map` | Command is accepted and current VLAN access-map state is shown |
| 2 | Confirm target VLAN exists | SW1 | `show vlan brief` | VLAN 20 exists and is active |
| 3 | Confirm host-facing ports are in the target VLAN | SW1 | `show vlan brief` | Target access ports appear under VLAN 20 |
| 4 | Confirm physical host links are up | SW1 | `show interfaces status` | Host-facing interfaces show `connected` |
| 5 | Confirm baseline same-VLAN reachability before filtering | H1 | `ping <h2-vlan-20-ip>` | Ping succeeds before VACL is applied |
| 6 | Confirm baseline TCP reachability before filtering if testing Telnet | H1 | `telnet <h2-vlan-20-ip> 23` | Telnet test result is known before VACL is applied |
| 7 | Confirm no existing VACL is already attached to VLAN 20 | SW1 | `show running-config | include vlan filter` | No unexpected `vlan filter` is attached to VLAN 20 |
| 8 | Confirm no existing VLAN access-map conflicts with the planned name | SW1 | `show vlan access-map` | Existing access-map names and sequence numbers are known |
| 9 | Enter global configuration mode | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 10 | Create extended ACL to classify ICMP traffic | SW1 | `ip access-list extended VACL_MATCH_ICMP` | Prompt changes to named ACL configuration mode |
| 11 | Match ICMP traffic | SW1 | `permit icmp any any` | ICMP traffic is classified for the VACL |
| 12 | Exit ICMP ACL configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 13 | Create extended ACL to classify Telnet traffic | SW1 | `ip access-list extended VACL_MATCH_TELNET` | Prompt changes to named ACL configuration mode |
| 14 | Match Telnet traffic | SW1 | `permit tcp any any eq 23` | Telnet traffic is classified for the VACL |
| 15 | Exit Telnet ACL configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 16 | Create extended ACL to classify other IPv4 traffic | SW1 | `ip access-list extended VACL_MATCH_OTHER_IP` | Prompt changes to named ACL configuration mode |
| 17 | Match all remaining IPv4 traffic | SW1 | `permit ip any any` | Remaining IPv4 traffic is classified for forwarding |
| 18 | Exit catch-all IPv4 ACL configuration | SW1 | `exit` | Prompt returns to global configuration mode |
| 19 | Create first VLAN access-map sequence for ICMP | SW1 | `vlan access-map VACL_VLAN20_FILTER 10` | Prompt changes to VLAN access-map configuration mode |
| 20 | Match ICMP classifier ACL | SW1 | `match ip address VACL_MATCH_ICMP` | Sequence 10 matches ICMP traffic |
| 21 | Drop matched ICMP traffic | SW1 | `action drop` | ICMP traffic matched by sequence 10 will be dropped |
| 22 | Exit sequence 10 | SW1 | `exit` | Prompt returns to global configuration mode |
| 23 | Create second VLAN access-map sequence for Telnet | SW1 | `vlan access-map VACL_VLAN20_FILTER 20` | Prompt changes to VLAN access-map configuration mode |
| 24 | Match Telnet classifier ACL | SW1 | `match ip address VACL_MATCH_TELNET` | Sequence 20 matches Telnet traffic |
| 25 | Drop and log matched Telnet traffic | SW1 | `action drop log` | Telnet traffic matched by sequence 20 will be dropped and logged if supported |
| 26 | Exit sequence 20 | SW1 | `exit` | Prompt returns to global configuration mode |
| 27 | Create final IPv4 forward sequence | SW1 | `vlan access-map VACL_VLAN20_FILTER 30` | Prompt changes to VLAN access-map configuration mode |
| 28 | Match all remaining IPv4 traffic | SW1 | `match ip address VACL_MATCH_OTHER_IP` | Sequence 30 matches remaining IPv4 traffic |
| 29 | Forward remaining IPv4 traffic | SW1 | `action forward` | Non-ICMP and non-Telnet IPv4 traffic is permitted |
| 30 | Exit sequence 30 | SW1 | `exit` | Prompt returns to global configuration mode |
| 31 | Apply the VLAN access-map to VLAN 20 | SW1 | `vlan filter VACL_VLAN20_FILTER vlan-list 20` | VACL is attached to VLAN 20 |
| 32 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 33 | Verify the VLAN access-map sequences | SW1 | `show vlan access-map` | Sequences 10, 20, and 30 show correct match and action logic |
| 34 | Verify the VACL is applied to VLAN 20 | SW1 | `show running-config | include vlan filter` | `vlan filter VACL_VLAN20_FILTER vlan-list 20` is present |
| 35 | Verify ACL classifier entries | SW1 | `show access-lists VACL_MATCH_ICMP` | ICMP match ACL exists |
| 36 | Verify Telnet classifier entries | SW1 | `show access-lists VACL_MATCH_TELNET` | Telnet match ACL exists |
| 37 | Verify catch-all IPv4 classifier entries | SW1 | `show access-lists VACL_MATCH_OTHER_IP` | IPv4 catch-all ACL exists |
| 38 | Test ICMP is blocked | H1 | `ping <h2-vlan-20-ip>` | Ping fails after VACL is applied |
| 39 | Test Telnet is blocked if a Telnet listener exists | H1 | `telnet <h2-vlan-20-ip> 23` | Telnet fails after VACL is applied |
| 40 | Test non-filtered IPv4 traffic still works | H1 | `ssh <h2-vlan-20-ip>` or application-specific test | Non-filtered IPv4 traffic works if permitted by endpoints |
| 41 | Verify VLAN 20 MAC learning remains stable | SW1 | `show mac address-table dynamic vlan 20` | Host MAC addresses remain learned on expected ports |
| 42 | Verify STP is not the reason traffic is failing | SW1 | `show spanning-tree vlan 20` | Host-facing and trunk ports are forwarding as expected |
| 43 | Check for drop logs if using `action drop log` | SW1 | `show logging | include VACL|ACCESS|Deny|drop` | Supported VACL log messages appear for logged drops |
| 44 | Save configuration | SW1 | `write memory` | VACL configuration survives reload |

# VACL_VLAN_Access_Map_Filtering_Skeleton

Block ICMP and Telnet inside VLAN 20, forward other IPv4 traffic:

configure terminal
!
ip access-list extended VACL_MATCH_ICMP
 permit icmp any any
 exit
!
ip access-list extended VACL_MATCH_TELNET
 permit tcp any any eq 23
 exit
!
ip access-list extended VACL_MATCH_OTHER_IP
 permit ip any any
 exit
!
vlan access-map VACL_VLAN20_FILTER 10
 match ip address VACL_MATCH_ICMP
 action drop
 exit
!
vlan access-map VACL_VLAN20_FILTER 20
 match ip address VACL_MATCH_TELNET
 action drop log
 exit
!
vlan access-map VACL_VLAN20_FILTER 30
 match ip address VACL_MATCH_OTHER_IP
 action forward
 exit
!
vlan filter VACL_VLAN20_FILTER vlan-list 20
!
end
write memory

Forward-all final sequence alternative:

configure terminal
!
vlan access-map VACL_VLAN20_FILTER 100
 action forward
 exit
!
end
write memory

Remove VACL from VLAN before editing heavily:

configure terminal
!
no vlan filter VACL_VLAN20_FILTER vlan-list 20
!
end

Reapply VACL after edit:

configure terminal
!
vlan filter VACL_VLAN20_FILTER vlan-list 20
!
end
write memory

Block only ICMP between two specific hosts in VLAN 20:

configure terminal
!
ip access-list extended VACL_MATCH_H1_H2_ICMP
 permit icmp host 192.0.2.11 host 192.0.2.12
 permit icmp host 192.0.2.12 host 192.0.2.11
 exit
!
ip access-list extended VACL_MATCH_OTHER_IP
 permit ip any any
 exit
!
vlan access-map VACL_VLAN20_FILTER 10
 match ip address VACL_MATCH_H1_H2_ICMP
 action drop
 exit
!
vlan access-map VACL_VLAN20_FILTER 20
 match ip address VACL_MATCH_OTHER_IP
 action forward
 exit
!
vlan filter VACL_VLAN20_FILTER vlan-list 20
!
end
write memory

MAC ACL match example for non-IP or Layer 2 filtering:

configure terminal
!
mac access-list extended VACL_MATCH_MAC
 permit host 0011.2233.4455 any
 exit
!
vlan access-map VACL_MAC_FILTER 10
 match mac address VACL_MATCH_MAC
 action drop
 exit
!
vlan access-map VACL_MAC_FILTER 20
 action forward
 exit
!
vlan filter VACL_MAC_FILTER vlan-list 20
!
end
write memory

# VACL_VLAN_Access_Map_Filtering_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm VLAN exists | SW1 | `show vlan brief` | VLAN 20 exists and is active |
| Confirm host ports are in VLAN 20 | SW1 | `show vlan brief` | Host-facing interfaces appear under VLAN 20 |
| Confirm VLAN access-map exists | SW1 | `show vlan access-map` | VACL name and sequences are visible |
| Confirm sequence 10 ICMP drop | SW1 | `show vlan access-map` | Sequence 10 matches ICMP ACL and drops |
| Confirm sequence 20 Telnet drop | SW1 | `show vlan access-map` | Sequence 20 matches Telnet ACL and drops with log if configured |
| Confirm sequence 30 forward | SW1 | `show vlan access-map` | Sequence 30 forwards remaining IPv4 traffic |
| Confirm VLAN filter attachment | SW1 | `show running-config | include vlan filter` | VACL is attached to VLAN 20 |
| Confirm classifier ACLs | SW1 | `show access-lists` | VACL match ACLs are present |
| Confirm ICMP match ACL | SW1 | `show access-lists VACL_MATCH_ICMP` | ACL permits ICMP as match traffic |
| Confirm Telnet match ACL | SW1 | `show access-lists VACL_MATCH_TELNET` | ACL permits TCP destination port 23 as match traffic |
| Confirm catch-all IPv4 match ACL | SW1 | `show access-lists VACL_MATCH_OTHER_IP` | ACL permits remaining IPv4 traffic |
| Confirm VACL running config | SW1 | `show running-config | section vlan access-map` | VLAN access-map sequences are present |
| Confirm VACL filter running config | SW1 | `show running-config | include vlan filter` | VLAN filter line is present |
| Confirm physical ports | SW1 | `show interfaces status` | Required ports are connected |
| Confirm switchport VLAN assignment | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Host port is in VLAN 20 |
| Confirm STP forwarding | SW1 | `show spanning-tree vlan 20` | Required ports are forwarding |
| Confirm MAC learning | SW1 | `show mac address-table dynamic vlan 20` | Host MACs are learned on expected ports |
| Confirm ICMP is blocked | H1 | `ping <h2-vlan-20-ip>` | Ping fails when ICMP drop sequence is active |
| Confirm Telnet is blocked | H1 | `telnet <h2-vlan-20-ip> 23` | Telnet fails when Telnet drop sequence is active |
| Confirm non-filtered traffic is forwarded | H1 | `ssh <h2-vlan-20-ip>` or application test | Non-filtered IPv4 traffic works if endpoint permits it |
| Confirm gateway reachability if not filtered | H1 | `ping <default-gateway-ip>` | Gateway ping works unless ICMP is intentionally filtered |
| Confirm logs for drop-log sequence | SW1 | `show logging | include VACL|ACCESS|Deny|drop` | Supported drop logs appear for `action drop log` traffic |
| Confirm no err-disabled ports | SW1 | `show interfaces status err-disabled` | No required ports are err-disabled |
| Confirm no unrelated ACL confusion | SW1 | `show running-config | include ip access-group|mac access-group|vlan filter` | PACL, RACL, and VACL attachments are clearly identified |
| Confirm saved configuration | SW1 | `show startup-config | include vlan access-map|vlan filter|VACL_MATCH` | Startup config contains intended VACL settings |

# VACL_VLAN_Access_Map_Filtering_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record current VACL state before rollback | SW1 | `show vlan access-map` | Current VACL sequences are documented |
| 2 | Record current VLAN filter attachment | SW1 | `show running-config | include vlan filter` | Applied VLAN filter state is documented |
| 3 | Record current ACL classifiers | SW1 | `show access-lists` | Match ACLs are documented |
| 4 | Enter global configuration mode | SW1 | `configure terminal` | Prompt changes to global configuration mode |
| 5 | Remove the VACL from VLAN 20 first | SW1 | `no vlan filter VACL_VLAN20_FILTER vlan-list 20` | VLAN 20 is no longer filtered by this VACL |
| 6 | Remove VLAN access-map sequence 10 | SW1 | `no vlan access-map VACL_VLAN20_FILTER 10` | ICMP drop sequence is removed |
| 7 | Remove VLAN access-map sequence 20 | SW1 | `no vlan access-map VACL_VLAN20_FILTER 20` | Telnet drop sequence is removed |
| 8 | Remove VLAN access-map sequence 30 | SW1 | `no vlan access-map VACL_VLAN20_FILTER 30` | Forward sequence is removed |
| 9 | Remove ICMP classifier ACL | SW1 | `no ip access-list extended VACL_MATCH_ICMP` | ICMP classifier ACL is removed |
| 10 | Remove Telnet classifier ACL | SW1 | `no ip access-list extended VACL_MATCH_TELNET` | Telnet classifier ACL is removed |
| 11 | Remove catch-all IPv4 classifier ACL | SW1 | `no ip access-list extended VACL_MATCH_OTHER_IP` | Catch-all classifier ACL is removed |
| 12 | Remove MAC classifier ACL if configured | SW1 | `no mac access-list extended VACL_MATCH_MAC` | MAC classifier ACL is removed |
| 13 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 14 | Verify VLAN filter is removed | SW1 | `show running-config | include vlan filter` | VACL is no longer applied to VLAN 20 |
| 15 | Verify VLAN access-map is removed | SW1 | `show vlan access-map` | Removed access-map sequences no longer appear |
| 16 | Verify classifier ACLs are removed | SW1 | `show access-lists` | Removed VACL classifier ACLs no longer appear |
| 17 | Verify ICMP works again if rollback should permit it | H1 | `ping <h2-vlan-20-ip>` | Ping succeeds if no other control blocks it |
| 18 | Verify Telnet behavior after rollback if endpoint supports it | H1 | `telnet <h2-vlan-20-ip> 23` | Telnet behavior returns to endpoint or service baseline |
| 19 | Save rollback | SW1 | `write memory` | Startup config reflects rollback |

# VACL_VLAN_Access_Map_Filtering_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| All traffic in VLAN fails after applying VACL | No final forward sequence exists | SW1 | `show vlan access-map` | Add a final `action forward` sequence or a catch-all forward match |
| ICMP still works | ICMP ACL does not match the traffic or VACL not applied to VLAN | SW1 | `show access-lists VACL_MATCH_ICMP` and `show running-config | include vlan filter` | Correct ACL match and apply `vlan filter` to the right VLAN |
| Telnet still works | Wrong TCP direction or destination port match | SW1 | `show access-lists VACL_MATCH_TELNET` | Match `tcp any any eq 23` for destination Telnet traffic |
| VACL exists but has no effect | VLAN filter is not applied | SW1 | `show running-config | include vlan filter` | Apply `vlan filter <map-name> vlan-list <vlan-id>` |
| VACL is applied to wrong VLAN | VLAN list is wrong | SW1 | `show running-config | include vlan filter` | Attach the filter to the correct VLAN list |
| Host traffic fails before VACL is applied | Baseline VLAN, port, trunk, or STP issue | SW1 | `show vlan brief`, `show interfaces trunk`, `show spanning-tree vlan 20` | Fix Layer 2 baseline before troubleshooting VACL |
| ACL counters do not increment | Platform does not count VACL classifier hits as expected or traffic does not match | SW1 | `show access-lists` | Use traffic tests and VACL behavior, not counters alone |
| Drop logging does not appear | Platform does not support VACL drop logging the way expected or logging level is insufficient | SW1 | `show logging | include VACL|ACCESS|drop` | Verify platform support and logging configuration |
| Non-IP traffic is blocked unexpectedly | Only IPv4 catch-all forward exists | SW1 | `show vlan access-map` | Add a no-match `action forward` sequence or MAC-based forward logic if required |
| ARP fails unexpectedly | VACL sequence does not forward non-IP traffic | SW1/HOSTS | `show arp`, `ip neigh`, `show vlan access-map` | Add final `action forward` sequence without a match or explicitly permit required non-IP traffic |
| DHCP fails after VACL | DHCP traffic is matched by a drop or not forwarded | SW1 | `show access-lists` and `show vlan access-map` | Permit or forward DHCP client and server traffic |
| Gateway access fails | ICMP or other traffic to gateway is intentionally or accidentally dropped | SW1/HOSTS | `show vlan access-map` | Exclude gateway traffic from drop ACL or add earlier forward sequence |
| Same-VLAN filtering works but routed traffic still passes | Routed traffic path does not traverse the filtered VLAN as expected | SW1/L3 device | `show ip route`, `show running-config | include vlan filter` | Apply VACL to the correct VLAN or use RACL where Layer 3 filtering is needed |
| Routed ACL and VACL behavior is confusing | PACL, VACL, and RACL are all present | SW1 | `show running-config | include ip access-group|mac access-group|vlan filter` | Identify each policy attachment and test one control at a time |
| VACL blocks traffic in both directions | Match ACL catches both source and destination directions | SW1 | `show access-lists <acl-name>` | Use more specific source and destination ACEs if one-way behavior is intended |
| One host pair is blocked but another is not | ACL match is host-specific or subnet-specific | SW1 | `show access-lists` | Correct source and destination match logic |
| Match ACL uses deny lines incorrectly | ACL is being treated like an enforcement ACL instead of classifier | SW1 | `show access-lists` | Use `permit` lines for traffic the VACL sequence should match |
| VACL sequence order is wrong | Broad forward sequence appears before drop sequence | SW1 | `show vlan access-map` | Put specific drop sequences before broad forward sequences |
| VACL sequence missing action | Sequence is incomplete | SW1 | `show vlan access-map` | Add `action forward` or `action drop` to every sequence |
| VACL sequence missing match | Sequence may match all traffic depending on use | SW1 | `show vlan access-map` | Use intentionally as catch-all or add an explicit match ACL |
| VLAN 20 missing from switch | VLAN database issue | SW1 | `show vlan brief` | Create VLAN 20 and assign ports |
| Port is in wrong VLAN | Access-port assignment issue | SW1 | `show interfaces <interface> switchport` | Configure `switchport access vlan 20` |
| Trunk does not carry VLAN 20 | VLAN missing from allowed trunk list | SW1 | `show interfaces trunk` | Add VLAN 20 to trunk allowed list |
| STP blocks the forwarding path | STP issue, not VACL issue | SW1 | `show spanning-tree vlan 20` | Fix root placement, trunk state, or STP protection condition |
| MAC addresses do not appear | No traffic, wrong VLAN, or port down | SW1 | `show mac address-table dynamic vlan 20` | Generate traffic and verify physical and VLAN state |
| Port is err-disabled | BPDU Guard, port security, or link-flap protection | SW1 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the port |
| VACL disappears after reload | Configuration was not saved | SW1 | `show startup-config | include vlan access-map|vlan filter|VACL_MATCH` | Reapply VACL and run `write memory` |
