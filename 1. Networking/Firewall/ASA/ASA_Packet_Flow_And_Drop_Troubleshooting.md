# ASA_Packet_Flow_And_Drop_Troubleshooting_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Packet-tracer | Simulates ASA decision phases for a specific flow |
| Live capture with trace | Shows how a real packet is handled by the firewall |
| ASP drop | Shows software-level drop counters and drop reasons |
| Connection table | Proves whether ASA built state for the flow |
| Xlate table | Proves whether NAT translation exists and what real/mapped addresses are being used |
| Route lookup | Determines expected egress interface unless overridden by NAT/PBR/current connection |
| ARP adjacency | Even if packet-tracer shows route lookup success, ASA can still drop if next-hop ARP is unresolved |
| ACL hit count | Proves whether the expected access rule is actually matching |
| Drop reason | The first serious clue. Do not guess before reading the drop reason |
# ASA_Packet_Flow_Decision_Order

| Order | Decision Point                            | What To Check                                                         |
| ----: | ----------------------------------------- | --------------------------------------------------------------------- |
|     1 | Existing connection lookup                | `show conn`, `show conn long`, `show conn detail`                     |
|     2 | NAT lookup, especially destination UN-NAT | `packet-tracer`, `show xlate`, `show nat detail`                      |
|     3 | Policy-Based Routing if configured        | `show route-map`, `show running-config route-map`                     |
|     4 | Routing table lookup                      | `show route`, `show route <destination-ip>`                           |
|     5 | ACL/security policy decision              | `show access-group`, `show access-list`, packet-tracer phases         |
|     6 | Inspection/state behavior                 | `show service-policy`, `show conn detail`, packet-tracer phases       |
|     7 | Egress adjacency                          | `show arp`, `show arp statistics`, packet-tracer no-v4-adjacency drop |
|     8 | ASP software drop                         | `show asp drop`, `capture <name> type asp-drop <drop-reason>`         |
# ASA_Packet_Flow_And_Drop_Troubleshooting_Checklist

| Step | Task                                                                | Device        | Command                                                                                            | Expected Result                                                                               |
| ---: | ------------------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
|    1 | Define the exact test flow                                          | ASA / Notes   | `<src-ip>, <src-port>, <dst-ip>, <dst-port>, <protocol>, <ingress-interface>`                      | You have a clear 5-tuple before troubleshooting                                               |
|    2 | Confirm interface state                                             | ASA           | `show interface ip brief`                                                                          | Ingress and expected egress interfaces are up/up                                              |
|    3 | Confirm logical interface names                                     | ASA           | `show nameif`                                                                                      | Ingress and egress `nameif` values are known                                                  |
|    4 | Confirm route to destination                                        | ASA           | `show route <dst-ip>`                                                                              | ASA selects the expected egress interface and next hop                                        |
|    5 | Confirm route back to source                                        | ASA           | `show route <src-ip>`                                                                              | ASA has a return path for the source network                                                  |
|    6 | Run packet-tracer for the exact flow                                | ASA           | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port>`          | Result is either `ALLOW` or gives a specific drop phase/reason                                |
|    7 | Re-run packet-tracer with detailed option if needed                 | ASA           | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port> detailed` | Full phase output shows NAT, route, ACL, inspect, and adjacency decisions                     |
|    8 | Check applied ACLs                                                  | ASA           | `show access-group`                                                                                | ACL is applied to the expected interface and direction                                        |
|    9 | Check ACL hit counts                                                | ASA           | `show access-list <acl-name>`                                                                      | Expected ACE hit count increments during test traffic                                         |
|   10 | Check NAT rules                                                     | ASA           | `show nat detail`                                                                                  | Expected NAT rule exists and is ordered correctly                                             |
|   11 | Check translation table                                             | ASA           | `show xlate`                                                                                       | Expected real-to-mapped translation exists after traffic                                      |
|   12 | Check connection table                                              | ASA           | `show conn`                                                                                        | Expected connection appears after real traffic                                                |
|   13 | Check detailed connection state                                     | ASA           | `show conn detail`                                                                                 | Initiator/responder and interface direction are visible                                       |
|   14 | Check NATed connection view                                         | ASA           | `show conn long`                                                                                   | NATed IPs appear in parentheses when NAT is involved                                          |
|   15 | Clear stale connection if egress behavior looks wrong               | ASA           | `clear conn address <ip-address>`                                                                  | Old connection state is removed                                                               |
|   16 | Clear ASP drop counters before a focused test                       | ASA           | `clear asp drop`                                                                                   | Drop counters reset                                                                           |
|   17 | Generate one controlled test flow                                   | Host / Router | `ping <dst-ip>` or `telnet <dst-ip> <port>`                                                        | Test traffic is sent while counters are clean                                                 |
|   18 | Check ASP drop counters after the test                              | ASA           | `show asp drop`                                                                                    | Increased counter identifies drop reason such as `acl-drop`, `no-route`, or `no-v4-adjacency` |
|   19 | Capture a specific ASP drop reason                                  | ASA           | `capture <capture-name> type asp-drop <drop-reason>`                                               | ASA captures packets matching that drop reason                                                |
|   20 | Review ASP drop capture                                             | ASA           | `show capture <capture-name>`                                                                      | Dropped packet appears in capture output                                                      |
|   21 | Check ARP cache                                                     | ASA           | `show arp`                                                                                         | ASA has ARP entry for the next hop                                                            |
|   22 | Check ARP statistics                                                | ASA           | `show arp statistics`                                                                              | Unresolved hosts or dropped ARP blocks indicate adjacency trouble                             |
|   23 | Capture ARP if next-hop resolution is suspect                       | ASA           | `capture <capture-name> ethernet-type arp interface <egress-nameif>`                               | ARP request/reply behavior is visible                                                         |
|   24 | Check interface counters                                            | ASA           | `show interface <interface-name>`                                                                  | No obvious link errors, drops, or state problems                                              |
|   25 | Check logs for deny messages                                        | ASA           | `show logging`                                                                                     | Deny logs or policy messages line up with failed flow                                         |
|   26 | Check service policy/inspection if protocol-specific failure exists | ASA           | `show service-policy`                                                                              | Inspection policy counters or drops explain protocol behavior                                 |
|   27 | Confirm packet leaves expected egress interface with capture        | ASA           | `capture <cap-name> interface <egress-nameif> match <protocol> host <src-ip> host <dst-ip>`        | Packet appears on expected egress interface                                                   |
|   28 | Confirm packet arrives on ingress interface with capture            | ASA           | `capture <cap-name> interface <ingress-nameif> match <protocol> host <src-ip> host <dst-ip>`       | Packet is actually reaching ASA                                                               |
|   29 | Remove captures after testing                                       | ASA           | `no capture <capture-name>`                                                                        | Capture is removed                                                                            |
|   30 | Final confirm with packet-tracer and real traffic                   | ASA / Host    | `packet-tracer ...` and `ping/telnet/curl`                                                         | Simulated and real traffic both show expected behavior                                        |
# ASA_Packet_Tracer_Skeleton

```text
packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port>

packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port> detailed
```

# ASA_Packet_Flow_Capture_Skeleton

```text
capture <ingress-cap-name> interface <ingress-nameif> match <protocol> host <src-ip> host <dst-ip>
capture <egress-cap-name> interface <egress-nameif> match <protocol> host <src-ip> host <dst-ip>

show capture <ingress-cap-name>
show capture <egress-cap-name>

no capture <ingress-cap-name>
no capture <egress-cap-name>
```

# ASA_ASP_Drop_Capture_Skeleton

```text
clear asp drop

show asp drop

capture <capture-name> type asp-drop <drop-reason>

show capture <capture-name>

no capture <capture-name>
```

# ASA_ARP_Troubleshooting_Skeleton

```text
show arp
show arp statistics

capture <arp-cap-name> ethernet-type arp interface <egress-nameif>

show capture <arp-cap-name>

no capture <arp-cap-name>
```

# ASA_Packet_Flow_And_Drop_Verification_Commands

| Task                              | Command                                                                                   | Expected Result                                              |
| --------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Verify interfaces                 | `show interface ip brief`                                                                 | Required interfaces are up/up                                |
| Verify nameifs                    | `show nameif`                                                                             | Interface logical names are correct                          |
| Verify destination route          | `show route <dst-ip>`                                                                     | ASA selects expected egress interface                        |
| Verify source return route        | `show route <src-ip>`                                                                     | ASA has a path back to the source                            |
| Simulate traffic                  | `packet-tracer input <ingress-nameif> <protocol> <src-ip> <src-port> <dst-ip> <dst-port>` | Result shows `ALLOW` or exact drop phase                     |
| Verify ACL binding                | `show access-group`                                                                       | ACL is applied to intended interface and direction           |
| Verify ACL hit count              | `show access-list <acl-name>`                                                             | Expected ACE hit count increases                             |
| Verify NAT rule order             | `show nat detail`                                                                         | Expected NAT rule exists and is hit before conflicting rules |
| Verify translation                | `show xlate`                                                                              | Expected translation exists                                  |
| Verify session state              | `show conn`                                                                               | Flow creates a connection                                    |
| Verify detailed session state     | `show conn detail`                                                                        | Initiator, responder, and interface direction are clear      |
| Verify NATed session view         | `show conn long`                                                                          | NATed IPs are shown in parentheses when applicable           |
| Verify drop counters              | `show asp drop`                                                                           | Drop counter identifies likely failure reason                |
| Verify next-hop adjacency         | `show arp`                                                                                | Egress next-hop has an ARP entry                             |
| Verify unresolved ARP             | `show arp statistics`                                                                     | No unresolved hosts for the expected next hop                |
| Verify packet entered ASA         | `show capture <ingress-cap-name>`                                                         | Packet appears on ingress capture                            |
| Verify packet left ASA            | `show capture <egress-cap-name>`                                                          | Packet appears on expected egress capture                    |
| Verify policy/inspection counters | `show service-policy`                                                                     | Inspection counters align with traffic behavior              |
# ASA_Packet_Flow_Common_Drop_Reasons

| Drop Reason            | Command To Confirm                                            | Usual Meaning                                                                |
| ---------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `acl-drop`             | `packet-tracer ...` and `show access-list`                    | Access policy denied the flow                                                |
| `no-route`             | `show route <dst-ip>`                                         | ASA has no route to the destination                                          |
| `no-v4-adjacency`      | `show arp` and `show arp statistics`                          | ASA cannot resolve the next-hop adjacency                                    |
| `interface-down`       | `show interface ip brief`                                     | Egress or ingress interface is down                                          |
| TCP state drop         | `show conn detail` and `show asp drop`                        | Packet does not match expected TCP state                                     |
| NAT mismatch           | `packet-tracer ... detailed`, `show nat detail`, `show xlate` | Wrong NAT rule, wrong order, or missing translation                          |
| Wrong egress interface | `packet-tracer ... detailed`, `show route`, `show conn`       | Existing connection, UN-NAT, PBR, or route lookup selected unexpected egress |
| Inspection drop        | `show service-policy`                                         | Protocol inspection policy is affecting the flow                             |
# ASA_Packet_Flow_And_Drop_Failure_Checks

| Symptom                                     | Command                                               | What Usually Broke                                                               |
| ------------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------- |
| Packet never reaches ASA                    | `show capture <ingress-cap-name>`                     | Upstream host, upstream route, switch/VLAN, or wrong ASA interface               |
| Packet reaches ASA but never exits          | `packet-tracer ... detailed`                          | ACL, NAT, route, inspection, or adjacency failure                                |
| Packet-tracer says ACL drop                 | `show access-group` and `show access-list <acl-name>` | ACL missing, wrong direction, wrong object, wrong service, or rule order problem |
| Packet-tracer says no route                 | `show route <dst-ip>`                                 | Missing static/default route or wrong next hop                                   |
| Packet-tracer says no valid adjacency       | `show arp` and `show arp statistics`                  | ASA cannot ARP for the next hop                                                  |
| Packet exits wrong interface                | `show conn`, `show route <dst-ip>`, `show nat detail` | Existing connection, UN-NAT, PBR, or route lookup changed egress                 |
| NAT translation is missing                  | `show xlate`                                          | NAT rule did not match or traffic has not generated translation                  |
| NAT translation is wrong                    | `show nat detail` and `packet-tracer ... detailed`    | NAT order or object definition is wrong                                          |
| ACL hit count does not increase             | `show access-list <acl-name>`                         | Traffic does not match that ACE or ACL is applied to wrong interface             |
| Connection remains embryonic                | `show conn detail`                                    | Return traffic is missing, server does not respond, or downstream path is broken |
| ASP drop counter increases                  | `show asp drop`                                       | Drop reason points to the firewall decision that killed the flow                 |
| Capture filter shows nothing                | `show capture <cap-name>`                             | Wrong source, destination, protocol, port, NATed address, or interface           |
| Real traffic fails but packet-tracer allows | `show arp`, `show capture <egress-cap-name>`          | Adjacency, return path, or downstream device problem                             |
| Return traffic fails                        | `show route <src-ip>`                                 | Missing return route on ASA or downstream router                                 |
# ASA_Packet_Flow_Troubleshooting_Cleanup

| Step | Task                             | Device     | Command                                   | Expected Result              |
| ---: | -------------------------------- | ---------- | ----------------------------------------- | ---------------------------- |
|    1 | Remove ingress capture           | ASA        | `no capture <ingress-cap-name>`           | Capture is removed           |
|    2 | Remove egress capture            | ASA        | `no capture <egress-cap-name>`            | Capture is removed           |
|    3 | Remove ASP capture               | ASA        | `no capture <asp-cap-name>`               | ASP drop capture is removed  |
|    4 | Remove ARP capture               | ASA        | `no capture <arp-cap-name>`               | ARP capture is removed       |
|    5 | Clear stale connection if needed | ASA        | `clear conn address <ip-address>`         | Stale connection is removed  |
|    6 | Clear stale xlate if needed      | ASA        | `clear xlate`                             | Translation table is cleared |
|    7 | Re-test intended flow            | ASA / Host | `packet-tracer ...` and real traffic test | Flow now behaves as expected |
