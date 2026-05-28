
Cisco’s official RSVP/SBM guide shows the DSBM workflow as: enable RSVP on the shared interface with ip rsvp bandwidth, make the interface a DSBM candidate with ip rsvp dsbm candidate, optionally set non-reserved send limits, then verify election state with show ip rsvp sbm [detail] [interface]. Cisco’s command reference states DSBM candidate priority ranges from 64 to 128, and the highest priority wins the election.  

RSVP_DSBM_Shared_LAN_Admission_Control.md

RSVP_DSBM_Shared_LAN_Admission_Control

# Source_Basis
# RSVP_DSBM_Shared_LAN_Admission_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| RSVP | Per-flow admission control protocol used to reserve resources before admitting traffic |
| SBM | Subnetwork Bandwidth Manager. RSVP admission-control mechanism for shared IEEE-style LAN segments |
| DSBM | Designated Subnetwork Bandwidth Manager. The elected SBM that controls admission on the shared segment |
| Shared LAN problem | Multiple RSVP devices on the same Ethernet segment can overcommit shared bandwidth unless one device coordinates admission control |
| DSBM election | Multiple candidates can exist, but one DSBM is elected for the segment |
| DSBM priority | Higher priority wins the election. Cisco priority range is 64 to 128 |
| Default DSBM priority | If `ip rsvp dsbm candidate` is configured without a priority, default priority is 64 |
| RSVP bandwidth pool | `ip rsvp bandwidth` enables RSVP on the interface and defines reservable bandwidth |
| Candidate vs participant | A router can be RSVP-enabled on a managed segment without being a DSBM candidate |
| NonResvSendLimit | Optional DSBM object that limits traffic sent without a valid RSVP reservation |
| Managed segment | Shared LAN segment where RSVP admission control is coordinated by the elected DSBM |
| Verification anchor | `show ip rsvp sbm` tells whether the local interface is a DSBM candidate and which DSBM won |
| Not CBWFQ or LLQ | DSBM controls reservation admission on a shared LAN. It is not a queue scheduler by itself |
# RSVP_DSBM_Shared_LAN_Admission_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm shared LAN interfaces are up | RSVP Router | `show ip interface brief` | Shared Ethernet interfaces are up/up |
| 2 | Confirm all RSVP candidates are on the same shared segment | RSVP Router | `show cdp neighbors` / `show lldp neighbors` / `show arp` | Devices appear reachable on the same LAN |
| 3 | Confirm IP routing between RSVP sender and receiver still works | RSVP Router | `show ip route <receiver-ip>` | Route exists toward the receiver or RSVP session address |
| 4 | Confirm reverse routing toward the sender | RSVP Router | `show ip route <sender-ip>` | Route exists back toward the sender |
| 5 | Confirm current RSVP interface state | RSVP Router | `show ip rsvp interface` | Existing RSVP state is known before DSBM changes |
| 6 | Confirm current SBM or DSBM state | RSVP Router | `show ip rsvp sbm` | Existing DSBM election state is known |
| 7 | Enter global configuration mode | RSVP Router | `configure terminal` | Device enters global config mode |
| 8 | Enter the shared LAN interface | RSVP Router | `interface <shared-lan-interface>` | Device enters interface configuration mode |
| 9 | Enable RSVP bandwidth on the shared interface | RSVP Router | `ip rsvp bandwidth <total-rsvp-kbps> <single-flow-kbps>` | RSVP is enabled and interface reservable bandwidth is defined |
| 10 | Configure the interface as a DSBM candidate | RSVP Router | `ip rsvp dsbm candidate <priority>` | Interface participates in DSBM election |
| 11 | Use a higher priority on the intended primary DSBM | Primary DSBM Candidate | `ip rsvp dsbm candidate 100` | Primary candidate should win against lower-priority candidates |
| 12 | Use a lower priority on backup candidates | Backup DSBM Candidate | `ip rsvp dsbm candidate 70` | Backup candidate participates but should not win while higher priority candidate is present |
| 13 | Leave non-candidate RSVP routers without DSBM candidacy | RSVP Participant | Do not configure `ip rsvp dsbm candidate` | Router can use the DSBM but does not contend in election |
| 14 | Optionally set non-reserved send average rate | DSBM Candidate | `ip rsvp dsbm non-resv-send-limit rate <kbps>` | DSBM candidate advertises non-reserved traffic average rate limit |
| 15 | Optionally set non-reserved send burst size | DSBM Candidate | `ip rsvp dsbm non-resv-send-limit burst <kilobytes>` | DSBM candidate advertises non-reserved traffic burst limit |
| 16 | Optionally set non-reserved send peak rate | DSBM Candidate | `ip rsvp dsbm non-resv-send-limit peak <kbps>` | DSBM candidate advertises non-reserved traffic peak limit |
| 17 | Optionally set non-reserved minimum packet unit | DSBM Candidate | `ip rsvp dsbm non-resv-send-limit min-unit <bytes>` | Minimum policed unit is defined |
| 18 | Optionally set non-reserved maximum packet unit | DSBM Candidate | `ip rsvp dsbm non-resv-send-limit max-unit <bytes>` | Maximum packet unit is defined |
| 19 | Exit interface configuration mode | RSVP Router | `end` | Device returns to privileged EXEC mode |
| 20 | Verify RSVP interface bandwidth | RSVP Router | `show ip rsvp interface` | Interface shows RSVP bandwidth information |
| 21 | Verify DSBM election summary | RSVP Router | `show ip rsvp sbm` | Interface shows DSBM address, DSBM priority, candidacy state, and local priority |
| 22 | Verify detailed DSBM state | RSVP Router | `show ip rsvp sbm detail` | Output shows whether local router is DSBM and displays NonResvSendLimit values |
| 23 | Verify a specific shared interface | RSVP Router | `show ip rsvp sbm detail <shared-lan-interface>` | Output is limited to the target shared LAN interface |
| 24 | Configure RSVP sender simulation if the lab does not use a real RSVP application | Sender-side Router | `ip rsvp sender-host <session-ip> <sender-ip> udp <session-dest-port> <sender-source-port> <bandwidth> <burst-size>` | RSVP PATH sender state is simulated |
| 25 | Configure RSVP receiver simulation if the lab does not use a real RSVP application | Receiver-side Router | `ip rsvp reservation-host <session-ip> <sender-ip> udp <session-dest-port> <sender-source-port> ff rate <bandwidth> <burst-size>` | RSVP RESV receiver state is simulated |
| 26 | Verify RSVP sender state | RSVP Router | `show ip rsvp sender` | PATH sender state appears |
| 27 | Verify RSVP reservation state | RSVP Router | `show ip rsvp reservation` | RESV reservation state appears |
| 28 | Verify installed RSVP state | RSVP Router | `show ip rsvp installed` | Installed reservation appears if admission succeeds |
| 29 | Test DSBM election failover | Primary DSBM Candidate | `configure terminal` then `interface <shared-lan-interface>` then `no ip rsvp dsbm candidate` | Backup candidate should become elected DSBM if it has next-highest priority |
| 30 | Recheck DSBM election after failover | RSVP Router | `show ip rsvp sbm detail` | Current DSBM address changes to the backup candidate |
| 31 | Restore primary DSBM candidacy | Primary DSBM Candidate | `ip rsvp dsbm candidate <higher-priority>` | Primary becomes candidate again and can retake DSBM role |
| 32 | Save the configuration | RSVP Router | `copy running-config startup-config` | DSBM configuration survives reload |
# RSVP_DSBM_Shared_LAN_Admission_Control_Skeleton
configure terminal
interface <SHARED_LAN_INTERFACE>
 ip rsvp bandwidth <TOTAL_RSVP_KBPS> <SINGLE_FLOW_KBPS>
 ip rsvp dsbm candidate <PRIORITY_64_TO_128>
 ip rsvp dsbm non-resv-send-limit rate <KBPS>
 ip rsvp dsbm non-resv-send-limit burst <KILOBYTES>
 ip rsvp dsbm non-resv-send-limit peak <KBPS>
 exit
end
show ip rsvp interface
show ip rsvp sbm
show ip rsvp sbm detail
show ip rsvp sbm detail <SHARED_LAN_INTERFACE>
configure terminal
ip rsvp sender-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> <BANDWIDTH> <BURST_SIZE>
ip rsvp reservation-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> ff rate <BANDWIDTH> <BURST_SIZE>
end
show ip rsvp sender
show ip rsvp reservation
show ip rsvp installed
show ip rsvp sbm detail
copy running-config startup-config
# RSVP_DSBM_Shared_LAN_Admission_Control_Example_Skeleton
configure terminal
interface Ethernet2
 ip address 145.2.2.150 255.255.255.0
 ip rsvp bandwidth 7500 7500
 ip rsvp dsbm candidate 100
 ip rsvp dsbm non-resv-send-limit rate 500
 ip rsvp dsbm non-resv-send-limit burst 1000
 ip rsvp dsbm non-resv-send-limit peak 500
 exit
end
show ip rsvp interface
show ip rsvp sbm
show ip rsvp sbm detail Ethernet2
configure terminal
ip rsvp sender-host 10.20.20.20 10.10.10.10 udp 5000 4000 1000 1000
ip rsvp reservation-host 10.20.20.20 10.10.10.10 udp 5000 4000 ff rate 1000 1000
end
show ip rsvp sender
show ip rsvp reservation
show ip rsvp installed
show ip rsvp sbm detail
copy running-config startup-config
# RSVP_DSBM_Shared_LAN_Admission_Control_Two_Candidate_Election_Example
! R1 intended primary DSBM
configure terminal
interface Ethernet0/0
 ip address 10.1.1.1 255.255.255.0
 ip rsvp bandwidth 7500 7500
 ip rsvp dsbm candidate 100
 ip rsvp dsbm non-resv-send-limit rate 500
 ip rsvp dsbm non-resv-send-limit burst 1000
 ip rsvp dsbm non-resv-send-limit peak 500
 exit
end
! R2 intended backup DSBM
configure terminal
interface Ethernet0/0
 ip address 10.1.1.2 255.255.255.0
 ip rsvp bandwidth 7500 7500
 ip rsvp dsbm candidate 70
 exit
end
! R3 RSVP participant, not a DSBM candidate
configure terminal
interface Ethernet0/0
 ip address 10.1.1.3 255.255.255.0
 ip rsvp bandwidth 7500 7500
 exit
end
show ip rsvp sbm
show ip rsvp sbm detail
# RSVP_DSBM_Shared_LAN_Admission_Control_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms shared LAN interfaces are operational | Shared LAN interfaces are up/up |
| `show ip route <receiver-ip>` | Confirms forward routing for RSVP flow | Route points toward expected next hop |
| `show ip route <sender-ip>` | Confirms reverse routing for RSVP flow | Route points back toward sender side |
| `show running-config interface <shared-lan-interface>` | Confirms RSVP and DSBM interface configuration | Interface shows `ip rsvp bandwidth` and optional `ip rsvp dsbm candidate` |
| `show ip rsvp interface` | Verifies RSVP interface bandwidth state | Shared interface shows RSVP reservable bandwidth |
| `show ip rsvp sbm` | Verifies DSBM election summary | Output shows DSBM address, DSBM priority, candidate state, and local priority |
| `show ip rsvp sbm detail` | Verifies detailed SBM and DSBM state | Output shows whether local router is DSBM and displays NonResvSendLimit values |
| `show ip rsvp sbm detail <interface>` | Verifies DSBM state for one shared segment | Target interface shows local and current DSBM information |
| `show ip rsvp sender` | Verifies RSVP PATH sender state | Sender/session state appears |
| `show ip rsvp reservation` | Verifies RSVP RESV state | Reservation state appears |
| `show ip rsvp installed` | Verifies admitted RSVP reservations | Installed reservation appears after successful admission |
| `debug ip rsvp detail sbm` | Troubleshoots SBM messages and DSBM state transitions | Debug shows SBM message processing and election behavior |
| `debug ip rsvp detail` | Troubleshoots detailed RSVP and SBM behavior | Debug shows RSVP and SBM processing details |
# RSVP_DSBM_Shared_LAN_Admission_Control_Rollback
configure terminal
interface <SHARED_LAN_INTERFACE>
 no ip rsvp dsbm candidate
 no ip rsvp dsbm non-resv-send-limit rate
 no ip rsvp dsbm non-resv-send-limit burst
 no ip rsvp dsbm non-resv-send-limit peak
 no ip rsvp dsbm non-resv-send-limit min-unit
 no ip rsvp dsbm non-resv-send-limit max-unit
 no ip rsvp bandwidth
 exit
no ip rsvp sender-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> <BANDWIDTH> <BURST_SIZE>
no ip rsvp reservation-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> ff rate <BANDWIDTH> <BURST_SIZE>
end
undebug all
show ip rsvp interface
show ip rsvp sbm
show ip rsvp sender
show ip rsvp reservation
show ip rsvp installed
show running-config | include ip rsvp
# RSVP_DSBM_Shared_LAN_Admission_Control_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `show ip rsvp sbm` shows no useful state | RSVP is not enabled on the shared interface | `show running-config interface <shared-lan-interface>` | Configure `ip rsvp bandwidth <total> <single-flow>` |
| Interface is RSVP-enabled but not a candidate | `ip rsvp dsbm candidate` is missing | `show ip rsvp sbm` | Add `ip rsvp dsbm candidate <priority>` if the router should participate in election |
| Wrong router becomes DSBM | Candidate priority is higher on the wrong device | `show ip rsvp sbm detail` | Raise intended DSBM priority or lower the unwanted candidate priority |
| No backup takes over after primary removal | Backup router was not configured as DSBM candidate | `show running-config interface <shared-lan-interface>` on backup | Configure `ip rsvp dsbm candidate <priority>` on backup |
| DSBM election appears unstable | Multiple candidates have competing priorities or interfaces flap | `show ip rsvp sbm detail` and `show ip interface brief` | Stabilize the LAN and set clear primary and backup priorities |
| Candidate configured but not elected | Another candidate has higher priority | `show ip rsvp sbm` | This is expected unless local router should be primary |
| RSVP reservation does not install | RSVP sender or receiver simulation is missing | `show ip rsvp sender` and `show ip rsvp reservation` | Configure both `ip rsvp sender-host` and `ip rsvp reservation-host` |
| RSVP sender exists but RESV is missing | Receiver side did not request reservation | `show ip rsvp sender` and `show ip rsvp reservation` | Fix receiver simulation or endpoint RSVP support |
| Reservation rejected | Requested bandwidth exceeds RSVP pool | `show ip rsvp interface` and `show ip rsvp installed` | Lower requested bandwidth or increase `ip rsvp bandwidth` |
| Non-reserved traffic behavior does not match expectation | NonResvSendLimit values are omitted or incorrect | `show ip rsvp sbm detail` | Configure `ip rsvp dsbm non-resv-send-limit rate`, `burst`, and `peak` |
| Non-reserved send limit appears unlimited | Default behavior allows unlimited non-reserved traffic | `show ip rsvp sbm detail` | Configure explicit NonResvSendLimit parameters |
| RSVP follows an unexpected path | Routing does not match the intended lab design | `show ip route <session-ip>` and `traceroute <session-ip>` | Fix routing before troubleshooting DSBM |
| Debug output is too noisy | RSVP debug left running | `show debugging` | Use `undebug all` after troubleshooting |
| Config is lost after reload | Configuration was not saved | `show startup-config \| include ip rsvp` | Run `copy running-config startup-config` |
##### Source_Basis
# RSVP_DSBM_Shared_LAN_Admission_Control_Mental_Model
# RSVP_DSBM_Shared_LAN_Admission_Control_Configuration_Checklist
# RSVP_DSBM_Shared_LAN_Admission_Control_Skeleton
# RSVP_DSBM_Shared_LAN_Admission_Control_Example_Skeleton
# RSVP_DSBM_Shared_LAN_Admission_Control_Two_Candidate_Election_Example
# RSVP_DSBM_Shared_LAN_Admission_Control_Verification_Commands
# RSVP_DSBM_Shared_LAN_Admission_Control_Rollback
# RSVP_DSBM_Shared_LAN_Admission_Control_Failure_Checks

