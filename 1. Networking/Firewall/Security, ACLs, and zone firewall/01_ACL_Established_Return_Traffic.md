

ACL_Established_Return_Traffic.md
# ACL_Established_Return_Traffic
# ACL_Established_Return_Traffic_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Extended ACL | Filters using protocol, source, destination, and optional Layer 4 fields |
| `established` keyword | Matches TCP packets with ACK or RST set, usually return traffic from a TCP session that was initiated from the trusted side |
| Not stateful inspection | The router is not tracking sessions like CBAC or ZBFW; it is only checking TCP flags |
| TCP only | `established` does not work for UDP, ICMP, GRE, ESP, or other non-TCP traffic |
| Direction matters | Usually applied inbound on the untrusted/WAN interface to allow TCP return traffic back toward the inside subnet |
| Explicit deny | Unsolicited TCP toward the protected subnet should be denied and logged after the `established` permit |
| Explicit non-TCP handling | UDP, ICMP, and other protocols need their own ACEs if they must be allowed |
| Lab purpose | Prove that inside-initiated TCP works, outside-initiated TCP fails, and ACL counters confirm the decision |
# ACL_Established_Return_Traffic_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm interface addressing and direction | R1 | `show ip interface brief` | Inside and outside interfaces are up/up with expected IP addresses |
| 2 | Confirm baseline reachability before filtering | R1 | `ping <inside_host_ip>`<br>`ping <outside_host_ip>` | Router can reach both sides before ACL enforcement |
| 3 | Confirm current ACL state | R1 | `show access-lists`<br>`show running-config interface <outside_interface>` | No conflicting ACL is already applied in the target direction |
| 4 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 5 | Create named extended ACL | R1 | `ip access-list extended ACL_WAN_IN_ESTABLISHED` | Router enters extended named ACL configuration mode |
| 6 | Permit TCP return traffic toward inside subnet | R1 | `permit tcp any <inside_subnet> <inside_wildcard> established` | TCP packets with ACK or RST set are permitted back toward the inside subnet |
| 7 | Deny unsolicited TCP toward inside subnet | R1 | `deny tcp any <inside_subnet> <inside_wildcard> log` | New inbound TCP attempts from the untrusted side are denied and logged |
| 8 | Permit required ICMP return or test traffic if needed | R1 | `permit icmp any <inside_subnet> <inside_wildcard> echo-reply`<br>`permit icmp any <inside_subnet> <inside_wildcard> time-exceeded`<br>`permit icmp any <inside_subnet> <inside_wildcard> unreachable` | Common diagnostic ICMP responses can return if the lab requires ping or traceroute testing |
| 9 | Permit required UDP return traffic if needed | R1 | `permit udp host <dns_server_ip> eq 53 <inside_subnet> <inside_wildcard>` | DNS replies are allowed if inside hosts use an external DNS server |
| 10 | Deny all other inbound traffic toward protected inside subnet | R1 | `deny ip any <inside_subnet> <inside_wildcard> log` | Non-approved traffic from outside toward the inside subnet is dropped and logged |
| 11 | Permit unrelated transit traffic if the interface carries more than protected-inside traffic | R1 | `permit ip any any` | ACL does not accidentally blackhole unrelated traffic on the outside interface |
| 12 | Exit ACL configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 13 | Apply ACL inbound on the outside interface | R1 | `interface <outside_interface>`<br>`ip access-group ACL_WAN_IN_ESTABLISHED in` | ACL is enforced as traffic enters from the untrusted side |
| 14 | Save the configuration | R1 | `end`<br>`write memory` | Configuration is saved |
| 15 | Test inside-initiated TCP session | Inside Host | `telnet <outside_server_ip> 80`<br>or<br>`curl http://<outside_server_ip>` | TCP session initiated from inside succeeds, return traffic is permitted by `established` |
| 16 | Test outside-initiated TCP session toward inside | Outside Host | `telnet <inside_host_ip> 80`<br>or<br>`nc -vz <inside_host_ip> 80` | New TCP session from outside toward inside fails |
| 17 | Verify ACL counters | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | Permit counter increments for return TCP; deny counters increment for unsolicited inbound traffic |
| 18 | Verify interface ACL attachment | R1 | `show ip interface <outside_interface>` | Output shows `Inbound access list is ACL_WAN_IN_ESTABLISHED` |
# ACL_Established_Return_Traffic_Skeleton
R1
configure terminal
ip access-list extended ACL_WAN_IN_ESTABLISHED
 permit tcp any <inside_subnet> <inside_wildcard> established
 deny tcp any <inside_subnet> <inside_wildcard> log
 permit icmp any <inside_subnet> <inside_wildcard> echo-reply
 permit icmp any <inside_subnet> <inside_wildcard> time-exceeded
 permit icmp any <inside_subnet> <inside_wildcard> unreachable
 permit udp host <dns_server_ip> eq 53 <inside_subnet> <inside_wildcard>
 deny ip any <inside_subnet> <inside_wildcard> log
 permit ip any any
exit
interface <outside_interface>
 ip access-group ACL_WAN_IN_ESTABLISHED in
exit
end
write memory
# ACL_Established_Return_Traffic_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Interface state | R1 | `show ip interface brief` | Inside and outside interfaces are up/up |
| ACL definition | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | ACL contains TCP `established` permit before the deny statements |
| ACL attachment | R1 | `show ip interface <outside_interface>` | Inbound ACL is `ACL_WAN_IN_ESTABLISHED` |
| Running config ACL | R1 | `show running-config \| section ip access-list extended ACL_WAN_IN_ESTABLISHED` | ACL entries appear in intended order |
| Running config interface | R1 | `show running-config interface <outside_interface>` | `ip access-group ACL_WAN_IN_ESTABLISHED in` is present |
| Inside-initiated TCP test | Inside Host | `telnet <outside_server_ip> 80` | Session succeeds if outside service is listening |
| Outside-initiated TCP test | Outside Host | `telnet <inside_host_ip> 80` | Session fails unless separately permitted |
| Counter validation | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | `permit tcp ... established` increments for return TCP; deny ACE increments for outside-initiated traffic |
| Logging validation | R1 | `show logging \| include ACL_WAN_IN_ESTABLISHED\|list` | Denied unsolicited traffic appears if logging is enabled |
# ACL_Established_Return_Traffic_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove ACL from outside interface | R1 | `configure terminal`<br>`interface <outside_interface>`<br>`no ip access-group ACL_WAN_IN_ESTABLISHED in` | Interface no longer enforces the ACL inbound |
| 2 | Confirm ACL is detached | R1 | `do show ip interface <outside_interface>` | Output does not show `Inbound access list is ACL_WAN_IN_ESTABLISHED` |
| 3 | Remove ACL object if no longer needed | R1 | `exit`<br>`no ip access-list extended ACL_WAN_IN_ESTABLISHED` | ACL is deleted from running configuration |
| 4 | Save rollback | R1 | `end`<br>`write memory` | Rollback is saved |
| 5 | Confirm ACL removal | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | ACL is absent or returns no entries |
# ACL_Established_Return_Traffic_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Inside-initiated TCP fails | ACL applied in wrong direction or wrong interface | R1 | `show ip interface <outside_interface>` | Apply ACL inbound on the actual untrusted-facing interface |
| Inside-initiated TCP fails | Return traffic does not match protected inside subnet | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | Correct `<inside_subnet>` and `<inside_wildcard>` |
| Outside-initiated TCP succeeds unexpectedly | `permit ip any any` placed before deny statements | R1 | `show running-config \| section ACL_WAN_IN_ESTABLISHED` | Move explicit deny entries before any broad permit |
| Outside-initiated TCP succeeds unexpectedly | ACL not attached | R1 | `show ip interface <outside_interface>` | Apply `ip access-group ACL_WAN_IN_ESTABLISHED in` |
| Ping fails even though TCP works | `established` only matches TCP, not ICMP | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | Add explicit ICMP permits if ping/traceroute are required |
| DNS fails from inside hosts | UDP DNS replies are not covered by `established` | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | Add explicit UDP permit for DNS replies |
| ACL counters do not increment | Test traffic is not crossing the filtered interface | R1 | `show ip route <destination_ip>`<br>`traceroute <destination_ip>` | Correct topology, routing, or ACL placement |
| ACL drops too much traffic | Missing protocol-specific permits before final deny | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED` | Add required ACEs above the final deny |
| Logs are missing | Logging disabled or ACE does not include `log` | R1 | `show logging`<br>`show running-config \| include logging` | Add `log` to deny ACEs and enable logging as needed |
| Return TCP still blocked | Return packet lacks ACK/RST or asymmetric routing bypasses expected path | R1 | `show access-lists ACL_WAN_IN_ESTABLISHED`<br>`show ip route` | Fix path symmetry or use stateful inspection instead of ACL `established` |
# ACL_Established_Return_Traffic_Related_Labs
- access-list-established-final
# ACL_Established_Return_Traffic_Index
ACL_Established_Return_Traffic.md
# ACL_Established_Return_Traffic
# ACL_Established_Return_Traffic_Mental_Model
# ACL_Established_Return_Traffic_Configuration_Checklist
# ACL_Established_Return_Traffic_Skeleton
# ACL_Established_Return_Traffic_Verification_Commands
# ACL_Established_Return_Traffic_Rollback
# ACL_Established_Return_Traffic_Failure_Checks
# ACL_Established_Return_Traffic_Related_Labs
# ACL_Established_Return_Traffic_Index