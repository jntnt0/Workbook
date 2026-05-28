# ASA_Static_PAT_Port_Range_Forwarding_Mental_Model

| Concept                      | Operational Meaning                                                                                                                                                 |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Static PAT port range        | One public IP and a range of public ports maps to one internal host and matching or translated service port range                                                   |
| Manual NAT                   | Port range forwarding usually uses Manual NAT / twice NAT style syntax, not simple object Auto NAT                                                                  |
| Service object               | Defines the TCP or UDP port range used by the NAT rule                                                                                                              |
| Source service in NAT syntax | Because the NAT rule is written from the real/server-side interface toward outside, the service object often uses source ports even though clients initiate inbound |
| Public IP object             | Represents the mapped public address used by outside clients                                                                                                        |
| Internal server object       | Represents the real private server receiving the forwarded traffic                                                                                                  |
| Inbound ACL                  | Still required for outside-initiated traffic and should reference the real/internal server IP in modern ASA syntax                                                  |
| NAT is not permission        | The NAT rule creates translation; ACL, route, server response, and inspection still matter                                                                          |
| Best use case                | Services that require a contiguous passive/data/media port range, such as passive FTP-like designs, RTP-like ranges, or application-specific UDP/TCP ranges         |
# ASA_Static_PAT_Port_Range_Forwarding_Configuration_Checklist

| Step | Task                                              | Device         | Command                                                                                                                                            | Expected Result                                                                                |
| ---: | ------------------------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
|    1 | Confirm ASA routed baseline is working            | ASA            | `show interface ip brief`                                                                                                                          | Inside/DMZ and outside interfaces are up/up                                                    |
|    2 | Confirm logical interface names                   | ASA            | `show nameif`                                                                                                                                      | Server-side and outside interfaces have correct `nameif` values                                |
|    3 | Confirm route to internal server                  | ASA            | `show route <inside-server-ip>`                                                                                                                    | ASA reaches the server through the expected inside/DMZ interface                               |
|    4 | Confirm outside/default route                     | ASA            | `show route 0.0.0.0`                                                                                                                               | ASA has a route toward outside clients                                                         |
|    5 | Confirm internal server return path               | Server / ASA   | Server default gateway check or `show conn` after testing                                                                                          | Server replies return through ASA                                                              |
|    6 | Review existing NAT rules                         | ASA            | `show nat`                                                                                                                                         | Existing NAT rules and sections are visible                                                    |
|    7 | Review detailed NAT order                         | ASA            | `show nat detail`                                                                                                                                  | Existing NAT rules do not conflict with the port-range rule                                    |
|    8 | Confirm protocol and port range                   | ASA / Notes    | `<tcp-or-udp> <real-start-port> <real-end-port>`                                                                                                   | Protocol and internal service range are known                                                  |
|    9 | Confirm mapped public range                       | ASA / Notes    | `<mapped-start-port> <mapped-end-port>`                                                                                                            | Public port range is known                                                                     |
|   10 | Confirm real and mapped ranges have matching size | ASA / Notes    | Compare real range to mapped range                                                                                                                 | Ranges line up cleanly for one-to-one port range translation                                   |
|   11 | Confirm public IP choice                          | ASA / Notes    | `<public-ip>`                                                                                                                                      | Public IP object is known                                                                      |
|   12 | Enter global configuration mode                   | ASA            | `configure terminal`                                                                                                                               | ASA enters configuration mode                                                                  |
|   13 | Create internal server object                     | ASA            | `object network <inside-server-object>`                                                                                                            | ASA enters network object mode                                                                 |
|   14 | Define internal server IP                         | ASA            | `host <inside-server-ip>`                                                                                                                          | Object represents the real server                                                              |
|   15 | Create public IP object                           | ASA            | `object network <public-ip-object>`                                                                                                                | ASA enters network object mode                                                                 |
|   16 | Define public mapped IP                           | ASA            | `host <public-ip>`                                                                                                                                 | Object represents the mapped public IP                                                         |
|   17 | Create real service range object                  | ASA            | `object service <real-service-object>`                                                                                                             | ASA enters service object mode                                                                 |
|   18 | Define real TCP service range                     | ASA            | `service tcp source range <real-start-port> <real-end-port>`                                                                                       | Service object represents the real TCP port range                                              |
|   19 | Or define real UDP service range                  | ASA            | `service udp source range <real-start-port> <real-end-port>`                                                                                       | Service object represents the real UDP port range                                              |
|   20 | Create mapped service range object                | ASA            | `object service <mapped-service-object>`                                                                                                           | ASA enters service object mode                                                                 |
|   21 | Define mapped TCP service range                   | ASA            | `service tcp source range <mapped-start-port> <mapped-end-port>`                                                                                   | Service object represents the mapped TCP port range                                            |
|   22 | Or define mapped UDP service range                | ASA            | `service udp source range <mapped-start-port> <mapped-end-port>`                                                                                   | Service object represents the mapped UDP port range                                            |
|   23 | Configure manual Static PAT port range            | ASA            | `nat (<server-side-nameif>,outside) source static <inside-server-object> <public-ip-object> service <real-service-object> <mapped-service-object>` | Public IP and mapped port range translate to the real server and real port range               |
|   24 | Create inbound ACL for TCP range                  | ASA            | `access-list <outside-acl> extended permit tcp <source> host <inside-server-ip> range <real-start-port> <real-end-port>`                           | Outside clients are permitted to reach the real server TCP range                               |
|   25 | Or create inbound ACL for UDP range               | ASA            | `access-list <outside-acl> extended permit udp <source> host <inside-server-ip> range <real-start-port> <real-end-port>`                           | Outside clients are permitted to reach the real server UDP range                               |
|   26 | Apply inbound ACL to outside interface            | ASA            | `access-group <outside-acl> in interface outside`                                                                                                  | ACL is active inbound on outside                                                               |
|   27 | Save configuration                                | ASA            | `write memory`                                                                                                                                     | Port range Static PAT configuration is saved                                                   |
|   28 | Verify NAT rule appears                           | ASA            | `show nat`                                                                                                                                         | Manual NAT port range rule appears in NAT table                                                |
|   29 | Verify NAT details                                | ASA            | `show nat detail`                                                                                                                                  | Rule shows source static mapping and service range objects                                     |
|   30 | Verify ACL is applied                             | ASA            | `show access-group`                                                                                                                                | Outside ACL is applied inbound                                                                 |
|   31 | Verify ACL contents                               | ASA            | `show access-list <outside-acl>`                                                                                                                   | ACL permits the real server and real service range                                             |
|   32 | Simulate outside-to-inside TCP flow               | ASA            | `packet-tracer input outside tcp <outside-client-ip> <src-port> <public-ip> <mapped-test-port> detailed`                                           | ASA untranslates public IP/port to real server/real port and expected result is `ALLOW`        |
|   33 | Simulate outside-to-inside UDP flow               | ASA            | `packet-tracer input outside udp <outside-client-ip> <src-port> <public-ip> <mapped-test-port> detailed`                                           | ASA untranslates public IP/port to real server/real port and expected result is `ALLOW`        |
|   34 | Generate real outside traffic                     | Outside client | Application test to `<public-ip>:<mapped-port>`                                                                                                    | Traffic reaches the internal server if policy and routing are correct                          |
|   35 | Verify xlate entry                                | ASA            | `show xlate`                                                                                                                                       | Static PAT range translation appears                                                           |
|   36 | Verify connection entry                           | ASA            | `show conn`                                                                                                                                        | Connection appears for outside client and real server                                          |
|   37 | Verify NATed connection detail                    | ASA            | `show conn long`                                                                                                                                   | Real and mapped address/port details are visible                                               |
|   38 | Verify NAT hit count                              | ASA            | `show nat detail`                                                                                                                                  | Static PAT port range rule hit count increments                                                |
|   39 | Verify ACL hit count                              | ASA            | `show access-list <outside-acl>`                                                                                                                   | Permit ACE hit count increments                                                                |
|   40 | Check drops if test fails                         | ASA            | `show asp drop`                                                                                                                                    | Drop reason points to ACL, NAT, route, inspection, TCP state, UDP handling, or adjacency issue |
# ASA_Static_PAT_Port_Range_TCP_Skeleton

```text
configure terminal

object network <inside-server-object>
 host <inside-server-ip>

object network <public-ip-object>
 host <public-ip>

object service <real-service-object>
 service tcp source range <real-start-port> <real-end-port>

object service <mapped-service-object>
 service tcp source range <mapped-start-port> <mapped-end-port>

nat (<server-side-nameif>,outside) source static <inside-server-object> <public-ip-object> service <real-service-object> <mapped-service-object>

access-list <outside-acl> extended permit tcp <source> host <inside-server-ip> range <real-start-port> <real-end-port>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Static_PAT_Port_Range_UDP_Skeleton

```text
configure terminal

object network <inside-server-object>
 host <inside-server-ip>

object network <public-ip-object>
 host <public-ip>

object service <real-service-object>
 service udp source range <real-start-port> <real-end-port>

object service <mapped-service-object>
 service udp source range <mapped-start-port> <mapped-end-port>

nat (<server-side-nameif>,outside) source static <inside-server-object> <public-ip-object> service <real-service-object> <mapped-service-object>

access-list <outside-acl> extended permit udp <source> host <inside-server-ip> range <real-start-port> <real-end-port>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Static_PAT_Port_Range_Same_Range_Example_Skeleton

```text
configure terminal

object network INSIDE-SERVER
 host 10.10.10.2

object network PUBLIC-IP
 host 1.1.1.20

object service UDP-RANGE-50000-65500
 service udp source range 50000 65500

nat (inside,outside) source static INSIDE-SERVER PUBLIC-IP service UDP-RANGE-50000-65500 UDP-RANGE-50000-65500

access-list OUTSIDE-IN extended permit udp any host 10.10.10.2 range 50000 65500
access-group OUTSIDE-IN in interface outside

write memory
```

# ASA_Static_PAT_Port_Range_Forwarding_Verification_Commands

| Task                           | Command                                                                                                  | Expected Result                                             |
| ------------------------------ | -------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Verify interface state         | `show interface ip brief`                                                                                | Server-side and outside interfaces are up/up                |
| Verify nameifs                 | `show nameif`                                                                                            | Server-side and outside interface names are correct         |
| Verify route to server         | `show route <inside-server-ip>`                                                                          | ASA routes to server through expected interface             |
| Verify outside route           | `show route 0.0.0.0`                                                                                     | ASA has outside/default route                               |
| Verify NAT table               | `show nat`                                                                                               | Manual Static PAT range rule appears                        |
| Verify NAT details             | `show nat detail`                                                                                        | Source static mapping and service range objects are visible |
| Verify ACL binding             | `show access-group`                                                                                      | Outside ACL is applied inbound                              |
| Verify ACL hit count           | `show access-list <outside-acl>`                                                                         | Permit ACE hit count increases during test                  |
| Verify packet-tracer TCP       | `packet-tracer input outside tcp <outside-client-ip> <src-port> <public-ip> <mapped-test-port> detailed` | Destination is untranslated to real server and real port    |
| Verify packet-tracer UDP       | `packet-tracer input outside udp <outside-client-ip> <src-port> <public-ip> <mapped-test-port> detailed` | Destination is untranslated to real server and real port    |
| Verify xlate                   | `show xlate`                                                                                             | Static PAT range translation exists                         |
| Verify connection              | `show conn`                                                                                              | Connection is built to real server                          |
| Verify NATed connection detail | `show conn long`                                                                                         | Real/mapped IP and port details are visible                 |
| Verify local host state        | `show local-host <inside-server-ip>`                                                                     | Xlate and connection state appear for the server            |
| Verify drops                   | `show asp drop`                                                                                          | No relevant drop reason increments during intended traffic  |

# ASA_Static_PAT_Port_Range_ACL_Address_Rule

| ASA NAT Model     | Inbound ACL Destination      | Operational Rule                                                                                              |
| ----------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ASA 8.3 and later | Real/internal server IP      | ACL should permit traffic to `<inside-server-ip>` and the real service port range                             |
| Mapped public IP  | Used by outside clients only | Do not use this as the ACL destination in modern ASA NAT logic unless the lab explicitly uses legacy behavior |
| Real port range   | Use in outside ACL           | NAT handles mapped-to-real service translation; ACL permits the real destination service                      |

# ASA_Static_PAT_Port_Range_Forwarding_Rollback

| Step | Task                                                     | Device | Command                                                                                                                                               | Expected Result                                               |
| ---: | -------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
|    1 | Identify Static PAT range rule                           | ASA    | `show nat detail`                                                                                                                                     | NAT rule, network objects, and service objects are identified |
|    2 | Remove Manual NAT range rule                             | ASA    | `no nat (<server-side-nameif>,outside) source static <inside-server-object> <public-ip-object> service <real-service-object> <mapped-service-object>` | Static PAT range rule is removed                              |
|    3 | Remove outside ACL binding if only used for this publish | ASA    | `no access-group <outside-acl> in interface outside`                                                                                                  | ACL is no longer applied                                      |
|    4 | Remove ACL if unused                                     | ASA    | `clear configure access-list <outside-acl>`                                                                                                           | ACL is removed                                                |
|    5 | Remove real service object if unused                     | ASA    | `no object service <real-service-object>`                                                                                                             | Real service range object is removed                          |
|    6 | Remove mapped service object if unused                   | ASA    | `no object service <mapped-service-object>`                                                                                                           | Mapped service range object is removed                        |
|    7 | Remove inside server object if unused                    | ASA    | `no object network <inside-server-object>`                                                                                                            | Server object is removed                                      |
|    8 | Remove public IP object if unused                        | ASA    | `no object network <public-ip-object>`                                                                                                                | Public IP object is removed                                   |
|    9 | Clear translations                                       | ASA    | `clear xlate`                                                                                                                                         | Old translations are cleared                                  |
|   10 | Clear stale connections                                  | ASA    | `clear conn address <inside-server-ip>`                                                                                                               | Old server connections are cleared                            |
|   11 | Verify NAT rule is gone                                  | ASA    | `show nat`                                                                                                                                            | Static PAT range rule no longer appears                       |
|   12 | Save rollback state                                      | ASA    | `write memory`                                                                                                                                        | Rollback is saved                                             |


# ASA_Static_PAT_Port_Range_Forwarding_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| NAT rule does not appear | `show nat` | Manual NAT syntax was wrong or object names do not exist |
| NAT rule has zero hits | `show nat detail` | Traffic is not hitting the public IP/port range, wrong protocol, or higher-priority NAT catches it |
| Packet-tracer does not UN-NAT to real server | `packet-tracer input outside <protocol> ... detailed` | Public IP object, service object, mapped range, or interface pair is wrong |
| ACL hit count stays zero | `show access-list <outside-acl>` | ACL is wrong, applied to wrong interface, wrong protocol, wrong range, or traffic never reaches ASA |
| ACL denies despite NAT working | `show access-list <outside-acl>` | ACL references public IP instead of real server IP |
| Only part of the range works | `show nat detail` and `show access-list <outside-acl>` | Real and mapped service ranges do not match or ACL range is narrower than NAT range |
| TCP range works but UDP range fails | `packet-tracer input outside udp ... detailed` | UDP service object or UDP ACL was not configured separately |
| UDP test appears inconsistent | `show conn` and application test | UDP is connectionless; application behavior and timeout may differ from TCP |
| Server never responds | `show conn detail` | Server service down, host firewall, wrong default gateway, or return path issue |
| Xlate exists but traffic fails | `show conn`, `show asp drop` | ACL, inspection, route, TCP state, or server-side issue |
| Public IP does not answer | Upstream route/ARP check and `show arp` | Upstream device cannot reach the mapped public IP through ASA |
| Dynamic PAT catches the flow instead | `show nat detail` | Static PAT range rule order/interface pair/object is wrong |
| Real traffic fails but packet-tracer allows | `show capture`, `show conn`, `show arp`, `show asp drop` | Upstream path, ARP, server host firewall, or return path issue |