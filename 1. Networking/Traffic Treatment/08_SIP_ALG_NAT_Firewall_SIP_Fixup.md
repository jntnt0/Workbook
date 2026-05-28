


Cisco’s IOS XE SIP ALG hardening guide says SIP NAT support is enabled by default on port 5060, can be re-enabled with ip nat service sip {tcp | udp} port <port-number>, and disabled with no ip nat service sip. The same guide shows the IOS XE ZBF workflow using class-map type inspect, match protocol sip, policy-map type inspect, inspect, zone-pair security, and service-policy type inspect. Cisco ASA documentation uses Modular Policy Framework with inspect sip under a policy-map class, then applies it with service-policy.  

SIP_ALG_NAT_Firewall_SIP_Fixup.md

SIP_ALG_NAT_Firewall_SIP_Fixup

| Related lab | `qos-sip-alg-final` | Lab focused on SIP ALG/NAT/firewall fixup behavior, not QoS queueing |
# SIP_ALG_NAT_Firewall_SIP_Fixup_Mental_Model
| Concept | Operational Meaning |
|---|---|
| SIP | Signaling protocol used to create, modify, and tear down voice/video sessions |
| RTP | Media stream that carries the actual voice or video payload after SIP signaling negotiates it |
| SIP ALG | Application-layer gateway that inspects SIP and SDP payloads so NAT/firewall can understand embedded addressing and dynamic media ports |
| SIP fixup | ASA-style term for application inspection that helps protocols work through NAT/firewall state |
| Embedded IP problem | SIP/SDP can carry IP addresses and ports inside the payload, not just in the IP and TCP/UDP headers |
| NAT door | NAT translation opened based on SIP/SDP information so related media traffic can pass |
| Firewall pinhole | Temporary firewall allowance opened for related RTP or SIP flows |
| Signaling vs media | SIP usually uses TCP/UDP 5060 or TLS 5061. RTP media usually uses dynamic UDP port ranges |
| ALG benefit | Can help simple SIP-through-NAT designs where endpoints advertise private IP addresses in SIP/SDP |
| ALG risk | Can break modern SIP trunks, SBC-controlled designs, encrypted SIP, or provider-specific SIP behavior |
| Not QoS | SIP ALG does not prioritize voice. LLQ handles voice queueing. SIP ALG handles NAT/firewall inspection |
| IOS XE NAT SIP support | NAT support for SIP is commonly controlled with `ip nat service sip {tcp | udp} port <port>` |
| IOS XE ZBF SIP inspection | Zone-Based Firewall SIP inspection uses inspect class-maps, inspect policy-maps, and zone-pairs |
| ASA SIP inspection | ASA uses MPF with `inspect sip` under a policy-map class, commonly under `global_policy` |
| Disable when harmful | If SIP phones or trunks fail because the ALG rewrites signaling incorrectly, disable SIP ALG and rely on SBC/STUN/TURN/provider design |
# SIP_ALG_NAT_Firewall_SIP_Fixup_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm basic IP connectivity before SIP inspection changes | Router/Firewall | `show ip interface brief` | Inside, outside, and voice-facing interfaces are up/up |
| 2 | Confirm routing between SIP endpoints, proxy, PBX, or provider edge | Router/Firewall | `show ip route <sip-peer-ip>` | Route exists toward the SIP peer |
| 3 | Confirm NAT role on IOS XE interfaces if NAT is part of the lab | IOS XE Router | `show running-config interface <inside-interface>` / `show running-config interface <outside-interface>` | Inside interface has `ip nat inside`; outside interface has `ip nat outside` |
| 4 | Confirm existing NAT rules | IOS XE Router | `show running-config | include ip nat` | NAT overload, static NAT, or port-forwarding rules are known |
| 5 | Confirm current SIP NAT ALG state or explicit overrides | IOS XE Router | `show running-config | include ip nat service sip` | Explicit SIP NAT service configuration is visible if configured |
| 6 | Enable SIP NAT ALG for SIP over TCP on the required port | IOS XE Router | `ip nat service sip tcp port <sip-port>` | IOS XE NAT support for SIP over TCP is enabled on the selected port |
| 7 | Enable SIP NAT ALG for SIP over UDP on the required port | IOS XE Router | `ip nat service sip udp port <sip-port>` | IOS XE NAT support for SIP over UDP is enabled on the selected port |
| 8 | Use default SIP port unless the lab or provider requires otherwise | IOS XE Router | `ip nat service sip udp port 5060` / `ip nat service sip tcp port 5060` | SIP ALG is enabled for standard SIP signaling |
| 9 | Disable SIP NAT ALG if the lab is proving ALG breakage or provider requires it off | IOS XE Router | `no ip nat service sip` | SIP NAT ALG support is disabled |
| 10 | Disable SIP NAT ALG on a specific protocol and port if platform syntax supports it | IOS XE Router | `no ip nat service sip udp port 5060` / `no ip nat service sip tcp port 5060` | SIP ALG is removed for the specified transport and port |
| 11 | Create IOS XE ZBF source zone if firewall inspection is part of the lab | IOS XE Router | `zone security <SOURCE_ZONE>` | Source security zone exists |
| 12 | Create IOS XE ZBF destination zone | IOS XE Router | `zone security <DESTINATION_ZONE>` | Destination security zone exists |
| 13 | Build SIP inspect class-map | IOS XE Router | `class-map type inspect match-any <SIP_CLASS>` | Device enters inspect class-map mode |
| 14 | Match SIP protocol in the inspect class | IOS XE Router | `match protocol sip` | SIP traffic is classified for inspection |
| 15 | Build SIP inspect policy-map | IOS XE Router | `policy-map type inspect <SIP_POLICY>` | Device enters inspect policy-map mode |
| 16 | Enter SIP inspect class inside the policy | IOS XE Router | `class type inspect <SIP_CLASS>` | SIP class is selected for firewall action |
| 17 | Enable stateful SIP inspection | IOS XE Router | `inspect` | SIP traffic receives stateful inspection |
| 18 | Define default behavior for unmatched traffic | IOS XE Router | `class class-default` then `drop log` or `drop` | Nonmatching traffic is dropped unless another policy allows it |
| 19 | Create source-to-destination zone pair | IOS XE Router | `zone-pair security <ZONE_PAIR_NAME> source <SOURCE_ZONE> destination <DESTINATION_ZONE>` | Zone pair exists for the inspected traffic direction |
| 20 | Attach SIP inspect policy to the zone pair | IOS XE Router | `service-policy type inspect <SIP_POLICY>` | SIP inspection policy is active between zones |
| 21 | Assign source interface to source zone | IOS XE Router | `interface <source-interface>` then `zone-member security <SOURCE_ZONE>` | Source interface belongs to the correct zone |
| 22 | Assign destination interface to destination zone | IOS XE Router | `interface <destination-interface>` then `zone-member security <DESTINATION_ZONE>` | Destination interface belongs to the correct zone |
| 23 | Configure reverse zone-pair if return-initiated SIP flows must be allowed | IOS XE Router | `zone-pair security <REVERSE_ZONE_PAIR> source <DESTINATION_ZONE> destination <SOURCE_ZONE>` | Reverse direction has explicit firewall policy if needed |
| 24 | Confirm ZBF policy attachment | IOS XE Router | `show policy-map type inspect zone-pair` | SIP class and inspect action appear under the zone pair |
| 25 | Confirm SIP NAT translations during call setup | IOS XE Router | `show ip nat translations | include 5060` | SIP control translations appear when SIP traffic crosses NAT |
| 26 | Confirm dynamic media translations or related flows | IOS XE Router | `show ip nat translations | include udp` | RTP/media-related UDP translations appear during active calls |
| 27 | Confirm firewall sessions or inspected flows | IOS XE Router | `show policy-map type inspect zone-pair` | SIP inspect counters increment |
| 28 | Configure ASA SIP inspection under the default inspection class if ASA is the firewall | ASA | `policy-map global_policy` then `class inspection_default` then `inspect sip` | ASA global policy inspects SIP traffic |
| 29 | Attach ASA global policy if not already attached | ASA | `service-policy global_policy global` | ASA global inspection policy is active |
| 30 | Configure ASA custom SIP inspection map if SIP behavior must be controlled | ASA | `policy-map type inspect sip <SIP_MAP>` then `parameters` | Custom SIP inspection map exists |
| 31 | Attach ASA custom SIP inspection map | ASA | `inspect sip <SIP_MAP>` | ASA uses the custom SIP inspection behavior |
| 32 | Verify ASA SIP inspection policy | ASA | `show running-config policy-map` | `inspect sip` appears under the intended policy-map class |
| 33 | Verify ASA service-policy attachment | ASA | `show running-config service-policy` | Global or interface service-policy is attached |
| 34 | Generate SIP call setup traffic | SIP Phones/PBX/Test Hosts | Place a SIP call or generate SIP INVITE traffic | SIP signaling crosses the NAT/firewall |
| 35 | Verify SIP signaling and RTP media both work | SIP Phones/PBX/Test Hosts | Call test with two-way audio | Call establishes and audio works in both directions |
| 36 | Check for ALG-caused call failures | Router/Firewall | Inspect logs, translations, connections, and SIP endpoint errors | No one-way audio, failed registration, or broken INVITE/200 OK behavior |
| 37 | Save configuration after validation | Router/Firewall | `copy running-config startup-config` / ASA `write memory` | SIP ALG/firewall configuration survives reload |
# SIP_ALG_NAT_Firewall_SIP_Fixup_IOS_XE_NAT_Skeleton
configure terminal
interface <INSIDE_INTERFACE>
 ip nat inside
 exit
interface <OUTSIDE_INTERFACE>
 ip nat outside
 exit
ip access-list standard <NAT_ACL>
 permit <INSIDE_SUBNET> <WILDCARD_MASK>
 exit
ip nat inside source list <NAT_ACL> interface <OUTSIDE_INTERFACE> overload
ip nat service sip udp port 5060
ip nat service sip tcp port 5060
end
show running-config | include ip nat
show ip nat translations | include 5060
show ip nat statistics
copy running-config startup-config
# SIP_ALG_NAT_Firewall_SIP_Fixup_IOS_XE_ZBF_Skeleton
configure terminal
zone security <SOURCE_ZONE>
 exit
zone security <DESTINATION_ZONE>
 exit
class-map type inspect match-any <SIP_CLASS>
 match protocol sip
 exit
policy-map type inspect <SIP_POLICY>
 class type inspect <SIP_CLASS>
  inspect
 exit
 class class-default
  drop log
 exit
 exit
zone-pair security <ZONE_PAIR_NAME> source <SOURCE_ZONE> destination <DESTINATION_ZONE>
 service-policy type inspect <SIP_POLICY>
 exit
interface <SOURCE_INTERFACE>
 zone-member security <SOURCE_ZONE>
 exit
interface <DESTINATION_INTERFACE>
 zone-member security <DESTINATION_ZONE>
 exit
end
show policy-map type inspect zone-pair
show running-config | section zone
show running-config | section policy-map type inspect
copy running-config startup-config
# SIP_ALG_NAT_Firewall_SIP_Fixup_IOS_XE_Example_Skeleton
configure terminal
interface GigabitEthernet1
 description INSIDE_VOICE_LAN
 ip address 10.10.10.1 255.255.255.0
 ip nat inside
 zone-member security INSIDE
 exit
interface GigabitEthernet2
 description OUTSIDE_PROVIDER
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 zone-member security OUTSIDE
 exit
ip access-list standard NAT_INSIDE
 permit 10.10.10.0 0.0.0.255
 exit
ip nat inside source list NAT_INSIDE interface GigabitEthernet2 overload
ip nat service sip udp port 5060
ip nat service sip tcp port 5060
zone security INSIDE
 exit
zone security OUTSIDE
 exit
class-map type inspect match-any SIP_INSPECT_CLASS
 match protocol sip
 exit
policy-map type inspect SIP_INSPECT_POLICY
 class type inspect SIP_INSPECT_CLASS
  inspect
 exit
 class class-default
  drop log
 exit
 exit
zone-pair security INSIDE_TO_OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect SIP_INSPECT_POLICY
 exit
end
show running-config | include ip nat service sip
show ip nat translations | include 5060
show policy-map type inspect zone-pair
copy running-config startup-config
# SIP_ALG_NAT_Firewall_SIP_Fixup_ASA_Skeleton
configure terminal
class-map inspection_default
 match default-inspection-traffic
 exit
policy-map global_policy
 class inspection_default
  inspect sip
 exit
 exit
service-policy global_policy global
end
show running-config policy-map
show running-config service-policy
show service-policy inspect sip
show conn detail
write memory
# SIP_ALG_NAT_Firewall_SIP_Fixup_ASA_Custom_SIP_Map_Skeleton
configure terminal
policy-map type inspect sip <SIP_INSPECTION_MAP>
 parameters
  no im
 exit
 exit
policy-map global_policy
 class inspection_default
  inspect sip <SIP_INSPECTION_MAP>
 exit
 exit
service-policy global_policy global
end
show running-config policy-map
show running-config service-policy
show service-policy inspect sip
write memory
# SIP_ALG_NAT_Firewall_SIP_Fixup_Disable_SIP_ALG_Skeleton
configure terminal
no ip nat service sip
! If platform syntax supports protocol and port scoped removal:
no ip nat service sip udp port 5060
no ip nat service sip tcp port 5060
end
show running-config | include ip nat service sip
show ip nat translations | include 5060
copy running-config startup-config
# SIP_ALG_NAT_Firewall_SIP_Fixup_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms routed interfaces are operational | Inside, outside, and voice interfaces are up/up |
| `show ip route <sip-peer-ip>` | Confirms routing to SIP peer, PBX, proxy, or provider | Route points toward the expected next hop |
| `show running-config | include ip nat service sip` | Checks explicit IOS XE SIP NAT ALG configuration | Shows configured `ip nat service sip` lines or no explicit override |
| `show running-config | include ip nat` | Confirms IOS XE NAT baseline | NAT inside/outside and NAT rules appear |
| `show ip nat translations | include 5060` | Confirms SIP control traffic is translated | UDP or TCP 5060 translations appear during call setup |
| `show ip nat translations | include udp` | Checks media-related UDP translations | RTP/media UDP translations appear during active call |
| `show ip nat statistics` | Confirms NAT is active | NAT counters and translation counts increase |
| `show policy-map type inspect zone-pair` | Verifies IOS XE ZBF SIP inspect behavior | SIP class counters increment under the zone pair |
| `show running-config | section policy-map type inspect` | Confirms IOS XE inspect policy configuration | SIP class has `inspect` action |
| `show running-config | section zone-pair` | Confirms IOS XE zone-pair service-policy attachment | Zone pair shows `service-policy type inspect <SIP_POLICY>` |
| `show running-config interface <interface-id>` | Confirms IOS XE zone membership | Interface shows `zone-member security <ZONE_NAME>` |
| ASA `show running-config policy-map` | Confirms ASA SIP inspection action | `inspect sip` appears under the intended policy-map class |
| ASA `show running-config service-policy` | Confirms ASA service-policy attachment | Global or interface service-policy is active |
| ASA `show service-policy inspect sip` | Verifies ASA SIP inspection counters | SIP inspection counters increase during call setup |
| ASA `show conn detail` | Checks SIP and RTP connection state | SIP control and related media connections appear |
| ASA `show xlate` | Checks NAT translations through ASA | Expected translations appear for SIP endpoint or server |
| ASA `show asp drop` | Checks packet drops caused by inspection or policy | No unexpected SIP/RTP drops during call testing |
# SIP_ALG_NAT_Firewall_SIP_Fixup_Rollback
configure terminal
! IOS XE NAT SIP ALG rollback
no ip nat service sip
no ip nat service sip udp port 5060
no ip nat service sip tcp port 5060
! IOS XE ZBF rollback
zone-pair security <ZONE_PAIR_NAME> source <SOURCE_ZONE> destination <DESTINATION_ZONE>
 no service-policy type inspect <SIP_POLICY>
 exit
no policy-map type inspect <SIP_POLICY>
no class-map type inspect <SIP_CLASS>
interface <SOURCE_INTERFACE>
 no zone-member security <SOURCE_ZONE>
 exit
interface <DESTINATION_INTERFACE>
 no zone-member security <DESTINATION_ZONE>
 exit
no zone security <SOURCE_ZONE>
no zone security <DESTINATION_ZONE>
end
show running-config | include ip nat service sip
show policy-map type inspect zone-pair
show running-config | section zone
copy running-config startup-config
# SIP_ALG_NAT_Firewall_SIP_Fixup_ASA_Rollback
configure terminal
policy-map global_policy
 class inspection_default
  no inspect sip
 exit
 exit
no policy-map type inspect sip <SIP_INSPECTION_MAP>
end
show running-config policy-map
show running-config service-policy
show service-policy inspect sip
write memory
# SIP_ALG_NAT_Firewall_SIP_Fixup_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| SIP registration fails through NAT | Routing, NAT, or SIP ALG behavior is wrong | `show ip route <sip-peer-ip>` and `show ip nat translations | include 5060` | Fix routing/NAT first, then retest SIP ALG |
| SIP call sets up but has one-way audio | RTP media ports are not being translated or allowed | `show ip nat translations | include udp` and ASA `show conn detail` | Allow RTP range, fix NAT, or validate ALG pinhole behavior |
| SIP signaling crosses but media never appears | Firewall pinhole is not opening for RTP | `show policy-map type inspect zone-pair` or ASA `show service-policy inspect sip` | Enable SIP inspection or explicitly permit required RTP ports |
| SIP works without firewall but fails with ZBF | Zone-pair or inspect policy is missing | `show running-config | section zone-pair` | Create correct source-to-destination zone-pair and attach SIP inspect policy |
| Traffic is dropped after assigning zone membership | Interface was added to a zone without a matching zone-pair policy | `show running-config interface <interface-id>` and `show running-config | section zone-pair` | Add zone-pair and service-policy for the traffic direction |
| SIP class counters stay at zero | `match protocol sip` is not matching the traffic | `show policy-map type inspect zone-pair` | Verify SIP port, protocol, and whether traffic is encrypted or nonstandard |
| SIP uses nonstandard port and ALG does not work | SIP ALG is only enabled for the wrong port | `show running-config | include ip nat service sip` | Configure `ip nat service sip udp port <port>` or `ip nat service sip tcp port <port>` |
| SIP TLS on 5061 does not inspect like clear SIP | Payload is encrypted | Packet capture, endpoint logs, ASA policy output | Use SBC/TLS proxy design where supported, or do not expect normal cleartext SIP ALG behavior |
| Provider says disable SIP ALG | ALG rewrite conflicts with provider SBC behavior | SIP provider logs and failed INVITE/200 OK/ACK flow | Disable SIP ALG using `no ip nat service sip` |
| SIP works until multiple calls share the trunk | ALG state or Call-ID handling issue | NAT/firewall sessions and SIP call logs | Prefer SBC design or tune/disable ALG depending on provider requirement |
| ASA `inspect sip` is configured but not active | Service-policy is not applied | ASA `show running-config service-policy` | Apply `service-policy global_policy global` or correct interface policy |
| ASA custom SIP map has no effect | `inspect sip <map>` is not attached under active policy | ASA `show running-config policy-map` | Attach the SIP map under the active class |
| NAT translation exists but SIP payload still advertises private IP | ALG is disabled or cannot parse the SIP/SDP payload | Packet capture on both sides | Enable SIP ALG for clear SIP or fix endpoint/SBC advertised address |
| Calls fail only after disabling ALG | Endpoints depended on ALG rewrite for NAT traversal | SIP packet capture and endpoint configuration | Configure SBC, static public signaling/media address, STUN/TURN, or re-enable ALG |
| QoS policy does not improve SIP call setup | SIP ALG is not a QoS mechanism | `show policy-map interface <interface-id>` | Use LLQ/CBWFQ separately for voice media treatment |
##### Source_Basis
# SIP_ALG_NAT_Firewall_SIP_Fixup_Mental_Model
# SIP_ALG_NAT_Firewall_SIP_Fixup_Configuration_Checklist
# SIP_ALG_NAT_Firewall_SIP_Fixup_IOS_XE_NAT_Skeleton
# SIP_ALG_NAT_Firewall_SIP_Fixup_IOS_XE_ZBF_Skeleton
# SIP_ALG_NAT_Firewall_SIP_Fixup_IOS_XE_Example_Skeleton
# SIP_ALG_NAT_Firewall_SIP_Fixup_ASA_Skeleton
# SIP_ALG_NAT_Firewall_SIP_Fixup_ASA_Custom_SIP_Map_Skeleton
# SIP_ALG_NAT_Firewall_SIP_Fixup_Disable_SIP_ALG_Skeleton
# SIP_ALG_NAT_Firewall_SIP_Fixup_Verification_Commands
# SIP_ALG_NAT_Firewall_SIP_Fixup_Rollback
# SIP_ALG_NAT_Firewall_SIP_Fixup_ASA_Rollback
# SIP_ALG_NAT_Firewall_SIP_Fixup_Failure_Checks