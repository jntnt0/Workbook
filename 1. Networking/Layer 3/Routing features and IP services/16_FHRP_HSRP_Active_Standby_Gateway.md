FHRP_HSRP_Active_Standby_Gateway.md
# FHRP_HSRP_Active_Standby_Gateway

# FHRP_HSRP_Active_Standby_Gateway_Mental_Model

| Concept | Operational Meaning |
|---|---|
| HSRP | Cisco first-hop redundancy protocol that gives hosts a stable virtual default gateway |
| Active router | Router currently forwarding traffic sent to the virtual IP and virtual MAC |
| Standby router | Router waiting to take over if the active router fails |
| Virtual IP | Default gateway IP configured on hosts or delivered by DHCP |
| Virtual MAC | MAC address associated with the HSRP group and owned by the active router |
| Group number | HSRP instance ID shared by routers participating in the same virtual gateway |
| Priority | Higher priority wins active role; default priority is 100 |
| Tie-breaker | If priorities match, the router with the higher interface IP address becomes active |
| Preemption | Allows a higher-priority router to reclaim active role after it recovers |
| No preemption default | A recovered higher-priority router does not automatically take back active role unless preempt is configured |
| Hello and hold timers | HSRP routers use hello messages to detect active or standby failure |
| Interface tracking | HSRP priority can be decremented when an uplink, WAN, or tracked object fails |
| Host transparency | End hosts keep using the same default gateway IP while the active physical router changes |
| Verification rule | Check host gateway, HSRP state, active router, standby router, priority, preempt, timers, and tracking together |

# FHRP_HSRP_Active_Standby_Gateway_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the client VLAN or routed segment that needs gateway redundancy | SW/RTR | `show ip interface brief` | Client gateway interfaces or SVIs are identified |
| 2 | Confirm Layer 3 forwarding on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 3 | Enable Layer 3 forwarding if required | L3SW | `conf t`<br>`ip routing`<br>`end` | Switch can route between VLANs |
| 4 | Confirm the client VLAN exists on switches | L3SW | `show vlan brief` | Client VLAN exists and is active |
| 5 | Confirm trunks carry the client VLAN if SVIs are used | L3SW | `show interfaces trunk` | Client VLAN is allowed and forwarding on trunks |
| 6 | Configure the first router or SVI physical IP | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <HSRP_A_REAL_IP> <MASK>`<br>` no shutdown`<br>`end` | First gateway interface is up with its unique real IP |
| 7 | Configure the second router or SVI physical IP | HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <HSRP_B_REAL_IP> <MASK>`<br>` no shutdown`<br>`end` | Second gateway interface is up with its unique real IP |
| 8 | Confirm both real gateway IPs are reachable on the segment | HSRP-A/HSRP-B | `ping <PEER_REAL_IP>` | HSRP peers can reach each other over the shared subnet |
| 9 | Select the virtual default gateway IP | HSRP-A/HSRP-B | `! Example: real IPs .2 and .3, virtual IP .1` | Virtual IP is in the client subnet and not assigned to a physical interface |
| 10 | Configure HSRP version if version 2 is required | HSRP-A/HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby version 2`<br>`end` | HSRP version is consistent across peers |
| 11 | Configure HSRP virtual IP on router A | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> ip <VIRTUAL_IP>`<br>`end` | Router A joins HSRP group with the virtual IP |
| 12 | Configure HSRP virtual IP on router B | HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> ip <VIRTUAL_IP>`<br>`end` | Router B joins the same HSRP group with the same virtual IP |
| 13 | Configure router A as preferred active router | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> priority 110`<br>`end` | Router A has higher priority than default 100 |
| 14 | Configure router B as standby preference | HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> priority 100`<br>`end` | Router B has lower priority than router A |
| 15 | Enable preemption on router A | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> preempt`<br>`end` | Router A can reclaim active role after recovery |
| 16 | Enable preemption on router B if deterministic failback behavior is wanted | HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> preempt`<br>`end` | Router B can also become active if its priority becomes highest |
| 17 | Optional: add preempt delay to avoid early takeover during boot | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> preempt delay minimum <SECONDS>`<br>`end` | Router waits before taking active role after recovery |
| 18 | Optional: tune HSRP timers consistently on both peers | HSRP-A/HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> timers <HELLO_SECONDS> <HOLD_SECONDS>`<br>`end` | Both peers use matching hello and hold timers |
| 19 | Optional: configure millisecond timers consistently in lab or approved designs | HSRP-A/HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> timers msec <HELLO_MSEC> msec <HOLD_MSEC>`<br>`end` | HSRP detects failures faster if platform and design support it |
| 20 | Optional: configure simple HSRP authentication | HSRP-A/HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> authentication text <PASSWORD>`<br>`end` | Peers require matching text authentication |
| 21 | Optional: configure MD5 key-string authentication | HSRP-A/HSRP-B | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> authentication md5 key-string <KEY_STRING>`<br>`end` | Peers require matching MD5 authentication |
| 22 | Optional: create tracking object for upstream reachability | HSRP-A | `conf t`<br>`track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol`<br>`end` | Track object follows uplink line protocol |
| 23 | Optional: tie HSRP priority to the tracked object | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` standby <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>`<br>`end` | Router A priority drops when tracked object goes down |
| 24 | Verify HSRP summary state | HSRP-A/HSRP-B | `show standby brief` | One router is `Active`; the other is `Standby` |
| 25 | Verify detailed HSRP state | HSRP-A/HSRP-B | `show standby <CLIENT_GATEWAY_INTERFACE>` | Output shows virtual IP, active router, standby router, priority, preempt, timers, and tracking |
| 26 | Verify HSRP interface configuration | HSRP-A/HSRP-B | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Interface has matching group, virtual IP, version, timers, authentication, and tracking as intended |
| 27 | Verify the virtual IP responds | Client/RTR | `ping <VIRTUAL_IP>` | Client or test router can reach the HSRP virtual gateway |
| 28 | Verify client ARP resolves the virtual IP to the HSRP virtual MAC | Client | `arp -a` | Virtual gateway IP maps to an HSRP virtual MAC |
| 29 | Verify client default gateway points to the HSRP virtual IP | Client | `ipconfig /all` | Default gateway is `<VIRTUAL_IP>`, not a physical router IP |
| 30 | Verify routed reachability through the active gateway | Client | `ping <REMOTE_IP>`<br>`tracert <REMOTE_IP>` | Client reaches remote networks through the active HSRP router |
| 31 | Test active router failure in the lab | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` shutdown`<br>`end` | HSRP-B transitions to active after hold timer expires |
| 32 | Verify failover state after active failure | HSRP-B | `show standby brief` | HSRP-B shows `Active` for the group |
| 33 | Verify client reachability after failover | Client | `ping <REMOTE_IP>` | Client traffic resumes through the new active router |
| 34 | Restore original active router | HSRP-A | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` no shutdown`<br>`end` | Router A rejoins HSRP |
| 35 | Verify preemption restores preferred active router | HSRP-A/HSRP-B | `show standby brief` | Router A becomes active again if priority and preempt are configured |
| 36 | Test tracking failover in the lab | HSRP-A | `conf t`<br>`interface <UPLINK_INTERFACE>`<br>` shutdown`<br>`end` | Track object goes down and HSRP priority decrements |
| 37 | Verify tracked priority decrement | HSRP-A | `show standby <CLIENT_GATEWAY_INTERFACE>` | Output shows track object down and priority reduced |
| 38 | Restore tracked uplink | HSRP-A | `conf t`<br>`interface <UPLINK_INTERFACE>`<br>` no shutdown`<br>`end` | Track object returns up and priority recovers |
| 39 | Confirm final stable active and standby roles | HSRP-A/HSRP-B | `show standby brief` | Preferred router is active and backup router is standby |
| 40 | Save the working configuration | HSRP-A/HSRP-B | `copy running-config startup-config` | HSRP configuration survives reload |

# FHRP_HSRP_Active_Standby_Gateway_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_HSRP
 ip address <REAL_IP> <MASK>
 standby version 2
 standby <GROUP_ID> ip <VIRTUAL_IP>
 standby <GROUP_ID> priority <PRIORITY>
 standby <GROUP_ID> preempt
 no shutdown
!
end
write memory

# HSRP_Preferred_Active_Router_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_HSRP_ACTIVE_PREFERRED
 ip address <HSRP_A_REAL_IP> <MASK>
 standby version 2
 standby <GROUP_ID> ip <VIRTUAL_IP>
 standby <GROUP_ID> priority 110
 standby <GROUP_ID> preempt
 no shutdown
!
end
write memory

# HSRP_Standby_Router_Skeleton

conf t
!
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_GATEWAY_HSRP_STANDBY
 ip address <HSRP_B_REAL_IP> <MASK>
 standby version 2
 standby <GROUP_ID> ip <VIRTUAL_IP>
 standby <GROUP_ID> priority 100
 standby <GROUP_ID> preempt
 no shutdown
!
end
write memory

# HSRP_SVI_Pair_Example_Skeleton

conf t
!
ip routing
!
vlan <CLIENT_VLAN_ID>
 name <CLIENT_VLAN_NAME>
!
interface Vlan<CLIENT_VLAN_ID>
 description CLIENT_VLAN_HSRP_GATEWAY
 ip address <REAL_SVI_IP> <MASK>
 standby version 2
 standby <GROUP_ID> ip <VIRTUAL_IP>
 standby <GROUP_ID> priority <PRIORITY>
 standby <GROUP_ID> preempt
 no shutdown
!
end
write memory

# HSRP_Tracking_Skeleton

conf t
!
track <TRACK_ID> interface <UPLINK_INTERFACE> line-protocol
!
interface <CLIENT_GATEWAY_INTERFACE>
 standby <GROUP_ID> priority 110
 standby <GROUP_ID> preempt
 standby <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>
!
end
write memory

# HSRP_Authentication_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 standby <GROUP_ID> authentication md5 key-string <KEY_STRING>
!
end
write memory

# HSRP_Timer_Tuning_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 standby <GROUP_ID> timers <HELLO_SECONDS> <HOLD_SECONDS>
!
end
write memory

# HSRP_Millisecond_Timer_Tuning_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 standby <GROUP_ID> timers msec <HELLO_MSEC> msec <HOLD_MSEC>
!
end
write memory

# FHRP_HSRP_Active_Standby_Gateway_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config interface <CLIENT_GATEWAY_INTERFACE>
show standby brief
show standby
show standby <CLIENT_GATEWAY_INTERFACE>
show standby <CLIENT_GATEWAY_INTERFACE> brief
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
debug standby terse
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
arp -a
ping <VIRTUAL_IP>
ping <REMOTE_IP>
tracert <REMOTE_IP>

# FHRP_HSRP_Active_Standby_Gateway_Rollback

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no standby <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>
 no standby <GROUP_ID> authentication
 no standby <GROUP_ID> timers
 no standby <GROUP_ID> preempt
 no standby <GROUP_ID> priority
 no standby <GROUP_ID> ip <VIRTUAL_IP>
 no standby version
!
no track <TRACK_ID>
!
end
write memory

# HSRP_Failover_Test_Restore

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no shutdown
!
interface <UPLINK_INTERFACE>
 no shutdown
!
end

# FHRP_HSRP_Active_Standby_Gateway_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Both routers show active | HSRP peers cannot hear each other | `show standby brief` and `ping <PEER_REAL_IP>` | Fix VLAN, trunk, subnet mask, ACL, or Layer 2 path |
| No router becomes active | Interface is down or HSRP not configured correctly | `show ip interface brief` and `show standby` | Bring interface up and correct group or virtual IP |
| Wrong router is active | Priority or preempt setting is wrong | `show standby brief` | Set higher priority on preferred router and configure `standby <GROUP_ID> preempt` |
| Preferred router does not reclaim active role | Preemption is not enabled | `show standby <CLIENT_GATEWAY_INTERFACE>` | Configure `standby <GROUP_ID> preempt` |
| HSRP peers do not form | Mismatched VLAN, subnet, HSRP version, or authentication | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Match VLAN, IP subnet, HSRP version, group, and authentication |
| Virtual IP does not respond | No active router or client is in wrong VLAN | `show standby brief` and client `ipconfig /all` | Fix HSRP state, VLAN assignment, or client addressing |
| Client uses physical gateway instead of virtual IP | DHCP or manual gateway is wrong | Client `ipconfig /all` | Set default gateway to the HSRP virtual IP |
| Client loses gateway during failover longer than expected | Timers are default or convergence depends on STP or upstream routing | `show standby` and `show spanning-tree` | Tune timers carefully and verify Layer 2 convergence |
| HSRP flaps repeatedly | Unstable interface, tracking object, or Layer 2 issue | `show standby` and `show track` | Fix physical link, tracking target, VLAN, or trunk instability |
| Tracking does not reduce priority | Track object is missing or not tied to HSRP group | `show track` and `show standby` | Configure `standby <GROUP_ID> track <TRACK_ID> decrement <VALUE>` |
| Tracking reduces priority but failover does not occur | Decrement value is too small | `show standby <CLIENT_GATEWAY_INTERFACE>` | Increase decrement so active router priority falls below standby router |
| HSRP active moves but upstream traffic fails | New active router lacks upstream route | `show ip route <REMOTE_IP>` on active router | Fix upstream routing from both HSRP routers |
| Return traffic fails after failover | Upstream path does not route back through the new active router | Traceroute and upstream `show ip route <CLIENT_SUBNET>` | Fix return routing, dynamic routing, or upstream FHRP |
| Authentication mismatch | HSRP authentication differs between peers | `show standby` | Configure matching authentication on all peers |
| Timer mismatch | Hello and hold timers differ between peers | `show standby <CLIENT_GATEWAY_INTERFACE>` | Configure matching timers on all peers |
| HSRP group mismatch | Routers use different group numbers | `show standby brief` | Configure the same group ID on both routers for the same virtual gateway |
| Virtual IP mismatch | Peers use different virtual IPs | `show standby brief` | Configure the same `standby <GROUP_ID> ip <VIRTUAL_IP>` on both peers |
| Real IP conflict | Virtual IP or real IP duplicates another address | `show ip arp` and duplicate address logs | Use unique real IPs and one shared virtual IP |
| HSRP works but ARP points to old MAC temporarily | Client ARP cache has stale entry | Client `arp -a` | Clear client ARP cache or wait for refresh |
| Multiple HSRP groups behave inconsistently | Load-sharing priorities or group mappings are wrong | `show standby brief` | Verify each group has intended active and standby router |
| Debug output is too noisy | HSRP debug left running | `show debugging` | Run `undebug all` |
| Config disappears after reload | Configuration was not saved | `show startup-config interface <CLIENT_GATEWAY_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |

# Index

FHRP_HSRP_Active_Standby_Gateway.md
FHRP_HSRP_Active_Standby_Gateway
FHRP_HSRP_Active_Standby_Gateway_Mental_Model
FHRP_HSRP_Active_Standby_Gateway_Configuration_Checklist
FHRP_HSRP_Active_Standby_Gateway_Skeleton
HSRP_Preferred_Active_Router_Skeleton
HSRP_Standby_Router_Skeleton
HSRP_SVI_Pair_Example_Skeleton
HSRP_Tracking_Skeleton
HSRP_Authentication_Skeleton
HSRP_Timer_Tuning_Skeleton
HSRP_Millisecond_Timer_Tuning_Skeleton
FHRP_HSRP_Active_Standby_Gateway_Verification_Commands
Client_Verification_Commands
FHRP_HSRP_Active_Standby_Gateway_Rollback
HSRP_Failover_Test_Restore
FHRP_HSRP_Active_Standby_Gateway_Failure_Checks