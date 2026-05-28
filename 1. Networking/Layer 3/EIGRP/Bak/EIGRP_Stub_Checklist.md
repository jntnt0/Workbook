EIGRP_Stub_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter EIGRP process | Stub router | `router eigrp <as-number>` | Router enters EIGRP config mode |
| 2 | Configure EIGRP stub | Stub router | `eigrp stub connected summary` | Router advertises only connected and summary routes |
| 3 | Verify stub status | Neighbor router | `show ip eigrp neighbors detail` | Neighbor identifies the router as a stub |