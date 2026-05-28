| Task               | IOS XE Command                                 |
| ------------------ | ---------------------------------------------- |
| Enter config mode  | `configure terminal`                           |
| Disable DNS lookup | `no ip domain-lookup`                          |
| Exit               | `end`                                          |
| Save               | `write memory`                                 |
| Check              | Command                                        |
| Confirm setting    | `show running-config \| include domain-lookup` |


| Output                | Meaning                                                               |
| --------------------- | --------------------------------------------------------------------- |
| `no ip domain-lookup` | DNS lookup disabled                                                   |
| No output             | May still be default/lookup enabled depending platform/config display |