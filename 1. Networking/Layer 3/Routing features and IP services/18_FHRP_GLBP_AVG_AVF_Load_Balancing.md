FHRP_GLBP_AVG_AVF_Load_Balancing.md
# FHRP_GLBP_AVG_AVF_Load_Balancing

# FHRP_GLBP_AVG_AVF_Load_Balancing_Mental_Model

| Concept | Operational Meaning |
|---|---|
| GLBP | Cisco first-hop redundancy protocol that provides both default gateway redundancy and host gateway load sharing |
| Virtual IP | The default gateway IP used by clients |
| AVG | Active Virtual Gateway answers ARP requests for the virtual IP and assigns clients to AVFs |
| AVF | Active Virtual Forwarder forwards traffic for one GLBP virtual MAC address |
| Standby AVG | Router ready to take over the AVG role if the current AVG fails |
| Virtual MAC | GLBP uses virtual MAC addresses so different clients can be assigned to different forwarders |
| Host load sharing | Clients can use the same default gateway IP while resolving it to different virtual MAC addresses |
| Round-robin load balancing | AVG rotates ARP replies across AVFs |
| Weighted load balancing | AVG assigns more clients to routers with higher configured GLBP weighting |
| Host-dependent load balancing | AVG attempts to keep the same host mapped to the same AVF |
| Priority | Higher priority influences which router becomes AVG |
| Weighting | Determines whether a router can participate as an AVF and how much traffic it should receive with weighted balancing |
| Preemption | Allows a higher-priority router to reclaim AVG role after recovery |
| Tracking | GLBP priority or weighting can be adjusted when an uplink, object, or path fails |
| Failure behavior | If an AVF fails, another router can take over forwarding responsibility for that virtual MAC |
| Verification rule | Check AVG role, standby AVG role, AVF state, virtual IP, virtual MACs, load-balancing method, priority, weighting, and client ARP behavior together |

# FHRP_GLBP_AVG_AVF_Load_Balancing_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the client VLAN or subnet that needs redundant gateway load sharing | RTR/L3SW | `show ip interface brief` | Client gateway interfaces or SVIs are identified |
| 2 | Confirm Layer 3 forwarding on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 3 | Enable Layer 3 forwarding if required | L3SW | `conf t`<br>`ip routing`<br>`end` | Switch can route between VLANs |
| 4 | Confirm the client VLAN exists | L3SW | `show vlan brief` | Client VLAN exists and is active |
| 5 | Confirm trunks carry the client VLAN if SVIs are used | L3SW | `show interfaces trunk` | Client VLAN is allowed and forwarding on trunks |
| 6 | Configure router A real gateway IP | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <GLBP_A_REAL_IP> <MASK>`<br>` no shutdown`<br>`end` | Router A interface is up with its unique real IP |
| 7 | Configure router B real gateway IP | GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <GLBP_B_REAL_IP> <MASK>`<br>` no shutdown`<br>`end` | Router B interface is up with its unique real IP |
| 8 | Confirm GLBP peers can reach each other on the client subnet | GLBP-A/GLBP-B | `ping <PEER_REAL_IP>` | GLBP peers can communicate over the shared subnet |
| 9 | Select the GLBP virtual default gateway IP | GLBP-A/GLBP-B | `! Example: real IPs .2 and .3, virtual IP .1` | Virtual IP is in the client subnet and is not assigned as a physical interface IP |
| 10 | Configure GLBP virtual IP on router A | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> ip <VIRTUAL_IP>`<br>`end` | Router A joins the GLBP group |
| 11 | Configure GLBP virtual IP on router B | GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> ip <VIRTUAL_IP>`<br>`end` | Router B joins the same GLBP group with the same virtual IP |
| 12 | Configure router A as preferred AVG | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> priority 110`<br>` glbp <GROUP_ID> preempt`<br>`end` | Router A has higher priority and can reclaim AVG role |
| 13 | Configure router B as backup AVG candidate | GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> priority 100`<br>` glbp <GROUP_ID> preempt`<br>`end` | Router B can serve as standby AVG and AVF |
| 14 | Verify default GLBP state | GLBP-A/GLBP-B | `show glbp brief` | One router is AVG active, another is standby AVG, and AVF entries are active |
| 15 | Verify detailed GLBP state | GLBP-A/GLBP-B | `show glbp` | Output shows virtual IP, active router, standby router, AVF states, virtual MACs, timers, and load-balancing method |
| 16 | Configure round-robin load balancing if default host distribution is intended | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> load-balancing round-robin`<br>`end` | AVG rotates ARP replies across available AVFs |
| 17 | Configure host-dependent load balancing if sticky host-to-AVF mapping is required | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> load-balancing host-dependent`<br>`end` | AVG attempts to keep a host mapped to the same AVF |
| 18 | Configure weighted load balancing if routers should receive traffic based on capacity | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> load-balancing weighted`<br>`end` | AVG uses AVF weighting to influence host assignment |
| 19 | Configure router A GLBP weighting | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> weighting <WEIGHT_A>`<br>`end` | Router A advertises the configured GLBP weight |
| 20 | Configure router B GLBP weighting | GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> weighting <WEIGHT_B>`<br>`end` | Router B advertises the configured GLBP weight |
| 21 | Optional: configure GLBP weighting thresholds | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> weighting <MAX_WEIGHT> lower <LOWER_THRESHOLD> upper <UPPER_THRESHOLD>`<br>`end` | Router participates as an AVF only when weight is above the configured threshold behavior |
| 22 | Optional: create an uplink tracking object | GLBP-A | `conf t`<br>`track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol`<br>`end` | Track object follows uplink line protocol |
| 23 | Optional: tie GLBP weighting to the tracked object | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> weighting track <TRACK_ID> decrement <DECREMENT_VALUE>`<br>`end` | Router A GLBP weighting decreases when tracked object fails |
| 24 | Optional: tune GLBP timers consistently on all peers | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> timers <HELLO_SECONDS> <HOLD_SECONDS>`<br>`end` | GLBP peers use matching hello and hold timers |
| 25 | Optional: configure millisecond GLBP timers where supported and approved | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> timers msec <HELLO_MSEC> msec <HOLD_MSEC>`<br>`end` | GLBP failure detection is faster if platform and design support it |
| 26 | Optional: configure text authentication | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> authentication text <PASSWORD>`<br>`end` | GLBP peers require matching text authentication |
| 27 | Optional: configure MD5 authentication | GLBP-A/GLBP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` glbp <GROUP_ID> authentication md5 key-string <KEY_STRING>`<br>`end` | GLBP peers require matching MD5 authentication |
| 28 | Verify GLBP configuration under the interface | GLBP-A/GLBP-B | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Interface has correct GLBP group, virtual IP, priority, preempt, timers, load-balancing, weighting, tracking, and authentication |
| 29 | Verify AVG and AVF state | GLBP-A/GLBP-B | `show glbp brief` | One AVG is active, one standby AVG exists, and AVFs are active |
| 30 | Verify virtual MAC assignments | GLBP-A/GLBP-B | `show glbp` | GLBP virtual MACs are shown for each forwarder |
| 31 | Verify client default gateway points to GLBP virtual IP | Client | `ipconfig /all` | Client default gateway is `<VIRTUAL_IP>` |
| 32 | Clear client ARP before testing load sharing | Client | `arp -d *` | Client removes stale gateway MAC mapping |
| 33 | Generate ARP from client 1 | Client-1 | `ping <VIRTUAL_IP>`<br>`arp -a` | Client 1 resolves the virtual IP to one GLBP virtual MAC |
| 34 | Generate ARP from client 2 | Client-2 | `ping <VIRTUAL_IP>`<br>`arp -a` | Client 2 may resolve the same virtual IP to a different GLBP virtual MAC |
| 35 | Verify client-to-AVF distribution | GLBP-A/GLBP-B | `show glbp` | AVF client assignment and counters show distribution across forwarders |
| 36 | Verify routed reachability through GLBP | Client | `ping <REMOTE_IP>`<br>`tracert <REMOTE_IP>` | Client reaches remote networks through the GLBP virtual gateway |
| 37 | Test AVG failure in the lab | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` shutdown`<br>`end` | GLBP-B takes over AVG role |
| 38 | Verify AVG failover | GLBP-B | `show glbp brief` | Router B shows active AVG for the group |
| 39 | Verify client reachability after AVG failover | Client | `ping <REMOTE_IP>` | Client traffic resumes through surviving GLBP router |
| 40 | Restore preferred AVG router | GLBP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` no shutdown`<br>`end` | Router A rejoins the GLBP group |
| 41 | Verify preemption restores preferred AVG | GLBP-A/GLBP-B | `show glbp brief` | Router A becomes AVG again if priority and preempt are configured |
| 42 | Test AVF or uplink tracking behavior in the lab | GLBP-A | `conf t`<br>`interface <UPLINK_INTERFACE>`<br>` shutdown`<br>`end` | Track object drops and GLBP weighting decreases |
| 43 | Verify weighting decrement | GLBP-A | `show glbp`<br>`show track <TRACK_ID>` | Output shows reduced weight and tracked object down |
| 44 | Restore tracked uplink | GLBP-A | `conf t`<br>`interface <UPLINK_INTERFACE>`<br>` no shutdown`<br>`end` | Track object returns up and GLBP weighting recovers |
| 45 | Save the working configuration | GLBP-A/GLBP-B | `copy running-config startup-config` | GLBP configuration survives reload |

# FHRP_GLBP_AVG_AVF_Load_Balancing_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_GLBP
 ip address <REAL_IP> <MASK>
 glbp <GROUP_ID> ip <VIRTUAL_IP>
 glbp <GROUP_ID> priority <PRIORITY>
 glbp <GROUP_ID> preempt
 no shutdown
!
end
write memory

# GLBP_Preferred_AVG_Router_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_GLBP_AVG_PREFERRED
 ip address <GLBP_A_REAL_IP> <MASK>
 glbp <GROUP_ID> ip <VIRTUAL_IP>
 glbp <GROUP_ID> priority 110
 glbp <GROUP_ID> preempt
 no shutdown
!
end
write memory

# GLBP_Backup_AVG_AVF_Router_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_GLBP_BACKUP_AVG_AVF
 ip address <GLBP_B_REAL_IP> <MASK>
 glbp <GROUP_ID> ip <VIRTUAL_IP>
 glbp <GROUP_ID> priority 100
 glbp <GROUP_ID> preempt
 no shutdown
!
end
write memory

# GLBP_Round_Robin_Load_Balancing_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> load-balancing round-robin
!
end
write memory

# GLBP_Host_Dependent_Load_Balancing_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> load-balancing host-dependent
!
end
write memory

# GLBP_Weighted_Load_Balancing_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> load-balancing weighted
 glbp <GROUP_ID> weighting <WEIGHT>
!
end
write memory

# GLBP_Weighted_Tracking_Skeleton

conf t
!
track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> load-balancing weighted
 glbp <GROUP_ID> weighting <MAX_WEIGHT> lower <LOWER_THRESHOLD> upper <UPPER_THRESHOLD>
 glbp <GROUP_ID> weighting track <TRACK_ID> decrement <DECREMENT_VALUE>
!
end
write memory

# GLBP_SVI_Pair_Example_Skeleton

conf t
!
ip routing
!
vlan <CLIENT_VLAN_ID>
 name <CLIENT_VLAN_NAME>
!
interface Vlan<CLIENT_VLAN_ID>
 description CLIENT_VLAN_GLBP_GATEWAY
 ip address <REAL_SVI_IP> <MASK>
 glbp <GROUP_ID> ip <VIRTUAL_IP>
 glbp <GROUP_ID> priority <PRIORITY>
 glbp <GROUP_ID> preempt
 glbp <GROUP_ID> load-balancing round-robin
 no shutdown
!
end
write memory

# GLBP_Authentication_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> authentication md5 key-string <KEY_STRING>
!
end
write memory

# GLBP_Timer_Tuning_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> timers <HELLO_SECONDS> <HOLD_SECONDS>
!
end
write memory

# GLBP_Millisecond_Timer_Tuning_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 glbp <GROUP_ID> timers msec <HELLO_MSEC> msec <HOLD_MSEC>
!
end
write memory

# FHRP_GLBP_AVG_AVF_Load_Balancing_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config interface <CLIENT_GATEWAY_INTERFACE>
show glbp brief
show glbp
show glbp <CLIENT_GATEWAY_INTERFACE>
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
debug glbp events
debug glbp packets
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
arp -a
arp -d *
ping <VIRTUAL_IP>
ping <REMOTE_IP>
tracert <REMOTE_IP>

# FHRP_GLBP_AVG_AVF_Load_Balancing_Rollback

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no glbp <GROUP_ID> weighting track <TRACK_ID> decrement <DECREMENT_VALUE>
 no glbp <GROUP_ID> weighting
 no glbp <GROUP_ID> load-balancing
 no glbp <GROUP_ID> authentication
 no glbp <GROUP_ID> timers
 no glbp <GROUP_ID> preempt
 no glbp <GROUP_ID> priority
 no glbp <GROUP_ID> ip <VIRTUAL_IP>
!
no track <TRACK_ID>
!
end
write memory

# GLBP_Failover_Test_Restore

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no shutdown
!
interface <UPLINK_INTERFACE>
 no shutdown
!
end

# FHRP_GLBP_AVG_AVF_Load_Balancing_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Both routers appear active or unstable | GLBP peers cannot hear each other correctly | `show glbp brief` and `ping <PEER_REAL_IP>` | Fix VLAN, trunk, subnet mask, ACL, multicast, or Layer 2 path |
| No AVG is elected | Interface is down or GLBP group is misconfigured | `show ip interface brief` and `show glbp` | Bring interface up and correct group or virtual IP |
| Wrong router becomes AVG | Priority or preempt setting is wrong | `show glbp brief` | Set higher priority on preferred AVG and configure `glbp <GROUP_ID> preempt` |
| Preferred AVG does not reclaim role | Preemption is missing | `show glbp` | Configure `glbp <GROUP_ID> preempt` |
| GLBP peers do not form | Group ID, virtual IP, authentication, subnet, or VLAN mismatch | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Match group, virtual IP, authentication, subnet, and VLAN |
| Clients all use one router | Only one AVF is active or client ARP cache is stale | `show glbp` and client `arp -a` | Clear client ARP and verify all AVFs are active |
| Round-robin appears not to load balance | Too few clients or stale ARP entries | Client `arp -a` across multiple clients | Test with multiple clients and clear ARP cache before testing |
| Weighted load balancing has no visible effect | Weighting not configured on all routers or too few ARP events | `show glbp` | Configure `glbp <GROUP_ID> load-balancing weighted` and router weights |
| Host-dependent mapping does not change after policy edit | Host-dependent mode attempts sticky host mapping | `show glbp` and client `arp -a` | Clear ARP and retest, or use round-robin if rotation is expected |
| Client uses physical gateway instead of virtual IP | DHCP or manual gateway is wrong | Client `ipconfig /all` | Set default gateway to the GLBP virtual IP |
| Virtual IP does not respond | No AVG active or client in wrong VLAN | `show glbp brief` and client IP settings | Fix GLBP state, VLAN assignment, or client addressing |
| Client ARP does not show GLBP virtual MAC | Client has stale ARP or wrong default gateway | Client `arp -a` | Clear ARP and verify gateway IP |
| AVG failover works but remote traffic fails | New AVG or AVF lacks upstream route | `show ip route <REMOTE_IP>` on each GLBP router | Fix upstream routing from every GLBP router |
| AVF forwards traffic but return traffic fails | Upstream return path does not know client subnet through that router | Traceroute and upstream `show ip route <CLIENT_SUBNET>` | Fix return routing, dynamic routing, or upstream policy |
| Tracking does not reduce weight | Track object missing or not tied to GLBP weighting | `show track` and `show glbp` | Configure `glbp <GROUP_ID> weighting track <TRACK_ID> decrement <VALUE>` |
| Weight decreases but router still forwards | Lower threshold not crossed or AVF remains eligible | `show glbp` | Tune `weighting <MAX> lower <LOWER> upper <UPPER>` and decrement value |
| GLBP flaps repeatedly | Unstable interface, trunk, multicast, or tracking object | `show glbp`, `show track`, and interface counters | Fix physical link, VLAN, trunk, multicast, or tracked object instability |
| Authentication mismatch | GLBP authentication differs between peers | `show glbp` and interface config | Configure matching authentication or remove it consistently |
| Timer mismatch | Hello and hold timers differ between peers | `show glbp` | Configure matching timers on all GLBP peers |
| GLBP command is unavailable | Platform or image does not support GLBP | `glbp ?` under interface configuration | Use HSRP or VRRP if GLBP is unsupported |
| Debug output is too noisy | GLBP debug left running | `show debugging` | Run `undebug all` |
| Configuration disappears after reload | Configuration was not saved | `show startup-config interface <CLIENT_GATEWAY_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |

# Index

FHRP_GLBP_AVG_AVF_Load_Balancing.md
FHRP_GLBP_AVG_AVF_Load_Balancing
FHRP_GLBP_AVG_AVF_Load_Balancing_Mental_Model
FHRP_GLBP_AVG_AVF_Load_Balancing_Configuration_Checklist
FHRP_GLBP_AVG_AVF_Load_Balancing_Skeleton
GLBP_Preferred_AVG_Router_Skeleton
GLBP_Backup_AVG_AVF_Router_Skeleton
GLBP_Round_Robin_Load_Balancing_Skeleton
GLBP_Host_Dependent_Load_Balancing_Skeleton
GLBP_Weighted_Load_Balancing_Skeleton
GLBP_Weighted_Tracking_Skeleton
GLBP_SVI_Pair_Example_Skeleton
GLBP_Authentication_Skeleton
GLBP_Timer_Tuning_Skeleton
GLBP_Millisecond_Timer_Tuning_Skeleton
FHRP_GLBP_AVG_AVF_Load_Balancing_Verification_Commands
Client_Verification_Commands
FHRP_GLBP_AVG_AVF_Load_Balancing_Rollback
GLBP_Failover_Test_Restore
FHRP_GLBP_AVG_AVF_Load_Balancing_Failure_Checks
