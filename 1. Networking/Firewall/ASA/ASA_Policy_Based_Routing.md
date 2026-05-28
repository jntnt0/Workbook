
```
# Index
# Source_Basis
# ASA_Policy_Based_Routing_Mental_Model
# ASA_Policy_Based_Routing_Configuration_Checklist
# ASA_Policy_Based_Routing_Basic_Next_Hop_Skeleton
# ASA_Policy_Based_Routing_Service_Specific_Skeleton
# ASA_Policy_Based_Routing_Set_Interface_Skeleton
# ASA_Policy_Based_Routing_Tracked_Next_Hop_Skeleton
# ASA_Policy_Based_Routing_Primary_And_Backup_Next_Hop_Skeleton
# ASA_Policy_Based_Routing_Source_Based_Dual_ISP_Skeleton
# ASA_Policy_Based_Routing_Verification_Commands
# ASA_Policy_Based_Routing_Rollback
# ASA_Policy_Based_Routing_Failure_Checks
```
# ASA_Policy_Based_Routing_Mental_Model
| Concept                               | Operational Meaning                                                                                              |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Policy Based Routing                  | Overrides normal destination-based routing for selected traffic                                                  |
| Match condition                       | ASA uses an ACL inside a route map to identify traffic that should follow policy routing                         |
| Set action                            | Route map tells ASA which next hop, recursive next hop, interface, default next hop, or default interface to use |
| Ingress-only feature                  | PBR is applied on incoming traffic on the interface where `policy-route route-map` is configured                 |
| First-packet decision                 | ASA applies PBR when the first packet creates the flow; later packets follow the established connection entry    |
| Route map sequence                    | Lower sequence numbers are evaluated first                                                                       |
| Permit route-map + permit ACL         | PBR set action is applied                                                                                        |
| Permit route-map + deny ACL           | ASA continues evaluating later route-map entries or falls back to normal routing if no match applies             |
| `set ip next-hop`                     | Forces matched traffic to a specific next-hop IP                                                                 |
| `set ip next-hop verify-availability` | Uses tracking so ASA only selects the next hop if the track object is up                                         |
| `set interface`                       | Sends matched traffic out a chosen egress interface                                                              |
| Default next-hop action               | Used only if normal route lookup does not find a usable route                                                    |
| Normal routing fallback               | Traffic that does not match PBR uses the normal routing table                                                    |
| NAT interaction                       | NAT can influence or select egress behavior before/around the final forwarding path                              |
| Asymmetric routing risk               | PBR can break stateful firewalling if forward and return traffic use different paths                             |
| Routed-mode only                      | ASA PBR is not for transparent firewall mode                                                                     |
| Blunt rule                            | PBR is not a routing-table replacement. It is a controlled exception for specific ingress traffic                |

# ASA_Policy_Based_Routing_Configuration_Checklist
| Step | Task                                                                | Device          | Command                                                                                                                       | Expected Result                                                                    |                                             |
| ---: | ------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------- |
|    1 | Confirm ASA interface status                                        | ASA             | `show interface ip brief`                                                                                                     | Ingress and candidate egress interfaces are up/up                                  |                                             |
|    2 | Confirm logical interface names                                     | ASA             | `show nameif`                                                                                                                 | Interfaces are named correctly, such as `inside`, `outside1`, `outside2`, or `dmz` |                                             |
|    3 | Confirm ASA is routed mode                                          | ASA             | `show firewall`                                                                                                               | ASA is in routed firewall mode                                                     |                                             |
|    4 | Confirm source network for PBR                                      | ASA / Notes     | `<source-network> <source-mask>`                                                                                              | Traffic source that should be policy-routed is known                               |                                             |
|    5 | Confirm destination network or service for PBR                      | ASA / Notes     | `<destination-network> <destination-mask> <protocol>/<port>`                                                                  | Traffic destination/service match is known                                         |                                             |
|    6 | Confirm ingress interface                                           | ASA / Notes     | `<ingress-interface>`                                                                                                         | Interface receiving the traffic is known                                           |                                             |
|    7 | Confirm intended egress interface                                   | ASA / Notes     | `<egress-interface>`                                                                                                          | Interface that matched traffic should use is known                                 |                                             |
|    8 | Confirm intended next-hop IP                                        | ASA / Notes     | `<next-hop-ip>`                                                                                                               | Next hop reachable through the intended egress interface is known                  |                                             |
|    9 | Confirm route to next hop                                           | ASA             | `show route <next-hop-ip>`                                                                                                    | ASA can resolve the next hop through the expected egress interface                 |                                             |
|   10 | Confirm ARP/adjacency to next hop                                   | ASA             | `show arp                                                                                                                     | include <next-hop-ip>`                                                             | ASA can resolve next-hop L2 adjacency       |
|   11 | Confirm normal routing baseline                                     | ASA             | `show route <destination-ip>`                                                                                                 | Current destination-based route is known before applying PBR                       |                                             |
|   12 | Confirm existing NAT rules                                          | ASA             | `show nat detail`                                                                                                             | Existing NAT rules and hit counts are visible                                      |                                             |
|   13 | Confirm existing ACLs                                               | ASA             | `show access-group`                                                                                                           | Interface ACL bindings are known                                                   |                                             |
|   14 | Confirm existing route maps                                         | ASA             | `show running-config route-map`                                                                                               | Existing route maps are visible                                                    |                                             |
|   15 | Confirm existing PBR bindings                                       | ASA             | `show policy-route`                                                                                                           | Current interface-to-route-map bindings are visible                                |                                             |
|   16 | Confirm uRPF state if used                                          | ASA             | `show running-config ip verify reverse-path`                                                                                  | Reverse-path checks are identified before adding asymmetric PBR                    |                                             |
|   17 | Confirm VPN/NAT asymmetric risk                                     | ASA / Notes     | NAT/VPN path review                                                                                                           | Forward and return paths are expected to remain symmetric                          |                                             |
|   18 | Enter configuration mode                                            | ASA             | `configure terminal`                                                                                                          | ASA enters global configuration mode                                               |                                             |
|   19 | Create PBR ACL remark                                               | ASA             | `access-list <pbr-acl> remark Match traffic for <pbr-purpose>`                                                                | ACL purpose is documented                                                          |                                             |
|   20 | Match source-only traffic                                           | ASA             | `access-list <pbr-acl> extended permit ip <source-network> <source-mask> any`                                                 | ASA matches all traffic from the source network                                    |                                             |
|   21 | Match source-to-destination traffic                                 | ASA             | `access-list <pbr-acl> extended permit ip <source-network> <source-mask> <destination-network> <destination-mask>`            | ASA matches traffic between specific networks                                      |                                             |
|   22 | Match source-to-service traffic                                     | ASA             | `access-list <pbr-acl> extended permit tcp <source-network> <source-mask> <destination-network> <destination-mask> eq <port>` | ASA matches only the intended TCP service                                          |                                             |
|   23 | Create route map sequence                                           | ASA             | `route-map <route-map-name> permit <sequence>`                                                                                | ASA enters route-map configuration mode                                            |                                             |
|   24 | Attach ACL match                                                    | ASA             | `match ip address <pbr-acl>`                                                                                                  | Route map matches the PBR ACL                                                      |                                             |
|   25 | Set primary next hop                                                | ASA             | `set ip next-hop <next-hop-ip>`                                                                                               | Matched traffic is forwarded to the specified next hop                             |                                             |
|   26 | Set egress interface instead of next hop if design requires it      | ASA             | `set interface <egress-interface>`                                                                                            | Matched traffic is forwarded out the specified interface                           |                                             |
|   27 | Add default route-map permit sequence for normal fallback if needed | ASA             | `route-map <route-map-name> permit <fallback-sequence>`                                                                       | Non-matching traffic can fall through to normal route lookup                       |                                             |
|   28 | Enter ingress interface                                             | ASA             | `interface <ingress-interface>`                                                                                               | ASA enters ingress interface configuration mode                                    |                                             |
|   29 | Apply PBR to ingress interface                                      | ASA             | `policy-route route-map <route-map-name>`                                                                                     | Route map is applied to traffic entering this interface                            |                                             |
|   30 | Add static route needed for next-hop reachability                   | ASA             | `route <egress-interface> <next-hop-network> <mask> <gateway-ip>`                                                             | ASA can resolve the PBR next hop                                                   |                                             |
|   31 | Add normal fallback/default route if missing                        | ASA             | `route <fallback-interface> 0.0.0.0 0.0.0.0 <fallback-gateway>`                                                               | Non-PBR traffic has a normal forwarding path                                       |                                             |
|   32 | Adjust NAT only if PBR changes egress interface                     | ASA             | `show nat detail` then add/edit NAT as needed                                                                                 | NAT matches the egress interface actually used by PBR                              |                                             |
|   33 | Confirm interface ACL permits the flow                              | ASA             | `show access-list <interface-acl>`                                                                                            | Interface ACL permits intended traffic                                             |                                             |
|   34 | Save configuration                                                  | ASA             | `write memory`                                                                                                                | PBR configuration is saved                                                         |                                             |
|   35 | Verify route-map config                                             | ASA             | `show running-config route-map`                                                                                               | Route map has correct match and set clauses                                        |                                             |
|   36 | Verify PBR interface binding                                        | ASA             | `show policy-route`                                                                                                           | Ingress interface shows the expected route map                                     |                                             |
|   37 | Verify route-map counters                                           | ASA             | `show route-map`                                                                                                              | Policy routing match counters increment after traffic                              |                                             |
|   38 | Simulate traffic with packet-tracer                                 | ASA             | `packet-tracer input <ingress-interface> tcp <source-ip> <src-port> <destination-ip> <dst-port> detailed`                     | Output includes PBR lookup and expected egress interface/next hop                  |                                             |
|   39 | Generate real traffic                                               | Client / Router | `ping <destination-ip>` or service-specific test                                                                              | Traffic is sent from the matched source                                            |                                             |
|   40 | Verify PBR hit counters                                             | ASA             | `show route-map`                                                                                                              | Matching route-map sequence hit count increments                                   |                                             |
|   41 | Verify ASP PBR classification                                       | ASA             | `show asp table classify domain pbr`                                                                                          | PBR classify entry shows hits                                                      |                                             |
|   42 | Verify connection egress path                                       | ASA             | `show conn address <source-ip>`                                                                                               | Connection uses expected egress interface                                          |                                             |
|   43 | Verify NAT translation                                              | ASA             | `show xlate`                                                                                                                  | Translation matches the intended egress interface                                  |                                             |
|   44 | Verify ARP for next hop                                             | ASA             | `show arp                                                                                                                     | include <next-hop-ip>`                                                             | ASA has adjacency for the selected next hop |
|   45 | Debug only if needed                                                | ASA             | `debug policy-route`                                                                                                          | Debug shows route-map sequence, matched ACL, egress interface, and next hop        |                                             |
|   46 | Stop debug                                                          | ASA             | `undebug all`                                                                                                                 | Debug output stops                                                                 |                                             |
|   47 | Check drops if traffic fails                                        | ASA             | `show asp drop`                                                                                                               | Drop reason points to PBR, NAT, ACL, route, uRPF, inspection, or adjacency issue   |                                             |

```
# ASA_Policy_Based_Routing_Basic_Next_Hop_Skeleton
configure terminal

access-list <pbr-acl> remark Match traffic for PBR
access-list <pbr-acl> extended permit ip <source-network> <source-mask> <destination-network> <destination-mask>

route-map <route-map-name> permit <sequence>
 match ip address <pbr-acl>
 set ip next-hop <next-hop-ip>

interface <ingress-interface>
 policy-route route-map <route-map-name>

write memory
```

```
# ASA_Policy_Based_Routing_Service_Specific_Skeleton
configure terminal

access-list <pbr-acl> remark Match service-specific traffic for PBR
access-list <pbr-acl> extended permit tcp <source-network> <source-mask> <destination-network> <destination-mask> eq <destination-port>

route-map <route-map-name> permit <sequence>
 match ip address <pbr-acl>
 set ip next-hop <next-hop-ip>

interface <ingress-interface>
 policy-route route-map <route-map-name>

write memory
```


```
# ASA_Policy_Based_Routing_Set_Interface_Skeleton
configure terminal

access-list <pbr-acl> remark Match traffic for interface-based PBR
access-list <pbr-acl> extended permit ip <source-network> <source-mask> <destination-network> <destination-mask>

route-map <route-map-name> permit <sequence>
 match ip address <pbr-acl>
 set interface <egress-interface>

interface <ingress-interface>
 policy-route route-map <route-map-name>

write memory
```

```
# ASA_Policy_Based_Routing_Tracked_Next_Hop_Skeleton
configure terminal

sla monitor <sla-id>
 type echo protocol ipIcmpEcho <tracked-next-hop-ip> interface <egress-interface>
 frequency <seconds>
 timeout <milliseconds>
sla monitor schedule <sla-id> life forever start-time now

track <track-id> rtr <sla-id> reachability

access-list <pbr-acl> remark Match traffic for tracked PBR next-hop
access-list <pbr-acl> extended permit ip <source-network> <source-mask> <destination-network> <destination-mask>

route-map <route-map-name> permit <sequence>
 match ip address <pbr-acl>
 set ip next-hop verify-availability <tracked-next-hop-ip> <next-hop-sequence> track <track-id>

interface <ingress-interface>
 policy-route route-map <route-map-name>

write memory
```

```
# ASA_Policy_Based_Routing_Primary_And_Backup_Next_Hop_Skeleton
configure terminal

sla monitor <primary-sla-id>
 type echo protocol ipIcmpEcho <primary-next-hop-ip> interface <primary-egress-interface>
 frequency <seconds>
 timeout <milliseconds>
sla monitor schedule <primary-sla-id> life forever start-time now

track <primary-track-id> rtr <primary-sla-id> reachability

sla monitor <backup-sla-id>
 type echo protocol ipIcmpEcho <backup-next-hop-ip> interface <backup-egress-interface>
 frequency <seconds>
 timeout <milliseconds>
sla monitor schedule <backup-sla-id> life forever start-time now

track <backup-track-id> rtr <backup-sla-id> reachability

access-list <pbr-acl> remark Match traffic for primary and backup PBR next-hop
access-list <pbr-acl> extended permit ip <source-network> <source-mask> <destination-network> <destination-mask>

route-map <route-map-name> permit <sequence>
 match ip address <pbr-acl>
 set ip next-hop verify-availability <primary-next-hop-ip> 1 track <primary-track-id>
 set ip next-hop verify-availability <backup-next-hop-ip> 2 track <backup-track-id>

interface <ingress-interface>
 policy-route route-map <route-map-name>

write memory
```


```
# ASA_Policy_Based_Routing_Source_Based_Dual_ISP_Skeleton
configure terminal

access-list <isp1-pbr-acl> remark Source group routed toward ISP1
access-list <isp1-pbr-acl> extended permit ip <source-network-1> <source-mask-1> any

access-list <isp2-pbr-acl> remark Source group routed toward ISP2
access-list <isp2-pbr-acl> extended permit ip <source-network-2> <source-mask-2> any

route-map <route-map-name> permit 10
 match ip address <isp1-pbr-acl>
 set ip next-hop <isp1-next-hop-ip>

route-map <route-map-name> permit 20
 match ip address <isp2-pbr-acl>
 set ip next-hop <isp2-next-hop-ip>

interface <ingress-interface>
 policy-route route-map <route-map-name>

write memory
```


# ASA_Policy_Based_Routing_Verification_Commands
| Task                           | Command                                                                                                   | Expected Result                                                             |                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------- |
| Verify routed mode             | `show firewall`                                                                                           | ASA is in routed mode                                                       |                                  |
| Verify interface state         | `show interface ip brief`                                                                                 | Ingress and egress interfaces are up/up                                     |                                  |
| Verify interface names         | `show nameif`                                                                                             | Interface names match PBR/NAT/route references                              |                                  |
| Verify normal route lookup     | `show route <destination-ip>`                                                                             | Normal route path is known before PBR                                       |                                  |
| Verify next-hop route          | `show route <next-hop-ip>`                                                                                | ASA can reach selected next hop                                             |                                  |
| Verify ARP adjacency           | `show arp                                                                                                 | include <next-hop-ip>`                                                      | ASA has L2 adjacency to next hop |
| Verify ACL match               | `show access-list <pbr-acl>`                                                                              | PBR ACL exists and hit count increments                                     |                                  |
| Verify route-map configuration | `show running-config route-map`                                                                           | Route map has correct match and set clauses                                 |                                  |
| Verify PBR interface binding   | `show policy-route`                                                                                       | Ingress interface has correct route-map applied                             |                                  |
| Verify route-map counters      | `show route-map`                                                                                          | Matching sequence shows packet/byte counters                                |                                  |
| Verify tracked next-hop state  | `show route-map`                                                                                          | Verified next-hop track state shows up/down behavior                        |                                  |
| Verify SLA config              | `show running-config sla monitor`                                                                         | SLA monitor config exists if tracked PBR is used                            |                                  |
| Verify SLA operational state   | `show sla monitor operational-state`                                                                      | SLA probe is active and reachable if expected                               |                                  |
| Verify track state             | `show track`                                                                                              | Track object is up for usable next hop                                      |                                  |
| Verify PBR classifier          | `show asp table classify domain pbr`                                                                      | PBR classify entry exists and hit count increments                          |                                  |
| Verify packet-tracer PBR       | `packet-tracer input <ingress-interface> tcp <source-ip> <src-port> <destination-ip> <dst-port> detailed` | Output shows PBR lookup and expected egress/next-hop                        |                                  |
| Verify real connection path    | `show conn address <source-ip>`                                                                           | Flow uses expected egress interface                                         |                                  |
| Verify NAT result              | `show xlate`                                                                                              | Translation matches selected egress path                                    |                                  |
| Verify drops                   | `show asp drop`                                                                                           | No relevant route, ACL, NAT, uRPF, adjacency, or inspection drops increment |                                  |
| Debug PBR                      | `debug policy-route`                                                                                      | Debug shows matched ACL, route-map sequence, egress interface, and next-hop |                                  |
| Stop debug                     | `undebug all`                                                                                             | Debug output stops                                                          |                                  |

# ASA_Policy_Based_Routing_Rollback
| Step | Task                                         | Device | Command                                                                                                   | Expected Result                                              |
| ---: | -------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
|    1 | Identify applied PBR policy                  | ASA    | `show policy-route`                                                                                       | Ingress interface and route-map name are identified          |
|    2 | Identify route-map entries                   | ASA    | `show running-config route-map`                                                                           | Route-map sequences, ACLs, and set actions are identified    |
|    3 | Identify PBR ACL                             | ASA    | `show access-list <pbr-acl>`                                                                              | ACL entries and hit counts are visible                       |
|    4 | Identify SLA/track objects if used           | ASA    | `show running-config sla monitor` and `show running-config track`                                         | Tracking objects are identified                              |
|    5 | Enter configuration mode                     | ASA    | `configure terminal`                                                                                      | ASA enters global configuration mode                         |
|    6 | Remove PBR from ingress interface            | ASA    | `interface <ingress-interface>` then `no policy-route route-map <route-map-name>`                         | PBR is no longer applied on the interface                    |
|    7 | Remove route-map next-hop sequence           | ASA    | `no route-map <route-map-name> permit <sequence>`                                                         | PBR route-map sequence is removed                            |
|    8 | Remove PBR ACL if unused                     | ASA    | `clear configure access-list <pbr-acl>`                                                                   | PBR match ACL is removed                                     |
|    9 | Remove SLA schedule if used                  | ASA    | `no sla monitor schedule <sla-id>`                                                                        | SLA schedule is removed                                      |
|   10 | Remove SLA monitor if used                   | ASA    | `no sla monitor <sla-id>`                                                                                 | SLA monitor is removed                                       |
|   11 | Remove track object if used                  | ASA    | `no track <track-id>`                                                                                     | Track object is removed                                      |
|   12 | Remove PBR-specific static route if lab-only | ASA    | `no route <egress-interface> <next-hop-network> <mask> <gateway-ip>`                                      | Lab-only route is removed                                    |
|   13 | Remove PBR-specific NAT if lab-only          | ASA    | `no nat ...` or object NAT removal                                                                        | NAT path change is removed only if created for this PBR test |
|   14 | Clear connections for test source            | ASA    | `clear conn address <source-ip>`                                                                          | Existing flows are forced to re-evaluate routing             |
|   15 | Clear translations if NAT changed            | ASA    | `clear xlate`                                                                                             | Old translations are cleared                                 |
|   16 | Verify PBR removal                           | ASA    | `show policy-route`                                                                                       | Interface no longer has PBR route-map applied                |
|   17 | Verify normal route behavior                 | ASA    | `packet-tracer input <ingress-interface> tcp <source-ip> <src-port> <destination-ip> <dst-port> detailed` | Flow uses normal routing table                               |
|   18 | Save rollback state                          | ASA    | `write memory`                                                                                            | Rollback is saved                                            |
# ASA_Policy_Based_Routing_Failure_Checks
| Symptom                                           | Command                                                          | What Usually Broke                                                                              |
| ------------------------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| PBR never triggers                                | `show policy-route`                                              | Route map is not applied to the ingress interface                                               |
| PBR ACL has zero hits                             | `show access-list <pbr-acl>`                                     | Source, destination, protocol, port, or ingress interface is wrong                              |
| Route-map has zero hits                           | `show route-map`                                                 | ACL does not match or traffic is entering a different interface                                 |
| Packet uses normal route instead of PBR           | `packet-tracer input <ingress-interface> ... detailed`           | Route-map did not match or PBR was not applied inbound                                          |
| PBR selects wrong egress interface                | `debug policy-route`                                             | Wrong next hop, wrong set action, or route to next hop resolves differently than expected       |
| Next hop is not reachable                         | `show route <next-hop-ip>` and `show arp`                        | Missing route, missing ARP adjacency, or wrong egress interface                                 |
| Tracked next hop stays down                       | `show track` and `show sla monitor operational-state`            | SLA probe target unreachable, wrong interface, wrong timeout, or wrong track ID                 |
| Backup next hop never activates                   | `show route-map`                                                 | Track object still up, backup sequence wrong, or backup next hop not configured                 |
| Existing traffic ignores new PBR                  | `show conn address <source-ip>`                                  | ASA already has an established flow; clear connection or start a new flow                       |
| NAT uses wrong interface                          | `show nat detail` and `show xlate`                               | NAT rule does not match the PBR-selected egress interface                                       |
| Return traffic gets dropped                       | `show asp drop`                                                  | PBR created asymmetric routing and ASA stateful inspection rejects return path                  |
| uRPF drops traffic                                | `show running-config ip verify reverse-path` and `show asp drop` | Reverse path check uses routing table, not PBR result                                           |
| VPN traffic breaks after PBR                      | `show crypto ipsec sa`, `show nat detail`, `show asp drop`       | PBR changed path, NAT exemption no longer matches, or return path is asymmetric                 |
| Interface ACL blocks matched traffic              | `show access-list <interface-acl>`                               | Interface ACL does not permit the flow before forwarding succeeds                               |
| Route-map deny behaves unexpectedly               | `show running-config route-map`                                  | Route-map deny or ACL deny causes fallback/continued evaluation instead of forced drop          |
| Packet-tracer shows PBR but real traffic fails    | `show conn`, `show xlate`, `show asp drop`                       | Stale connection, stale translation, missing ARP, upstream next-hop issue, or return-path issue |
| PBR works for TCP but not ICMP                    | `show access-list <pbr-acl>`                                     | ACL only matches TCP or service-specific traffic                                                |
| PBR works from one subnet but not another         | `show access-list <pbr-acl>`                                     | Source network match is too narrow                                                              |
| PBR sends all traffic unexpectedly                | `show running-config route-map`                                  | Route-map sequence has no match statement, which means implicit match any                       |
| Traffic is dropped after final route-map sequence | `packet-tracer ... detailed`                                     | No usable set action and normal route lookup fails                                              |
| Debug floods terminal                             | `show debug`                                                     | `debug policy-route` was left enabled; run `undebug all`                                        |
