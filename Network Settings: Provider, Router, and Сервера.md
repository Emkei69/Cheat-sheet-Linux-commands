The preface  
  
(This manual describes in detail the correct and consistent configuration of the integrated network circuit. Configurations for various network components are presented here, from the provider and the main routers to servers and clients. Special attention is paid to interface settings, NAT, routing, GRE tunnels and the OSPF protocol, as well as DNS and DHCP services. This guide will help system administrators and engineers set up and ensure stable and secure operation of the corporate network)  
  
==========NetProvider==========  
  
-------host name---------  
(Setting the system hostname)  
```hostnamectl hostname NetProvider```  
```exec bash```  
  
-------setting up INTERNAL interfaces---------  
(Creating directories and configuration files for internal network interfaces, specifying the interface type and IP addresses)  
```mkdir -p /etc/net/ifaces/ens19```  
```mkdir -p /etc/net/ifaces/ens20```  
  
```echo 'TYPE=eth' > /etc/net/ifaces/ens19/options```  
```echo 'TYPE=eth' > /etc/net/ifaces/ens20/options```  
  
```echo '172.16.4.1/28' > /etc/net/ifaces/ens19/ipv4address```  
```echo '172.16.5.1/28' > /etc/net/ifaces/ens20/ipv4address```  
  
-------setting up NAT---------
( Installing and configuring nftables for NAT (masquerading) on the external interface)  
```apt-get update && apt-get install nftables -y```  
```vim /etc/nftables/nftables.nft```  
  
```  
#!/usr/sbin/nft -f  
  
flush ruleset  
  
table ip nat {  
  
chain postrouting {  
  
type nat hook postrouting priority srcnat;  
  
oifname "ens18" masquerade  
  
}  
  
}  
```  
  
```systemctl enable --now nftables```  
  
-------enabling routing---------  
(Enabling IP routing in the Linux kernel and restarting the network service)  
```echo net.ipv4.ip_forward=1 >> /etc/net/sysctl.conf```  
```systemctl restart network```  
```sysctl -p```  
```sysctl net.ipv4.ip_forward```  
  
==========MainRouter==========  
  
-------host name---------  
(Switching to configuration mode and setting the router's hostname and domain name)  
```ecorouter>en```  
```ecorouter#configure terminal```  
```ecorouter(config)#hostname MainRouter.au-team.irpo```  
```MainRouter.au-team.irpo(config)#ip domain-name au-team.irpo```  
  
-------setting up NTP/DNS---------  
(Configuring DNS server and NTP time zone)  
```(config)#ip name-server 192.168.1.10```  
```(config)#ntp timezone utc+3```  
  
-------configuring the net_admin connection---------  
(Creating user accounts)  
(Creating a net_admin user with the administrator role)
``(config)#username net_admin``  
```(config-user)#password P@$word```  
```(config-user)#role admin```  
```(config-user)#exit```  
  
-------the DHCP pool---------  
(Creating a pool of IP addresses for DHCP with a range)
``(config)#ip pool dhcp 1``  
```(config-ip-pool)# range 192.168.2.2-192.168.2.9```  
```(config-ip-pool)#exit```  
  
-------configuring the parameters of the DHCP server---------  
(Configuring the DHCP server)  
(Configuration of the DHCP parameters: mask, gateway, DNS, domain name and static IP for the client)
``(config)#dhcp-server 1```  
```(config-dhcp-server)#pool dhcp 64```  
```(config-dhcp-server-pool)#mask 255.255.255.240```  
```(config-dhcp-server-pool)#gateway 192.168.2.1```  
```(config-dhcp-server-pool)#dns 192.168.1.10```  
```(config-dhcp-server-pool)#domain-name au-team.irpo```  
```(config-dhcp-server-pool)#static ip 192.168.2.10```  
```(config-dhcp-server-static)#client-id mac 1111.1111.1111```  
```(config-dhcp-server-static)#exit```  
```(config-dhcp-server)#exit```  
  
-------configuring the external interface-------  
(Service Instance)  
(Port configuration and service-instance creation with untagged encapsulation)  
```(config)#port te0```  
```(config-port)#service-instance te0/netprovider-hq```  
```(config-service-instance)#encapsulation untagged```  
```(config-service-instance)#exit```  
```(config-port)#exit```  
  
-------setting up a Service Instance---------  
(Creating multiple service instances with different VLAN tags and configuring tag rewriting)  
```(config)#port te1```  
```(config-port)#service-instance te1/srv-net```  
```(config-service-instance)#encapsulation dot1q 100```  
```(config-service-instance)#rewrite pop 1```  
```(config-service-instance)#service-instance te1/cli-net```  
```(config-service-instance)#encapsulation dot1q 200```  
```(config-service-instance)#rewrite pop 1```  
```(config-service-instance)#service-instance te1/management```  
```(config-service-instance)#encapsulation dot1q 999```  
```(config-service-instance)#rewrite pop 1```  
```(config-service-instance)#exit```  
```(config-port)#exit```  
  
-------configuring interface addressing-------------  
(Assigning IP addresses to interfaces, configuring NAT inside/outside, and linking to a service instance)  
```(config)#interface netprovider-hq```  
```(config-if)#ip nat outside```  
```(config-if)#ip address 172.16.4.4/28```  
```(config-if)#connect port te0 service-instance te0/netprovider-hq```  
```(config-if)#exit```  
  
```(config)#interface srv-net```  
```(config-if)#ip nat inside```  
```(config-if)#ip address 192.168.1.1/26```  
```(config-if)#connect port te1 service-instance te1/srv-net```  
```(config-if)#exit```  
  
```(config)#interface cli-net```  
```(config-if)#ip nat inside```  
```(config-if)#ip address 192.168.2.1/28```  
```(config-if)#connect port te1/service-instance te1/cli-net```  
```(config-if)#exit```  
  
```(config)#interface management```  
```(config-if)#ip address 192.168.99.1/29```  
```(config-if)#connect port te1/service-instance te1/management```  
```(config-if)#exit```  
  
-------setting up NAT---------  
(Creating a NAT pool and configuring dynamic NAT with congestion via an external interface)  
```(config)#ip nat pool nat 192.168.1.1-192.168.2.254,192.168.99.1-192.168.99.254```  
```(config)#ip nat source dynamic inside-to-outside pool nat overload interface netprovider-hq```  
  
-------default route setting---------  
(Adding a default route through the provider's gateway)
``(config)#ip route 0.0.0.0/0 172.16.4.1 description default``  
  
-------setting up the GRE tunnel---------  
(Creating a GRE tunnel with IP addressing and setting up OSPF authentication on the tunnel)  
```(config)#interface tunnel.1```  
```(config-if-tunnel)#ip add 192.168.5.1/30```  
```(config-if-tunnel)#ip tunnel 172.16.4.4 172.16.5.5 mode gre```  
```(config-if-tunnel)#ip ospf authentication```  
```(config-if-tunnel)#ip ospf authentication-key P@$word```  
```(config-if-tunnel)#exit```  
  
-------setting up OSPF---------  
(Configuration of the OSPF protocol with indication of networks, passive interfaces and tunnel activation)  
```(config)#router ospf 1```  
```(config-router)#network 192.168.5.0/30 area 0```  
```(config-router)#network 192.168.1.0/26 area 0```  
```(config-router)#network 192.168.2.0/28 area 0```  
```(config-router)#network 192.168.99.0/29 area 0```  
```(config-router)#passive-interface default```  
```(config-router)#no passive-interface tunnel.1```  
```(config-router)#exit```  
```(config)#exit```  
  
-------recording the configuration---------
( Saving the router configuration)  
```MainRouter.au-team.irpo#write```  
  
-------verification commands---------  
(View brief information about interfaces, ports, routes, and current configuration)  
```MainRouter.au-team.irpo#show ip interface brief```  
```MainRouter.au-team.irpo#show port brief```  
```MainRouter.au-team.irpo#sh ip route```  
```MainRouter.au-team.irpo#sh running-config```  
  
==========BranchRouter==========  
  
-------host name---------
( Setting the hostname and domain name of the branch router)  
```ecorouter>en```  
```ecorouter#configure terminal```  
```ecorouter(config)#hostname BranchRouter.au-team.irpo```  
```BranchRouter.au-team.irpo(config)#ip domain-name au-team.irpo```  
  
-------setting up NTP/DNS---------  
(Configuring DNS server and NTP time zone)  
```(config)#ip name-server 192.168.1.10```  
```(config)#ntp timezone utc+3```  
  
-------configuring the net_admin connection---------  
(Creating user accounts)  
(Creating a net_admin user with an administrator role)  
```(config)#username net_admin```  
```(config-user)#password P@$word```  
```(config-user)#role admin```  
```(config-user)#exit```  
  
-------setting up a Service Instance-------  
(Creating a service-instance for ports with untagged encapsulation)  
```(config)#port te0```  
```(config-port)#service-instance te0/netprovider-br```  
```(config-service-instance)#encapsulation untagged```  
```(config-service-instance)#exit```  
```(config-port)#exit```  
  
```(config)#port te1```  
```(config-port)#service-instance te1/branchrouter-net```  
```(config-service-instance)#encapsulation untagged```  
```(config-service-instance)#exit```  
```(config-port)#exit```  
  
-------configuring interface addressing-------------  
(Configuring IP addresses of interfaces, NAT inside/outside, and binding to a service instance)  
```(config)#interface netprovider-br```  
```(config-if)#ip nat outside```  
```(config-if)#ip address 172.16.5.5/28```  
```(config-if)#connect port te0 service-instance te0/netprovider-br```  
```(config-if)#exit```  
  
```(config)#interface branchrouter-net```  
```(config-if)# ip nat inside```  
```(config-if)#ip address 192.168.3.1/27```  
```(config-if)# connect port te1 service-instance te1/branchrouter-net```  
  
-------setting up NAT---------  
(Creating a NAT pool and configuring dynamic NAT with congestion via an external interface)  
```(config)#ip nat pool nat 192.168.3.1-192.168.3.254```  
```(config)#ip nat source dynamic inside-to-outside pool nat overload interface netprovider-br```  
  
-------default route setting---------  
(Adding a default route through the gateway of the branch provider)
```(config)#ip route 0.0.0.0/0 172.16.5.1 description default```  
  
-------setting up the GRE tunnel---------  
(Creating a GRE tunnel with IP addressing and setting up OSPF authentication on the tunnel)  
```(config)#interface tunnel.1```  
```(config-if-tunnel)#ip add 192.168.5.2/30```  
```(config-if-tunnel)#ip tunnel 172.16.5.5 172.16.4.4 mode gre```  
```(config-if-tunnel)#ip ospf authentication```  
```(config-if-tunnel)#ip ospf authentication-key P@$word```  
```(config-if-tunnel)#exit```  
  
-------setting up OSPF---------  
(Configuration of the OSPF protocol with indication of networks, passive interfaces and tunnel activation)  
```(config)#router ospf 1```  
```(config-router)#network 192.168.5.0/30 area 0```  
```(config-router)#network 192.168.3.0/27 area 0```  
```(config-router)#passive-interface default```  
```(config-router)#no passive-interface tunnel.1```  
```(config-router)#exit```  
```(config)#exit```  
  
-------verification commands---------
( Save configuration and view the status of interfaces, ports, routes, and current configuration)  
```MainRouter.au-team.irpo#write```  
```MainRouter.au-team.irpo#show ip interface brief```  
```MainRouter.au-team.irpo#show port brief```  
```MainRouter.au-team.irpo#sh ip route```  
```MainRouter.au-team.irpo#sh running-config```  
  
==========MainServer==========  
  
-------host name/NTP---------
( Setting the server's hostname and time zone)  
```hostnamectl hostname MainServer.au-team.irpo```  
```exec bash```  
```timedatectl set-timezone Europe/Moscow```  
  
-------configuring interfaces-------------  
(Network interface configuration with IP address, default route, and DNS)  
```echo 'TYPE=eth' > /etc/net/ifaces/ens19/options```  
```echo '192.168.1.10/26' > /etc/net/ifaces/ens19/ipv4address```  
```echo 'default via 192.168.1.1' > /etc/net/ifaces/ens19/ipv4route```  
```echo 'nameserver 1.1.1.1' > /etc/net/ifaces/ens19/resolv.conf```  
```systemctl restart network```  
  
-------DnsMasq-------------
( Installing and configuring dnsmasq for local DNS and DHCP caching with records and aliases)  
```apt-get update && apt-get install dnsmasq -y```  
```echo 'OPTIONS=""' > /etc/sysconfig/dnsmasq```  
  
```vim /etc/dnsmasq.conf```  
  
```  
no-resolv  
no-poll  
no-hosts  
  
server=8.8.8.8  
cache-size=1000  
all-servers  
no-negcache  
  
address=/mainrouter.au-team.irpo/192.168.1.1  
address=/branchrouter.au-team.irpo/192.168.3.1  
address=/mainserver.au-team.irpo/192.168.1.10  
address=/mainclient.au-team.irpo/192.168.2.10  
address=/branchserver.au-team.irpo/192.168.3.10  
  
host-record=mainrouter.au-team.irpo,192.168.1.1  
host-record=mainserver.au-team.irpo,192.168.1.10  
host-record=mainclient.au-team.irpo,192.168.2.10  
  
cname=moodle.au-team.irpo,mainrouter.au-team.irpo  
cname=wiki.au-team.irpo,mainrouter.au-team.irpo  
```  
  
```systemctl enable --now dnsmasq```  
```echo 'nameserver 127.0.0.1' > /etc/net/ifaces/ens19/resolv.conf```  
```service network restart```  
  
-------setting up sshuser-------------  
(Creating an sshuser user with uid 1010 and adding sudo rights to the wheel group)  
```useradd -u 1010 sshuser```  
```passwd sshuser```  
```usermod -aG wheel sshuser```  
  
==========BranchServer==========  
  
-------host name/NTP---------
( Setting the hostname and time zone of the branch server)  
```hostnamectl hostname BranchServer.au-team.irpo```  
```exec bash```  
```timedatectl set-timezone Europe/Moscow```  
  
-------configuring interfaces-------------  
(Network interface configuration with IP address, default route, and DNS)  
```echo 'TYPE=eth' > /etc/net/ifaces/ens18/options```  
```echo '192.168.3.10/27' > /etc/net/ifaces/ens18/ipv4address```  
```echo 'default via 192.168.3.1' > /etc/net/ifaces/ens18/ipv4route```  
```echo 'nameserver 192.168.1.10' > /etc/net/ifaces/ens18/resolv.conf```  
```service network restart```  
  
-------setting up sshuser-------------  
(Creating an sshuser user, setting a password, adding to wheel, and configuring sudo without a password)  
```useradd -u 1010 sshuser```  
```passwd sshuser```  
```usermod -aG wheel sshuser```  
```vim /etc/sudoers```  
(Add a line: WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL)  
  
--------SSH setup----------------  
(SSH server configuration: non-standard port, sshuser access only, limited authentication attempts, and banner)  
```echo "Authorized access only" > /etc/openssh/banner```  
(In the /etc/openssh/sshd_config file)  
  
```  
Port 2224  
AllowUsers sshuser  
MaxAuthTries 2  
Banner /etc/openssh/banner  
```  
  
```systemctl restart sshd```  
  
==========MainClient==========  
  
-------host name/NTP---------
( Setting the client's hostname and time zone, configuring the DHCP client-id via nmcli)  
```hostnamectl hostname MainClient.au-team.irpo```  
```exec bash```  
```timedatectl set-timezone Europe/Moscow```  
  
```nmcli connection modify 'Wired connection 1' ipv4.dhcp-client-id 01:11:11:11:11:11:11```  
```nmcli connection show```  
  
-------configuring the EXTERNAL interface---------  
(Renaming the external interface configuration directory)  
```mv /etc/net/ifaces/ens18/ /etc/net/ifaces/ens19/```  
  
-------setting up INTERNAL interfaces---------  
(Creation of directories and configuration files for internal interfaces with IP address assignment)  
```mkdir -p /etc/net/ifaces/ens2{0,1}```  
```echo 'TYPE=eth' | tee /etc/net/ifaces/ens2{0,1}/options```  
```echo '172.16.4.1/28' > /etc/net/ifaces/ens20/ipv4address```  
```echo '172.16.5.1/28' > /etc/net/ifaces/ens21/ipv4address```  
```systemctl restart network```  
  
-------setting up NAT---------
( Installing and configuring nftables for NAT (masquerading) on the external interface)  
```apt-get update && apt-get install nftables -y```  
```vim /etc/nftables/nftables.nft```  
  
```  
#!/usr/sbin/nft -f  
flush ruleset  
  
table ip nat {  
chain postrouting {  
type nat hook postrouting priority srcnat;  
oifname "ens19" masquerade  
}  
}  
```  
  
```systemctl enable --now nftables```  
  
-------enabling routing---------  
(Enabling IP routing in the kernel and applying settings)  
```echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf```  
```sysctl -p```
