# LISP_Baseline_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the LISP roles | All devices | `show running-config | section router lisp` | You know which devices are xTRs and which device is the map-server/map-resolver |
| 2 | Verify underlay interfaces are up | All LISP routers | `show ip interface brief` | RLOC-facing and EID-facing interfaces show `up/up` |
| 3 | Verify CEF is enabled | All LISP routers | `show running-config | include ip cef` | `ip cef` is enabled |
| 4 | Verify xTR underlay reachability to map-server/map-resolver | xTR routers | `ping <map-server-rloc-ip>` | Ping succeeds from the xTR to the MSMR |
| 5 | Verify map-server/map-resolver reachability to xTR RLOCs | MSMR | `ping <xtr-rloc-ip>` | Ping succeeds from the MSMR to each xTR RLOC |
| 6 | Verify EID-side connected routes | xTR routers | `show ip route connected` | Local EID subnet appears as a connected route |
| 7 | Verify RLOC-side route toward the core | xTR routers | `show ip route <map-server-rloc-ip>` | xTR has a route toward the map-server/map-resolver |
| 8 | Verify MSMR route toward xTR RLOCs | MSMR | `show ip route <xtr-rloc-ip>` | MSMR has a route back to each xTR RLOC |
| 9 | Enter LISP configuration mode on the MSMR | MSMR | `conf t` then `router lisp` | Device enters LISP router configuration mode |
| 10 | Create the LISP site definition | MSMR | `site <site-name>` | LISP site configuration mode is entered |
| 11 | Configure the site authentication key | MSMR | `authentication-key 0 <key>` | Site key matches the key that will be configured on the xTR |
| 12 | Define the allowed EID prefix | MSMR | `eid-prefix <eid-prefix>/<length>` | MSMR is authoritative for the site EID prefix |
| 13 | Exit the site section | MSMR | `exit` | Device returns to LISP configuration mode |
| 14 | Enable IPv4 map-server function | MSMR | `ipv4 map-server` | MSMR can accept EID registrations from ETRs |
| 15 | Enable IPv4 map-resolver function | MSMR | `ipv4 map-resolver` | MSMR can answer map requests from ITRs |
| 16 | Save MSMR configuration | MSMR | `end` then `write memory` | MSMR LISP configuration is saved |
| 17 | Enter LISP configuration mode on the xTR | xTR router | `conf t` then `router lisp` | xTR enters LISP router configuration mode |
| 18 | Configure EID-to-RLOC database mapping | xTR router | `database-mapping <eid-prefix>/<length> <rloc-ip> priority <priority> weight <weight>` | Local EID prefix is mapped to the xTR RLOC |
| 19 | Add additional RLOC mapping if multihomed | xTR router | `database-mapping <eid-prefix>/<length> <second-rloc-ip> priority <priority> weight <weight>` | Same EID can be reached through multiple RLOCs |
| 20 | Enable IPv4 ITR function | xTR router | `ipv4 itr` | Router can encapsulate traffic toward remote EIDs |
| 21 | Enable IPv4 ETR function | xTR router | `ipv4 etr` | Router can decapsulate traffic for local EIDs |
| 22 | Point ITR toward the map-resolver | xTR router | `ipv4 itr map-resolver <map-resolver-rloc-ip>` | xTR knows where to send map requests |
| 23 | Point ETR toward the map-server | xTR router | `ipv4 etr map-server <map-server-rloc-ip> key 0 <key>` | xTR can register its EID prefix with the MSMR |
| 24 | Add second map-resolver if redundant MSMR exists | xTR router | `ipv4 itr map-resolver <second-map-resolver-rloc-ip>` | xTR has redundant map resolution path |
| 25 | Add second map-server if redundant MSMR exists | xTR router | `ipv4 etr map-server <second-map-server-rloc-ip> key 0 <key>` | xTR has redundant EID registration path |
| 26 | Configure default or underlay route if needed | xTR router | `ip route 0.0.0.0 0.0.0.0 <underlay-next-hop>` | xTR has underlay reachability for LISP control and data traffic |
| 27 | Save xTR configuration | xTR router | `end` then `write memory` | xTR LISP configuration is saved |
| 28 | Verify final LISP configuration | All LISP routers | `show running-config | section router lisp` | LISP process, site, database mapping, ITR, ETR, MS, and MR commands are present as expected |
| 29 | Verify LISP process status | xTR router | `show ip lisp` | LISP IPv4 process is active with expected settings |
| 30 | Verify local EID database mapping | xTR router | `show ip lisp database` | Local EID prefix appears in the LISP database |
| 31 | Verify EID registration on map-server | MSMR | `show lisp site` | Site shows `Up yes` and the xTR RLOC as the registering device |
| 32 | Verify one specific site registration | MSMR | `show lisp site name <site-name>` | Correct EID prefix and registering xTR are displayed |
| 33 | Verify remote EID map-cache entry | xTR router | `show ip lisp map-cache` | Remote EID appears with an RLOC and `complete` state after traffic or lookup |
| 34 | Force a LISP lookup | xTR router | `lig <remote-eid-ip>` | LISP lookup returns the remote EID-to-RLOC mapping |
| 35 | Test EID-to-EID forwarding | Source host or xTR | `ping <remote-eid-ip>` | Ping succeeds between EID subnets |
| 36 | Verify encapsulation path | Source host or xTR | `traceroute <remote-eid-ip>` | Path follows the expected LISP/underlay direction |
| 37 | Clear map-cache after policy or mapping changes | xTR router | `clear ip lisp map-cache *` | Dynamic map-cache entries are cleared and relearned |
| 38 | Recheck map-cache after traffic resumes | xTR router | `show ip lisp map-cache` | Remote EID mapping repopulates correctly |
| 39 | Check for key mismatch | MSMR | `show lisp site name <site-name>` | Site registration should not be missing or stuck down |
| 40 | Check for EID-prefix mismatch | MSMR and xTR | `show running-config | section router lisp` | MSMR `eid-prefix` matches xTR `database-mapping` prefix |
| 41 | Check for RLOC reachability problem | xTR and MSMR | `ping <remote-rloc-ip>` | RLOC-to-RLOC underlay reachability works both directions |
| 42 | Confirm final forwarding state | All LISP routers | `show ip route` and `show ip lisp map-cache` | Underlay routes and LISP mappings both look correct |