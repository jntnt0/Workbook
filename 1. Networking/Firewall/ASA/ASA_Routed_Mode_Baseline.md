# ASA_Routed_Mode_Baseline_Configuration_Checklist

| Step | Task                                                          | Device        | Command                                                                                   | Expected Result                                                    |
| ---: | ------------------------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
|    1 | Confirm ASA is booted and reachable                           | ASA           | `show version`                                                                            | ASA version and platform information display                       |
|    2 | Confirm current firewall mode                                 | ASA           | `show firewall`                                                                           | Firewall mode shows routed, not transparent                        |
|    3 | Review current interface state                                | ASA           | `show interface ip brief`                                                                 | Interfaces and IP state are visible                                |
|    4 | Review current logical interface names                        | ASA           | `show nameif`                                                                             | Existing `nameif` mappings are visible                             |
|    5 | Enter global configuration mode                               | ASA           | `configure terminal`                                                                      | ASA enters configuration mode                                      |
|    6 | Configure hostname if needed                                  | ASA           | `hostname <asa-name>`                                                                     | Prompt changes to the configured hostname                          |
|    7 | Enter outside interface                                       | ASA           | `interface <outside-interface>`                                                           | ASA enters outside interface configuration                         |
|    8 | Enable outside interface                                      | ASA           | `no shutdown`                                                                             | Interface is administratively enabled                              |
|    9 | Assign outside logical name                                   | ASA           | `nameif outside`                                                                          | Interface is named `outside`                                       |
|   10 | Assign outside security level                                 | ASA           | `security-level 0`                                                                        | Outside interface has security level 0                             |
|   11 | Assign outside IP address                                     | ASA           | `ip address <outside-ip> <outside-mask>`                                                  | Outside interface has expected IP address                          |
|   12 | Enter inside interface                                        | ASA           | `interface <inside-interface>`                                                            | ASA enters inside interface configuration                          |
|   13 | Enable inside interface                                       | ASA           | `no shutdown`                                                                             | Interface is administratively enabled                              |
|   14 | Assign inside logical name                                    | ASA           | `nameif inside`                                                                           | Interface is named `inside`                                        |
|   15 | Assign inside security level                                  | ASA           | `security-level 100`                                                                      | Inside interface has security level 100                            |
|   16 | Assign inside IP address                                      | ASA           | `ip address <inside-ip> <inside-mask>`                                                    | Inside interface has expected IP address                           |
|   17 | Configure DMZ interface if used                               | ASA           | `interface <dmz-interface>`                                                               | ASA enters DMZ interface configuration                             |
|   18 | Enable DMZ interface if used                                  | ASA           | `no shutdown`                                                                             | DMZ interface is administratively enabled                          |
|   19 | Assign DMZ logical name if used                               | ASA           | `nameif dmz`                                                                              | Interface is named `dmz`                                           |
|   20 | Assign DMZ security level if used                             | ASA           | `security-level <dmz-level>`                                                              | DMZ has a security level between inside and outside                |
|   21 | Assign DMZ IP address if used                                 | ASA           | `ip address <dmz-ip> <dmz-mask>`                                                          | DMZ interface has expected IP address                              |
|   22 | Permit same-security inter-interface traffic only if required | ASA           | `same-security-traffic permit inter-interface`                                            | Same-security interfaces can pass traffic between each other       |
|   23 | Configure default route toward outside next hop               | ASA           | `route outside 0.0.0.0 0.0.0.0 <outside-next-hop> 1`                                      | ASA has a default route toward the outside network                 |
|   24 | Configure route to inside remote networks if needed           | ASA           | `route inside <remote-inside-network> <mask> <inside-next-hop> 1`                         | ASA can reach inside networks behind an inside router              |
|   25 | Configure route to DMZ remote networks if needed              | ASA           | `route dmz <remote-dmz-network> <mask> <dmz-next-hop> 1`                                  | ASA can reach DMZ-side routed networks                             |
|   26 | Create inbound ACL only if lower-security traffic must enter  | ASA           | `access-list <acl-name> extended permit <protocol> <source> <destination> <service>`      | Specific inbound traffic is permitted                              |
|   27 | Apply inbound ACL to the correct interface                    | ASA           | `access-group <acl-name> in interface <interface-name>`                                   | ACL is active inbound on the intended interface                    |
|   28 | Save configuration                                            | ASA           | `write memory`                                                                            | Configuration is saved                                             |
|   29 | Verify interface operational state                            | ASA           | `show interface ip brief`                                                                 | Configured interfaces show correct IPs and are up/up               |
|   30 | Verify logical interface mapping                              | ASA           | `show nameif`                                                                             | Interfaces show correct names and security levels                  |
|   31 | Verify routing table                                          | ASA           | `show route`                                                                              | Connected routes, default route, and static routes are present     |
|   32 | Verify ACL application                                        | ASA           | `show access-group`                                                                       | ACL is applied to the intended interface and direction             |
|   33 | Verify ACL contents and hit counts                            | ASA           | `show access-list <acl-name>`                                                             | ACEs are present and hit counts increase after traffic             |
|   34 | Emulate packet flow through ASA                               | ASA           | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port>` | Final result should be `ALLOW` for expected traffic                |
|   35 | Verify connection build                                       | ASA           | `show conn`                                                                               | Expected connection appears after real traffic                     |
|   36 | Verify drops if traffic fails                                 | ASA           | `show asp drop`                                                                           | Drop counters help identify the failure reason                     |
|   37 | Test end-to-end reachability                                  | Host / Router | `ping <remote-ip>`                                                                        | Reachability succeeds if routing, ACL, and return path are correct |
# ASA_Routed_Mode_Basic_Skeleton

```text
configure terminal

interface <outside-interface>
 no shutdown
 nameif outside
 security-level 0
 ip address <outside-ip> <outside-mask>

interface <inside-interface>
 no shutdown
 nameif inside
 security-level 100
 ip address <inside-ip> <inside-mask>

route outside 0.0.0.0 0.0.0.0 <outside-next-hop> 1

write memory
```


# ASA_Routed_Mode_Three_Zone_Skeleton

```text
configure terminal

interface <outside-interface>
 no shutdown
 nameif outside
 security-level 0
 ip address <outside-ip> <outside-mask>

interface <inside-interface>
 no shutdown
 nameif inside
 security-level 100
 ip address <inside-ip> <inside-mask>

interface <dmz-interface>
 no shutdown
 nameif dmz
 security-level 50
 ip address <dmz-ip> <dmz-mask>

route outside 0.0.0.0 0.0.0.0 <outside-next-hop> 1

write memory
```

# ASA_Routed_Mode_Inbound_ACL_Skeleton

```text
configure terminal

access-list <acl-name> extended permit <protocol> <source> <destination> <service>
access-list <acl-name> extended deny ip any any log
access-group <acl-name> in interface <interface-name>

write memory
```


# ASA_Routed_Mode_Baseline_Verification_Commands

| Task                            | Command                                                                                   | Expected Result                                                       |
| ------------------------------- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Verify firewall mode            | `show firewall`                                                                           | ASA is in routed mode                                                 |
| Verify interface state          | `show interface ip brief`                                                                 | Interfaces show correct IPs and are up/up                             |
| Verify logical names            | `show nameif`                                                                             | `inside`, `outside`, and optional `dmz` are mapped correctly          |
| Verify detailed interface stats | `show interface <interface>`                                                              | Interface is up with expected IP, speed, duplex, and counters         |
| Verify route table              | `show route`                                                                              | Connected routes and static routes are present                        |
| Verify specific route           | `show route <destination-ip>`                                                             | ASA selects expected egress interface and next hop                    |
| Verify ACL binding              | `show access-group`                                                                       | ACL is applied to correct interface and direction                     |
| Verify ACL hit counts           | `show access-list <acl-name>`                                                             | Hit counts increase after matching traffic                            |
| Verify packet decision          | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port>` | Packet-tracer result is `ALLOW` for expected flow                     |
| Verify connection table         | `show conn`                                                                               | Real traffic creates expected connection entry                        |
| Verify drops                    | `show asp drop`                                                                           | Drop reason points to ACL, route, NAT, inspection, or adjacency issue |
| Verify saved config             | `show startup-config`                                                                     | Startup config contains intended baseline                             |

# ASA_Routed_Mode_Baseline_Rollback

| Step | Task                                               | Device | Command                                                    | Expected Result                    |
| ---: | -------------------------------------------------- | ------ | ---------------------------------------------------------- | ---------------------------------- |
|    1 | Remove inbound ACL from interface                  | ASA    | `no access-group <acl-name> in interface <interface-name>` | ACL is no longer applied           |
|    2 | Remove ACL entries if no longer needed             | ASA    | `clear configure access-list <acl-name>`                   | ACL is removed                     |
|    3 | Remove static route                                | ASA    | `no route <interface-name> <network> <mask> <gateway>`     | Static route is removed            |
|    4 | Shut unused interface                              | ASA    | `interface <interface>` then `shutdown`                    | Interface is administratively down |
|    5 | Remove interface IP address                        | ASA    | `interface <interface>` then `no ip address`               | Interface IP is removed            |
|    6 | Remove logical name if decommissioning interface   | ASA    | `interface <interface>` then `no nameif`                   | Logical name is removed            |
|    7 | Remove security level if decommissioning interface | ASA    | `interface <interface>` then `no security-level`           | Security level is removed          |
|    8 | Save rollback state                                | ASA    | `write memory`                                             | Rollback is saved                  |
# ASA_Routed_Mode_Baseline_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| Interface does not pass traffic | `show interface ip brief` | Interface is shut, no IP address, no `nameif`, or physical link is down |
| Interface has no logical role | `show nameif` | Missing `nameif` |
| ASA cannot reach next hop | `show route` and `ping <next-hop-ip>` | Wrong interface IP, wrong mask, bad L2 adjacency, or missing connected route |
| ASA has no path to destination | `show route <destination-ip>` | Missing static/default route |
| Return traffic fails | `show route <source-ip>` | Missing return route on ASA or adjacent router |
| Inbound lower-to-higher traffic fails | `show access-group` | ACL missing or applied to wrong interface/direction |
| ACL exists but traffic still fails | `show access-list <acl-name>` | ACE order wrong, object wrong, service wrong, or hit count not increasing |
| Packet-tracer denies traffic | `packet-tracer input <ingress-nameif> ...` | ASA decision phase shows ACL, route, NAT, inspect, or adjacency failure |
| No connection entry appears | `show conn` | Flow never passed policy or never received return traffic |
| Silent drops occur | `show asp drop` | ASA is dropping due to route, ACL, inspection, TCP state, or adjacency issue |
| Same-security interfaces cannot communicate | `show running-config same-security-traffic` | Missing `same-security-traffic permit inter-interface` |
| Lab expects Internet-style outbound traffic but fails | `show xlate` | NAT may be required, but NAT belongs in separate NAT note |
| Inside-to-outside ping fails but ASA config looks right | Adjacent router `show ip route` | Return path missing outside the ASA |
