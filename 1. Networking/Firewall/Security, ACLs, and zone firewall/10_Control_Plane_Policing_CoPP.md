
Control_Plane_Policing_CoPP.md
# Control_Plane_Policing_CoPP
# Control_Plane_Policing_CoPP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| CoPP | Control Plane Policing protects traffic punted to the router or switch CPU |
| Control plane | Traffic destined to, sourced by, or processed by the device CPU |
| Data plane | Transit traffic forwarded through hardware or CEF; CoPP is not meant to police normal transit forwarding |
| MQC model | CoPP uses class maps, policy maps, policing actions, and `service-policy` under `control-plane` |
| Input CoPP | Most common deployment model; limits traffic coming into the device control plane |
| Output CoPP | Less common; controls traffic sourced from the device control plane when supported |
| Classification | ACLs match protocol families such as ICMP, management, routing, IPsec, and initialization traffic |
| Policing | Rate-limits matched control-plane traffic with conform, exceed, and violate actions |
| Learning mode | Safer initial mode where critical classes use `violate-action transmit` while counters are observed |
| Enforcement mode | Hardened mode where excessive or unknown traffic uses `violate-action drop` |
| Class-default | Captures unexpected punted traffic; should normally be low-volume and monitored carefully |
| Platform default CoPP | Some Catalyst platforms include default CoPP policies; do not overwrite them blindly |
| Not an ACL replacement | CoPP protects CPU resources; management ACLs, VTY ACLs, ZBFW, and MPP still control access policy |
| Lab purpose | Prove critical control-plane traffic remains reachable, excessive traffic is policed, and counters show which classes are hit |
# Control_Plane_Policing_CoPP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify platform and existing CoPP behavior | R1 | `show version`<br>`show policy-map control-plane input` | Platform is known and any existing CoPP policy is identified |
| 2 | Check existing control-plane service policy | R1 | `show running-config \| section control-plane` | Existing `service-policy input` or `service-policy output` is known |
| 3 | Confirm CPU baseline before enforcement | R1 | `show processes cpu sorted`<br>`show processes cpu history` | Normal CPU use is understood before policing |
| 4 | Identify open management and control-plane services | R1 | `show control-plane host open-ports` | SSH, SNMP, routing, and other CPU-facing services are visible if supported |
| 5 | Confirm management reachability before CoPP | Management Host | `ssh <router_ip>`<br>`ping <router_ip>` | Management access works before applying policy |
| 6 | Confirm routing protocol state before CoPP | R1 | `show ip protocols`<br>`show ip ospf neighbor`<br>`show ip eigrp neighbors`<br>`show bgp ipv4 unicast summary` | Existing routing adjacencies are stable before CoPP |
| 7 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 8 | Create ICMP and traceroute classification ACL | R1 | `ip access-list extended ACL-CoPP-ICMP` | CLI enters extended ACL mode |
| 9 | Match ICMP and traceroute traffic | R1 | `permit icmp any any echo-reply`<br>`permit icmp any any ttl-exceeded`<br>`permit icmp any any unreachable`<br>`permit icmp any any echo`<br>`permit udp any any range 33434 33463 ttl eq 1` | ICMP and traceroute-like traffic are classified |
| 10 | Exit ICMP ACL | R1 | `exit` | CLI returns to global configuration mode |
| 11 | Create IPsec and tunneling classification ACL | R1 | `ip access-list extended ACL-CoPP-IPsec` | CLI enters extended ACL mode |
| 12 | Match IPsec and GRE control-plane-related traffic | R1 | `permit esp any any`<br>`permit gre any any`<br>`permit udp any eq isakmp any eq isakmp`<br>`permit udp any any eq non500-isakmp`<br>`permit udp any eq non500-isakmp any` | ESP, GRE, IKE, and NAT-T style traffic are classified |
| 13 | Exit IPsec ACL | R1 | `exit` | CLI returns to global configuration mode |
| 14 | Create initialization ACL | R1 | `ip access-list extended ACL-CoPP-Initialize` | CLI enters extended ACL mode |
| 15 | Match DHCP initialization traffic | R1 | `permit udp any eq bootps any eq bootpc` | DHCP server-to-client initialization traffic is classified |
| 16 | Exit initialization ACL | R1 | `exit` | CLI returns to global configuration mode |
| 17 | Create management classification ACL | R1 | `ip access-list extended ACL-CoPP-Management` | CLI enters extended ACL mode |
| 18 | Match management traffic | R1 | `permit udp any eq ntp any`<br>`permit udp any any eq snmp`<br>`permit tcp any any eq 22`<br>`permit tcp any eq 22 any established` | NTP, SNMP, SSH, and SSH return traffic are classified |
| 19 | Exit management ACL | R1 | `exit` | CLI returns to global configuration mode |
| 20 | Create routing classification ACL | R1 | `ip access-list extended ACL-CoPP-Routing` | CLI enters extended ACL mode |
| 21 | Match routing and multicast control traffic | R1 | `permit tcp any eq bgp any established`<br>`permit eigrp any host 224.0.0.10`<br>`permit ospf any host 224.0.0.5`<br>`permit ospf any host 224.0.0.6`<br>`permit pim any host 224.0.0.13`<br>`permit igmp any any` | Common routing and multicast control traffic are classified |
| 22 | Add unicast routing protocol matches if the topology requires them | R1 | `permit ospf <neighbor_ip> 0.0.0.0 any`<br>`permit eigrp <neighbor_ip> 0.0.0.0 any`<br>`permit pim host <neighbor_ip> any` | Unicast protocol packets are classified when multicast-only entries are insufficient |
| 23 | Exit routing ACL | R1 | `exit` | CLI returns to global configuration mode |
| 24 | Create IPsec class map | R1 | `class-map match-all CLASS-CoPP-IPsec`<br>`match access-group name ACL-CoPP-IPsec` | IPsec and tunnel traffic class is created |
| 25 | Create routing class map | R1 | `class-map match-all CLASS-CoPP-Routing`<br>`match access-group name ACL-CoPP-Routing` | Routing control traffic class is created |
| 26 | Create initialization class map | R1 | `class-map match-all CLASS-CoPP-Initialize`<br>`match access-group name ACL-CoPP-Initialize` | Initialization traffic class is created |
| 27 | Create management class map | R1 | `class-map match-all CLASS-CoPP-Management`<br>`match access-group name ACL-CoPP-Management` | Management traffic class is created |
| 28 | Create ICMP class map | R1 | `class-map match-all CLASS-CoPP-ICMP`<br>`match access-group name ACL-CoPP-ICMP` | ICMP traffic class is created |
| 29 | Create CoPP policy map | R1 | `policy-map POLICY-CoPP` | CLI enters policy-map mode |
| 30 | Police ICMP traffic | R1 | `class CLASS-CoPP-ICMP`<br>`police 8000 conform-action transmit exceed-action transmit violate-action drop` | ICMP is rate-limited and excessive traffic is dropped |
| 31 | Police IPsec traffic in learning-safe mode | R1 | `class CLASS-CoPP-IPsec`<br>`police 64000 conform-action transmit exceed-action transmit violate-action transmit` | IPsec traffic is measured without dropping violate traffic during initial tuning |
| 32 | Police initialization traffic | R1 | `class CLASS-CoPP-Initialize`<br>`police 8000 conform-action transmit exceed-action transmit violate-action drop` | DHCP initialization traffic is rate-limited |
| 33 | Police management traffic in learning-safe mode | R1 | `class CLASS-CoPP-Management`<br>`police 32000 conform-action transmit exceed-action transmit violate-action transmit` | Management traffic is measured without immediate hard drops |
| 34 | Police routing traffic in learning-safe mode | R1 | `class CLASS-CoPP-Routing`<br>`police 64000 conform-action transmit exceed-action transmit violate-action transmit` | Routing traffic is measured without immediate hard drops |
| 35 | Police unknown control-plane traffic | R1 | `class class-default`<br>`police 8000 conform-action transmit exceed-action transmit violate-action drop` | Unexpected punted traffic is limited and excessive traffic is dropped |
| 36 | Exit policy-map mode | R1 | `exit` | CLI returns to global configuration mode |
| 37 | Apply policy to control plane input | R1 | `control-plane`<br>`service-policy input POLICY-CoPP` | CoPP policy is active for traffic entering the control plane |
| 38 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 39 | Verify policy is attached | R1 | `show policy-map control-plane input` | CoPP classes and policing counters are displayed |
| 40 | Verify management still works | Management Host | `ssh <router_ip>`<br>`ping <router_ip>` | Approved management traffic still reaches the device |
| 41 | Verify routing still works | R1 | `show ip ospf neighbor`<br>`show ip eigrp neighbors`<br>`show bgp ipv4 unicast summary` | Existing adjacencies remain stable |
| 42 | Generate controlled test traffic | Test Host | `ping <router_ip>`<br>`traceroute <router_ip>`<br>`ssh <router_ip>` | CoPP class counters increment under expected classes |
| 43 | Observe class-default | R1 | `show policy-map control-plane input` | Class-default traffic is minimal or investigated |
| 44 | Save configuration | R1 | `write memory` | CoPP configuration is saved |
# Control_Plane_Policing_CoPP_Learning_Mode_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Apply CoPP with critical classes set to transmit on violate | R1 | `policy-map POLICY-CoPP` | Policy exists with conservative learning-safe behavior |
| 2 | Keep routing and management violate action as transmit initially | R1 | `class CLASS-CoPP-Routing`<br>`police 64000 conform-action transmit exceed-action transmit violate-action transmit`<br>`class CLASS-CoPP-Management`<br>`police 32000 conform-action transmit exceed-action transmit violate-action transmit` | Routing and management traffic is measured before hard dropping |
| 3 | Apply policy to control plane | R1 | `control-plane`<br>`service-policy input POLICY-CoPP` | CoPP is active |
| 4 | Observe counters during normal operation | R1 | `show policy-map control-plane input` | Conform, exceed, and violate counters reveal normal traffic rates |
| 5 | Investigate class-default traffic | R1 | `show policy-map control-plane input` | Unexpected traffic in class-default is identified before tightening policy |
| 6 | Use packet capture if class-default is not understood | R1 | `monitor capture <capture_name> control-plane in` | Unknown punted traffic can be captured on platforms that support EPC control-plane capture |
| 7 | Adjust ACLs and class rates based on observations | R1 | `show access-lists`<br>`show policy-map control-plane input` | CoPP classes classify expected traffic and reduce class-default noise |
| 8 | Move to enforcement only after management and routing are proven stable | R1 | `show ip protocols`<br>`show ip ospf neighbor`<br>`show bgp ipv4 unicast summary` | Critical protocols remain stable before stricter drops are configured |
# Control_Plane_Policing_CoPP_Enforcement_Mode_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm current counters support stricter enforcement | R1 | `show policy-map control-plane input` | Normal traffic is within selected police rates |
| 2 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 3 | Enter CoPP policy | R1 | `policy-map POLICY-CoPP` | CLI enters CoPP policy-map mode |
| 4 | Tighten management class only if management rate is understood | R1 | `class CLASS-CoPP-Management`<br>`police 32000 conform-action transmit exceed-action transmit violate-action drop` | Excessive management traffic is dropped |
| 5 | Tighten routing class only if protocol stability is understood | R1 | `class CLASS-CoPP-Routing`<br>`police 64000 conform-action transmit exceed-action transmit violate-action drop` | Excessive routing control traffic is dropped |
| 6 | Tighten IPsec class only if tunnel/control behavior is understood | R1 | `class CLASS-CoPP-IPsec`<br>`police 64000 conform-action transmit exceed-action transmit violate-action drop` | Excessive IPsec-related traffic is dropped |
| 7 | Keep class-default restrictive | R1 | `class class-default`<br>`police 8000 conform-action transmit exceed-action transmit violate-action drop` | Unknown control-plane traffic remains rate-limited |
| 8 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 9 | Verify critical services immediately | R1 | `show ip ospf neighbor`<br>`show bgp ipv4 unicast summary`<br>`show policy-map control-plane input` | Routing and management remain functional |
| 10 | Save only after stability is confirmed | R1 | `write memory` | Enforced CoPP policy is saved |
# Control_Plane_Policing_CoPP_Skeleton
configure terminal
ip access-list extended ACL-CoPP-ICMP
 permit icmp any any echo-reply
 permit icmp any any ttl-exceeded
 permit icmp any any unreachable
 permit icmp any any echo
 permit udp any any range 33434 33463 ttl eq 1
exit
ip access-list extended ACL-CoPP-IPsec
 permit esp any any
 permit gre any any
 permit udp any eq isakmp any eq isakmp
 permit udp any any eq non500-isakmp
 permit udp any eq non500-isakmp any
exit
ip access-list extended ACL-CoPP-Initialize
 permit udp any eq bootps any eq bootpc
exit
ip access-list extended ACL-CoPP-Management
 permit udp any eq ntp any
 permit udp any any eq snmp
 permit tcp any any eq 22
 permit tcp any eq 22 any established
exit
ip access-list extended ACL-CoPP-Routing
 permit tcp any eq bgp any established
 permit eigrp any host 224.0.0.10
 permit ospf any host 224.0.0.5
 permit ospf any host 224.0.0.6
 permit pim any host 224.0.0.13
 permit igmp any any
exit
class-map match-all CLASS-CoPP-IPsec
 match access-group name ACL-CoPP-IPsec
exit
class-map match-all CLASS-CoPP-Routing
 match access-group name ACL-CoPP-Routing
exit
class-map match-all CLASS-CoPP-Initialize
 match access-group name ACL-CoPP-Initialize
exit
class-map match-all CLASS-CoPP-Management
 match access-group name ACL-CoPP-Management
exit
class-map match-all CLASS-CoPP-ICMP
 match access-group name ACL-CoPP-ICMP
exit
policy-map POLICY-CoPP
 class CLASS-CoPP-ICMP
  police 8000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-IPsec
  police 64000 conform-action transmit exceed-action transmit violate-action transmit
 class CLASS-CoPP-Initialize
  police 8000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-Management
  police 32000 conform-action transmit exceed-action transmit violate-action transmit
 class CLASS-CoPP-Routing
  police 64000 conform-action transmit exceed-action transmit violate-action transmit
 class class-default
  police 8000 conform-action transmit exceed-action transmit violate-action drop
exit
control-plane
 service-policy input POLICY-CoPP
exit
end
write memory
# Control_Plane_Policing_CoPP_Management_Restricted_Skeleton
configure terminal
ip access-list extended ACL-CoPP-Management
 permit udp <ntp_server_ip> 0.0.0.0 any eq ntp
 permit udp <snmp_manager_ip> 0.0.0.0 any eq snmp
 permit tcp <mgmt_subnet> <mgmt_wildcard> any eq 22
 permit tcp any eq 22 <mgmt_subnet> <mgmt_wildcard> established
exit
class-map match-all CLASS-CoPP-Management
 match access-group name ACL-CoPP-Management
exit
policy-map POLICY-CoPP
 class CLASS-CoPP-Management
  police 32000 conform-action transmit exceed-action transmit violate-action transmit
exit
control-plane
 service-policy input POLICY-CoPP
exit
end
write memory
# Control_Plane_Policing_CoPP_Stricter_Enforcement_Skeleton
configure terminal
policy-map POLICY-CoPP
 class CLASS-CoPP-ICMP
  police 8000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-IPsec
  police 64000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-Initialize
  police 8000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-Management
  police 32000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-Routing
  police 64000 conform-action transmit exceed-action transmit violate-action drop
 class class-default
  police 8000 conform-action transmit exceed-action transmit violate-action drop
exit
end
write memory
# Control_Plane_Policing_CoPP_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Existing CoPP policy | R1 | `show policy-map control-plane input` | Active control-plane input policy and counters are displayed |
| Control-plane config | R1 | `show running-config \| section control-plane` | `service-policy input POLICY-CoPP` is present |
| ACL definitions | R1 | `show access-lists ACL-CoPP-ICMP`<br>`show access-lists ACL-CoPP-Management`<br>`show access-lists ACL-CoPP-Routing` | CoPP match ACLs exist and counters increment |
| Class maps | R1 | `show class-map` | CoPP class maps reference the expected ACLs |
| Policy map | R1 | `show policy-map POLICY-CoPP` | Police rates and actions are visible |
| Control-plane counters | R1 | `show policy-map control-plane input` | Conformed, exceeded, and violated counters are visible per class |
| CPU state | R1 | `show processes cpu sorted` | CPU remains stable during normal and test traffic |
| CPU history | R1 | `show processes cpu history` | No sustained CPU spike from control-plane traffic |
| Management test | Management Host | `ssh <router_ip>` | SSH still works from approved source |
| ICMP test | Test Host | `ping <router_ip>` | ICMP reaches the device but counters increment under ICMP class |
| Traceroute test | Test Host | `traceroute <router_ip>` | Traceroute-like packets increment expected counters |
| OSPF stability | R1 | `show ip ospf neighbor` | OSPF neighbors remain full if OSPF is used |
| EIGRP stability | R1 | `show ip eigrp neighbors` | EIGRP neighbors remain stable if EIGRP is used |
| BGP stability | R1 | `show bgp ipv4 unicast summary` | BGP neighbors remain established if BGP is used |
| IPsec path sanity | R1 | `show crypto ikev2 sa`<br>`show crypto ipsec sa` | IPsec control or tunnel state remains stable if IPsec is used |
| Class-default check | R1 | `show policy-map control-plane input` | Class-default counters are low or investigated |
| Open port visibility | R1 | `show control-plane host open-ports` | Exposed CPU-facing services are known |
| Logging check | R1 | `show logging` | Related control-plane or policy messages appear if supported |
# Control_Plane_Policing_CoPP_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 2 | Enter control-plane configuration mode | R1 | `control-plane` | CLI enters control-plane mode |
| 3 | Remove CoPP input service policy | R1 | `no service-policy input POLICY-CoPP` | CoPP policy is detached from the control plane |
| 4 | Exit control-plane mode | R1 | `exit` | CLI returns to global configuration mode |
| 5 | Remove CoPP policy map if not reused | R1 | `no policy-map POLICY-CoPP` | CoPP policy map is deleted |
| 6 | Remove CoPP class maps if not reused | R1 | `no class-map CLASS-CoPP-IPsec`<br>`no class-map CLASS-CoPP-Routing`<br>`no class-map CLASS-CoPP-Initialize`<br>`no class-map CLASS-CoPP-Management`<br>`no class-map CLASS-CoPP-ICMP` | CoPP class maps are deleted |
| 7 | Remove CoPP ACLs if not reused | R1 | `no ip access-list extended ACL-CoPP-ICMP`<br>`no ip access-list extended ACL-CoPP-IPsec`<br>`no ip access-list extended ACL-CoPP-Initialize`<br>`no ip access-list extended ACL-CoPP-Management`<br>`no ip access-list extended ACL-CoPP-Routing` | CoPP match ACLs are deleted |
| 8 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 9 | Confirm control-plane policy removal | R1 | `show policy-map control-plane input` | Removed policy no longer appears |
| 10 | Confirm config cleanup | R1 | `show running-config \| include CoPP\|control-plane` | Removed CoPP objects are absent |
| 11 | Verify management access after rollback | Management Host | `ssh <router_ip>` | Management access works according to remaining access controls |
| 12 | Save rollback | R1 | `write memory` | Rollback is saved |
# Control_Plane_Policing_CoPP_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| SSH to router fails after CoPP | Management ACL does not match SSH source or port | R1 | `show access-lists ACL-CoPP-Management`<br>`show policy-map control-plane input` | Correct management ACL or temporarily set management violate action to transmit |
| SNMP stops working | SNMP manager source is not matched or police rate is too low | R1 | `show access-lists ACL-CoPP-Management`<br>`show policy-map control-plane input` | Add SNMP manager match and tune police rate |
| NTP stops working | NTP direction or server source is not matched | R1 | `show access-lists ACL-CoPP-Management` | Add correct NTP source and destination match |
| OSPF neighbor drops | OSPF packets are not classified or are over-policed | R1 | `show ip ospf neighbor`<br>`show access-lists ACL-CoPP-Routing`<br>`show policy-map control-plane input` | Add OSPF multicast or unicast matches and raise/tune routing class rate |
| EIGRP neighbor drops | EIGRP traffic not matched or strict policing drops it | R1 | `show ip eigrp neighbors`<br>`show access-lists ACL-CoPP-Routing` | Add correct EIGRP match and tune routing class |
| BGP session resets | BGP traffic direction is not matched or rate is too low | R1 | `show bgp ipv4 unicast summary`<br>`show access-lists ACL-CoPP-Routing` | Add correct BGP source/destination matches and tune rate |
| Routing traffic hits class-default | ACL only matches multicast routing packets while topology uses unicast packets | R1 | `show policy-map control-plane input`<br>`show access-lists ACL-CoPP-Routing` | Add unicast protocol-specific ACL entries |
| Class-default counters are high | Unknown punted traffic is not classified | R1 | `show policy-map control-plane input` | Capture or inspect traffic, then add a specific class or block it intentionally |
| CPU remains high after CoPP | Traffic is not control-plane traffic or policy is not applied | R1 | `show policy-map control-plane input`<br>`show processes cpu sorted` | Confirm traffic path and apply policy under `control-plane` |
| Counters do not increment | Test traffic is transit traffic, not traffic to the device CPU | R1 | `show policy-map control-plane input`<br>`show ip cef <destination>` | Test traffic destined to the router itself or punted traffic |
| Policy-map command is rejected under control-plane | Platform or image has restricted/default CoPP model | R1 | `show version`<br>`show policy-map control-plane input` | Use platform-specific CoPP syntax or leave default policy intact |
| Existing platform default CoPP disappears or changes | Custom policy replaced default policy without review | R1 | `show running-config \| section control-plane` | Restore platform default or use supported modification method |
| Policing is too aggressive | CIR, burst, or violate drop is too low for normal traffic | R1 | `show policy-map control-plane input` | Increase rate or return critical class to learning mode |
| ICMP troubleshooting fails | ICMP class drops excessive ping/traceroute traffic | R1 | `show policy-map control-plane input` | Raise ICMP rate temporarily or reduce test volume |
| IPsec tunnel control fails | IKE, ESP, GRE, or NAT-T traffic is missing from IPsec class | R1 | `show access-lists ACL-CoPP-IPsec`<br>`show crypto ikev2 sa` | Add missing tunnel/control protocol match |
| CoPP blocks DHCP relay or initialization behavior | DHCP initialization traffic is missing or over-policed | R1 | `show access-lists ACL-CoPP-Initialize`<br>`show policy-map control-plane input` | Add correct bootpc/bootps direction and tune rate |
| Logging does not explain drops | CoPP policing counters are the main evidence, not ACL deny logs | R1 | `show policy-map control-plane input` | Use CoPP counters, ACL counters, CPU data, and packet capture |
| Control-plane capture is unsupported | Platform does not support EPC control-plane capture syntax | R1 | `show monitor capture ?` | Use interface capture, SPAN, logs, or temporary broader transmit actions |
| Rollback does not restore access | VTY ACL, MPP, ZBFW self-zone, interface ACL, or AAA issue remains | R1 | `show running-config \| section line vty`<br>`show access-lists`<br>`show zone-pair security` | Troubleshoot remaining management-plane controls |
# Control_Plane_Policing_CoPP_Related_Labs
- control-plane-policing-final
# Control_Plane_Policing_CoPP_Index
Control_Plane_Policing_CoPP.md
# Control_Plane_Policing_CoPP
# Control_Plane_Policing_CoPP_Mental_Model
# Control_Plane_Policing_CoPP_Configuration_Checklist
# Control_Plane_Policing_CoPP_Learning_Mode_Checklist
# Control_Plane_Policing_CoPP_Enforcement_Mode_Checklist
# Control_Plane_Policing_CoPP_Skeleton
# Control_Plane_Policing_CoPP_Management_Restricted_Skeleton
# Control_Plane_Policing_CoPP_Stricter_Enforcement_Skeleton
# Control_Plane_Policing_CoPP_Verification_Commands
# Control_Plane_Policing_CoPP_Rollback
# Control_Plane_Policing_CoPP_Failure_Checks
# Control_Plane_Policing_CoPP_Related_Labs
# Control_Plane_Policing_CoPP_Index

