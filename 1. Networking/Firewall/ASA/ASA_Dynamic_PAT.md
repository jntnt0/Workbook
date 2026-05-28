# ASA_Dynamic_PAT_Mental_Model

| Concept                           | Operational Meaning                                                              |
| --------------------------------- | -------------------------------------------------------------------------------- |
| Dynamic PAT                       | Many inside hosts share one translated address, usually the outside interface IP |
| Source translation                | ASA changes the inside source IP and source port for outbound flows              |
| Return traffic                    | Return traffic is allowed only if it matches an existing connection/xlate        |
| Outside-initiated inbound traffic | Not allowed to initiate directly to inside hosts through Dynamic PAT             |
| Auto NAT / Object NAT             | Cleanest model for simple inside-to-outside PAT                                  |
| Section 2 NAT                     | Auto NAT rules appear in Section 2 of the ASA NAT table                          |
| Interface PAT                     | `dynamic interface` means translate to the egress interface IP                   |
| Xlate proof                       | `show xlate` proves the active PAT mapping                                       |
| Conn proof                        | `show conn` proves the stateful firewall session exists                          |
# ASA_Dynamic_PAT_Configuration_Checklist

| Step | Task                                                  | Device      | Command                                                                                | Expected Result                                                                              |
| ---: | ----------------------------------------------------- | ----------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
|    1 | Confirm routed ASA baseline is working                | ASA         | `show interface ip brief`                                                              | Inside and outside interfaces are up/up                                                      |
|    2 | Confirm logical interface names                       | ASA         | `show nameif`                                                                          | Interfaces are named correctly, usually `inside` and `outside`                               |
|    3 | Confirm inside network route exists                   | ASA         | `show route <inside-host-ip>`                                                          | ASA knows the inside network as connected or routed                                          |
|    4 | Confirm outside/default route exists                  | ASA         | `show route 0.0.0.0`                                                                   | ASA has a default route toward the outside next hop                                          |
|    5 | Review existing NAT rules                             | ASA         | `show nat`                                                                             | Existing NAT rules and sections are visible                                                  |
|    6 | Review detailed NAT rules and hit counts              | ASA         | `show nat detail`                                                                      | Existing NAT order and hit counts are visible                                                |
|    7 | Confirm no higher-priority NAT rule will override PAT | ASA         | `show nat detail`                                                                      | Section 1 identity/policy NAT does not unintentionally catch Internet-bound traffic          |
|    8 | Enter global configuration mode                       | ASA         | `configure terminal`                                                                   | ASA enters configuration mode                                                                |
|    9 | Create inside network object                          | ASA         | `object network <inside-network-object>`                                               | ASA enters network-object configuration mode                                                 |
|   10 | Define inside subnet                                  | ASA         | `subnet <inside-network> <inside-mask>`                                                | Inside network object represents the real inside subnet                                      |
|   11 | Add optional object description                       | ASA         | `description <description>`                                                            | Object purpose is documented                                                                 |
|   12 | Configure Dynamic PAT to outside interface            | ASA         | `nat (inside,outside) dynamic interface`                                               | Inside source addresses PAT to the outside interface IP                                      |
|   13 | Exit object configuration                             | ASA         | `exit`                                                                                 | ASA returns to global configuration mode                                                     |
|   14 | Save configuration                                    | ASA         | `write memory`                                                                         | Dynamic PAT configuration is saved                                                           |
|   15 | Verify NAT rule appears in Auto NAT section           | ASA         | `show nat`                                                                             | Dynamic PAT appears in Auto NAT Policies, usually Section 2                                  |
|   16 | Verify NAT rule details                               | ASA         | `show nat detail`                                                                      | Rule shows source object, interface pair, and hit count                                      |
|   17 | Simulate outbound flow                                | ASA         | `packet-tracer input inside tcp <inside-host-ip> <src-port> <internet-ip> 80 detailed` | NAT phase matches Dynamic PAT rule and final result is `ALLOW` if policy/routing are correct |
|   18 | Generate real outbound traffic                        | Inside host | `ping <internet-ip>` or `curl http://<internet-ip>`                                    | Traffic is sent from inside to outside                                                       |
|   19 | Verify active PAT translation                         | ASA         | `show xlate`                                                                           | Inside host is translated to outside interface IP with a port mapping                        |
|   20 | Verify local host state                               | ASA         | `show local-host <inside-host-ip>`                                                     | Xlate and connection information appear for the inside host                                  |
|   21 | Verify connection table                               | ASA         | `show conn`                                                                            | Outbound connection appears                                                                  |
|   22 | Verify NATed connection detail                        | ASA         | `show conn long`                                                                       | Translated address/port are visible in connection output                                     |
|   23 | Verify NAT hit count increments                       | ASA         | `show nat detail`                                                                      | Dynamic PAT rule hit count increases after connection build                                  |
|   24 | Verify no packet drops                                | ASA         | `show asp drop`                                                                        | No relevant drop counter increases for the test flow                                         |
|   25 | Confirm end-to-end return traffic                     | Inside host | `ping <internet-ip>` or application test                                               | Reply traffic returns successfully through the PAT mapping                                   |
# ASA_Dynamic_PAT_Basic_Skeleton

```text
configure terminal

object network <inside-network-object>
 subnet <inside-network> <inside-mask>
 description <description>
 nat (inside,outside) dynamic interface

write memory
```

# ASA_Dynamic_PAT_Example_Skeleton

```text
configure terminal

object network INSIDE-NET
 subnet 192.168.10.0 255.255.255.0
 description Internal Network Object with Dynamic PAT
 nat (inside,outside) dynamic interface

write memory
```

# ASA_Dynamic_PAT_Verification_Commands

| Task                           | Command                                                                                | Expected Result                                             |
| ------------------------------ | -------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Verify interface state         | `show interface ip brief`                                                              | Inside and outside interfaces are up/up                     |
| Verify logical interface names | `show nameif`                                                                          | `inside` and `outside` are mapped to the correct interfaces |
| Verify default route           | `show route 0.0.0.0`                                                                   | ASA has a default route toward outside                      |
| Verify NAT rule order          | `show nat`                                                                             | Dynamic PAT appears in the expected NAT section             |
| Verify NAT rule detail         | `show nat detail`                                                                      | Rule shows object NAT and hit counts                        |
| Verify packet path             | `packet-tracer input inside tcp <inside-host-ip> <src-port> <internet-ip> 80 detailed` | NAT phase matches Dynamic PAT and final result is expected  |
| Verify active translation      | `show xlate`                                                                           | PAT translation exists for the inside host                  |
| Verify host translation state  | `show local-host <inside-host-ip>`                                                     | Xlate and connection details are visible                    |
| Verify connection state        | `show conn`                                                                            | Outbound connection is built                                |
| Verify NATed connection view   | `show conn long`                                                                       | Translated address and port are visible                     |
| Verify drops                   | `show asp drop`                                                                        | No relevant drop reason increments during test              |

# ASA_Dynamic_PAT_Rollback

| Step | Task                              | Device | Command                                     | Expected Result                                    |
| ---: | --------------------------------- | ------ | ------------------------------------------- | -------------------------------------------------- |
|    1 | Identify Dynamic PAT rule         | ASA    | `show nat detail`                           | Object and NAT rule are identified                 |
|    2 | Enter object configuration        | ASA    | `object network <inside-network-object>`    | ASA enters the source object                       |
|    3 | Remove Dynamic PAT rule           | ASA    | `no nat (inside,outside) dynamic interface` | Dynamic PAT rule is removed from the object        |
|    4 | Exit object configuration         | ASA    | `exit`                                      | ASA returns to global configuration mode           |
|    5 | Remove object if no longer needed | ASA    | `no object network <inside-network-object>` | Object is deleted if unused                        |
|    6 | Clear stale translations          | ASA    | `clear xlate`                               | Existing PAT translations are cleared              |
|    7 | Clear stale connections if needed | ASA    | `clear conn address <inside-host-ip>`       | Existing connection state for test host is cleared |
|    8 | Verify NAT rule is gone           | ASA    | `show nat`                                  | Dynamic PAT rule no longer appears                 |
|    9 | Save rollback state               | ASA    | `write memory`                              | Rollback is saved                                  |

# ASA_Dynamic_PAT_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| No PAT translation appears | `show xlate` | Traffic did not match NAT rule, no real traffic occurred, or connection failed before translation |
| NAT rule has zero hits | `show nat detail` | Wrong source subnet, wrong interface pair, or higher-priority NAT rule matched first |
| Packet-tracer misses PAT rule | `packet-tracer input inside tcp <inside-host-ip> <src-port> <internet-ip> 80 detailed` | Object subnet, interface pair, or NAT order is wrong |
| Traffic still uses wrong NAT | `show nat detail` | Section 1 manual NAT or identity NAT is catching the flow first |
| Inside host cannot reach outside | `show route 0.0.0.0` | Missing or wrong default route |
| ASA cannot reach outside next hop | `show arp` | Next-hop ARP/adja­cency problem |
| Translation exists but no return traffic | `show conn detail` | Upstream route, outside path, ISP/router, or remote host issue |
| Return traffic dropped | `show asp drop` | TCP state, ACL, inspection, or route issue |
| ACL denies outbound traffic | `show access-group` and `show access-list <acl-name>` | ACL applied on inside or outside interface denies the flow |
| PAT works for one host but not another | `show local-host <host-ip>` | Source host is outside the NAT object subnet or routed incorrectly |
| ICMP fails but TCP works | `packet-tracer input inside icmp <inside-host-ip> 8 0 <internet-ip> detailed` | ICMP inspection/policy or return behavior differs from TCP test |
| Outside host cannot initiate to inside host | `show xlate` and `show conn` | Expected behavior for Dynamic PAT. Use static NAT/PAT for inbound publishing |
