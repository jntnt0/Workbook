Floating_Static_Route_Backup_Path.md
# Floating_Static_Route_Backup_Path

# Floating_Static_Route_Backup_Path_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Floating static route | A static route with a higher administrative distance than the preferred route |
| Backup path | The floating route stays out of the routing table until the better route disappears |
| Administrative distance | Lower AD wins when the prefix length is the same |
| Prefix match boundary | Longest prefix match still happens first; floating behavior only compares routes to the same prefix |
| Static primary with static backup | Primary static route uses default AD 1; backup static route uses a higher AD such as 5, 10, or 250 |
| Dynamic primary with static backup | Floating static route must have an AD higher than the dynamic protocol route |
| RIB installation | The backup route is not active until it appears in `show ip route` |
| CEF forwarding | CEF forwards using the active route from the routing table, not inactive floating routes |
| Failure detection limit | A plain floating static route reacts to route removal, not necessarily remote path failure |
| IP SLA boundary | Use IP SLA and object tracking when the interface stays up but upstream reachability fails |

# Floating_Static_Route_Backup_Path_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are up | RTR/L3SW | `show ip interface brief` | Primary and backup transit interfaces are `up/up` |
| 2 | Confirm Layer 3 forwarding is enabled on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 3 | Confirm current route to the destination | RTR/L3SW | `show ip route <DESTINATION_PREFIX>` | Preferred route is installed, or no route exists yet |
| 4 | Confirm primary next-hop reachability | RTR/L3SW | `ping <PRIMARY_NEXT_HOP>` | Primary next hop responds |
| 5 | Confirm backup next-hop reachability | RTR/L3SW | `ping <BACKUP_NEXT_HOP>` | Backup next hop responds |
| 6 | Configure the primary static route if using static primary path | RTR/L3SW | `conf t`<br>`ip route <DESTINATION_PREFIX> <MASK> <PRIMARY_NEXT_HOP>`<br>`end` | Primary static route installs with default AD 1 |
| 7 | Configure the floating static backup route | RTR/L3SW | `conf t`<br>`ip route <DESTINATION_PREFIX> <MASK> <BACKUP_NEXT_HOP> <BACKUP_AD>`<br>`end` | Backup static route is configured with higher AD |
| 8 | Verify the route configuration exists | RTR/L3SW | `show running-config | include ^ip route` | Primary and floating backup static routes are visible |
| 9 | Verify only the preferred route is active | RTR/L3SW | `show ip route <DESTINATION_PREFIX>` | Routing table shows the primary path, not the floating backup path |
| 10 | Verify CEF uses the active primary path | RTR/L3SW | `show ip cef <DESTINATION_IP>` | CEF points to the primary next hop |
| 11 | Test destination reachability over the primary path | RTR/L3SW | `ping <DESTINATION_IP> source <SOURCE_INTERFACE_OR_IP>` | Ping succeeds over the primary path |
| 12 | Trace the active forwarding path | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP>` | First routed hop matches the primary next hop |
| 13 | Simulate primary path failure in the lab | RTR/L3SW | `conf t`<br>`interface <PRIMARY_INTERFACE>`<br>`shutdown`<br>`end` | Primary interface goes down and the primary route is removed |
| 14 | Verify the floating route installs after primary failure | RTR/L3SW | `show ip route <DESTINATION_PREFIX>` | Backup static route appears in the routing table |
| 15 | Verify CEF moves to the backup path | RTR/L3SW | `show ip cef <DESTINATION_IP>` | CEF points to the backup next hop |
| 16 | Test destination reachability over the backup path | RTR/L3SW | `ping <DESTINATION_IP> source <SOURCE_INTERFACE_OR_IP>` | Ping succeeds if downstream and return routing are correct |
| 17 | Trace the backup forwarding path | RTR/L3SW | `traceroute <DESTINATION_IP> source <SOURCE_IP>` | First routed hop matches the backup next hop |
| 18 | Restore the primary path | RTR/L3SW | `conf t`<br>`interface <PRIMARY_INTERFACE>`<br>`no shutdown`<br>`end` | Primary interface returns to `up/up` |
| 19 | Verify route preference returns to primary | RTR/L3SW | `show ip route <DESTINATION_PREFIX>` | Primary route is active again because it has lower AD |
| 20 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | Floating static route design survives reload |

# Floating_Static_Route_Backup_Path_Skeleton

conf t
!
! Required on multilayer switches.
ip routing
!
interface GigabitEthernet0/0
 description PRIMARY_PATH_TO_NEXT_HOP
 ip address 10.12.12.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description BACKUP_PATH_TO_NEXT_HOP
 ip address 10.13.13.1 255.255.255.0
 no shutdown
!
! Primary static route, default AD 1.
ip route 192.168.50.0 255.255.255.0 10.12.12.2
!
! Floating static backup route, higher AD.
ip route 192.168.50.0 255.255.255.0 10.13.13.3 10
!
end
write memory

# Floating_Static_Route_Backup_With_Dynamic_Primary_Skeleton

conf t
!
! Example: dynamic routing owns the primary path.
! Floating static AD must be higher than the dynamic route AD.
!
! OSPF AD is 110, so this backup static route uses AD 200.
ip route 192.168.50.0 255.255.255.0 10.13.13.3 200
!
end
write memory

# Floating_Static_Default_Route_Backup_Skeleton

conf t
!
! Primary default route.
ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP>
!
! Floating backup default route.
ip route 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
!
end
write memory

# Floating_Static_Route_Backup_Path_Verification_Commands

show running-config | include ^ip routing
show ip interface brief
show running-config | include ^ip route
show ip route
show ip route static
show ip route <DESTINATION_PREFIX>
show ip route <DESTINATION_IP>
show ip cef <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
show ip arp <PRIMARY_NEXT_HOP>
show ip arp <BACKUP_NEXT_HOP>
ping <PRIMARY_NEXT_HOP>
ping <BACKUP_NEXT_HOP>
ping <DESTINATION_IP> source <SOURCE_INTERFACE_OR_IP>
traceroute <DESTINATION_IP> source <SOURCE_IP>
show interfaces <PRIMARY_INTERFACE>
show interfaces <BACKUP_INTERFACE>

# Floating_Static_Route_Backup_Path_Rollback

conf t
no ip route <DESTINATION_PREFIX> <MASK> <PRIMARY_NEXT_HOP>
no ip route <DESTINATION_PREFIX> <MASK> <BACKUP_NEXT_HOP> <BACKUP_AD>
end
write memory

# Floating_Static_Default_Route_Backup_Rollback

conf t
no ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP>
no ip route 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
end
write memory

# Floating_Static_Route_Backup_Path_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Backup route installs immediately | Backup AD is too low or primary route is missing | `show ip route <DESTINATION_PREFIX>` | Raise backup AD or repair the primary route |
| Backup route never installs after primary failure | Backup next hop is unreachable | `show ip route <BACKUP_NEXT_HOP>` | Add reachability to the backup next hop |
| Primary route never returns after recovery | Primary next hop or interface is still down | `show ip interface brief` and `ping <PRIMARY_NEXT_HOP>` | Restore interface, VLAN, link, or next-hop reachability |
| Traffic still fails after backup route installs | Downstream or return path is missing | `traceroute <DESTINATION_IP>` and downstream `show ip route <SOURCE_IP>` | Add reverse route or repair downstream routing |
| Wrong route wins | Routes have the same prefix but unexpected AD | `show ip route <DESTINATION_PREFIX>` | Correct administrative distance values |
| More specific route wins over intended floating route | Longest prefix match overrides AD comparison | `show ip route <DESTINATION_IP>` | Check overlapping prefixes and correct route specificity |
| Backup route is configured but invisible | Floating route is inactive because better route still exists | `show running-config | include ^ip route` | This is normal if the primary route is active |
| CEF still points to old next hop | RIB or adjacency did not update as expected | `show ip cef <DESTINATION_IP>` | Verify route change, ARP, and adjacency state |
| Backup route points to Ethernet exit interface only | Multiaccess static route causes ARP issues | `show running-config | include ^ip route` | Use next-hop IP or fully specified static route |
| Failover does not happen during remote ISP outage | Interface remains up and plain static route is still installed | `show ip route <DESTINATION_PREFIX>` | Use IP SLA and object tracking for remote reachability detection |
| Dynamic primary route beats floating static as intended but path is bad | Dynamic protocol still advertises a broken path | Protocol-specific neighbor and database checks | Fix dynamic routing or use tracking for failure logic |
| PBR ignores the floating route decision | Ingress route map overrides normal destination routing | `show ip policy` | Remove or correct policy-based routing |
| Backup path creates asymmetric routing | Forward and return paths differ unexpectedly | Traceroute from both directions | Adjust routing on both sides |
| Floating default route works but specific destination fails | A more specific route overrides the default route | `show ip route <DESTINATION_IP>` | Fix or remove the specific conflicting route |
| Route removed after reload | Configuration was not saved | `show startup-config | include ^ip route` | Reconfigure and save with `copy running-config startup-config` |

# Index

Floating_Static_Route_Backup_Path.md
Floating_Static_Route_Backup_Path
Floating_Static_Route_Backup_Path_Mental_Model
Floating_Static_Route_Backup_Path_Configuration_Checklist
Floating_Static_Route_Backup_Path_Skeleton
Floating_Static_Route_Backup_With_Dynamic_Primary_Skeleton
Floating_Static_Default_Route_Backup_Skeleton
Floating_Static_Route_Backup_Path_Verification_Commands
Floating_Static_Route_Backup_Path_Rollback
Floating_Static_Default_Route_Backup_Rollback
Floating_Static_Route_Backup_Path_Failure_Checks
