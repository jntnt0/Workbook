# ASA_NAT_Order_And_Object_Model_Mental_Model

| Concept           | Operational Meaning                                                                                              |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| Unified NAT table | ASA 8.3+ evaluates NAT rules from a unified table, top down, first match wins                                    |
| Section 1         | Manual NAT before Auto NAT; processed in configured order                                                        |
| Section 2         | Auto NAT / Object NAT; configured inside network objects; ASA orders rules by specificity                        |
| Section 3         | Manual NAT after Auto NAT; configured with `after-auto`; processed in configured order                           |
| Auto NAT          | Best for simple source NAT, dynamic PAT, static NAT, and static PAT tied to one object                           |
| Manual NAT        | Best for policy NAT, identity NAT, VPN NAT exemption, overlapping networks, and source plus destination matching |
| Twice NAT         | Manual NAT that can match and translate source and destination in one rule                                       |
| Identity NAT      | NAT rule where real and mapped addresses are the same; usually used to exempt VPN traffic from PAT               |
| NAT hit count     | `show nat detail` hit count increments when a connection is built using the NAT rule, not for every packet       |
| Xlate table       | `show xlate` proves the active translation that ASA actually built                                               |
# ASA_NAT_Section_Order

| Section | NAT Type | Example Use | Processing Rule |
|---:|---|---|---|
| 1 | Manual NAT before-auto | VPN identity NAT, policy NAT, overlapping subnet twice NAT | Processed first, top down, in configured order |
| 2 | Auto NAT / Object NAT | Dynamic PAT to interface, static NAT for server, simple static PAT | ASA orders automatically by specificity |
| 3 | Manual NAT after-auto | Fallback policy NAT, lower-priority manual rules | Processed after Auto NAT, top down |

| Rule                                   | Operational Meaning                                                 |
| -------------------------------------- | ------------------------------------------------------------------- |
| First match wins                       | Once ASA matches a NAT rule, it stops checking later NAT rules      |
| Specific exemptions go high            | VPN identity NAT and policy NAT usually belong in Section 1         |
| Simple source PAT can live in Auto NAT | Inside-to-outside Internet PAT is usually clean as Auto NAT         |
| After-auto is lower priority           | Use Section 3 only when the rule should be checked after object NAT |
| Verify order, do not assume it         | Use `show nat` and `show nat detail` to see actual section/order    |
# ASA_NAT_Object_Model

| Object Type    | Example Syntax                                                      | Used For                                         |
| -------------- | ------------------------------------------------------------------- | ------------------------------------------------ |
| Host object    | `object network <name>` then `host <ip>`                            | Single server, static NAT, static PAT            |
| Subnet object  | `object network <name>` then `subnet <network> <mask>`              | Inside LAN, VPN local network, remote network    |
| Range object   | `object network <name>` then `range <start-ip> <end-ip>`            | Pool-style translation object                    |
| Mapped object  | `object network <name>` then `host/subnet/range <translated-space>` | Public IP, translated subnet, NAT pool           |
| Service object | `object service <name>`                                             | Static PAT or service-specific rules when needed |

# ASA_NAT_Order_And_Object_Model_Configuration_Checklist

| Step | Task                                                          | Device        | Command                                                                                                                             | Expected Result                                                       |
| ---: | ------------------------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
|    1 | Confirm ASA version and platform                              | ASA           | `show version`                                                                                                                      | ASA version is visible; modern NAT model applies to ASA 8.3+          |
|    2 | Verify routed baseline first                                  | ASA           | `show interface ip brief`                                                                                                           | Inside, outside, and optional DMZ interfaces are up/up                |
|    3 | Verify interface names                                        | ASA           | `show nameif`                                                                                                                       | NAT interface names such as `inside` and `outside` are correct        |
|    4 | Verify routing before NAT                                     | ASA           | `show route`                                                                                                                        | ASA has connected/static/default routes needed for the flow           |
|    5 | Review existing NAT table                                     | ASA           | `show nat`                                                                                                                          | NAT rules are visible by section                                      |
|    6 | Review detailed NAT table                                     | ASA           | `show nat detail`                                                                                                                   | NAT sections, order, object names, and hit counts are visible         |
|    7 | Identify the traffic flow                                     | ASA / Notes   | `<src-interface>, <dst-interface>, <src-ip>, <dst-ip>, <service>`                                                                   | You know what traffic must match NAT                                  |
|    8 | Decide whether the rule is simple source NAT                  | ASA / Notes   | Auto NAT decision                                                                                                                   | Use Auto NAT if the rule only depends mainly on source object         |
|    9 | Decide whether the rule needs source and destination matching | ASA / Notes   | Manual NAT decision                                                                                                                 | Use Manual NAT if the rule depends on both source and destination     |
|   10 | Decide whether NAT exemption is needed                        | ASA / Notes   | Identity NAT decision                                                                                                               | VPN or special traffic keeps original addresses                       |
|   11 | Decide correct NAT section                                    | ASA / Notes   | Section 1, 2, or 3                                                                                                                  | Specific/manual rules are ordered before broad PAT rules              |
|   12 | Enter global configuration mode                               | ASA           | `configure terminal`                                                                                                                | ASA enters configuration mode                                         |
|   13 | Create source network object                                  | ASA           | `object network <source-object>` then `subnet <source-network> <mask>`                                                              | Source object exists                                                  |
|   14 | Create host object if translating a server                    | ASA           | `object network <server-object>` then `host <server-ip>`                                                                            | Server object exists                                                  |
|   15 | Create mapped object if using translated address/subnet       | ASA           | `object network <mapped-object>` then `host/subnet/range <mapped-address>`                                                          | Translated object exists                                              |
|   16 | Configure simple dynamic PAT with Auto NAT                    | ASA           | `object network <source-object>` then `nat (inside,outside) dynamic interface`                                                      | Inside source traffic PATs to outside interface                       |
|   17 | Configure simple static NAT with Auto NAT                     | ASA           | `object network <server-object>` then `nat (inside,outside) static <mapped-ip-or-object>`                                           | Server has static one-to-one translation                              |
|   18 | Configure Manual NAT for policy NAT                           | ASA           | `nat (inside,outside) source static <real-src-object> <mapped-src-object> destination static <real-dst-object> <mapped-dst-object>` | NAT applies only when both source and destination match               |
|   19 | Configure identity NAT for NAT exemption                      | ASA           | `nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object>`               | Matching traffic is explicitly not translated                         |
|   20 | Configure after-auto Manual NAT if lower priority is intended | ASA           | `nat (inside,outside) after-auto source dynamic <source-object> interface`                                                          | Rule appears in Section 3                                             |
|   21 | Add `no-proxy-arp` if appropriate                             | ASA           | `nat ... no-proxy-arp`                                                                                                              | ASA does not proxy ARP for that NAT rule                              |
|   22 | Add `route-lookup` if required by design                      | ASA           | `nat ... route-lookup`                                                                                                              | ASA uses route lookup behavior where appropriate for the NAT rule     |
|   23 | Save configuration                                            | ASA           | `write memory`                                                                                                                      | NAT configuration is saved                                            |
|   24 | Verify NAT table order                                        | ASA           | `show nat`                                                                                                                          | Rules appear in intended Section 1, 2, or 3                           |
|   25 | Verify detailed NAT rule output                               | ASA           | `show nat detail`                                                                                                                   | Rule order, translated objects, and hit counts are visible            |
|   26 | Simulate the exact flow                                       | ASA           | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port> detailed`                                  | NAT phase matches the intended rule                                   |
|   27 | Generate real traffic                                         | Host / Router | `ping`, `telnet`, `curl`, or application test                                                                                       | Traffic creates a connection and translation if NAT applies           |
|   28 | Verify active translation                                     | ASA           | `show xlate`                                                                                                                        | Expected real-to-mapped translation appears                           |
|   29 | Verify local host state                                       | ASA           | `show local-host <source-ip>`                                                                                                       | Xlate and connection details show expected behavior                   |
|   30 | Verify connection table                                       | ASA           | `show conn` or `show conn long`                                                                                                     | Connection exists and NATed addresses are visible when applicable     |
|   31 | Verify NAT hit count                                          | ASA           | `show nat detail`                                                                                                                   | Matching NAT rule hit count increments after connection build         |
|   32 | Check drops if NAT behavior fails                             | ASA           | `show asp drop`                                                                                                                     | Drop reason points to route, NAT, ACL, inspection, or adjacency issue |

# ASA_Auto_NAT_Dynamic_PAT_Skeleton

```text
configure terminal

object network <inside-network-object>
 subnet <inside-network> <inside-mask>
 nat (inside,outside) dynamic interface

write memory
```

# ASA_Auto_NAT_Static_NAT_Skeleton

```text
configure terminal

object network <inside-server-object>
 host <inside-server-ip>
 nat (inside,outside) static <public-ip-or-public-object>

write memory
```

# ASA_Manual_NAT_Policy_NAT_Skeleton

```text
configure terminal

object network <real-source-object>
 subnet <real-source-network> <real-source-mask>

object network <mapped-source-object>
 subnet <mapped-source-network> <mapped-source-mask>

object network <real-destination-object>
 subnet <real-destination-network> <real-destination-mask>

object network <mapped-destination-object>
 subnet <mapped-destination-network> <mapped-destination-mask>

nat (inside,outside) source static <real-source-object> <mapped-source-object> destination static <real-destination-object> <mapped-destination-object>

write memory
```


# ASA_Identity_NAT_Exemption_Skeleton

```text
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp

write memory
```

# ASA_After_Auto_Manual_NAT_Skeleton

```text
configure terminal

object network <source-object>
 subnet <source-network> <source-mask>

nat (inside,outside) after-auto source dynamic <source-object> interface

write memory
```

# ASA_NAT_Order_And_Object_Model_Verification_Commands

| Task                                  | Command                                                                                            | Expected Result                                                    |
| ------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Verify NAT table order                | `show nat`                                                                                         | NAT rules appear in Section 1, Section 2, or Section 3 as intended |
| Verify detailed NAT rules             | `show nat detail`                                                                                  | Rule order, real/mapped objects, and hit counts are visible        |
| Verify active translations            | `show xlate`                                                                                       | Expected xlate entries appear after real traffic                   |
| Verify host xlate and conn state      | `show local-host <ip-address>`                                                                     | Local-host output shows xlate and connection state                 |
| Verify connection table               | `show conn`                                                                                        | Connection exists after traffic                                    |
| Verify NATed connection detail        | `show conn long`                                                                                   | NATed addresses are visible in connection output                   |
| Verify exact packet path              | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port> detailed` | NAT phase matches the intended NAT rule                            |
| Verify routing after NAT decision     | `show route <destination-ip>`                                                                      | ASA has expected route to destination or next hop                  |
| Verify ACL uses correct address model | `show access-list <acl-name>`                                                                      | ACL matches real address expectations for ASA 8.3+ behavior        |
| Verify drops                          | `show asp drop`                                                                                    | Drop counters do not increase for intended working traffic         |
# ASA_NAT_Rule_Selection_Table

| Requirement                                            | Best NAT Type                     | Typical Section        | Example Command Pattern                                                           |
| ------------------------------------------------------ | --------------------------------- | ---------------------- | --------------------------------------------------------------------------------- |
| Inside users need Internet access using ASA outside IP | Auto NAT dynamic PAT              | Section 2              | `nat (inside,outside) dynamic interface`                                          |
| Inside server needs one public IP                      | Auto NAT static NAT               | Section 2              | `nat (inside,outside) static <public-ip>`                                         |
| Inside server needs one public port forwarded          | Auto NAT static PAT or Manual NAT | Section 2 or Section 1 | `nat (inside,outside) static interface service tcp <real-port> <mapped-port>`     |
| VPN traffic must not be PATed                          | Manual identity NAT               | Section 1              | `nat (inside,outside) source static LOCAL LOCAL destination static REMOTE REMOTE` |
| NAT depends on both source and destination             | Manual NAT / twice NAT            | Section 1              | `nat (inside,outside) source static SRC MAPPED destination static DST DST`        |
| Overlapping networks need translation                  | Manual NAT / twice NAT            | Section 1              | `nat (...) source static REAL MAPPED destination static REAL-DST MAPPED-DST`      |
| Lower-priority fallback NAT                            | Manual NAT after-auto             | Section 3              | `nat (...) after-auto source dynamic OBJ interface`                               |

# ASA_NAT_Order_And_Object_Model_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify NAT rule to remove | ASA | `show nat detail` | Exact NAT section and rule are identified |
| 2 | Remove Auto NAT from object | ASA | `object network <object-name>` then `no nat (inside,outside) dynamic interface` | Auto NAT rule is removed from object |
| 3 | Remove static Auto NAT from object | ASA | `object network <object-name>` then `no nat (inside,outside) static <mapped-ip-or-object>` | Static object NAT is removed |
| 4 | Remove Manual NAT rule | ASA | `no nat (inside,outside) source static <real-src-object> <mapped-src-object> destination static <real-dst-object> <mapped-dst-object>` | Manual NAT rule is removed |
| 5 | Remove after-auto Manual NAT rule | ASA | `no nat (inside,outside) after-auto source dynamic <source-object> interface` | Section 3 NAT rule is removed |
| 6 | Remove unused object | ASA | `no object network <object-name>` | Object is removed if no longer referenced |
| 7 | Clear translations after NAT change | ASA | `clear xlate` | Old translations are cleared |
| 8 | Clear stale connections if needed | ASA | `clear conn address <ip-address>` | Old connection state is cleared |
| 9 | Verify NAT table after rollback | ASA | `show nat` | Removed NAT rule is gone |
| 10 | Save rollback state | ASA | `write memory` | Rollback is saved |

# ASA_NAT_Order_And_Object_Model_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| Wrong NAT rule matches | `show nat detail` | Rule order problem; Section 1 rule, Auto NAT specificity, or Section 3 placement is wrong |
| VPN traffic gets PATed | `packet-tracer ... detailed` and `show nat detail` | Identity NAT exemption is missing or below a broader PAT rule |
| NAT rule never gets hits | `show nat detail` | Source/destination object does not match the real flow, wrong interface pair, or traffic never reaches ASA |
| Xlate is missing | `show xlate` | NAT rule did not match, no traffic generated, or connection failed before xlate creation |
| Xlate is wrong | `show xlate` and `show nat detail` | Wrong object, wrong mapped address, or higher-priority NAT rule matched first |
| Packet-tracer shows unexpected UN-NAT | `packet-tracer ... detailed` | Static NAT or destination NAT is changing the destination before route/ACL logic |
| Static NAT works but inbound traffic fails | `show access-group` and `show access-list` | ACL missing or applied to wrong interface/direction |
| NAT works but return traffic fails | `show route <source-ip>` | Missing return route, downstream route issue, or asymmetric path |
| ACL hit count stays zero | `show access-list <acl-name>` | ACL uses wrong real/mapped address expectation or traffic is not reaching that interface |
| After-auto rule never matches | `show nat` | Auto NAT or Section 1 rule catches the flow first |
| Identity NAT exists but not used | `show nat detail` | Destination object mismatch, wrong interface pair, or Section 1 ordering issue |
| Overlapping subnet NAT behaves wrong | `packet-tracer ... detailed` | Twice NAT source/destination object mapping is reversed or incomplete |
| Real traffic fails but packet-tracer looks correct | `show conn`, `show xlate`, `show asp drop` | Stale connection/xlate, ARP, route, or downstream device problem |
