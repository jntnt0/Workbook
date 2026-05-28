
| Symptom | Command | What Usually Broke |
|---|---|---|
| Site does not register | `show lisp site` | xTR cannot reach MSMR, key mismatch, or EID prefix mismatch |
| Site shows wrong EID | `show lisp site name <site-name>` | MSMR `eid-prefix` does not match xTR `database-mapping` |
| xTR registration rejected | `show running-config | section router lisp` | Missing `accept-more-specifics` or wrong site prefix |
| Remote xTR cannot resolve EID | `lig <eid-ip>` | Map-Resolver not enabled or MSMR unreachable |
| Local database exists but remote cache is empty | `show ip lisp map-cache` | No traffic triggered lookup, map-resolver missing, or stale cache |
| Ping fails after successful registration | `show ip route <remote-rloc-ip>` | Underlay RLOC reachability problem |
| MSMR config looks correct but nothing registers | `ping <xtr-rloc-ip>` | Routing, ACL, firewall, or wrong RLOC address |
| Registration works only for exact prefix | `show running-config | section router lisp` | Site needs `accept-more-specifics` for more-specific EID registrations |
| Wrong site receives registration | `show lisp site` | Overlapping EID prefixes or incorrect site definition |
| Map-cache keeps old locator | `clear ip lisp map-cache *` | Stale map-cache after RLOC or EID mapping change |