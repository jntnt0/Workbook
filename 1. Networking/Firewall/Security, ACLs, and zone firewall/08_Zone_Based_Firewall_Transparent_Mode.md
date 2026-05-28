
Zone_Based_Firewall_Transparent_Mode.md
# Zone_Based_Firewall_Transparent_Mode
# Zone_Based_Firewall_Transparent_Mode_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Transparent ZBFW | IOS firewall enforcement while the device forwards traffic at Layer 2 instead of acting as the default gateway |
| Layer 2 transit firewall | Hosts on both sides can remain in the same IP subnet while traffic passes through the firewall bridge path |
| Bridge group | Logical Layer 2 forwarding domain that joins the trusted and untrusted data interfaces |
| BVI | Bridge Virtual Interface used for management or control-plane reachability, not normal host default-gateway routing |
| Security zone | Logical firewall boundary assigned to bridge member interfaces |
| Zone pair | Directional firewall policy path from one zone to another |
| Direction matters | `INSIDE` to `OUTSIDE` is separate from `OUTSIDE` to `INSIDE` |
| `inspect` action | Stateful allow; return traffic for inspected flows is permitted dynamically |
| `pass` action | Stateless allow; reverse traffic needs its own policy |
| `drop` action | Blocks matched traffic; `drop log` adds visibility |
| Same subnet, policy enforced | Even though endpoints may share the same subnet, traffic can still be inspected as it crosses zones |
| Not routed ZBFW | Routed ZBFW applies between routed interfaces; transparent ZBFW applies while bridging traffic |
| Not ASA transparent mode | Do not use ASA commands such as `firewall transparent`; this is IOS/IOS XE ZBFW with bridging |
| Common failure point | Bridge forwarding, zone membership, or zone pair direction is wrong |
| Lab purpose | Prove inside-to-outside traffic is inspected across a Layer 2 bridge, return traffic passes dynamically, and outside-initiated traffic is blocked |
# Zone_Based_Firewall_Transparent_Mode_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify trusted and untrusted Layer 2 firewall interfaces | R1 | `show ip interface brief`<br>`show interfaces description` | Two data interfaces are identified for transparent firewall transit |
| 2 | Confirm the interfaces are physically up | R1 | `show interfaces <inside_interface>`<br>`show interfaces <outside_interface>` | Both interfaces are up/up or ready to be enabled |
| 3 | Confirm baseline addressing plan | R1 | `show running-config interface <inside_interface>`<br>`show running-config interface <outside_interface>` | Data interfaces should not keep routed IP addresses in transparent mode |
| 4 | Confirm existing bridge configuration | R1 | `show bridge group`<br>`show bridge`<br>`show running-config \| include bridge` | Existing bridge groups are known before changes |
| 5 | Confirm existing ZBFW configuration | R1 | `show zone security`<br>`show zone-pair security`<br>`show policy-map type inspect` | Existing zones, zone pairs, and inspect policies are known |
| 6 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 7 | Enable integrated routing and bridging | R1 | `bridge irb` | Router supports BVI use with bridged interfaces |
| 8 | Define the bridge-group protocol | R1 | `bridge <bridge_group> protocol ieee` | Bridge group uses IEEE bridging behavior |
| 9 | Enable IP routing for the bridge group if BVI management is required | R1 | `bridge <bridge_group> route ip` | BVI can be used for IP management/control traffic |
| 10 | Create inside security zone | R1 | `zone security INSIDE`<br>`description Trusted bridged side` | INSIDE zone exists |
| 11 | Create outside security zone | R1 | `zone security OUTSIDE`<br>`description Untrusted bridged side` | OUTSIDE zone exists |
| 12 | Configure inside bridge member interface | R1 | `interface <inside_interface>`<br>`no ip address`<br>`no ip proxy-arp`<br>`bridge-group <bridge_group>`<br>`zone-member security INSIDE`<br>`no shutdown` | Inside interface bridges traffic and belongs to INSIDE zone |
| 13 | Configure outside bridge member interface | R1 | `interface <outside_interface>`<br>`no ip address`<br>`no ip proxy-arp`<br>`bridge-group <bridge_group>`<br>`zone-member security OUTSIDE`<br>`no shutdown` | Outside interface bridges traffic and belongs to OUTSIDE zone |
| 14 | Configure BVI for management if required | R1 | `interface BVI<bridge_group>`<br>`ip address <bvi_ip> <mask>`<br>`no shutdown` | BVI has an IP address in the bridged subnet |
| 15 | Create ACL matching inside-to-outside traffic | R1 | `ip access-list extended ACL_INSIDE_TO_OUTSIDE` | CLI enters extended ACL mode |
| 16 | Match trusted-side traffic for stateful inspection | R1 | `permit ip <inside_subnet> <inside_wildcard> any` | Inside-originated traffic is selected for inspection |
| 17 | Exit ACL mode | R1 | `exit` | CLI returns to global configuration mode |
| 18 | Create inspect class map | R1 | `class-map type inspect match-any CM_INSIDE_TO_OUTSIDE` | CLI enters inspect class-map mode |
| 19 | Match the inside-to-outside ACL | R1 | `match access-group name ACL_INSIDE_TO_OUTSIDE` | Class map selects inside-originated traffic |
| 20 | Exit class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 21 | Create inspect policy map | R1 | `policy-map type inspect PM_INSIDE_TO_OUTSIDE` | CLI enters inspect policy-map mode |
| 22 | Add inspection class | R1 | `class type inspect CM_INSIDE_TO_OUTSIDE` | CLI enters class action mode |
| 23 | Inspect matched traffic | R1 | `inspect` | Inside-to-outside traffic is statefully allowed |
| 24 | Drop everything else in the policy | R1 | `class class-default`<br>`drop log` | Unmatched traffic is denied and logged |
| 25 | Exit policy-map mode | R1 | `exit`<br>`exit` | CLI returns to global configuration mode |
| 26 | Create inside-to-outside zone pair | R1 | `zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE` | Directional zone pair is created |
| 27 | Attach inspect policy to zone pair | R1 | `service-policy type inspect PM_INSIDE_TO_OUTSIDE` | Inside-to-outside policy is active |
| 28 | Exit zone-pair mode | R1 | `exit` | CLI returns to global configuration mode |
| 29 | Create optional outside-to-inside explicit deny policy for visibility | R1 | `policy-map type inspect PM_OUTSIDE_TO_INSIDE`<br>`class class-default`<br>`drop log` | Outside-to-inside traffic can be explicitly dropped and logged |
| 30 | Create outside-to-inside zone pair if explicit drop visibility is required | R1 | `zone-pair security ZP_OUTSIDE_TO_INSIDE source OUTSIDE destination INSIDE`<br>`service-policy type inspect PM_OUTSIDE_TO_INSIDE` | Outside-initiated traffic is explicitly dropped and counted |
| 31 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 32 | Verify bridge state | R1 | `show bridge group`<br>`show bridge` | Both data interfaces are in the bridge group and MAC learning occurs |
| 33 | Verify zones and zone membership | R1 | `show zone security` | Inside and outside bridge interfaces appear in correct zones |
| 34 | Verify zone pairs | R1 | `show zone-pair security` | Zone pairs show correct source and destination zones |
| 35 | Verify inspect policy | R1 | `show policy-map type inspect` | Policy contains inspect action and class-default drop |
| 36 | Verify zone-pair counters | R1 | `show policy-map type inspect zone-pair` | Counters are present for inspected or dropped traffic |
| 37 | Test inside-to-outside traffic | Inside Host | `ping <outside_host_ip>`<br>`telnet <outside_host_ip> <port>`<br>`curl http://<outside_host_ip>` | Inside-originated traffic succeeds if protocol and service are valid |
| 38 | Verify stateful inspection behavior | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Inspect counters increment |
| 39 | Test outside-to-inside initiation | Outside Host | `ping <inside_host_ip>`<br>`telnet <inside_host_ip> <port>` | Unsolicited outside-originated traffic fails unless a reverse allow policy exists |
| 40 | Save configuration | R1 | `write memory` | Transparent ZBFW configuration is saved |
# Zone_Based_Firewall_Transparent_Mode_Minimum_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enable IRB and bridge group | R1 | `configure terminal`<br>`bridge irb`<br>`bridge 1 protocol ieee`<br>`bridge 1 route ip` | Bridge group 1 is ready |
| 2 | Create zones | R1 | `zone security INSIDE`<br>`zone security OUTSIDE` | INSIDE and OUTSIDE zones exist |
| 3 | Configure inside bridge interface | R1 | `interface <inside_interface>`<br>`no ip address`<br>`bridge-group 1`<br>`zone-member security INSIDE`<br>`no shutdown` | Inside interface bridges traffic and belongs to INSIDE |
| 4 | Configure outside bridge interface | R1 | `interface <outside_interface>`<br>`no ip address`<br>`bridge-group 1`<br>`zone-member security OUTSIDE`<br>`no shutdown` | Outside interface bridges traffic and belongs to OUTSIDE |
| 5 | Configure BVI if needed | R1 | `interface BVI1`<br>`ip address <bvi_ip> <mask>`<br>`no shutdown` | BVI is available for management |
| 6 | Match inside-to-outside traffic | R1 | `ip access-list extended ACL_INSIDE_TO_OUTSIDE`<br>`permit ip <inside_subnet> <inside_wildcard> any` | Inside-originated traffic is selected |
| 7 | Create inspect class map | R1 | `class-map type inspect match-any CM_INSIDE_TO_OUTSIDE`<br>`match access-group name ACL_INSIDE_TO_OUTSIDE` | Class map references the ACL |
| 8 | Create inspect policy | R1 | `policy-map type inspect PM_INSIDE_TO_OUTSIDE`<br>`class type inspect CM_INSIDE_TO_OUTSIDE`<br>`inspect`<br>`class class-default`<br>`drop log` | Inside-to-outside traffic is inspected; unmatched traffic is dropped |
| 9 | Create zone pair | R1 | `zone-pair security ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE`<br>`service-policy type inspect PM_INSIDE_TO_OUTSIDE` | Policy is active between INSIDE and OUTSIDE |
| 10 | Verify transparent ZBFW | R1 | `show bridge group`<br>`show zone security`<br>`show policy-map type inspect zone-pair` | Bridge state, zone membership, and policy counters are visible |
# Zone_Based_Firewall_Transparent_Mode_Outside_To_Inside_Service_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the inside server and allowed service | R1 | `show bridge`<br>`show arp` | Server MAC/IP is visible or reachable across the bridge |
| 2 | Create ACL for allowed outside-to-inside service | R1 | `configure terminal`<br>`ip access-list extended ACL_OUTSIDE_TO_INSIDE_SERVICE` | CLI enters ACL mode |
| 3 | Permit the specific service only | R1 | `permit tcp any host <inside_server_ip> eq <service_port>` | Only selected outside-to-inside service traffic is matched |
| 4 | Exit ACL mode | R1 | `exit` | CLI returns to global configuration mode |
| 5 | Create inspect class map | R1 | `class-map type inspect match-any CM_OUTSIDE_TO_INSIDE_SERVICE` | CLI enters class-map mode |
| 6 | Match the service ACL | R1 | `match access-group name ACL_OUTSIDE_TO_INSIDE_SERVICE` | Class map selects only the allowed service |
| 7 | Exit class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 8 | Create outside-to-inside policy | R1 | `policy-map type inspect PM_OUTSIDE_TO_INSIDE_SERVICE` | CLI enters policy-map mode |
| 9 | Inspect the allowed service | R1 | `class type inspect CM_OUTSIDE_TO_INSIDE_SERVICE`<br>`inspect` | Allowed service is statefully permitted |
| 10 | Drop all other outside-to-inside traffic | R1 | `class class-default`<br>`drop log` | Unmatched outside-to-inside traffic is dropped |
| 11 | Create outside-to-inside zone pair | R1 | `zone-pair security ZP_OUTSIDE_TO_INSIDE source OUTSIDE destination INSIDE` | Directional reverse zone pair is created |
| 12 | Attach the outside-to-inside service policy | R1 | `service-policy type inspect PM_OUTSIDE_TO_INSIDE_SERVICE` | Outside-to-inside policy is active |
| 13 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 14 | Test allowed service | Outside Host | `telnet <inside_server_ip> <service_port>`<br>`curl http://<inside_server_ip>` | Allowed service succeeds if server and bridge path are valid |
| 15 | Test denied service | Outside Host | `telnet <inside_server_ip> <denied_port>` | Non-approved service fails |
| 16 | Verify counters | R1 | `show policy-map type inspect zone-pair ZP_OUTSIDE_TO_INSIDE` | Allowed service counters and default drop counters increment |
| 17 | Save configuration | R1 | `write memory` | Reverse service policy is saved |
# Zone_Based_Firewall_Transparent_Mode_Self_Zone_Management_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify required management access to the BVI | R1 | `show running-config interface BVI<bridge_group>`<br>`show control-plane host open-ports` | BVI IP and required services are known |
| 2 | Create ACL for trusted management hosts | R1 | `configure terminal`<br>`ip access-list extended ACL_TO_SELF_MGMT` | CLI enters ACL mode |
| 3 | Permit SSH to BVI from management subnet | R1 | `permit tcp <mgmt_subnet> <mgmt_wildcard> host <bvi_ip> eq 22` | SSH to the BVI is selected |
| 4 | Permit ICMP to BVI if required | R1 | `permit icmp <mgmt_subnet> <mgmt_wildcard> host <bvi_ip>` | Ping to BVI is selected if required |
| 5 | Exit ACL mode | R1 | `exit` | CLI returns to global configuration mode |
| 6 | Create self-zone class map | R1 | `class-map type inspect match-any CM_TO_SELF_MGMT` | CLI enters class-map mode |
| 7 | Match management ACL | R1 | `match access-group name ACL_TO_SELF_MGMT` | Class map selects allowed management traffic |
| 8 | Exit class-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 9 | Create self-zone policy | R1 | `policy-map type inspect PM_TO_SELF_MGMT` | CLI enters policy-map mode |
| 10 | Pass allowed management traffic | R1 | `class type inspect CM_TO_SELF_MGMT`<br>`pass` | Allowed management traffic reaches the router control plane |
| 11 | Drop other traffic to self | R1 | `class class-default`<br>`drop log` | Unmatched traffic to the router is dropped |
| 12 | Create inside-to-self zone pair if management comes from inside | R1 | `zone-pair security ZP_INSIDE_TO_SELF source INSIDE destination self`<br>`service-policy type inspect PM_TO_SELF_MGMT` | Inside-to-router management is controlled |
| 13 | Create outside-to-self zone pair if management comes from outside | R1 | `zone-pair security ZP_OUTSIDE_TO_SELF source OUTSIDE destination self`<br>`service-policy type inspect PM_TO_SELF_MGMT` | Outside-to-router management is controlled |
| 14 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 15 | Test allowed management | Management Host | `ssh <bvi_ip>`<br>`ping <bvi_ip>` | Approved management host reaches BVI |
| 16 | Test denied management | Untrusted Host | `ssh <bvi_ip>` | Unapproved management host fails |
| 17 | Verify self-zone counters | R1 | `show policy-map type inspect zone-pair` | Self-zone policy counters increment |
| 18 | Save configuration | R1 | `write memory` | Self-zone management policy is saved |
# Zone_Based_Firewall_Transparent_Mode_Skeleton
configure terminal
bridge irb
bridge <bridge_group> protocol ieee
bridge <bridge_group> route ip
zone security INSIDE
 description Trusted bridged side
exit
zone security OUTSIDE
 description Untrusted bridged side
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
 no ip address
 no ip proxy-arp
 bridge-group <bridge_group>
 zone-member security INSIDE
 no shutdown
exit
interface <outside_interface>
 no ip address
 no ip proxy-arp
 bridge-group <bridge_group>
 zone-member security OUTSIDE
 no shutdown
exit
interface BVI<bridge_group>
 ip address <bvi_ip> <mask>
 no shutdown
exit
end
write memory
# Zone_Based_Firewall_Transparent_Mode_Explicit_Outside_Drop_Skeleton
configure terminal
policy-map type inspect PM_OUTSIDE_TO_INSIDE
 class class-default
  drop log
exit
zone-pair security ZP_OUTSIDE_TO_INSIDE source OUTSIDE destination INSIDE
 service-policy type inspect PM_OUTSIDE_TO_INSIDE
exit
end
write memory
# Zone_Based_Firewall_Transparent_Mode_Outside_To_Inside_Service_Skeleton
configure terminal
ip access-list extended ACL_OUTSIDE_TO_INSIDE_SERVICE
 permit tcp any host <inside_server_ip> eq <service_port>
exit
class-map type inspect match-any CM_OUTSIDE_TO_INSIDE_SERVICE
 match access-group name ACL_OUTSIDE_TO_INSIDE_SERVICE
exit
policy-map type inspect PM_OUTSIDE_TO_INSIDE_SERVICE
 class type inspect CM_OUTSIDE_TO_INSIDE_SERVICE
  inspect
 class class-default
  drop log
exit
zone-pair security ZP_OUTSIDE_TO_INSIDE source OUTSIDE destination INSIDE
 service-policy type inspect PM_OUTSIDE_TO_INSIDE_SERVICE
exit
end
write memory
# Zone_Based_Firewall_Transparent_Mode_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Interface state | R1 | `show ip interface brief` | Inside, outside, and BVI interfaces are up/up as expected |
| Bridge group state | R1 | `show bridge group` | Inside and outside data interfaces are in the same bridge group |
| Bridge MAC learning | R1 | `show bridge` | MAC addresses are learned after traffic crosses the bridge |
| Bridge configuration | R1 | `show running-config \| include bridge` | `bridge irb`, bridge protocol, and bridge route commands are present |
| BVI configuration | R1 | `show running-config interface BVI<bridge_group>` | BVI has correct management IP if used |
| Zone existence | R1 | `show zone security` | INSIDE and OUTSIDE zones exist |
| Zone membership | R1 | `show zone security` | Bridge member interfaces are assigned to correct zones |
| Zone pairs | R1 | `show zone-pair security` | Directional zone pairs show correct source and destination zones |
| Class maps | R1 | `show class-map type inspect` | Class maps reference expected ACLs |
| Policy maps | R1 | `show policy-map type inspect` | Policies show inspect, pass, drop, or drop log actions |
| Zone-pair counters | R1 | `show policy-map type inspect zone-pair` | Counters increment when traffic crosses the transparent firewall |
| Specific inside-to-outside counters | R1 | `show policy-map type inspect zone-pair ZP_INSIDE_TO_OUTSIDE` | Inspect counters increment for allowed inside-originated traffic |
| ACL counters | R1 | `show access-lists ACL_INSIDE_TO_OUTSIDE` | ACL counters increment for matched inside-originated traffic |
| Interface running config | R1 | `show running-config interface <inside_interface>`<br>`show running-config interface <outside_interface>` | Data interfaces have `no ip address`, `bridge-group`, and `zone-member security` |
| Inside-to-outside test | Inside Host | `ping <outside_host_ip>`<br>`telnet <outside_host_ip> <port>`<br>`curl http://<outside_host_ip>` | Inside-originated traffic succeeds if inspected and service is valid |
| Outside-to-inside denied test | Outside Host | `ping <inside_host_ip>`<br>`telnet <inside_host_ip> <port>` | New unsolicited outside-to-inside traffic fails unless explicitly permitted |
| BVI management test | Management Host | `ssh <bvi_ip>`<br>`ping <bvi_ip>` | Management works only if self-zone and VTY policy allow it |
| Drop logging | R1 | `show logging` | Drop logs appear for policies using `drop log` |
# Zone_Based_Firewall_Transparent_Mode_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove zone membership from inside bridge interface | R1 | `configure terminal`<br>`interface <inside_interface>`<br>`no zone-member security INSIDE` | Inside interface is removed from ZBFW enforcement |
| 2 | Remove zone membership from outside bridge interface | R1 | `interface <outside_interface>`<br>`no zone-member security OUTSIDE` | Outside interface is removed from ZBFW enforcement |
| 3 | Remove bridge-group membership from inside interface if dismantling transparent firewall | R1 | `interface <inside_interface>`<br>`no bridge-group <bridge_group>` | Inside interface leaves the bridge group |
| 4 | Remove bridge-group membership from outside interface if dismantling transparent firewall | R1 | `interface <outside_interface>`<br>`no bridge-group <bridge_group>` | Outside interface leaves the bridge group |
| 5 | Exit interface mode | R1 | `exit` | CLI returns to global configuration mode |
| 6 | Remove inside-to-outside zone pair | R1 | `no zone-pair security ZP_INSIDE_TO_OUTSIDE` | Inside-to-outside zone pair is removed |
| 7 | Remove outside-to-inside zone pair if configured | R1 | `no zone-pair security ZP_OUTSIDE_TO_INSIDE` | Outside-to-inside zone pair is removed |
| 8 | Remove self-zone pairs if configured | R1 | `no zone-pair security ZP_INSIDE_TO_SELF`<br>`no zone-pair security ZP_OUTSIDE_TO_SELF` | Self-zone policies are removed |
| 9 | Remove inspect policy maps | R1 | `no policy-map type inspect PM_INSIDE_TO_OUTSIDE`<br>`no policy-map type inspect PM_OUTSIDE_TO_INSIDE`<br>`no policy-map type inspect PM_OUTSIDE_TO_INSIDE_SERVICE` | Inspect policies are deleted |
| 10 | Remove inspect class maps | R1 | `no class-map type inspect CM_INSIDE_TO_OUTSIDE`<br>`no class-map type inspect CM_OUTSIDE_TO_INSIDE_SERVICE` | Inspect class maps are deleted |
| 11 | Remove match ACLs if not reused | R1 | `no ip access-list extended ACL_INSIDE_TO_OUTSIDE`<br>`no ip access-list extended ACL_OUTSIDE_TO_INSIDE_SERVICE` | ACL objects are deleted |
| 12 | Remove BVI if no longer needed | R1 | `no interface BVI<bridge_group>` | BVI is deleted |
| 13 | Remove bridge routing if no longer needed | R1 | `no bridge <bridge_group> route ip` | Bridge-group IP routing support is removed |
| 14 | Remove bridge group if no longer needed | R1 | `no bridge <bridge_group>` | Bridge group is removed |
| 15 | Disable IRB only if no other bridge groups use it | R1 | `no bridge irb` | IRB is disabled |
| 16 | Remove zones if no longer used | R1 | `no zone security INSIDE`<br>`no zone security OUTSIDE` | Security zones are deleted |
| 17 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 18 | Confirm ZBFW cleanup | R1 | `show zone security`<br>`show zone-pair security`<br>`show policy-map type inspect` | Removed zones, zone pairs, and policies are absent |
| 19 | Confirm bridge cleanup | R1 | `show bridge group`<br>`show bridge` | Removed bridge group no longer forwards traffic |
| 20 | Save rollback | R1 | `write memory` | Rollback is saved |
# Zone_Based_Firewall_Transparent_Mode_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No traffic crosses the firewall | Data interfaces are not in the same bridge group | R1 | `show bridge group`<br>`show running-config interface <inside_interface>`<br>`show running-config interface <outside_interface>` | Add both data interfaces to the same `bridge-group` |
| No traffic crosses the firewall | Bridge interfaces are not forwarding | R1 | `show bridge group` | Correct bridge/STP state or wait for forwarding |
| No traffic crosses the firewall | Interface is administratively down | R1 | `show ip interface brief` | Configure `no shutdown` and fix link state |
| Traffic crosses bridge but firewall counters do not increment | Interfaces are not assigned to zones | R1 | `show zone security` | Add `zone-member security INSIDE` and `zone-member security OUTSIDE` |
| Traffic crosses bridge but wrong policy is hit | Zone pair source and destination are reversed | R1 | `show zone-pair security` | Build zone pair in the actual traffic direction |
| Inside-to-outside traffic fails | Missing inside-to-outside zone pair | R1 | `show zone-pair security` | Create `ZP_INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE` |
| Inside-to-outside traffic fails | Policy is not attached to zone pair | R1 | `show zone-pair security` | Add `service-policy type inspect PM_INSIDE_TO_OUTSIDE` |
| Inside-to-outside traffic hits class-default | ACL or class map does not match the traffic | R1 | `show access-lists ACL_INSIDE_TO_OUTSIDE`<br>`show class-map type inspect` | Correct ACL source, destination, protocol, wildcard, or class-map reference |
| Return traffic fails | Policy uses `pass` instead of `inspect` | R1 | `show policy-map type inspect PM_INSIDE_TO_OUTSIDE` | Use `inspect` for stateful flows or add reverse policy |
| Outside-to-inside traffic succeeds unexpectedly | Reverse zone pair permits it | R1 | `show zone-pair security`<br>`show policy-map type inspect zone-pair ZP_OUTSIDE_TO_INSIDE` | Remove or tighten OUTSIDE-to-INSIDE policy |
| BVI is unreachable | BVI IP address is wrong or missing | R1 | `show running-config interface BVI<bridge_group>` | Configure correct BVI IP in the bridged subnet |
| BVI is unreachable | `bridge <bridge_group> route ip` is missing | R1 | `show running-config \| include bridge <bridge_group> route ip` | Add `bridge <bridge_group> route ip` |
| Router management breaks after self-zone policy | Required SSH, ICMP, SNMP, or routing protocol was not permitted to self zone | R1 | `show zone-pair security`<br>`show policy-map type inspect zone-pair` | Add explicit self-zone policy entries or remove self-zone enforcement |
| Same-subnet hosts cannot ARP | Bridge path or host subnet design is wrong | R1 | `show bridge`<br>`show arp` | Verify both hosts are in the same subnet and bridge MAC learning works |
| MAC table remains empty | No traffic is crossing the bridge or cabling is wrong | R1 | `show bridge`<br>`show interfaces <interface>` | Generate host traffic and verify topology |
| ACL counters increment but policy counters do not | ACL is not referenced by active class map or policy | R1 | `show class-map type inspect`<br>`show policy-map type inspect` | Attach the correct ACL to the active class map and policy |
| Policy counters increment but application fails | Server, host firewall, DNS, or application service is broken | R1 | `show policy-map type inspect zone-pair`<br>`show arp` | Troubleshoot endpoint and application after firewall path is proven |
| Command is rejected | Platform or image does not support transparent ZBFW or bridge syntax | R1 | `show version` | Use an IOS/IOS XE image that supports ZBFW and bridging features |
| Traffic still fails after ZBFW rollback | ACL, uRPF, VACL, port security, IP Source Guard, or host firewall still blocks traffic | R1 | `show access-lists`<br>`show ip verify source`<br>`show port-security`<br>`show running-config interface <interface>` | Remove or correct the remaining enforcement feature |
# Zone_Based_Firewall_Transparent_Mode_Related_Labs
- zone-based-firewall-transparent-mode-final
# Zone_Based_Firewall_Transparent_Mode_Index
Zone_Based_Firewall_Transparent_Mode.md
# Zone_Based_Firewall_Transparent_Mode
# Zone_Based_Firewall_Transparent_Mode_Mental_Model
# Zone_Based_Firewall_Transparent_Mode_Configuration_Checklist
# Zone_Based_Firewall_Transparent_Mode_Minimum_Checklist
# Zone_Based_Firewall_Transparent_Mode_Outside_To_Inside_Service_Checklist
# Zone_Based_Firewall_Transparent_Mode_Self_Zone_Management_Checklist
# Zone_Based_Firewall_Transparent_Mode_Skeleton
# Zone_Based_Firewall_Transparent_Mode_Explicit_Outside_Drop_Skeleton
# Zone_Based_Firewall_Transparent_Mode_Outside_To_Inside_Service_Skeleton
# Zone_Based_Firewall_Transparent_Mode_Verification_Commands
# Zone_Based_Firewall_Transparent_Mode_Rollback
# Zone_Based_Firewall_Transparent_Mode_Failure_Checks
# Zone_Based_Firewall_Transparent_Mode_Related_Labs
# Zone_Based_Firewall_Transparent_Mode_Index
