
Cisco’s RSVP config guide shows ip rsvp bandwidth as the interface command for RSVP bandwidth reservation, and Cisco’s command reference confirms ip rsvp sender-host, ip rsvp reservation-host, show ip rsvp interface, show ip rsvp sender, show ip rsvp reservation, and show ip rsvp installed as the core test and verification commands.  

QoS_IntServ_RSVP_Admission_Control.md

#  QoS_IntServ_RSVP_Admission_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IntServ | Integrated Services. A per-flow QoS model where applications request resources from the network before traffic is admitted |
| RSVP | Resource Reservation Protocol. RSVP signals resource requests across the routed path |
| Admission control | Router checks whether enough reservable bandwidth remains before accepting a flow |
| Reservation state | RSVP creates per-flow state on routers along the path |
| PATH message | Sender-side RSVP signaling that describes the traffic flow |
| RESV message | Receiver-side RSVP signaling that requests the reservation back toward the sender |
| `ip rsvp bandwidth` | Enables RSVP on an interface and defines how much bandwidth can be reserved |
| Total RSVP bandwidth | Maximum reservable bandwidth pool on the interface |
| Single-flow bandwidth | Maximum bandwidth one RSVP flow can reserve |
| RSVP interface dependency | Every RSVP-aware hop must have RSVP enabled on the correct transit interface |
| Routing dependency | RSVP follows the existing routed path. Broken routing means broken RSVP |
| RSVP simulation | `ip rsvp sender-host` and `ip rsvp reservation-host` can simulate RSVP hosts for lab testing |
| Not DiffServ | DiffServ marks classes of traffic. RSVP/IntServ reserves resources per flow |
| Not CBWFQ or LLQ | RSVP admission control does not replace queuing, shaping, policing, or LLQ packet treatment |
| Plain RSVP vs MPLS TE | Do not treat `ip rsvp qos` as a baseline IntServ command. It is tied to RSVP QoS flows with MPLS TE use cases |
# QoS_IntServ_RSVP_Admission_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm RSVP path interfaces are up | Router | `show ip interface brief` | Sender, receiver, and transit interfaces are up/up |
| 2 | Confirm routing from sender side to receiver side | Router | `show ip route <receiver-ip>` | Route exists toward the receiver or RSVP session address |
| 3 | Confirm reverse-path routing from receiver side to sender side | Router | `show ip route <sender-ip>` | Route exists back toward the sender |
| 4 | Confirm the actual routed path | Router | `traceroute <receiver-ip>` | Traffic crosses the expected RSVP transit routers |
| 5 | Confirm interface bandwidth references | Router | `show interfaces <interface-id>` | Interface bandwidth and line protocol are visible |
| 6 | Enter global configuration mode | Router | `configure terminal` | Device enters global config mode |
| 7 | Enter the first RSVP transit interface | Router | `interface <interface-id>` | Device enters interface config mode |
| 8 | Enable RSVP with a fixed reservable bandwidth pool | Router | `ip rsvp bandwidth <total-rsvp-kbps> <single-flow-kbps>` | RSVP is enabled and bandwidth limits are defined |
| 9 | Enable RSVP with percentage-based reservable bandwidth if required | Router | `ip rsvp bandwidth percent <total-percent> percent <single-flow-percent>` | RSVP pool and per-flow limit are percentage-based |
| 10 | Enable RSVP with fixed total bandwidth and percentage single-flow limit if required | Router | `ip rsvp bandwidth <total-rsvp-kbps> percent <single-flow-percent>` | RSVP pool is fixed and single-flow limit is percentage-based |
| 11 | Enable RSVP with percentage total bandwidth and fixed single-flow limit if required | Router | `ip rsvp bandwidth percent <total-percent> <single-flow-kbps>` | RSVP pool is percentage-based and single-flow limit is fixed |
| 12 | Configure ingress RSVP admission control if the lab requires ingress CAC | Router | `ip rsvp bandwidth ingress <ingress-kbps>` | Interface has an ingress RSVP bandwidth limit |
| 13 | Exit interface config mode | Router | `exit` | Device returns to global config mode |
| 14 | Repeat RSVP bandwidth configuration on each transit interface | Router | `interface <next-interface-id>` then `ip rsvp bandwidth <values>` | Every RSVP-aware interface along the path can process reservations |
| 15 | Return to privileged EXEC mode | Router | `end` | Device exits configuration mode |
| 16 | Verify RSVP is enabled on interfaces | Router | `show ip rsvp interface` | RSVP-enabled interfaces show reservable bandwidth information |
| 17 | Configure sender-side RSVP host simulation if no real RSVP application exists | Sender-side Router | `configure terminal` then `ip rsvp sender-host <session-ip> <sender-ip> udp <session-dest-port> <sender-source-port> <bandwidth> <burst-size>` | Router simulates an RSVP PATH sender |
| 18 | Configure receiver-side RSVP host simulation if no real RSVP application exists | Receiver-side Router | `configure terminal` then `ip rsvp reservation-host <session-ip> <sender-ip> udp <session-dest-port> <sender-source-port> ff rate <bandwidth> <burst-size>` | Router simulates an RSVP RESV receiver |
| 19 | Verify RSVP PATH sender state | Router | `show ip rsvp sender` | Sender state appears for the expected session |
| 20 | Verify RSVP RESV reservation state | Router | `show ip rsvp reservation` | Reservation state appears for the expected session |
| 21 | Verify installed RSVP reservation state | Router | `show ip rsvp installed` | Installed bandwidth reservation appears |
| 22 | Verify RSVP neighbor state if signaling crossed the link | Router | `show ip rsvp neighbor` | RSVP neighbors appear where RSVP signaling is active |
| 23 | Confirm interface bandwidth consumption after reservation | Router | `show ip rsvp interface` | Available RSVP bandwidth decreases after admitted reservation |
| 24 | Test admission failure with an oversized reservation | Sender/Receiver Router | Increase `<bandwidth>` above available RSVP pool | Reservation is rejected or not installed |
| 25 | Restore the intended lab reservation value | Sender/Receiver Router | Reapply `ip rsvp sender-host` and `ip rsvp reservation-host` with valid bandwidth | Reservation installs successfully |
| 26 | Save the configuration | Router | `copy running-config startup-config` | RSVP configuration survives reload |
# QoS_IntServ_RSVP_Admission_Control_Skeleton
configure terminal
interface <RSVP_INTERFACE_1>
 ip rsvp bandwidth <TOTAL_RSVP_KBPS> <SINGLE_FLOW_KBPS>
 exit
interface <RSVP_INTERFACE_2>
 ip rsvp bandwidth <TOTAL_RSVP_KBPS> <SINGLE_FLOW_KBPS>
 exit
end
show ip rsvp interface
show ip rsvp neighbor
configure terminal
ip rsvp sender-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> <BANDWIDTH> <BURST_SIZE>
ip rsvp reservation-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> ff rate <BANDWIDTH> <BURST_SIZE>
end
show ip rsvp sender
show ip rsvp reservation
show ip rsvp installed
show ip rsvp interface
copy running-config startup-config
# QoS_IntServ_RSVP_Admission_Control_Example_Skeleton
configure terminal
interface GigabitEthernet1
 ip rsvp bandwidth 10000 2000
 exit
interface GigabitEthernet2
 ip rsvp bandwidth 10000 2000
 exit
end
show ip rsvp interface
show ip rsvp neighbor
configure terminal
ip rsvp sender-host 10.20.20.20 10.10.10.10 udp 5000 4000 1000 1000
ip rsvp reservation-host 10.20.20.20 10.10.10.10 udp 5000 4000 ff rate 1000 1000
end
show ip rsvp sender
show ip rsvp reservation
show ip rsvp installed
show ip rsvp interface
copy running-config startup-config
# QoS_IntServ_RSVP_Admission_Control_Percentage_Bandwidth_Example
configure terminal
interface GigabitEthernet1
 ip rsvp bandwidth percent 50 percent 10
 exit
interface GigabitEthernet2
 ip rsvp bandwidth percent 50 percent 10
 exit
end
show ip rsvp interface
show ip rsvp installed
# QoS_IntServ_RSVP_Admission_Control_Ingress_CAC_Example
configure terminal
interface Tunnel0
 ip rsvp bandwidth ingress 200
 exit
end
show ip rsvp interface
show ip rsvp installed
# QoS_IntServ_RSVP_Admission_Control_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms RSVP path interfaces are operational | Interfaces are up/up |
| `show ip route <receiver-ip>` | Confirms routing toward the RSVP receiver or session address | Route points toward the expected next hop |
| `show ip route <sender-ip>` | Confirms reverse routing toward the sender | Route points back toward the expected sender side |
| `traceroute <receiver-ip>` | Confirms the routed RSVP path | Path crosses the intended routers |
| `show running-config interface <interface-id>` | Confirms interface RSVP configuration | Interface shows `ip rsvp bandwidth` |
| `show running-config \| include ip rsvp` | Quick RSVP configuration check | RSVP bandwidth, sender, or reservation commands appear |
| `show ip rsvp interface` | Verifies RSVP interface state and bandwidth pool | RSVP-enabled interfaces show reservable bandwidth |
| `show ip rsvp neighbor` | Verifies RSVP neighbors after signaling occurs | RSVP neighbor entries appear |
| `show ip rsvp sender` | Verifies RSVP PATH sender state | Sender state appears for the expected session |
| `show ip rsvp reservation` | Verifies RSVP RESV receiver reservation state | Reservation appears for the expected session |
| `show ip rsvp installed` | Verifies installed RSVP bandwidth state | Installed reservation and bandwidth information appears |
| `show ip rsvp host senders` | Verifies host-simulated sender state when supported | Sender host state appears |
| `show ip rsvp host receivers` | Verifies host-simulated receiver state when supported | Receiver host state appears |
# QoS_IntServ_RSVP_Admission_Control_Rollback
configure terminal
interface <RSVP_INTERFACE_1>
 no ip rsvp bandwidth
 no ip rsvp bandwidth ingress
 exit
interface <RSVP_INTERFACE_2>
 no ip rsvp bandwidth
 no ip rsvp bandwidth ingress
 exit
no ip rsvp sender-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> <BANDWIDTH> <BURST_SIZE>
no ip rsvp reservation-host <SESSION_IP> <SENDER_IP> udp <SESSION_DEST_PORT> <SENDER_SOURCE_PORT> ff rate <BANDWIDTH> <BURST_SIZE>
end
show ip rsvp interface
show ip rsvp sender
show ip rsvp reservation
show ip rsvp installed
show running-config | include ip rsvp
# QoS_IntServ_RSVP_Admission_Control_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `show ip rsvp interface` shows no RSVP interface | `ip rsvp bandwidth` is missing | `show running-config interface <interface-id>` | Configure `ip rsvp bandwidth` on the RSVP path interface |
| Sender state appears but reservation does not install | Receiver-side RESV is missing | `show ip rsvp sender` and `show ip rsvp reservation` | Configure `ip rsvp reservation-host` or fix the real RSVP receiver |
| Reservation fails across the path | RSVP is not enabled on every required transit interface | `traceroute <receiver-ip>` and `show ip rsvp interface` on each router | Enable RSVP on each transit interface in the routed path |
| Reservation is rejected | Requested bandwidth exceeds available RSVP pool | `show ip rsvp interface` and `show ip rsvp installed` | Lower reservation bandwidth or increase the RSVP bandwidth pool |
| One flow is rejected while total RSVP bandwidth remains | Single-flow limit is too low | `show running-config interface <interface-id>` | Increase the single-flow value in `ip rsvp bandwidth <total> <single-flow>` |
| RSVP messages follow the wrong path | Routing path does not match the lab design | `show ip route <session-ip>` and `traceroute <session-ip>` | Fix routing before troubleshooting RSVP |
| RSVP neighbor does not appear | No RSVP signaling has crossed the link yet | `show ip rsvp neighbor` | Generate PATH and RESV traffic with sender-host and reservation-host |
| PATH exists but RESV does not | Receiver side did not request the reservation | `show ip rsvp sender` and `show ip rsvp reservation` | Configure receiver simulation or verify endpoint RSVP support |
| Reservation appears but traffic still performs poorly | RSVP admission exists but packet treatment is not aligned | `show policy-map interface <interface-id>` | Add or fix LLQ, CBWFQ, shaping, or marking policy where needed |
| RSVP works one way only | Reverse path or reverse RSVP interface is missing | `show ip route <sender-ip>` and `show ip rsvp interface` | Fix reverse routing or enable RSVP on reverse-path interfaces |
| Ingress CAC test does not behave as expected | Ingress RSVP bandwidth pool is missing | `show running-config interface <interface-id>` | Add `ip rsvp bandwidth ingress <kbps>` or the percentage equivalent |
| Config includes `ip rsvp qos` but plain RSVP still fails | `ip rsvp qos` was mistaken for baseline IntServ RSVP enablement | `show running-config \| include ip rsvp qos` | Use interface-level `ip rsvp bandwidth` for the lab path |
| Config is lost after reload | Configuration was not saved | `show startup-config \| include ip rsvp` | Run `copy running-config startup-config` |
##### Source_Basis
# QoS_IntServ_RSVP_Admission_Control_Mental_Model
# QoS_IntServ_RSVP_Admission_Control_Configuration_Checklist
# QoS_IntServ_RSVP_Admission_Control_Skeleton
# QoS_IntServ_RSVP_Admission_Control_Example_Skeleton
# QoS_IntServ_RSVP_Admission_Control_Percentage_Bandwidth_Example
# QoS_IntServ_RSVP_Admission_Control_Ingress_CAC_Example
# QoS_IntServ_RSVP_Admission_Control_Verification_Commands
# QoS_IntServ_RSVP_Admission_Control_Rollback
# QoS_IntServ_RSVP_Admission_Control_Failure_Checks
