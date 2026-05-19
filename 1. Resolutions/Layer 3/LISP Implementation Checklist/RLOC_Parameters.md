

| Step | Task                                              | Device               | Command                                                                                                      | Expected Result                                                             |                                                                         |
| ---: | ------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
|    1 | Identify the local EID prefix                     | xTR / ETR            | `show ip route connected`                                                                                    | The local EID subnet is present as a connected route                        |                                                                         |
|    2 | Identify the local RLOC interface                 | xTR / ETR            | `show ip interface brief`                                                                                    | RLOC-facing interface has the locator IP and shows `up/up`                  |                                                                         |
|    3 | Verify RLOC is reachable in the underlay          | xTR / ETR            | `show ip route <remote-rloc-ip>`                                                                             | Router has an underlay route toward the remote RLOC                         |                                                                         |
|    4 | Verify map-server reachability from the RLOC side | xTR / ETR            | `ping <map-server-rloc-ip>`                                                                                  | xTR can reach the map-server/map-resolver                                   |                                                                         |
|    5 | Verify reverse reachability to this RLOC          | MSMR                 | `ping <xtr-rloc-ip>`                                                                                         | MSMR can reach the xTR RLOC                                                 |                                                                         |
|    6 | Enter LISP configuration mode                     | xTR / ETR            | `conf t` then `router lisp`                                                                                  | Router enters LISP configuration mode                                       |                                                                         |
|    7 | Configure the primary RLOC for the EID prefix     | xTR / ETR            | `database-mapping <eid-prefix>/<length> <primary-rloc-ip> priority <priority> weight <weight>`               | EID prefix is mapped to the primary RLOC                                    |                                                                         |
|    8 | Configure a backup RLOC for the same EID prefix   | xTR / ETR            | `database-mapping <eid-prefix>/<length> <backup-rloc-ip> priority <higher-priority-value> weight <weight>`   | Backup RLOC is registered but used only if preferred RLOC is unavailable    |                                                                         |
|    9 | Configure equal-priority RLOCs for load sharing   | xTR / ETR            | `database-mapping <eid-prefix>/<length> <second-rloc-ip> priority <same-priority> weight <different-weight>` | Multiple RLOCs are eligible, with traffic share influenced by weight        |                                                                         |
|   10 | Prefer one RLOC over another                      | xTR / ETR            | `database-mapping <eid-prefix>/<length> <preferred-rloc-ip> priority 1 weight 100`                           | RLOC with the lowest priority value is preferred                            |                                                                         |
|   11 | De-prefer a backup RLOC                           | xTR / ETR            | `database-mapping <eid-prefix>/<length> <backup-rloc-ip> priority 10 weight 0`                               | Backup RLOC is less preferred than the primary RLOC                         |                                                                         |
|   12 | Verify ITR function is enabled                    | xTR                  | `show running-config                                                                                         | section router lisp`                                                        | `ipv4 itr` is present if this router must encapsulate traffic           |
|   13 | Verify ETR function is enabled                    | xTR / ETR            | `show running-config                                                                                         | section router lisp`                                                        | `ipv4 etr` is present if this router must register EID-to-RLOC mappings |
|   14 | Verify map-resolver is configured                 | xTR                  | `show running-config                                                                                         | section router lisp`                                                        | `ipv4 itr map-resolver <map-resolver-rloc-ip>` is present               |
|   15 | Verify map-server is configured with matching key | xTR / ETR            | `show running-config                                                                                         | section router lisp`                                                        | `ipv4 etr map-server <map-server-rloc-ip> key 0 <key>` is present       |
|   16 | Save the RLOC parameter configuration             | xTR / ETR            | `end` then `write memory`                                                                                    | Configuration is saved                                                      |                                                                         |
|   17 | Verify local LISP database mapping                | xTR / ETR            | `show ip lisp database`                                                                                      | EID prefix shows the configured RLOC, priority, and weight                  |                                                                         |
|   18 | Verify LISP site registration                     | MSMR                 | `show lisp site`                                                                                             | Site shows registered EID prefix and xTR RLOC                               |                                                                         |
|   19 | Verify detailed site registration                 | MSMR                 | `show lisp site name <site-name>`                                                                            | Registered RLOC, priority, and weight match the intended design             |                                                                         |
|   20 | Verify remote map-cache entry                     | Remote xTR           | `show ip lisp map-cache <eid-prefix>`                                                                        | Remote xTR learns the EID-to-RLOC mapping                                   |                                                                         |
|   21 | Force a LISP lookup                               | Remote xTR           | `lig <remote-eid-ip>`                                                                                        | Lookup returns the correct RLOC list                                        |                                                                         |
|   22 | Test EID-to-EID forwarding                        | Source host / router | `ping <remote-eid-ip>`                                                                                       | Traffic succeeds across the LISP overlay                                    |                                                                         |
|   23 | Verify forwarding path                            | Source host / router | `traceroute <remote-eid-ip>`                                                                                 | Traffic follows the expected underlay/RLOC path                             |                                                                         |
|   24 | Clear map-cache after changing RLOC parameters    | xTR                  | `clear ip lisp map-cache *`                                                                                  | Old EID-to-RLOC entries are cleared                                         |                                                                         |
|   25 | Re-test after cache clear                         | xTR / Host           | `ping <remote-eid-ip>` then `show ip lisp map-cache`                                                         | Map-cache repopulates with updated RLOC parameters                          |                                                                         |
|   26 | Confirm final state                               | xTR / ETR / MSMR     | `show ip lisp database`, `show ip lisp map-cache`, `show lisp site`                                          | RLOC mapping, registration, and learned cache all match the intended design |                                                                         |

---
# RLOC_Parameter_Rules

| Parameter | Meaning | Operational Rule |
|---|---|---|
| RLOC IP | Underlay-reachable locator address | Must be reachable through the non-LISP underlay |
| Priority | Primary path selection value | Lower value wins |
| Weight | Load-sharing value among same-priority RLOCs | Higher weight receives more traffic share |
| EID prefix | Endpoint subnet being advertised into LISP | Must match the site prefix allowed on the map-server |
| Map-server key | Authentication key for EID registration | Must match between xTR/ETR and MSMR |
| Map-cache | Learned remote EID-to-RLOC mapping | Clear it after RLOC parameter changes if stale behavior appears |



---

```
conf t
router lisp
 database-mapping <eid-prefix>/<length> <primary-rloc-ip> priority 1 weight 100
 database-mapping <eid-prefix>/<length> <backup-rloc-ip> priority 10 weight 0
 ipv4 itr
 ipv4 etr
 ipv4 itr map-resolver <map-resolver-rloc-ip>
 ipv4 etr map-server <map-server-rloc-ip> key 0 <key>
end
write memory
```