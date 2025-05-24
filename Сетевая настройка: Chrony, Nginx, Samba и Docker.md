---------ISP---------  
  
---------Настройка сервера времени chrony на машине ISP---------  
  
---------Настройка chrony на Alt---------
  
```apt-get update && apt-get install chrony nginx```  
  
---------Вносим изменения в /etc/chrony.conf--------- 
```vim /etc/chrony.conf```  
  
```~~~```  
```#########```  
```server ntp1.vniiftri.ru iburst```  
```local stratum 5```  
```allow all```  
```######```  
```#Record the rate at which the system... <--- (Чтобы определить нужную строку)```  
```~~~```  
  
```Где```  
```server ntp1.vniiftri.ru iburst  эталонный сервер времени```  
```local stratum 5        локальный стратум```  
```allow all           разрешение для всех клиентов```  
  
```Запустим chrony и поставим в автозагрузку```  
```systemctl enable –now chronyd```  
  
---------на клиентах необходимо будет установить chrony, и прописать в  chrony.conf строку---------
```на HQ-SRV, HQ-CLI:```  
```server 172.16.4.1 iburst```  
  
```на BR-RTR, BR-SRV:```  
```server 172.16.5.1 iburst```  
  
---------ISP---------  
  
---------Настройте веб-сервер nginx как обратный прокси-сервер---------  
  
```Настройка обратного прокси Nginx```  
```apt-get install nginx```  
  
---------Создаём новый конфигурационный файл proxy---------  
```/etc/nginx/sites-available.d/proxy.conf```  
  
```server {```  
```listen 80;```  
```server_name moodle.au-team.irpo;```  
```location / {```  
```proxy_pass http://192.168.1.10:80;```  
```proxy_set_header Host $host;```  
```proxy_set_header X-Real-IP  $remote_addr;```  
```proxy_set_header X-Forwarded-For $remote_addr;```  
```}```  
```}```  
  
```server {```  
```listen 80;```  
```server_name wiki.au-team.irpo;```  
```location / {```  
```proxy_pass http://192.168.3.10:8080;```  
```proxy_set_header Host $host;```  
```proxy_set_header X-Real-IP  $remote_addr;```  
```proxy_set_header X-Forwarded-For $remote_addr;```  
```}```  
```}```  
  
---------Включаем созданную нами (proxy.conf), путём создания  ссылки---------  
```ln /etc/nginx/sites-available.d/proxy.conf /etc/nginx/sites-enabled.d/proxy.conf```  
```ls -la /etc/nginx/sites-enabled```  
  
---------Проверьте конфигурацию на наличие ошибок---------  
  
```nginx -t```  
```Затем запускаем службу nginx:```  
```systemctl enable --now nginx```

---------BR-SRV ---------

---------Настройка доменного контроллера Samba на машине BR-SRV---------
 
SambaAD на Alt  
 
apt-get update && apt-get install task-samba-dc -y 
 
удаляем файл /etc/samba/smb.conf ДО НАЧАЛА УСТАНОВКИ!!! 
 
rm -f /etc/samba/smb.conf 

---------Начинаем установку:---------
 
samba-tool domain provision  -–interactive 

 Realm [AU-TeAM.IrPO] : 
 Server Role (dc, member, standalone) [dc] : 
 DNS backend (SAMBA_intERnal, Bind9_flAtfile, bind9_dlz, none) [SAMBA_intERnaL] : 
 DNS forwarder IP address (write 'none' to disable forwarding) [192.168.1.10] : 
 Administrator password:
 Retype password:
 
 info 2025-01-27 19:09:05,088 pid: 16982 /usr/lib64/samba-dc/python3.9/samba/provi 
 info 2025-01-27 19:09:05,092 pid: 16982 /usr/lib64/samba-dc/python3.9/samba/provi iG5-01-27 1099 hisr/lib64/sambsm
 
 (Это окно терминала, в котором отображаются выходные данные инструмента командной строки, вероятно, связанные с настройкой Samba.  Показано множество строк текста, включая команды, сообщения и параметры конфигурации.  В выходных данных представлены настройки для предоставления домена и настройки DNS.  В тексте приведены различные настройки и конфигурации, такие как область, домен, роль сервера, серверная часть DNS, IP-адреса пересылки и пароли администратора.)
 
 ! В качестве пароля указываем P@ssw0rd 
 
---------Запускаем сервис---------

systemctl enable --now samba.service
Synchronizing state of samba.service with SysV service with /lib/sys
Executing: /lib/systemd/systemd-sysv-install enable samba

(Вывод командной строки, показывающей выполнение команд systemctl для активации службы samba.  В выводе перечислены операции:  включение службы samba, синхронизация состояния службы с системным скриптом SysV и создание символической ссылки.)

---------Меняем адрес DNS сервера на свой локальный---------
vim /etc/net/ifaces/ens19/resolv.conf 

~~~
nameserver 127.0.0.1
search au-team.irpo
~~~

---------Перезапускаем сетевую службу---------
systemctl restart network 

---------Перемещаем сгенерированный конфиг krb5.conf:---------
mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf 
 
---------Проверяем состояние домена---------
samba-tool domain info 127.0.0.1

---------
Forest	: au-team.irpo
Domain	: au-team.irpo
Netbios domain	: AU-TEAM
DC name	: br-srv.au-team.irpo
DC netbios name	: BR-SRV
Server site	: Default-First-Site-Name
Client site	: Default-First-Site-Name
---------

Домен работает. 
---------Создадим пользователей---------
 
---------Создаем пользователей, создаем группу---------

samba-tool user add user1.hq P@ssw0rd
User 'user1.hq' added successfully

samba-tool user add user2.hq P@ssw0rd
User 'user2.hq' added successfully

samba-tool user add user3.hq P@ssw0rd
User 'user3.hq' added successfully

samba-tool user add user4.hq P@ssw0rd
User 'user4.hq' added successfully

samba-tool user add user5.hq P@ssw0rd
User 'user5.hq' added successfully

Добавляем пользователей в группу 
samba-tool group addmembers hq 
user1.hq,user2.hq,user3.hq,user4.hq,user5.hq 

---------проверим---------
samba-tool group listmembers hq
user5.hq
user4.hq
user3.hq
user2.hq
user1.hq

---------Смотрим настройки пользователя---------
samba-tool user show user1.hq 
 
Из соображений безопасности все создаваемые УЗ блокированы 

Параметр accountExpires: 

accountExpires: 9223372036854775807
logonCount: 0

Любое значение отличное от 0 обозначает, что учетная запись пользователя заблокирована 

---------Разблокируем УЗ---------
samba-tool user setexpiry user1.hq --noexpiry 

samba-tool user setexpiry user1.hq --noexpiry
Expiry for user 'user1.hq' disabled.

Повторная проверка показывает, что мы разблокировали УЗ 

accountExpires: 0
logonCount: 20250111103606.0z

Выполните  импорт  пользователей  из  файла  users.csv. 
 
Создаём файл import.sh 

~~~
#!/bin/bash 
# 
csv_file="/opt/users.csv" 
while IFS=";" read -r firstName lastName role phone ou street zip city 
country password; do 
if [ "$firstName" == "First Name" ]; then 
              continue 
fi 
username="${firstName,,}.${lastName,,}" 
        samba-tool user add "$username" P@ssw0rd1 
done < "$csv_file" 
~~~

даем право на исполнение 
chmod +x import.sh 
 
и запускаем 
./import.sh 
 
После выполнения скрипта проверяем 
samba-tool user list

---------Сконфигурируйте файловое хранилище---------
 
Проверяем наличие дисков

lsblk
Name MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sda	8:0 	0	10G 	0	disk	/
sdb	8:16	0	1G 		0	disk
sdc	8:32	0	1G 		0	disk
sdd	8:64	0	1G		0	disk

---------Программный RAID в ALT---------
Создадим дисковый массив уровня 5 из трёх дополнительных дисков следующей командой: 
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sd[b-d] 
 
Посмотрим статус raid-массива:
 
cat /proc/mdstat 
 
Сохраним конфигурацию массива в файл /etc/mdadm.conf следующей командой:

mdadm --detail -scan --verbose > /etc/mdadm.conf 
 
теперь необходимо создать раздел 
! Команда parted
воспользуемся утилитой parted, уже установленной в ALT
 
parted /dev/md0 

1) Необходимо создать таблицу разделов. Используем самый простой и распространённый тип MBR 
(msdos).
! Команда mktable msdos 

parted /dev/md0
GNU Parted 3.2.46-e4ae
Using /dev/md0
Welcome to GNU Parted! Type 'help' to view a list of command.
(parted) mktable mados

2) Посмотрим таблицу разделов, чтобы выяснить размер свободного пространства. 
! Команда print 

(parted) print
Model: Linux Software RAID Array (md)
Disk /dev/md0: 2143MB
Sector size (logical/physycal): 512B/512B/512B
Disk Flags:

Number Start End Size Type File system Flags

!Свободно 2143Mb

3) Создадим раздел.
! Команда mkpart primary ext4 1 2143М, где 
 
primary  	тип раздела 
ext4  		метка файловой системы 
1  			начало раздела 
2143М  		конец раздела

(parted) mkpart primary ext4 1 2143MB

(parted) print
Model: Linux Software RAID Array (md)
Disk /dev/md0: 2143MB
Sector size (logical/physycal): 512B/512B/512B
Partition Table: msdos
Disk Flags:

Number	Start	End	Size

4) Снова посмотрим таблицу разделов

(parted) print
Model: Linux Software RAID Array (md)
Disk /dev/md0: 2143MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number	Start	End		Size	Type	File	system	Flags
	1	1049kB	2143MB	2142MB	primary	ext4			lba
(parted)

5)  Выходим из parted. Команда quit. 

---------Проверим lsblk---------

lsblk /dev/md0

NAME	MAJ:MIN	RM	SIZE	RO	TYPE	MOUNTPOINTS
md0		9:0		0	2G		0	raid5
|
 --md0p1 259:1	0	2G		0	part
 
 Теперь создадим файловую систему, по заданию требуется ext4, создаём её следующей командой: 
mkfs.ext4 /dev/md0p1 

mke2fs 1.46.2 (28-Feb-2021)
Creating filesystem with 523008 4k blocks and 130816 inodes
Filesystem UUID: 9b32d258-f704-4319-b0b8-e700bf1f88f5
Superblock backups stored on blocks:
		32768, 98304, 163840, 229376, 294912
Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

---------Настроим автоматическое монтирование в /raid5--------- 
Создаём каталог /raid5: 
mkdir /raid5 

---------Обеспечим автоматическое монтирование в папку /raid5---------

Для этого добавим в конец файла /etc/fstab: 
/dev/md0p1  /raid5  ext4 defaults  0  0 

UUID=5cfdfaf0-839f-4578-ad0f-b52b0336dfe2	/	ext4	relatime
	1
/dev/sr0		/media/ALTLinux udf, iso9660	ro,noauto,user,utf8,nofail,comm...
gvfs-show	0 0

/dev/md0p1		/raid5	ext4	defaults		0		0

fstab  в ALT 
---------Выполним команду монтирования---------
mount -a 
! Не должно быть никаких сообщений! 
Посмотрим точки монтирования командой df 

Filesystem	Size	Used	Avail	Use&	Mounted on
udevfs		5.0M	64K		5.0M	2&		/dev
runfs		991M	608K	990M	1%		/run
/dev/sda	9.8G	3.0G	6.3G	32%		/
tmpfs		991M	0		991M	0%		/dev/shm
tmpfs		991M	0		991M	0%		/tmp
tmpfs		991M	0		991M	0%		/run/user/0
/dev/md0p1	2.0G	24K		1.9G	1%		/raid5

---------Настройте сервер сетевой файловой системы(nfs)---------

---------Создадим файловые ресурсы и настроим права к ним---------
mkdir /raid5/nfs 
chmod 777 /raid5/nfs

---------Установим необходимое ПО---------

apt-get install nfs-server 
NFS в ALT 

---------запустим сервер NFS---------
systemctl enable --now nfs-server 

---------пропишем доступ к каталогу в файле /etc/exports---------
vim /etc/exports 

##########
/raid5/nfs 192.168.2.0/28(rw,no_subtree_check)
##########

Применим наши настройки 

exportfs -vra
exporting 192.168.2.0/28:/raid5/nfs
exporting *:/srv/public

---------Ansible на сервере BR-SRV---------
 
---------Предварительные настройки---------

Включаем SSH на HQ-CLI

systemctl enable --now sshd.service
Synchronizing state of sshd.service with SysV service script with /lib/systemd/system
d-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable sshd
Created symlink /etc/systmd/system/multi-user.target.wanta/sshd.service -> /lib/systemd/system/sshd.service

---------Создаем ключевую пару на BR-SRV---------

ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ss/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.

---------Настраиваем бесключевой доступ на HQ-SRV и HQ-CLI---------

ssh-copy-id user@192.168.1.10
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be install

ssh-copy-id user@192.168.2.10
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be install
The authenticity of host '192.168.2.10 (192.168.2.10)' can...

 Устанавливаем ansible: 
apt-get install ansible 
 
редактируем конфиг: 
vim /etc/ansible/ansible.cfg

[defaults]

# some basic default values...

####################
inventory		= /etc/ansible/hosts
interpreter_python = auto_silent
####################

#library		= /usr/share/my_modules/

---------Создаем инвентарный файл:---------
vim /etc/ansible/hosts 

[Linux]
hq-srv ansible_host=user@192.168.1.10
hq-cli ansible_host=user@192.168.2.10

[ECO_ROUTERS]
hq-rtr ansible_host=192.168.5.1
br-rtr ansible_host=192.168.5.2

[ECO_ROUTERS:vars]

ansible_connection=network_cli
ansible_network_os=ios
ansible_user-admin
ansible_password-admin

---------Выполняем проверку доступности---------
ansible all -m ping

br-rtr | SUCCESS => {
	"ansible_facts": {
		"discovered_interpreter_python": "/usr/bin/python3"
	},
	"changed": false,
	"ping": "pong"
}

hq-rtz | SUCCESS =>	{
	"ansible_facts": {
		"discovered_interpreter_python": "/usr/bin/python3"
	},
	"changed": false,
	"ping": "pong"
}
hq-srv | SUCCESS => {
	"ansible facts": {
		"discovered_interpreter_python": "/usr/bin/python3"
	"changed": false,
	"ping": "pong"
}
hq-cli | SUCCESS => {
	"ansible facts": {
		"discovered_interpreter_python": "/usr/bin/python3"
	"changed": false,
	"ping": "pong"
}

---------Развертывание приложений в Docker на сервере BR-SRV---------
 
---------Установить docker-engine и docker-compose---------

apt-get install docker-engine docker-compose -y 

---------запустим службу docker:---------

systemctl enable --now docker
 
---------добавляем пользователя sshuser в группу doсker, что он имел возможность работать с контейнерами:---------

usermod sshuser -aG docker 
grep docker /etc/group

docker:x:465:sshuser

---------Переходим в контекст пользователя sshuser---------

su -l sshuser

---------Загружаем образы следующей командой---------

docker pull mediawiki 
docker pull mariadb 

---------Создаем в домашней директории пользователя файл wiki.yml для приложения MediaWiki.---------

vim wiki.yml

services:
	wiki:
	    image: mediawiki
	    container_name: wiki
	    environment:
	      MEDIAWIKI_DB_HOST: mariadb
	      MEDIAWIKI_DB_USER: wiki
	      MEDIAWIKI_PASSWORD: WikiP@ssw0rd
	      MEDIAWIKI_DB_NAME: mediawiki
	    ports:
			-8080:80
			
#		volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]
	db:
	  image: mariadb
	  container_name: mariadb
	  hostname: mariadb
	  environment:
		MYSQL_DATABASE: mariadb
		MYSQL_USER: wiki
		MYSQL_PASSWORD: WikiP@ssw0rd
	  volumes:
	   - ./db:/var/lib/mysql
	  
volumes:
  db:

---------Проверяем конфигурацию---------

docker-compose -f wiki.yml config 

---------после проверки, запускаем---------

 docker-compose -f wiki.yml up -d
 
"wiki.yml" 26L, 620B written
[sshuser@BR-SRV ~]$ docker compose -f wiki.yml up -d
[+] Running 2/2
\ Container mariadb Started
\ Container wiki Started

---------!Переходим на HQ-CLI, в браузер по адресу http://192.168.3.10:8080---------

---

Заполняем поля, Далее

---

По окончании настройки загружаем LocalSettings.php 
Копируем его на BR-SRV, в домашний каталог sshuser 

scp LocalSettings.php sshuser@192.168.3.10:/home/sshuser/

sshuser@192.168.3.10's password:

cat LocalSettings.php

?php
This file was automatically generated by the MediaWiki 1.43.0
installer. If you make manual changes, please keep track in case you
need to recreate them later.

see includes/MainConfigSchema.php for all configurable setting

---------После этого останавливаем все контейнеры---------

docker stop $(docker ps -a -q) 

---------удаляем все контейнеры---------

docker rm $(docker ps -a -q) 
 
---------Раскомментируем строку wiki.yml---------

	   - 8080:80
	volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]
	
! '#' - убрять комментарий

---------и снова запускаем wiki.yml---------

docker-compose -f wiki.yml up -d 

---------HQ-SRV---------
 
---------Запустите сервис moodle на сервере HQ-SRV---------
Устанавливаем для ряд пакетов, которые будут нам нужны для работы: 

apt-get update && apt-get install apache2 php8.2 apache2-mod_php8.2 mariadb-server php8.2-opcache php8.2-curl php8.2-gd php8.2-intl php8.2-mysqli php8.2-xml php8.2-xmlrpc php8.2-ldap php8.2-zip php8.2-soap php8.2-mbstring php8.2-json php8.2-xmlreader php8.2-fileinfo php8.2-sodium 
 
---------Включаем службы httpd2 и mysqld---------

systemctl enable –-now httpd2 mysqld 

---------Настроим безопасный доступ к нашей будущей базе данных с помощью команды:---------

mysql_secure_installation 
 
---------меняем пароль на P@ssw0rd, все остальное по умолчанию---------
---------заходим в СУБД для создания и настройки базы данных:---------

mariadb -u root -p 

MariaDB [(none)]> CREATE DATABASE moodledb;
Query OK, 1 row affected (0.002 sec)

MariaDB [(none)]> CREATE USER moodle IDENTIFIED BY 'P@ssw0rd';
Query K, 0 rows affected (0.004 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON moodledb.* TO moodle;
Query OK, rows affected (0.003 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]>

---------скачаем MOODLE стабильной версии---------

curl -L https://tinyurl.com/2z2btyrz > /root/moodle.zip 

---------Разархивируем его в /var/www/html/ для дальнейшей настройки:---------

unzip /root/moodle.zip -d /var/www/html 
mv /var/www/html/moodle-4.5.0/* /var/www/html/ 
ls /var/www/html 

---------Создадим новый каталог moodledata, там будут храниться данные и изменим владельца на 
каталогах html и moodledata:---------

mkdir /var/www/moodledata 
chown apache2:apache2 /var/www/html 
chown apache2:apache2 /var/www/moodledata 

---------Поменяем значение параметра max_input_vars в файле php.ini:---------

vim /etc/php/8.2/apache2-mod_php/php.ini 

; How many GET/POST/COOKIE input variables may be accepted
; max_input_vars = 1000

; Maximum amount of memory a script may consume (128MB)
; http://php.net/memory-limit
memory_limit = 128M

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; Error handling and logging ;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

; This directive informs PHP of which errors, warnings and notices you would like
; it to take action for. The recommended way of setting values for this
/max_input_va

! Раскоментируем строку, новое значение 5000 

---------Удаляем стандартную страницу apache---------

rm -f /var/www/html/index.html 

---------Перезапускаем службу httpd2:---------

systemctl restart httpd2 

---------Подключаемся с клиента HQ-CLI и начинаем настройку:---------

http://192.168.1.10/install.php 

Выбираем MariaDB в качестве драйвера базы данных 


