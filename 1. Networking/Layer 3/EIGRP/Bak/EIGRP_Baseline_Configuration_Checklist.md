|      |                                                 |                       |                                           |                                                                          |
| ---- | ----------------------------------------------- | --------------------- | ----------------------------------------- | ------------------------------------------------------------------------ |
| Step | Task                                            | Device                | Command                                   | Expected Result                                                          |
| 1    | Verify router interfaces are up                 | All EIGRP routers     | show ip interface brief                   | Peer-facing interfaces show up/up                                        |
| 2    | Verify connected routes exist                   | All EIGRP routers     | show ip route connected                   | Local LANs and transit links appear as connected routes                  |
| 3    | Verify direct neighbor reachability             | Local router          | ping <neighbor-interface-ip>              | Ping succeeds before EIGRP is configured                                 |
| 4    | Enter global configuration mode                 | Local router          | conf t                                    | Router enters configuration mode                                         |
| 5    | Start the EIGRP process                         | Local router          | router eigrp <as-number>                  | EIGRP process is created                                                 |
| 6    | Set a stable EIGRP router ID                    | Local router          | eigrp router-id <router-id>               | Router ID is manually defined                                            |
| 7    | Disable automatic classful summarization        | Local router          | no auto-summary                           | Router does not auto-summarize at classful boundaries                    |
| 8    | Advertise the transit network                   | Local router          | network <transit-network> <wildcard-mask> | EIGRP runs on the router-to-router link                                  |
| 9    | Advertise the local LAN network                 | Local router          | network <lan-network> <wildcard-mask>     | Local LAN becomes eligible for EIGRP advertisement                       |
| 10   | Make all interfaces passive by default          | Local router          | passive-interface default                 | EIGRP does not form unwanted adjacencies                                 |
| 11   | Enable EIGRP on the neighbor-facing interface   | Local router          | no passive-interface <interface>          | EIGRP hellos are sent on the peer link                                   |
| 12   | Exit EIGRP configuration mode                   | Local router          | exit                                      | Returns to global configuration mode                                     |
| 13   | Save the configuration                          | Local router          | end then write memory                     | EIGRP config is saved                                                    |
| 14   | Verify EIGRP process settings                   | Local router          | show ip protocols                         | Correct AS number, router ID, networks, and passive interfaces are shown |
| 15   | Verify EIGRP-enabled interfaces                 | Local router          | show ip eigrp interfaces                  | Correct interfaces are participating in EIGRP                            |
| 16   | Verify neighbor adjacency                       | Local router          | show ip eigrp neighbors                   | Neighbor appears with uptime and hold timer                              |
| 17   | Verify EIGRP topology table                     | Local router          | show ip eigrp topology                    | Learned prefixes appear with successor routes                            |
| 18   | Verify EIGRP routes in routing table            | Local router          | show ip route eigrp                       | Learned EIGRP routes appear with D code                                  |
| 19   | Verify a specific learned prefix                | Local router          | show ip route <remote-prefix>             | Remote route points to the expected next hop                             |
| 20   | Test end-to-end forwarding                      | Source router or host | ping <remote-lan-ip>                      | Ping succeeds across the EIGRP domain                                    |
| 21   | Trace the forwarding path                       | Source router or host | traceroute <remote-lan-ip>                | Path follows the expected routers                                        |
| 22   | Check for active route problems                 | Local router          | show ip eigrp topology active             | Ideally no routes are stuck active                                       |
| 23   | Check for mismatched EIGRP settings             | Local router          | show ip eigrp neighbors detail            | Neighbor details show stable adjacency behavior                          |
| 24   | Verify no unwanted interfaces are participating | Local router          | show ip protocols                         | Only intended interfaces are non-passive                                 |
| 25   | Confirm final routing state                     | All EIGRP routers     | show ip route                             | Connected, local, and EIGRP routes are present as expected               |

Basic EIGRP IPv4 skeleton:
```
conf t
router eigrp <as-number>
 eigrp router-id <router-id>
 no auto-summary
 network <transit-network> <wildcard-mask>
 network <local-lan-network> <wildcard-mask>
 passive-interface default
 no passive-interface <neighbor-facing-interface>
end
write memory
```
Example:
```
conf t
router eigrp 100
 eigrp router-id 1.1.1.1
 no auto-summary
 network 10.0.12.0 0.0.0.3
 network 192.168.10.0 0.0.0.255
 passive-interface default
 no passive-interface GigabitEthernet0/0
end
write memory
```

Optional EIGRP summary route checklist:
Title: Optional_EIGRP_Summary_Route_Checklist

| Step | Task                               | Device             | Command                                                                | Expected Result                                                 |
| ---: | ---------------------------------- | ------------------ | ---------------------------------------------------------------------- | --------------------------------------------------------------- |
|    1 | Enter the outbound interface       | Summarizing router | `interface <interface>`                                                | Router enters interface configuration mode                      |
|    2 | Configure EIGRP summary address    | Summarizing router | `ip summary-address eigrp <as-number> <summary-prefix> <summary-mask>` | Summary route is advertised out that interface                  |
|    3 | Verify summary route behavior      | Neighbor router    | `show ip route eigrp`                                                  | Neighbor sees the summary route instead of every specific route |
|    4 | Verify local summary discard route | Summarizing router | `show ip route <summary-prefix>`                                       | Local router may install a discard route to `Null0`             |

Title: Optional_EIGRP_Stub_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter EIGRP process | Stub router | `router eigrp <as-number>` | Router enters EIGRP config mode |
| 2 | Configure EIGRP stub | Stub router | `eigrp stub connected summary` | Router advertises only connected and summary routes |
| 3 | Verify stub status | Neighbor router | `show ip eigrp neighbors detail` | Neighbor identifies the router as a stub |

Title: Common_EIGRP_Failure_Checks

| Symptom                                                | Command                          | What Usually Broke                                                      |
| ------------------------------------------------------ | -------------------------------- | ----------------------------------------------------------------------- |
| No EIGRP neighbor                                      | `show ip eigrp neighbors`        | AS mismatch, passive interface, wrong network statement, interface down |
| Neighbor forms but routes missing                      | `show ip protocols`              | Missing `network` statement or passive interface issue                  |
| Route appears in topology but not routing table        | `show ip eigrp topology`         | Route is not successor or worse than another route                      |
| Unexpected summary route                               | `show run interface <interface>` | `ip summary-address eigrp` configured                                   |
| Routes missing behind branch router                    | `show ip eigrp neighbors detail` | Stub configuration limiting advertisements                              |
| Authentication lab fails                               | `show ip eigrp neighbors`        | Key-chain/authentication mismatch                                       |
| Network command looks right but interface not included | `show ip protocols`              | Wildcard mask does not match the interface IP                           |
