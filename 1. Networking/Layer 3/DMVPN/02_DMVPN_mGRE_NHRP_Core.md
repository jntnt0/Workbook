
# DMVPN_mGRE_NHRP_Core_Mental_Model
| Concept           | Operational Meaning                                                           |
| ----------------- | ----------------------------------------------------------------------------- |
| mGRE              | Lets one tunnel interface support multiple remote DMVPN peers                 |
| NHRP              | Maps overlay tunnel IP addresses to underlay NBMA addresses                   |
| NHS               | Next Hop Server, usually the hub, receives spoke registrations                |
| NHC               | Next Hop Client, usually the spoke, registers to the hub                      |
| Tunnel IP         | Overlay IP address assigned to the DMVPN tunnel interface                     |
| NBMA IP           | Real underlay IP address used to send GRE packets                             |
| Tunnel source     | Local underlay interface or IP used to source GRE packets                     |
| NHRP network ID   | Enables NHRP on the tunnel and identifies the local DMVPN cloud               |
| Tunnel key        | Optional tunnel cloud identifier, useful when multiple tunnels share a source |
| Multicast mapping | Required for multicast-based routing protocols such as EIGRP, OSPF, and RIP   |
| MTU/MSS           | Prevents GRE/IPsec overhead from breaking larger packets                      |
| Phase 1 spoke     | Uses point-to-point GRE with `tunnel destination <hub-nbma-ip>`               |
| Phase 2/3 spoke   | Uses mGRE with `tunnel mode gre multipoint`                                   |
| Hard rule         | Underlay NBMA reachability must work before mGRE/NHRP troubleshooting starts  |
# DMVPN_mGRE_NHRP_Core_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before tunnel configuration |
| 2 | Confirm tunnel source interface state | Hub, Spokes | `show ip interface brief` | Tunnel source interface is `up/up` |
| 3 | Confirm transport route to remote NBMA | Hub, Spokes | `show ip route <remote-nbma-ip>` | Remote NBMA resolves through the underlay |
| 4 | Build hub tunnel interface | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub tunnel is configured for mGRE and NHRP |
| 5 | Configure hub tunnel IP | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub has an overlay tunnel address |
| 6 | Configure hub tunnel source | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub sources GRE from the correct underlay address |
| 7 | Configure hub mGRE mode | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub can support multiple dynamic DMVPN peers |
| 8 | Enable NHRP on hub | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub participates in the DMVPN NHRP cloud |
| 9 | Enable dynamic multicast mapping on hub | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub supports multicast from dynamically registered spokes |
| 10 | Configure hub MTU/MSS | Hub | See `Hub_mGRE_NHRP_Core_Config` | Tunnel accounts for GRE/IPsec overhead |
| 11 | Configure optional hub tunnel key | Hub | See `Hub_mGRE_NHRP_Core_Config` | Hub uses the expected tunnel cloud key |
| 12 | Build Phase 1 spoke tunnel if lab is Phase 1 | Spokes | See `Spoke_Phase1_NHRP_Core_Config` | Spoke uses point-to-point GRE toward hub NBMA |
| 13 | Build Phase 2/3 spoke tunnel if lab is Phase 2 or 3 | Spokes | See `Spoke_Phase2_3_NHRP_Core_Config` | Spoke uses mGRE and can support dynamic peer resolution |
| 14 | Configure spoke tunnel IP | Spokes | See spoke config block | Spoke has a unique overlay tunnel address |
| 15 | Configure spoke tunnel source | Spokes | See spoke config block | Spoke sources GRE from the correct underlay address |
| 16 | Enable NHRP on spoke | Spokes | See spoke config block | Spoke participates in the same NHRP cloud |
| 17 | Configure hub as NHS on spoke | Spokes | See spoke config block | Spoke registers to the hub NHS |
| 18 | Configure spoke multicast mapping to hub | Spokes | See spoke config block | Routing protocol multicast can cross the tunnel |
| 19 | Configure spoke MTU/MSS | Spokes | See spoke config block | Tunnel accounts for GRE/IPsec overhead |
| 20 | Configure optional spoke tunnel key | Spokes | See spoke config block | Spoke tunnel key matches the hub |
| 21 | Bring up tunnel interfaces | Hub, Spokes | `no shutdown` | Tunnel interfaces are administratively enabled |
| 22 | Verify tunnel interface state | Hub, Spokes | `show ip interface brief | include Tunnel` | Tunnel interfaces show `up/up` |
| 23 | Verify DMVPN peer state | Hub, Spokes | `show dmvpn` | Peers show `UP` |
| 24 | Verify NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub learns spokes and spokes know the hub |
| 25 | Verify hub-to-spoke tunnel reachability | Hub | `ping <spoke-tunnel-ip>` | Hub can reach each spoke tunnel IP |
| 26 | Verify spoke-to-hub tunnel reachability | Spokes | `ping <hub-tunnel-ip>` | Each spoke can reach the hub tunnel IP |
| 27 | Stop before routing protocols if NHRP is unstable | Hub, Spokes | `show dmvpn` and `show ip nhrp` | Routing protocols are not added until DMVPN core is stable |
# DMVPN_mGRE_NHRP_Core_Skeleton
```text
# Hub_mGRE_NHRP_Core_Config
conf t
interface Tunnel<id>
 description DMVPN_CORE_HUB
 bandwidth <kbps>
 ip address <hub-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id <nhrp-id>
 ip tcp adjust-mss 1360
 tunnel source <hub-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```
```text
# Spoke_Phase1_NHRP_Core_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE1_SPOKE
 bandwidth <kbps>
 ip address <spoke-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 ip tcp adjust-mss 1360
 tunnel source <spoke-underlay-interface-or-nbma-ip>
 tunnel destination <hub-nbma-ip>
 tunnel key <key-id>
 no shutdown
end
```
```text
# Spoke_Phase2_3_NHRP_Core_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE2_3_SPOKE
 bandwidth <kbps>
 ip address <spoke-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 ip tcp adjust-mss 1360
 tunnel source <spoke-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```
```text
# Spoke_Separate_NHRP_Mapping_Alternative
conf t
interface Tunnel<id>
 ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 ip nhrp map multicast <hub-nbma-ip>
 ip nhrp nhs <hub-tunnel-ip>
end
```
# DMVPN_mGRE_NHRP_Core_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| Transport route | Hub, Spokes | `show ip route <remote-nbma-ip>` | Route points through underlay |
| Tunnel state | Hub, Spokes | `show ip interface brief | include Tunnel` | Tunnel interface shows `up/up` |
| Hub tunnel mode | Hub | `show running-config interface Tunnel<id>` | Hub has `tunnel mode gre multipoint` |
| Phase 1 spoke tunnel mode | Spokes | `show running-config interface Tunnel<id>` | Spoke has `tunnel destination <hub-nbma-ip>` |
| Phase 2/3 spoke tunnel mode | Spokes | `show running-config interface Tunnel<id>` | Spoke has `tunnel mode gre multipoint` |
| DMVPN peer state | Hub, Spokes | `show dmvpn` | Peers show `UP` |
| DMVPN detail | Hub, Spokes | `show dmvpn detail` | Shows peer NBMA, tunnel address, and state |
| NHRP cache | Hub, Spokes | `show ip nhrp` | Tunnel IPs map to correct NBMA IPs |
| NHRP brief | Hub, Spokes | `show ip nhrp brief` | Hub has dynamic spoke entries and spokes have hub entry |
| Hub-to-spoke ping | Hub | `ping <spoke-tunnel-ip>` | Ping succeeds |
| Spoke-to-hub ping | Spokes | `ping <hub-tunnel-ip>` | Ping succeeds |
| Multicast mapping | Hub, Spokes | `show running-config interface Tunnel<id> | include multicast` | Hub and spokes have expected multicast mapping |
| MTU/MSS | Hub, Spokes | `show running-config interface Tunnel<id> | include mtu|mss` | Tunnel has expected overhead settings |
# DMVPN_mGRE_NHRP_Core_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Shut down tunnel first | Hub, Spokes | `shutdown` under tunnel | Tunnel stops forming peers |
| 2 | Remove spoke NHS compact command | Spokes | See `Rollback_Spoke_NHRP` | Spoke no longer registers to hub |
| 3 | Remove separate NHRP mappings if used | Spokes | See `Rollback_Spoke_NHRP` | Static hub mappings are removed |
| 4 | Remove NHRP network ID | Hub, Spokes | See `Rollback_Common_Tunnel_Core` | NHRP is disabled on tunnel |
| 5 | Remove multicast mapping | Hub, Spokes | See rollback blocks | Multicast NHRP mapping is removed |
| 6 | Remove tunnel key if used | Hub, Spokes | See `Rollback_Common_Tunnel_Core` | Tunnel key is removed |
| 7 | Remove tunnel mode or destination | Hub, Spokes | See rollback blocks | Tunnel role is cleared |
| 8 | Remove tunnel source | Hub, Spokes | See `Rollback_Common_Tunnel_Core` | Tunnel source is removed |
| 9 | Remove tunnel IP address | Hub, Spokes | `no ip address` under tunnel | Overlay address is removed |
| 10 | Delete tunnel interface if fully resetting | Hub, Spokes | `no interface Tunnel<id>` | Tunnel interface is deleted |
```text
# Rollback_Spoke_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 no ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 no ip nhrp map multicast <hub-nbma-ip>
 no ip nhrp nhs <hub-tunnel-ip>
end
```
```text
# Rollback_Hub_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp map multicast dynamic
 no ip nhrp network-id <nhrp-id>
end
```
```text
# Rollback_Common_Tunnel_Core
conf t
interface Tunnel<id>
 no ip nhrp network-id <nhrp-id>
 no tunnel key <key-id>
 no tunnel source <underlay-interface-or-nbma-ip>
 no ip mtu 1400
 no ip tcp adjust-mss 1360
 no ip address
end
```
```text
# Rollback_Phase1_Spoke_Tunnel_Destination
conf t
interface Tunnel<id>
 no tunnel destination <hub-nbma-ip>
end
```
```text
# Rollback_mGRE_Tunnel_Mode
conf t
interface Tunnel<id>
 no tunnel mode gre multipoint
end
```
# DMVPN_mGRE_NHRP_Core_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No DMVPN peers appear | Underlay broken, tunnel shut, or wrong source | `show dmvpn` | Fix underlay, source, or `no shutdown` |
| Spoke cannot register | Wrong NHS or hub NBMA mapping | `show ip nhrp` | Correct spoke NHS mapping |
| Hub does not learn spokes | Spokes are not registering | `show ip nhrp` on hub | Fix spoke NHS, NHRP ID, or tunnel source |
| Spoke has wrong hub NBMA | Bad static NHRP map | `show running-config interface Tunnel<id>` | Correct hub tunnel-to-NBMA mapping |
| Routing protocol multicast fails | Missing multicast mapping | `show running-config interface Tunnel<id>` | Add hub dynamic multicast and spoke multicast mapping |
| Phase 1 spoke looks wrong | Spoke uses mGRE instead of destination | `show running-config interface Tunnel<id>` | Use `tunnel destination <hub-nbma-ip>` |
| Phase 2/3 spoke looks wrong | Spoke uses point-to-point tunnel destination | `show running-config interface Tunnel<id>` | Use `tunnel mode gre multipoint` |
| Tunnel key mismatch | Peers do not form despite reachability | `show running-config interface Tunnel<id>` | Match or remove tunnel keys |
| Large packets fail | MTU/MSS issue | `ping <tunnel-ip> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| NHRP mappings look stale | Cache has old mapping | `show ip nhrp` | Clear NHRP or fix tunnel source |
| Engineer blames routing too early | NHRP core is not stable | `show dmvpn` and `show ip nhrp` | Fix mGRE/NHRP before routing |
##### Source_Basis
# DMVPN_mGRE_NHRP_Core_Mental_Model
# DMVPN_mGRE_NHRP_Core_Configuration_Checklist
# DMVPN_mGRE_NHRP_Core_Skeleton
# DMVPN_mGRE_NHRP_Core_Verification_Commands
# DMVPN_mGRE_NHRP_Core_Rollback
# DMVPN_mGRE_NHRP_Core_Failure_Checks
