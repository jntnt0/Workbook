Performance_Monitor_Flow_Visibility.md
# Performance_Monitor_Flow_Visibility

# Performance_Monitor_Flow_Visibility_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Flow visibility | The device records who talked to whom, using what protocol or port, through which interface, and how much traffic was seen |
| Flexible NetFlow | Modular flow telemetry feature built from flow records, flow exporters, and flow monitors |
| Flow record | Defines what fields identify a flow and what counters or metadata are collected |
| Match fields | Fields that define flow uniqueness, such as source IP, destination IP, protocol, ports, and input interface |
| Collect fields | Extra information gathered for each flow, such as byte counters, packet counters, timestamps, TCP flags, and interface IDs |
| Flow exporter | Defines where flow records are sent, usually a NetFlow collector using UDP |
| Flow monitor | Combines a flow record, cache behavior, and optional exporter |
| Flow cache | Local table of observed flows before they age out, export, or are cleared |
| Interface attachment | Flexible NetFlow collects nothing until the monitor is applied to an interface in input or output direction |
| Input monitoring | Captures traffic as it enters the interface |
| Output monitoring | Captures traffic as it exits the interface if platform supports that direction |
| Cache timers | Active and inactive timers control when flow data is expired or exported |
| Collector dependency | Exported telemetry requires reachability to the collector and the correct UDP port and NetFlow version |
| Local-only visibility | A flow monitor can still be useful locally even without a collector by inspecting the flow cache |
| Troubleshooting rule | Verify record, exporter, monitor, interface attachment, cache entries, and collector reachability separately |

# Performance_Monitor_Flow_Visibility_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces and traffic path before monitoring | RTR/L3SW | `show ip interface brief` | Monitored interfaces are `up/up` |
| 2 | Confirm the expected forwarding path for test traffic | RTR/L3SW | `show ip route <DESTINATION_IP>` | Router has a valid route to the destination |
| 3 | Confirm CEF path for the tested source and destination | RTR/L3SW | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Egress path is known before telemetry is enabled |
| 4 | Confirm current flow configuration | RTR/L3SW | `show running-config | section flow` | Existing flow records, exporters, and monitors are known |
| 5 | Confirm platform supports Flexible NetFlow | RTR/L3SW | `flow record ?`<br>`flow monitor ?`<br>`flow exporter ?` | CLI supports Flexible NetFlow objects |
| 6 | Create a custom IPv4 flow record | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>`end` | Custom flow record exists |
| 7 | Match IPv4 source address | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` match ipv4 source address`<br>`end` | Source IP becomes part of the flow key |
| 8 | Match IPv4 destination address | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` match ipv4 destination address`<br>`end` | Destination IP becomes part of the flow key |
| 9 | Match Layer 4 protocol | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` match ip protocol`<br>`end` | IP protocol becomes part of the flow key |
| 10 | Match transport source port | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` match transport source-port`<br>`end` | Source TCP/UDP port becomes part of the flow key |
| 11 | Match transport destination port | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` match transport destination-port`<br>`end` | Destination TCP/UDP port becomes part of the flow key |
| 12 | Match input interface | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` match interface input`<br>`end` | Input interface becomes part of the flow key |
| 13 | Collect packet counter | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` collect counter packets long`<br>`end` | Flow cache records packet count |
| 14 | Collect byte counter | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` collect counter bytes long`<br>`end` | Flow cache records byte count |
| 15 | Collect first and last timestamps | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` collect timestamp sys-uptime first`<br>` collect timestamp sys-uptime last`<br>`end` | Flow cache records first-seen and last-seen times |
| 16 | Optional: collect TCP flags | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` collect transport tcp flags`<br>`end` | Flow records include TCP flag information |
| 17 | Optional: collect output interface | RTR/L3SW | `conf t`<br>`flow record FR-IPV4-TRAFFIC`<br>` collect interface output`<br>`end` | Flow records include output interface where supported |
| 18 | Verify the custom flow record | RTR/L3SW | `show flow record FR-IPV4-TRAFFIC` | Record shows intended match and collect fields |
| 19 | Create a flow exporter if sending to a collector | RTR/L3SW | `conf t`<br>`flow exporter FE-NETFLOW-COLLECTOR`<br>`end` | Exporter object exists |
| 20 | Configure collector destination | RTR/L3SW | `conf t`<br>`flow exporter FE-NETFLOW-COLLECTOR`<br>` destination <COLLECTOR_IP>`<br>`end` | Exporter points to the collector IP |
| 21 | Configure exporter source interface | RTR/L3SW | `conf t`<br>`flow exporter FE-NETFLOW-COLLECTOR`<br>` source <SOURCE_INTERFACE>`<br>`end` | Export packets use the selected source interface |
| 22 | Configure export transport port | RTR/L3SW | `conf t`<br>`flow exporter FE-NETFLOW-COLLECTOR`<br>` transport udp <COLLECTOR_UDP_PORT>`<br>`end` | Exporter sends NetFlow records to the collector UDP port |
| 23 | Configure NetFlow export version | RTR/L3SW | `conf t`<br>`flow exporter FE-NETFLOW-COLLECTOR`<br>` export-protocol netflow-v9`<br>`end` | Exporter uses NetFlow Version 9 |
| 24 | Optional: configure template refresh timeout | RTR/L3SW | `conf t`<br>`flow exporter FE-NETFLOW-COLLECTOR`<br>` template data timeout <SECONDS>`<br>`end` | Collector receives template refreshes at the configured interval |
| 25 | Verify exporter configuration | RTR/L3SW | `show flow exporter FE-NETFLOW-COLLECTOR` | Exporter shows destination, source, protocol, port, and template settings |
| 26 | Confirm collector reachability | RTR/L3SW | `ping <COLLECTOR_IP> source <SOURCE_INTERFACE_OR_IP>` | Collector is reachable from the export source |
| 27 | Create a flow monitor | RTR/L3SW | `conf t`<br>`flow monitor FM-IPV4-TRAFFIC`<br>`end` | Flow monitor exists |
| 28 | Add a clear monitor description | RTR/L3SW | `conf t`<br>`flow monitor FM-IPV4-TRAFFIC`<br>` description IPV4_TRAFFIC_VISIBILITY`<br>`end` | Monitor purpose is documented |
| 29 | Attach the custom flow record to the monitor | RTR/L3SW | `conf t`<br>`flow monitor FM-IPV4-TRAFFIC`<br>` record FR-IPV4-TRAFFIC`<br>`end` | Monitor uses the custom flow record |
| 30 | Attach exporter to the monitor if exporting to a collector | RTR/L3SW | `conf t`<br>`flow monitor FM-IPV4-TRAFFIC`<br>` exporter FE-NETFLOW-COLLECTOR`<br>`end` | Monitor exports cached flows to the collector |
| 31 | Configure inactive cache timeout | RTR/L3SW | `conf t`<br>`flow monitor FM-IPV4-TRAFFIC`<br>` cache timeout inactive <SECONDS>`<br>`end` | Inactive flows age out after the configured idle period |
| 32 | Configure active cache timeout | RTR/L3SW | `conf t`<br>`flow monitor FM-IPV4-TRAFFIC`<br>` cache timeout active <SECONDS>`<br>`end` | Long-lived flows are exported or refreshed at the configured interval |
| 33 | Verify flow monitor configuration | RTR/L3SW | `show flow monitor FM-IPV4-TRAFFIC` | Monitor shows record, exporter, cache type, and timers |
| 34 | Apply flow monitor inbound on the target interface | RTR/L3SW | `conf t`<br>`interface <MONITORED_INTERFACE>`<br>` ip flow monitor FM-IPV4-TRAFFIC input`<br>`end` | Interface begins collecting ingress IPv4 flows |
| 35 | Optional: apply flow monitor outbound if supported and needed | RTR/L3SW | `conf t`<br>`interface <MONITORED_INTERFACE>`<br>` ip flow monitor FM-IPV4-TRAFFIC output`<br>`end` | Interface begins collecting egress IPv4 flows if platform supports it |
| 36 | Verify monitor attachment on the interface | RTR/L3SW | `show running-config interface <MONITORED_INTERFACE>` | Interface shows `ip flow monitor FM-IPV4-TRAFFIC input` or output |
| 37 | Verify Flexible NetFlow interface status | RTR/L3SW | `show flow interface <MONITORED_INTERFACE>` | Interface shows the applied flow monitor and direction |
| 38 | Generate test traffic across the monitored interface | Host/RTR | `ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>` | Traffic crosses the monitored interface |
| 39 | Generate TCP or UDP test traffic for richer flow entries | Host | `curl <URL>`<br>`iperf3 -c <SERVER_IP>` | Flow cache includes protocol and port-based entries |
| 40 | Verify flow cache entries | RTR/L3SW | `show flow monitor FM-IPV4-TRAFFIC cache` | Cache shows observed flows |
| 41 | Verify cache in record format | RTR/L3SW | `show flow monitor name FM-IPV4-TRAFFIC cache format record` | Flow entries show match fields and collected counters |
| 42 | Verify exporter statistics | RTR/L3SW | `show flow exporter FE-NETFLOW-COLLECTOR statistics` | Exporter shows packets and bytes exported, with no unexpected drops |
| 43 | Verify collector receives flows | Collector | `! Check collector dashboard or NetFlow listener for router source IP` | Collector receives records from the device |
| 44 | Optional: clear flow cache before a controlled test | RTR/L3SW | `clear flow monitor FM-IPV4-TRAFFIC cache` | Old cache entries are cleared |
| 45 | Retest controlled traffic after clearing cache | Host/RTR | `ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>` | New flow entries appear for the controlled test |
| 46 | Verify counters rise on the monitored interface | RTR/L3SW | `show interfaces <MONITORED_INTERFACE>` | Interface counters increase during test traffic |
| 47 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | Flow visibility configuration survives reload |

# Performance_Monitor_Flow_Visibility_Skeleton

conf t
!
flow record FR-IPV4-TRAFFIC
 description IPV4_5_TUPLE_TRAFFIC_RECORD
 match ipv4 source address
 match ipv4 destination address
 match ip protocol
 match transport source-port
 match transport destination-port
 match interface input
 collect interface output
 collect counter packets long
 collect counter bytes long
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last
 collect transport tcp flags
!
flow exporter FE-NETFLOW-COLLECTOR
 description EXPORT_TO_NETFLOW_COLLECTOR
 destination <COLLECTOR_IP>
 source <SOURCE_INTERFACE>
 transport udp <COLLECTOR_UDP_PORT>
 export-protocol netflow-v9
 template data timeout <SECONDS>
!
flow monitor FM-IPV4-TRAFFIC
 description IPV4_TRAFFIC_VISIBILITY
 record FR-IPV4-TRAFFIC
 exporter FE-NETFLOW-COLLECTOR
 cache timeout inactive <INACTIVE_SECONDS>
 cache timeout active <ACTIVE_SECONDS>
!
interface <MONITORED_INTERFACE>
 ip flow monitor FM-IPV4-TRAFFIC input
!
end
write memory

# Local_Only_Flow_Visibility_Skeleton

conf t
!
flow record FR-IPV4-LOCAL
 match ipv4 source address
 match ipv4 destination address
 match ip protocol
 match transport source-port
 match transport destination-port
 match interface input
 collect counter packets long
 collect counter bytes long
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last
!
flow monitor FM-IPV4-LOCAL
 description LOCAL_CACHE_ONLY_FLOW_VISIBILITY
 record FR-IPV4-LOCAL
 cache timeout inactive 15
 cache timeout active 60
!
interface <MONITORED_INTERFACE>
 ip flow monitor FM-IPV4-LOCAL input
!
end
write memory

# Predefined_Record_Flexible_NetFlow_Skeleton

conf t
!
flow exporter FE-NETFLOW-COLLECTOR
 destination <COLLECTOR_IP>
 source <SOURCE_INTERFACE>
 transport udp <COLLECTOR_UDP_PORT>
 export-protocol netflow-v9
!
flow monitor FM-IPV4-PREDEFINED
 description PREDEFINED_IPV4_ORIGINAL_INPUT_RECORD
 record netflow ipv4 original-input
 exporter FE-NETFLOW-COLLECTOR
 cache timeout inactive 15
 cache timeout active 60
!
interface <MONITORED_INTERFACE>
 ip flow monitor FM-IPV4-PREDEFINED input
!
end
write memory

# IPv6_Flow_Visibility_Skeleton

conf t
!
flow record FR-IPV6-TRAFFIC
 description IPV6_TRAFFIC_RECORD
 match ipv6 source address
 match ipv6 destination address
 match ipv6 protocol
 match transport source-port
 match transport destination-port
 match interface input
 collect interface output
 collect counter packets long
 collect counter bytes long
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last
!
flow monitor FM-IPV6-TRAFFIC
 description IPV6_TRAFFIC_VISIBILITY
 record FR-IPV6-TRAFFIC
 cache timeout inactive 15
 cache timeout active 60
!
interface <MONITORED_INTERFACE>
 ipv6 flow monitor FM-IPV6-TRAFFIC input
!
end
write memory

# Sampled_Flow_Visibility_Skeleton

conf t
!
sampler SAMPLER-1-OUT-OF-100
 mode random 1 out-of 100
!
flow record FR-IPV4-SAMPLED
 match ipv4 source address
 match ipv4 destination address
 match ip protocol
 match transport source-port
 match transport destination-port
 match interface input
 collect counter packets long
 collect counter bytes long
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last
!
flow monitor FM-IPV4-SAMPLED
 description SAMPLED_IPV4_FLOW_VISIBILITY
 record FR-IPV4-SAMPLED
 cache timeout inactive 15
 cache timeout active 60
!
interface <MONITORED_INTERFACE>
 ip flow monitor FM-IPV4-SAMPLED sampler SAMPLER-1-OUT-OF-100 input
!
end
write memory

# Performance_Monitor_Flow_Visibility_Verification_Commands

show ip interface brief
show ip route <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
show running-config | section flow
show running-config flow record
show running-config flow exporter
show running-config flow monitor
show running-config interface <MONITORED_INTERFACE>
show flow record
show flow record FR-IPV4-TRAFFIC
show flow exporter
show flow exporter FE-NETFLOW-COLLECTOR
show flow exporter FE-NETFLOW-COLLECTOR statistics
show flow monitor
show flow monitor FM-IPV4-TRAFFIC
show flow monitor FM-IPV4-TRAFFIC cache
show flow monitor name FM-IPV4-TRAFFIC cache format record
show flow interface
show flow interface <MONITORED_INTERFACE>
show sampler
show interfaces <MONITORED_INTERFACE>
show access-lists
ping <COLLECTOR_IP> source <SOURCE_INTERFACE_OR_IP>
ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>
traceroute <DESTINATION_IP> source <SOURCE_IP>
clear flow monitor FM-IPV4-TRAFFIC cache

# Collector_Verification_Commands

show listening UDP port <COLLECTOR_UDP_PORT>
confirm NetFlow v9 template received
confirm exporter source IP is <SOURCE_INTERFACE_IP>
confirm flow records appear for <SOURCE_IP> to <DESTINATION_IP>
confirm protocol, source port, destination port, byte count, and packet count fields are populated

# Performance_Monitor_Flow_Visibility_Rollback

conf t
!
interface <MONITORED_INTERFACE>
 no ip flow monitor FM-IPV4-TRAFFIC input
 no ip flow monitor FM-IPV4-TRAFFIC output
 no ipv6 flow monitor FM-IPV6-TRAFFIC input
 no ipv6 flow monitor FM-IPV6-TRAFFIC output
!
no flow monitor FM-IPV4-TRAFFIC
no flow monitor FM-IPV4-LOCAL
no flow monitor FM-IPV4-PREDEFINED
no flow monitor FM-IPV6-TRAFFIC
no flow monitor FM-IPV4-SAMPLED
!
no flow exporter FE-NETFLOW-COLLECTOR
!
no flow record FR-IPV4-TRAFFIC
no flow record FR-IPV4-LOCAL
no flow record FR-IPV6-TRAFFIC
no flow record FR-IPV4-SAMPLED
!
no sampler SAMPLER-1-OUT-OF-100
!
end
write memory

# Performance_Monitor_Flow_Cache_Reset

clear flow monitor FM-IPV4-TRAFFIC cache
clear flow monitor FM-IPV4-LOCAL cache
clear flow monitor FM-IPV4-PREDEFINED cache
clear flow monitor FM-IPV6-TRAFFIC cache
clear flow monitor FM-IPV4-SAMPLED cache

# Performance_Monitor_Flow_Visibility_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No flows appear in cache | Monitor is not applied to an interface | `show flow interface` | Apply `ip flow monitor <MONITOR_NAME> input` or output on the correct interface |
| Monitor is applied but cache is empty | No matching traffic is crossing that interface and direction | `show interfaces <MONITORED_INTERFACE>` | Generate traffic through the monitored interface and verify direction |
| Ingress flows appear but egress flows do not | Platform may not support output monitoring or traffic does not exit that interface | `show flow interface <MONITORED_INTERFACE>` | Use input monitoring on the correct ingress interface or verify platform support |
| Flow monitor command is rejected | Record fields are unsupported for that platform or direction | `show flow record <RECORD_NAME>` | Remove unsupported fields or use a predefined record |
| Exporter sends nothing | Monitor has no exporter or no flows are expiring | `show flow monitor <MONITOR_NAME>` and `show flow exporter <EXPORTER_NAME> statistics` | Attach exporter to monitor and tune active or inactive timers |
| Collector receives no flows | Collector IP, source, UDP port, routing, or firewall is wrong | `show flow exporter <EXPORTER_NAME>` and `ping <COLLECTOR_IP> source <SOURCE_INTERFACE_OR_IP>` | Correct exporter destination, source interface, UDP port, route, or firewall |
| Collector rejects or cannot decode records | Export version or templates are wrong or missing | Collector logs and `show flow exporter <EXPORTER_NAME>` | Use `export-protocol netflow-v9` and configure template refresh |
| Flows show wrong source interface | Monitor applied on wrong interface or wrong direction | `show flow interface` | Move monitor to the interface where traffic actually enters |
| Flows show unexpected destination path | Routing, PBR, NAT, or ECMP changes forwarding | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Verify exact path and monitor the correct ingress or egress point |
| Flow count is too high | Match fields are too granular or traffic volume is high | `show flow monitor <MONITOR_NAME>` | Use sampling, fewer match fields, or a collector sized for the load |
| Cache fills and drops new flows | Cache size or timeout behavior is wrong for traffic volume | `show flow monitor <MONITOR_NAME> cache` | Tune active and inactive timers, use sampling, or reduce monitored scope |
| CPU or memory impact rises | Flow monitoring is too broad or unsampled on a busy interface | `show processes cpu` and platform resource checks | Use sampling, restrict interfaces, or reduce record complexity |
| Sampled data seems incomplete | Sampling intentionally records only selected packets | `show sampler` | Use unsampled monitoring for short lab tests or adjust sampler rate |
| Byte and packet counters look too low | Cache has not aged, sampling is enabled, or traffic path differs | `show flow monitor name <MONITOR_NAME> cache format record` | Clear cache, retest with known traffic, and verify sampler status |
| Application visibility is missing | Basic Flexible NetFlow record does not classify applications | `show flow record <RECORD_NAME>` | Add supported NBAR/application fields only if the platform supports AVC |
| TCP flags are missing | Flow record does not collect TCP flags | `show flow record <RECORD_NAME>` | Add `collect transport tcp flags` |
| Output interface is missing | Flow record does not collect output interface or platform does not support it | `show flow record <RECORD_NAME>` | Add `collect interface output` where supported |
| IPv6 traffic is not monitored | IPv4 monitor applied only | `show running-config interface <MONITORED_INTERFACE>` | Apply `ipv6 flow monitor <MONITOR_NAME> input` with an IPv6 flow record |
| NAT makes flow records confusing | Monitoring point is before or after translation | `show ip nat translations` | Monitor the intended side of the NAT boundary and document pre-NAT or post-NAT view |
| Tunnel traffic hides inner flows | Monitor sees outer tunnel headers only | `show running-config interface <TUNNEL_INTERFACE>` | Monitor inside tunnel or before encapsulation if inner-flow visibility is required |
| Export works locally but not across VRF or management network | Exporter source or route uses wrong table or interface | `show ip route <COLLECTOR_IP>` | Correct source interface, routing, VRF design, or management path |
| ACL blocks exporter packets | UDP export port blocked | `show access-lists` | Permit exporter source IP to collector UDP port |
| Flow cache clears too quickly | Inactive timer too short or immediate cache behavior used | `show flow monitor <MONITOR_NAME>` | Increase inactive timer or avoid immediate cache unless required |
| Long-lived flows do not export often enough | Active timer too long | `show flow monitor <MONITOR_NAME>` | Lower `cache timeout active <SECONDS>` |
| Local cache works but collector shows delayed data | Cache timers delay export | `show flow monitor <MONITOR_NAME>` | Tune active and inactive timers |
| Monitor survives but exporter disappears after reload | Configuration not saved or object deleted | `show startup-config | section flow` | Reconfigure and save with `copy running-config startup-config` |
| Temporary test data pollutes cache | Old flows remain from prior test | `show flow monitor <MONITOR_NAME> cache` | Clear the flow monitor cache before controlled tests |
| Rollback leaves collection active | Interface attachment was not removed | `show flow interface` | Remove `ip flow monitor` from all monitored interfaces before deleting objects |

# Index

Performance_Monitor_Flow_Visibility.md
Performance_Monitor_Flow_Visibility
Performance_Monitor_Flow_Visibility_Mental_Model
Performance_Monitor_Flow_Visibility_Configuration_Checklist
Performance_Monitor_Flow_Visibility_Skeleton
Local_Only_Flow_Visibility_Skeleton
Predefined_Record_Flexible_NetFlow_Skeleton
IPv6_Flow_Visibility_Skeleton
Sampled_Flow_Visibility_Skeleton
Performance_Monitor_Flow_Visibility_Verification_Commands
Collector_Verification_Commands
Performance_Monitor_Flow_Visibility_Rollback
Performance_Monitor_Flow_Cache_Reset
Performance_Monitor_Flow_Visibility_Failure_Checks