# Network commands

- Check the connection with the host: `ping <address>`
- View network interface settings: `ifconfig or ip addr`
- Trace the route to the host: `traceroute <address>`
- Show the routing table: `route or ip route`
- - Check open ports and network connections: `netstat -tuln or ss -tuln`
- Check the active network processes: `lsof -i`
- Find out the MAC address of the interface: `ip link show <interface>`
- Enable or disable network interface: `ip link set <interface> up/down`
- Get information about DNS servers: `cat /etc/resolv.conf`
- Perform a network scan (requires nmap installation): `nmap <address/subnet>`
