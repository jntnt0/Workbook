

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter the outbound interface | Summarizing router | `interface <interface>` | Router enters interface configuration mode |
| 2 | Configure EIGRP summary address | Summarizing router | `ip summary-address eigrp <as-number> <summary-prefix> <summary-mask>` | Summary route is advertised out that interface |
| 3 | Verify summary route behavior | Neighbor router | `show ip route eigrp` | Neighbor sees the summary route instead of every specific route |
| 4 | Verify local summary discard route | Summarizing router | `show ip route <summary-prefix>` | Local router may install a discard route to `Null0` |