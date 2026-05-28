
VACL_IP_Filter_VLAN_Local_Filtering.md
# VACL_IP_Filter_VLAN_Local_Filtering
# VACL_IP_Filter_VLAN_Local_Filtering_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VACL | VLAN access control list used to filter traffic inside a VLAN or traffic routed into or out of a VLAN |
| VLAN-scoped filtering | The filter is applied to a VLAN with `vlan filter`, not directly inbound or outbound on a physical interface |
| VLAN access map | Route-map-like structure that uses sequence numbers, match statements, and actions |
| Match ACL | The ACL is used only to identify traffic; the VACL action decides whether to forward or drop it |
| ACL `permit` inside match ACL | Means "this traffic matches the VACL sequence" |
| ACL `deny` inside match ACL | Means "this traffic does not match this VACL sequence" |
| `action drop` | Drops traffic matched by that VACL sequence |
| `action drop log` | Drops matched traffic and logs the drop when supported |
| `action forward` | Allows matched traffic to continue forwarding |
| Catch-all forward | Usually required with `permit ip any any` plus `action forward` so unmatched traffic is not accidentally dropped |
| Sequence order | Lower sequence numbers are evaluated before higher sequence numbers |
| Implicit deny | Traffic that does not match a forwarding sequence can be dropped by the implicit end behavior |
| Lab purpose | Prove selected IP traffic in a VLAN is blocked while other VLAN traffic still forwards |
# VACL_IP_Filter_VLAN_Local_Filtering_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the VLAN that needs local filtering | SW1 | `show vlan brief` | Target VLAN exists and has expected ports |
| 2 | Confirm access and trunk port placement | SW1 | `show interfaces status`<br>`show interfaces trunk` | Hosts and inter-switch links carry the target VLAN |
| 3 | Confirm baseline traffic works before filtering | Host | `ping <target_host_ip>`<br>`telnet <target_host_ip> <port>` | Traffic works before VACL enforcement |
| 4 | Check for existing routed ACLs, PACLs, or VACLs | SW1 | `show access-lists`<br>`show vlan access-map`<br>`show vlan filter` | Existing filters are known before adding a new VACL |
| 5 | Enter global configuration mode | SW1 | `configure terminal` | CLI enters global configuration mode |
| 6 | Create match ACL for traffic to drop | SW1 | `ip access-list extended VACL_DROP_TRAFFIC` | CLI enters extended ACL configuration mode |
| 7 | Match the specific IP traffic that should be dropped | SW1 | `permit <protocol> <source> <source_wildcard> <destination> <destination_wildcard> <operator> <port>` | ACL permits only the traffic that should match the VACL drop sequence |
| 8 | Exit the drop match ACL | SW1 | `exit` | CLI returns to global configuration mode |
| 9 | Create catch-all match ACL for allowed traffic | SW1 | `ip access-list extended VACL_ALLOW_OTHER` | CLI enters extended ACL configuration mode |
| 10 | Match all remaining IP traffic | SW1 | `permit ip any any` | ACL matches all other IP traffic for forwarding |
| 11 | Exit the allow match ACL | SW1 | `exit` | CLI returns to global configuration mode |
| 12 | Create first VLAN access-map sequence | SW1 | `vlan access-map VACL_IP_FILTER 10` | CLI enters VLAN access-map configuration mode |
| 13 | Match traffic that should be dropped | SW1 | `match ip address VACL_DROP_TRAFFIC` | Sequence 10 matches the drop ACL |
| 14 | Drop the matched traffic | SW1 | `action drop log` | Matched traffic is dropped and logged when supported |
| 15 | Exit sequence 10 | SW1 | `exit` | CLI returns to global configuration mode |
| 16 | Create forwarding VLAN access-map sequence | SW1 | `vlan access-map VACL_IP_FILTER 20` | CLI enters VLAN access-map sequence 20 |
| 17 | Match all other IP traffic | SW1 | `match ip address VACL_ALLOW_OTHER` | Sequence 20 matches remaining IP traffic |
| 18 | Forward all other matched traffic | SW1 | `action forward` | Non-blocked IP traffic is allowed |
| 19 | Exit sequence 20 | SW1 | `exit` | CLI returns to global configuration mode |
| 20 | Apply the VACL to the target VLAN | SW1 | `vlan filter VACL_IP_FILTER vlan-list <vlan_id>` | VACL is active on the target VLAN |
| 21 | Exit configuration mode | SW1 | `end` | CLI returns to privileged EXEC mode |
| 22 | Verify VACL definition | SW1 | `show vlan access-map` | VACL sequences, match ACLs, and actions are displayed |
| 23 | Verify VACL application | SW1 | `show vlan filter` | VACL is applied to the correct VLAN |
| 24 | Verify ACL counters | SW1 | `show access-lists VACL_DROP_TRAFFIC`<br>`show access-lists VACL_ALLOW_OTHER` | Match counters increment during testing |
| 25 | Test blocked traffic | Host | `ping <blocked_target_ip>`<br>`telnet <blocked_target_ip> <blocked_port>` | Traffic matching drop ACL fails |
| 26 | Test allowed traffic | Host | `ping <allowed_target_ip>`<br>`telnet <allowed_target_ip> <allowed_port>` | Traffic matching catch-all forward path succeeds |
| 27 | Save configuration | SW1 | `write memory` | VACL configuration is saved |
# VACL_IP_Filter_VLAN_Local_Filtering_ICMP_Drop_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create ACL matching ICMP to drop | SW1 | `ip access-list extended VACL_DROP_ICMP`<br>`permit icmp any any`<br>`exit` | ICMP traffic is selected for VACL drop action |
| 2 | Create ACL matching all other IP traffic | SW1 | `ip access-list extended VACL_ALLOW_OTHER`<br>`permit ip any any`<br>`exit` | Remaining IP traffic is selected for forwarding |
| 3 | Create drop sequence | SW1 | `vlan access-map VACL_IP_FILTER 10`<br>`match ip address VACL_DROP_ICMP`<br>`action drop log`<br>`exit` | ICMP is dropped in sequence 10 |
| 4 | Create forward sequence | SW1 | `vlan access-map VACL_IP_FILTER 20`<br>`match ip address VACL_ALLOW_OTHER`<br>`action forward`<br>`exit` | Other IP traffic is forwarded |
| 5 | Apply VACL to VLAN | SW1 | `vlan filter VACL_IP_FILTER vlan-list <vlan_id>` | ICMP filtering is active on the VLAN |
| 6 | Test ICMP drop | Host | `ping <same_vlan_host_ip>` | Ping fails if it crosses the filtered VLAN |
| 7 | Test other traffic | Host | `telnet <same_vlan_host_ip> <allowed_port>` | Non-ICMP traffic succeeds if service and routing are valid |
# VACL_IP_Filter_VLAN_Local_Filtering_Skeleton
configure terminal
ip access-list extended VACL_DROP_TRAFFIC
 permit <protocol> <source> <source_wildcard> <destination> <destination_wildcard> <operator> <port>
exit
ip access-list extended VACL_ALLOW_OTHER
 permit ip any any
exit
vlan access-map VACL_IP_FILTER 10
 match ip address VACL_DROP_TRAFFIC
 action drop log
exit
vlan access-map VACL_IP_FILTER 20
 match ip address VACL_ALLOW_OTHER
 action forward
exit
vlan filter VACL_IP_FILTER vlan-list <vlan_id>
end
write memory
# VACL_IP_Filter_VLAN_Local_Filtering_ICMP_Drop_Skeleton
configure terminal
ip access-list extended VACL_DROP_ICMP
 permit icmp any any
exit
ip access-list extended VACL_ALLOW_OTHER
 permit ip any any
exit
vlan access-map VACL_IP_FILTER 10
 match ip address VACL_DROP_ICMP
 action drop log
exit
vlan access-map VACL_IP_FILTER 20
 match ip address VACL_ALLOW_OTHER
 action forward
exit
vlan filter VACL_IP_FILTER vlan-list <vlan_id>
end
write memory
# VACL_IP_Filter_VLAN_Local_Filtering_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| VLAN exists | SW1 | `show vlan brief` | Target VLAN exists |
| VLAN forwarding path | SW1 | `show interfaces trunk` | Target VLAN is carried on required trunks |
| Access port placement | SW1 | `show interfaces status` | Host ports are in the expected VLAN |
| VACL definition | SW1 | `show vlan access-map` | VACL sequences show correct match ACLs and actions |
| VACL application | SW1 | `show vlan filter` | VACL is applied to the intended VLAN |
| Match ACLs | SW1 | `show access-lists` | Drop and forward ACLs exist |
| Drop ACL counters | SW1 | `show access-lists VACL_DROP_TRAFFIC` | Counter increments for blocked traffic |
| Forward ACL counters | SW1 | `show access-lists VACL_ALLOW_OTHER` | Counter increments for allowed traffic |
| Running config VACL | SW1 | `show running-config \| section vlan access-map` | VLAN access-map sequences are present |
| Running config VLAN filter | SW1 | `show running-config \| include vlan filter` | VACL is attached with `vlan filter` |
| Blocked traffic test | Host | `ping <blocked_target_ip>`<br>`telnet <blocked_target_ip> <blocked_port>` | Traffic matching drop ACL fails |
| Allowed traffic test | Host | `ping <allowed_target_ip>`<br>`telnet <allowed_target_ip> <allowed_port>` | Traffic matching forward sequence succeeds |
| Log validation | SW1 | `show logging` | Drop log entries appear if `action drop log` is supported and triggered |
# VACL_IP_Filter_VLAN_Local_Filtering_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | SW1 | `configure terminal` | CLI enters global configuration mode |
| 2 | Remove VACL from target VLAN | SW1 | `no vlan filter VACL_IP_FILTER vlan-list <vlan_id>` | VLAN no longer uses the VACL |
| 3 | Remove VLAN access-map sequence 10 | SW1 | `no vlan access-map VACL_IP_FILTER 10` | Drop sequence is removed |
| 4 | Remove VLAN access-map sequence 20 | SW1 | `no vlan access-map VACL_IP_FILTER 20` | Forward sequence is removed |
| 5 | Remove drop match ACL if not reused | SW1 | `no ip access-list extended VACL_DROP_TRAFFIC` | Drop ACL is removed |
| 6 | Remove catch-all forward ACL if not reused | SW1 | `no ip access-list extended VACL_ALLOW_OTHER` | Forward ACL is removed |
| 7 | Exit configuration mode | SW1 | `end` | CLI returns to privileged EXEC mode |
| 8 | Confirm VACL is detached | SW1 | `show vlan filter` | VACL no longer appears on the target VLAN |
| 9 | Confirm access-map removal | SW1 | `show vlan access-map` | Removed VACL sequences are absent |
| 10 | Confirm ACL removal | SW1 | `show access-lists` | Removed match ACLs are absent |
| 11 | Save rollback | SW1 | `write memory` | Rollback is saved |
# VACL_IP_Filter_VLAN_Local_Filtering_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Blocked traffic still passes | VACL is not applied to the VLAN | SW1 | `show vlan filter` | Apply with `vlan filter VACL_IP_FILTER vlan-list <vlan_id>` |
| Blocked traffic still passes | Traffic does not match the drop ACL | SW1 | `show access-lists VACL_DROP_TRAFFIC` | Correct protocol, source, destination, wildcard, or port |
| Blocked traffic still passes | Test traffic is not in the filtered VLAN | SW1 | `show mac address-table`<br>`show vlan brief` | Test through the VLAN where the VACL is applied |
| All traffic is dropped | Missing catch-all forward sequence | SW1 | `show vlan access-map` | Add `permit ip any any` match ACL and `action forward` sequence |
| All traffic is dropped | Forward sequence exists but does not match traffic | SW1 | `show access-lists VACL_ALLOW_OTHER` | Use `permit ip any any` for the catch-all forward ACL |
| ACL looks correct but VACL behavior is wrong | ACL permit is being misunderstood as allow | SW1 | `show vlan access-map`<br>`show access-lists` | Remember ACL permit only matches traffic; VACL action decides drop or forward |
| Ping fails but TCP works | ICMP is matched by a drop sequence | SW1 | `show access-lists VACL_DROP_TRAFFIC` | Remove ICMP from drop ACL or add more specific matching |
| TCP service fails but ping works | TCP port is matched by a drop sequence | SW1 | `show access-lists VACL_DROP_TRAFFIC` | Correct the TCP port match |
| VACL counters do not move | Test traffic is locally generated by the switch or not crossing the VLAN datapath | SW1 | `show mac address-table`<br>`show interfaces trunk` | Generate traffic between hosts through the filtered VLAN |
| Inter-VLAN traffic unexpectedly affected | VACL also filters traffic routed into or out of the VLAN | SW1 | `show vlan filter`<br>`show ip route` | Narrow ACL match or apply VACL only to intended VLANs |
| Logging does not appear | Platform does not support `action drop log` or logging is not enabled | SW1 | `show logging`<br>`show vlan access-map` | Use counters for validation or enable logging as supported |
| Command is rejected | Platform or image does not support VACL syntax | SW1 | `show version` | Use a supported Catalyst or IOS XE switching image |
| Trunk-side traffic does not behave as expected | VLAN is not allowed or active on trunk | SW1 | `show interfaces trunk` | Allow the VLAN on the trunk and confirm STP forwarding |
| VACL removed but traffic still blocked | PACL, RACL, port security, IP Source Guard, or another filter is still active | SW1 | `show access-lists`<br>`show running-config interface <interface>`<br>`show ip verify source`<br>`show port-security` | Remove or correct the remaining enforcement feature |
# VACL_IP_Filter_VLAN_Local_Filtering_Related_Labs
- vacl-ip-filter-final
# VACL_IP_Filter_VLAN_Local_Filtering_Index
VACL_IP_Filter_VLAN_Local_Filtering.md
# VACL_IP_Filter_VLAN_Local_Filtering
# VACL_IP_Filter_VLAN_Local_Filtering_Mental_Model
# VACL_IP_Filter_VLAN_Local_Filtering_Configuration_Checklist
# VACL_IP_Filter_VLAN_Local_Filtering_ICMP_Drop_Checklist
# VACL_IP_Filter_VLAN_Local_Filtering_Skeleton
# VACL_IP_Filter_VLAN_Local_Filtering_ICMP_Drop_Skeleton
# VACL_IP_Filter_VLAN_Local_Filtering_Verification_Commands
# VACL_IP_Filter_VLAN_Local_Filtering_Rollback
# VACL_IP_Filter_VLAN_Local_Filtering_Failure_Checks
# VACL_IP_Filter_VLAN_Local_Filtering_Related_Labs
# VACL_IP_Filter_VLAN_Local_Filtering_Index
