sudo ifconfig eth0 192.168.1.10 netmask 255.255.255.0 up
- No Gateway


|                     |                                                     |
| ------------------- | --------------------------------------------------- |
| Task                | Command                                             |
| Assign IP           | sudo ifconfig eth0 10.0.0.50 netmask 255.255.0.0 up |
| Add default gateway | sudo route add default gw <gateway-ip>              |
| Verify              | ifconfig eth0                                       |
| Test gateway        | ping <gateway-ip>                                   |
