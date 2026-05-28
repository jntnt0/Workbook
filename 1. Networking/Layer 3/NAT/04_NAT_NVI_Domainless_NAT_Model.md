Local source checked: _KB_INDEX.md points IOS XE NAT to iosxe_combined_pdfs_.md, lines 2,798 to 3,545. The local source does not clearly expose NVI, so the NVI-specific syntax below uses Cisco’s NAT Virtual Interface feature documentation: interfaces use ip nat enable, NVI rules use ip nat source, and static, dynamic, PAT, and static PAT are configured without inside or outside keywords. Cisco’s IOS XE NAT guide is still the baseline for normal NAT requirements and verification.  

NAT_NVI_Domainless_NAT_Model.md

NAT_NVI_Domainless_NAT_Model

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `iosxe_combined_pdfs_.md` NAT guide reference | Points IOS XE NAT configuration content to the IOS XE combined source |
| `iosxe_combined_pdfs_.md` | `NAT Configuration Guide` > `Configuring NAT for IP Address Conservation` | Supports NAT requirements, NAT pools, NAT ACLs, static NAT, dynamic NAT, PAT, and verification workflow |
| Cisco NAT Virtual Interface feature documentation | `NAT Virtual Interface` > `ip nat enable` | Provides NVI interface enablement syntax without classic inside/outside roles |
| Cisco NAT Virtual Interface feature documentation | `ip nat source` | Provides NVI dynamic NAT, static NAT, static PAT, network static NAT, overload, and VRF-aware syntax |
| Related lab | `nat-virtual-interface-nat-dynamic-final` | Dynamic NVI NAT lab target |
| Related lab | `nat-virtual-interface-nat-static-final` | Static NVI NAT lab target |
| Related lab | `nat-virtual-interface-pat-overload-final` | NVI PAT overload lab target |
| Related lab | `nat-virtual-interface-pat-static-final` | NVI static PAT lab target |
# NAT_NVI_Domainless_NAT_Model_Mental_Model
| Concept | Operational Meaning |
|---|---|
| NVI | NAT Virtual Interface. It removes the classic requirement to define one interface as `ip nat inside` and another as `ip nat outside`. |
| Domainless NAT | NVI NAT is enabled on participating interfaces with `ip nat enable` instead of `ip nat inside` and `ip nat outside`. |
| NVI rule syntax | NVI uses `ip nat source`, not `ip nat inside source`. That is the main syntax difference from classic NAT. |
| Route-first thinking | NVI is easier to reason about as NAT applied after the router decides how traffic should be forwarded. Routing still must be correct. |
| Dynamic NVI NAT | A source ACL selects local addresses, and a NAT pool provides translated addresses. |
| NVI PAT overload | Many local hosts can share one translated address by using transport-layer port numbers. |
| Static NVI NAT | One local address maps permanently to one global address using `ip nat source static <local> <global>`. |
| Static NVI PAT | One local service maps permanently to one global service using protocol and port detail. |
| Do not mix mental models | Classic NAT uses `ip nat inside source` plus `ip nat inside/outside`; NVI uses `ip nat source` plus `ip nat enable`. |
| Verification difference | Some platforms support NVI-specific show commands. If not, use the regular NAT translation and statistics commands. |
# NAT_NVI_Domainless_NAT_Model_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm interfaces are up before NAT work | NAT-RTR | `show ip interface brief` | All participating interfaces show up/up |
| 2 | Confirm routing between local and remote networks | NAT-RTR | `show ip route` | Router has routes toward local networks, global networks, and test destinations |
| 3 | Confirm no classic NAT role conflict exists | NAT-RTR | `show running-config | include ip nat inside|ip nat outside|ip nat enable|ip nat source` | Existing NAT model is visible before changes |
| 4 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters configuration mode |
| 5 | Enter the first participating interface | NAT-RTR | `interface <INTERFACE_1>` | Device enters interface configuration mode |
| 6 | Enable NVI NAT on the first participating interface | NAT-RTR | `ip nat enable` | Interface becomes eligible for NVI translation |
| 7 | Enter the second participating interface | NAT-RTR | `interface <INTERFACE_2>` | Device enters interface configuration mode |
| 8 | Enable NVI NAT on the second participating interface | NAT-RTR | `ip nat enable` | Interface becomes eligible for NVI translation |
| 9 | Optional: enable NVI NAT on additional routed interfaces | NAT-RTR | `interface <INTERFACE_N>` then `ip nat enable` | Additional path is eligible for NVI translation |
| 10 | Return to global configuration mode | NAT-RTR | `exit` | Device returns to global configuration mode |
| 11 | Create a standard ACL matching local addresses for dynamic NVI NAT | NAT-RTR | `access-list <ACL_ID> permit <LOCAL_SUBNET> <WILDCARD_MASK>` | ACL matches local source addresses that should be translated |
| 12 | Optional: create a NAT pool for dynamic NVI NAT | NAT-RTR | `ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>` | Pool is available for dynamic NVI translations |
| 13 | Configure dynamic NVI NAT using a pool | NAT-RTR | `ip nat source list <ACL_ID> pool <POOL_NAME>` | Matching local sources translate to addresses from the pool |
| 14 | Optional: configure dynamic NVI PAT using a pool | NAT-RTR | `ip nat source list <ACL_ID> pool <POOL_NAME> overload` | Matching local sources share the pool using port overload |
| 15 | Optional: configure NVI PAT using an interface address | NAT-RTR | `ip nat source list <ACL_ID> interface <GLOBAL_INTERFACE> overload` | Matching local sources share the interface address using port overload |
| 16 | Optional: configure static NVI one-to-one NAT | NAT-RTR | `ip nat source static <LOCAL_IP> <GLOBAL_IP>` | Local host has a permanent global mapping |
| 17 | Optional: configure static NVI TCP PAT | NAT-RTR | `ip nat source static tcp <LOCAL_IP> <LOCAL_TCP_PORT> <GLOBAL_IP> <GLOBAL_TCP_PORT> extendable` | Local TCP service has a permanent global IP and port mapping |
| 18 | Optional: configure static NVI UDP PAT | NAT-RTR | `ip nat source static udp <LOCAL_IP> <LOCAL_UDP_PORT> <GLOBAL_IP> <GLOBAL_UDP_PORT> extendable` | Local UDP service has a permanent global IP and port mapping |
| 19 | Exit configuration mode | NAT-RTR | `end` | Device returns to privileged EXEC mode |
| 20 | Verify NVI interface enablement | NAT-RTR | `show running-config | include ip nat enable|ip nat source` | Interfaces show `ip nat enable`; NAT rules show `ip nat source` |
| 21 | Verify NAT statistics | NAT-RTR | `show ip nat statistics` | NAT counters and configured rules are visible |
| 22 | Verify NVI translations if supported | NAT-RTR | `show ip nat nvi translations` | Active NVI translations appear if the platform supports this command |
| 23 | Verify translations with generic NAT command | NAT-RTR | `show ip nat translations` | Active translations appear after matching traffic |
| 24 | Generate outbound dynamic or PAT test traffic | Local Host | `ping <REMOTE_DESTINATION_IP>` | Matching local source traffic triggers a dynamic or PAT translation |
| 25 | Generate static PAT service test traffic | Remote Host | `telnet <GLOBAL_IP> <GLOBAL_TCP_PORT>` | Remote session reaches the mapped local TCP service |
| 26 | Recheck translation table after test traffic | NAT-RTR | `show ip nat translations verbose` | Translation details match the configured NVI rule |
| 27 | Save the working configuration | NAT-RTR | `write memory` | NVI NAT configuration survives reload |
# NAT_NVI_Domainless_NAT_Model_Skeleton
conf t
!
interface <INTERFACE_1>
 description NVI_PARTICIPATING_INTERFACE_1
 ip address <INTERFACE_1_IP> <INTERFACE_1_MASK>
 ip nat enable
 no shutdown
!
interface <INTERFACE_2>
 description NVI_PARTICIPATING_INTERFACE_2
 ip address <INTERFACE_2_IP> <INTERFACE_2_MASK>
 ip nat enable
 no shutdown
!
! Dynamic NVI NAT source selector
access-list <ACL_ID> permit <LOCAL_SUBNET> <WILDCARD_MASK>
!
! Option A: Dynamic NVI NAT using a pool without overload
ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
ip nat source list <ACL_ID> pool <POOL_NAME>
!
! Option B: NVI PAT overload using a pool
! ip nat source list <ACL_ID> pool <POOL_NAME> overload
!
! Option C: NVI PAT overload using an interface address
! ip nat source list <ACL_ID> interface <GLOBAL_INTERFACE> overload
!
! Option D: Static NVI one-to-one NAT
! ip nat source static <LOCAL_IP> <GLOBAL_IP>
!
! Option E: Static NVI TCP PAT
! ip nat source static tcp <LOCAL_IP> <LOCAL_TCP_PORT> <GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
!
! Option F: Static NVI UDP PAT
! ip nat source static udp <LOCAL_IP> <LOCAL_UDP_PORT> <GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
!
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>
!
end
write memory
# NAT_NVI_Domainless_NAT_Model_Dynamic_NAT_Skeleton
conf t
!
interface <LOCAL_SIDE_INTERFACE>
 ip nat enable
!
interface <REMOTE_SIDE_INTERFACE>
 ip nat enable
!
access-list <ACL_ID> permit <LOCAL_SUBNET> <WILDCARD_MASK>
ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
ip nat source list <ACL_ID> pool <POOL_NAME>
!
end
write memory
# NAT_NVI_Domainless_NAT_Model_PAT_Overload_Skeleton
conf t
!
interface <LOCAL_SIDE_INTERFACE>
 ip nat enable
!
interface <REMOTE_SIDE_INTERFACE>
 ip nat enable
!
access-list <ACL_ID> permit <LOCAL_SUBNET> <WILDCARD_MASK>
ip nat source list <ACL_ID> interface <GLOBAL_INTERFACE> overload
!
end
write memory
# NAT_NVI_Domainless_NAT_Model_Static_NAT_Skeleton
conf t
!
interface <LOCAL_SIDE_INTERFACE>
 ip nat enable
!
interface <REMOTE_SIDE_INTERFACE>
 ip nat enable
!
ip nat source static <LOCAL_IP> <GLOBAL_IP>
!
end
write memory
# NAT_NVI_Domainless_NAT_Model_Static_PAT_Skeleton
conf t
!
interface <LOCAL_SIDE_INTERFACE>
 ip nat enable
!
interface <REMOTE_SIDE_INTERFACE>
 ip nat enable
!
ip nat source static tcp <LOCAL_IP> <LOCAL_TCP_PORT> <GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
ip nat source static udp <LOCAL_IP> <LOCAL_UDP_PORT> <GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
!
end
write memory
# NAT_NVI_Domainless_NAT_Model_Verification_Commands
show ip interface brief
show running-config | include ip nat enable
show running-config | include ip nat source
show running-config | include ip nat pool
show running-config | include access-list
show access-lists
show ip route
show ip route <REMOTE_DESTINATION_IP>
show ip route <GLOBAL_IP>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
show ip nat nvi translations
show ip nat nvi statistics
ping <REMOTE_DESTINATION_IP>
telnet <GLOBAL_IP> <GLOBAL_TCP_PORT>
clear ip nat translation *
show ip nat translations
# NAT_NVI_Domainless_NAT_Model_Rollback
conf t
!
no ip nat source list <ACL_ID> pool <POOL_NAME>
no ip nat source list <ACL_ID> pool <POOL_NAME> overload
no ip nat source list <ACL_ID> interface <GLOBAL_INTERFACE> overload
no ip nat source static <LOCAL_IP> <GLOBAL_IP>
no ip nat source static tcp <LOCAL_IP> <LOCAL_TCP_PORT> <GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
no ip nat source static udp <LOCAL_IP> <LOCAL_UDP_PORT> <GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
no ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
no access-list <ACL_ID>
!
interface <INTERFACE_1>
 no ip nat enable
!
interface <INTERFACE_2>
 no ip nat enable
!
interface <INTERFACE_N>
 no ip nat enable
!
end
clear ip nat translation *
write memory
# NAT_NVI_Domainless_NAT_Model_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| NVI translations do not appear | `ip nat enable` is missing on a participating interface | `show running-config | include ip nat enable` | Add `ip nat enable` to every interface that should participate in NVI NAT |
| Router accepts classic NAT but NVI lab fails | Classic syntax was used instead of NVI syntax | `show running-config | include ip nat inside source|ip nat source` | Replace `ip nat inside source` with `ip nat source` for the NVI lab |
| Dynamic NAT ACL has no hits | ACL does not match the actual local source address | `show access-lists <ACL_ID>` | Correct the ACL source network and wildcard mask |
| PAT overload does not create translations | Traffic is not routed through the NAT router or does not cross NVI-enabled interfaces | `show ip route <REMOTE_DESTINATION_IP>` and `show running-config | include ip nat enable` | Fix routing and enable NVI NAT on the traffic path |
| Static NAT entry exists but inbound traffic fails | Remote side does not route the global address toward the NAT router | `show ip route <GLOBAL_IP>` on upstream path | Add a route for the global address or use a global address owned by the NAT router path |
| Static PAT fails | Wrong protocol or port was configured | `show ip nat translations verbose` | Match TCP versus UDP and verify local/global port numbers |
| `show ip nat nvi translations` fails | Platform or image does not expose NVI-specific show command | `show ip nat ?` | Use `show ip nat translations` and `show ip nat statistics` |
| Translations remain after rule changes | Existing dynamic NAT state survived the config edit | `show ip nat translations verbose` | Use `clear ip nat translation *` during maintenance |
| NVI and classic NAT outputs look mixed | Old `ip nat inside/outside` commands were left behind | `show running-config | include ip nat inside|ip nat outside|ip nat enable` | Remove classic interface roles from the NVI lab unless intentionally testing both models |
| Ping works but application test fails | ICMP translation worked, but the application port or static PAT rule is wrong | `show ip nat translations verbose` during app test | Verify the actual TCP or UDP service and correct the static PAT mapping |
| Pool NAT fails after several hosts | Pool is exhausted because overload was not configured | `show ip nat statistics` | Add `overload` or expand the NAT pool |
| Return traffic fails | Routing, firewalling, or host gateway is wrong after translation | `show ip route`, `show arp`, host default gateway check | Fix return route, host gateway, or firewall policy |
##### Source_Basis
# NAT_NVI_Domainless_NAT_Model_Mental_Model
# NAT_NVI_Domainless_NAT_Model_Configuration_Checklist
# NAT_NVI_Domainless_NAT_Model_Skeleton
# NAT_NVI_Domainless_NAT_Model_Dynamic_NAT_Skeleton
# NAT_NVI_Domainless_NAT_Model_PAT_Overload_Skeleton
# NAT_NVI_Domainless_NAT_Model_Static_NAT_Skeleton
# NAT_NVI_Domainless_NAT_Model_Static_PAT_Skeleton
# NAT_NVI_Domainless_NAT_Model_Verification_Commands
# NAT_NVI_Domainless_NAT_Model_Rollback
# NAT_NVI_Domainless_NAT_Model_Failure_Checks
