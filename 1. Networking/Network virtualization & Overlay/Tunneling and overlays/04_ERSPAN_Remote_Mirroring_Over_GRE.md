
ERSPAN_Remote_Mirroring_Over_GRE.md
# ERSPAN_Remote_Mirroring_Over_GRE
# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | File Summary, CiscoPress and security configuration corpus | Identifies the local Cisco Press and Secure Firewall source files used for monitoring and ERSPAN context |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Encapsulated Remote SPAN, Specifying the Source Ports | Defines ERSPAN as routed Layer 3 SPAN and provides `monitor session <ID> type erspan-source`, `source interface`, `filter vlan`, and `no shutdown` syntax |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Encapsulated Remote SPAN, Specifying the Destination | Provides ERSPAN destination IP, `erspan-id`, `origin ip address`, `erspan ttl`, and verification with `show monitor session erspan-source session` |
| `CiscoPress_combined_part1.md` | ERSPAN Source and Destination Switches | Explains ERSPAN source session, GRE-encapsulated routed transport, and ERSPAN destination session |
| `CiscoPress_combined_part1.md` | NX-OS ERSPAN Source Switch Configuration | Supports source-side syntax using `monitor session <ID> type erspan-source`, `destination ip`, `erspan-id`, `vrf`, and `monitor erspan origin ip-address` |
| `CiscoPress_combined_part1.md` | NX-OS ERSPAN Destination Switch Configuration | Supports destination-side syntax using `monitor session <ID> type erspan-destination`, `source ip`, `destination interface`, `erspan-id`, and `switchport monitor` |
# ERSPAN_Remote_Mirroring_Over_GRE_Mental_Model
| Concept | Operational Meaning |
|---|---|
| ERSPAN | Encapsulated Remote Switched Port Analyzer, used to mirror traffic across a routed IP network |
| Not a VPN | ERSPAN is for visibility and packet capture, not user data transport |
| GRE transport | Mirrored packets are encapsulated in GRE and routed across the network |
| ERSPAN source session | The device that copies traffic from a source interface or VLAN |
| ERSPAN destination session | The device that receives ERSPAN traffic, decapsulates it, and sends it to an analyzer-facing port |
| Source traffic direction | `rx`, `tx`, or `both` determines which direction of source traffic is mirrored |
| ERSPAN destination IP | The routed IP address where mirrored GRE-encapsulated traffic is sent |
| Origin IP | The source/origin IP used to identify where the ERSPAN traffic came from |
| ERSPAN ID | Session identifier that must match between ERSPAN source and destination roles |
| Analyzer port | Local port connected to Wireshark, IDS, packet broker, or capture host |
| VLAN filter | Narrows mirrored traffic when the monitored source is a trunk or broad source |
| TTL or ToS | Optional ERSPAN packet treatment values for routed transport behavior |
| Failure boundary | If routed reachability between ERSPAN source and destination IPs is broken, ERSPAN cannot deliver mirrored traffic |
| Capture expectation | The analyzer sees copied packets, not an active forwarding path for production traffic |
# ERSPAN_Remote_Mirroring_Over_GRE_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the device that owns the traffic source | Source Switch | `show cdp neighbors` or `show lldp neighbors` | Correct access, trunk, or routed source device is identified |
| 2 | Identify the interface or VLAN to mirror | Source Switch | `show interfaces status` and `show vlan brief` | Correct source interface or VLAN is known |
| 3 | Identify the traffic direction to mirror | Source Switch | Design choice: `rx`, `tx`, or `both` | Capture scope is clear before configuration |
| 4 | Identify ERSPAN destination/analyzer IP | Destination Switch or Analyzer | `show ip interface brief` | Destination IP address is known and reachable |
| 5 | Verify routed reachability from source to ERSPAN destination IP | Source Switch | `ping <ERSPAN_DESTINATION_IP> source <ERSPAN_ORIGIN_IP_OR_SOURCE_INTERFACE>` | ERSPAN destination IP is reachable before monitor configuration |
| 6 | Verify reverse reachability if using an ERSPAN destination switch | Destination Switch | `ping <ERSPAN_ORIGIN_IP> source <ERSPAN_DESTINATION_IP>` | Destination side can reach the origin IP if required |
| 7 | Confirm route to ERSPAN destination | Source Switch | `show ip route <ERSPAN_DESTINATION_IP>` | Route exits the routed underlay, not a broken or unintended path |
| 8 | Confirm GRE is not blocked in the transport path | Transit Device | `show access-lists` | GRE protocol 47 is permitted between ERSPAN source and destination |
| 9 | Choose a unique ERSPAN session ID | Admin | `ERSPAN_ID=<ID>` | Source and destination roles will use the same ERSPAN ID |
| 10 | Choose a local monitor session number | Source Switch | `monitor session <SESSION_ID> type erspan-source` | ERSPAN source session mode opens |
| 11 | Add a useful session description | Source Switch | `description <DESCRIPTION>` | Monitor session purpose is documented |
| 12 | Configure source interface mirroring | Source Switch | `source interface <INTERFACE> rx` or `source interface <INTERFACE> tx` or `source interface <INTERFACE> both` | Selected interface traffic is copied in the chosen direction |
| 13 | Configure source VLAN mirroring if the lab mirrors a VLAN instead of a port | Source Switch | `source vlan <VLAN_ID> both` | Selected VLAN traffic is copied |
| 14 | Filter mirrored VLANs if the source is a trunk | Source Switch | `filter vlan <VLAN_ID>` | Only the intended VLAN is mirrored from the trunk source |
| 15 | Enable the ERSPAN source session | Source Switch | `no shutdown` | Source session is administratively enabled |
| 16 | Enter ERSPAN destination submode on IOS XE style syntax | Source Switch | `destination` | Destination subconfiguration mode opens |
| 17 | Configure remote ERSPAN destination IP | Source Switch | `ip address <ERSPAN_DESTINATION_IP>` | Mirrored traffic is sent toward the remote analyzer or destination switch |
| 18 | Configure ERSPAN ID | Source Switch | `erspan-id <ERSPAN_ID>` | ERSPAN source traffic carries the intended session identifier |
| 19 | Configure origin IP address | Source Switch | `origin ip address <ERSPAN_ORIGIN_IP>` | ERSPAN traffic has a stable source/origin address |
| 20 | Exit destination submode | Source Switch | `exit` | Return to ERSPAN source session mode |
| 21 | Configure ERSPAN TTL if required | Source Switch | `erspan ttl <TTL_VALUE>` | ERSPAN packets have an explicit TTL value |
| 22 | Configure ERSPAN ToS if required | Source Switch | `erspan tos <TOS_VALUE>` | ERSPAN packets have an explicit ToS value |
| 23 | Configure destination ERSPAN session if the lab uses a destination switch | Destination Switch | `monitor session <DEST_SESSION_ID> type erspan-destination` | ERSPAN destination session mode opens |
| 24 | Configure ERSPAN source IP on destination role | Destination Switch | `source ip <ERSPAN_DESTINATION_IP>` | Destination switch listens for ERSPAN traffic sent to its IP |
| 25 | Configure local analyzer-facing destination port | Destination Switch | `destination interface <ANALYZER_INTERFACE>` | Decapsulated mirrored traffic exits toward the analyzer port |
| 26 | Configure matching ERSPAN ID on destination role | Destination Switch | `erspan-id <ERSPAN_ID>` | Destination session matches the source session |
| 27 | Configure VRF if the platform requires it | Source/Destination Switch | `vrf <VRF_NAME>` | ERSPAN lookup uses the intended routing table |
| 28 | Enable the ERSPAN destination session | Destination Switch | `no shutdown` | Destination session is administratively enabled |
| 29 | Put analyzer-facing port into monitor mode if required | Destination Switch | `interface <ANALYZER_INTERFACE>` then `switchport monitor` | Analyzer-facing port is ready to receive copied traffic |
| 30 | Verify ERSPAN source session status | Source Switch | `show monitor session erspan-source session` or `show monitor session <SESSION_ID>` | Session is enabled and shows source, destination IP, origin IP, and ERSPAN ID |
| 31 | Verify ERSPAN destination session status | Destination Switch | `show monitor session <DEST_SESSION_ID>` | Destination session is up and matches the expected ERSPAN ID |
| 32 | Generate test traffic from the monitored source | Test Host | `ping <REMOTE_IP>` or application traffic | Traffic is available for mirroring |
| 33 | Verify analyzer receives mirrored packets | Analyzer | Packet capture on analyzer NIC | Analyzer sees copied source traffic |
| 34 | Verify production forwarding is not affected | Source/Destination Switch | `show interfaces <SOURCE_INTERFACE>` and end-to-end ping | Source traffic still forwards normally |
| 35 | Save working configuration | Source/Destination Switch | `write memory` or `copy running-config startup-config` | ERSPAN configuration persists after reload |
# ERSPAN_Remote_Mirroring_Over_GRE_Skeleton
! =========================================================
! IOS XE STYLE ERSPAN SOURCE SESSION
! Use this when the platform uses destination submode.
! =========================================================
conf t
!
monitor session <SESSION_ID> type erspan-source
 description <DESCRIPTION>
 source interface <SOURCE_INTERFACE> rx
 ! source interface <SOURCE_INTERFACE> tx
 ! source interface <SOURCE_INTERFACE> both
 ! source vlan <VLAN_ID> both
 ! filter vlan <VLAN_ID>
 no shutdown
 destination
  ip address <ERSPAN_DESTINATION_IP>
  erspan-id <ERSPAN_ID>
  origin ip address <ERSPAN_ORIGIN_IP>
 exit
!
erspan ttl <TTL_VALUE>
! erspan tos <TOS_VALUE>
!
end
write memory
! =========================================================
! NX-OS STYLE ERSPAN SOURCE SESSION
! Use this when the platform uses destination ip directly under the session.
! =========================================================
conf t
!
monitor session <SESSION_ID> type erspan-source
 source interface <SOURCE_INTERFACE> both
 destination ip <ERSPAN_DESTINATION_IP>
 erspan-id <ERSPAN_ID>
 vrf <VRF_NAME>
 no shut
!
monitor erspan origin ip-address <ERSPAN_ORIGIN_IP> global
!
end
copy running-config startup-config
! =========================================================
! NX-OS STYLE ERSPAN DESTINATION SESSION
! Use this when a remote switch decapsulates ERSPAN and forwards it to a capture port.
! =========================================================
conf t
!
monitor session <DEST_SESSION_ID> type erspan-destination
 source ip <ERSPAN_DESTINATION_IP>
 destination interface <ANALYZER_INTERFACE>
 erspan-id <ERSPAN_ID>
 vrf <VRF_NAME>
 no shut
!
interface <ANALYZER_INTERFACE>
 switchport monitor
 no shutdown
!
end
copy running-config startup-config
! =========================================================
! BASIC REACHABILITY TESTS
! =========================================================
show ip route <ERSPAN_DESTINATION_IP>
ping <ERSPAN_DESTINATION_IP> source <ERSPAN_ORIGIN_IP>
show monitor session <SESSION_ID>
# ERSPAN_Remote_Mirroring_Over_GRE_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Confirm source interface exists | Source Switch | `show interfaces status` | Source interface is present and operational |
| Confirm source VLAN exists if used | Source Switch | `show vlan brief` | Source VLAN exists and has expected member ports |
| Confirm ERSPAN destination IP reachability | Source Switch | `ping <ERSPAN_DESTINATION_IP> source <ERSPAN_ORIGIN_IP>` | Ping succeeds |
| Confirm route to ERSPAN destination | Source Switch | `show ip route <ERSPAN_DESTINATION_IP>` | Route points through routed underlay |
| Confirm GRE is allowed | Transit Device | `show access-lists` | GRE protocol 47 is not blocked |
| Confirm ERSPAN source session | Source Switch | `show monitor session erspan-source session` | Session is admin enabled |
| Confirm ERSPAN source details | Source Switch | `show monitor session <SESSION_ID>` | Output shows source interface/VLAN, destination IP, origin IP, and ERSPAN ID |
| Confirm running monitor configuration | Source Switch | `show running-config | section monitor` | Config contains the expected ERSPAN source session |
| Confirm global ERSPAN TTL or ToS | Source Switch | `show running-config | include erspan ttl|erspan tos` | TTL or ToS value appears if configured |
| Confirm destination session if used | Destination Switch | `show monitor session <DEST_SESSION_ID>` | Destination ERSPAN session is up |
| Confirm analyzer port state | Destination Switch | `show interface <ANALYZER_INTERFACE> status` | Analyzer port is connected and ready |
| Confirm analyzer port monitor mode | Destination Switch | `show running-config interface <ANALYZER_INTERFACE>` | Output shows monitor-port behavior if required |
| Confirm ERSPAN ID match | Source/Destination Switch | `show monitor session <SESSION_ID>` and `show monitor session <DEST_SESSION_ID>` | ERSPAN ID matches on both roles |
| Confirm traffic generation | Test Host | `ping <REMOTE_IP>` or application traffic | Source host generates packets |
| Confirm capture visibility | Analyzer | Packet capture on analyzer interface | Mirrored packets are visible |
| Confirm production path still works | Test Host | `ping <REMOTE_IP>` | Original traffic is not interrupted by ERSPAN |
| Confirm no source interface errors | Source Switch | `show interfaces <SOURCE_INTERFACE>` | No unexpected drops or interface failures |
| Confirm no ERSPAN-related logs | Source/Destination Switch | `show logging | include ERSPAN|SPAN|monitor|GRE` | No ERSPAN failure messages appear |
# ERSPAN_Remote_Mirroring_Over_GRE_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop traffic mirroring on the source session | Source Switch | `monitor session <SESSION_ID> type erspan-source` then `shutdown` | ERSPAN source session stops copying traffic |
| 2 | Remove ERSPAN source session | Source Switch | `no monitor session <SESSION_ID> type erspan-source` | Source ERSPAN session is deleted |
| 3 | Remove global ERSPAN origin if configured on NX-OS | Source Switch | `no monitor erspan origin ip-address <ERSPAN_ORIGIN_IP> global` | Global ERSPAN origin is removed |
| 4 | Remove global ERSPAN TTL if configured | Source Switch | `no erspan ttl <TTL_VALUE>` | ERSPAN TTL override is removed |
| 5 | Remove global ERSPAN ToS if configured | Source Switch | `no erspan tos <TOS_VALUE>` | ERSPAN ToS override is removed |
| 6 | Stop destination ERSPAN session if used | Destination Switch | `monitor session <DEST_SESSION_ID> type erspan-destination` then `shutdown` | Destination session stops receiving mirrored traffic |
| 7 | Remove destination ERSPAN session if used | Destination Switch | `no monitor session <DEST_SESSION_ID> type erspan-destination` | Destination ERSPAN session is deleted |
| 8 | Remove monitor mode from analyzer port if no longer needed | Destination Switch | `interface <ANALYZER_INTERFACE>` then `no switchport monitor` | Analyzer port returns to normal switchport behavior |
| 9 | Verify source session removal | Source Switch | `show monitor session <SESSION_ID>` | Session no longer appears or is shutdown |
| 10 | Verify destination session removal | Destination Switch | `show monitor session <DEST_SESSION_ID>` | Destination session no longer appears or is shutdown |
| 11 | Verify production forwarding remains normal | Test Host | `ping <REMOTE_IP>` | Original traffic path still works |
| 12 | Save rollback state | Source/Destination Switch | `write memory` or `copy running-config startup-config` | Rollback persists after reload |
# ERSPAN_Remote_Mirroring_Over_GRE_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| ERSPAN session is shutdown | Session was not enabled | Source/Destination Switch | `show monitor session <SESSION_ID>` | Configure `no shutdown` or `no shut` under the ERSPAN session |
| Analyzer sees no packets | No routed reachability to ERSPAN destination IP | Source Switch | `ping <ERSPAN_DESTINATION_IP> source <ERSPAN_ORIGIN_IP>` | Fix routing to ERSPAN destination IP |
| Analyzer sees no packets but ping works | ERSPAN ID mismatch | Source/Destination Switch | `show monitor session <SESSION_ID>` | Match `erspan-id` on source and destination roles |
| Analyzer sees no packets from expected host | Wrong source interface or VLAN | Source Switch | `show monitor session <SESSION_ID>` | Correct `source interface` or `source vlan` |
| Only one traffic direction appears | Wrong `rx`, `tx`, or `both` direction | Source Switch | `show monitor session <SESSION_ID>` | Change source direction to the required value |
| Trunk source mirrors too much traffic | Missing VLAN filter | Source Switch | `show monitor session <SESSION_ID>` | Add `filter vlan <VLAN_ID>` |
| ERSPAN packets do not cross network | GRE blocked by ACL or firewall | Transit Device | `show access-lists` | Permit GRE protocol 47 between ERSPAN endpoints |
| Destination switch receives ERSPAN but analyzer port is silent | Analyzer destination port is not in monitor mode | Destination Switch | `show running-config interface <ANALYZER_INTERFACE>` | Configure `switchport monitor` if required by platform |
| ERSPAN destination session remains down | Wrong source IP on destination session | Destination Switch | `show monitor session <DEST_SESSION_ID>` | Configure source/listener IP to match the ERSPAN destination IP design |
| Capture shows ERSPAN outer packets instead of decapsulated mirrored traffic | Analyzer is connected before decapsulation or destination session is not configured | Destination Switch/Analyzer | Packet capture | Configure ERSPAN destination session or capture with ERSPAN decode support |
| Mirrored traffic volume overloads analyzer | Source scope too broad | Source Switch | `show monitor session <SESSION_ID>` | Narrow source, direction, VLAN filter, or capture window |
| Production traffic is affected | Wrong port used as analyzer destination or monitor port design error | Destination Switch | `show interfaces status` and `show running-config interface <ANALYZER_INTERFACE>` | Use a dedicated analyzer port, not a production forwarding port |
| ERSPAN source config accepted but no useful packets appear | Source traffic is locally generated supervisor traffic or unsupported source | Source Switch | `show monitor session <SESSION_ID>` | Mirror supported data-plane source traffic instead |
| ERSPAN works locally but fails across VRF | Wrong VRF for ERSPAN routing lookup | Source/Destination Switch | `show monitor session <SESSION_ID>` and `show ip route vrf <VRF_NAME> <ERSPAN_DESTINATION_IP>` | Configure the correct `vrf` and route |
##### Source_Basis
# ERSPAN_Remote_Mirroring_Over_GRE_Mental_Model
# ERSPAN_Remote_Mirroring_Over_GRE_Configuration_Checklist
# ERSPAN_Remote_Mirroring_Over_GRE_Skeleton
# ERSPAN_Remote_Mirroring_Over_GRE_Verification_Commands
# ERSPAN_Remote_Mirroring_Over_GRE_Rollback
# ERSPAN_Remote_Mirroring_Over_GRE_Failure_Checks
# index of each title throughout note, not in table format

