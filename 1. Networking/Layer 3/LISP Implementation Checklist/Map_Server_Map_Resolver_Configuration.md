
| Step | Task                                             | Device            | Command                                                             | Expected Result                                                                   |                                                                          |
| ---: | ------------------------------------------------ | ----------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
|    1 | Identify the MSMR device                         | MSMR              | `show running-config                                                | section router lisp`                                                              | You know which router will act as the Map-Server / Map-Resolver          |
|    2 | Verify MSMR interface state                      | MSMR              | `show ip interface brief`                                           | RLOC-facing interface shows `up/up`                                               |                                                                          |
|    3 | Verify MSMR RLOC address                         | MSMR              | `show ip interface brief`                                           | Correct underlay IP address is assigned to the MSMR                               |                                                                          |
|    4 | Verify route to each xTR RLOC                    | MSMR              | `show ip route <xtr-rloc-ip>`                                       | MSMR has underlay reachability to each xTR RLOC                                   |                                                                          |
|    5 | Test reachability to each xTR RLOC               | MSMR              | `ping <xtr-rloc-ip>`                                                | Ping succeeds from MSMR to each xTR RLOC                                          |                                                                          |
|    6 | Verify xTRs can reach the MSMR RLOC              | xTR / ETR         | `ping <msmr-rloc-ip>`                                               | xTRs can reach the Map-Server / Map-Resolver                                      |                                                                          |
|    7 | Enter global configuration mode                  | MSMR              | `conf t`                                                            | Device enters global configuration mode                                           |                                                                          |
|    8 | Enter LISP configuration mode                    | MSMR              | `router lisp`                                                       | Device enters LISP router configuration mode                                      |                                                                          |
|    9 | Create the LISP site                             | MSMR              | `site <site-name>`                                                  | Site configuration mode is entered                                                |                                                                          |
|   10 | Configure the site authentication key            | MSMR              | `authentication-key 0 <key>`                                        | Site key is configured                                                            |                                                                          |
|   11 | Permit the site EID prefix                       | MSMR              | `eid-prefix <eid-prefix>/<length>`                                  | MSMR accepts registration for that exact EID prefix                               |                                                                          |
|   12 | Permit more-specific EID registrations if needed | MSMR              | `eid-prefix <summary-eid-prefix>/<length> accept-more-specifics`    | MSMR accepts more-specific prefixes inside the summary                            |                                                                          |
|   13 | Exit the site configuration                      | MSMR              | `exit`                                                              | Device returns to LISP router configuration mode                                  |                                                                          |
|   14 | Enable IPv4 Map-Server function                  | MSMR              | `ipv4 map-server`                                                   | MSMR can receive EID registrations from ETRs                                      |                                                                          |
|   15 | Enable IPv4 Map-Resolver function                | MSMR              | `ipv4 map-resolver`                                                 | MSMR can answer map requests from ITRs                                            |                                                                          |
|   16 | Repeat site configuration for additional sites   | MSMR              | `site <second-site-name>`                                           | Additional EID spaces are allowed to register                                     |                                                                          |
|   17 | Save the MSMR configuration                      | MSMR              | `end` then `write memory`                                           | MSMR configuration is saved                                                       |                                                                          |
|   18 | Verify LISP running configuration                | MSMR              | `show running-config                                                | section router lisp`                                                              | Site, key, EID prefix, map-server, and map-resolver commands are present |
|   19 | Verify Map-Server / Map-Resolver function        | MSMR              | `show ip lisp`                                                      | LISP is enabled and MSMR role is active                                           |                                                                          |
|   20 | Verify registered sites                          | MSMR              | `show lisp site`                                                    | Site appears and shows registration status                                        |                                                                          |
|   21 | Verify a specific site                           | MSMR              | `show lisp site name <site-name>`                                   | Correct EID prefix and registering xTR RLOC are shown                             |                                                                          |
|   22 | Verify site registration after xTR config        | MSMR              | `show lisp site`                                                    | Site shows registered/up after the ETR registers                                  |                                                                          |
|   23 | Verify xTR database mapping                      | xTR / ETR         | `show ip lisp database`                                             | xTR has local EID-to-RLOC mapping configured                                      |                                                                          |
|   24 | Verify remote map resolution                     | Remote xTR        | `lig <eid-ip>`                                                      | Lookup returns the EID-to-RLOC mapping from the MSMR                              |                                                                          |
|   25 | Verify remote map-cache population               | Remote xTR        | `show ip lisp map-cache <eid-prefix>`                               | Remote xTR learns the EID mapping                                                 |                                                                          |
|   26 | Test EID-to-EID reachability                     | Source host / xTR | `ping <remote-eid-ip>`                                              | Traffic succeeds across the LISP overlay                                          |                                                                          |
|   27 | Verify forwarding path                           | Source host / xTR | `traceroute <remote-eid-ip>`                                        | Traffic follows the expected RLOC/underlay path                                   |                                                                          |
|   28 | Clear stale map-cache after changes              | xTR               | `clear ip lisp map-cache *`                                         | Old cached mappings are removed                                                   |                                                                          |
|   29 | Recheck registration and resolution              | MSMR / xTR        | `show lisp site` and `show ip lisp map-cache`                       | Site registration and map-cache entries repopulate correctly                      |                                                                          |
|   30 | Confirm final operational state                  | MSMR / xTR        | `show lisp site`, `show ip lisp database`, `show ip lisp map-cache` | MSMR registration, local database, and remote cache all match the intended design |                                                                          |

---
```
conf t
router lisp
 site <site-name>
  authentication-key 0 <key>
  eid-prefix <eid-prefix>/<length>
 exit
 ipv4 map-server
 ipv4 map-resolver
end
write memory
```

```
conf t
router lisp
 site <site-name>
  authentication-key 0 <key>
  eid-prefix <summary-eid-prefix>/<length> accept-more-specifics
 exit
 ipv4 map-server
 ipv4 map-resolver
end
write memory
```

