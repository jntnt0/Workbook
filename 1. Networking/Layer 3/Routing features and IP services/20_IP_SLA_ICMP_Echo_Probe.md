IP_SLA_ICMP_Echo_Probe.md
# IP_SLA_ICMP_Echo_Probe

# IP_SLA_ICMP_Echo_Probe_Mental_Model

| Concept | Operational Meaning |
|---|---|
| IP SLA | Cisco feature that generates synthetic test traffic from the router to measure reachability or performance |
| ICMP echo probe | IP SLA operation that sends ping-like probes to a target IP address |
| Source router | Device that owns and runs the IP SLA operation |
| Target IP | Destination being tested by the probe |
| Source IP | Local address used as the source of the ICMP echo packets |
| Source control | Source IP or source interface matters when the router has multiple paths, VRFs, or policy routing |
| Frequency | How often the IP SLA operation runs |
| Timeout | How long the operation waits before declaring the probe failed |
| Threshold | Performance boundary used for reporting when round-trip time exceeds an expected value |
| Schedule | IP SLA operation must be scheduled before it actively runs |
| Latest return code | Operational result of the most recent probe, such as `OK` or timeout |
| Statistics | IP SLA records successes, failures, RTT, and operation lifetime |
| Responder boundary | ICMP echo probes do not require an IP SLA responder; the destination only needs to answer ICMP echo |
| Tracking boundary | IP SLA can feed object tracking, but the probe itself only measures reachability |
| Troubleshooting rule | Verify route to target, source IP validity, return path, ACL/CoPP ICMP permissions, and schedule state |

# IP_SLA_ICMP_Echo_Probe_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the probe target | IP SLA Source | `show ip route <TARGET_IP>` | Router has a route toward the target IP |
| 2 | Identify the local source IP or source interface | IP SLA Source | `show ip interface brief` | Source interface is `up/up` and has the intended source IP |
| 3 | Confirm the target is reachable with a normal sourced ping | IP SLA Source | `ping <TARGET_IP> source <SOURCE_IP_OR_INTERFACE>` | Target replies to ICMP echo sourced from the intended address |
| 4 | Confirm return routing from the target side if possible | Target Side | `show ip route <SOURCE_IP>` | Target side has a return path to the IP SLA source |
| 5 | Confirm no existing IP SLA operation uses the planned ID | IP SLA Source | `show ip sla configuration` | Planned operation ID is unused or intentionally being replaced |
| 6 | Confirm supported IP SLA operation types | IP SLA Source | `show ip sla application` | `icmpEcho` appears as a supported operation type |
| 7 | Enter IP SLA operation configuration mode | IP SLA Source | `conf t`<br>`ip sla <OPERATION_ID>` | Router enters IP SLA operation configuration mode |
| 8 | Configure ICMP echo probe with source IP | IP SLA Source | `icmp-echo <TARGET_IP> source-ip <SOURCE_IP>` | Probe sends ICMP echo to target using the selected source IP |
| 9 | Optional: configure ICMP echo probe with source interface instead of source IP | IP SLA Source | `icmp-echo <TARGET_IP> source-interface <SOURCE_INTERFACE>` | Probe sources traffic from the selected interface |
| 10 | Optional: configure probe inside a VRF | IP SLA Source | `vrf <VRF_NAME>` | Probe uses the specified VRF routing table |
| 11 | Configure probe frequency | IP SLA Source | `frequency <SECONDS>` | Probe runs at the configured interval |
| 12 | Optional: configure operation timeout | IP SLA Source | `timeout <MILLISECONDS>` | Probe waits only the configured time before declaring failure |
| 13 | Optional: configure RTT threshold | IP SLA Source | `threshold <MILLISECONDS>` | Operation can report threshold violations when RTT exceeds the value |
| 14 | Optional: add an owner label | IP SLA Source | `owner <OWNER_NAME>` | Operation has an administrative owner label |
| 15 | Optional: add a tag label | IP SLA Source | `tag <TAG_TEXT>` | Operation has a descriptive tag |
| 16 | Exit IP SLA operation configuration | IP SLA Source | `exit` | Router returns to global configuration mode |
| 17 | Schedule the IP SLA operation to start now and run forever | IP SLA Source | `ip sla schedule <OPERATION_ID> life forever start-time now` | Operation becomes active immediately and does not expire |
| 18 | Optional: schedule the operation with a delayed start | IP SLA Source | `ip sla schedule <OPERATION_ID> life forever start-time <HH:MM>` | Operation starts at the configured time |
| 19 | Verify the operation configuration | IP SLA Source | `show ip sla configuration <OPERATION_ID>` | Output shows ICMP echo target, source, frequency, timeout, threshold, and schedule |
| 20 | Verify operation statistics | IP SLA Source | `show ip sla statistics <OPERATION_ID>` | Latest return code, RTT, successes, failures, and lifetime are visible |
| 21 | Verify the latest return code is healthy | IP SLA Source | `show ip sla statistics <OPERATION_ID>` | Latest return code is `OK` when the probe succeeds |
| 22 | Verify repeated success over time | IP SLA Source | `show ip sla statistics <OPERATION_ID>` | Number of successes increases and failures do not continuously rise |
| 23 | Verify routing for the exact source and target path | IP SLA Source | `show ip cef exact-route <SOURCE_IP> <TARGET_IP>` | CEF chooses the expected egress path for the probe |
| 24 | Optional: debug the specific IP SLA operation in a lab | IP SLA Source | `debug ip sla trace <OPERATION_ID>` | Debug shows probe start, source, target, response, or timeout |
| 25 | Disable debugging after testing | IP SLA Source | `undebug all` | Debugging is disabled |
| 26 | Save the working configuration | IP SLA Source | `copy running-config startup-config` | IP SLA probe configuration survives reload |

# IP_SLA_ICMP_Echo_Probe_Skeleton

conf t
!
ip sla <OPERATION_ID>
 icmp-echo <TARGET_IP> source-ip <SOURCE_IP>
 frequency <SECONDS>
 timeout <MILLISECONDS>
 threshold <MILLISECONDS>
 owner <OWNER_NAME>
 tag <TAG_TEXT>
exit
!
ip sla schedule <OPERATION_ID> life forever start-time now
!
end
write memory

# IP_SLA_ICMP_Echo_Source_Interface_Skeleton

conf t
!
ip sla <OPERATION_ID>
 icmp-echo <TARGET_IP> source-interface <SOURCE_INTERFACE>
 frequency <SECONDS>
 timeout <MILLISECONDS>
 threshold <MILLISECONDS>
 tag <TAG_TEXT>
exit
!
ip sla schedule <OPERATION_ID> life forever start-time now
!
end
write memory

# IP_SLA_ICMP_Echo_VRF_Skeleton

conf t
!
ip sla <OPERATION_ID>
 icmp-echo <TARGET_IP> source-ip <SOURCE_IP>
 vrf <VRF_NAME>
 frequency <SECONDS>
 timeout <MILLISECONDS>
 threshold <MILLISECONDS>
 tag <TAG_TEXT>
exit
!
ip sla schedule <OPERATION_ID> life forever start-time now
!
end
write memory

# IP_SLA_ICMP_Echo_Basic_Example_Skeleton

conf t
!
ip sla 10
 icmp-echo 10.1.100.100 source-ip 192.168.1.11
 frequency 15
 timeout 5000
 threshold 5000
 tag ICMP_ECHO_REACHABILITY_TEST
exit
!
ip sla schedule 10 life forever start-time now
!
end
write memory

# IP_SLA_ICMP_Echo_Probe_Verification_Commands

show ip sla application
show ip sla configuration
show ip sla configuration <OPERATION_ID>
show ip sla statistics
show ip sla statistics <OPERATION_ID>
show running-config | section ip sla
show running-config | include ip sla schedule
show ip interface brief
show ip route <TARGET_IP>
show ip route <SOURCE_IP>
show ip cef <TARGET_IP>
show ip cef exact-route <SOURCE_IP> <TARGET_IP>
show access-lists
show control-plane
ping <TARGET_IP> source <SOURCE_IP_OR_INTERFACE>
traceroute <TARGET_IP> source <SOURCE_IP>
debug ip sla trace <OPERATION_ID>
show debugging
undebug all

# IP_SLA_ICMP_Echo_Probe_Rollback

conf t
!
no ip sla schedule <OPERATION_ID>
no ip sla <OPERATION_ID>
!
end
write memory

# IP_SLA_ICMP_Echo_Debug_Rollback

undebug all

# IP_SLA_ICMP_Echo_Probe_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Operation is configured but not running | IP SLA operation was not scheduled | `show ip sla configuration <OPERATION_ID>` | Configure `ip sla schedule <OPERATION_ID> life forever start-time now` |
| Latest return code shows timeout | Target did not reply before timeout | `show ip sla statistics <OPERATION_ID>` | Verify target reachability, return path, ICMP filtering, and timeout value |
| Normal ping works but IP SLA fails | Wrong source IP or source interface in the IP SLA operation | `show ip sla configuration <OPERATION_ID>` | Correct `source-ip` or `source-interface` |
| IP SLA probe exits the wrong path | Source IP, VRF, routing, or PBR causes unexpected path selection | `show ip cef exact-route <SOURCE_IP> <TARGET_IP>` | Correct source, VRF, route, or PBR policy |
| Probe fails only in a VRF | VRF missing route to target or return path | `show ip route vrf <VRF_NAME> <TARGET_IP>` | Add or repair VRF route and return route |
| Probe target is reachable from router but not from selected source | Source interface subnet lacks return route | `ping <TARGET_IP> source <SOURCE_IP_OR_INTERFACE>` | Fix return routing to the selected source IP |
| Operation ID does not appear | Wrong operation ID or operation was deleted | `show ip sla configuration` | Recreate the IP SLA operation with the correct ID |
| Operation remains inactive | Schedule missing, expired, or start time not reached | `show ip sla configuration <OPERATION_ID>` | Correct schedule life and start time |
| Successes increase but RTT is too high | Path is reachable but performance is poor | `show ip sla statistics <OPERATION_ID>` | Investigate congestion, QoS, path selection, or target load |
| Threshold violations occur | RTT exceeds configured threshold | `show ip sla statistics <OPERATION_ID>` | Adjust threshold or fix latency problem |
| Failures increment intermittently | Packet loss, congestion, policing, or unstable target | `show ip sla statistics <OPERATION_ID>` | Check interface drops, QoS, CoPP, ACLs, and target stability |
| ICMP echo is blocked | ACL, firewall, or CoPP drops ICMP echo or echo reply | `show access-lists` and control-plane policy counters | Permit required ICMP echo and echo-reply for the probe path |
| Target does not answer pings by policy | Destination host or firewall ignores ICMP | External host test or packet capture | Choose an approved target that responds to ICMP or use a different IP SLA operation |
| Probe uses stale or wrong route | Routing table changed after configuration | `show ip route <TARGET_IP>` | Fix routing or choose the intended source interface |
| Probe result is OK but application still fails | ICMP reachability does not prove application health | Application test or TCP-based IP SLA operation | Use TCP connect, HTTP, DNS, or UDP-jitter operation as appropriate |
| IP SLA responder is missing | Misunderstanding of ICMP echo operation | `show ip sla configuration <OPERATION_ID>` | ICMP echo does not require an IP SLA responder; verify the target answers ICMP |
| Debug output shows timeout | No ICMP echo reply received | `debug ip sla trace <OPERATION_ID>` | Fix target reachability, return path, or ICMP filtering |
| Debug output is too noisy | Debug left enabled during repeated probes | `show debugging` | Run `undebug all` |
| Frequency is too aggressive | Probe load is too high for device or network | `show ip sla configuration <OPERATION_ID>` | Increase `frequency` or reduce the number of operations |
| Timeout is longer than frequency | New probe scheduling can become inefficient or misleading | `show ip sla configuration <OPERATION_ID>` | Keep timeout lower than frequency in normal designs |
| Operation works before reload but disappears afterward | Configuration was not saved | `show startup-config | section ip sla` | Reconfigure and save with `copy running-config startup-config` |
| Legacy IOS syntax differs | Older image uses `ip sla monitor` syntax | `ip sla ?` | Use platform-supported IP SLA syntax for that image |

# Index

IP_SLA_ICMP_Echo_Probe.md
IP_SLA_ICMP_Echo_Probe
IP_SLA_ICMP_Echo_Probe_Mental_Model
IP_SLA_ICMP_Echo_Probe_Configuration_Checklist
IP_SLA_ICMP_Echo_Probe_Skeleton
IP_SLA_ICMP_Echo_Source_Interface_Skeleton
IP_SLA_ICMP_Echo_VRF_Skeleton
IP_SLA_ICMP_Echo_Basic_Example_Skeleton
IP_SLA_ICMP_Echo_Probe_Verification_Commands
IP_SLA_ICMP_Echo_Probe_Rollback
IP_SLA_ICMP_Echo_Debug_Rollback
IP_SLA_ICMP_Echo_Probe_Failure_Checks
