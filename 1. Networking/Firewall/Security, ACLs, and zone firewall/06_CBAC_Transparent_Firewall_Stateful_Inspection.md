CBAC transparent IOS firewall uses CBAC and ACLs on bridged interfaces, with ip inspect configurable on bridged interfaces; Cisco’s transparent firewall guide also calls out bridge-group/BVI behavior and show bridge validation.  

CBAC_Transparent_Firewall_Stateful_Inspection.md
# CBAC_Transparent_Firewall_Stateful_Inspection
# CBAC_Transparent_Firewall_Stateful_Inspection_Mental_Model
| Concept | Operational Meaning |
|---|---|
| CBAC | Classic IOS stateful inspection using `ip inspect` |
| Transparent IOS firewall | Router bridges traffic at Layer 2 while applying firewall inspection and ACL enforcement |
| Bridged data interfaces | Physical interfaces carry traffic without IP addresses and belong to a bridge group |
| BVI | Bridge Virtual Interface used for management or routed reachability into the bridge group |
| Trusted side | Interface where inside/client-initiated sessions enter the firewall |
| Untrusted side | Interface where return traffic comes back and where unsolicited inbound traffic should be blocked |
| `ip inspect name` | Defines which protocols CBAC should inspect and track |
| `ip inspect <name> in` | Applies inspection to traffic entering the trusted bridged interface |
| Dynamic return allowance | CBAC opens temporary return holes for inspected flows |
| Static inbound ACL | Blocks unsolicited traffic from the untrusted side while CBAC allows inspected return traffic |
| Not ZBFW | CBAC does not use zones, zone pairs, class maps, or policy maps |
| Not ASA transparent mode | This is IOS router transparent firewalling, not ASA `firewall transparent` |
| ACL still matters | CBAC tracks sessions, but ACLs still define what unsolicited traffic is blocked or statically permitted |
| Lab purpose | Prove client-initiated traffic passes, return traffic is allowed dynamically, and untrusted-initiated traffic is denied |
# CBAC_Transparent_Firewall_Stateful_Inspection_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify trusted and untrusted bridged interfaces | R1 | `show ip interface brief`<br>`show interfaces description` | Two firewall data interfaces are identified |
| 2 | Confirm baseline physical status | R1 | `show interfaces <trusted_interface>`<br>`show interfaces <untrusted_interface>` | Both interfaces are up/up or ready to be enabled |
| 3 | Confirm existing bridge configuration | R1 | `show running-config \| include bridge`<br>`show bridge group` | Existing bridge groups are known before changes |
| 4 | Confirm existing CBAC configuration | R1 | `show ip inspect config`<br>`show ip inspect interfaces` | Existing inspection policies and attachments are known |
| 5 | Confirm existing ACL configuration | R1 | `show access-lists`<br>`show running-config interface <untrusted_interface>` | Existing interface ACLs are known |
| 6 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 7 | Enable transparent bridging with IRB | R1 | `bridge irb` | Integrated routing and bridging is enabled |
| 8 | Define bridge-group protocol | R1 | `bridge <bridge_group> protocol ieee` | Bridge group uses IEEE spanning tree behavior |
| 9 | Allow IP routing through the bridge group if BVI is used | R1 | `bridge <bridge_group> route ip` | IP can be routed through the BVI for management or control needs |
| 10 | Configure the trusted bridged interface | R1 | `interface <trusted_interface>`<br>`no ip address`<br>`no ip proxy-arp`<br>`bridge-group <bridge_group>`<br>`no shutdown` | Trusted interface is a Layer 2 member of the bridge group |
| 11 | Configure the untrusted bridged interface | R1 | `interface <untrusted_interface>`<br>`no ip address`<br>`no ip proxy-arp`<br>`bridge-group <bridge_group>`<br>`no shutdown` | Untrusted interface is a Layer 2 member of the bridge group |
| 12 | Configure BVI for management if required | R1 | `interface BVI<bridge_group>`<br>`ip address <bvi_ip> <mask>`<br>`no shutdown` | BVI has an IP address in the bridged subnet |
| 13 | Define CBAC inspection policy for TCP | R1 | `ip inspect name CBAC_FW tcp` | TCP flows are inspected statefully |
| 14 | Define CBAC inspection policy for UDP if needed | R1 | `ip inspect name CBAC_FW udp` | UDP flows are inspected with timeout-based state |
| 15 | Define CBAC inspection policy for ICMP if needed | R1 | `ip inspect name CBAC_FW icmp` | ICMP return traffic can be tracked |
| 16 | Define application-specific inspection if required | R1 | `ip inspect name CBAC_FW ftp`<br>`ip inspect name CBAC_FW dns`<br>`ip inspect name CBAC_FW http` | Selected application protocols are inspected when supported by the image |
| 17 | Enable audit trail if useful for lab visibility | R1 | `ip inspect audit-trail` | CBAC session creation and teardown logging is enabled |
| 18 | Enable drop packet logging if useful for troubleshooting | R1 | `ip inspect log drop-pkt` | Firewall drops can be logged |
| 19 | Apply inspection inbound on trusted bridged interface | R1 | `interface <trusted_interface>`<br>`ip inspect CBAC_FW in` | Trusted-to-untrusted sessions are inspected as they enter the firewall |
| 20 | Create untrusted inbound ACL | R1 | `ip access-list extended CBAC_UNTRUSTED_IN` | CLI enters extended ACL configuration mode |
| 21 | Permit DHCP replies if DHCP crosses the bridge | R1 | `permit udp any eq bootps any eq bootpc` | DHCP server replies are allowed if needed |
| 22 | Permit DHCP requests if required by the topology | R1 | `permit udp any eq bootpc any eq bootps` | DHCP client requests are allowed if required by ACL placement |
| 23 | Deny unsolicited traffic toward protected client subnet | R1 | `deny ip any <protected_subnet> <protected_wildcard> log` | New untrusted-to-protected flows are blocked and logged |
| 24 | Permit unrelated traffic if the interface carries traffic outside the protected subnet | R1 | `permit ip any any` | ACL does not accidentally block unrelated bridged traffic |
| 25 | Apply ACL inbound on untrusted bridged interface | R1 | `interface <untrusted_interface>`<br>`ip access-group CBAC_UNTRUSTED_IN in` | Unsolicited untrusted inbound traffic is filtered |
| 26 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 27 | Verify bridge forwarding state | R1 | `show bridge group`<br>`show bridge` | Trusted and untrusted interfaces are in bridge group and forwarding |
| 28 | Verify CBAC policy | R1 | `show ip inspect config` | `CBAC_FW` contains expected protocols |
| 29 | Verify inspection attachment | R1 | `show ip inspect interfaces` | `CBAC_FW` is applied inbound on the trusted interface |
| 30 | Verify ACL attachment | R1 | `show ip interface <untrusted_interface>` | Inbound ACL is `CBAC_UNTRUSTED_IN` |
| 31 | Test trusted-to-untrusted traffic | Client | `telnet <server_ip> <port>`<br>`ping <server_ip>` | Client-initiated traffic succeeds if protocol is inspected or statically permitted |
| 32 | Verify CBAC session table | R1 | `show ip inspect sessions` | Active inspected sessions appear |
| 33 | Test untrusted-to-trusted initiation | Server/Test Host | `telnet <client_ip> <port>`<br>`ping <client_ip>` | New untrusted-initiated traffic fails unless explicitly permitted |
| 34 | Verify ACL counters | R1 | `show access-lists CBAC_UNTRUSTED_IN` | Deny counter increments for unsolicited untrusted traffic |
| 35 | Save configuration | R1 | `write memory` | Transparent CBAC firewall configuration is saved |
# CBAC_Transparent_Firewall_Stateful_Inspection_Minimum_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enable bridge mode support | R1 | `configure terminal`<br>`bridge irb` | IRB is enabled |
| 2 | Define bridge group | R1 | `bridge 1 protocol ieee`<br>`bridge 1 route ip` | Bridge group 1 is ready for IP bridging and BVI use |
| 3 | Add trusted interface to bridge group | R1 | `interface <trusted_interface>`<br>`no ip address`<br>`ip inspect CBAC_FW in`<br>`bridge-group 1`<br>`no shutdown` | Trusted interface bridges traffic and inspects trusted-initiated sessions |
| 4 | Add untrusted interface to bridge group | R1 | `interface <untrusted_interface>`<br>`no ip address`<br>`ip access-group CBAC_UNTRUSTED_IN in`<br>`bridge-group 1`<br>`no shutdown` | Untrusted interface bridges traffic and filters unsolicited inbound traffic |
| 5 | Configure BVI | R1 | `interface BVI1`<br>`ip address <bvi_ip> <mask>`<br>`no shutdown` | Router has a management IP in the bridged subnet |
| 6 | Configure inspection policy | R1 | `ip inspect name CBAC_FW tcp`<br>`ip inspect name CBAC_FW udp`<br>`ip inspect name CBAC_FW icmp` | Common TCP, UDP, and ICMP traffic can be inspected |
| 7 | Configure untrusted inbound ACL | R1 | `ip access-list extended CBAC_UNTRUSTED_IN`<br>`deny ip any <protected_subnet> <protected_wildcard> log`<br>`permit ip any any` | Unsolicited traffic toward protected side is blocked |
| 8 | Verify firewall behavior | R1 | `show ip inspect sessions`<br>`show access-lists CBAC_UNTRUSTED_IN` | Inspected sessions and denied unsolicited attempts are visible |
# CBAC_Transparent_Firewall_Stateful_Inspection_Skeleton
configure terminal
bridge irb
bridge <bridge_group> protocol ieee
bridge <bridge_group> route ip
ip inspect name CBAC_FW tcp
ip inspect name CBAC_FW udp
ip inspect name CBAC_FW icmp
ip inspect audit-trail
ip inspect log drop-pkt
ip access-list extended CBAC_UNTRUSTED_IN
 permit udp any eq bootps any eq bootpc
 permit udp any eq bootpc any eq bootps
 deny ip any <protected_subnet> <protected_wildcard> log
 permit ip any any
exit
interface <trusted_interface>
 no ip address
 no ip proxy-arp
 ip inspect CBAC_FW in
 bridge-group <bridge_group>
 no shutdown
exit
interface <untrusted_interface>
 no ip address
 no ip proxy-arp
 ip access-group CBAC_UNTRUSTED_IN in
 bridge-group <bridge_group>
 no shutdown
exit
interface BVI<bridge_group>
 ip address <bvi_ip> <mask>
 no shutdown
exit
end
write memory
# CBAC_Transparent_Firewall_Stateful_Inspection_TCP_Only_Skeleton
configure terminal
bridge irb
bridge 1 protocol ieee
bridge 1 route ip
ip inspect name CBAC_FW tcp
ip access-list extended CBAC_UNTRUSTED_IN
 deny ip any <protected_subnet> <protected_wildcard> log
 permit ip any any
exit
interface <trusted_interface>
 no ip address
 ip inspect CBAC_FW in
 bridge-group 1
 no shutdown
exit
interface <untrusted_interface>
 no ip address
 ip access-group CBAC_UNTRUSTED_IN in
 bridge-group 1
 no shutdown
exit
interface BVI1
 ip address <bvi_ip> <mask>
 no shutdown
exit
end
write memory
# CBAC_Transparent_Firewall_Stateful_Inspection_No_BVI_Skeleton
configure terminal
no ip routing
bridge 1 protocol ieee
ip inspect name CBAC_FW tcp
ip inspect name CBAC_FW udp
ip inspect name CBAC_FW icmp
ip access-list extended CBAC_UNTRUSTED_IN
 deny ip any <protected_subnet> <protected_wildcard> log
 permit ip any any
exit
interface <trusted_interface>
 no ip address
 ip inspect CBAC_FW in
 bridge-group 1
 no shutdown
exit
interface <untrusted_interface>
 no ip address
 ip access-group CBAC_UNTRUSTED_IN in
 bridge-group 1
 no shutdown
exit
end
write memory
# CBAC_Transparent_Firewall_Stateful_Inspection_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Interface state | R1 | `show ip interface brief` | Trusted, untrusted, and BVI interfaces are up/up as expected |
| Bridge group state | R1 | `show bridge group` | Bridged interfaces are listed under the expected bridge group |
| Bridge MAC table | R1 | `show bridge` | MAC addresses are learned after traffic crosses the firewall |
| Bridge configuration | R1 | `show running-config \| include bridge` | `bridge irb`, bridge protocol, and bridge route commands are present |
| Trusted interface config | R1 | `show running-config interface <trusted_interface>` | Interface has `no ip address`, `bridge-group`, and `ip inspect CBAC_FW in` |
| Untrusted interface config | R1 | `show running-config interface <untrusted_interface>` | Interface has `no ip address`, `bridge-group`, and inbound ACL |
| BVI config | R1 | `show running-config interface BVI<bridge_group>` | BVI has management IP address if used |
| CBAC policy | R1 | `show ip inspect config` | Inspection policy lists expected protocols |
| CBAC interface attachment | R1 | `show ip inspect interfaces` | Inspection policy is applied to trusted interface in correct direction |
| CBAC sessions | R1 | `show ip inspect sessions` | Active state entries appear for inspected flows |
| CBAC statistics | R1 | `show ip inspect statistics` | Counters increment for inspected traffic |
| Inbound ACL definition | R1 | `show access-lists CBAC_UNTRUSTED_IN` | ACL contains DHCP permits if needed, protected deny, and catch-all permit |
| Inbound ACL attachment | R1 | `show ip interface <untrusted_interface>` | Inbound access list is `CBAC_UNTRUSTED_IN` |
| Trusted-to-untrusted TCP test | Client | `telnet <server_ip> <tcp_port>` | Client-initiated TCP succeeds if service is listening |
| Trusted-to-untrusted ICMP test | Client | `ping <server_ip>` | ICMP succeeds if ICMP inspection is configured |
| Untrusted-to-trusted initiation test | Server/Test Host | `telnet <client_ip> <tcp_port>` | New untrusted-initiated TCP fails unless statically permitted |
| Drop logging | R1 | `show logging` | Denied untrusted attempts appear if logging is enabled |
| Transparent firewall debug | R1 | `debug ip inspect L2-transparent` | Debug shows transparent firewall handling during controlled testing |
| Modern policy firewall debug | R1 | `debug policy-firewall` | Used on releases where `debug ip inspect` behavior is replaced |
| Stop debugging | R1 | `undebug all` | Debugging is disabled after test |
# CBAC_Transparent_Firewall_Stateful_Inspection_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove CBAC inspection from trusted interface | R1 | `configure terminal`<br>`interface <trusted_interface>`<br>`no ip inspect CBAC_FW in` | CBAC inspection is detached from trusted interface |
| 2 | Remove inbound ACL from untrusted interface | R1 | `interface <untrusted_interface>`<br>`no ip access-group CBAC_UNTRUSTED_IN in` | ACL is detached from untrusted interface |
| 3 | Remove bridged interface membership if dismantling transparent firewall | R1 | `interface <trusted_interface>`<br>`no bridge-group <bridge_group>`<br>`exit`<br>`interface <untrusted_interface>`<br>`no bridge-group <bridge_group>` | Interfaces are removed from bridge group |
| 4 | Remove BVI if no longer needed | R1 | `no interface BVI<bridge_group>` | BVI is removed |
| 5 | Remove CBAC inspection policy | R1 | `no ip inspect name CBAC_FW tcp`<br>`no ip inspect name CBAC_FW udp`<br>`no ip inspect name CBAC_FW icmp` | CBAC policy entries are removed |
| 6 | Disable optional CBAC logging | R1 | `no ip inspect audit-trail`<br>`no ip inspect log drop-pkt` | Optional CBAC logging is disabled |
| 7 | Remove untrusted inbound ACL | R1 | `no ip access-list extended CBAC_UNTRUSTED_IN` | ACL object is deleted |
| 8 | Remove bridge routing if no longer needed | R1 | `no bridge <bridge_group> route ip` | Bridge group no longer routes IP through BVI |
| 9 | Remove bridge protocol if no longer needed | R1 | `no bridge <bridge_group>` | Bridge group is removed |
| 10 | Disable IRB only if no other bridge groups depend on it | R1 | `no bridge irb` | IRB is disabled |
| 11 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 12 | Confirm CBAC removal | R1 | `show ip inspect interfaces`<br>`show ip inspect config` | Removed inspection policy is absent |
| 13 | Confirm ACL removal | R1 | `show access-lists CBAC_UNTRUSTED_IN` | ACL is absent |
| 14 | Confirm bridge cleanup | R1 | `show bridge group` | Removed bridge group no longer forwards interfaces |
| 15 | Save rollback | R1 | `write memory` | Rollback is saved |
# CBAC_Transparent_Firewall_Stateful_Inspection_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No traffic crosses firewall | Bridged interfaces are not in the same bridge group | R1 | `show bridge group`<br>`show running-config interface <trusted_interface>`<br>`show running-config interface <untrusted_interface>` | Configure matching `bridge-group <bridge_group>` on both data interfaces |
| No traffic crosses firewall | Bridge interfaces are not forwarding yet | R1 | `show bridge group` | Wait for STP state or correct bridge/STP issue |
| No traffic crosses firewall | Interface is down or administratively shut | R1 | `show ip interface brief` | Configure `no shutdown` and fix physical/link problem |
| BVI is unreachable | BVI IP is not in the bridged subnet | R1 | `show running-config interface BVI<bridge_group>`<br>`show ip route` | Configure BVI IP in the correct subnet |
| BVI is unreachable | `bridge <bridge_group> route ip` is missing | R1 | `show running-config \| include bridge <bridge_group> route ip` | Add `bridge <bridge_group> route ip` |
| CBAC sessions do not appear | Inspection applied on wrong interface or wrong direction | R1 | `show ip inspect interfaces` | Apply `ip inspect CBAC_FW in` on the trusted ingress interface |
| Client-initiated TCP fails | Untrusted inbound ACL blocks return traffic and CBAC did not create state | R1 | `show ip inspect sessions`<br>`show access-lists CBAC_UNTRUSTED_IN` | Fix inspection placement or protocol inspection |
| ICMP fails while TCP works | ICMP inspection is missing | R1 | `show ip inspect config` | Add `ip inspect name CBAC_FW icmp` |
| UDP application fails | UDP inspection is missing or timeout expires | R1 | `show ip inspect config`<br>`show ip inspect sessions` | Add `ip inspect name CBAC_FW udp` or tune timeout |
| FTP fails | Application-specific inspection is missing | R1 | `show ip inspect config` | Add `ip inspect name CBAC_FW ftp` |
| DHCP fails across the bridge | Inbound ACL blocks DHCP packets | R1 | `show access-lists CBAC_UNTRUSTED_IN` | Add DHCP permit entries before deny |
| Untrusted host can initiate traffic to protected side | ACL is missing, too broad, or applied wrong direction | R1 | `show ip interface <untrusted_interface>`<br>`show access-lists CBAC_UNTRUSTED_IN` | Apply restrictive ACL inbound on untrusted interface |
| All untrusted traffic is blocked, including unrelated traffic | Missing catch-all permit after protected subnet deny | R1 | `show access-lists CBAC_UNTRUSTED_IN` | Add controlled `permit ip any any` only if the topology requires unrelated traffic |
| ACL counters do not increment | Test traffic is not entering through the untrusted interface | R1 | `show bridge`<br>`show interfaces <untrusted_interface>` | Correct cabling, topology, or test direction |
| Bridge MAC table is empty | No traffic has crossed the bridge yet | R1 | `show bridge` | Generate traffic across the firewall |
| Debug output missing | Wrong debug command for IOS release | R1 | `debug ip inspect L2-transparent`<br>`debug policy-firewall` | Use the debug command supported by the IOS release |
| Feature commands missing | IOS image does not support CBAC or transparent firewall feature set | R1 | `show version`<br>`show parser dump \| include ip inspect` | Use an IOS image that supports CBAC transparent firewalling |
| Return traffic works briefly then fails | State timeout expires before return traffic arrives | R1 | `show ip inspect sessions` | Tune CBAC timeout or verify application behavior |
| Traffic still blocked after rollback | ACL, ZBFW, uRPF, VACL, or host firewall still blocks traffic | R1 | `show access-lists`<br>`show running-config interface <interface>`<br>`show policy-map type inspect zone-pair` | Remove or correct the remaining enforcement feature |
# CBAC_Transparent_Firewall_Stateful_Inspection_Related_Labs
- transparent-firewall-cbac-final
# CBAC_Transparent_Firewall_Stateful_Inspection_Index
CBAC_Transparent_Firewall_Stateful_Inspection.md
# CBAC_Transparent_Firewall_Stateful_Inspection
# CBAC_Transparent_Firewall_Stateful_Inspection_Mental_Model
# CBAC_Transparent_Firewall_Stateful_Inspection_Configuration_Checklist
# CBAC_Transparent_Firewall_Stateful_Inspection_Minimum_Checklist
# CBAC_Transparent_Firewall_Stateful_Inspection_Skeleton
# CBAC_Transparent_Firewall_Stateful_Inspection_TCP_Only_Skeleton
# CBAC_Transparent_Firewall_Stateful_Inspection_No_BVI_Skeleton
# CBAC_Transparent_Firewall_Stateful_Inspection_Verification_Commands
# CBAC_Transparent_Firewall_Stateful_Inspection_Rollback
# CBAC_Transparent_Firewall_Stateful_Inspection_Failure_Checks
# CBAC_Transparent_Firewall_Stateful_Inspection_Related_Labs
# CBAC_Transparent_Firewall_Stateful_Inspection_Index
