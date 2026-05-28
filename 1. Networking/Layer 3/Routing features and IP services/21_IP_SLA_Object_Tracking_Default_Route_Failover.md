IP_SLA_Object_Tracking_Default_Route_Failover.md
# IP_SLA_Object_Tracking_Default_Route_Failover

# IP_SLA_Object_Tracking_Default_Route_Failover_Mental_Model

| Concept | Operational Meaning |
|---|---|
| IP SLA probe | Router-generated test traffic used to verify reachability beyond simple interface state |
| ICMP echo operation | Ping-style probe sourced from the router toward a target IP |
| Object tracking | A tracking object follows the state of the IP SLA probe |
| Tracked static route | Static route is installed only while the tracking object is up |
| Floating static route | Backup route has a higher administrative distance and installs only when the tracked primary route is removed |
| Default route failover | Primary `0.0.0.0/0` is preferred while the probe succeeds; backup `0.0.0.0/0` takes over when the probe fails |
| Remote failure detection | IP SLA detects upstream or beyond-next-hop failure even when the local interface stays up |
| Source address importance | Probe source must match the primary path being validated |
| Probe target choice | Target should prove the primary path, not just prove the router can ping something over any path |
| Probe pinning | A host route to the probe target through the primary next hop prevents the probe from succeeding through the backup path |
| Track delay | Optional delay prevents route flapping during short probe loss |
| Failback | When the probe succeeds again, the tracked primary route returns and beats the floating backup route |
| Troubleshooting rule | Verify IP SLA statistics, track state, route table installation, CEF path, and actual client traffic path separately |

# IP_SLA_Object_Tracking_Default_Route_Failover_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the primary ISP next hop | Edge RTR | `show ip interface brief` | Primary WAN interface and next-hop IP are known |
| 2 | Identify the backup ISP next hop | Edge RTR | `show ip interface brief` | Backup WAN interface and next-hop IP are known |
| 3 | Confirm primary WAN interface is up | Edge RTR | `show ip interface brief` | Primary WAN interface is `up/up` |
| 4 | Confirm backup WAN interface is up | Edge RTR | `show ip interface brief` | Backup WAN interface is `up/up` |
| 5 | Confirm primary next-hop reachability | Edge RTR | `ping <PRIMARY_NEXT_HOP> source <PRIMARY_WAN_IP_OR_INTERFACE>` | Primary ISP next hop responds |
| 6 | Confirm backup next-hop reachability | Edge RTR | `ping <BACKUP_NEXT_HOP> source <BACKUP_WAN_IP_OR_INTERFACE>` | Backup ISP next hop responds |
| 7 | Choose an IP SLA target that validates the primary path | Edge RTR | `! Example targets: ISP DNS, provider loopback, monitored upstream IP` | Probe target represents primary path health |
| 8 | Confirm the probe target is reachable over the primary path | Edge RTR | `ping <SLA_TARGET_IP> source <PRIMARY_WAN_IP>` | Target replies when sourced from the primary WAN IP |
| 9 | Optional but recommended: pin the SLA target to the primary next hop | Edge RTR | `conf t`<br>`ip route <SLA_TARGET_IP> 255.255.255.255 <PRIMARY_NEXT_HOP>`<br>`end` | Probe to the SLA target stays on the primary path |
| 10 | Create the IP SLA operation | Edge RTR | `conf t`<br>`ip sla <SLA_ID>` | Router enters IP SLA operation mode |
| 11 | Configure ICMP echo probe | Edge RTR | `icmp-echo <SLA_TARGET_IP> source-ip <PRIMARY_WAN_IP>` | Probe sends ICMP echo to the target using the primary WAN source |
| 12 | Configure probe frequency | Edge RTR | `frequency <SECONDS>` | Probe runs at the configured interval |
| 13 | Configure probe timeout | Edge RTR | `timeout <MILLISECONDS>` | Probe fails if no response is received before timeout |
| 14 | Optional: configure RTT threshold | Edge RTR | `threshold <MILLISECONDS>` | Probe records threshold violations when RTT exceeds the configured value |
| 15 | Exit IP SLA operation mode | Edge RTR | `exit` | Router returns to global configuration mode |
| 16 | Schedule the IP SLA operation | Edge RTR | `ip sla schedule <SLA_ID> life forever start-time now` | IP SLA operation starts immediately and runs continuously |
| 17 | Create object tracking for IP SLA reachability | Edge RTR | `track <TRACK_ID> ip sla <SLA_ID> reachability` | Track object follows the success or failure of the IP SLA operation |
| 18 | Optional: add tracking delay to reduce route flapping | Edge RTR | `track <TRACK_ID>`<br>` delay down <DOWN_SECONDS> up <UP_SECONDS>` | Track state changes only after the configured delay |
| 19 | Configure the primary tracked default route | Edge RTR | `ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP> track <TRACK_ID>` | Primary default route installs only when track object is up |
| 20 | Configure the floating backup default route | Edge RTR | `ip route 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>` | Backup default route remains inactive while the primary tracked route is installed |
| 21 | Verify IP SLA configuration | Edge RTR | `show ip sla configuration <SLA_ID>` | Output shows ICMP echo target, source IP, frequency, timeout, and schedule |
| 22 | Verify IP SLA operation statistics | Edge RTR | `show ip sla statistics <SLA_ID>` | Latest return code is `OK` while the primary path is healthy |
| 23 | Verify track object state | Edge RTR | `show track <TRACK_ID>` | Track object is `Up` while the IP SLA operation succeeds |
| 24 | Verify primary default route is installed | Edge RTR | `show ip route 0.0.0.0` | Default route points to the primary next hop while track is up |
| 25 | Verify backup route is configured but not active | Edge RTR | `show running-config | include ^ip route 0.0.0.0` | Both primary tracked and backup floating defaults exist in running config |
| 26 | Verify CEF forwards default traffic toward primary | Edge RTR | `show ip cef 8.8.8.8` | CEF points toward the primary next hop while the tracked route is active |
| 27 | Verify actual traffic follows primary path | Edge RTR | `traceroute <REMOTE_TEST_IP> source <LAN_SOURCE_IP_OR_INTERFACE>` | Path exits through the primary ISP |
| 28 | Test primary path failure in the lab | Edge RTR | `conf t`<br>`interface <PRIMARY_WAN_INTERFACE>`<br>` shutdown`<br>`end` | Primary path is removed or probe fails |
| 29 | Verify IP SLA failure | Edge RTR | `show ip sla statistics <SLA_ID>` | Latest return code shows timeout or failure |
| 30 | Verify track object goes down | Edge RTR | `show track <TRACK_ID>` | Track object changes to `Down` after any configured delay |
| 31 | Verify primary default route is removed | Edge RTR | `show ip route 0.0.0.0` | Tracked primary default route is no longer installed |
| 32 | Verify floating backup default route installs | Edge RTR | `show ip route 0.0.0.0` | Default route now points to the backup next hop |
| 33 | Verify CEF forwards default traffic toward backup | Edge RTR | `show ip cef 8.8.8.8` | CEF points toward the backup next hop |
| 34 | Verify traffic works over backup path | Edge RTR | `ping <REMOTE_TEST_IP> source <LAN_SOURCE_IP_OR_INTERFACE>` | Traffic succeeds over backup if NAT, routing, and return path are correct |
| 35 | Trace backup path | Edge RTR | `traceroute <REMOTE_TEST_IP> source <LAN_SOURCE_IP_OR_INTERFACE>` | Path exits through backup ISP |
| 36 | Restore primary path | Edge RTR | `conf t`<br>`interface <PRIMARY_WAN_INTERFACE>`<br>` no shutdown`<br>`end` | Primary WAN interface returns to `up/up` |
| 37 | Verify IP SLA recovers | Edge RTR | `show ip sla statistics <SLA_ID>` | Latest return code returns to `OK` |
| 38 | Verify track object returns up | Edge RTR | `show track <TRACK_ID>` | Track object changes back to `Up` after any configured delay |
| 39 | Verify primary default route reinstalls | Edge RTR | `show ip route 0.0.0.0` | Default route points back to primary next hop |
| 40 | Verify backup route becomes inactive again | Edge RTR | `show ip route 0.0.0.0` | Floating backup route is no longer the active default |
| 41 | Confirm final forwarding path | Edge RTR | `show ip cef 8.8.8.8` | CEF again points toward the primary next hop |
| 42 | Save the working configuration | Edge RTR | `copy running-config startup-config` | IP SLA, track object, and default routes survive reload |

# IP_SLA_Object_Tracking_Default_Route_Failover_Skeleton

conf t
!
! Optional but recommended so the probe tests the primary path only.
ip route <SLA_TARGET_IP> 255.255.255.255 <PRIMARY_NEXT_HOP>
!
ip sla <SLA_ID>
 icmp-echo <SLA_TARGET_IP> source-ip <PRIMARY_WAN_IP>
 frequency <SECONDS>
 timeout <MILLISECONDS>
 threshold <MILLISECONDS>
exit
!
ip sla schedule <SLA_ID> life forever start-time now
!
track <TRACK_ID> ip sla <SLA_ID> reachability
 delay down <DOWN_SECONDS> up <UP_SECONDS>
!
ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP> track <TRACK_ID>
ip route 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
!
end
write memory

# IP_SLA_Default_Route_Failover_Basic_Example_Skeleton

conf t
!
ip route 192.0.2.3 255.255.255.255 203.0.113.1
!
ip sla 1
 icmp-echo 192.0.2.3 source-ip 203.0.113.2
 frequency 5
 timeout 1000
 threshold 1000
exit
!
ip sla schedule 1 life forever start-time now
!
track 22 ip sla 1 reachability
 delay down 10 up 30
!
ip route 0.0.0.0 0.0.0.0 203.0.113.1 track 22
ip route 0.0.0.0 0.0.0.0 198.51.100.1 4
!
end
write memory

# IP_SLA_Default_Route_Source_Interface_Skeleton

conf t
!
ip route <SLA_TARGET_IP> 255.255.255.255 <PRIMARY_NEXT_HOP>
!
ip sla <SLA_ID>
 icmp-echo <SLA_TARGET_IP> source-interface <PRIMARY_WAN_INTERFACE>
 frequency <SECONDS>
 timeout <MILLISECONDS>
exit
!
ip sla schedule <SLA_ID> life forever start-time now
!
track <TRACK_ID> ip sla <SLA_ID> reachability
!
ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP> track <TRACK_ID>
ip route 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
!
end
write memory

# IP_SLA_Default_Route_VRF_Skeleton

conf t
!
ip route vrf <VRF_NAME> <SLA_TARGET_IP> 255.255.255.255 <PRIMARY_NEXT_HOP>
!
ip sla <SLA_ID>
 icmp-echo <SLA_TARGET_IP> source-ip <PRIMARY_WAN_IP>
 vrf <VRF_NAME>
 frequency <SECONDS>
 timeout <MILLISECONDS>
exit
!
ip sla schedule <SLA_ID> life forever start-time now
!
track <TRACK_ID> ip sla <SLA_ID> reachability
 delay down <DOWN_SECONDS> up <UP_SECONDS>
!
ip route vrf <VRF_NAME> 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP> track <TRACK_ID>
ip route vrf <VRF_NAME> 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
!
end
write memory

# IP_SLA_Object_Tracking_Default_Route_Failover_Verification_Commands

show ip sla application
show ip sla configuration
show ip sla configuration <SLA_ID>
show ip sla statistics
show ip sla statistics <SLA_ID>
show running-config | section ip sla
show running-config | include ip sla schedule
show running-config | section track
show track
show track <TRACK_ID>
show running-config | include ^ip route
show ip route 0.0.0.0
show ip route <SLA_TARGET_IP>
show ip route <PRIMARY_NEXT_HOP>
show ip route <BACKUP_NEXT_HOP>
show ip cef 0.0.0.0
show ip cef <REMOTE_TEST_IP>
show ip cef exact-route <LAN_SOURCE_IP> <REMOTE_TEST_IP>
show ip interface brief
show interfaces <PRIMARY_WAN_INTERFACE>
show interfaces <BACKUP_WAN_INTERFACE>
ping <PRIMARY_NEXT_HOP> source <PRIMARY_WAN_IP_OR_INTERFACE>
ping <BACKUP_NEXT_HOP> source <BACKUP_WAN_IP_OR_INTERFACE>
ping <SLA_TARGET_IP> source <PRIMARY_WAN_IP_OR_INTERFACE>
ping <REMOTE_TEST_IP> source <LAN_SOURCE_IP_OR_INTERFACE>
traceroute <REMOTE_TEST_IP> source <LAN_SOURCE_IP_OR_INTERFACE>
debug ip sla trace <SLA_ID>
show debugging
undebug all

# IP_SLA_Object_Tracking_Default_Route_Failover_Rollback

conf t
!
no ip route 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP> track <TRACK_ID>
no ip route 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
no ip route <SLA_TARGET_IP> 255.255.255.255 <PRIMARY_NEXT_HOP>
!
no track <TRACK_ID>
!
no ip sla schedule <SLA_ID>
no ip sla <SLA_ID>
!
end
write memory

# IP_SLA_Object_Tracking_Default_Route_VRF_Rollback

conf t
!
no ip route vrf <VRF_NAME> 0.0.0.0 0.0.0.0 <PRIMARY_NEXT_HOP> track <TRACK_ID>
no ip route vrf <VRF_NAME> 0.0.0.0 0.0.0.0 <BACKUP_NEXT_HOP> <BACKUP_AD>
no ip route vrf <VRF_NAME> <SLA_TARGET_IP> 255.255.255.255 <PRIMARY_NEXT_HOP>
!
no track <TRACK_ID>
!
no ip sla schedule <SLA_ID>
no ip sla <SLA_ID>
!
end
write memory

# IP_SLA_Failover_Test_Restore

conf t
!
interface <PRIMARY_WAN_INTERFACE>
 no shutdown
!
interface <BACKUP_WAN_INTERFACE>
 no shutdown
!
end

show ip interface brief
show ip sla statistics <SLA_ID>
show track <TRACK_ID>
show ip route 0.0.0.0

# IP_SLA_Object_Tracking_Default_Route_Failover_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| IP SLA operation is configured but inactive | Operation was not scheduled | `show ip sla configuration <SLA_ID>` | Configure `ip sla schedule <SLA_ID> life forever start-time now` |
| IP SLA latest return code is timeout | Probe target is unreachable or ICMP is blocked | `show ip sla statistics <SLA_ID>` | Fix reachability, ICMP filtering, target selection, or timeout |
| Track object stays down | IP SLA operation is failing | `show track <TRACK_ID>` and `show ip sla statistics <SLA_ID>` | Fix the IP SLA target path first |
| Track object stays up during primary ISP failure | Probe is succeeding through the backup path | `show ip route <SLA_TARGET_IP>` | Add a host route to the SLA target through the primary next hop |
| Primary default route never installs | Track object is down or primary next hop is unreachable | `show track <TRACK_ID>` and `show ip route <PRIMARY_NEXT_HOP>` | Restore probe success or primary next-hop reachability |
| Backup default route installs immediately | Primary tracked route is absent or track is down | `show ip route 0.0.0.0` | Fix IP SLA, tracking, or primary next-hop reachability |
| Backup default route never installs | Backup AD is not higher or backup next hop is unreachable | `show ip route <BACKUP_NEXT_HOP>` | Fix backup next-hop reachability and use a valid floating AD |
| Both default routes appear active | Same AD or ECMP was configured unintentionally | `show ip route 0.0.0.0` | Give backup route a higher administrative distance |
| Failover occurs but traffic still fails | Backup path lacks NAT, return routing, ACL permit, or upstream access | `traceroute <REMOTE_TEST_IP>` | Fix backup NAT, routing, ACLs, firewall, or ISP path |
| Failback does not occur | IP SLA target is still unreachable over primary | `ping <SLA_TARGET_IP> source <PRIMARY_WAN_IP>` | Restore primary path, target reachability, or probe host route |
| Route flaps repeatedly | Probe target has intermittent loss or timers are too aggressive | `show ip sla statistics <SLA_ID>` and `show track <TRACK_ID>` | Increase frequency, timeout, or add `delay down` and `delay up` |
| Probe works manually but not in IP SLA | IP SLA source IP or VRF is wrong | `show ip sla configuration <SLA_ID>` | Correct `source-ip`, `source-interface`, or `vrf` |
| Probe target route follows default route | SLA target was not pinned to primary | `show ip route <SLA_TARGET_IP>` | Add `/32` host route to target through primary next hop |
| Probe target is the primary next hop only | Next-hop reachability does not prove internet or upstream path health | `show ip sla configuration <SLA_ID>` | Choose a target beyond the ISP next hop if remote failure detection is needed |
| Remote target blocks ICMP | Target firewall or upstream policy ignores ICMP echo | `ping <SLA_TARGET_IP> source <PRIMARY_WAN_IP>` | Use an approved reachable target or a different IP SLA operation type |
| Default route is removed but CEF still appears wrong briefly | Forwarding table reconvergence lag or stale observation | `show ip cef <REMOTE_TEST_IP>` | Recheck after route change and confirm RIB first |
| LAN clients fail after failover but router ping works | NAT, policy, or return route for LAN traffic is missing on backup path | Test from LAN source and check NAT policy | Add backup NAT and verify return path |
| Local router-generated traffic fails but transit traffic works | Source interface or local policy differs from LAN traffic | `show ip cef exact-route <SOURCE_IP> <REMOTE_TEST_IP>` | Test with correct source and fix routing or NAT per source |
| VRF failover does not work | Non-VRF route or non-VRF IP SLA was configured | `show ip route vrf <VRF_NAME> 0.0.0.0` | Configure IP SLA `vrf` and `ip route vrf` syntax |
| Debug output is too noisy | IP SLA debug left enabled | `show debugging` | Run `undebug all` |
| Configuration disappears after reload | Configuration was not saved | `show startup-config | section ip sla` | Reconfigure and save with `copy running-config startup-config` |

# Index

IP_SLA_Object_Tracking_Default_Route_Failover.md
IP_SLA_Object_Tracking_Default_Route_Failover
IP_SLA_Object_Tracking_Default_Route_Failover_Mental_Model
IP_SLA_Object_Tracking_Default_Route_Failover_Configuration_Checklist
IP_SLA_Object_Tracking_Default_Route_Failover_Skeleton
IP_SLA_Default_Route_Failover_Basic_Example_Skeleton
IP_SLA_Default_Route_Source_Interface_Skeleton
IP_SLA_Default_Route_VRF_Skeleton
IP_SLA_Object_Tracking_Default_Route_Failover_Verification_Commands
IP_SLA_Object_Tracking_Default_Route_Failover_Rollback
IP_SLA_Object_Tracking_Default_Route_VRF_Rollback
IP_SLA_Failover_Test_Restore
IP_SLA_Object_Tracking_Default_Route_Failover_Failure_Checks