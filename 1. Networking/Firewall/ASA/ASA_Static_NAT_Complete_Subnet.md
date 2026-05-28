# ASA_Static_NAT_Complete_Subnet_Mental_Model

| Concept                    | Operational Meaning                                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Static NAT complete subnet | One real subnet maps one-to-one to one translated subnet                                                    |
| One-to-one mapping         | Each inside IP maps predictably to a matching public/mapped IP                                              |
| Same subnet size           | Real subnet and mapped subnet should be the same size for clean one-to-one subnet translation               |
| Bidirectional translation  | Inside-to-outside and outside-to-inside flows can use the same static mapping if policy allows              |
| Auto NAT option            | Clean for simple real subnet to mapped subnet translation                                                   |
| Manual NAT option          | Better when NAT order, policy matching, or source/destination matching matters                              |
| Inbound ACL behavior       | On ASA 8.3+, ACLs usually reference the real/internal destination address, not the mapped public address    |
| Upstream reachability      | The outside network must know how to reach the translated subnet, either by proxy ARP or routing toward ASA |
| NAT is not permission      | Static NAT creates translation. ACLs and routing still control forwarding                                   |


# ASA_Static_NAT_Complete_Subnet_Configuration_Checklist

| Step | Task                                                         | Device                | Command                                                                                        | Expected Result                                                              |
| ---: | ------------------------------------------------------------ | --------------------- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
|    1 | Confirm routed ASA baseline is working                       | ASA                   | `show interface ip brief`                                                                      | Inside and outside interfaces are up/up                                      |
|    2 | Confirm logical interface names                              | ASA                   | `show nameif`                                                                                  | Interfaces are named correctly, usually `inside` and `outside`               |
|    3 | Confirm route to real inside subnet                          | ASA                   | `show route <real-inside-subnet-ip>`                                                           | ASA knows the real inside subnet                                             |
|    4 | Confirm outside/default route                                | ASA                   | `show route 0.0.0.0`                                                                           | ASA has an outside route for return/outbound traffic                         |
|    5 | Review existing NAT table                                    | ASA                   | `show nat`                                                                                     | Existing NAT rules and sections are visible                                  |
|    6 | Review detailed NAT order and hit counts                     | ASA                   | `show nat detail`                                                                              | Existing NAT order and hit counts are visible                                |
|    7 | Confirm real and mapped subnet sizes match                   | ASA / Notes           | `<real-subnet>/<prefix>` and `<mapped-subnet>/<prefix>`                                        | Real and mapped subnets are equal size                                       |
|    8 | Confirm no higher-priority NAT rule overlaps                 | ASA                   | `show nat detail`                                                                              | Section 1 or earlier Auto NAT rules do not catch this subnet unintentionally |
|    9 | Enter global configuration mode                              | ASA                   | `configure terminal`                                                                           | ASA enters configuration mode                                                |
|   10 | Create real inside subnet object                             | ASA                   | `object network <real-subnet-object>`                                                          | ASA enters network object mode                                               |
|   11 | Define real inside subnet                                    | ASA                   | `subnet <real-inside-network> <real-inside-mask>`                                              | Object represents the real/internal subnet                                   |
|   12 | Exit object mode                                             | ASA                   | `exit`                                                                                         | ASA returns to global configuration mode                                     |
|   13 | Create mapped subnet object                                  | ASA                   | `object network <mapped-subnet-object>`                                                        | ASA enters network object mode                                               |
|   14 | Define mapped/public subnet                                  | ASA                   | `subnet <mapped-network> <mapped-mask>`                                                        | Object represents the translated subnet                                      |
|   15 | Exit object mode                                             | ASA                   | `exit`                                                                                         | ASA returns to global configuration mode                                     |
|   16 | Re-enter real subnet object for Auto NAT                     | ASA                   | `object network <real-subnet-object>`                                                          | ASA enters real object config mode                                           |
|   17 | Configure static subnet NAT using Auto NAT                   | ASA                   | `nat (inside,outside) static <mapped-subnet-object>`                                           | Real subnet maps one-to-one to mapped subnet                                 |
|   18 | Or configure static subnet NAT using Manual NAT              | ASA                   | `nat (inside,outside) source static <real-subnet-object> <mapped-subnet-object>`               | Manual NAT rule maps real subnet to mapped subnet                            |
|   19 | Add `no-proxy-arp` only if routing handles the mapped subnet | ASA                   | `nat ... no-proxy-arp`                                                                         | ASA does not answer ARP for mapped addresses                                 |
|   20 | Add outside inbound ACL if outside hosts must initiate       | ASA                   | `access-list <outside-acl> extended permit <protocol> <source> <real-inside-subnet> <service>` | ACL permits traffic to the real/internal addresses                           |
|   21 | Apply outside ACL inbound                                    | ASA                   | `access-group <outside-acl> in interface outside`                                              | ACL is active inbound on outside                                             |
|   22 | Confirm upstream route or ARP behavior for mapped subnet     | Upstream router / ASA | `show route <mapped-subnet>` or upstream route check                                           | Outside network can reach the translated subnet                              |
|   23 | Save configuration                                           | ASA                   | `write memory`                                                                                 | Static subnet NAT configuration is saved                                     |
|   24 | Verify NAT table order                                       | ASA                   | `show nat`                                                                                     | Static subnet NAT appears in intended NAT section                            |
|   25 | Verify NAT rule details                                      | ASA                   | `show nat detail`                                                                              | Real object, mapped object, interface pair, and hit count are visible        |
|   26 | Simulate inside-to-outside flow                              | ASA                   | `packet-tracer input inside tcp <real-inside-host> <src-port> <outside-host> 80 detailed`      | NAT phase translates source to matching mapped IP                            |
|   27 | Simulate outside-to-inside flow                              | ASA                   | `packet-tracer input outside tcp <outside-host> <src-port> <mapped-host-ip> 80 detailed`       | ASA untranslates mapped IP to the correct real inside IP                     |
|   28 | Generate real inside-to-outside traffic                      | Inside host           | `ping <outside-host>` or application test                                                      | Traffic creates xlate/conn entries                                           |
|   29 | Generate real outside-to-inside traffic if allowed           | Outside host          | `ping <mapped-host-ip>` or service test                                                        | Traffic reaches the real inside host if ACL/routing permit it                |
|   30 | Verify active translations                                   | ASA                   | `show xlate`                                                                                   | Real hosts show static translations to mapped addresses                      |
|   31 | Verify connection table                                      | ASA                   | `show conn`                                                                                    | Expected connection entries appear                                           |
|   32 | Verify NAT hit count increments                              | ASA                   | `show nat detail`                                                                              | Static subnet NAT rule hit count increases                                   |
|   33 | Verify ACL hit count if inbound is tested                    | ASA                   | `show access-list <outside-acl>`                                                               | Expected ACL hit count increments                                            |
|   34 | Check drops if traffic fails                                 | ASA                   | `show asp drop`                                                                                | Drop reason points to ACL, route, NAT, inspection, or adjacency issue        |
# ASA_Static_NAT_Complete_Subnet_Auto_NAT_Skeleton

```text
configure terminal

object network <real-subnet-object>
 subnet <real-inside-network> <real-inside-mask>

object network <mapped-subnet-object>
 subnet <mapped-network> <mapped-mask>

object network <real-subnet-object>
 nat (inside,outside) static <mapped-subnet-object>

write memory
```

# ASA_Static_NAT_Complete_Subnet_Manual_NAT_Skeleton

```text
configure terminal

object network <real-subnet-object>
 subnet <real-inside-network> <real-inside-mask>

object network <mapped-subnet-object>
 subnet <mapped-network> <mapped-mask>

nat (inside,outside) source static <real-subnet-object> <mapped-subnet-object>

write memory
```

# ASA_Static_NAT_Complete_Subnet_Inbound_ACL_Skeleton

```text
configure terminal

access-list <outside-acl> extended permit <protocol> <source-network-or-any> <real-inside-subnet> <service>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Static_NAT_Complete_Subnet_Verification_Commands

| Task                                     | Command                                                                                   | Expected Result                                                  |
| ---------------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Verify interface state                   | `show interface ip brief`                                                                 | Inside and outside interfaces are up/up                          |
| Verify interface names                   | `show nameif`                                                                             | `inside` and `outside` are assigned correctly                    |
| Verify route to real subnet              | `show route <real-inside-host>`                                                           | ASA can reach the real inside host/subnet                        |
| Verify outside/default route             | `show route 0.0.0.0`                                                                      | ASA has outside route for outbound/return traffic                |
| Verify NAT rule order                    | `show nat`                                                                                | Static subnet NAT appears in intended section                    |
| Verify NAT rule details                  | `show nat detail`                                                                         | Real/mapped objects and hit counts are visible                   |
| Verify inside-to-outside NAT decision    | `packet-tracer input inside tcp <real-inside-host> <src-port> <outside-host> 80 detailed` | Source is translated to mapped subnet address                    |
| Verify outside-to-inside UN-NAT decision | `packet-tracer input outside tcp <outside-host> <src-port> <mapped-host-ip> 80 detailed`  | Destination is untranslated to real inside host                  |
| Verify translations                      | `show xlate`                                                                              | Real hosts map to translated subnet addresses                    |
| Verify local-host state                  | `show local-host <real-inside-host>`                                                      | Xlate and connection details are visible                         |
| Verify connection table                  | `show conn`                                                                               | Expected flows create connections                                |
| Verify NATed connection detail           | `show conn long`                                                                          | Real and translated addresses are visible                        |
| Verify inbound ACL                       | `show access-group`                                                                       | Outside ACL is applied inbound if outside initiation is required |
| Verify ACL hit counts                    | `show access-list <outside-acl>`                                                          | Permit ACE hit count increments during inbound test              |
| Verify drops                             | `show asp drop`                                                                           | No relevant drop reason increments during intended traffic       |
# ASA_Static_NAT_Complete_Subnet_ACL_Address_Rule

| Direction                     | ACL Destination To Use                         | Reason                                                                     |
| ----------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------- |
| Outside-to-inside on ASA 8.3+ | Real/internal IP or subnet                     | ASA ACLs reference real addresses after the NAT model change               |
| Inside-to-outside             | Usually no ACL needed unless inside ACL exists | Higher-to-lower traffic is statefully allowed by default unless restricted |
| DMZ-to-inside                 | Real/internal destination                      | NAT does not replace ACL permission                                        |
| Published service access      | Real server IP and real service                | Static NAT exposes translation, ACL permits the real service               |

# ASA_Static_NAT_Complete_Subnet_Rollback

| Step | Task                                           | Device | Command                                                                                            | Expected Result                                |
| ---: | ---------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
|    1 | Identify static subnet NAT rule                | ASA    | `show nat detail`                                                                                  | Exact NAT rule and object names are identified |
|    2 | Remove Auto NAT rule from real object          | ASA    | `object network <real-subnet-object>` then `no nat (inside,outside) static <mapped-subnet-object>` | Auto NAT static subnet rule is removed         |
|    3 | Or remove Manual NAT rule                      | ASA    | `no nat (inside,outside) source static <real-subnet-object> <mapped-subnet-object>`                | Manual NAT static subnet rule is removed       |
|    4 | Remove outside ACL if it was only for this NAT | ASA    | `no access-group <outside-acl> in interface outside`                                               | Outside ACL is no longer applied               |
|    5 | Remove ACL object/rules if unused              | ASA    | `clear configure access-list <outside-acl>`                                                        | ACL is removed                                 |
|    6 | Remove mapped subnet object if unused          | ASA    | `no object network <mapped-subnet-object>`                                                         | Mapped object is removed                       |
|    7 | Remove real subnet object if unused            | ASA    | `no object network <real-subnet-object>`                                                           | Real object is removed                         |
|    8 | Clear translations                             | ASA    | `clear xlate`                                                                                      | Old translations are cleared                   |
|    9 | Clear stale connections if needed              | ASA    | `clear conn address <real-inside-host>`                                                            | Old connection state is cleared                |
|   10 | Verify NAT rule is gone                        | ASA    | `show nat`                                                                                         | Static subnet NAT rule no longer appears       |
|   11 | Save rollback state                            | ASA    | `write memory`                                                                                     | Rollback is saved                              |

# ASA_Static_NAT_Complete_Subnet_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| NAT rule does not appear | `show nat` | NAT was configured under wrong object, wrong syntax, or not saved/applied |
| NAT rule has zero hits | `show nat detail` | Traffic does not match real subnet, wrong interface pair, or higher-priority NAT rule catches it |
| Inside host translates to wrong address | `show xlate` | Mapped subnet object is wrong or overlapping NAT rule takes priority |
| Outside-to-inside traffic fails | `packet-tracer input outside ... detailed` | ACL missing, UN-NAT wrong, route missing, or real host unreachable |
| Packet-tracer allows but real traffic fails | `show arp`, `show conn`, `show asp drop` | ARP, upstream routing, downstream host, or return path issue |
| Outside host cannot reach mapped subnet | Upstream route check and `show arp` | Upstream device has no route to mapped subnet or proxy ARP is disabled unexpectedly |
| ACL hit count stays zero | `show access-list <outside-acl>` | ACL uses mapped IP instead of real IP, wrong interface, wrong protocol, or traffic never reaches ASA |
| Static NAT works outbound but not inbound | `show access-group` and `show route <real-inside-host>` | NAT exists but inbound ACL or route to real host is missing |
| Some hosts work and others do not | `show xlate` and `show route <real-inside-host>` | Real subnet/mapped subnet size mismatch or missing route to part of real subnet |
| Mapped subnet overlaps another NAT rule | `show nat detail` | NAT object overlap or rule-order conflict |
| Return traffic drops | `show asp drop` and `show conn detail` | TCP state, asymmetric routing, ACL, or inspection issue |
| Dynamic PAT catches traffic instead | `show nat detail` | Static subnet NAT is in wrong section or broad PAT rule is matched first |