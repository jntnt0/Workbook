
CLI syntax for configured gRPC telemetry was checked against Cisco’s IOS XE 17.18 programmability guide because the local KB source has some examples embedded as images. Cisco confirms that gRPC is used for configured subscriptions, uses kvGPB encoding, and shows the telemetry ietf subscription ... receiver ip address ... protocol grpc-tcp pattern.  

09_IOS_XE_gRPC_Model_Driven_Telemetry.md

09_IOS_XE_gRPC_Model_Driven_Telemetry

# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Model-driven telemetry | IOS XE streams structured YANG-modeled data to a collector instead of waiting for polling or parsing CLI output |
| Publisher | The IOS XE device that sends telemetry updates |
| Receiver | The collector that listens for telemetry data, such as YANG Suite, Telegraf, or another gRPC collector |
| Controller | The system that creates subscriptions. It may be the same system as the receiver, but does not have to be |
| Subscriber | The system that requests a dynamic subscription from the device |
| Configured subscription | Persistent dial-out subscription stored in device configuration |
| Dynamic subscription | Temporary dial-in subscription created by an external client through NETCONF or another supported management session |
| Dial-out | IOS XE initiates the connection to the collector |
| Dial-in | The external client connects to IOS XE and requests telemetry on that same session |
| gRPC telemetry | A dial-out telemetry transport where IOS XE connects to the receiver over gRPC |
| `grpc-tcp` | Cleartext gRPC transport used in lab telemetry testing |
| `grpc-tls` | Encrypted gRPC transport that requires TLS profile and trustpoint planning |
| `yang-push` | Common telemetry stream for YANG datastore updates |
| XPath filter | The exact YANG subtree to stream, such as `/memory-ios-xe-oper:memory-statistics/memory-statistic` |
| Periodic update | IOS XE sends data at a configured interval whether the value changed or not |
| On-change update | IOS XE sends data when the subscribed object changes, if the subtree supports on-change |
| `encode-kvgpb` | Key-value Google Protocol Buffers encoding. Required for `grpc-tcp` telemetry |
| `source-vrf` | VRF used by IOS XE to reach the telemetry collector |
| `source-address` | Source IP used by IOS XE when building the collector connection |
| Receiver state | Valid subscription does not guarantee active collector delivery. The receiver connection must also be active |
| Correct workflow | Start collector, confirm reachability, pick YANG path, configure subscription, verify subscription validity, verify receiver connection, verify collector data |
| Common mistake | Seeing `Valid` in `show telemetry ietf subscription all` and assuming data is arriving. You still need `show telemetry internal connection` and collector-side verification |
| Related labs | `grpc-final` |
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm IOS XE management or telemetry source interface state | IOS XE publisher | `show ip interface brief` | Source interface has expected IP and is up/up |
| 2 | Confirm the IOS XE publisher has a route to the collector | IOS XE publisher | `show ip route <COLLECTOR_IP>` | Route exists toward the telemetry receiver |
| 3 | Confirm collector reachability from the publisher | IOS XE publisher | `ping <COLLECTOR_IP> source <SOURCE_IP>` | Ping succeeds from the intended source address |
| 4 | Confirm collector port reachability from the publisher | IOS XE publisher | `telnet <COLLECTOR_IP> <COLLECTOR_PORT>` | TCP connection opens, or collector shows inbound attempt |
| 5 | Start the gRPC telemetry receiver before configuring or testing the subscription | Collector | `<START_COLLECTOR_OR_YANG_SUITE_RECEIVER>` | Collector is listening on `<COLLECTOR_IP>:<COLLECTOR_PORT>` |
| 6 | Identify the operational data to stream | Operator | `Target data: <CPU | memory | interface counters | VLAN state | other>` | Telemetry objective is specific before XPath selection |
| 7 | Identify the YANG module | Operator or YANG tool | `<MODULE_NAME>` | Module matches the operational data being streamed |
| 8 | Identify the XPath filter | Operator or YANG tool | `<XPATH_FILTER>` | XPath points to one valid YANG subtree |
| 9 | Validate XPath syntax before entering CLI | Operator | `Check for format like /prefix:path/to/node` | XPath has no extra slash after the module prefix |
| 10 | Enter global configuration mode | IOS XE publisher | `configure terminal` | Device enters global config mode |
| 11 | Enable NETCONF/YANG if YANG Suite or NETCONF RPC workflow is used to create the subscription | IOS XE publisher | `netconf-yang` | NETCONF/YANG process is enabled for YANG Suite workflows |
| 12 | Create the configured telemetry subscription | IOS XE publisher | `telemetry ietf subscription <SUB_ID>` | Device enters telemetry subscription config mode |
| 13 | Set the stream to YANG Push | IOS XE publisher | `stream yang-push` | Subscription uses the `yang-push` stream |
| 14 | Set the XPath filter | IOS XE publisher | `filter xpath <XPATH_FILTER>` | Subscription targets the intended YANG subtree |
| 15 | Configure periodic publication | IOS XE publisher | `update-policy periodic <PERIOD_CENTISECONDS>` | Device sends updates at the configured interval |
| 16 | Configure on-change publication only when the subtree supports it | IOS XE publisher | `update-policy on-change` | Device sends updates when supported data changes |
| 17 | Set gRPC-compatible encoding | IOS XE publisher | `encoding encode-kvgpb` | Subscription uses kvGPB encoding |
| 18 | Configure source VRF when collector is reachable through a VRF | IOS XE publisher | `source-vrf <VRF_NAME>` | Subscription uses the intended VRF for collector reachability |
| 19 | Configure source address when the collector expects a specific source IP | IOS XE publisher | `source-address <SOURCE_IP>` | Publisher uses the expected source address |
| 20 | Configure the gRPC receiver | IOS XE publisher | `receiver ip address <COLLECTOR_IP> <COLLECTOR_PORT> protocol grpc-tcp` | Subscription points to the collector using cleartext gRPC |
| 21 | Exit telemetry subscription config mode | IOS XE publisher | `end` | Device returns to privileged EXEC mode |
| 22 | Save the configured subscription | IOS XE publisher | `write memory` | Subscription survives reload |
| 23 | Verify the subscription list | IOS XE publisher | `show telemetry ietf subscription all` | Subscription appears as `Configured` and ideally `Valid` |
| 24 | Verify subscription details | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Stream, XPath, period/on-change, encoding, source, and receiver are correct |
| 25 | Verify receiver-specific state | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> receiver` | Receiver shows expected address, port, protocol, and state |
| 26 | Verify publisher-to-receiver connection state | IOS XE publisher | `show telemetry internal connection` | Collector connection is `Active` |
| 27 | Verify telemetry update counters | IOS XE publisher | `show telemetry internal subscription all stats` | Update counters increase over time |
| 28 | Verify data on the collector | Collector | `<CHECK_COLLECTOR_OUTPUT>` | Collector receives telemetry from the IOS XE publisher |
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Skeleton
! =========================
! IOS XE publisher reachability checks
! =========================
show ip interface brief
show ip route <COLLECTOR_IP>
ping <COLLECTOR_IP> source <SOURCE_IP>
telnet <COLLECTOR_IP> <COLLECTOR_PORT>
! =========================
! Optional NETCONF/YANG foundation
! Use when YANG Suite or NETCONF RPC is used to create subscriptions
! =========================
configure terminal
 netconf-yang
end
write memory
! =========================
! Periodic configured gRPC dial-out subscription
! Period value is in centiseconds
! Example: 6000 = 60 seconds, 500 = 5 seconds
! =========================
configure terminal
!
telemetry ietf subscription <SUB_ID>
 stream yang-push
 filter xpath <XPATH_FILTER>
 update-policy periodic <PERIOD_CENTISECONDS>
 encoding encode-kvgpb
 source-vrf <VRF_NAME>
 source-address <SOURCE_IP>
 receiver ip address <COLLECTOR_IP> <COLLECTOR_PORT> protocol grpc-tcp
!
end
write memory
! =========================
! Minimal periodic example without VRF or explicit source address
! =========================
configure terminal
!
telemetry ietf subscription 100
 stream yang-push
 filter xpath /memory-ios-xe-oper:memory-statistics/memory-statistic
 update-policy periodic 6000
 encoding encode-kvgpb
 receiver ip address <COLLECTOR_IP> <COLLECTOR_PORT> protocol grpc-tcp
!
end
write memory
! =========================
! Periodic example with management VRF and source address
! =========================
configure terminal
!
telemetry ietf subscription 101
 stream yang-push
 filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
 update-policy periodic 2000
 encoding encode-kvgpb
 source-vrf <MGMT_VRF>
 source-address <SOURCE_IP>
 receiver ip address <COLLECTOR_IP> <COLLECTOR_PORT> protocol grpc-tcp
!
end
write memory
! =========================
! On-change configured subscription
! Use only when the selected YANG subtree supports on-change
! =========================
configure terminal
!
telemetry ietf subscription <SUB_ID>
 stream yang-push
 filter xpath <XPATH_FILTER>
 update-policy on-change
 encoding encode-kvgpb
 source-vrf <VRF_NAME>
 source-address <SOURCE_IP>
 receiver ip address <COLLECTOR_IP> <COLLECTOR_PORT> protocol grpc-tcp
!
end
write memory
! =========================
! IOS XE verification
! =========================
show telemetry ietf subscription all
show telemetry ietf subscription <SUB_ID> detail
show telemetry ietf subscription <SUB_ID> receiver
show telemetry internal connection
show telemetry internal subscription all stats
show running-config | section telemetry ietf subscription <SUB_ID>
! =========================
! Collector-side verification placeholder
! =========================
<CHECK_COLLECTOR_OUTPUT>
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify publisher source interface | IOS XE publisher | `show ip interface brief` | Intended source interface is up/up with expected IP |
| 2 | Verify route to collector | IOS XE publisher | `show ip route <COLLECTOR_IP>` | Route points toward the collector |
| 3 | Verify source-based reachability to collector | IOS XE publisher | `ping <COLLECTOR_IP> source <SOURCE_IP>` | Ping succeeds |
| 4 | Verify collector TCP port reachability | IOS XE publisher | `telnet <COLLECTOR_IP> <COLLECTOR_PORT>` | TCP connection opens or collector logs an inbound connection |
| 5 | Verify telemetry subscription config exists | IOS XE publisher | `show running-config | section telemetry ietf subscription <SUB_ID>` | Subscription config shows stream, XPath, update policy, encoding, and receiver |
| 6 | Verify all telemetry subscriptions | IOS XE publisher | `show telemetry ietf subscription all` | Subscription appears as `Configured` |
| 7 | Verify subscription is valid | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | State is `Valid` and no XPath parse error is listed |
| 8 | Verify XPath in subscription detail | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | XPath exactly matches the intended YANG path |
| 9 | Verify stream selection | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Stream is `yang-push` |
| 10 | Verify encoding | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Encoding is `encode-kvgpb` |
| 11 | Verify update policy | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Update trigger is periodic or on-change as intended |
| 12 | Verify receiver details | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> receiver` | Receiver address, port, and protocol are correct |
| 13 | Verify gRPC connection state | IOS XE publisher | `show telemetry internal connection` | Connection to collector is `Active` |
| 14 | Verify telemetry update counters | IOS XE publisher | `show telemetry internal subscription all stats` | Sent update counters increment |
| 15 | Verify NETCONF/YANG process if YANG Suite or RPC workflow is used | IOS XE publisher | `show platform software yang-management process` | YANG management processes are running |
| 16 | Verify NETCONF access if YANG Suite created the subscription | Client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | Device returns NETCONF `<hello>` capabilities |
| 17 | Verify collector is receiving data | Collector | `<CHECK_COLLECTOR_OUTPUT>` | Collector shows incoming telemetry from the IOS XE publisher |
| 18 | Verify periodic behavior | Collector | `<WATCH_COLLECTOR_OUTPUT_FOR_2_PERIODS>` | Updates arrive at roughly the configured interval |
| 19 | Verify on-change behavior if used | IOS XE and collector | `change the subscribed object` | Collector receives update after the modeled data changes |
| 20 | Verify persistence | IOS XE publisher | `show startup-config | section telemetry ietf subscription <SUB_ID>` | Startup-config contains the subscription |
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Rollback
! =========================
! Remove one configured telemetry subscription
! =========================
configure terminal
!
no telemetry ietf subscription <SUB_ID>
!
end
write memory
! =========================
! Verify subscription removal
! =========================
show telemetry ietf subscription all
show telemetry internal connection
show running-config | section telemetry ietf subscription <SUB_ID>
! =========================
! Remove NETCONF/YANG only if this telemetry lab enabled it
! Do not remove it if NETCONF, YANG Suite, RESTCONF-adjacent workflows, or controller workflows depend on it
! =========================
configure terminal
!
no netconf-yang
!
end
write memory
! =========================
! Collector cleanup
! =========================
<STOP_COLLECTOR_OR_TELEMETRY_RECEIVER>
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Subscription does not appear | IOS XE publisher | `show running-config | section telemetry ietf subscription` | Subscription was not configured or was configured with the wrong ID | Recreate `telemetry ietf subscription <SUB_ID>` |
| Subscription appears as `Invalid` | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | XPath syntax error, unsupported stream, unsupported model, or bad update policy | Fix XPath, model path, stream, or update policy |
| Detail output shows XPath parse error | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | XPath has typo, extra slash, missing prefix, or wrong module path | Rebuild XPath from YANG Suite or YANG tree |
| Subscription is valid but collector receives nothing | IOS XE publisher | `show telemetry internal connection` | Receiver connection is not active | Check collector IP, port, route, source VRF, source address, and collector listener |
| Connection state is `Connecting` | IOS XE publisher | `show telemetry internal connection` | Publisher cannot establish TCP/gRPC session to collector | Verify `ping`, `telnet <COLLECTOR_IP> <PORT>`, ACLs, firewall, and collector process |
| Receiver state is not connected | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> receiver` | Collector unreachable or receiver parameters are wrong | Correct receiver IP, port, protocol, VRF, or source address |
| Ping works but telemetry connection fails | IOS XE publisher | `telnet <COLLECTOR_IP> <COLLECTOR_PORT>` | TCP port blocked or collector is not listening | Start collector and permit the collector port |
| Telemetry uses wrong source IP | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Missing or wrong `source-address` | Configure `source-address <SOURCE_IP>` |
| Telemetry fails through management VRF | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Missing or wrong `source-vrf` | Configure `source-vrf <VRF_NAME>` and verify route in that VRF |
| gRPC subscription fails with wrong encoding | IOS XE publisher | `show running-config | section telemetry ietf subscription <SUB_ID>` | Encoding is not `encode-kvgpb` | Configure `encoding encode-kvgpb` |
| Collector receives undecodable data | Collector | Collector logs | Collector expects different encoding or protocol | Match collector to `grpc-tcp` and kvGPB |
| Periodic data arrives too often | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Period value too low | Increase `update-policy periodic <PERIOD_CENTISECONDS>` |
| Periodic data arrives too slowly | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Period value too high | Lower the periodic interval within lab limits |
| On-change subscription stays quiet | IOS XE publisher | `show telemetry ietf subscription <SUB_ID> detail` | Selected subtree does not support on-change or no data changed | Use periodic mode or select an on-change supported subtree |
| Dynamic subscription disappears | IOS XE publisher | `show telemetry ietf subscription all` | Dial-in client session ended | Use configured dial-out subscription for persistent lab behavior |
| YANG Suite cannot create subscription | IOS XE publisher | `show platform software yang-management process` | NETCONF/YANG not enabled or not healthy | Configure `netconf-yang` and verify TCP/830 NETCONF access |
| NETCONF test to port 830 fails | Client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | NETCONF/YANG disabled, auth issue, or TCP/830 blocked | Fix NETCONF baseline before using YANG Suite RPC workflow |
| Subscription works until reload then disappears | IOS XE publisher | `show startup-config | section telemetry ietf subscription <SUB_ID>` | Running config was not saved | Run `write memory` after creating configured subscription |
| Collector shows connection but no useful metric | Collector | `<CHECK_COLLECTOR_OUTPUT>` | XPath targets wrong subtree or data does not exist | Pick a known-good XPath such as memory or CPU first |
| Operator confuses NETCONF and gRPC | Operator | Subscription worksheet | NETCONF can create/manage subscriptions, but gRPC is the telemetry transport to the receiver | Keep management protocol, telemetry stream, encoding, and transport as separate fields |
##### Source_Basis
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Mental_Model
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Configuration_Checklist
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Skeleton
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Verification_Commands
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Rollback
# 09_IOS_XE_gRPC_Model_Driven_Telemetry_Failure_Checks
