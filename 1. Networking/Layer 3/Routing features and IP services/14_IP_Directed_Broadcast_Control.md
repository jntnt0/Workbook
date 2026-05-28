IP_Directed_Broadcast_Control.md
# IP_Directed_Broadcast_Control

# IP_Directed_Broadcast_Control_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Directed broadcast | An IPv4 packet sent to the broadcast address of a specific remote subnet |
| Local broadcast | `255.255.255.255` stays on the local segment and is not routed |
| Directed subnet broadcast | Example: traffic sent to `192.168.10.255` for subnet `192.168.10.0/24` |
| Last-hop router | The router directly attached to the destination subnet decides whether to convert the packet into a Layer 2 broadcast |
| Transit routers | Routers before the final subnet treat the packet like normal unicast traffic toward the destination subnet |
| `ip directed-broadcast` | Interface command that permits directed broadcasts to be forwarded as Layer 2 broadcasts onto that attached subnet |
| `no ip directed-broadcast` | Interface command that drops directed broadcasts destined for the subnet attached to that interface |
| Default behavior | Directed broadcast forwarding is disabled by default on modern Cisco IOS and IOS XE |
| Security risk | Directed broadcasts can be abused for amplification attacks, especially Smurf-style behavior |
| Controlled use case | Wake-on-LAN or legacy discovery workflows may require tightly filtered directed broadcast forwarding |
| ACL filter | An optional ACL can limit which sources or broadcast types are allowed when directed broadcast must be enabled |
| Troubleshooting rule | Enable directed broadcast only on the destination subnet interface, not everywhere in the routed path |

# IP_Directed_Broadcast_Control_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the destination subnet broadcast address | RTR/L3SW | `show ip interface brief` | Destination interface and subnet are known |
| 2 | Confirm the destination-facing interface is up | RTR/L3SW | `show ip interface brief` | Destination subnet interface is `up/up` |
| 3 | Confirm Layer 3 routing is enabled on a multilayer switch | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 4 | Confirm current directed broadcast state on the destination interface | RTR/L3SW | `show ip interface <DESTINATION_INTERFACE>` | Output shows `Directed broadcast forwarding is disabled` or enabled |
| 5 | Confirm routed reachability to the destination subnet | Source RTR | `show ip route <DESTINATION_SUBNET>` | Source side has a route toward the destination subnet |
| 6 | Confirm return routing from the destination subnet | Destination RTR/L3SW | `show ip route <SOURCE_SUBNET>` | Destination side has a route back toward the source |
| 7 | Keep directed broadcast disabled when there is no explicit approved use case | RTR/L3SW | `conf t`<br>`interface <DESTINATION_INTERFACE>`<br>` no ip directed-broadcast`<br>`end` | Directed broadcasts toward the attached subnet are dropped |
| 8 | Create an ACL to limit approved directed broadcast sources if enabling the feature | RTR/L3SW | `conf t`<br>`ip access-list extended ACL-DIRECTED-BCAST`<br>` permit udp host <APPROVED_SOURCE_IP> host <DESTINATION_BROADCAST_IP> eq <UDP_PORT>`<br>` deny ip any host <DESTINATION_BROADCAST_IP>`<br>`end` | ACL permits only the approved source and protocol toward the subnet broadcast address |
| 9 | Enable filtered directed broadcast on the destination-facing interface | RTR/L3SW | `conf t`<br>`interface <DESTINATION_INTERFACE>`<br>` ip directed-broadcast ACL-DIRECTED-BCAST`<br>`end` | Directed broadcasts matching the ACL can be forwarded onto the attached subnet |
| 10 | Enable unfiltered directed broadcast only in a controlled lab | RTR/L3SW | `conf t`<br>`interface <DESTINATION_INTERFACE>`<br>` ip directed-broadcast`<br>`end` | Router converts directed broadcasts into Layer 2 broadcasts on the attached subnet |
| 11 | Verify directed broadcast state after configuration | RTR/L3SW | `show ip interface <DESTINATION_INTERFACE>` | Output shows whether directed broadcast forwarding is enabled or disabled |
| 12 | Verify ACL configuration if filtering is used | RTR/L3SW | `show access-lists ACL-DIRECTED-BCAST` | ACL entries match only the intended source, protocol, and broadcast destination |
| 13 | Verify interface configuration | RTR/L3SW | `show running-config interface <DESTINATION_INTERFACE>` | Interface shows `ip directed-broadcast` or `ip directed-broadcast ACL-DIRECTED-BCAST` |
| 14 | Send a controlled directed broadcast test from the approved source | Source Host/RTR | `ping <DESTINATION_BROADCAST_IP>` | Test traffic reaches the last-hop router; endpoint responses depend on host firewall and OS behavior |
| 15 | Test UDP-based Wake-on-LAN if that is the approved use case | Source Host | `wolcmd <TARGET_MAC> <DESTINATION_BROADCAST_IP> <SUBNET_MASK> <UDP_PORT>` | Magic packet is sent toward the directed broadcast address |
| 16 | Verify ACL hit counters during testing | RTR/L3SW | `show access-lists ACL-DIRECTED-BCAST` | Permit counter increments for approved directed broadcast traffic |
| 17 | Verify interface counters during testing | RTR/L3SW | `show interfaces <DESTINATION_INTERFACE>` | Output packets or broadcast counters increase during the test |
| 18 | Verify local subnet hosts receive the broadcast if packet capture is available | Destination Host | `tcpdump -n -i <INTERFACE> broadcast` | Broadcast traffic is visible on the destination subnet |
| 19 | Confirm unauthorized directed broadcast traffic is blocked | Unauthorized Source | `ping <DESTINATION_BROADCAST_IP>` | Unauthorized source does not cause broadcast forwarding |
| 20 | Verify deny ACL counters for unauthorized tests | RTR/L3SW | `show access-lists ACL-DIRECTED-BCAST` | Deny counter increments for unauthorized directed broadcast attempts |
| 21 | Return interface to secure default if the test was temporary | RTR/L3SW | `conf t`<br>`interface <DESTINATION_INTERFACE>`<br>` no ip directed-broadcast`<br>`end` | Directed broadcast forwarding is disabled again |
| 22 | Confirm secure default after rollback | RTR/L3SW | `show ip interface <DESTINATION_INTERFACE>` | Output shows directed broadcast forwarding is disabled |
| 23 | Save the intended configuration | RTR/L3SW | `copy running-config startup-config` | Directed broadcast state survives reload |

# IP_Directed_Broadcast_Control_Skeleton

conf t
!
ip routing
!
interface <DESTINATION_INTERFACE>
 description DESTINATION_SUBNET_LAST_HOP
 ip address <DESTINATION_GATEWAY_IP> <SUBNET_MASK>
 no ip directed-broadcast
 no shutdown
!
end
write memory

# IP_Directed_Broadcast_Filtered_Enable_Skeleton

conf t
!
ip access-list extended ACL-DIRECTED-BCAST
 permit udp host <APPROVED_SOURCE_IP> host <DESTINATION_BROADCAST_IP> eq <UDP_PORT>
 deny ip any host <DESTINATION_BROADCAST_IP>
!
interface <DESTINATION_INTERFACE>
 description DESTINATION_SUBNET_LAST_HOP_FILTERED_DIRECTED_BROADCAST
 ip directed-broadcast ACL-DIRECTED-BCAST
!
end
write memory

# IP_Directed_Broadcast_Lab_Enable_Skeleton

conf t
!
interface <DESTINATION_INTERFACE>
 description LAB_ONLY_UNFILTERED_DIRECTED_BROADCAST
 ip directed-broadcast
!
end
write memory

# Wake_On_LAN_Directed_Broadcast_Example_Skeleton

conf t
!
ip access-list extended ACL-WOL-DIRECTED-BCAST
 permit udp host <WOL_SERVER_IP> host <DESTINATION_BROADCAST_IP> eq 7
 permit udp host <WOL_SERVER_IP> host <DESTINATION_BROADCAST_IP> eq 9
 deny ip any host <DESTINATION_BROADCAST_IP>
!
interface <DESTINATION_INTERFACE>
 ip directed-broadcast ACL-WOL-DIRECTED-BCAST
!
end
write memory

# IP_Directed_Broadcast_Control_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config interface <DESTINATION_INTERFACE>
show ip interface <DESTINATION_INTERFACE>
show ip route <DESTINATION_SUBNET>
show ip route <SOURCE_SUBNET>
show access-lists
show access-lists ACL-DIRECTED-BCAST
show interfaces <DESTINATION_INTERFACE>
show ip cef <DESTINATION_BROADCAST_IP>
ping <DESTINATION_BROADCAST_IP>
traceroute <DESTINATION_BROADCAST_IP>
debug ip packet detail
show debugging
undebug all

# Destination_Host_Verification_Commands

ipconfig /all
arp -a
ping <DESTINATION_GATEWAY_IP>
tcpdump -n -i <INTERFACE> broadcast
tcpdump -n -i <INTERFACE> udp port 7 or udp port 9

# IP_Directed_Broadcast_Control_Rollback

conf t
!
interface <DESTINATION_INTERFACE>
 no ip directed-broadcast
!
end
write memory

# IP_Directed_Broadcast_ACL_Rollback

conf t
!
interface <DESTINATION_INTERFACE>
 no ip directed-broadcast
!
no ip access-list extended ACL-DIRECTED-BCAST
no ip access-list extended ACL-WOL-DIRECTED-BCAST
!
end
write memory

# IP_Directed_Broadcast_Control_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Directed broadcast does not forward | `no ip directed-broadcast` is configured or default behavior is active | `show ip interface <DESTINATION_INTERFACE>` | Enable `ip directed-broadcast` only if the use case is approved |
| Directed broadcast enabled on wrong interface | Command was applied on a transit or source-facing interface | `show running-config | include ip directed-broadcast` | Move the command to the destination subnet last-hop interface |
| Source cannot reach broadcast address | Missing route to destination subnet | Source `show ip route <DESTINATION_SUBNET>` | Fix routing toward the destination subnet |
| Destination replies never return | Missing return route | Destination side `show ip route <SOURCE_SUBNET>` | Add or repair return routing |
| ACL blocks approved traffic | ACL source, destination, protocol, or port is wrong | `show access-lists ACL-DIRECTED-BCAST` | Correct ACL permit entry |
| ACL permits too much traffic | ACL is too broad or lacks explicit deny for broadcast destination | `show access-lists ACL-DIRECTED-BCAST` | Restrict source, protocol, destination broadcast IP, and UDP port |
| Ping to broadcast gets no replies | Hosts or firewalls ignore ICMP broadcast | Host firewall and packet capture | Test with the actual application traffic, such as Wake-on-LAN UDP |
| Wake-on-LAN does not work | Wrong MAC, UDP port, subnet broadcast, or host power/NIC setting | Packet capture on destination subnet | Correct WOL packet, port 7 or 9, host BIOS/NIC settings, and broadcast IP |
| Directed broadcast reaches subnet but target host does not wake | Host is not configured for Wake-on-LAN | Destination host BIOS/NIC/OS settings | Enable WOL in BIOS, NIC driver, and OS power settings |
| Broadcast storm risk exists | Unfiltered directed broadcast enabled | `show running-config interface <DESTINATION_INTERFACE>` | Use ACL-filtered directed broadcast or disable the feature |
| Smurf-style amplification exposure | Any source can send directed broadcasts | `show access-lists` and interface config | Restrict to approved sources or configure `no ip directed-broadcast` |
| Unexpected broadcast traffic appears | Unauthorized source is permitted | `show access-lists ACL-DIRECTED-BCAST` | Tighten ACL and verify deny counters |
| Router CPU rises during test | Excessive broadcast or debug load | `show processes cpu` and `show debugging` | Stop test traffic and run `undebug all` |
| `debug ip packet detail` floods output | Debug is too broad | `show debugging` | Use ACL-limited debugging or avoid debug on busy devices |
| CEF lookup looks like normal unicast | Transit routers forward directed broadcast toward subnet like unicast | `show ip cef <DESTINATION_BROADCAST_IP>` | This is normal; last-hop interface controls final broadcast conversion |
| Broadcast address calculated incorrectly | Wrong subnet mask or broadcast IP | `show ip interface <DESTINATION_INTERFACE>` | Recalculate destination subnet broadcast address |
| Client subnet uses /31 or /32 style addressing | No normal usable subnet broadcast behavior for that segment | Interface addressing check | Do not use directed broadcast for that link type |
| Firewall or ACL blocks inbound directed broadcast before last hop | Security policy drops packet upstream | Path ACL/firewall checks | Permit only the approved source and destination broadcast traffic if required |
| Feature disappears after reload | Configuration was not saved | `show startup-config interface <DESTINATION_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |
| Directed broadcast works in lab but not production | Production hosts, firewalls, or switches suppress broadcast behavior | Packet capture and endpoint firewall checks | Validate the real application path instead of relying on ping |

# Index

IP_Directed_Broadcast_Control.md
IP_Directed_Broadcast_Control
IP_Directed_Broadcast_Control_Mental_Model
IP_Directed_Broadcast_Control_Configuration_Checklist
IP_Directed_Broadcast_Control_Skeleton
IP_Directed_Broadcast_Filtered_Enable_Skeleton
IP_Directed_Broadcast_Lab_Enable_Skeleton
Wake_On_LAN_Directed_Broadcast_Example_Skeleton
IP_Directed_Broadcast_Control_Verification_Commands
Destination_Host_Verification_Commands
IP_Directed_Broadcast_Control_Rollback
IP_Directed_Broadcast_ACL_Rollback
IP_Directed_Broadcast_Control_Failure_Checks