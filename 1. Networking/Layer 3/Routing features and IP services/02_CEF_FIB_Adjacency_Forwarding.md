CEF_FIB_Adjacency_Forwarding.md
# CEF_FIB_Adjacency_Forwarding

# CEF_FIB_Adjacency_Forwarding_Mental_Model

| Concept | Operational Meaning |
|---|---|
| CEF | Cisco Express Forwarding is the normal Cisco forwarding path used to move packets without punting every packet to the CPU |
| RIB | The routing table is the control-plane source of truth for best routes |
| FIB | The Forwarding Information Base is built from the routing table and used by CEF for fast destination lookup |
| Adjacency table | The adjacency table stores the Layer 2 rewrite information needed to send the packet to the next hop |
| ARP dependency | On Ethernet, CEF needs ARP resolution so it knows the next-hop MAC address |
| Packet rewrite | CEF rewrites Layer 2 headers, decrements TTL, recalculates checksum, and forwards the packet out the selected interface |
| Glean adjacency | A glean entry means CEF knows the connected prefix but still needs Layer 2 resolution for the specific destination |
| Incomplete adjacency | An incomplete adjacency usually means the next-hop Layer 2 information is missing or unresolved |
| Control plane vs data plane | `show ip route` proves the control-plane decision; `show ip cef` and `show adjacency detail` prove the forwarding-plane programming |
| Troubleshooting rule | If the route looks correct but traffic forwards incorrectly or punts, inspect CEF, ARP, and adjacency state |

# CEF_FIB_Adjacency_Forwarding_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the device has Layer 3 forwarding enabled | L3SW/RTR | `show running-config | include ^ip routing` | `ip routing` is present on multilayer switches; routers route by default |
| 2 | Confirm CEF is enabled globally | L3SW/RTR | `show ip cef` | CEF table is displayed instead of an error or empty unsupported output |
| 3 | Enable CEF if this is an older lab image where CEF is disabled | L3SW/RTR | `conf t`<br>`ip cef`<br>`end` | CEF is enabled globally |
| 4 | Confirm routed interfaces are up | L3SW/RTR | `show ip interface brief` | Ingress, egress, and next-hop interfaces are `up/up` |
| 5 | Confirm the destination route exists in the RIB | L3SW/RTR | `show ip route <DESTINATION_IP>` | Routing table shows the best prefix, next hop, and outgoing interface |
| 6 | Confirm the next hop is reachable in the RIB | L3SW/RTR | `show ip route <NEXT_HOP_IP>` | Router has a valid route to the next-hop IP address |
| 7 | Configure a basic route for the test destination if needed | L3SW/RTR | `conf t`<br>`ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_IP>`<br>`end` | Static route is added to the routing table |
| 8 | Confirm the FIB entry for the destination | L3SW/RTR | `show ip cef <DESTINATION_IP>` | Output shows destination prefix, next hop, outgoing interface, and valid cached adjacency if resolved |
| 9 | Confirm FIB lookup for a source and destination pair | L3SW/RTR | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Output shows the exact egress interface and next hop selected by CEF |
| 10 | Confirm ARP resolution for Ethernet next hop | L3SW/RTR | `show ip arp <NEXT_HOP_IP>` | Next-hop IP maps to a MAC address on the expected interface |
| 11 | Generate traffic to trigger ARP and adjacency creation | L3SW/RTR | `ping <DESTINATION_IP> source <SOURCE_INTERFACE_OR_IP>` | Ping succeeds or at least triggers ARP/CEF adjacency activity |
| 12 | Verify adjacency table details | L3SW/RTR | `show adjacency detail` | Expected interface and next-hop address show Layer 2 rewrite information |
| 13 | Verify the specific adjacency for the egress interface | L3SW/RTR | `show adjacency <EGRESS_INTERFACE> detail` | Egress interface shows valid rewrite information for the next hop |
| 14 | Recheck the CEF entry after ARP resolution | L3SW/RTR | `show ip cef <DESTINATION_IP>` | Entry now shows a valid adjacency instead of glean or incomplete state |
| 15 | Confirm actual forwarding path | L3SW/RTR | `traceroute <DESTINATION_IP> source <SOURCE_IP>` | First hop matches the CEF next-hop decision |
| 16 | Confirm traffic is not being punted unexpectedly | L3SW/RTR | `show interfaces <EGRESS_INTERFACE>` | Interface counters increment normally without obvious errors or drops |
| 17 | Confirm no policy is overriding normal CEF destination forwarding | L3SW/RTR | `show ip policy` | No unexpected route map is applied to the ingress interface |
| 18 | Save the working configuration | L3SW/RTR | `copy running-config startup-config` | CEF/routing baseline survives reload |

# CEF_FIB_Adjacency_Forwarding_Skeleton

conf t
!
! Required on multilayer switches.
ip routing
!
! CEF is normally enabled by default on modern Cisco IOS/IOS XE.
! Use this only in lab images or platforms where CEF is disabled.
ip cef
!
interface GigabitEthernet0/0
 description INGRESS_TEST_INTERFACE
 ip address 10.1.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description EGRESS_TO_NEXT_HOP
 ip address 10.12.12.1 255.255.255.0
 no shutdown
!
ip route 192.168.100.0 255.255.255.0 10.12.12.2
!
end
write memory

# CEF_FIB_Adjacency_Forwarding_Verification_Commands

show running-config | include ^ip routing
show running-config | include ^ip cef
show ip interface brief
show ip route <DESTINATION_IP>
show ip route <NEXT_HOP_IP>
show ip cef
show ip cef <DESTINATION_IP>
show ip cef <DESTINATION_PREFIX> <MASK>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
show ip arp
show ip arp <NEXT_HOP_IP>
show adjacency
show adjacency detail
show adjacency <EGRESS_INTERFACE> detail
show interfaces <EGRESS_INTERFACE>
show ip policy
ping <NEXT_HOP_IP>
ping <DESTINATION_IP> source <SOURCE_INTERFACE_OR_IP>
traceroute <DESTINATION_IP> source <SOURCE_IP>

# CEF_FIB_Adjacency_Forwarding_Rollback

conf t
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_IP>
end
write memory

! Lab-only rollback if CEF was manually enabled only for testing:
conf t
no ip cef
end

! Production note:
! Do not disable CEF on production routers or switches unless a vendor-supported troubleshooting procedure explicitly requires it.

# CEF_FIB_Adjacency_Forwarding_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `show ip route` is correct but forwarding fails | CEF or adjacency programming issue | `show ip cef <DESTINATION_IP>` | Recheck CEF entry, next-hop recursion, ARP, and interface state |
| CEF entry shows glean | Destination is on a connected Ethernet prefix and Layer 2 resolution is needed | `show ip cef <DESTINATION_IP>` and `show ip arp <DESTINATION_IP>` | Generate traffic, confirm ARP works, and verify the host exists on the segment |
| CEF entry shows incomplete adjacency | Next-hop MAC address is unresolved | `show ip arp <NEXT_HOP_IP>` | Fix next-hop reachability, VLAN/trunking, ARP filtering, or wrong next-hop address |
| CEF points to the wrong next hop | RIB selected a different route than expected | `show ip route <DESTINATION_IP>` | Fix route preference, prefix length, metric, AD, or route source |
| `show ip cef exact-route` selects unexpected interface | ECMP hashing or policy changes path selection | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Validate ECMP design, source/destination pair, and any PBR on ingress |
| Route exists but ARP does not resolve | Layer 2 path to next hop is broken | `show ip arp <NEXT_HOP_IP>` and `show interfaces <EGRESS_INTERFACE>` | Fix VLAN, trunk, access port, cabling, SVI, or next-hop address |
| Traffic is punted to CPU | Missing adjacency, unsupported feature path, or exception packet | `show ip cef <DESTINATION_IP>` | Resolve adjacency or remove unsupported forwarding feature from the path |
| CEF table is empty or unavailable | CEF disabled or platform limitation | `show running-config | include ^ip cef` | Enable CEF in lab with `ip cef` if supported |
| Ping to next hop works but destination fails | Downstream or return routing problem | `traceroute <DESTINATION_IP>` and downstream `show ip route <SOURCE_IP>` | Add or fix reverse route and downstream forwarding |
| Forwarding breaks after route change | FIB has not programmed expected new path yet or route recursion changed | `show ip route <DESTINATION_IP>` and `show ip cef <DESTINATION_IP>` | Fix recursive next-hop route and verify CEF reconverges |
| Ethernet rewrite looks wrong | Incorrect adjacency or stale ARP entry | `show adjacency detail` and `show ip arp` | Clear or refresh ARP after fixing Layer 2 addressing issue |
| Traffic follows PBR instead of normal route | Route map applied on ingress interface | `show ip policy` | Remove or correct ingress PBR policy |

# Index

CEF_FIB_Adjacency_Forwarding.md
CEF_FIB_Adjacency_Forwarding
CEF_FIB_Adjacency_Forwarding_Mental_Model
CEF_FIB_Adjacency_Forwarding_Configuration_Checklist
CEF_FIB_Adjacency_Forwarding_Skeleton
CEF_FIB_Adjacency_Forwarding_Verification_Commands
CEF_FIB_Adjacency_Forwarding_Rollback
CEF_FIB_Adjacency_Forwarding_Failure_Checks
