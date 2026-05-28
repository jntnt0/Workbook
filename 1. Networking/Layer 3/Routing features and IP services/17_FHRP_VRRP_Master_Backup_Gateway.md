FHRP_VRRP_Master_Backup_Gateway.md
# FHRP_VRRP_Master_Backup_Gateway

# FHRP_VRRP_Master_Backup_Gateway_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VRRP | Standards-based first-hop redundancy protocol that gives hosts a stable virtual default gateway |
| Master router | Router currently owns the virtual IP and forwards traffic sent to the virtual gateway |
| Backup router | Router waiting to take over if the master fails |
| Virtual IP | Default gateway IP used by hosts or delivered through DHCP |
| Virtual MAC | VRRP virtual MAC uses `0000.5e00.01xx`, where `xx` is the group ID in hex |
| Group ID | VRRP instance number shared by routers participating in the same virtual gateway |
| Priority | Higher priority wins master role; default priority is 100 |
| IP address owner | Router that owns the virtual IP as a real interface IP uses priority 255 and should become master |
| Preemption | VRRP preemption is enabled by default, so a higher-priority router can reclaim master role |
| Advertisement | VRRP routers use advertisements to monitor the master state |
| Multicast | VRRP uses multicast `224.0.0.18` for protocol communication |
| VRRPv2 | IPv4-only VRRP version with legacy interface-level syntax |
| VRRPv3 | Supports IPv4 and IPv6 with hierarchical address-family syntax on IOS XE |
| Tracking | VRRP priority can decrement when an uplink, route, or object fails |
| Host transparency | Hosts keep using the same default gateway IP while the physical master router changes |
| Troubleshooting rule | Confirm same subnet, same group, same version, same virtual IP, working multicast, and correct priority before blaming routing |

# FHRP_VRRP_Master_Backup_Gateway_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the client VLAN or subnet that needs VRRP | RTR/L3SW | `show ip interface brief` | Client gateway interfaces or SVIs are identified |
| 2 | Confirm Layer 3 forwarding on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 3 | Enable Layer 3 forwarding if required | L3SW | `conf t`<br>`ip routing`<br>`end` | Switch can route between VLANs |
| 4 | Confirm the client VLAN exists on switches | L3SW | `show vlan brief` | Client VLAN exists and is active |
| 5 | Confirm trunks carry the client VLAN if SVIs are used | L3SW | `show interfaces trunk` | Client VLAN is allowed and forwarding on trunks |
| 6 | Configure the first router or SVI real IP | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <VRRP_A_REAL_IP> <MASK>`<br>` no shutdown`<br>`end` | First gateway interface is up with its unique real IP |
| 7 | Configure the second router or SVI real IP | VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <VRRP_B_REAL_IP> <MASK>`<br>` no shutdown`<br>`end` | Second gateway interface is up with its unique real IP |
| 8 | Confirm peers can reach each other on the shared segment | VRRP-A/VRRP-B | `ping <PEER_REAL_IP>` | VRRP peers can reach each other over the client subnet |
| 9 | Select the VRRP virtual default gateway IP | VRRP-A/VRRP-B | `! Example: real IPs .2 and .3, virtual IP .1` | Virtual IP is inside the client subnet and does not conflict with another host unless intentionally using IP owner behavior |
| 10 | Decide whether to use VRRPv2 or VRRPv3 | VRRP-A/VRRP-B | `show version`<br>`vrrp ?` | Platform syntax is confirmed before configuration |
| 11 | Configure legacy VRRPv2 virtual IP on router A | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> ip <VIRTUAL_IP>`<br>`end` | Router A joins the VRRP group with the virtual IP |
| 12 | Configure legacy VRRPv2 virtual IP on router B | VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> ip <VIRTUAL_IP>`<br>`end` | Router B joins the same VRRP group with the same virtual IP |
| 13 | Configure preferred master priority on router A | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> priority 110`<br>`end` | Router A has higher priority than the default value of 100 |
| 14 | Configure backup router priority on router B | VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> priority 100`<br>`end` | Router B has lower priority than router A |
| 15 | Confirm preemption behavior | VRRP-A/VRRP-B | `show vrrp` | Output shows preemption enabled unless explicitly disabled |
| 16 | Optional: disable preemption only if stable non-revertive failover is required | VRRP-A/VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` no vrrp <GROUP_ID> preempt`<br>`end` | Higher-priority router does not automatically reclaim master after recovery |
| 17 | Optional: configure VRRPv2 authentication if supported and required | VRRP-A/VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> authentication md5 key-string <KEY_STRING>`<br>`end` | Peers require matching authentication for the group |
| 18 | Optional: create a track object for upstream failure detection | VRRP-A | `conf t`<br>`track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol`<br>`end` | Track object follows uplink line protocol |
| 19 | Optional: tie VRRP priority to the tracked object | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>`<br>`end` | Router A priority drops when the tracked object fails |
| 20 | Configure VRRPv3 globally if using hierarchical IOS XE syntax | VRRP-A/VRRP-B | `conf t`<br>`fhrp version vrrp v3`<br>`end` | VRRPv3 syntax is enabled |
| 21 | Configure VRRPv3 IPv4 group on router A | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> address-family ipv4`<br>`  address <VIRTUAL_IP>`<br>`  priority 110`<br>`end` | Router A joins VRRPv3 IPv4 group as preferred master |
| 22 | Configure VRRPv3 IPv4 group on router B | VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> address-family ipv4`<br>`  address <VIRTUAL_IP>`<br>`  priority 100`<br>`end` | Router B joins the same VRRPv3 IPv4 group as backup |
| 23 | Optional: configure VRRPv3 object tracking | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> address-family ipv4`<br>`  track <TRACK_ID> decrement <DECREMENT_VALUE>`<br>`end` | VRRPv3 priority decrements when the tracked object fails |
| 24 | Optional: configure VRRPv2 compatibility mode under VRRPv3 only when required | VRRP-A/VRRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` vrrp <GROUP_ID> address-family ipv4`<br>`  vrrpv2`<br>`end` | VRRPv3 group operates in VRRPv2 compatibility mode where supported |
| 25 | Verify VRRP brief state | VRRP-A/VRRP-B | `show vrrp brief` | One router is `Master`; the other router is `Backup` |
| 26 | Verify detailed VRRP state | VRRP-A/VRRP-B | `show vrrp` | Output shows virtual IP, virtual MAC, priority, preemption, master address, and timers |
| 27 | Verify interface configuration | VRRP-A/VRRP-B | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Interface has matching group, virtual IP, version, priority, and tracking as intended |
| 28 | Verify the virtual IP responds | Client/RTR | `ping <VIRTUAL_IP>` | Client or test router can reach the VRRP virtual gateway |
| 29 | Verify client default gateway points to the virtual IP | Client | `ipconfig /all` | Default gateway is `<VIRTUAL_IP>`, not a physical router IP |
| 30 | Verify client ARP resolves virtual IP to VRRP virtual MAC | Client | `arp -a` | Virtual gateway IP maps to a MAC beginning with `00-00-5e-00-01` |
| 31 | Verify routed reachability through the master gateway | Client | `ping <REMOTE_IP>`<br>`tracert <REMOTE_IP>` | Client reaches remote networks through the active master router |
| 32 | Test master failure in the lab | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` shutdown`<br>`end` | Backup router becomes master after master-down detection |
| 33 | Verify failover state | VRRP-B | `show vrrp brief` | Router B shows `Master` for the VRRP group |
| 34 | Verify client reachability after failover | Client | `ping <REMOTE_IP>` | Client traffic resumes through the new master router |
| 35 | Restore preferred master router | VRRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` no shutdown`<br>`end` | Router A rejoins the VRRP group |
| 36 | Verify default preemption restores preferred master | VRRP-A/VRRP-B | `show vrrp brief` | Router A becomes master again if it has the highest priority |
| 37 | Test tracking failover in the lab | VRRP-A | `conf t`<br>`interface <UPLINK_INTERFACE>`<br>` shutdown`<br>`end` | Track object goes down and VRRP priority decrements |
| 38 | Verify tracked priority decrement | VRRP-A | `show vrrp`<br>`show track <TRACK_ID>` | Output shows tracking state and reduced priority |
| 39 | Restore tracked uplink | VRRP-A | `conf t`<br>`interface <UPLINK_INTERFACE>`<br>` no shutdown`<br>`end` | Track object returns up and priority recovers |
| 40 | Confirm final stable master and backup roles | VRRP-A/VRRP-B | `show vrrp brief` | Preferred router is master and backup router is backup |
| 41 | Save the working configuration | VRRP-A/VRRP-B | `copy running-config startup-config` | VRRP configuration survives reload |

# FHRP_VRRP_Master_Backup_Gateway_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_VRRP
 ip address <REAL_IP> <MASK>
 vrrp <GROUP_ID> ip <VIRTUAL_IP>
 vrrp <GROUP_ID> priority <PRIORITY>
 no shutdown
!
end
write memory

# VRRPv2_Preferred_Master_Router_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_VRRP_MASTER_PREFERRED
 ip address <VRRP_A_REAL_IP> <MASK>
 vrrp <GROUP_ID> ip <VIRTUAL_IP>
 vrrp <GROUP_ID> priority 110
 no shutdown
!
end
write memory

# VRRPv2_Backup_Router_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_VRRP_BACKUP
 ip address <VRRP_B_REAL_IP> <MASK>
 vrrp <GROUP_ID> ip <VIRTUAL_IP>
 vrrp <GROUP_ID> priority 100
 no shutdown
!
end
write memory

# VRRPv2_Tracking_Skeleton

conf t
!
track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol
!
interface <CLIENT_GATEWAY_INTERFACE>
 vrrp <GROUP_ID> priority 110
 vrrp <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>
!
end
write memory

# VRRPv2_Authentication_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 vrrp <GROUP_ID> authentication md5 key-string <KEY_STRING>
!
end
write memory

# VRRPv3_Preferred_Master_Router_Skeleton

conf t
!
ip routing
fhrp version vrrp v3
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_VRRPV3_MASTER_PREFERRED
 ip address <VRRP_A_REAL_IP> <MASK>
 vrrp <GROUP_ID> address-family ipv4
  address <VIRTUAL_IP>
  priority 110
 exit-vrrp
 no shutdown
!
end
write memory

# VRRPv3_Backup_Router_Skeleton

conf t
!
ip routing
fhrp version vrrp v3
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_VRRPV3_BACKUP
 ip address <VRRP_B_REAL_IP> <MASK>
 vrrp <GROUP_ID> address-family ipv4
  address <VIRTUAL_IP>
  priority 100
 exit-vrrp
 no shutdown
!
end
write memory

# VRRPv3_Tracking_Skeleton

conf t
!
track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol
!
interface <CLIENT_GATEWAY_INTERFACE>
 vrrp <GROUP_ID> address-family ipv4
  priority 110
  track <TRACK_ID> decrement <DECREMENT_VALUE>
 exit-vrrp
!
end
write memory

# VRRP_SVI_Pair_Example_Skeleton

conf t
!
ip routing
!
vlan <CLIENT_VLAN_ID>
 name <CLIENT_VLAN_NAME>
!
interface Vlan<CLIENT_VLAN_ID>
 description CLIENT_VLAN_VRRP_GATEWAY
 ip address <REAL_SVI_IP> <MASK>
 vrrp <GROUP_ID> ip <VIRTUAL_IP>
 vrrp <GROUP_ID> priority <PRIORITY>
 no shutdown
!
end
write memory

# VRRP_Three_Switch_Group_Split_Skeleton

conf t
!
ip routing
!
interface Vlan<CLIENT_VLAN_ID_A>
 description VLAN_A_VRRP_GROUP_A
 ip address <REAL_SVI_IP_A> <MASK_A>
 vrrp <GROUP_ID_A> ip <VIRTUAL_IP_A>
 vrrp <GROUP_ID_A> priority <PRIORITY_FOR_THIS_SWITCH>
!
interface Vlan<CLIENT_VLAN_ID_B>
 description VLAN_B_VRRP_GROUP_B
 ip address <REAL_SVI_IP_B> <MASK_B>
 vrrp <GROUP_ID_B> ip <VIRTUAL_IP_B>
 vrrp <GROUP_ID_B> priority <PRIORITY_FOR_THIS_SWITCH>
!
end
write memory

# FHRP_VRRP_Master_Backup_Gateway_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config | include ^fhrp version
show running-config interface <CLIENT_GATEWAY_INTERFACE>
show vrrp brief
show vrrp
show track
show track <TRACK_ID>
show ip route
show ip route <REMOTE_IP>
show ip cef <REMOTE_IP>
show ip arp
show interfaces <CLIENT_GATEWAY_INTERFACE>
show vlan brief
show interfaces trunk
ping <PEER_REAL_IP>
ping <VIRTUAL_IP>
ping <REMOTE_IP> source <CLIENT_GATEWAY_INTERFACE>
traceroute <REMOTE_IP> source <CLIENT_GATEWAY_INTERFACE>
debug vrrp events
debug vrrp packets
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
arp -a
ping <VIRTUAL_IP>
ping <REMOTE_IP>
tracert <REMOTE_IP>

# FHRP_VRRP_Master_Backup_Gateway_Rollback

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no vrrp <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>
 no vrrp <GROUP_ID> authentication
 no vrrp <GROUP_ID> priority
 no vrrp <GROUP_ID> ip <VIRTUAL_IP>
!
no track <TRACK_ID>
!
end
write memory

# VRRPv3_Rollback

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no vrrp <GROUP_ID> address-family ipv4
!
no track <TRACK_ID>
!
end
write memory

# VRRP_Failover_Test_Restore

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no shutdown
!
interface <UPLINK_INTERFACE>
 no shutdown
!
end

# FHRP_VRRP_Master_Backup_Gateway_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Both routers show master | VRRP peers cannot hear each other | `show vrrp brief` and `ping <PEER_REAL_IP>` | Fix VLAN, trunk, subnet mask, ACL, multicast, or Layer 2 path |
| No router becomes master | Interface is down or VRRP group is not configured correctly | `show ip interface brief` and `show vrrp` | Bring interface up and correct group or virtual IP |
| Wrong router is master | Priority or IP owner behavior is controlling election | `show vrrp` | Set intended priority or avoid unintended virtual IP ownership |
| Higher-priority router keeps taking master role | VRRP preemption is enabled by default | `show vrrp` | Lower the priority or configure `no vrrp <GROUP_ID> preempt` if non-revertive behavior is required |
| Preferred master does not reclaim role | Preemption was disabled | `show vrrp` | Remove `no vrrp <GROUP_ID> preempt` or configure preemption as required |
| VRRP peers do not form | Mismatched VRRP version or group configuration | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Match VRRP version, group ID, and virtual IP |
| VRRPv2 and VRRPv3 peers do not communicate | Version incompatibility | `show running-config | include fhrp version` and `show vrrp` | Use the same VRRP version or configure VRRPv2 compatibility where supported |
| Virtual IP does not respond | No master router or client is in wrong VLAN | `show vrrp brief` and client `ipconfig /all` | Fix VRRP state, VLAN assignment, or client addressing |
| Client uses physical gateway instead of virtual IP | DHCP or manual gateway is wrong | Client `ipconfig /all` | Set default gateway to the VRRP virtual IP |
| Client ARP does not show VRRP virtual MAC | Client is not using the virtual IP or ARP cache is stale | Client `arp -a` | Clear client ARP and verify default gateway |
| VRRP failover works but remote traffic fails | New master lacks upstream route | `show ip route <REMOTE_IP>` on new master | Fix upstream routing from every VRRP router |
| Return traffic fails after failover | Upstream network does not route back through the new master path | Traceroute and upstream `show ip route <CLIENT_SUBNET>` | Fix return routing, dynamic routing, or upstream policy |
| Tracking does not reduce priority | Track object missing or not attached to VRRP group | `show track` and `show vrrp` | Configure `track <TRACK_ID> decrement <VALUE>` under the VRRP group |
| Tracking reduces priority but no failover occurs | Decrement value is too small | `show vrrp` | Increase decrement so master priority falls below backup router |
| VRRP flaps repeatedly | Unstable interface, trunk, multicast, or tracking object | `show vrrp`, `show track`, and interface counters | Fix physical link, VLAN, trunk, multicast, or tracked object instability |
| Authentication mismatch | VRRP authentication differs between peers | `show vrrp` and interface config | Configure matching VRRP authentication or remove it consistently |
| Group ID mismatch | Routers use different VRRP group numbers | `show vrrp brief` | Configure the same group ID for the same virtual gateway |
| Virtual IP mismatch | Peers use different virtual gateway IP addresses | `show vrrp brief` | Configure the same virtual IP on all routers in the group |
| Real IP conflict exists | Virtual IP or real IP duplicates another host | `show ip arp` and duplicate address logs | Use unique real IPs and one shared virtual IP |
| IP owner unexpectedly wins | Router has the virtual IP configured as a real interface IP | `show vrrp` | Use separate real IPs and virtual IP unless IP owner behavior is intended |
| Three-switch VRRP behaves unevenly | Priorities are not planned per VLAN or group | `show vrrp brief` on all switches | Assign master role intentionally per VRRP group |
| Debug output is too noisy | VRRP debug left running | `show debugging` | Run `undebug all` |
| Configuration disappears after reload | Configuration was not saved | `show startup-config interface <CLIENT_GATEWAY_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |

# Index

FHRP_VRRP_Master_Backup_Gateway.md
FHRP_VRRP_Master_Backup_Gateway
FHRP_VRRP_Master_Backup_Gateway_Mental_Model
FHRP_VRRP_Master_Backup_Gateway_Configuration_Checklist
FHRP_VRRP_Master_Backup_Gateway_Skeleton
VRRPv2_Preferred_Master_Router_Skeleton
VRRPv2_Backup_Router_Skeleton
VRRPv2_Tracking_Skeleton
VRRPv2_Authentication_Skeleton
VRRPv3_Preferred_Master_Router_Skeleton
VRRPv3_Backup_Router_Skeleton
VRRPv3_Tracking_Skeleton
VRRP_SVI_Pair_Example_Skeleton
VRRP_Three_Switch_Group_Split_Skeleton
FHRP_VRRP_Master_Backup_Gateway_Verification_Commands
Client_Verification_Commands
FHRP_VRRP_Master_Backup_Gateway_Rollback
VRRPv3_Rollback
VRRP_Failover_Test_Restore
FHRP_VRRP_Master_Backup_Gateway_Failure_Checks