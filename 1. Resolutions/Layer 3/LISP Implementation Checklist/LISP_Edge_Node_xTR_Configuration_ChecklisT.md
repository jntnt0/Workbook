

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the xTR role | xTR | `show running-config | section router lisp` | Router is intended to act as both ITR and ETR |
| 2 | Verify EID-facing interface state | xTR | `show ip interface brief` | LAN/EID-facing interface shows `up/up` |
| 3 | Verify RLOC-facing interface state | xTR | `show ip interface brief` | WAN/underlay-facing interface shows `up/up` |
| 4 | Verify local EID subnet exists | xTR | `show ip route connected` | Local EID subnet appears as a connected route |
| 5 | Verify local RLOC address | xTR | `show ip interface brief` | RLOC-facing interface has the intended locator IP |
| 6 | Verify route to Map-Resolver | xTR | `show ip route <map-resolver-rloc-ip>` | xTR has an underlay route to the Map-Resolver |
| 7 | Verify route to Map-Server | xTR | `show ip route <map-server-rloc-ip>` | xTR has an underlay route to the Map-Server |
| 8 | Test reachability to Map-Resolver | xTR | `ping <map-resolver-rloc-ip>` | Ping succeeds |
| 9 | Test reachability to Map-Server | xTR | `ping <map-server-rloc-ip>` | Ping succeeds |
| 10 | Enter global configuration mode | xTR | `conf t` | Router enters global configuration mode |
| 11 | Enter LISP configuration mode | xTR | `router lisp` | Router enters LISP configuration mode |
| 12 | Map local EID prefix to primary RLOC | xTR | `database-mapping <eid-prefix>/<length> <primary-rloc-ip> priority <priority> weight <weight>` | Local EID prefix is associated with the primary RLOC |
| 13 | Map local EID prefix to backup or second RLOC | xTR | `database-mapping <eid-prefix>/<length> <second-rloc-ip> priority <priority> weight <weight>` | Local EID prefix has an additional RLOC |
| 14 | Enable ITR function | xTR | `ipv4 itr` | Router can encapsulate traffic toward remote EIDs |
| 15 | Enable ETR function | xTR | `ipv4 etr` | Router can decapsulate traffic for local EIDs and register mappings |
| 16 | Point ITR to Map-Resolver | xTR | `ipv4 itr map-resolver <map-resolver-rloc-ip>` | xTR knows where to send EID-to-RLOC map requests |
| 17 | Add redundant Map-Resolver if available | xTR | `ipv4 itr map-resolver <second-map-resolver-rloc-ip>` | xTR has a second map-resolution path |
| 18 | Point ETR to Map-Server | xTR | `ipv4 etr map-server <map-server-rloc-ip> key 0 <key>` | xTR can register local EID mapping with the Map-Server |
| 19 | Add redundant Map-Server if available | xTR | `ipv4 etr map-server <second-map-server-rloc-ip> key 0 <key>` | xTR has a second registration target |
| 20 | Exit LISP configuration mode | xTR | `exit` | Router returns to global configuration mode |
| 21 | Configure underlay default route if needed | xTR | `ip route 0.0.0.0 0.0.0.0 <underlay-next-hop>` | xTR has general underlay reachability |
| 22 | Configure specific underlay routes if preferred | xTR | `ip route <msmr-prefix> <mask> <underlay-next-hop>` | xTR can reach mapping infrastructure without relying on default route |
| 23 | Save configuration | xTR | `end` then `write memory` | xTR configuration is saved |
| 24 | Verify LISP running configuration | xTR | `show running-config | section router lisp` | `database-mapping`, `ipv4 itr`, `ipv4 etr`, map-resolver, and map-server commands are present |
| 25 | Verify LISP operational role | xTR | `show ip lisp` | ITR and ETR show enabled |
| 26 | Verify local EID database | xTR | `show ip lisp database` | Local EID prefix appears with correct RLOC, priority, and weight |
| 27 | Verify Map-Server registration | MSMR | `show lisp site` | Site shows registered and up |
| 28 | Verify detailed site registration | MSMR | `show lisp site name <site-name>` | EID prefix, registering RLOC, and registration state are correct |
| 29 | Force lookup for remote EID | xTR | `lig <remote-eid-ip>` | xTR receives remote EID-to-RLOC mapping |
| 30 | Verify remote map-cache | xTR | `show ip lisp map-cache <remote-eid-prefix>` | Remote EID appears in map-cache with RLOC information |
| 31 | Test EID-to-EID reachability | Source host / xTR | `ping <remote-eid-ip>` | Ping succeeds across the LISP overlay |
| 32 | Verify forwarding path | Source host / xTR | `traceroute <remote-eid-ip>` | Traffic follows expected LISP/underlay path |
| 33 | Clear stale map-cache after changes | xTR | `clear ip lisp map-cache *` | Old dynamic mappings are cleared |
| 34 | Re-test after cache clear | xTR | `ping <remote-eid-ip>` then `show ip lisp map-cache` | Map-cache repopulates and traffic succeeds |
| 35 | Confirm final state | xTR / MSMR | `show ip lisp`, `show ip lisp database`, `show ip lisp map-cache`, `show lisp site` | xTR role, local database, remote cache, and registration all match the intended design |


---

```
conf t
router lisp
 database-mapping <eid-prefix>/<length> <xtr-rloc-ip> priority 1 weight 100
 ipv4 itr
 ipv4 etr
 ipv4 itr map-resolver <map-resolver-rloc-ip>
 ipv4 etr map-server <map-server-rloc-ip> key 0 <key>
exit
ip route 0.0.0.0 0.0.0.0 <underlay-next-hop>
end
write memory
```


---
```
conf t
router lisp
 database-mapping <eid-prefix>/<length> <primary-rloc-ip> priority 1 weight 50
 database-mapping <eid-prefix>/<length> <secondary-rloc-ip> priority 1 weight 50
 ipv4 itr
 ipv4 etr
 ipv4 itr map-resolver <map-resolver-1-rloc-ip>
 ipv4 itr map-resolver <map-resolver-2-rloc-ip>
 ipv4 etr map-server <map-server-1-rloc-ip> key 0 <key>
 ipv4 etr map-server <map-server-2-rloc-ip> key 0 <key>
exit
ip route 0.0.0.0 0.0.0.0 <primary-underlay-next-hop>
ip route 0.0.0.0 0.0.0.0 <secondary-underlay-next-hop>
end
write memory
```