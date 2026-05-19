

| Symptom | Command | What Usually Broke |
|---|---|---|
| xTR does not show ITR enabled | `show ip lisp` | Missing `ipv4 itr` |
| xTR does not show ETR enabled | `show ip lisp` | Missing `ipv4 etr` |
| Local EID not in LISP database | `show ip lisp database` | Missing or incorrect `database-mapping` |
| Site does not register | `show lisp site` | Key mismatch, EID-prefix mismatch, or RLOC reachability issue |
| Remote EID does not resolve | `lig <remote-eid-ip>` | Missing map-resolver, MSMR unreachable, or remote site not registered |
| Map-cache is empty | `show ip lisp map-cache` | No lookup triggered, map-resolver issue, or stale/cleared cache |
| Ping fails despite registration | `show ip route <remote-rloc-ip>` | Underlay RLOC reachability failure |
| Wrong RLOC appears | `show ip lisp database` | Incorrect RLOC in `database-mapping` |
| Backup RLOC used unexpectedly | `show ip lisp database` | Priority/weight values do not match intended design |
| Remote mapping stays old | `clear ip lisp map-cache *` | Stale dynamic map-cache entry |
| xTR cannot reach MSMR | `ping <map-server-rloc-ip>` | Underlay route, interface, ACL, or next-hop issue |