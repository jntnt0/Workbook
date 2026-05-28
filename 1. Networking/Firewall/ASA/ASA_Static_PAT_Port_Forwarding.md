# ASA_Static_PAT_Port_Forwarding_Mental_Model

| Concept                 | Operational Meaning                                                                                    |
| ----------------------- | ------------------------------------------------------------------------------------------------------ |
| Static PAT              | One public IP and public port maps to one internal server IP and real service port                     |
| Port forwarding         | Outside users connect to `<public-ip>:<mapped-port>`, ASA forwards to `<inside-server-ip>:<real-port>` |
| Interface PAT           | `static interface` uses the ASA outside interface IP as the public IP                                  |
| Dedicated public IP PAT | `static <public-ip>` uses a separate mapped public IP                                                  |
| Service translation     | Public port and real server port can be the same or different                                          |
| Inbound ACL             | ASA 8.3+ ACLs normally reference the real/internal server IP, not the public mapped IP                 |
| NAT is not permission   | Static PAT creates translation, but ACL and routing still decide whether traffic is allowed            |
| Return path             | Inside server must route replies back through the ASA                                                  |
| Use case                | Publishing a single service like HTTP, HTTPS, SSH, or RDP to an internal/DMZ server                    |
# ASA_Static_PAT_Port_Forwarding_Configuration_Checklist

| Step | Task                                              | Device         | Command                                                                                             | Expected Result                                                                  |
| ---: | ------------------------------------------------- | -------------- | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
|    1 | Confirm ASA routed baseline is working            | ASA            | `show interface ip brief`                                                                           | Inside/DMZ and outside interfaces are up/up                                      |
|    2 | Confirm logical interface names                   | ASA            | `show nameif`                                                                                       | Server-side and outside interfaces have correct `nameif` values                  |
|    3 | Confirm route to internal server                  | ASA            | `show route <inside-server-ip>`                                                                     | ASA reaches the server through the expected inside/DMZ interface                 |
|    4 | Confirm outside/default route                     | ASA            | `show route 0.0.0.0`                                                                                | ASA has a route toward outside clients                                           |
|    5 | Confirm server default gateway path               | Server / ASA   | Server gateway check or `show conn` after test                                                      | Server replies return through ASA                                                |
|    6 | Review existing NAT rules                         | ASA            | `show nat`                                                                                          | Existing NAT rules and sections are visible                                      |
|    7 | Review NAT order and hit counts                   | ASA            | `show nat detail`                                                                                   | Existing NAT rules do not conflict with the port-forward rule                    |
|    8 | Confirm public IP choice                          | ASA / Notes    | `<outside-interface-ip>` or `<dedicated-public-ip>`                                                 | You know whether NAT uses `interface` or a dedicated public IP                   |
|    9 | Confirm real and mapped service ports             | ASA / Notes    | `<real-port>` and `<mapped-port>`                                                                   | You know whether ports are same or translated                                    |
|   10 | Enter global configuration mode                   | ASA            | `configure terminal`                                                                                | ASA enters configuration mode                                                    |
|   11 | Create server network object                      | ASA            | `object network <server-object>`                                                                    | ASA enters network object mode                                                   |
|   12 | Define internal server IP                         | ASA            | `host <inside-server-ip>`                                                                           | Object represents the real server                                                |
|   13 | Configure Static PAT using outside interface IP   | ASA            | `nat (<server-side-nameif>,outside) static interface service tcp <real-port> <mapped-port>`         | Public outside interface IP and mapped port forward to real server and real port |
|   14 | Or configure Static PAT using dedicated public IP | ASA            | `nat (<server-side-nameif>,outside) static <public-ip> service tcp <real-port> <mapped-port>`       | Dedicated public IP and mapped port forward to real server and real port         |
|   15 | Use UDP variant if publishing UDP service         | ASA            | `nat (<server-side-nameif>,outside) static interface service udp <real-port> <mapped-port>`         | UDP service is statically port-forwarded                                         |
|   16 | Exit object configuration mode                    | ASA            | `exit`                                                                                              | ASA returns to global configuration mode                                         |
|   17 | Create inbound ACL for published service          | ASA            | `access-list <outside-acl> extended permit tcp <source> host <inside-server-ip> eq <real-port>`     | Outside traffic to the real server/service is permitted                          |
|   18 | Apply inbound ACL to outside interface            | ASA            | `access-group <outside-acl> in interface outside`                                                   | Outside ACL is active inbound                                                    |
|   19 | Save configuration                                | ASA            | `write memory`                                                                                      | Static PAT configuration is saved                                                |
|   20 | Verify NAT rule order                             | ASA            | `show nat`                                                                                          | Static PAT rule appears in NAT table                                             |
|   21 | Verify NAT rule details                           | ASA            | `show nat detail`                                                                                   | Rule shows server object, static PAT, service translation, and hit count         |
|   22 | Verify ACL is applied                             | ASA            | `show access-group`                                                                                 | ACL is applied inbound on outside                                                |
|   23 | Verify ACL contents                               | ASA            | `show access-list <outside-acl>`                                                                    | ACL permits the real server IP and real service port                             |
|   24 | Simulate outside-to-inside flow                   | ASA            | `packet-tracer input outside tcp <outside-client-ip> <src-port> <public-ip> <mapped-port> detailed` | NAT phase untranslates destination to real server and final result is `ALLOW`    |
|   25 | Generate real outside traffic                     | Outside client | `curl http://<public-ip>:<mapped-port>` or `telnet <public-ip> <mapped-port>`                       | Client attempts connection to published service                                  |
|   26 | Verify xlate entry                                | ASA            | `show xlate`                                                                                        | Static PAT translation appears                                                   |
|   27 | Verify connection entry                           | ASA            | `show conn`                                                                                         | Connection appears from outside client to real server                            |
|   28 | Verify NATed connection detail                    | ASA            | `show conn long`                                                                                    | Real and mapped address/port details are visible                                 |
|   29 | Verify NAT hit count                              | ASA            | `show nat detail`                                                                                   | Static PAT rule hit count increments                                             |
|   30 | Verify ACL hit count                              | ASA            | `show access-list <outside-acl>`                                                                    | Permit ACE hit count increments                                                  |
|   31 | Check drops if test fails                         | ASA            | `show asp drop`                                                                                     | Drop reason points to ACL, NAT, route, inspection, TCP state, or adjacency issue |
# ASA_Static_PAT_Interface_HTTP_Skeleton

```text
configure terminal

object network <server-object>
 host <inside-server-ip>
 nat (<server-side-nameif>,outside) static interface service tcp www www

access-list <outside-acl> extended permit tcp <source> host <inside-server-ip> eq www
access-group <outside-acl> in interface outside

write memory
```

# ASA_Static_PAT_Interface_Custom_Port_Skeleton

```text
configure terminal

object network <server-object>
 host <inside-server-ip>
 nat (<server-side-nameif>,outside) static interface service tcp <real-port> <mapped-port>

access-list <outside-acl> extended permit tcp <source> host <inside-server-ip> eq <real-port>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Static_PAT_Dedicated_Public_IP_Skeleton

```text
configure terminal

object network <server-object>
 host <inside-server-ip>
 nat (<server-side-nameif>,outside) static <public-ip> service tcp <real-port> <mapped-port>

access-list <outside-acl> extended permit tcp <source> host <inside-server-ip> eq <real-port>
access-group <outside-acl> in interface outside

write memory
```


# ASA_Static_PAT_UDP_Skeleton

```text
configure terminal

object network <server-object>
 host <inside-server-ip>
 nat (<server-side-nameif>,outside) static interface service udp <real-port> <mapped-port>

access-list <outside-acl> extended permit udp <source> host <inside-server-ip> eq <real-port>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Static_PAT_Port_Forwarding_Verification_Commands

| Task                    | Command                                                                                             | Expected Result                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Verify interface state  | `show interface ip brief`                                                                           | Server-side and outside interfaces are up/up                      |
| Verify nameifs          | `show nameif`                                                                                       | Server-side and outside interface names are correct               |
| Verify route to server  | `show route <inside-server-ip>`                                                                     | ASA routes to server through expected interface                   |
| Verify outside route    | `show route 0.0.0.0`                                                                                | ASA has outside/default route                                     |
| Verify NAT table        | `show nat`                                                                                          | Static PAT rule appears in NAT table                              |
| Verify NAT details      | `show nat detail`                                                                                   | Static PAT service mapping and hit count are visible              |
| Verify ACL binding      | `show access-group`                                                                                 | Outside ACL is applied inbound                                    |
| Verify ACL hit count    | `show access-list <outside-acl>`                                                                    | Permit ACE hit count increases during test                        |
| Verify packet-tracer    | `packet-tracer input outside tcp <outside-client-ip> <src-port> <public-ip> <mapped-port> detailed` | Destination is untranslated to real server and result is expected |
| Verify xlate            | `show xlate`                                                                                        | Static PAT translation exists                                     |
| Verify connection       | `show conn`                                                                                         | Connection is built to real server                                |
| Verify NATed connection | `show conn long`                                                                                    | Real/mapped IP and port details are visible                       |
| Verify drops            | `show asp drop`                                                                                     | No relevant drop reason increments                                |
# ASA_Static_PAT_ACL_Address_Rule

| ASA Version Model | Inbound ACL Destination        | Example                                                                                 |
| ----------------- | ------------------------------ | --------------------------------------------------------------------------------------- |
| ASA 8.3 and later | Real/internal server IP        | `access-list OUTSIDE-IN extended permit tcp any host <inside-server-ip> eq <real-port>` |
| Pre-8.3 model     | Mapped/public IP               | Legacy only, not preferred for current labs                                             |
| Practical rule    | Use real IP in modern ASA labs | NAT translates, ACL permits the real destination                                        |

# ASA_Static_PAT_Port_Forwarding_Rollback

| Step | Task                                                     | Device | Command                                                                                          | Expected Result                                  |
| ---: | -------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
|    1 | Identify Static PAT rule                                 | ASA    | `show nat detail`                                                                                | Server object and service mapping are identified |
|    2 | Enter server object                                      | ASA    | `object network <server-object>`                                                                 | ASA enters object configuration mode             |
|    3 | Remove interface Static PAT                              | ASA    | `no nat (<server-side-nameif>,outside) static interface service tcp <real-port> <mapped-port>`   | Interface Static PAT rule is removed             |
|    4 | Or remove dedicated-IP Static PAT                        | ASA    | `no nat (<server-side-nameif>,outside) static <public-ip> service tcp <real-port> <mapped-port>` | Dedicated public IP Static PAT rule is removed   |
|    5 | Remove UDP Static PAT if used                            | ASA    | `no nat (<server-side-nameif>,outside) static interface service udp <real-port> <mapped-port>`   | UDP Static PAT rule is removed                   |
|    6 | Remove outside ACL binding if only used for this publish | ASA    | `no access-group <outside-acl> in interface outside`                                             | ACL is no longer applied                         |
|    7 | Remove ACL if unused                                     | ASA    | `clear configure access-list <outside-acl>`                                                      | ACL is removed                                   |
|    8 | Remove server object if unused                           | ASA    | `no object network <server-object>`                                                              | Server object is removed                         |
|    9 | Clear translations                                       | ASA    | `clear xlate`                                                                                    | Old translations are cleared                     |
|   10 | Clear stale connections                                  | ASA    | `clear conn address <inside-server-ip>`                                                          | Old server connections are cleared               |
|   11 | Verify NAT rule is gone                                  | ASA    | `show nat`                                                                                       | Static PAT rule no longer appears                |
|   12 | Save rollback state                                      | ASA    | `write memory`                                                                                   | Rollback is saved                                |

# ASA_Static_PAT_Port_Forwarding_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| Outside connection fails immediately | `packet-tracer input outside tcp <client-ip> <src-port> <public-ip> <mapped-port> detailed` | NAT, ACL, route, or inspection decision denies the flow |
| NAT rule does not appear | `show nat` | NAT was configured under wrong object or wrong interface pair |
| NAT rule has zero hits | `show nat detail` | Traffic is not hitting the public IP/port, wrong mapped port, or another NAT rule matches first |
| Packet-tracer does not UN-NAT to real server | `packet-tracer ... detailed` | Static PAT syntax, public IP, mapped port, or interface pair is wrong |
| ACL hit count stays zero | `show access-list <outside-acl>` | ACL is wrong, applied to wrong interface, or traffic never reaches ASA |
| ACL denies despite NAT working | `show access-list <outside-acl>` | ACL references public IP instead of real server IP in ASA 8.3+ |
| Connection builds but server does not respond | `show conn detail` | Server default gateway, host firewall, service down, or return path issue |
| Xlate exists but no connection works | `show conn` and `show asp drop` | ACL, TCP state, inspection, route, or server-side issue |
| Public interface IP PAT conflicts with ASA management/service | `show asp drop` and `show running-config access-group` | ASA itself may be listening on the same port or service conflict exists |
| Wrong internal service port used | `show conn long` | Real port and mapped port were reversed |
| TCP works from inside but not outside | `show access-group`, `show route <inside-server-ip>` | Missing inbound ACL or outside-to-inside route/policy issue |
| Dedicated public IP does not answer | Upstream route/ARP check and `show arp` | Upstream device cannot route/ARP to mapped public IP through ASA |
| Dynamic PAT catches the flow instead | `show nat detail` | Static PAT rule order/interface pair/object is wrong |
| Real traffic fails but packet-tracer allows | `show capture`, `show conn`, `show arp`, `show asp drop` | Upstream path, ARP, server host firewall, or return path issue |