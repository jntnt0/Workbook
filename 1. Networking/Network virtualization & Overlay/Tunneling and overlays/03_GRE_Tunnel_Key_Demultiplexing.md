GRE_Tunnel_Key_Demultiplexing.md
# GRE_Tunnel_Key_Demultiplexing
# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | ENARSI Chapter 19, DMVPN Tunnels, GRE Tunnels | Points GRE and tunnel-key behavior to the ENARSI tunnel section |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Hub Configuration | Provides `tunnel key 0-4294967295` syntax and explains that tunnel keys help identify the correct tunnel interface when multiple tunnels use the same tunnel source |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Spoke Configuration for DMVPN Phase 1 | Confirms tunnel keys must match between tunnel peers |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Phase 1 DMVPN Configuration Example | Shows `tunnel key 100` under tunnel interfaces |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Command Reference | Defines `tunnel key` as a global tunnel interface command |
| `CiscoPress_combined_part1.md` | Over-the-Top WAN Virtualization Design Options | States that point-to-point GRE tunnels with the same source and destination require a unique tunnel key as a demultiplexer |
# GRE_Tunnel_Key_Demultiplexing_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Tunnel key | A GRE header value used to identify which logical tunnel should receive the packet |
| Demultiplexing | Separating multiple GRE tunnels that share the same tunnel source and tunnel destination |
| Same source and destination | Multiple tunnel interfaces can use the same underlay endpoints only when each logical tunnel has a unique key |
| Matching key pair | The key configured on one tunnel must match the corresponding tunnel on the peer |
| Unique key per tunnel | Tunnel100 and Tunnel200 should not use the same key if they share the same source and destination |
| Not encryption | `tunnel key` does not encrypt, authenticate, or secure the tunnel |
| Not a password | The key is not a secret credential; it is a tunnel identification value |
| Header overhead | A tunnel key adds GRE header overhead |
| Overlay separation | Each keyed tunnel should normally have its own tunnel subnet, routing scope, or VRF role |
| Wrong key behavior | The tunnel may appear configured, but traffic for that overlay fails or gets dropped because the receiving router cannot match the GRE packet to the expected tunnel |
| Best use case | Multiple GRE tunnels between the same two routers, especially when separating overlays, VRFs, or lab traffic paths |
| Failure boundary | If underlay reachability is broken, the key is irrelevant; underlay must work first |
# GRE_Tunnel_Key_Demultiplexing_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm local underlay interface is up | R1/R2 | `show ip interface brief` | Tunnel source interface is up/up |
| 2 | Confirm R1 can reach R2 underlay address | R1 | `ping <R2_UNDERLAY_IP> source <R1_UNDERLAY_IP_OR_INTERFACE>` | Underlay ping succeeds |
| 3 | Confirm R2 can reach R1 underlay address | R2 | `ping <R1_UNDERLAY_IP> source <R2_UNDERLAY_IP_OR_INTERFACE>` | Reverse underlay ping succeeds |
| 4 | Confirm route to remote tunnel destination does not use a tunnel | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Route exits physical underlay, not Tunnel interface |
| 5 | Build the tunnel-key map before configuring | Admin | `Tunnel100 = key 100, Tunnel200 = key 200` | Each logical tunnel has a unique key value |
| 6 | Create first GRE tunnel on R1 | R1 | `interface Tunnel100` | Tunnel100 exists |
| 7 | Configure R1 Tunnel100 overlay IP | R1 | `ip address <R1_TUN100_IP> <TUN100_MASK>` | Tunnel100 has overlay address |
| 8 | Configure R1 Tunnel100 source | R1 | `tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>` | Tunnel100 uses correct local underlay source |
| 9 | Configure R1 Tunnel100 destination | R1 | `tunnel destination <R2_UNDERLAY_IP>` | Tunnel100 points to R2 underlay endpoint |
| 10 | Configure R1 Tunnel100 key | R1 | `tunnel key 100` | Tunnel100 GRE packets carry key 100 |
| 11 | Configure R1 Tunnel100 operational values | R1 | `bandwidth <KBPS>` then `ip mtu 1400` then `keepalive 5 3` | Tunnel has bandwidth reference, reduced MTU, and optional keepalive |
| 12 | Create matching first GRE tunnel on R2 | R2 | `interface Tunnel100` | Peer Tunnel100 exists |
| 13 | Configure R2 Tunnel100 overlay IP | R2 | `ip address <R2_TUN100_IP> <TUN100_MASK>` | R2 Tunnel100 is in the same overlay subnet |
| 14 | Configure R2 Tunnel100 source | R2 | `tunnel source <R2_UNDERLAY_INTERFACE_OR_IP>` | Tunnel100 uses correct local underlay source |
| 15 | Configure R2 Tunnel100 destination | R2 | `tunnel destination <R1_UNDERLAY_IP>` | Tunnel100 points back to R1 underlay endpoint |
| 16 | Configure matching R2 Tunnel100 key | R2 | `tunnel key 100` | R2 Tunnel100 matches R1 Tunnel100 |
| 17 | Configure R2 Tunnel100 operational values | R2 | `bandwidth <KBPS>` then `ip mtu 1400` then `keepalive 5 3` | Peer has matching operational values |
| 18 | Create second GRE tunnel on R1 using the same source and destination | R1 | `interface Tunnel200` | Tunnel200 exists |
| 19 | Configure R1 Tunnel200 overlay IP | R1 | `ip address <R1_TUN200_IP> <TUN200_MASK>` | Tunnel200 has a different overlay subnet |
| 20 | Configure R1 Tunnel200 source and destination | R1 | `tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>` then `tunnel destination <R2_UNDERLAY_IP>` | Tunnel200 shares the same underlay endpoints as Tunnel100 |
| 21 | Configure unique R1 Tunnel200 key | R1 | `tunnel key 200` | Tunnel200 GRE packets carry key 200 |
| 22 | Create matching second GRE tunnel on R2 | R2 | `interface Tunnel200` | Peer Tunnel200 exists |
| 23 | Configure R2 Tunnel200 overlay IP | R2 | `ip address <R2_TUN200_IP> <TUN200_MASK>` | R2 Tunnel200 is in the Tunnel200 overlay subnet |
| 24 | Configure R2 Tunnel200 source and destination | R2 | `tunnel source <R2_UNDERLAY_INTERFACE_OR_IP>` then `tunnel destination <R1_UNDERLAY_IP>` | Peer Tunnel200 shares same underlay endpoints |
| 25 | Configure matching R2 Tunnel200 key | R2 | `tunnel key 200` | R2 Tunnel200 matches R1 Tunnel200 |
| 26 | Verify tunnel key values in running config | R1/R2 | `show running-config interface Tunnel100` and `show running-config interface Tunnel200` | Each tunnel has the intended `tunnel key` value |
| 27 | Verify tunnel status | R1/R2 | `show interface Tunnel100` and `show interface Tunnel200` | Tunnels are up/up and show GRE/IP transport |
| 28 | Test Tunnel100 overlay reachability | R1 | `ping <R2_TUN100_IP> source <R1_TUN100_IP>` | Tunnel100 ping succeeds |
| 29 | Test Tunnel200 overlay reachability | R1 | `ping <R2_TUN200_IP> source <R1_TUN200_IP>` | Tunnel200 ping succeeds |
| 30 | Add routing for traffic that should use Tunnel100 | R1/R2 | `ip route <REMOTE_PREFIX_A> <MASK> <REMOTE_TUN100_IP>` | Prefix A uses Tunnel100 |
| 31 | Add routing for traffic that should use Tunnel200 | R1/R2 | `ip route <REMOTE_PREFIX_B> <MASK> <REMOTE_TUN200_IP>` | Prefix B uses Tunnel200 |
| 32 | Verify route separation | R1/R2 | `show ip route <REMOTE_PREFIX_A>` and `show ip route <REMOTE_PREFIX_B>` | Each prefix points to the intended tunnel next hop |
| 33 | Verify forwarding separation | R1/R2 | `traceroute <REMOTE_HOST_A> source <LOCAL_SOURCE_A>` and `traceroute <REMOTE_HOST_B> source <LOCAL_SOURCE_B>` | Traffic follows the intended keyed GRE tunnel |
| 34 | Save the working configuration | R1/R2 | `write memory` | Keyed GRE tunnel design persists after reload |
# GRE_Tunnel_Key_Demultiplexing_Skeleton
! =========================================================
! DESIGN RULE
! =========================================================
! tunnel key is a demultiplexer.
! It is not encryption.
! It is not authentication.
! It is not a password.
!
! Use a unique key when multiple GRE tunnels share:
! - same tunnel source
! - same tunnel destination
!
! The key must match on both ends of the same logical tunnel.
! =========================================================
! R1 TUNNEL100
! =========================================================
conf t
!
interface Tunnel100
 description GRE_TO_R2_OVERLAY_A_KEY_100
 bandwidth 4000
 ip address <R1_TUN100_IP> <TUN100_MASK>
 ip mtu 1400
 keepalive 5 3
 tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R2_UNDERLAY_IP>
 tunnel key 100
 no shutdown
!
end
! =========================================================
! R2 TUNNEL100
! =========================================================
conf t
!
interface Tunnel100
 description GRE_TO_R1_OVERLAY_A_KEY_100
 bandwidth 4000
 ip address <R2_TUN100_IP> <TUN100_MASK>
 ip mtu 1400
 keepalive 5 3
 tunnel source <R2_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R1_UNDERLAY_IP>
 tunnel key 100
 no shutdown
!
end
! =========================================================
! R1 TUNNEL200
! Same underlay source and destination as Tunnel100.
! Different overlay subnet.
! Different tunnel key.
! =========================================================
conf t
!
interface Tunnel200
 description GRE_TO_R2_OVERLAY_B_KEY_200
 bandwidth 4000
 ip address <R1_TUN200_IP> <TUN200_MASK>
 ip mtu 1400
 keepalive 5 3
 tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R2_UNDERLAY_IP>
 tunnel key 200
 no shutdown
!
end
! =========================================================
! R2 TUNNEL200
! =========================================================
conf t
!
interface Tunnel200
 description GRE_TO_R1_OVERLAY_B_KEY_200
 bandwidth 4000
 ip address <R2_TUN200_IP> <TUN200_MASK>
 ip mtu 1400
 keepalive 5 3
 tunnel source <R2_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R1_UNDERLAY_IP>
 tunnel key 200
 no shutdown
!
end
! =========================================================
! OPTIONAL STATIC ROUTING SEPARATION
! =========================================================
conf t
!
! Prefix set A uses Tunnel100
ip route <REMOTE_PREFIX_A> <MASK_A> <REMOTE_TUN100_IP>
!
! Prefix set B uses Tunnel200
ip route <REMOTE_PREFIX_B> <MASK_B> <REMOTE_TUN200_IP>
!
end
write memory
! =========================================================
! OPTIONAL VRF-AWARE SEPARATION
! Use only if the lab is separating VRFs across keyed tunnels.
! =========================================================
conf t
!
ip vrf RED
 rd 100:100
!
ip vrf BLUE
 rd 200:200
!
interface Tunnel100
 ip vrf forwarding RED
 ip address <R1_RED_TUNNEL_IP> <MASK>
 tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R2_UNDERLAY_IP>
 tunnel key 100
!
interface Tunnel200
 ip vrf forwarding BLUE
 ip address <R1_BLUE_TUNNEL_IP> <MASK>
 tunnel source <R1_UNDERLAY_INTERFACE_OR_IP>
 tunnel destination <R2_UNDERLAY_IP>
 tunnel key 200
!
end
write memory
# GRE_Tunnel_Key_Demultiplexing_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Confirm underlay interface state | R1/R2 | `show ip interface brief` | Underlay tunnel source is up/up |
| Confirm underlay route to peer | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Route exits physical underlay |
| Confirm underlay reachability | R1/R2 | `ping <REMOTE_UNDERLAY_IP> source <LOCAL_UNDERLAY_IP_OR_INTERFACE>` | Ping succeeds |
| Confirm Tunnel100 key | R1/R2 | `show running-config interface Tunnel100` | Output includes `tunnel key 100` |
| Confirm Tunnel200 key | R1/R2 | `show running-config interface Tunnel200` | Output includes `tunnel key 200` |
| Confirm tunnel source/destination reuse | R1/R2 | `show running-config interface Tunnel100` and `show running-config interface Tunnel200` | Both tunnels can share source and destination because keys are unique |
| Confirm Tunnel100 interface state | R1/R2 | `show interface Tunnel100` | Tunnel100 is up/up |
| Confirm Tunnel200 interface state | R1/R2 | `show interface Tunnel200` | Tunnel200 is up/up |
| Confirm GRE transport | R1/R2 | `show interface Tunnel100` and `show interface Tunnel200` | Output shows GRE/IP tunnel transport |
| Confirm Tunnel100 overlay ping | R1 | `ping <R2_TUN100_IP> source <R1_TUN100_IP>` | Ping succeeds through key 100 tunnel |
| Confirm Tunnel200 overlay ping | R1 | `ping <R2_TUN200_IP> source <R1_TUN200_IP>` | Ping succeeds through key 200 tunnel |
| Confirm reverse Tunnel100 overlay ping | R2 | `ping <R1_TUN100_IP> source <R2_TUN100_IP>` | Reverse ping succeeds through key 100 tunnel |
| Confirm reverse Tunnel200 overlay ping | R2 | `ping <R1_TUN200_IP> source <R2_TUN200_IP>` | Reverse ping succeeds through key 200 tunnel |
| Confirm route using Tunnel100 | R1/R2 | `show ip route <REMOTE_PREFIX_A>` | Prefix A points to Tunnel100 next hop |
| Confirm route using Tunnel200 | R1/R2 | `show ip route <REMOTE_PREFIX_B>` | Prefix B points to Tunnel200 next hop |
| Confirm forwarding separation | R1/R2 | `traceroute <REMOTE_HOST_A> source <LOCAL_SOURCE_A>` | Traffic follows overlay A |
| Confirm second forwarding path | R1/R2 | `traceroute <REMOTE_HOST_B> source <LOCAL_SOURCE_B>` | Traffic follows overlay B |
| Confirm no recursive routing | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Remote tunnel destination does not point to Tunnel100 or Tunnel200 |
| Confirm no tunnel failure logs | R1/R2 | `show logging | include Tunnel|GRE|RECUR|LINEPROTO` | No tunnel recursion, line protocol, or key-related failure symptoms appear |
# GRE_Tunnel_Key_Demultiplexing_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove static routes using Tunnel100 | R1/R2 | `no ip route <REMOTE_PREFIX_A> <MASK_A> <REMOTE_TUN100_IP>` | Prefix A no longer routes through Tunnel100 |
| 2 | Remove static routes using Tunnel200 | R1/R2 | `no ip route <REMOTE_PREFIX_B> <MASK_B> <REMOTE_TUN200_IP>` | Prefix B no longer routes through Tunnel200 |
| 3 | Remove Tunnel100 key only if keeping tunnel without demux | R1/R2 | `interface Tunnel100` then `no tunnel key` | Tunnel100 no longer carries GRE key |
| 4 | Remove Tunnel200 key only if keeping tunnel without demux | R1/R2 | `interface Tunnel200` then `no tunnel key` | Tunnel200 no longer carries GRE key |
| 5 | Shut Tunnel100 before deletion | R1/R2 | `interface Tunnel100` then `shutdown` | Tunnel100 stops forwarding |
| 6 | Shut Tunnel200 before deletion | R1/R2 | `interface Tunnel200` then `shutdown` | Tunnel200 stops forwarding |
| 7 | Delete Tunnel100 if rolling back fully | R1/R2 | `no interface Tunnel100` | Tunnel100 is removed |
| 8 | Delete Tunnel200 if rolling back fully | R1/R2 | `no interface Tunnel200` | Tunnel200 is removed |
| 9 | Verify tunnel interfaces are removed | R1/R2 | `show ip interface brief | include Tunnel` | Removed tunnels no longer appear |
| 10 | Verify underlay remains intact | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Underlay route remains present |
| 11 | Save rollback state | R1/R2 | `write memory` | Rollback persists after reload |
# GRE_Tunnel_Key_Demultiplexing_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Tunnel interface exists but overlay ping fails | Tunnel key mismatch | R1/R2 | `show running-config interface Tunnel100` | Configure the same key on both ends of the same tunnel |
| Tunnel100 traffic enters wrong logical design | Duplicate key reused on another tunnel | R1/R2 | `show running-config interface Tunnel100` and `show running-config interface Tunnel200` | Use a unique key per tunnel when source and destination match |
| Tunnel200 does not pass traffic | Peer key missing | R1/R2 | `show running-config interface Tunnel200` | Add matching `tunnel key <VALUE>` on the peer |
| First tunnel works, second tunnel fails | Same source and destination configured without unique demux values | R1/R2 | `show running-config interface Tunnel100` and `show running-config interface Tunnel200` | Assign different tunnel keys to the tunnels |
| Tunnel line protocol flaps with keepalive enabled | GRE keepalives are not returning through the matching keyed tunnel | R1/R2 | `show interface TunnelX` and `show logging` | Fix key mismatch, underlay reachability, or remove keepalive for testing |
| Underlay ping fails | Transport problem, not a key problem | R1/R2 | `ping <REMOTE_UNDERLAY_IP> source <LOCAL_UNDERLAY_IP_OR_INTERFACE>` | Restore underlay routing before troubleshooting the key |
| Route to peer underlay points to a tunnel | Recursive routing | R1/R2 | `show ip route <REMOTE_UNDERLAY_IP>` | Force tunnel destination reachability through the underlay |
| Overlay ping uses wrong source | Wrong ping source address | R1/R2 | `ping <REMOTE_TUNNEL_IP> source <LOCAL_TUNNEL_IP>` | Source the test from the correct tunnel IP |
| Remote prefix uses wrong tunnel | Static route points to wrong tunnel next hop | R1/R2 | `show ip route <REMOTE_PREFIX>` | Correct the next hop to the intended tunnel peer IP |
| Keyed tunnels work but large traffic fails | MTU issue from GRE overhead | R1/R2 | `ping <REMOTE_TUNNEL_IP> size 1476 df-bit` | Lower `ip mtu`, adjust MSS, or correct path MTU |
| Config looks correct but no traffic passes | GRE protocol blocked in the underlay | Transit device | `show access-lists` | Permit GRE protocol 47 between tunnel endpoints |
| Operator assumes key secures traffic | Misunderstanding of tunnel key purpose | Admin | `show running-config interface TunnelX` | Use IPsec if confidentiality, integrity, or authentication is required |
##### Source_Basis
# GRE_Tunnel_Key_Demultiplexing_Mental_Model
# GRE_Tunnel_Key_Demultiplexing_Configuration_Checklist
# GRE_Tunnel_Key_Demultiplexing_Skeleton
# GRE_Tunnel_Key_Demultiplexing_Verification_Commands
# GRE_Tunnel_Key_Demultiplexing_Rollback
# GRE_Tunnel_Key_Demultiplexing_Failure_Checks
# index of each title throughout note, not in table format

