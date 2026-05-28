

| Symptom | Command | What Usually Broke |
|---|---|---|
| No EIGRP neighbor | `show ip eigrp neighbors` | AS mismatch, passive interface, wrong network statement, interface down |
| Neighbor forms but routes missing | `show ip protocols` | Missing `network` statement or passive interface issue |
| Route appears in topology but not routing table | `show ip eigrp topology` | Route is not successor or worse than another route |
| Unexpected summary route | `show run interface <interface>` | `ip summary-address eigrp` configured |
| Routes missing behind branch router | `show ip eigrp neighbors detail` | Stub configuration limiting advertisements |
| Authentication lab fails | `show ip eigrp neighbors` | Key-chain/authentication mismatch |
| Network command looks right but interface not included | `show ip protocols` | Wildcard mask does not match the interface IP |