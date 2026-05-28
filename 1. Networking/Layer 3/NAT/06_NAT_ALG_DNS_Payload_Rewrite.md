Local source checked: _KB_INDEX.md points IOS XE NAT to iosxe_combined_pdfs_.md. The local NAT source exposes the no-payload static NAT option and shows no ip nat service dns tcp / no ip nat service dns udp in IOS XE config output. Cisco’s NAT ALG docs confirm DNS ALG processing control, and Cisco’s command reference confirms ip nat service dns-reset-ttl for DNS RR TTL behavior.  

NAT_ALG_DNS_Payload_Rewrite.md

NAT_ALG_DNS_Payload_Rewrite

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `iosxe_combined_pdfs_.md` NAT guide reference | Points IOS XE NAT configuration content to the IOS XE combined source |
| `iosxe_combined_pdfs_.md` | `NAT Configuration Guide` > `Application-Level Gateways with NAT` | Establishes that ALG is part of IOS XE NAT behavior |
| `iosxe_combined_pdfs_.md` | NAT static syntax with `[no-payload]` | Supports header-only NAT behavior where payload address translation should be disabled |
| `iosxe_combined_pdfs_.md` | IOS XE NAT service config examples | Shows `no ip nat service dns tcp` and `no ip nat service dns udp` as DNS ALG disablement syntax |
| Cisco IOS XE NAT ALG documentation | `Using Application-Level Gateways with NAT` | Supports DNS ALG control and protocol-aware payload handling |
| Cisco IOS IP Addressing Services Command Reference | `ip nat service dns-reset-ttl` | Supports DNS resource record TTL reset behavior control |
| Related lab | `nat-alg-dns-final` | Lab target for DNS ALG payload rewrite behavior |
| Related lab | `nat-alg-dns2-final` | Second lab target for DNS ALG payload rewrite behavior |
# NAT_ALG_DNS_Payload_Rewrite_Mental_Model
| Concept | Operational Meaning |
|---|---|
| NAT header rewrite | Normal NAT changes the packet IP header and sometimes the transport header. |
| NAT payload rewrite | DNS ALG can inspect DNS payloads and rewrite embedded IP addresses inside DNS answers. |
| DNS ALG | DNS Application-Level Gateway logic makes DNS answers line up with the translated address view seen by the querying side. |
| Why DNS is special | A DNS reply can contain an A record that exposes an address different from the address view the client should use. |
| Static NAT with payload translation | A static NAT mapping can allow the router to rewrite matching DNS payload addresses. |
| `no-payload` | The `no-payload` keyword tells NAT to translate headers only and leave embedded payload addresses unchanged. |
| DNS ALG enablement | `ip nat service dns tcp` and `ip nat service dns udp` enable DNS ALG behavior for DNS over TCP or UDP when supported by the image. |
| DNS ALG disablement | `no ip nat service dns tcp` and `no ip nat service dns udp` disable DNS ALG behavior for TCP or UDP DNS traffic. |
| TTL behavior | `ip nat service dns-reset-ttl` can reset DNS resource record TTL values in NAT-processed DNS replies. |
| Testing principle | Do not validate DNS ALG with ping only. Validate the DNS answer itself using `nslookup` or `dig`, then validate traffic to the returned address. |
| Common trap | NAT translations can look correct while DNS answers are wrong because the issue is payload rewrite, not basic header translation. |
# NAT_ALG_DNS_Payload_Rewrite_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NAT router interfaces are up | NAT-RTR | `show ip interface brief` | Inside, outside, and DNS-relevant interfaces are up/up |
| 2 | Confirm existing NAT interface roles | NAT-RTR | `show running-config | section interface` | Inside-facing interface has `ip nat inside`; outside-facing interface has `ip nat outside` |
| 3 | Confirm existing DNS ALG service state | NAT-RTR | `show running-config | include ip nat service dns` | Output shows whether DNS ALG is explicitly enabled or disabled |
| 4 | Confirm existing static NAT payload behavior | NAT-RTR | `show running-config | include ip nat inside source static|no-payload` | Static NAT entries and any `no-payload` keywords are visible |
| 5 | Confirm routing toward the inside server | NAT-RTR | `show ip route <INSIDE_SERVER_IP>` | NAT router has a valid route toward the inside DNS-published server |
| 6 | Confirm routing toward the DNS client | NAT-RTR | `show ip route <DNS_CLIENT_IP>` | NAT router has a valid return path toward the client |
| 7 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters configuration mode |
| 8 | Configure the inside interface role | NAT-RTR | `interface <INSIDE_INTERFACE>` then `ip nat inside` | Server-side interface is marked as NAT inside |
| 9 | Configure the outside interface role | NAT-RTR | `interface <OUTSIDE_INTERFACE>` then `ip nat outside` | Client-side or upstream interface is marked as NAT outside |
| 10 | Return to global configuration mode | NAT-RTR | `exit` | Device returns to global configuration mode |
| 11 | Configure static NAT for the DNS-published inside host with payload rewrite allowed | NAT-RTR | `ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP>` | Static NAT mapping exists and DNS payload rewrite is not suppressed |
| 12 | Optional: configure static TCP PAT with payload rewrite allowed | NAT-RTR | `ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> extendable` | Static TCP service mapping exists and payload rewrite is not suppressed |
| 13 | Optional: configure static UDP PAT with payload rewrite allowed | NAT-RTR | `ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> extendable` | Static UDP service mapping exists and payload rewrite is not suppressed |
| 14 | Enable DNS ALG for UDP if the lab requires payload rewrite and the image supports the command | NAT-RTR | `ip nat service dns udp` | DNS ALG is enabled for normal UDP DNS queries |
| 15 | Enable DNS ALG for TCP if the lab requires TCP DNS payload rewrite and the image supports the command | NAT-RTR | `ip nat service dns tcp` | DNS ALG is enabled for TCP DNS queries and large DNS responses |
| 16 | Optional: preserve DNS TTL behavior instead of resetting TTL | NAT-RTR | `no ip nat service dns-reset-ttl` | DNS RR TTL values are not forced to zero by NAT behavior |
| 17 | Optional: force DNS TTL reset behavior if the lab specifically requires it | NAT-RTR | `ip nat service dns-reset-ttl` | DNS RR TTL reset behavior is enabled |
| 18 | Exit configuration mode | NAT-RTR | `end` | Device returns to privileged EXEC mode |
| 19 | Clear old NAT state before testing | NAT-RTR | `clear ip nat translation *` | Stale translations are removed |
| 20 | Verify DNS ALG service state after configuration | NAT-RTR | `show running-config | include ip nat service dns` | DNS ALG TCP and UDP state matches the lab requirement |
| 21 | Verify static NAT mapping | NAT-RTR | `show ip nat translations` | Static inside local and inside global mapping appears |
| 22 | Query DNS from the client side | DNS Client | `nslookup <FQDN> <DNS_SERVER_IP>` | DNS answer returns the address expected from the client side of NAT |
| 23 | Optional: test with DNS over TCP if required | DNS Client | `dig +tcp <FQDN> @<DNS_SERVER_IP>` | TCP DNS answer returns the expected translated or untranslated address |
| 24 | Confirm NAT saw the DNS flow | NAT-RTR | `show ip nat translations verbose` | NAT table shows DNS client and server flow detail |
| 25 | Test traffic to the address returned by DNS | DNS Client | `ping <DNS_RETURNED_IP>` | Client reaches the DNS-returned address if ICMP and routing allow it |
| 26 | Test the actual application service if static PAT is involved | DNS Client | `telnet <DNS_RETURNED_IP> <GLOBAL_TCP_PORT>` | Client reaches the translated service |
| 27 | Save the working configuration | NAT-RTR | `write memory` | DNS ALG NAT configuration survives reload |
# NAT_ALG_DNS_Payload_Rewrite_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 description INSIDE_SERVER_SIDE
 ip address <INSIDE_INTERFACE_IP> <INSIDE_MASK>
 ip nat inside
 no shutdown
!
interface <OUTSIDE_INTERFACE>
 description OUTSIDE_CLIENT_OR_UPSTREAM_SIDE
 ip address <OUTSIDE_INTERFACE_IP> <OUTSIDE_MASK>
 ip nat outside
 no shutdown
!
! Static NAT where DNS payload rewrite is allowed.
ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP>
!
! Optional static service mappings where payload rewrite is allowed.
ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
!
! Enable DNS ALG for DNS payload processing if supported and required.
ip nat service dns udp
ip nat service dns tcp
!
! Optional TTL behavior.
no ip nat service dns-reset-ttl
!
ip route 0.0.0.0 0.0.0.0 <OUTSIDE_NEXT_HOP>
!
end
write memory
# NAT_ALG_DNS_Payload_Rewrite_No_Payload_Skeleton
conf t
!
interface <INSIDE_INTERFACE>
 ip nat inside
!
interface <OUTSIDE_INTERFACE>
 ip nat outside
!
! Header-only static NAT. Embedded addresses in DNS payload are left untouched.
ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP> no-payload
!
! Header-only static TCP PAT.
ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> no-payload
!
! Header-only static UDP PAT.
ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> no-payload
!
end
write memory
# NAT_ALG_DNS_Payload_Rewrite_Disable_DNS_ALG_Skeleton
conf t
!
no ip nat service dns udp
no ip nat service dns tcp
!
end
clear ip nat translation *
write memory
# NAT_ALG_DNS_Payload_Rewrite_Verification_Commands
show ip interface brief
show running-config | section interface
show running-config | include ip nat
show running-config | include ip nat service dns
show running-config | include no-payload
show ip route <INSIDE_SERVER_IP>
show ip route <DNS_CLIENT_IP>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
show access-lists
clear ip nat translation *
nslookup <FQDN> <DNS_SERVER_IP>
dig <FQDN> @<DNS_SERVER_IP>
dig +tcp <FQDN> @<DNS_SERVER_IP>
ping <DNS_RETURNED_IP>
telnet <DNS_RETURNED_IP> <GLOBAL_TCP_PORT>
# NAT_ALG_DNS_Payload_Rewrite_Rollback
conf t
!
no ip nat service dns udp
no ip nat service dns tcp
no ip nat service dns-reset-ttl
!
no ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP>
no ip nat inside source static <INSIDE_LOCAL_IP> <INSIDE_GLOBAL_IP> no-payload
no ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> extendable
no ip nat inside source static tcp <INSIDE_LOCAL_IP> <LOCAL_TCP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_TCP_PORT> no-payload
no ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> extendable
no ip nat inside source static udp <INSIDE_LOCAL_IP> <LOCAL_UDP_PORT> <INSIDE_GLOBAL_IP> <GLOBAL_UDP_PORT> no-payload
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
# NAT_ALG_DNS_Payload_Rewrite_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| DNS answer shows the untranslated inside address | DNS ALG is disabled or static NAT uses `no-payload` | `show running-config | include ip nat service dns|no-payload` | Enable DNS ALG or remove `no-payload` from the relevant static NAT rule |
| DNS answer shows the translated address when it should not | Payload rewrite is active but the design requires header-only NAT | `show running-config | include ip nat inside source static` | Add `no-payload` to the relevant static NAT rule |
| UDP DNS works but TCP DNS does not | DNS ALG is enabled for UDP only | `show running-config | include ip nat service dns` | Add `ip nat service dns tcp` if supported and required |
| TCP DNS works but UDP DNS does not | DNS ALG is enabled for TCP only or UDP DNS is blocked | `show running-config | include ip nat service dns` | Add `ip nat service dns udp` and verify ACL or firewall policy |
| NAT table looks correct but DNS answer is wrong | Header NAT is working but payload rewrite behavior is wrong | `show ip nat translations verbose` and `nslookup <FQDN> <DNS_SERVER_IP>` | Troubleshoot DNS ALG state and `no-payload` behavior instead of basic NAT |
| Static NAT works for direct IP but name-based access fails | DNS payload answer does not match the address clients must use | `nslookup <FQDN> <DNS_SERVER_IP>` | Fix DNS ALG behavior or correct the DNS zone record |
| DNS query times out | Routing, DNS server reachability, or ACL filtering is broken | `show ip route <DNS_SERVER_IP>` and `show ip nat translations` | Fix routing, NAT path, or packet filter before troubleshooting ALG |
| ALG command is rejected | IOS XE image or platform does not support that exact command | `ip nat service ?` | Use supported syntax on that image or validate with `no-payload` static NAT behavior |
| DNS TTL behavior is unexpected | `ip nat service dns-reset-ttl` state does not match the lab goal | `show running-config | include dns-reset-ttl` | Configure `ip nat service dns-reset-ttl` or `no ip nat service dns-reset-ttl` as required |
| Results stay wrong after config changes | Old NAT translations or DNS cache are still being used | `show ip nat translations` and client DNS cache check | Clear NAT translations and flush the DNS client cache |
| Ping to returned address fails | DNS rewrite is correct but ICMP or routing is broken | `show ip route <DNS_RETURNED_IP>` and `show ip nat translations` | Fix routing, host gateway, firewall, or test the real application port |
| Static PAT service fails after correct DNS answer | DNS resolves correctly but the service port mapping is wrong | `show running-config | include static tcp|static udp` | Correct protocol and local/global port values |
##### Source_Basis
# NAT_ALG_DNS_Payload_Rewrite_Mental_Model
# NAT_ALG_DNS_Payload_Rewrite_Configuration_Checklist
# NAT_ALG_DNS_Payload_Rewrite_Skeleton
# NAT_ALG_DNS_Payload_Rewrite_No_Payload_Skeleton
# NAT_ALG_DNS_Payload_Rewrite_Disable_DNS_ALG_Skeleton
# NAT_ALG_DNS_Payload_Rewrite_Verification_Commands
# NAT_ALG_DNS_Payload_Rewrite_Rollback
# NAT_ALG_DNS_Payload_Rewrite_Failure_Checks
