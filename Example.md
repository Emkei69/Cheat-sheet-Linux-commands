==========NetProvider==========  

-------имя хоста---------  
``` hostnamectl hostname NetProvider ```  
``` exec bash ```  

-------настройка ВНУТРЕННИХ интерфейсов---------  
``` mkdir -p /etc/net/ifaces/ens19 ```  
``` mkdir -p /etc/net/ifaces/ens20 ```  

``` echo 'TYPE=eth' > /etc/net/ifaces/ens19/options ```  
``` echo 'TYPE=eth' > /etc/net/ifaces/ens20/options ```  

``` echo '172.16.4.1/28' > /etc/net/ifaces/ens19/ipv4address ```  
``` echo '172.16.5.1/28' > /etc/net/ifaces/ens20/ipv4address ```  

-------настройка NAT---------  
``` apt-get update && apt-get install nftables -y ```  
``` vim /etc/nftables/nftables.nft ```  

```
		-------nftables.nft------- 
 
#!/usr/sbin/nft -f   

 flush ruleset  

 table ip nat {   
 chain postrouting {   
 type nat hook postrouting priority srcnat;   
 oifname "ens18" masquerade   
 } 
} 

		----------------------------
```

(https://wiki.archlinux.org/title/Nftables_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)#%D0%9C%D0%B0%D1%81%D0%BA%D0%B0%D1%80%D0%B0%D0%B4%D0%B8%D0%BD%D0%B3)  

``` systemctl enable --now nftables ```  

-------включение маршрутизации---------  
``` echo net.ipv4.ip_forward=1 >> /etc/net/sysctl.conf ```  
``` systemctl restart network ```  

``` sysctl -p ```  
``` sysctl net.ipv4.ip_forward ```  

(https://www.altlinux.org/Static_Multicast_Routing#%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0_%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8)  

==========MainRouter==========  

-------имя хоста---------  
``` ecorouter>en ```  
``` ecorouter#configure terminal ```  
``` ecorouter(config)#hostname MainRouter.au-team.irpo ```  
``` MainRouter.au-team.irpo(config)#ip domain-name au-team.irpo ```  

-------настройка NTP/DNS---------  
``` (config)#ip name-server 192.168.1.10 ```  
``` (config)#ntp timezone utc+3 ```  

-------настройка УЗ net_admin---------3.3 Создание учетных записей пользователей  
``` (config)#username net_admin ```  
``` (config-user)#password P@$word  ```  
``` (config-user)#role admin ```  
``` (config-user)#exit ```  

-------пул DHCP---------  
``` (config)#ip pool dhcp 1 ```  
``` (config-ip-pool)# range 192.168.2.2-192.168.2.9 ```  
``` (config-ip-pool)#exit ```  

-------настройка параметров DHCP сервера---------8.3 Настройка DHCP-сервера  
``` (config)#dhcp-server 1 ```  
``` (config-dhcp-server)#pool dhcp 64 ```  
``` (config-dhcp-server-pool)#mask 255.255.255.240 ```  
``` (config-dhcp-server-pool)#gateway 192.168.2.1 ```  
``` (config-dhcp-server-pool)#dns 192.168.1.10 ```  
``` (config-dhcp-server-pool)#domain-name au-team.irpo ```  
``` (config-dhcp-server-pool)#static ip 192.168.2.10 ```  
``` (config-dhcp-server-static)#client-id mac 1111.1111.1111 ```  
``` (config-dhcp-server-static)#exit ```  
``` (config-dhcp-server)#exit ```  

-------настройка внешнего интерфейса-------4.9 Service Instance  
``` (config)#port te0 ```  
``` (config-port)#service-instance te0/netprovider-hq ```  
``` (config-service-instance)#encapsulation untagged ```  
``` (config-service-instance)#exit ```  
``` (config-port)#exit ```  

-------настройка Service Instance---------  
``` (config)#port te1 ```  
``` (config-port)#service-instance te1/srv-net ```  
``` (config-service-instance)#encapsulation dot1q 100 ```  
``` (config-service-instance)#rewrite pop 1 ```  
``` (config-service-instance)#service-instance te1/cli-net ```  
``` (config-service-instance)#encapsulation dot1q 200 ```  
``` (config-service-instance)#rewrite pop 1 ```  
``` (config-service-instance)#service-instance te1/management ```  
``` (config-service-instance)#encapsulation dot1q 999 ```  
``` (config-service-instance)#rewrite pop 1 ```  
``` (config-service-instance)#exit ```  
``` (config-port)#exit ```  

-------настройка адресации интерфейсов-------------  
``` (config)#interface netprovider-hq ```  
``` (config-if)#ip nat outside ```  
``` (config-if)#ip address 172.16.4.4/28 ```  
``` (config-if)#connect port te0 service-instance te0/netprovider-hq ```  
``` (config-if)#exit ```  

``` (config)#interface srv-net ```  
``` (config-if)#ip nat inside ```  
``` (config-if)#ip address 192.168.1.1/26 ```  
``` (config-if)#connect port te1 service-instance te1/srv-net ```  
``` (config-if)#exit ```  
	
``` (config)#interface cli-net ```  
``` (config-if)#ip nat inside ```  
``` (config-if)#ip address 192.168.2.1/28 ```  
``` (config-if)#connect port te1/service-instance te1/cli-net ```  
``` (config-if)#exit ```  

``` (config)#interface management ```  
``` (config-if)#ip address 192.168.99.1/29 ```  
``` (config-if)#connect port te1/service-instance te1/management ```  
``` (config-if)#exit ```  
``` (config)# ```  

-------настройка NAT---------28 Встроенный NAT  
``` (config)#ip nat pool nat 192.168.1.1-192.168.2.254,192.168.99.1-192.168.99.254 ```  
``` (config)#ip nat source dynamic inside-to-outside pool nat overload interface netprovider-hq ```  

-------настройка маршрута по умолчанию---------13.4.4 Маршрут по умолчанию  
``` (config)#ip route 0.0.0.0/0 172.16.4.1 description default ```  

-------настройка туннеля GRE---------15.1 GRE  
``` (config)#interface tunnel.1 ```  
``` (config-if-tunnel)#ip add 192.168.5.1/30 ```  
``` (config-if-tunnel)#ip tunnel 172.16.4.4 172.16.5.5 mode gre ```  
``` (config-if-tunnel)#ip ospf authentication ```  
``` (config-if-tunnel)#ip ospf authentication-key P@$word ```  
``` (config-if-tunnel)#exit ```  

-------настройка OSPF---------13.4 Настройка OSPF  
``` (config)#router ospf 1 ```  
``` (config-router)#network 192.168.5.0/30 area 0 ```  
``` (config-router)#network 192.168.1.0/26 area 0 ```  
``` (config-router)#network 192.168.2.0/28 area 0 ```  
``` (config-router)#network 192.168.99.0/29 area 0 ```  
``` (config-router)#passive-interface default ```  
``` (config-router)#no passive-interface tunnel.1 ```  
``` (config-router)#exit ```  
``` (config)#exit ```  

-------запись конфигурации---------  
``` MainRouter.au-team.irpo#write  ```  

-------команды проверки---------  
``` MainRouter.au-team.irpo#show ip interface brief ```  
``` MainRouter.au-team.irpo#show port brief  ```  
``` MainRouter.au-team.irpo#sh ip route ```  
``` MainRouter.au-team.irpo#sh running-config  ```  

==========BranchRouter==========  

-------имя хоста---------  
``` ecorouter>en ```  
``` ecorouter#configure terminal ```  
``` ecorouter(config)#hostname BranchRouter.au-team.irpo ```  
``` BranchRouter.au-team.irpo(config)#ip domain-name au-team.irpo ```  

-------настройка NTP/DNS---------  
``` (config)#ip name-server 192.168.1.10 ```  
``` (config)#ntp timezone utc+3 ```  

-------настройка УЗ net_admin---------3.3 Создание учетных записей пользователей  
``` (config)#username net_admin ```  
``` (config-user)#password P@$word  ```  
``` (config-user)#role admin ```  
``` (config-user)#exit ```  

-------настройка Service Instance-------4.9 Service Instance  
``` (config)#port te0 ```  
``` (config-port)#service-instance te0/netprovider-br ```  
``` (config-service-instance)#encapsulation untagged  ```  
``` (config-service-instance)#exit ```  
``` (config-port)#exit ```  

``` (config)#port te1 ```  
``` (config-port)#service-instance te1/br-net ```  
``` (config-service-instance)#encapsulation untagged  ```  
``` (config-service-instance)#exit ```  
``` (config-port)#exit ```  

-------настройка адресации интерфейсов-------------  
``` (config)#interface netprovider-br ```  
``` (config-if)#ip nat outside ```  
``` (config-if)#ip address 172.16.5.5/28 ```  
``` (config-if)#connect port te0 service-instance te0/netprovider-br ```  
``` (config-if)#exit ```  

``` (config)#interface br-net ```  
``` (config-if)# ip nat inside ```  
``` (config-if)#ip address 192.168.3.1/27 ```  
``` (config-if)# connect port te1 service-instance te1/br-net ```  

-------настройка NAT---------28 Встроенный NAT  
``` (config)#ip nat pool nat 192.168.3.1-192.168.3.254 ```  
``` (config)#ip nat source dynamic inside-to-outside pool nat overload interface netprovider-br ```  

-------настройка маршрута по умолчанию---------13.4.4 Маршрут по умолчанию  
``` (config)#ip route 0.0.0.0/0 172.16.5.1 description default ```  

-------настройка туннеля GRE---------15.1 GRE  
``` (config)#interface tunnel.1 ```  
``` (config-if-tunnel)#ip add 192.168.5.2/30 ```  
``` (config-if-tunnel)#ip tunnel 172.16.5.5 172.16.4.4 mode gre ```  
``` (config-if-tunnel)#ip ospf authentication ```  
``` (config-if-tunnel)#ip ospf authentication-key P@$word ```  
``` (config-if-tunnel)#exit ```  

-------настройка OSPF---------13.4 Настройка OSPF  
``` (config)#router ospf 1 ```  
``` (config-router)#network 192.168.5.0/30 area 0 ```  
``` (config-router)#network 192.168.3.0/27 area 0 ```  
``` (config-router)#passive-interface default ```  
``` (config-router)#no passive-interface tunnel.1 ```  
``` (config-router)#exit ```  
``` (config)#exit ```  

-------команды проверки---------  
``` MainRouter.au-team.irpo#write ```  
``` MainRouter.au-team.irpo#show ip interface brief ```  
``` MainRouter.au-team.irpo#show port brief ```  
``` MainRouter.au-team.irpo#sh ip route ```  
``` MainRouter.au-team.irpo#sh running-config ```  

==========MainServer==========  

-------имя хоста/NTP---------  
``` hostnamectl hostname MainServer.au-team.irpo ```  
``` exec bash ```  
``` timedatectl set-timezone Europe/Moscow ```  

-------настройка интерфейсов-------------  
``` echo 'TYPE=eth' > /etc/net/ifaces/ens19/options ```  
``` echo '192.168.1.10/26' > /etc/net/ifaces/ens19/ipv4address ```  
``` echo 'default via 192.168.1.1' > /etc/net/ifaces/ens19/ipv4route ```  
``` echo 'nameserver 1.1.1.1' > /etc/net/ifaces/ens19/resolv.conf ```  
``` systemctl restart network ```  

-------DnsMasq-------------  
``` https://dnsmasq.org/docs/dnsmasq-man.html ```  

``` apt-get update && apt-get install dnsmasq -y ```  
``` echo 'OPTIONS=""' > /etc/sysconfig/dnsmasq ```  

``` vim /etc/dnsmasq.conf ```  

``` +++++dnsmasq.conf ```  

``` no-resolv ```  
``` no-poll ```  
``` no-hosts ```  

``` server=8.8.8.8 ```  
``` cache-size=1000 ```  
``` all-servers ```  
``` no-negcache ```  

``` address=/mainrouter.au-team.irpo/192.168.1.1 ```  
``` address=/branchrouter.au-team.irpo/192.168.3.1 ```  
``` address=/mainserver.au-team.irpo/192.168.1.10 ```  
``` address=/mainclient.au-team.irpo/192.168.2.10 ```  
``` address=/branchserver.au-team.irpo/192.168.3.10 ```  

``` host-record=mainrouter.au-team.irpo,192.168.1.1 ```  
``` host-record=mainserver.au-team.irpo,192.168.1.10 ```  
``` host-record=mainclient.au-team.irpo,192.168.2.10 ```  

``` cname=moodle.au-team.irpo,mainrouter.au-team.irpo ```  
``` cname=wiki.au-team.irpo,mainrouter.au-team.irpo ```  

``` +++++++ ```  

``` systemctl enable --now dnsmasq ```  
``` echo 'nameserver 127.0.0.1' > /etc/net/ifaces/ens19/resolv.conf ```  
``` service network restart ```  

-------настройка sshuser-------------  
``` useradd -u 1010 sshuser ```  
``` passwd sshuser ```  
``` usermod -aG wheel ```

==========BranchServer==========  

-------имя хоста/NTP---------  
``` hostnamectl hostname BranchServer.au-team.irpo ```  
``` exec bash ```  
``` timedatectl set-timezone Europe/Moscow ```  

-------настройка интерфейсов-------------  
``` echo 'TYPE=eth' > /etc/net/ifaces/ens18/options ```  
``` echo '192.168.3.10/27' > /etc/net/ifaces/ens18/ipv4address ```  
``` echo 'default via 192.168.3.1' > /etc/net/ifaces/ens18/ipv4route ```  
``` echo 'nameserver 192.168.1.10' > /etc/net/ifaces/ens18/resolv.conf ```  
``` service network restart ```  

-------настройка sshuser-------------  
``` useradd -u 1010 sshuser ```  
``` passwd sshuser ```  
``` usermod -aG wheel sshuser ```  
``` vim /etc/sudoers ```  
``` WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL ```  

--------настройка SSH----------------  
``` https://www.altlinux.org/%D0%94%D0%BE%D1%81%D1%82%D1%83%D0%BF_%D0%BF%D0%BE_SSH#%D0%94%D0%BE%D1%81%D1%82%D1%83%D0%BF_%D0%BF%D0%BE_SSH_%D1%87%D0%B5%D1%80%D0%B5%D0%B7_%D1%81%D0%B5%D1%82%D1%8C ```  

``` echo "Authorized access only" > /etc/openssh/banner ```  
``` в файле /etc/openssh/sshd_config ```  

``` +++++sshd_config++++++++++ ```  

``` Задать "нестандартный" порт ```  
``` Port 2224 ```  

``` Разрешите подключения только пользователю sshuser ```  
``` AllowUsers sshuser ```  

``` Ограничьте ввод попыток до 2 ```  
``` MaxAuthTries 6 --> 2 ```  

``` Баннер ```  
``` #Banner none ---> Banner /etc/openssh/banner ```  

``` +++++++ ```  
``` systemctl restart sshd ```  

==========MainClient==========  

-------имя хоста/NTP---------  
``` hostnamectl hostname MainClient.au-team.irpo ```  
``` exec bash ```  
``` timedatectl set-timezone Europe/Moscow ```  

``` https://networkmanager.dev/docs/api/latest/nm-settings-nmcli.html ```  

``` nmcli connection modify 'Wired connection 1' ipv4.dhcp-client-id 01:11:11:11:11:11:11 ```  

``` nmcli connection show ```  

-------настройка ВНЕШНЕГО интерфейса---------  
``` mv /etc/net/ifaces/ens18/ /etc/net/ifaces/ens19/ ```  

-------настройка ВНУТРЕННИХ интерфейсов---------  
``` mkdir -p /etc/net/ifaces/ens2{0,1} ```  
``` echo 'TYPE=eth' | tee /etc/net/ifaces/ens2{0,1}/options ```  
``` echo '172.16.4.1/28' > /etc/net/ifaces/ens20/ipv4address ```  
``` echo '172.16.5.1/28' > /etc/net/ifaces/ens21/ipv4address ```  
``` systemctl restart network ```  

-------настройка NAT---------  
``` apt-get update && apt-get install nftables -y ```  
``` vim /etc/nftables/nftables.nft ```  

``` #!/usr/sbin/nft -f ```  
``` flush ruleset ```  

``` table ip nat { ```  
```   chain postrouting { ```  
```     type nat hook postrouting priority srcnat; ```  
```     oifname "ens19" masquerade ```  
```   } ```  
``` } ```  

``` systemctl enable --now nftables ```  

-------включение маршрутизации---------  
``` echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf ```  
``` sysctl -p ```  
