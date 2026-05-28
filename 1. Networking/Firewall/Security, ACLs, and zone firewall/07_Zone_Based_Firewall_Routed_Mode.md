Zone_Based_Firewall_Routed_Mode.md
# Zone_Based_Firewall_Routed_Mode
# Zone_Based_Firewall_Routed_Mode_Mental_Model
| Concept | Operational Meaning |
|---|---|
| ZBFW | IOS XE stateful firewall model based on zones, zone pairs, class maps, and policy maps |
| Routed mode | Firewall interfaces route IP traffic normally; ZBFW controls which routed flows are allowed between zones |
| Security zone | Logical trust boundary assigned to one or more router interfaces |
| Same-zone traffic | Interfaces in the same zone can communicate by default |
| Interzone traffic | Traffic between different zones is dropped unless a zone pair and policy explicitly allow it |
| Zone pair | Directional policy path from a source zone to a destination zone |
| Direction matters | `INSIDE` to `OUTSIDE` is different from `OUTSIDE` to `INSIDE` |
| Class map | Identifies traffic to be handled by firewall policy |
| Policy map | Applies actions to matched classes |
| `inspect` action | Stateful allow; return traffic is permitted dynamically |
| `pass` action | One-way stateless forwarding; return traffic needs a separate reverse policy |
| `drop` action | Drops matched traffic; `drop log` adds logging |
| Class-default | Implicit default drop exists; explicitly configuring `class class-default drop` improves readability |
| Self zone | Represents traffic destined to or sourced from the router itself |
| Self-zone caution | Once a self-zone policy is applied, management and control-plane traffic must be explicitly allowed |
| Interface zone membership | `zone-member security <zone>` activates zone enforcement on that interface |
| Lab purpose | Prove inside-initiated traffic is inspected and allowed, return traffic passes dynamically, and unsolicited outside-initiated traffic is dropped |
# Zone_Based_Firewall_Routed_Mode_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify inside, outside, and optional DMZ interfaces | R1 | `show ip interface brief` | Routed interfaces are up/up with expected IP addresses |
| 2 | Confirm routing before firewall enforcement | R1 | `show ip route`<br>`show ip cef` | Router has routes for inside, outside, and destination networks |
| 3 | Confirm baseline connectivity before ZBFW | Inside Host | `ping <outside_host_ip>`<br>`telnet <outside_host_ip> <port>` | Traffic works before enforcement, proving routing is not the problem |
| 4 | Check for existing ZBFW configuration | R1 | `show running-config \| section zone`<br>`show policy-map type inspect`<br>`show class-map type inspect` | Existing zones, class maps, policies, and zone pairs are known |
| 5 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 6 | Create inside security zone | R1 | `zone security INSIDE`<br>`description Trusted inside LAN zone` | Inside zone exists |
| 7 | Create outside security zone | R1 | `zone security OUTSIDE`<br>`description Untrusted outside WAN or Internet zone` | Outside zone exists |
| 8 | Create optional DMZ security zone if used | R1 | `zone security DMZ`<br>`description Server or semi-trusted DMZ zone` | DMZ zone exists if required |
| 9 | Create ACL matching inside-to-outside traffic | R1 | `ip access-list extended ACL_INSIDE_TO_OUTSIDE` | CLI enters extended ACL configuration mode |
| 10 | Match inside traffic that should be statefully inspected | R1 | `permit ip <inside_subnet> <inside_wildcard> any` | Inside-originated traffic is selected for inspection |
| 11 | Exit ACL configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 12 | Create inspect class map | R1 | `class-map type inspect match-any CM_INSIDE_TO_OUTSIDE` | CLI enters inspect class-map configuration mode |
| 13 | Match the inside-to-outside ACL | R1 | `match access-group name ACL_INSIDE_TO_OUTSIDE` | Class map matches the desired inside-originated traffic |
| 14 | Exit class-map configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 15 | Create inspect policy map | R1 | `policy-map type inspect PM_INSIDE_TO_OUTSIDE` | CLI enters inspect policy-map configuration mode |
| 16 | Add the inspect class to the policy | R1 | `class type inspect CM_INSIDE_TO_OUTSIDE` | CLI enters the class action section |
| 17 | Apply stateful inspection | R1 | `inspect` | Matched traffic is statefully allowed and return traffic is dynamically permitted |
| 18 | Add explicit default drop | R1 | `class class-default`<br>`drop log` | Unmatched traffic is dropped and logged |
| 19 | Exit policy-map configuration mode | R1 | `exit`<br>`exit` | CLI returns to global configuration mode |
| 20 | Create inside-to-outside zone pair | R1 | `zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE` | Directional zone pair is created |
| 21 | Attach policy to zone pair | R1 | `service-policy type inspect PM_INSIDE_TO_OUTSIDE` | Inside-to-outside policy is active on the zone pair |
| 22 | Exit zone-pair configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 23 | Assign inside interface to INSIDE zone | R1 | `interface <inside_interface>`<br>`zone-member security INSIDE` | Inside interface becomes part of the INSIDE zone |
| 24 | Assign outside interface to OUTSIDE zone | R1 | `interface <outside_interface>`<br>`zone-member security OUTSIDE` | Outside interface becomes part of the OUTSIDE zone |
| 25 | Assign DMZ interface if used | R1 | `interface <dmz_interface>`<br>`zone-member security DMZ` | DMZ interface becomes part of the DMZ zone |
| 26 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 27 | Verify zones exist | R1 | `show zone security` | INSIDE, OUTSIDE, and optional DMZ zones are present |
| 28 | Verify zone pairs | R1 | `show zone-pair security` | Zone pair shows correct source zone, destination zone, and policy |
| 29 | Verify class map | R1 | `show class-map type inspect CM_INSIDE_TO_OUTSIDE` | Class map references the correct ACL |
| 30 | Verify policy map | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | Policy shows inspect action and class-default drop |
| 31 | Verify policy counters | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Counters appear for inspected and dropped traffic |
| 32 | Test inside-to-outside traffic | Inside Host | `ping <outside_host_ip>`<br>`telnet <outside_host_ip> <port>`<br>`curl http://<outside_host_ip>` | Inside-originated traffic succeeds if protocol and service are valid |
| 33 | Verify stateful sessions | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Inspect counters and session counters increment |
| 34 | Test unsolicited outside-to-inside traffic | Outside Host | `telnet <inside_host_ip> <port>`<br>`ping <inside_host_ip>` | New outside-originated traffic fails unless a reverse zone pair allows it |
| 35 | Verify dropped traffic evidence | R1 | `show policy-map type inspect zone-pair`<br>`show logging` | Class-default drop counters or logs increment |
| 36 | Save configuration | R1 | `write memory` | ZBFW routed-mode configuration is saved |
# Zone_Based_Firewall_Routed_Mode_Outside_To_DMZ_Service_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the public service to allow | R1 | `show ip interface brief`<br>`show ip route <dmz_server_ip>` | DMZ server path is known |
| 2 | Create ACL matching outside-to-DMZ service traffic | R1 | `configure terminal`<br>`ip access-list extended ACL_OUTSIDE_TO_DMZ_WEB` | CLI enters ACL configuration mode |
| 3 | Match the allowed service only | R1 | `permit tcp any host <dmz_server_ip> eq 80`<br>`permit tcp any host <dmz_server_ip> eq 443` | HTTP and HTTPS to the DMZ server are selected |
| 4 | Exit ACL configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 5 | Create service class map | R1 | `class-map type inspect match-any CM_OUTSIDE_TO_DMZ_WEB` | CLI enters class-map mode |
| 6 | Match the service ACL | R1 | `match access-group name ACL_OUTSIDE_TO_DMZ_WEB` | Class map selects only allowed web service traffic |
| 7 | Exit class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 8 | Create outside-to-DMZ policy | R1 | `policy-map type inspect PM_OUTSIDE_TO_DMZ` | CLI enters policy-map mode |
| 9 | Inspect allowed outside-to-DMZ service traffic | R1 | `class type inspect CM_OUTSIDE_TO_DMZ_WEB`<br>`inspect` | Allowed web traffic is statefully inspected |
| 10 | Drop all other outside-to-DMZ traffic | R1 | `class class-default`<br>`drop log` | Unmatched outside-to-DMZ traffic is dropped |
| 11 | Create outside-to-DMZ zone pair | R1 | `zone-pair security ZP_OUTSIDE_TO_DMZ source OUTSIDE destination DMZ` | Directional zone pair is created |
| 12 | Attach outside-to-DMZ policy | R1 | `service-policy type inspect PM_OUTSIDE_TO_DMZ` | Policy is active between OUTSIDE and DMZ |
| 13 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 14 | Test allowed web service | Outside Host | `curl http://<dmz_server_ip>`<br>`curl https://<dmz_server_ip>` | Allowed service succeeds if server and routing are valid |
| 15 | Test denied service | Outside Host | `telnet <dmz_server_ip> 22` | Non-approved service fails |
| 16 | Verify counters | R1 | `show policy-map type inspect zone-pair ZP_OUTSIDE_TO_DMZ` | Web class counters increment for allowed traffic and class-default increments for denied traffic |
| 17 | Save configuration | R1 | `write memory` | Outside-to-DMZ service policy is saved |
# Zone_Based_Firewall_Routed_Mode_Self_Zone_Management_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify management and control-plane traffic before touching self zone | R1 | `show control-plane host open-ports`<br>`show ip protocols`<br>`show running-config \| include line vty\|snmp-server\|ntp server` | Required SSH, SNMP, NTP, routing, and ICMP traffic is known |
| 2 | Create ACL for trusted management sources | R1 | `configure terminal`<br>`ip access-list extended ACL_OUTSIDE_TO_SELF_MGMT` | CLI enters ACL configuration mode |
| 3 | Permit SSH from management subnet | R1 | `permit tcp <mgmt_subnet> <mgmt_wildcard> any eq 22` | SSH from trusted management hosts is selected |
| 4 | Permit ICMP if required | R1 | `permit icmp <mgmt_subnet> <mgmt_wildcard> any` | Management ICMP is selected if required |
| 5 | Permit routing protocol/control traffic if required | R1 | `permit <protocol> <source> <wildcard> any` | Required control-plane traffic is selected |
| 6 | Exit ACL configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 7 | Create self-zone class map | R1 | `class-map type inspect match-any CM_OUTSIDE_TO_SELF_MGMT` | CLI enters inspect class-map mode |
| 8 | Match self-zone management ACL | R1 | `match access-group name ACL_OUTSIDE_TO_SELF_MGMT` | Class map selects allowed management/control traffic |
| 9 | Exit class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 10 | Create self-zone policy map | R1 | `policy-map type inspect PM_OUTSIDE_TO_SELF` | CLI enters policy-map mode |
| 11 | Pass or inspect allowed self-zone traffic | R1 | `class type inspect CM_OUTSIDE_TO_SELF_MGMT`<br>`pass` | Allowed traffic to router itself is forwarded to the control plane |
| 12 | Drop all other traffic to self | R1 | `class class-default`<br>`drop log` | Unmatched traffic to router itself is dropped |
| 13 | Create outside-to-self zone pair | R1 | `zone-pair security ZP_OUTSIDE_TO_SELF source OUTSIDE destination self` | Directional zone pair to router self zone is created |
| 14 | Attach self-zone policy | R1 | `service-policy type inspect PM_OUTSIDE_TO_SELF` | Management/control policy is active |
| 15 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 16 | Test allowed management | Management Host | `ssh <router_outside_ip>` | SSH succeeds only from approved management source |
| 17 | Test denied management | Untrusted Host | `ssh <router_outside_ip>` | SSH fails from unapproved source |
| 18 | Verify self-zone counters | R1 | `show policy-map type inspect zone-pair ZP_OUTSIDE_TO_SELF` | Allowed and dropped counters increment correctly |
| 19 | Save configuration | R1 | `write memory` | Self-zone policy is saved |
# Zone_Based_Firewall_Routed_Mode_Skeleton
configure terminal
zone security INSIDE
 description Trusted inside LAN zone
exit
zone security OUTSIDE
 description Untrusted outside WAN or Internet zone
exit
ip access-list extended ACL_INSIDE_TO_OUTSIDE
 permit ip <inside_subnet> <inside_wildcard> any
exit
class-map type inspect match-any CM_INSIDE_TO_OUTSIDE
 match access-group name ACL_INSIDE_TO_OUTSIDE
exit
policy-map type inspect PM_INSIDE_TO_OUTSIDE
 class type inspect CM_INSIDE_TO_OUTSIDE
  inspect
 class class-default
  drop log
exit
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM_INSIDE_TO_OUTSIDE
exit
interface <inside_interface>
 zone-member security INSIDE
 no shutdown
exit
interface <outside_interface>
 zone-member security OUTSIDE
 no shutdown
exit
end
write memory
# Zone_Based_Firewall_Routed_Mode_Inside_Outside_With_DMZ_Skeleton
configure terminal
zone security INSIDE
 description Trusted inside LAN zone
exit
zone security OUTSIDE
 description Untrusted outside WAN or Internet zone
exit
zone security DMZ
 description Semi-trusted server zone
exit
ip access-list extended ACL_INSIDE_TO_OUTSIDE
 permit ip <inside_subnet> <inside_wildcard> any
exit
ip access-list extended ACL_INSIDE_TO_DMZ
 permit ip <inside_subnet> <inside_wildcard> <dmz_subnet> <dmz_wildcard>
exit
ip access-list extended ACL_OUTSIDE_TO_DMZ_WEB
 permit tcp any host <dmz_server_ip> eq 80
 permit tcp any host <dmz_server_ip> eq 443
exit
class-map type inspect match-any CM_INSIDE_TO_OUTSIDE
 match access-group name ACL_INSIDE_TO_OUTSIDE
exit
class-map type inspect match-any CM_INSIDE_TO_DMZ
 match access-group name ACL_INSIDE_TO_DMZ
exit
class-map type inspect match-any CM_OUTSIDE_TO_DMZ_WEB
 match access-group name ACL_OUTSIDE_TO_DMZ_WEB
exit
policy-map type inspect PM_INSIDE_TO_OUTSIDE
 class type inspect CM_INSIDE_TO_OUTSIDE
  inspect
 class class-default
  drop log
exit
policy-map type inspect PM_INSIDE_TO_DMZ
 class type inspect CM_INSIDE_TO_DMZ
  inspect
 class class-default
  drop log
exit
policy-map type inspect PM_OUTSIDE_TO_DMZ
 class type inspect CM_OUTSIDE_TO_DMZ_WEB
  inspect
 class class-default
  drop log
exit
zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM_INSIDE_TO_OUTSIDE
exit
zone-pair security ZP_INSIDE_TO_DMZ source INSIDE destination DMZ
 service-policy type inspect PM_INSIDE_TO_DMZ
exit
zone-pair security ZP_OUTSIDE_TO_DMZ source OUTSIDE destination DMZ
 service-policy type inspect PM_OUTSIDE_TO_DMZ
exit
interface <inside_interface>
 zone-member security INSIDE
 no shutdown
exit
interface <outside_interface>
 zone-member security OUTSIDE
 no shutdown
exit
interface <dmz_interface>
 zone-member security DMZ
 no shutdown
exit
end
write memory
# Zone_Based_Firewall_Routed_Mode_Pass_Action_Skeleton
configure terminal
ip access-list extended ACL_PASS_ESP
 permit esp any any
exit
class-map type inspect match-any CM_PASS_ESP
 match access-group name ACL_PASS_ESP
exit
policy-map type inspect PM_PASS_ONLY
 class type inspect CM_PASS_ESP
  pass
 class class-default
  drop log
exit
zone-pair security ZP_SOURCE_TO_DEST source <source_zone> destination <destination_zone>
 service-policy type inspect PM_PASS_ONLY
exit
end
write memory
# Zone_Based_Firewall_Routed_Mode_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Interface state | R1 | `show ip interface brief` | Routed interfaces are up/up |
| Routing table | R1 | `show ip route` | Routes exist for inside, outside, and tested destinations |
| CEF table | R1 | `show ip cef` | CEF entries exist for forwarding paths |
| Security zones | R1 | `show zone security` | Expected zones exist |
| Zone members | R1 | `show zone security` | Interfaces appear under the intended zones |
| Zone pairs | R1 | `show zone-pair security` | Source and destination zones are correct |
| Class maps | R1 | `show class-map type inspect` | Class maps reference expected ACLs or protocols |
| Policy maps | R1 | `show policy-map type inspect` | Policies show inspect, pass, drop, or drop log actions |
| Zone-pair policy counters | R1 | `show policy-map type inspect zone-pair` | Counters increment for matched and dropped traffic |
| Specific zone-pair counters | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Inside-to-outside inspect counters increment |
| Match ACL counters | R1 | `show access-lists ACL_INSIDE_TO_OUTSIDE` | ACL counters increment for class-map matched traffic |
| Interface configuration | R1 | `show running-config interface <interface>` | Interface has correct `zone-member security <zone>` |
| Zone-pair running config | R1 | `show running-config \| section zone-pair` | Zone pairs and service policies are present |
| Inspect policy running config | R1 | `show running-config \| section policy-map type inspect` | Policy maps contain expected class actions |
| Inside-to-outside test | Inside Host | `ping <outside_host_ip>`<br>`telnet <outside_host_ip> <port>`<br>`curl http://<outside_host_ip>` | Inside-originated traffic succeeds if permitted and service is valid |
| Return traffic behavior | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Inspect session or packet counters increment for return traffic |
| Outside-to-inside denied test | Outside Host | `telnet <inside_host_ip> <port>` | New unsolicited outside-to-inside flow fails |
| Outside-to-DMZ allowed test | Outside Host | `curl http://<dmz_server_ip>` | Allowed DMZ service succeeds if configured |
| Dropped traffic logs | R1 | `show logging` | Logs appear for classes using `drop log` |
# Zone_Based_Firewall_Routed_Mode_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove zone membership from inside interface | R1 | `configure terminal`<br>`interface <inside_interface>`<br>`no zone-member security INSIDE` | Inside interface is removed from ZBFW zone enforcement |
| 2 | Remove zone membership from outside interface | R1 | `interface <outside_interface>`<br>`no zone-member security OUTSIDE` | Outside interface is removed from ZBFW zone enforcement |
| 3 | Remove zone membership from DMZ interface if used | R1 | `interface <dmz_interface>`<br>`no zone-member security DMZ` | DMZ interface is removed from ZBFW zone enforcement |
| 4 | Exit interface configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 5 | Remove inside-to-outside zone pair | R1 | `no zone-pair security ZP_INSIDE_TO_OUTSIDE` | Zone pair and attached service policy are removed |
| 6 | Remove inside-to-DMZ zone pair if used | R1 | `no zone-pair security ZP_INSIDE_TO_DMZ` | Inside-to-DMZ zone pair is removed |
| 7 | Remove outside-to-DMZ zone pair if used | R1 | `no zone-pair security ZP_OUTSIDE_TO_DMZ` | Outside-to-DMZ zone pair is removed |
| 8 | Remove self-zone pair if configured | R1 | `no zone-pair security ZP_OUTSIDE_TO_SELF` | Outside-to-self policy is removed |
| 9 | Remove inspect policy maps | R1 | `no policy-map type inspect PM_INSIDE_TO_OUTSIDE`<br>`no policy-map type inspect PM_INSIDE_TO_DMZ`<br>`no policy-map type inspect PM_OUTSIDE_TO_DMZ` | Inspect policy maps are deleted |
| 10 | Remove inspect class maps | R1 | `no class-map type inspect CM_INSIDE_TO_OUTSIDE`<br>`no class-map type inspect CM_INSIDE_TO_DMZ`<br>`no class-map type inspect CM_OUTSIDE_TO_DMZ_WEB` | Inspect class maps are deleted |
| 11 | Remove match ACLs if not reused | R1 | `no ip access-list extended ACL_INSIDE_TO_OUTSIDE`<br>`no ip access-list extended ACL_INSIDE_TO_DMZ`<br>`no ip access-list extended ACL_OUTSIDE_TO_DMZ_WEB` | ACL objects are deleted |
| 12 | Remove zones if no longer used | R1 | `no zone security INSIDE`<br>`no zone security OUTSIDE`<br>`no zone security DMZ` | Security zones are deleted |
| 13 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 14 | Confirm zone membership removal | R1 | `show zone security` | Interfaces no longer appear as zone members |
| 15 | Confirm zone-pair removal | R1 | `show zone-pair security` | Removed zone pairs are absent |
| 16 | Confirm policy cleanup | R1 | `show policy-map type inspect`<br>`show class-map type inspect` | Removed policy and class maps are absent |
| 17 | Save rollback | R1 | `write memory` | Rollback is saved |
# Zone_Based_Firewall_Routed_Mode_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| All interzone traffic fails | Interface was placed into a zone before a valid zone pair and policy existed | R1 | `show zone security`<br>`show zone-pair security` | Build class map, policy map, and zone pair before assigning production interfaces |
| Inside-to-outside traffic fails | Missing inside-to-outside zone pair | R1 | `show zone-pair security` | Create `zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE` |
| Inside-to-outside traffic fails | Policy not attached to zone pair | R1 | `show zone-pair security` | Add `service-policy type inspect PM_INSIDE_TO_OUTSIDE` |
| Inside-to-outside traffic fails | Class map does not match the traffic | R1 | `show class-map type inspect`<br>`show access-lists ACL_INSIDE_TO_OUTSIDE` | Correct ACL, class-map match, source subnet, destination, protocol, or port |
| Inside-to-outside traffic hits class-default | ACL wildcard or protocol is wrong | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE`<br>`show access-lists ACL_INSIDE_TO_OUTSIDE` | Correct the match ACL and retest |
| Return traffic fails after inside initiated flow | Policy uses `pass` instead of `inspect` | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | Use `inspect` for stateful flows or add reverse zone pair for `pass` traffic |
| Outside-to-inside traffic succeeds unexpectedly | Reverse zone pair permits it | R1 | `show zone-pair security`<br>`show policy-map type inspect zone-pair` | Remove or tighten OUTSIDE-to-INSIDE policy |
| Same-zone traffic is not filtered | Interfaces are in the same zone | R1 | `show zone security` | Put interfaces in different zones if firewall policy must apply between them |
| Traffic involving unzoned interface fails | One interface is in a zone and the other is not | R1 | `show zone security`<br>`show ip interface brief` | Assign both interfaces to zones and create a zone pair |
| Router management breaks after self-zone policy | Required SSH, SNMP, ICMP, or routing protocol was not allowed to self zone | R1 | `show policy-map type inspect zone-pair ZP_OUTSIDE_TO_SELF`<br>`show logging` | Add explicit self-zone permits or remove the self-zone policy |
| Router cannot ping outside after ZBFW | Locally generated traffic from self zone lacks self-to-outside policy | R1 | `show zone-pair security` | Create `zone-pair security ZP_SELF_TO_OUTSIDE source self destination OUTSIDE` with required traffic |
| DMZ service is unreachable from outside | Missing OUTSIDE-to-DMZ zone pair or wrong service match | R1 | `show zone-pair security`<br>`show access-lists ACL_OUTSIDE_TO_DMZ_WEB` | Create or correct OUTSIDE-to-DMZ policy |
| Counters do not increment | Traffic is not crossing the expected zone pair | R1 | `show ip route <destination>`<br>`traceroute <destination>`<br>`show zone security` | Fix routing, interface zone membership, or test path |
| ACL counters increment but policy counters do not | ACL is not referenced by the active class map or policy | R1 | `show class-map type inspect`<br>`show policy-map type inspect` | Reference the correct ACL in the class map and correct class in the policy |
| Policy counters increment but application fails | Server, DNS, NAT, or routing issue outside firewall policy | R1 | `show ip route`<br>`show ip nat translations`<br>`show access-lists` | Troubleshoot service path outside ZBFW |
| Logs are too noisy | `drop log` on class-default is logging high-volume background traffic | R1 | `show logging`<br>`show policy-map type inspect zone-pair` | Remove `log` from class-default or make more specific drop classes |
| Command is rejected | IOS image does not support ZBFW syntax | R1 | `show version` | Use IOS XE or an image with security/ZBFW support |
| ZBFW removed but traffic still fails | ACL, uRPF, NAT, route, host firewall, or CoPP is still affecting traffic | R1 | `show access-lists`<br>`show ip interface <interface>`<br>`show ip route`<br>`show policy-map control-plane` | Remove or correct the remaining feature |
# Zone_Based_Firewall_Routed_Mode_Related_Labs
- zone-based-firewall-final
# Zone_Based_Firewall_Routed_Mode_Index
Zone_Based_Firewall_Routed_Mode.md
# Zone_Based_Firewall_Routed_Mode
# Zone_Based_Firewall_Routed_Mode_Mental_Model
# Zone_Based_Firewall_Routed_Mode_Configuration_Checklist
# Zone_Based_Firewall_Routed_Mode_Outside_To_DMZ_Service_Checklist
# Zone_Based_Firewall_Routed_Mode_Self_Zone_Management_Checklist
# Zone_Based_Firewall_Routed_Mode_Skeleton
# Zone_Based_Firewall_Routed_Mode_Inside_Outside_With_DMZ_Skeleton
# Zone_Based_Firewall_Routed_Mode_Pass_Action_Skeleton
# Zone_Based_Firewall_Routed_Mode_Verification_Commands
# Zone_Based_Firewall_Routed_Mode_Rollback
# Zone_Based_Firewall_Routed_Mode_Failure_Checks
# Zone_Based_Firewall_Routed_Mode_Related_Labs
# Zone_Based_Firewall_Routed_Mode_Index
