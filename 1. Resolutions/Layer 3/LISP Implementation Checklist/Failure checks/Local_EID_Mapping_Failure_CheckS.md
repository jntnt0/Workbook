

| Symptom | Command | What Usually Broke |
|---|---|---|
| Local EID does not appear in LISP database | `show ip lisp database` | Missing or wrong `database-mapping` |
| Site does not register on map-server | `show lisp site` | Key mismatch, EID-prefix mismatch, or RLOC reachability failure |
| Map-server rejects the EID | `show running-config | section router lisp` | xTR `database-mapping` does not match MSMR `eid-prefix` |
| Remote xTR cannot resolve the EID | `lig <eid-ip>` | Map-resolver missing, MSMR unreachable, or site not registered |
| Ping fails even though registration works | `show ip route <remote-rloc-ip>` | RLOC underlay routing problem |
| Local host subnet is not advertised | `show ip route <eid-prefix>` | EID subnet is not present or is mismatched |
| Wrong locator is returned | `show ip lisp database` | Incorrect RLOC IP in `database-mapping` |
| Old mapping keeps being used | `clear ip lisp map-cache *` | Stale map-cache entry after config change |
