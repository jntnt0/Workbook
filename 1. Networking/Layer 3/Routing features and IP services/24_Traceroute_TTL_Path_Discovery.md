Traceroute_TTL_Path_Discovery.md
# Traceroute_TTL_Path_Discovery

# Traceroute_TTL_Path_Discovery_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Traceroute | Diagnostic tool used to discover the apparent Layer 3 path from a source to a destination |
| TTL | Time To Live is decremented by each router hop |
| TTL expiry | When TTL reaches 0, the router drops the packet and usually sends ICMP time-exceeded back to the source |
| Hop discovery | Traceroute sends probes with increasing TTL values to reveal each hop along the path |
| Final destination response | Final hop normally replies with destination reached behavior, such as ICMP port unreachable for IOS UDP traceroute |
| IOS traceroute | Cisco IOS traceroute normally uses UDP probes to high destination ports |
| Windows tracert | Windows `tracert` normally uses ICMP echo probes |
| Linux traceroute | Linux traceroute commonly uses UDP by default, with options for ICMP or TCP depending on tool version |
| Asterisk | `*` means the probe timed out or the return ICMP message was filtered, delayed, or lost |
| `!H` | Host unreachable response, usually meaning a router had no route or could not reach the final host |
| `!N` | Network unreachable response |
| `!P` | Protocol unreachable response |
| `!A` | Administratively prohibited response, usually filtering or policy |
| Source matters | A traceroute sourced from a loopback, SVI, or interface can follow a different path than a default sourced traceroute |
| Return path matters | Traceroute depends on ICMP replies getting back to the source; failure can be on the forward path or return path |
| ECMP behavior | Different probes may hash over different equal-cost paths depending on source, destination, port, and platform |
| Tunnel behavior | GRE, IPsec, DMVPN, VXLAN, and other overlays can hide underlay hops or reduce visible hop detail |
| MPLS behavior | MPLS cores may hide hops, show provider addresses, or depend on TTL propagation behavior |
| Filtering boundary | Firewalls, ACLs, CoPP, and host firewalls can block traceroute even when application traffic works |
| Troubleshooting rule | Use traceroute with source, numeric, probe, timeout, TTL, and port options to test the exact flow path |

# Traceroute_TTL_Path_Discovery_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the exact source and destination being tested | RTR/L3SW | `show ip interface brief` | Correct source interface or source IP is known |
| 2 | Confirm the destination route exists before tracing | RTR/L3SW | `show ip route <DESTINATION_IP>` | Router has a route to the destination |
| 3 | Confirm CEF path for the tested source and destination | RTR/L3SW | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Expected egress interface and next hop are known before traceroute |
| 4 | Confirm basic reachability with sourced ping | RTR/L3SW | `ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>` | Ping succeeds or gives a baseline failure before traceroute |
| 5 | Run a basic IOS traceroute | RTR/L3SW | `traceroute <DESTINATION_IP>` | Output shows hop-by-hop path or timeout points |
| 6 | Run a sourced IOS traceroute | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>` | Trace uses the intended source address |
| 7 | Run numeric traceroute to avoid DNS lookup delays | RTR/L3SW | `traceroute <DESTINATION_IP> numeric` | Output displays IP addresses without reverse DNS delay |
| 8 | Run traceroute with source and numeric display | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> numeric` | Output shows hop IPs for the exact sourced flow |
| 9 | Increase timeout when testing slow or congested paths | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> timeout <SECONDS>` | Probes wait longer before displaying `*` |
| 10 | Increase probe count when intermittent loss is suspected | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> probe <COUNT>` | More probes are sent per hop for better loss visibility |
| 11 | Limit TTL range when focusing on a specific part of the path | RTR/L3SW | `traceroute <DESTINATION_IP> ttl <MIN_TTL> <MAX_TTL>` | Trace starts and stops within the selected hop range |
| 12 | Change UDP destination port when a firewall or ECMP hash may treat default traceroute differently | RTR/L3SW | `traceroute <DESTINATION_IP> port <UDP_PORT>` | Trace probes use the selected UDP destination port |
| 13 | Combine source, probe, timeout, and port options for controlled testing | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> probe <COUNT> timeout <SECONDS> port <UDP_PORT> numeric` | Trace tests a controlled source and UDP probe behavior |
| 14 | Use extended traceroute when multiple options must be controlled interactively | RTR/L3SW | `traceroute` | IOS prompts for protocol, target, source, timeout, probe count, TTL range, and port |
| 15 | Capture the first failing hop | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> numeric` | Last responding hop and first nonresponding hop are identified |
| 16 | Check the route from the last responding hop if accessible | Last Hop RTR | `show ip route <DESTINATION_IP>` | Last responding router has or lacks a route to the destination |
| 17 | Check return route toward the traceroute source | Last Hop RTR / Remote RTR | `show ip route <SOURCE_IP>` | Device can route ICMP time-exceeded or unreachable messages back to source |
| 18 | Check ACLs that may block traceroute return messages | RTR/L3SW/FW | `show access-lists` | ICMP time-exceeded and unreachable are not blocked unintentionally |
| 19 | Permit ICMP time-exceeded for traceroute diagnostics if policy allows | RTR/L3SW | `conf t`<br>`ip access-list extended <ACL_NAME>`<br>` permit icmp any any time-exceeded`<br>`end` | Intermediate hop TTL-expired messages can return to the source |
| 20 | Permit ICMP unreachable if IOS UDP traceroute final-hop completion must work | RTR/L3SW | `conf t`<br>`ip access-list extended <ACL_NAME>`<br>` permit icmp any any unreachable`<br>`end` | Destination or final router can return unreachable responses |
| 21 | Permit UDP traceroute probes if an ACL blocks outbound probes | RTR/L3SW/FW | `conf t`<br>`ip access-list extended <ACL_NAME>`<br>` permit udp <SOURCE_SUBNET> <SOURCE_WILDCARD> host <DESTINATION_IP> range 33434 33534`<br>`end` | Default IOS or Linux UDP traceroute probe range is allowed where required |
| 22 | Test from a Windows client when endpoint path matters | Windows Client | `tracert <DESTINATION_IP>` | ICMP-based hop path is displayed from the client perspective |
| 23 | Test from a Linux client using default traceroute | Linux Client | `traceroute <DESTINATION_IP>` | Default Linux traceroute behavior tests the path from the client |
| 24 | Test from a Linux client using ICMP mode if UDP is filtered | Linux Client | `traceroute -I <DESTINATION_IP>` | ICMP-based path test is attempted |
| 25 | Test from a Linux client using TCP mode if ICMP and UDP are filtered | Linux Client | `traceroute -T -p <TCP_PORT> <DESTINATION_IP>` | TCP SYN traceroute tests a path closer to application behavior |
| 26 | Compare traceroute to actual application traffic | Client/RTR | `traceroute -T -p 443 <DESTINATION_IP>` | TCP path to application port is tested where supported |
| 27 | Compare reverse direction traceroute | Remote RTR/Host | `traceroute <SOURCE_IP>` | Reverse path is checked for asymmetry or filtering |
| 28 | Verify ECMP hash effect with CEF exact-route | RTR/L3SW | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Expected CEF path is compared against traceroute output |
| 29 | Test alternate source IPs when ECMP or policy routing exists | RTR/L3SW | `traceroute <DESTINATION_IP> source <ALT_SOURCE_IP>` | Different source may reveal a different path |
| 30 | Check PBR if traceroute does not match the routing table | RTR/L3SW | `show ip policy` | Ingress policy route maps are visible |
| 31 | Check NAT if traceroute crosses a NAT boundary | Edge RTR/FW | `show ip nat translations` | NAT behavior is known for source and destination |
| 32 | Check tunnel path behavior if overlay hops hide the underlay | Tunnel RTR | `show interfaces Tunnel<TUNNEL_ID>`<br>`show running-config interface Tunnel<TUNNEL_ID>` | Tunnel source, destination, TTL, MTU, and routing are known |
| 33 | Check MPLS TTL behavior if provider/core hops are hidden | PE/P RTR | `show mpls forwarding-table` | MPLS forwarding path and label behavior are checked where available |
| 34 | Disable broad debugging after any lab trace testing | RTR/L3SW | `undebug all` | Debugging is disabled |
| 35 | Save any intentional ACL or diagnostic config changes | RTR/L3SW | `copy running-config startup-config` | Permanent traceroute support policy survives reload |

# Traceroute_TTL_Path_Discovery_Skeleton

traceroute <DESTINATION_IP>

traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>

traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> numeric

traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> probe <COUNT> timeout <SECONDS>

traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> port <UDP_PORT>

traceroute <DESTINATION_IP> ttl <MIN_TTL> <MAX_TTL>

traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> probe <COUNT> timeout <SECONDS> port <UDP_PORT> numeric

# IOS_Extended_Traceroute_Skeleton

traceroute
Protocol [ip]:
Target IP address: <DESTINATION_IP>
Source address: <SOURCE_IP_OR_INTERFACE>
Numeric display [n]: y
Timeout in seconds [3]: <SECONDS>
Probe count [3]: <COUNT>
Minimum Time to Live [1]: <MIN_TTL>
Maximum Time to Live [30]: <MAX_TTL>
Port Number [33434]: <UDP_PORT>
Loose, Strict, Record, Timestamp, Verbose[none]: <OPTION_OR_ENTER>

# Traceroute_ACL_Support_Skeleton

conf t
!
ip access-list extended <ACL_NAME>
 remark Permit traceroute diagnostics where policy allows
 permit icmp any any time-exceeded
 permit icmp any any unreachable
 permit udp <SOURCE_SUBNET> <SOURCE_WILDCARD> host <DESTINATION_IP> range 33434 33534
!
end
write memory

# Windows_Traceroute_Skeleton

tracert <DESTINATION_IP>

tracert -d <DESTINATION_IP>

tracert -h <MAX_HOPS> <DESTINATION_IP>

tracert -w <TIMEOUT_MSEC> <DESTINATION_IP>

# Linux_Traceroute_Skeleton

traceroute <DESTINATION_IP>

traceroute -n <DESTINATION_IP>

traceroute -I <DESTINATION_IP>

traceroute -T -p <TCP_PORT> <DESTINATION_IP>

traceroute -s <SOURCE_IP> <DESTINATION_IP>

traceroute -m <MAX_TTL> <DESTINATION_IP>

# Traceroute_TTL_Path_Discovery_Verification_Commands

show ip interface brief
show ip route <DESTINATION_IP>
show ip route <SOURCE_IP>
show ip cef <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
show ip policy
show route-map
show access-lists
show interfaces <EGRESS_INTERFACE>
show ip nat translations
show ip nat statistics
show running-config interface <TUNNEL_INTERFACE>
show interfaces <TUNNEL_INTERFACE>
show mpls forwarding-table
ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>
traceroute <DESTINATION_IP>
traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>
traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> numeric
traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> probe <COUNT> timeout <SECONDS>
traceroute <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE> port <UDP_PORT>
traceroute <DESTINATION_IP> ttl <MIN_TTL> <MAX_TTL>
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
route print
tracert <DESTINATION_IP>
tracert -d <DESTINATION_IP>
ping <DESTINATION_IP>

ip addr
ip route
traceroute <DESTINATION_IP>
traceroute -n <DESTINATION_IP>
traceroute -I <DESTINATION_IP>
traceroute -T -p 443 <DESTINATION_IP>
ping <DESTINATION_IP>

# Traceroute_TTL_Path_Discovery_Rollback

conf t
!
ip access-list extended <ACL_NAME>
 no permit icmp any any time-exceeded
 no permit icmp any any unreachable
 no permit udp <SOURCE_SUBNET> <SOURCE_WILDCARD> host <DESTINATION_IP> range 33434 33534
!
end
write memory

# Traceroute_ACL_Full_Rollback

conf t
!
no ip access-list extended <ACL_NAME>
!
end
write memory

# Traceroute_Debug_Rollback

undebug all

# Traceroute_TTL_Path_Discovery_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Every hop shows `* * *` | No route, wrong source, filtering, or no return path | `show ip route <DESTINATION_IP>` and sourced ping | Fix route, source selection, ACL, or return routing |
| First hop responds, later hops all timeout | Downstream filtering or no return ICMP from later hops | `show access-lists` on downstream path | Permit ICMP time-exceeded where diagnostics are allowed |
| Final destination never completes but intermediate hops show | Final host or firewall blocks final response | Host firewall and ACL checks | Use TCP traceroute to the application port or permit required final response |
| IOS UDP traceroute fails but Windows tracert works | UDP probes are filtered while ICMP echo is allowed | Compare IOS traceroute and Windows `tracert` | Use ICMP or TCP mode from a host, or permit UDP traceroute probes |
| Windows tracert fails but IOS traceroute works | ICMP echo is filtered while UDP probes are allowed | Compare Windows `tracert` and IOS traceroute | Use the probe type that matches the real diagnostic need |
| Output shows `!H` | Router reports host unreachable | `show ip route <DESTINATION_IP>` on reporting router | Add or fix route to destination host or subnet |
| Output shows `!N` | Router reports network unreachable | `show ip route <DESTINATION_NETWORK>` on reporting router | Add or repair network route |
| Output shows `!A` | Administratively prohibited by ACL, firewall, or policy | `show access-lists` and firewall policy | Permit diagnostic traffic only where approved |
| Output alternates between different hops | ECMP load sharing sends probes across different paths | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Test with controlled source, port, and probe count |
| Traceroute path differs from routing table expectation | PBR, NAT, VRF, ECMP, or source selection changes forwarding | `show ip policy`, `show ip cef exact-route`, and `show ip route` | Test exact source and fix policy or route assumptions |
| Traceroute from router works but client fails | Client path, gateway, NAT, ACL, or return route differs | Client `route print` or `ip route` | Test from the real client and fix endpoint gateway or policy |
| Traceroute from one source succeeds but another fails | Source-specific routing, ACL, NAT, or PBR difference | Sourced traceroute and `show ip cef exact-route` | Fix source-specific policy or return path |
| DNS delay makes traceroute slow | Reverse DNS lookup is delaying hop display | `traceroute <DESTINATION_IP> numeric` | Use `numeric` or disable name resolution for the test |
| A single `*` appears among successful probes | One probe was dropped, delayed, or rate-limited | Repeat with `probe <COUNT>` | Treat isolated asterisks as possible rate limiting unless consistent |
| Traceroute stops at firewall | Firewall blocks TTL-expired or unreachable messages | Firewall policy and logs | Permit required ICMP return messages or use TCP traceroute |
| Traceroute does not show underlay tunnel hops | Overlay encapsulation hides underlay path | `show running-config interface Tunnel<TUNNEL_ID>` | Trace tunnel endpoints or underlay destination separately |
| GRE tunnel appears as one hop | Original packet TTL is decremented across overlay view, not every underlay hop | Tunnel interface config | Trace to tunnel destination to inspect underlay |
| MPLS provider core is hidden | MPLS TTL propagation or provider policy hides hops | PE/P router MPLS checks | Validate from provider edge or request provider path visibility |
| Traceroute fails across NAT | Return messages do not map cleanly or are filtered | NAT translations and firewall logs | Use inside source testing and permit required ICMP through NAT/firewall |
| TCP application works but traceroute fails | Diagnostic probes are filtered while application port is allowed | Application test and traceroute comparison | Use TCP traceroute to the real application port |
| Traceroute works but application fails | Path exists, but application port, server, DNS, or firewall is broken | TCP connect, DNS, and firewall tests | Troubleshoot the application flow, not only routing |
| Hop latency looks high at one hop but later hops are normal | Router rate-limits TTL-expired replies to its control plane | Compare later hop latency | Do not treat one high intermediate response as path latency unless it continues downstream |
| Final hop latency is high | Actual destination path or host response is slow | Repeated traceroute and ping | Investigate congestion, QoS, server load, or path delay |
| Traceroute loops between two hops | Routing loop | Repeated hop pattern in output | Fix routing loop, redistribution, static route, or summarization issue |
| Maximum TTL reached before destination | Path too long or loop exists | `traceroute <DESTINATION_IP> ttl 1 60` | Increase max TTL for testing, then fix loop if repeated hops appear |
| Debug output is too noisy | Packet debug or related debug left enabled | `show debugging` | Run `undebug all` |
| ACL diagnostic permit remains after test | Temporary traceroute ACL exception was not removed | `show access-lists` | Remove temporary entries or document permanent diagnostic policy |

# Index

Traceroute_TTL_Path_Discovery.md
Traceroute_TTL_Path_Discovery
Traceroute_TTL_Path_Discovery_Mental_Model
Traceroute_TTL_Path_Discovery_Configuration_Checklist
Traceroute_TTL_Path_Discovery_Skeleton
IOS_Extended_Traceroute_Skeleton
Traceroute_ACL_Support_Skeleton
Windows_Traceroute_Skeleton
Linux_Traceroute_Skeleton
Traceroute_TTL_Path_Discovery_Verification_Commands
Client_Verification_Commands
Traceroute_TTL_Path_Discovery_Rollback
Traceroute_ACL_Full_Rollback
Traceroute_Debug_Rollback
Traceroute_TTL_Path_Discovery_Failure_Checks

