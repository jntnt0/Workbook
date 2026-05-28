Proxy_ARP_Layer2_Reachability_Workaround.md
# Proxy_ARP_Layer2_Reachability_Workaround

# Proxy_ARP_Layer2_Reachability_Workaround_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Proxy ARP | Router answers an ARP request for an off-subnet IP address using its own MAC address |
| Layer 2 workaround | Proxy ARP makes a Layer 3 reachability problem look like a Layer 2 adjacency |
| Default behavior | Proxy ARP is commonly enabled by default on Cisco routed interfaces |
| Route dependency | Router only answers proxy ARP if it has a route to the requested destination |
| Host mask mistake | Proxy ARP can hide hosts with an overly broad subnet mask or missing default gateway |
| Ethernet static route risk | Static routes pointing only to an Ethernet exit interface can force ARP for many destinations and rely on proxy ARP behavior |
| ARP cache clue | Many different destination IPs resolving to the same router MAC is a proxy ARP indicator |
| Encapsulation failure | If proxy ARP is disabled and no real Layer 2 destination exists, the device cannot build the frame |
| Security concern | Proxy ARP can allow unintended reachability and can make segmentation mistakes harder to see |
| Preferred fix | Correct host masks, default gateways, routing, and static route next hops instead of relying on proxy ARP |
| Local proxy ARP | Local proxy ARP is separate behavior where the router answers ARP for hosts on the same interface to force traffic through itself |
| Troubleshooting rule | Verify `show ip interface`, ARP entries, routes, and actual host addressing before enabling proxy ARP as a fix |

# Proxy_ARP_Layer2_Reachability_Workaround_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the client-facing routed interface is up | RTR/L3SW | `show ip interface brief` | Interface facing the ARP-requesting host is `up/up` |
| 2 | Confirm Layer 3 forwarding is enabled on a multilayer switch | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 3 | Confirm the router has a route to the destination being ARPed for | RTR/L3SW | `show ip route <DESTINATION_IP>` | Routing table has a valid route to the destination |
| 4 | Confirm the router has a route back to the source host | RTR/L3SW | `show ip route <SOURCE_HOST_IP>` | Router can return traffic toward the ARP-requesting host |
| 5 | Confirm current proxy ARP state on the interface | RTR/L3SW | `show ip interface <CLIENT_FACING_INTERFACE>` | Output shows whether `Proxy ARP is enabled` or disabled |
| 6 | Confirm current local proxy ARP state on the interface | RTR/L3SW | `show ip interface <CLIENT_FACING_INTERFACE>` | Output shows whether `Local Proxy ARP is enabled` or disabled |
| 7 | Confirm current ARP table before testing | RTR/L3SW | `show ip arp` | Baseline ARP entries are visible |
| 8 | Confirm the host is ARPing for an off-subnet or unintended destination | Client/RTR | `show ip arp` or client ARP table command | Destination IP appears as an ARP target instead of being sent to a default gateway |
| 9 | Enable proxy ARP on the client-facing interface if the design intentionally requires it | RTR/L3SW | `conf t`<br>`interface <CLIENT_FACING_INTERFACE>`<br>` ip proxy-arp`<br>`end` | Router is allowed to answer ARP requests for reachable off-subnet destinations |
| 10 | Keep proxy ARP disabled where segmentation or explicit gateway behavior is required | RTR/L3SW | `conf t`<br>`interface <CLIENT_FACING_INTERFACE>`<br>` no ip proxy-arp`<br>`end` | Router does not answer proxy ARP requests on that interface |
| 11 | Enable local proxy ARP only for a specific same-interface forwarding requirement | RTR/L3SW | `conf t`<br>`interface <CLIENT_FACING_INTERFACE>`<br>` ip local-proxy-arp`<br>`end` | Router may answer ARP requests for hosts on the same interface so traffic hairpins through the router |
| 12 | Disable local proxy ARP when same-subnet host-to-host traffic should not be forced through the router | RTR/L3SW | `conf t`<br>`interface <CLIENT_FACING_INTERFACE>`<br>` no ip local-proxy-arp`<br>`end` | Router stops answering local proxy ARP requests |
| 13 | Clear stale ARP entries before retesting | RTR/L3SW | `clear arp-cache` | Router refreshes ARP information during the next test |
| 14 | Clear stale ARP on the client if possible | Client | `arp -d *` | Client removes old MAC mappings |
| 15 | Generate traffic from the client to the destination | Client | `ping <DESTINATION_IP>` | Client triggers ARP and sends traffic if proxy ARP and routing are working |
| 16 | Verify the client learned the router MAC for the destination | Client | `arp -a` | Destination IP maps to the router interface MAC when proxy ARP is active |
| 17 | Verify router ARP table during test | RTR/L3SW | `show ip arp` | Router sees the client IP and MAC on the expected interface |
| 18 | Verify route lookup for the destination after ARP succeeds | RTR/L3SW | `show ip route <DESTINATION_IP>` | Router forwards the packet using a valid routed path |
| 19 | Verify CEF forwarding for the destination | RTR/L3SW | `show ip cef <DESTINATION_IP>` | CEF points to the expected next hop or outgoing interface |
| 20 | Verify packet path with traceroute | Client/RTR | `traceroute <DESTINATION_IP>` | Path shows traffic forwarding through the router |
| 21 | Check for suspicious many-to-one ARP mappings | Client/RTR | `show ip arp` or `arp -a` | Multiple off-subnet destinations resolving to the same router MAC indicate proxy ARP behavior |
| 22 | Correct the preferred design if proxy ARP is only masking bad addressing | Client/RTR | `! Correct host subnet mask, default gateway, or static route next hop` | Host sends off-subnet traffic to its gateway instead of ARPing for remote IPs |
| 23 | Replace Ethernet exit-interface static routes with next-hop static routes | RTR/L3SW | `conf t`<br>`no ip route <PREFIX> <MASK> <ETHERNET_INTERFACE>`<br>`ip route <PREFIX> <MASK> <NEXT_HOP_IP>`<br>`end` | Router resolves only the next hop instead of ARPing for every destination in the route |
| 24 | Verify proxy ARP is disabled after correcting the real design if it is no longer needed | RTR/L3SW | `show ip interface <CLIENT_FACING_INTERFACE>` | Output shows `Proxy ARP is disabled` if the workaround is removed |
| 25 | Save the intended configuration | RTR/L3SW | `copy running-config startup-config` | Proxy ARP state and corrected routing survive reload |

# Proxy_ARP_Layer2_Reachability_Workaround_Skeleton

conf t
!
ip routing
!
interface <CLIENT_FACING_INTERFACE>
 description CLIENT_SEGMENT_PROXY_ARP_TEST
 ip address <GATEWAY_IP> <MASK>
 ip proxy-arp
 no ip local-proxy-arp
 no shutdown
!
end
write memory

# Proxy_ARP_Disabled_Interface_Skeleton

conf t
!
interface <CLIENT_FACING_INTERFACE>
 description CLIENT_SEGMENT_EXPLICIT_GATEWAY_REQUIRED
 no ip proxy-arp
 no ip local-proxy-arp
!
end
write memory

# Local_Proxy_ARP_Skeleton

conf t
!
interface <CLIENT_FACING_INTERFACE>
 description SAME_INTERFACE_HOST_TRAFFIC_FORCED_THROUGH_ROUTER
 ip local-proxy-arp
!
end
write memory

# Ethernet_Static_Route_Correction_Skeleton

conf t
!
! Bad pattern on Ethernet multiaccess networks.
no ip route <PREFIX> <MASK> <ETHERNET_INTERFACE>
!
! Preferred pattern.
ip route <PREFIX> <MASK> <NEXT_HOP_IP>
!
end
write memory

# Proxy_ARP_Layer2_Reachability_Workaround_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config interface <CLIENT_FACING_INTERFACE>
show ip interface <CLIENT_FACING_INTERFACE>
show ip route <DESTINATION_IP>
show ip route <SOURCE_HOST_IP>
show ip cef <DESTINATION_IP>
show ip cef exact-route <SOURCE_HOST_IP> <DESTINATION_IP>
show ip arp
show ip arp <SOURCE_HOST_IP>
show ip arp <DESTINATION_IP>
show interfaces <CLIENT_FACING_INTERFACE>
show adjacency detail
ping <DESTINATION_IP>
ping <DESTINATION_IP> source <SOURCE_HOST_IP_OR_INTERFACE>
traceroute <DESTINATION_IP>
clear arp-cache
debug arp
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
arp -a
arp -d *
ping <DESTINATION_IP>
tracert <DESTINATION_IP>

# Proxy_ARP_Layer2_Reachability_Workaround_Rollback

conf t
!
interface <CLIENT_FACING_INTERFACE>
 no ip proxy-arp
 no ip local-proxy-arp
!
end
write memory

# Proxy_ARP_Reenable_Rollback

conf t
!
interface <CLIENT_FACING_INTERFACE>
 ip proxy-arp
!
end
write memory

# Ethernet_Static_Route_Correction_Rollback

conf t
!
no ip route <PREFIX> <MASK> <NEXT_HOP_IP>
ip route <PREFIX> <MASK> <ETHERNET_INTERFACE>
!
end
write memory

# Proxy_ARP_Layer2_Reachability_Workaround_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Client ARPs for remote destination IPs | Client subnet mask is too broad or default gateway is missing | Client `ipconfig /all` and `arp -a` | Correct client subnet mask and default gateway |
| Proxy ARP does not answer | Proxy ARP disabled on client-facing interface | `show ip interface <CLIENT_FACING_INTERFACE>` | Configure `ip proxy-arp` only if the workaround is intended |
| Proxy ARP answers but ping fails | Router lacks route to destination or return path is missing | `show ip route <DESTINATION_IP>` and remote `show ip route <SOURCE_HOST_IP>` | Fix routing in both directions |
| Router cannot build frame toward destination | Next-hop ARP or adjacency is unresolved | `show ip arp` and `show adjacency detail` | Fix next-hop Layer 2 reachability |
| Client ARP table maps many remote IPs to one MAC | Proxy ARP is masking a design issue | Client `arp -a` | Correct host gateway and subnet mask, then disable proxy ARP if not needed |
| Traffic works only when proxy ARP is enabled | Host or static route design depends on proxy ARP | Client `ipconfig /all` and router `show running-config | include ^ip route` | Replace workaround with explicit default gateway or next-hop route |
| Disabling proxy ARP breaks hosts | Hosts were relying on proxy ARP instead of a gateway | Client ARP table and gateway config | Fix endpoint addressing before disabling proxy ARP |
| Ethernet static route causes excessive ARP | Static route points only to Ethernet exit interface | `show running-config | include ^ip route` | Use next-hop static route or fully specified static route |
| ARP table fills with many destination entries | Router is ARPing for many destinations because of exit-interface static route behavior | `show ip arp` | Replace Ethernet exit-interface route with next-hop route |
| Proxy ARP appears enabled but client still fails | Client and router are not in the same VLAN or Layer 2 domain | `show vlan brief`, `show interfaces trunk`, `show ip arp` | Fix VLAN, trunk, access port, or cabling |
| Local proxy ARP does not work | `ip local-proxy-arp` is not enabled or platform behavior differs | `show ip interface <CLIENT_FACING_INTERFACE>` | Enable local proxy ARP only if same-interface hairpin forwarding is intended |
| Same-subnet hosts unexpectedly send traffic through router | Local proxy ARP is enabled | `show ip interface <CLIENT_FACING_INTERFACE>` | Configure `no ip local-proxy-arp` |
| Security segmentation is bypassed | Proxy ARP allows reachability that should require explicit routing or gateway policy | `show ip interface <CLIENT_FACING_INTERFACE>` and ACL checks | Disable proxy ARP and enforce correct gateway or ACL design |
| First ping fails but later pings work | Initial ARP resolution delay | `show ip arp` | Normal if ARP completes; retest after ARP entry exists |
| Debug output shows ARP but no reply | Router has no valid route to requested destination | `debug arp` and `show ip route <DESTINATION_IP>` | Add or repair the route |
| CEF route exists but forwarding fails | Adjacency or return path issue | `show ip cef <DESTINATION_IP>` and `show adjacency detail` | Fix adjacency, downstream route, or reverse path |
| Proxy ARP configuration disappears after reload | Configuration was not saved | `show startup-config interface <CLIENT_FACING_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |
| IPv6 test fails using proxy ARP logic | Proxy ARP is IPv4 ARP behavior only | `show ipv6 neighbors` | Use correct IPv6 neighbor discovery and routing design |
| ARP conflict appears after enabling proxy ARP | Duplicate IP or rogue ARP responder | `show ip arp` and switch MAC table | Remove duplicate IP or rogue responder |
| Troubleshooting is misleading because proxy ARP hides the real issue | Workaround is making bad addressing look functional | Compare client IP/mask/gateway to routing design | Fix the actual addressing and routing design |

# Index

Proxy_ARP_Layer2_Reachability_Workaround.md
Proxy_ARP_Layer2_Reachability_Workaround
Proxy_ARP_Layer2_Reachability_Workaround_Mental_Model
Proxy_ARP_Layer2_Reachability_Workaround_Configuration_Checklist
Proxy_ARP_Layer2_Reachability_Workaround_Skeleton
Proxy_ARP_Disabled_Interface_Skeleton
Local_Proxy_ARP_Skeleton
Ethernet_Static_Route_Correction_Skeleton
Proxy_ARP_Layer2_Reachability_Workaround_Verification_Commands
Client_Verification_Commands
Proxy_ARP_Layer2_Reachability_Workaround_Rollback
Proxy_ARP_Reenable_Rollback
Ethernet_Static_Route_Correction_Rollback
Proxy_ARP_Layer2_Reachability_Workaround_Failure_Checks