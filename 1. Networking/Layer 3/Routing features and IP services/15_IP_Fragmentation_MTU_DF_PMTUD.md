IP_Fragmentation_MTU_DF_PMTUD.md
# IP_Fragmentation_MTU_DF_PMTUD

# IP_Fragmentation_MTU_DF_PMTUD_Mental_Model

| Concept | Operational Meaning |
|---|---|
| MTU | Maximum packet size an interface can forward without fragmentation at that layer |
| IP MTU | Maximum IPv4 packet size allowed before the router must fragment or drop |
| Fragmentation | Router splits an oversized IPv4 packet into smaller fragments when DF is not set |
| DF bit | Do Not Fragment bit tells routers to drop oversized packets instead of fragmenting them |
| PMTUD | Path MTU Discovery depends on DF being set and ICMP feedback returning from the constraining hop |
| ICMP fragmentation-needed | Router drops an oversized DF-set packet and sends ICMP unreachable fragmentation-needed back to the sender |
| Black-hole MTU | Traffic fails because oversized DF-set packets are dropped and ICMP feedback is blocked |
| TCP MSS | TCP Maximum Segment Size controls TCP payload size before IP headers and encapsulation overhead are added |
| MSS adjustment | Router rewrites TCP SYN MSS to keep TCP packets small enough for the path |
| Tunnel overhead | GRE, IPsec, VXLAN, DMVPN, and other overlays reduce usable payload MTU because extra headers are added |
| `mtu` vs `ip mtu` | `mtu` changes interface frame size where supported; `ip mtu` changes the IPv4 packet size handled by the interface |
| Verification rule | Prove MTU with DF-set pings, then fix with correct interface MTU, tunnel MTU, MSS clamp, or ICMP PMTUD allowance |

# IP_Fragmentation_MTU_DF_PMTUD_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the source, destination, and suspected constrained path | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP>` | Path toward the destination is known |
| 2 | Confirm all relevant interfaces are up | RTR/L3SW | `show ip interface brief` | Source-facing, transit, tunnel, and destination-facing interfaces are `up/up` |
| 3 | Check physical interface MTU | RTR/L3SW | `show interfaces <INTERFACE>` | Output shows the interface MTU, such as `MTU 1500 bytes` |
| 4 | Check IPv4 interface MTU behavior | RTR/L3SW | `show ip interface <INTERFACE>` | IPv4 interface parameters are visible |
| 5 | Check tunnel MTU if traffic uses a tunnel | RTR/L3SW | `show interfaces Tunnel<TUNNEL_ID>` | Tunnel interface shows tunnel MTU and operational state |
| 6 | Check crypto path MTU if traffic uses IPsec | RTR/L3SW | `show crypto ipsec sa` | Output shows path MTU, IP MTU, and encapsulation details where supported |
| 7 | Confirm routing path before MTU testing | RTR/L3SW | `show ip route <DESTINATION_IP>` | Router has a valid route to the destination |
| 8 | Test normal reachability first | RTR/L3SW | `ping <DESTINATION_IP> source <SOURCE_IP>` | Basic reachability succeeds before testing packet size |
| 9 | Test 1500-byte path behavior with DF set from IOS | RTR/L3SW | `ping <DESTINATION_IP> size 1500 df-bit source <SOURCE_IP>` | Success means the path supports that IPv4 packet size; failure indicates MTU or filtering issue |
| 10 | Test common Ethernet payload size with DF set from IOS | RTR/L3SW | `ping <DESTINATION_IP> size 1472 df-bit source <SOURCE_IP>` | For IPv4 ICMP, 1472 payload plus 28 bytes headers equals 1500-byte packet |
| 11 | Find the largest working DF-set ping size | RTR/L3SW | `ping <DESTINATION_IP> size <TEST_SIZE> df-bit source <SOURCE_IP>` | Largest successful size identifies usable path MTU for ICMP test traffic |
| 12 | Test from Windows client if needed | Windows Client | `ping <DESTINATION_IP> -f -l <PAYLOAD_SIZE>` | Largest successful payload plus 28 bytes gives approximate IPv4 path MTU |
| 13 | Test from Linux client if needed | Linux Client | `ping -M do -s <PAYLOAD_SIZE> <DESTINATION_IP>` | Largest successful payload plus 28 bytes gives approximate IPv4 path MTU |
| 14 | Confirm PMTUD ICMP is not blocked by infrastructure ACLs | RTR/L3SW/FW | `show access-lists` | ACLs do not block required ICMP unreachable or fragmentation-needed feedback |
| 15 | Permit IPv4 ICMP unreachable for PMTUD if ACLs are blocking feedback | RTR/L3SW | `conf t`<br>`ip access-list extended <ACL_NAME>`<br>` permit icmp any any unreachable`<br>`end` | PMTUD feedback is allowed through the IPv4 ACL path |
| 16 | Permit ICMP time-exceeded if traceroute diagnostics are required | RTR/L3SW | `conf t`<br>`ip access-list extended <ACL_NAME>`<br>` permit icmp any any time-exceeded`<br>`end` | Traceroute and TTL-based diagnostics can return useful feedback |
| 17 | Set interface IP MTU when a routed interface must enforce a smaller IPv4 packet size | RTR/L3SW | `conf t`<br>`interface <INTERFACE>`<br>` ip mtu <IP_MTU>`<br>`end` | IPv4 packet MTU is reduced on the interface |
| 18 | Set tunnel IP MTU to account for overlay overhead | RTR/L3SW | `conf t`<br>`interface Tunnel<TUNNEL_ID>`<br>` ip mtu 1400`<br>`end` | Tunnel interface forwards smaller IPv4 packets to avoid post-encapsulation oversize packets |
| 19 | Configure TCP MSS adjustment on the tunnel or ingress interface | RTR/L3SW | `conf t`<br>`interface <INTERFACE>`<br>` ip tcp adjust-mss 1360`<br>`end` | Router rewrites TCP SYN MSS so TCP sessions avoid oversized packets |
| 20 | Optional: enable tunnel PMTUD where supported and required | RTR/L3SW | `conf t`<br>`interface Tunnel<TUNNEL_ID>`<br>` tunnel path-mtu-discovery`<br>`end` | Tunnel can use PMTUD behavior for the tunnel path where platform and design support it |
| 21 | Increase physical or system MTU only when every hop in the path supports it | RTR/L3SW | `conf t`<br>`interface <INTERFACE>`<br>` mtu <MTU_BYTES>`<br>`end` | Interface frame MTU changes if the platform supports per-interface MTU |
| 22 | Confirm whether system MTU changes require reload on the platform | L3SW | `show system mtu` | Platform MTU behavior is known before changing switchwide MTU |
| 23 | Configure system MTU only when required and approved | L3SW | `conf t`<br>`system mtu <MTU_BYTES>`<br>`end` | Switch system MTU is staged or applied depending on platform behavior |
| 24 | Recheck interface MTU after changes | RTR/L3SW | `show interfaces <INTERFACE>` | Interface shows intended MTU value |
| 25 | Recheck tunnel MTU and MSS after changes | RTR/L3SW | `show running-config interface Tunnel<TUNNEL_ID>` | Tunnel shows intended `ip mtu` and `ip tcp adjust-mss` |
| 26 | Retest largest DF-set packet after mitigation | RTR/L3SW | `ping <DESTINATION_IP> size <TEST_SIZE> df-bit source <SOURCE_IP>` | DF-set packet succeeds at the expected safe size |
| 27 | Retest application-style TCP behavior after MSS adjustment | Client | `curl <URL>` or application test | TCP application succeeds without stalling on larger transfers |
| 28 | Check interface drops and errors | RTR/L3SW | `show interfaces <INTERFACE>` | No rising drops, giants, or fragmentation-related symptoms during testing |
| 29 | Check CEF path for the destination | RTR/L3SW | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Forwarding path is the expected path being tested |
| 30 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | MTU, MSS, and ACL changes survive reload |

# IP_Fragmentation_MTU_DF_PMTUD_Skeleton

conf t
!
interface <INTERFACE>
 description MTU_CONTROL_INTERFACE
 ip mtu <IP_MTU>
 ip tcp adjust-mss <MSS_VALUE>
!
end
write memory

# Tunnel_MTU_MSS_Skeleton

conf t
!
interface Tunnel<TUNNEL_ID>
 description TUNNEL_WITH_OVERHEAD_CONTROL
 ip address <TUNNEL_IP> <TUNNEL_MASK>
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source <TUNNEL_SOURCE>
 tunnel destination <TUNNEL_DESTINATION>
 tunnel path-mtu-discovery
!
end
write memory

# Physical_Interface_MTU_Skeleton

conf t
!
interface <INTERFACE>
 description PHYSICAL_LINK_WITH_APPROVED_MTU
 mtu <MTU_BYTES>
!
end
write memory

# PMTUD_ICMP_ACL_Skeleton

conf t
!
ip access-list extended <ACL_NAME>
 permit icmp any any unreachable
 permit icmp any any time-exceeded
!
end
write memory

# System_MTU_Skeleton

conf t
!
system mtu <MTU_BYTES>
!
end
write memory

# IP_Fragmentation_MTU_DF_PMTUD_Verification_Commands

show ip interface brief
show interfaces <INTERFACE>
show ip interface <INTERFACE>
show running-config interface <INTERFACE>
show running-config interface Tunnel<TUNNEL_ID>
show system mtu
show ip route <DESTINATION_IP>
show ip cef <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
show access-lists
show crypto ipsec sa
show interfaces Tunnel<TUNNEL_ID>
ping <DESTINATION_IP> source <SOURCE_IP>
ping <DESTINATION_IP> size 1500 df-bit source <SOURCE_IP>
ping <DESTINATION_IP> size 1472 df-bit source <SOURCE_IP>
ping <DESTINATION_IP> size <TEST_SIZE> df-bit source <SOURCE_IP>
traceroute <DESTINATION_IP> source <SOURCE_IP>
debug ip icmp
show debugging
undebug all

# Client_Verification_Commands

ping <DESTINATION_IP> -f -l <PAYLOAD_SIZE>
ping -M do -s <PAYLOAD_SIZE> <DESTINATION_IP>
tracert <DESTINATION_IP>
traceroute <DESTINATION_IP>
curl <URL>
iperf3 -c <SERVER_IP>

# IP_Fragmentation_MTU_DF_PMTUD_Rollback

conf t
!
interface <INTERFACE>
 no ip mtu
 no ip tcp adjust-mss
!
end
write memory

# Tunnel_MTU_MSS_Rollback

conf t
!
interface Tunnel<TUNNEL_ID>
 no ip mtu
 no ip tcp adjust-mss
 no tunnel path-mtu-discovery
!
end
write memory

# Physical_Interface_MTU_Rollback

conf t
!
interface <INTERFACE>
 no mtu
!
end
write memory

# PMTUD_ICMP_ACL_Rollback

conf t
!
ip access-list extended <ACL_NAME>
 no permit icmp any any unreachable
 no permit icmp any any time-exceeded
!
end
write memory

# System_MTU_Rollback

conf t
!
no system mtu
!
end
write memory

# IP_Fragmentation_MTU_DF_PMTUD_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Small pings work but large transfers fail | Path MTU issue | `ping <DESTINATION_IP> size <TEST_SIZE> df-bit source <SOURCE_IP>` | Lower tunnel IP MTU, adjust TCP MSS, or fix physical MTU |
| DF-set ping fails at 1500 bytes | Path cannot carry 1500-byte IPv4 packets | `ping <DESTINATION_IP> size 1472 df-bit source <SOURCE_IP>` | Find actual path MTU and adjust MSS or MTU |
| DF-set packet fails with no useful ICMP response | PMTUD ICMP feedback is blocked | `show access-lists` | Permit required ICMP unreachable feedback for PMTUD |
| TCP session opens but stalls during data transfer | TCP MSS is too high for the path | Packet capture or application test | Configure `ip tcp adjust-mss <MSS_VALUE>` on the tunnel or ingress interface |
| GRE or DMVPN traffic fails for larger packets | Encapsulation overhead reduces usable payload MTU | `show interfaces Tunnel<TUNNEL_ID>` | Configure `ip mtu 1400` and `ip tcp adjust-mss 1360` on tunnel interfaces |
| IPsec traffic drops large packets | Crypto overhead exceeds path MTU | `show crypto ipsec sa` | Lower tunnel IP MTU or clamp TCP MSS |
| Interface shows giants | Frames exceed physical interface MTU | `show interfaces <INTERFACE>` | Match MTU across the path or stop sending oversized frames |
| OSPF neighbor stuck in ExStart or Exchange | Interface MTU mismatch | `show ip ospf neighbor` and `debug ip ospf adj` | Correct MTU mismatch instead of hiding it with `ip ospf mtu-ignore` unless lab-only |
| DF ping works from router but not from client | Client path or host firewall differs from router test path | Client ping with DF and `show ip cef exact-route` | Test from real source and fix that path |
| MSS clamp has no effect | Applied on wrong interface or not in SYN path | `show running-config interface <INTERFACE>` | Apply MSS adjustment where TCP SYN packets transit |
| UDP application still fails after TCP MSS fix | MSS only affects TCP, not UDP | Application test and packet capture | Reduce application packet size or fix actual path MTU |
| Tunnel PMTUD command has no effect | Platform or tunnel type does not support expected behavior | `show running-config interface Tunnel<TUNNEL_ID>` | Use explicit `ip mtu` and `ip tcp adjust-mss` |
| Physical MTU increase does not fix path | Not every hop supports larger MTU | Test hop by hop with DF-set pings | Increase MTU end to end or keep payload below smallest MTU |
| System MTU change does not apply immediately | Platform requires reload | `show system mtu` | Schedule reload if platform requires it |
| Route path changes during testing | ECMP or routing changes cause different MTU path | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Test exact source and destination flow path |
| PMTUD works one direction but not the other | Asymmetric routing or ACL filtering | Traceroute and ACL checks both directions | Fix return path and ICMP filtering |
| Extended ping size interpretation is wrong | ICMP payload size confused with full packet size | Compare payload plus IP and ICMP headers | For IPv4 ICMP, payload plus 28 bytes approximates packet size |
| Clearing DF makes traffic work | Routers are fragmenting instead of solving MTU | DF and non-DF ping comparison | Prefer PMTUD, MSS clamp, or correct MTU instead of relying on fragmentation |
| Fragmentation causes poor performance | Fragment loss or reassembly overhead | Interface counters and application tests | Avoid fragmentation with correct MTU and MSS |
| Configuration disappears after reload | Config was not saved | `show startup-config interface <INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |

# Index

IP_Fragmentation_MTU_DF_PMTUD.md
IP_Fragmentation_MTU_DF_PMTUD
IP_Fragmentation_MTU_DF_PMTUD_Mental_Model
IP_Fragmentation_MTU_DF_PMTUD_Configuration_Checklist
IP_Fragmentation_MTU_DF_PMTUD_Skeleton
Tunnel_MTU_MSS_Skeleton
Physical_Interface_MTU_Skeleton
PMTUD_ICMP_ACL_Skeleton
System_MTU_Skeleton
IP_Fragmentation_MTU_DF_PMTUD_Verification_Commands
Client_Verification_Commands
IP_Fragmentation_MTU_DF_PMTUD_Rollback
Tunnel_MTU_MSS_Rollback
Physical_Interface_MTU_Rollback
PMTUD_ICMP_ACL_Rollback
System_MTU_Rollback
IP_Fragmentation_MTU_DF_PMTUD_Failure_Checks