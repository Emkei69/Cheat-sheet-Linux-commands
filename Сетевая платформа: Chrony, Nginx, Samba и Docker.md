========== Network provider==========  
  
---------Server Setup to Change Time to Other Network Provider---------  
  
---Customize the time to a different---  
  
```apt-get update && apt-get install chrony nginx``.  
  
---Make changes to /etc/chrony.conf---  
```vim /etc/chrony.conf```  
  
```  
~~~  
######  
server ntp1.vniiftri.ru I'm
run local level 5
Allow everything.    
######  
#Record the speed at which the system.... <--- (To limit the desired speed)  
~~~  
```  
  
---Where---  
| Configuration | Description |
|----------------------------------|-------------------------------|
| ```server ntp1.vniiftri.ru iburst``` | local change server |
| ```local layer 5``` | local strat | local strat |
| “Allow all” | change for all terms |

  
---Turn on chronicle and set it to autoload---.  
``-systemctl enable---now chronid''.  
  
---It may be necessary to install chronid, and write an article in chronid.conf--- ``-  
```On the master server, master client:``  
```Server 172.16.4.1 loaded```  
  
```on branch router, branch server:``  
```Server 172.16.5.1 is loaded```  
    
---------Install the nginx web server as a shared browser-server---------  

Translated with DeepL.com (free version)

---Configure Nginx sharing---.  

```apt-get install nginx''.  
  
---Create a new proxy configuration file---  
``/etc/nginx/sites-available.d/proxy.conf``.  

```  
~~~  
server {
listen 80;
server_name moodle.au-team.irpo;
location / {
proxy_pass http://192.168.1.10:80 ;
proxy_set_header Host $host;
proxy_set_header X-Real IP $remote address;  
proxy_set_header X-Forwarded-To $remote_addr;  
}  
}  
  
server {  
listening 80;  
server_name wiki.au-team.irpo;
location / {  
прокси-сервер_pass http://192.168.3.10:8080;  
proxy_set_header Host $host;
proxy_set_header X-Real IP $remote address;  
proxy_set_header X-Redirected-To $remote_addr;    
}   
}  
~~~  
```  
  
---Include information by us (proxy.conf), possibly links---.  
``` ```ln /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/proxy.conf```  
```ls -la /etc/nginx/sites-enabled``.  
  
---Check the configuration for errors---  
  
```nginx -t```  

---Now connect the nginx service---  

```systemctl enable---now nginx``.  

---------Branch server ---------  

---------Configure a separate Samba controller on the new branch server---------  
  
---SambaAD to alternate---.    
  
---thank you /etc/samba/smb.conf FOR WHAT YOU ARE!!!---  
  
```apt-get update && apt-get install-samba-dc -y``` ```  
  
```rm -f /etc/samba/smb.conf```  
  
---Start the installation---  
  
```-provide samba-tool-domain-interactive'' 

Translated with DeepL.com (free version)

---------Performing an availability check---------

```ansible all -m ping""

---------Creating applications in Docker on a branch server---------

---------Install the docker engine and docker-compose---------

```apt-get installs the docker engine docker-compose -y```

---------enabling the docker service---------

```enable systemctl -now docker```

---------we provide an opportunity to combine users into a docker group so that it can work with containers---------

```usermod sshuser -aG configuration tool```
```grep /etc/group configuration tool'

---------We go into the context of the sshuser user---------

```su -l sshuser```

---------Upload images with the following command---------

```docker extracts mediawiki```
```docker extracts mariadb```

---------Creating a wiki.yml for the MediaWiki application in the user's home directory.---------

```vim wiki.yml```  

~~~  
```  

wiki services:  
 image: mediawiki_name of the
container: wiki
-Wednesday:
MEDIAWIKI_DB_HOST: mariadb  
 MEDIAWIKI_DB_USER: wiki  
 MEDIAWIKI PASSWORD: WikiP@ssw0rd  
 MEDIAWIKI_DB NAME:
mediawiki ports:
-8080:80  
 # volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]  
 database:
image: mariadb container name
: mariadb
host name:
mariadb environment
: MYSQL_DATABASE: mariadb  
 MYSQL_ USER: wiki  
 MYSQL_PASSWORD: volumes Wikipedia@ssw0rd
:
- ./db:volumes/var/lib/mysql
:
dB:  
```  
~~~  

---------Checking the configuration---------

```configuration of docker-compose -f wiki.yml```

---after checking, we launch---

```docker-compose -f wiki.yml up -d```

![image](https://github.com/user-attachments/assets/7139c3c2-ba46-4a0c-8240-adf657168f79 )

---------!We go to the main client, we look at him ```http://192.168.3.10:8080 `---------  

![image](https://github.com/user-attachments/assets/b98a70d4-3f7d-4fe1-b3fb-0233c1f35e37 )  

Fill in the fields, Then   

![image](https://github.com/user-attachments/assets/700808e9-6cba-4e79-82df-ce89a421feac )


```scp LocalSettings.php sshuser@192.168.3.10:/home/sshuser/``  

```the cat LocalSettings.php ```  

---------After that, we stop all containers---------  

```stopping docker$(docker ps -a -q)``  

---------delete all containers---------  

```docker rm$(docker ps -a -q)`

---------Commenting on the wiki.yml page---------  

``` - 8080:80```  
``` volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]`  

--------- I just downloaded wiki.yml---------  

```docker-compose -f wiki.yml up -d```  
```On the main server, in the browser at http://192.168.3.10:8080 ```  

![image](https://github.com/user-attachments/assets/c3403324-ee47-411e-b8c5-e0d552a59238 )

```Wiki works```  

========== The main server==========  

---------Install moodle on the main server---------  
---We are installing a number of packages that we will need to work with---

```apt-get the update && apt-Get the installation of apache2 php8.2 apache2-mod_php8.2 mariadb-Server php8.2-opcache php8.2-curl php8.2-gd php8.2-International php8.2-mysqli php8.2-xml php8.2-xmlrpc php8.2-ldap php8.2-zip php8.2-soap php8.2-mbstring php8.2-json php8.2-xml-reader php8.2-fileinfo php8.2-sodium```  

---Connecting httpd2 and mysqld services---  

```enable systemctl - now httpd2 mysqld```  

---Set up secure access to our future database using the command---  

```mysql_secure_installation```  

--- I'm sure you're right, I do the rest of the time.---  
---go to the DBMS to create and configure the database---  

```mariadb -u root -p```  

```MariaDB [(no)]> CREATE A moodledb DATABASE;`  
```The request is OK, 1 line is affected (0.002 seconds)``  

```MariaDB [(no)]> CREATE A CUSTOM moodle IDENTIFIED AS 'P@ssw0rd';"`  
```Request K, 0 rows affected (0.004 seconds)``  

```MariaDB [(no)]> GRANTS moodle ALL PRIVILEGES ON moodledb.*;"`  
```The query is OK, rows are affected (0.003 seconds)``  

```MariaDB [(no)]> RESET PRIVILEGES;`  
```The query is OK, 0 lines are affected (0.002 seconds)``  

```MariaDB [(no)]>``  

---download MOODLE in the online version---  

```curl -L https://tinyurl.com/2z2btyrz > /root/moodle.zip```  

---Configure it in /var/www/html/ for long-range navigation---  

```unpack /root/moodle.zip -d /var/www/html```  
```mv /var/www/html/moodle-4.5.0/* /var/www/html/``  
```ls /var/www/html```  

---Create a new moodledata directory, where you can find data and use html and moodledata---  

```mkdir /var/www/moodledata```  
```launching apache2:apache2 /var/www/html```  
```launch apache2:apache2 /var/www/moodledata```  

---Note the value of the max_input_vars parameter in the php.ini file---  

```vim /etc/php/8.2/apache2-mod_php/php.ini```  
```  
~~~  
```How many GET/POST/COOKIE input variables can be accepted```  
``; max_input_vars = 1000```  

`; The maximum amount of memory that a script can occupy (128 MB)``  
``; http://php.net/memory-limit```  
```memory_limit = 128 MB```  

```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  
`; Error handling and logging ;`  
```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  

```This directive informs PHP about what errors, warnings and notifications you would like to receive```  
```actions must be taken to achieve this. The recommended way to set values for this is```  
``/max_input_va```  
~~~
```  

--- !Uncomment the string, the new value is 5000---  

---We invite you to cooperate **apache**---  

```rm -f /var/www/html/index.html```  

---------Enabling the httpd2 service---------  

```systemctl restarting httpd2```  

---Join the client and get acquainted with the mood---  

```http://192.168.1.10/install.php```  

---Choosing MariaDB as a data base partner---  

![image](https://github.com/user-attachments/assets/15196dbc-0181-4cc0-afc8-2117c6e2d10c )

---View everything in the **“OK"** or “Check” and continue **“Continue”**---  

---Fill in the required fields to create a primary administrator---  

![image](https://github.com/user-attachments/assets/9c6f2f25-0893-400a-a11d-dffaaa19ff85 )

| Fill in some more lines in the next step: | |
|---------------------------------------------------|----------------------------------|
| Full name of the website: | moodle |
| Short name of the website: | XX (workplace number) |
| Location settings:                         | Europe/Moscow (according to region) |
| Support Service contacts: | any@random.com (more) |


--------- Let's add the client machine to the domain---------  
 
---It is impossible to install DNS on the user's computer---  

![image](https://github.com/user-attachments/assets/4f20d78e-b2dd-4124-8564-f66e88731862 )

---For convenience, we remember the export editor for use in vim---  
```Export EDITOR=/usr/bin/vim```
 
---------Management of the group leaders' headquarters---------  
 
---We will use the role mechanism for simplicity and speed.---  
  
---Install the required module---  
```apt-get installs the libnss role```  
  
---create a local hq_users group---  
```add the hq_users group```  
  
---configure the role, for which the members of the local hq_users group automatically become members of the local hq_users group---  
```vim /etc/role```  
  
```  
~~~  
Domain Users:users  
Domain Administrators:localadmins  

Headquarters:hq_users  
~~~  
```  
  
---The government's commitment to using the court by all users---  
```public control```    
  
--- Indeed, we need a commitment to fighting pederasts. Let's use this to improve our eyesight, which checks the sensitivity of the sensitivity.---  
 
```Visudo```  

---Add the following block to the file (conveniently at the end)---   

```  
~~~  
###############  

Cmnd_Alias HQ = /usr/bin/cat, /bin/grep, /usr/bin/id  

%hq_users ALL=(ALL) NO ACCESS: HQ  

###############  
~~~  
```  
  
---Where---
```  
Cmnd_Alias HQ = /usr/bin/cat, /bin/grep, /usr/bin/id is the Alias HEADQUARTERS in which  
a set of executable files is described  
%hq_users ALL=(ALL) NO access: HQ – connects the hq_users group with other HQ users and
sets up a win-win trial  
```  
  
---For verification, we register together with a user from a separate hq group---

![image](https://github.com/user-attachments/assets/7fac3a25-6bcb-4298-add5-9df66bd9e809 )
  
---We check the employees in the local and domain group utilitoy id---

![image](https://github.com/user-attachments/assets/f2a07ebb-28ed-4534-9382-a6e308ec4bfa )
  
---The domain user is a member of the local group---  
--- So we want to help you sudo---

![image](https://github.com/user-attachments/assets/12ab551d-533d-4c68-9b18-142a2a831477 )
  
---You can only use the court ID due to entering a password---  
---And it is not allowed to perform other tasks at the same time---

![image](https://github.com/user-attachments/assets/ea83943a-f768-48ba-9b61-bdc826849ecd )
  
---Install the NFS server---  
  
```apt-get install nfs client```  
```  
Reading the package lists... Made by  
Building a dependency tree... Made by  
nfs-clients already has the latest version.    
0 updated, 0 recently installed, 0 deleted, and 629 not updated.  
```  
  
---Checking the availability of resources---  
  
```showmount -e 192.168.3.10```  
```  
Export list for 192.168.3.10:
/srv/public *
/raid5/nfs 192.168.2.0/28  
```  
  
---Creating a catalog---  
  
```mkdir /mnt/nfs```  
```mount -t nfs 192.168.3.10:/raid5/nfs/mnt/nfs```  
```df /mnt/nfs```  
  
```  
The file system size used, %, is set to  
192.168.3.10:/raid5/nfs 2.0G 0 1.9G 0% /mnt/nfs  
```

```umount -a```  

```
umount: /run/user/1080001104: the target object is busy.  
umount: /run/user/0: The target object is busy.  
umount: /tmp: the target object is busy.  
umount: /sys/fs/cgroup: the target object is busy.  
umount: /: the target object is busy.  
umount: /run: The target object is busy.  
umount: /dev: The target object is busy.  
```

---Add the following line to the end of the /etc/fstab file---  
  
```192.168.3.10:/raid5/nfs /mnt/nfs nfs intr,soft,_netdev 0  0```
  
```
intr/nointr — turn off/on mounting operations by ctrl+C  
soft — prevents hanging, in case of resource unavailability  
_netdev — Initialization system management, mounting after network access  
```  
  
------------------Install the Yandex Browser application for organizations on the Main Client---------
------ Install the Yandex Browser on the Main Client via the terminal using the commands---  
  
```apt-get install yandex-browser-stable```    
  
-------port forwarding---------  
---NAT port forwarding---
  
====================Main Router==========  
  
```(config)#ip nat source static tcp 192.168.1.10 2024 172.16.4.4 2024```  
  
====================Branch Router==========  
  
```(config)#ip nat source static tcp 192.168.3.10 8080 172.16.5.5 80```  
```(config)#ip nat source static tcp 192.168.3.10 2024 172.16.5.5 2024```
