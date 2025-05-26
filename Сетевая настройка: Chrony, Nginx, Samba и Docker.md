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

```---------BR-SRV ---------```

```---------Настройка доменного контроллера Samba на машине BR-SRV---------```

```SambaAD на Alt```

```apt-get update && apt-get install task-samba-dc -y```

```удаляем файл /etc/samba/smb.conf ДО НАЧАЛА УСТАНОВКИ!!!```

```rm -f /etc/samba/smb.conf```

```---------Начинаем установку:---------```

```samba-tool domain provision -–interactive```

``` Realm [AU-TeAM.IrPO] :```
``` Server Role (dc, member, standalone) [dc] :```
``` DNS backend (SAMBA_intERnal, Bind9_flAtfile, bind9_dlz, none) [SAMBA_intERnaL] :```
``` DNS forwarder IP address (write 'none' to disable forwarding) [192.168.1.10] :```
``` Administrator password:```
``` Retype password:```

```! В качестве пароля указываем P@ssw0rd```

```---------Запускаем сервис---------```

```systemctl enable --now samba.service```

```---------Меняем адрес DNS сервера на свой локальный---------```

```vim /etc/net/ifaces/ens19/resolv.conf```

```~~~```
```nameserver 127.0.0.1```
```search au-team.irpo```
```~~~```

```---------Перезапускаем сетевую службу---------```

```systemctl restart network```

```---------Перемещаем сгенерированный конфиг krb5.conf:---------```

```mv -f /var/lib/samba/private/krb5.conf /etc/krb5.conf```

```---------Проверяем состояние домена---------```

```samba-tool domain info 127.0.0.1```

```---------Создадим пользователей---------```

```---------Создаем пользователей, создаем группу---------```

```samba-tool user add user1.hq P@ssw0rd```
```samba-tool user add user2.hq P@ssw0rd```
```samba-tool user add user3.hq P@ssw0rd```
```samba-tool user add user4.hq P@ssw0rd```
```samba-tool user add user5.hq P@ssw0rd```

```Добавляем пользователей в группу```
```samba-tool group addmembers hq user1.hq,user2.hq,user3.hq,user4.hq,user5.hq```

```---------проверим---------```

```samba-tool group listmembers hq```

```---------Смотрим настройки пользователя---------```

```samba-tool user show user1.hq```

```---------Разблокируем УЗ---------```

```samba-tool user setexpiry user1.hq --noexpiry```

```Повторная проверка показывает, что мы разблокировали УЗ```

```accountExpires: 0```
```logonCount: 20250111103606.0z```

```Выполните импорт пользователей из файла users.csv.```

```Создаём файл import.sh```

```~~~```
```#!/bin/bash```
```csv_file="/opt/users.csv"```
```while IFS=";" read -r firstName lastName role phone ou street zip city country password; do```
```if [ "
f
i
r
s
t
N
a
m
e
"
=
=
"
F
i
r
s
t
N
a
m
e
"
]
;
t
h
e
n
@
@
@
@
@
@
c
o
n
t
i
n
u
e
@
@
@
@
@
@
f
i
@
@
@
@
@
@
u
s
e
r
n
a
m
e
=
"
firstName"=="FirstName"];then``````continue``````fi``````username="{firstName,,}.
l
a
s
t
N
a
m
e
,
,
"
@
@
@
@
@
@
s
a
m
b
a
−
t
o
o
l
u
s
e
r
a
d
d
"
lastName,,"``````samba−tooluseradd"username" P@ssw0rd1```
```done < "$csv_file"```
```~~~```

```даем право на исполнение```
```chmod +x import.sh```

```и запускаем```
```./import.sh```

```После выполнения скрипта проверяем```
```samba-tool user list```

```---------Сконфигурируйте файловое хранилище---------```

```Проверяем наличие дисков```

```lsblk```

```---------Программный RAID в ALT---------```

```Создадим дисковый массив уровня 5 из трёх дополнительных дисков следующей командой:```
```mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sd[b-d]```

```Посмотрим статус raid-массива:```

```cat /proc/mdstat```

```Сохраним конфигурацию массива в файл /etc/mdadm.conf следующей командой:```

```mdadm --detail -scan --verbose > /etc/mdadm.conf```

```теперь необходимо создать раздел```
```parted /dev/md0```

```1) Необходимо создать таблицу разделов. Используем самый простой и распространённый тип MBR (msdos).```
```parted /dev/md0```
```GNU Parted 3.2.46-e4ae```
```Using /dev/md0```
```Welcome to GNU Parted! Type 'help' to view a list of command.```
```(parted) mktable msdos```

```2) Посмотрим таблицу разделов, чтобы выяснить размер свободного пространства.```
```(parted) print```

```3) Создадим раздел.```
```(parted) mkpart primary ext4 1 2143MB```

```4) Снова посмотрим таблицу разделов```
```(parted) print```

```5) Выходим из parted. Команда quit.```

```---------Проверим lsblk---------```

```lsblk /dev/md0```

```Теперь создадим файловую систему, по заданию требуется ext4, создаём её следующей командой:```
```mkfs.ext4 /dev/md0p1```

```---------Настроим автоматическое монтирование в /raid5---------```

```Создаём каталог /raid5:```
```mkdir /raid5```

```---------Обеспечим автоматическое монтирование в папку /raid5---------```

```Для этого добавим в конец файла /etc/fstab:```
```/dev/md0p1 /raid5 ext4 defaults 0 0```

```---------Выполним команду монтирования---------```

```mount -a```

```Посмотрим точки монтирования командой df```

```---------Настройте сервер сетевой файловой системы(nfs)---------```

```---------Создадим файловые ресурсы и настроим права к ним---------```

```mkdir /raid5/nfs```
```chmod 777 /raid5/nfs```

```---------Установим необходимое ПО---------```

```apt-get install nfs-server```

```---------запустим сервер NFS---------```

```systemctl enable --now nfs-server```

```---------пропишем доступ к каталогу в файле /etc/exports---------```

```vim /etc/exports```

```##########```
```/raid5/nfs 192.168.2.0/28(rw,no_subtree_check)```
```##########```

```Применим наши настройки```

```exportfs -vra```

```---------Ansible на сервере BR-SRV---------```

```---------Предварительные настройки---------```

```Включаем SSH на HQ-CLI```

```systemctl enable --now sshd.service```

```---------Создаем ключевую пару на BR-SRV---------```

```ssh-keygen```

```---------Настраиваем бесключевой доступ на HQ-SRV и HQ-CLI---------```

```ssh-copy-id user@192.168.1.10```
```ssh-copy-id user@192.168.2.10```

```Устанавливаем ansible:```
```apt-get install ansible```

```редактируем конфиг:```
```vim /etc/ansible/ansible.cfg```

```[defaults]```

```inventory = /etc/ansible/hosts```
```interpreter_python = auto_silent```

```---------Создаем инвентарный файл:---------```

```vim /etc/ansible/hosts```

```[Linux]```
```hq-srv ansible_host=user@192.168.1.10```
```hq-cli ansible_host=user@192.168.2.10```

```[ECO_ROUTERS]```
```hq-rtr ansible_host=192.168.5.1```
```br-rtr ansible_host=192.168.5.2```

```[ECO_ROUTERS:vars]```

```ansible_connection=network_cli```
```ansible_network_os=ios```
```ansible_user-admin```
```ansible_password-admin```

```---------Выполняем проверку доступности---------```

```ansible all -m ping```

```---------Развертывание приложений в Docker на сервере BR-SRV---------```

```---------Установить docker-engine и docker-compose---------```

```apt-get install docker-engine docker-compose -y```

```---------запустим службу docker:---------```

```systemctl enable --now docker```

```---------добавляем пользователя sshuser в группу doсker, что он имел возможность работать с контейнерами:---------```

```usermod sshuser -aG docker```
```grep docker /etc/group```

```---------Переходим в контекст пользователя sshuser---------```

```su -l sshuser```

```---------Загружаем образы следующей командой---------```

```docker pull mediawiki```
```docker pull mariadb```

```---------Создаем в домашней директории пользователя файл wiki.yml для приложения MediaWiki.---------```

```vim wiki.yml```

```services:```
``` wiki:```
``` image: mediawiki```
``` container_name: wiki```
``` environment:```
``` MEDIAWIKI_DB_HOST: mariadb```
``` MEDIAWIKI_DB_USER: wiki```
``` MEDIAWIKI_PASSWORD: WikiP@ssw0rd```
``` MEDIAWIKI_DB_NAME: mediawiki```
``` ports:```
``` -8080:80```
```# volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]```
``` db:```
``` image: mariadb```
``` container_name: mariadb```
``` hostname: mariadb```
``` environment:```
``` MYSQL_DATABASE: mariadb```
``` MYSQL_USER: wiki```
``` MYSQL_PASSWORD: WikiP@ssw0rd```
``` volumes:```
``` - ./db:/var/lib/mysql```
```volumes:```
``` db:```

```---------Проверяем конфигурацию---------```

```docker-compose -f wiki.yml config```

```---------после проверки, запускаем---------```

```docker-compose -f wiki.yml up -d```

```---------!Переходим на HQ-CLI, в браузер по адресу http://192.168.3.10:8080---------```

```scp LocalSettings.php sshuser@192.168.3.10:/home/sshuser/```

```cat LocalSettings.php```

```---------После этого останавливаем все контейнеры---------```

```docker stop $(docker ps -a -q)```

```---------удаляем все контейнеры---------```

```docker rm $(docker ps -a -q)```

```---------Раскомментируем строку wiki.yml---------```

``` - 8080:80```
``` volumes: [ ~/LocalSettings.php:/var/www/html/LocalSettings.php ]```

```---------и снова запускаем wiki.yml---------```

```docker-compose -f wiki.yml up -d```

---------HQ-SRV---------  

---------Запустите сервис moodle на сервере HQ-SRV---------  
```Устанавливаем для ряд пакетов, которые будут нам нужны для работы:```  

```apt-get update && apt-get install apache2 php8.2 apache2-mod_php8.2 mariadb-server php8.2-opcache php8.2-curl php8.2-gd php8.2-intl php8.2-mysqli php8.2-xml php8.2-xmlrpc php8.2-ldap php8.2-zip php8.2-soap php8.2-mbstring php8.2-json php8.2-xmlreader php8.2-fileinfo php8.2-sodium```  

---------Включаем службы httpd2 и mysqld---------  

```systemctl enable –-now httpd2 mysqld```  

---------Настроим безопасный доступ к нашей будущей базе данных с помощью команды:---------  

```mysql_secure_installation```  

---------меняем пароль на P@ssw0rd, все остальное по умолчанию---------  
---------заходим в СУБД для создания и настройки базы данных:---------  

```mariadb -u root -p```  

```MariaDB [(none)]> CREATE DATABASE moodledb;```  
```Query OK, 1 row affected (0.002 sec)```  

```MariaDB [(none)]> CREATE USER moodle IDENTIFIED BY 'P@ssw0rd';```  
```Query K, 0 rows affected (0.004 sec)```  

```MariaDB [(none)]> GRANT ALL PRIVILEGES ON moodledb.* TO moodle;```  
```Query OK, rows affected (0.003 sec)```  

```MariaDB [(none)]> FLUSH PRIVILEGES;```  
```Query OK, 0 rows affected (0.002 sec)```  

```MariaDB [(none)]>```  

---------скачаем MOODLE стабильной версии---------  

```curl -L https://tinyurl.com/2z2btyrz > /root/moodle.zip```  

---------Разархивируем его в /var/www/html/ для дальнейшей настройки:---------  

```unzip /root/moodle.zip -d /var/www/html```  
```mv /var/www/html/moodle-4.5.0/* /var/www/html/```  
```ls /var/www/html```  

---------Создадим новый каталог moodledata, там будут храниться данные и изменим владельца на каталогах html и moodledata:---------  

```mkdir /var/www/moodledata```  
```chown apache2:apache2 /var/www/html```  
```chown apache2:apache2 /var/www/moodledata```  

---------Поменяем значение параметра max_input_vars в файле php.ini:---------  

```vim /etc/php/8.2/apache2-mod_php/php.ini```  

```; How many GET/POST/COOKIE input variables may be accepted```  
```; max_input_vars = 1000```  

```; Maximum amount of memory a script may consume (128MB)```  
```; http://php.net/memory-limit```  
```memory_limit = 128M```  

```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  
```; Error handling and logging ;```  
```;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;```  

```; This directive informs PHP of which errors, warnings and notices you would like```  
```; it to take action for. The recommended way of setting values for this```  
```/max_input_va```  

```! Раскоментируем строку, новое значение 5000```  

---------Удаляем стандартную страницу apache---------  

```rm -f /var/www/html/index.html```  

---------Перезапускаем службу httpd2:---------  

```systemctl restart httpd2```  

---------Подключаемся с клиента HQ-CLI и начинаем настройку:---------  

```http://192.168.1.10/install.php```  

```Выбираем MariaDB в качестве драйвера базы данных```  

```Выбираем MariaDB в качестве драйвера базы данных```  
