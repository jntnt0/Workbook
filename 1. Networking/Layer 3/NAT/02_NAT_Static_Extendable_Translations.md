# NAT_Static_Extendable_Translations_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Static NAT | A permanent mapping between an inside local address and an inside global address. The mapping exists even before user traffic creates a dynamic entry. |
| Inside local | The real internal host address before NAT. Example: `10.10.10.10`. |
| Inside global | The translated address that outside devices use to reach the inside host. Example: `203.0.113.10`. |
| Extendable static NAT | Allows the router to keep multiple static NAT entries that would otherwise look ambiguous or conflicting. |
| Why `extendable` matters | Without enough unique tuple detail, IOS may reject overlapping static mappings because the same local or global address is already mapped. |
| Common extendable use case | One inside host is reachable through more than one public address, often for multi-ISP or policy-specific reachability. |
| Static PAT use case | Multiple inside services can share one public IP when TCP or UDP ports make each mapping unique. |
| Translation tuple | Static NAT uniqueness can be based on local IP, global IP, protocol, local port, and global port. `extendable` tells IOS the static entry can be extended beyond a simple one-to-one host mapping. |
| Not a dynamic NAT feature | `extendable` belongs to static NAT or static PAT logic. It is not used to allocate addresses from a pool. |
| Routing still matters | NAT creates the address rewrite. Routing still has to deliver traffic to the NAT router and then to the real host. |
# NAT_Static_Extendable_Translations_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm inside and outside interfaces are up | NAT-RTR | `show ip interface brief` | Inside and outside interfaces show up/up |
| 2 | Confirm current NAT interface role config | NAT-RTR | `show running-config | section interface` | Intended LAN interface has `ip nat inside`; intended WAN interface has `ip nat outside` |
| 3 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters configuration mode |
| 4 | Mark the LAN-facing interface as inside | NAT-RTR | `interface <INSIDE_INTERFACE>` then `ip nat inside` | Inside interface is classified for classic NAT |
| 5 | Mark the WAN-facing interface as outside | NAT-RTR | `interface <OUTSIDE_INTERFACE>` then `ip nat outside` | Outside interface is classified for classic NAT |
| 6 | Return to global configuration mode | NAT-RTR | `exit` | Device returns to global configuration mode |
| 7 | Configure the first one-to-one static NAT mapping | NAT-RTR | `ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP_1> extendable` | Inside host is statically reachable by the first global address |
| 8 | Configure the second static mapping for the same inside host if required | NAT-RTR | `ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP_2> extendable` | IOS accepts another static mapping for the same inside local host |
| 9 | Optional: configure static TCP port translation | NAT-RTR | `ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> extendable` | TCP service is reachable through the configured global IP and port |
| 10 | Optional: configure static UDP port translation | NAT-RTR | `ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> extendable` | UDP service is reachable through the configured global IP and port |
| 11 | Confirm static NAT entries exist | NAT-RTR | `show ip nat translations` | Static entries appear with the expected inside local and inside global addresses |
| 12 | Confirm NAT interface roles and counters | NAT-RTR | `show ip nat statistics` | Inside and outside interfaces are listed correctly |
| 13 | Confirm routing toward the inside real host | NAT-RTR | `show ip route <INSIDE_LOCAL_IP>` | NAT router has a valid route toward the inside host |
| 14 | Confirm routing toward outside clients or upstream gateway | NAT-RTR | `show ip route <OUTSIDE_CLIENT_IP>` | NAT router has a valid route toward the outside path |
| 15 | Test inbound access to the static global address | Outside Host | `ping <INSIDE_GLOBAL_IP>` | Traffic reaches the inside host if ICMP is allowed and routed |
| 16 | Test inbound TCP static PAT if configured | Outside Host | `telnet <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT>` | TCP session reaches the inside service through the static port mapping |
| 17 | Verify active translation behavior after testing | NAT-RTR | `show ip nat translations verbose` | Translation output shows the expected static mapping and session detail |
| 18 | Save the working configuration | NAT-RTR | `write memory` | Static NAT configuration survives reload |
# NAT_Static_Extendable_Translations_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 description INSIDE_LAN
 ip address <INSIDE_GATEWAY_IP> <INSIDE_MASK>
 ip nat inside
 no shutdown
!
interface <OUTSIDE_INTERFACE>
 description OUTSIDE_WAN
 ip address <OUTSIDE_IP> <OUTSIDE_MASK>
 ip nat outside
 no shutdown
!
! One inside host mapped to one public/global address
ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP_1> extendable
!
! Optional: same inside host mapped to a second public/global address
ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP_2> extendable
!
! Optional: static TCP service mapping
ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
!
! Optional: static UDP service mapping
ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
!
ip route 0.0.0.0 0.0.0.0 <OUTSIDE_NEXT_HOP>
!
end
write memory
# NAT_Static_Extendable_Translations_Verification_Commands
show ip interface brief
show running-config | section interface
show running-config | include ip nat
show ip nat statistics
show ip nat translations
show ip nat translations verbose
show ip route <INSIDE_LOCAL_IP>
show ip route <INSIDE_GLOBAL_IP>
show ip route <OUTSIDE_CLIENT_IP>
show arp
ping <INSIDE_GLOBAL_IP>
telnet <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT>
clear ip nat translation *
show ip nat translations
# NAT_Static_Extendable_Translations_Rollback
conf t
!
no ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP_1> extendable
no ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP_2> extendable
no ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
no ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
!
interface <INSIDE_INTERFACE>
 no ip nat inside
!
interface <OUTSIDE_INTERFACE>
 no ip nat outside
!
end
clear ip nat translation *
write memory
# NAT_Static_Extendable_Translations_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| IOS rejects the second static NAT entry | Same local or global address is already mapped without an extendable static entry | Attempted config error and `show running-config | include ip nat inside source static` | Add `extendable` to overlapping static entries or make the translation tuple unique |
| Static entry exists but outside clients cannot connect | Routing to the inside host or return path is broken | `show ip route <INSIDE_LOCAL_IP>` and `show ip route <OUTSIDE_CLIENT_IP>` | Fix inside route, default route, or upstream return routing |
| Static PAT does not work | Wrong protocol or port was configured | `show ip nat translations verbose` | Match TCP versus UDP correctly and verify local/global ports |
| Ping fails but TCP service works | ICMP is not the service being statically translated, or host firewall blocks ICMP | `show ip nat translations` and host firewall check | Test the actual mapped service or permit ICMP if required |
| Static NAT entry appears but no sessions pass | Interface roles are missing or reversed | `show ip nat statistics` and `show running-config | section interface` | Correct `ip nat inside` and `ip nat outside` placement |
| Inbound traffic reaches router but not the server | Inside host default gateway is wrong | Check server gateway and router ARP table with `show arp` | Point the server default gateway back toward the NAT router |
| Wrong inside host receives traffic | Static mapping points to the wrong inside local address | `show running-config | include ip nat inside source static` | Correct the inside local address in the static NAT statement |
| Multiple public IPs fail from one ISP path | Upstream does not route or ARP for the additional global address | Upstream route and ARP check | Add upstream route, proxy ARP support, or use an address actually routed to the NAT router |
| Old behavior remains after edits | Existing translation state remained after NAT changes | `show ip nat translations verbose` | Clear translations with `clear ip nat translation *` after maintenance changes |
| Dynamic PAT works but static NAT does not | Dynamic NAT was tested outbound, but inbound static NAT path is different | Compare `show ip nat translations` during outbound and inbound tests | Troubleshoot inbound routing, ACLs, host firewall, and static mapping separately |
##### Source_Basis
# NAT_Static_Extendable_Translations_Mental_Model
# NAT_Static_Extendable_Translations_Configuration_Checklist
# NAT_Static_Extendable_Translations_Skeleton
# NAT_Static_Extendable_Translations_Verification_Commands
# NAT_Static_Extendable_Translations_Rollback
# NAT_Static_Extendable_Translations_Failure_Checks